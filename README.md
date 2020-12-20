// Juego-en-c

/*-------------------------------------------------------------------------------------------------------------------------------------------Parte de Jeferson*/
#include <vector>
#include <windows.h>
#include <cstdio>
#include <cstdlib>
#include <iostream>
#include <conio.h>
#include <string>

// Simbolos o Caracteres
#define CARACTER_FANTASMA 2

// Movimiento con las teclas
#define TECLA_ARRIBA       'i'
#define TECLA_ABAJO        'k'
#define TECLA_IZQUIERDA    'j'
#define TECLA_DERECHA      'l'

// Dirección de pacman
#define DIRECCION_IZQUIERDA 0
#define DIRECCION_DERECHA   1
#define DIRECCION_ARRIBA    2
#define DIRECCION_ABAJO     3
#define DIRECCION_INVALIDA  -1


// Define los colores
#define COLOR_PARED            0x009 // AZUL CLARO
#define COLOR_COMIDA           0x00F // BLANCO BRILLANTE
#define COLOR_PACMAN           0x00E // AMARILLO
#define COLOR_FANTASMA1        0x00C // ROJO CLARO
#define COLOR_FANTASMA2        0x00D  // PÚRPURA CLARO
#define COLOR_FANTASMA3        0x00B // AGUAMARINA CLARO
#define COLOR_FANTASMA4        0x00A // VERDE CLARO
#define COLOR_MARCADOR         0x00F   // BLANCO_BRILLANTE
#define COLOR_FANTASMA_ZOMBIE  0x004 // AZUL

#define BLANCO_BRILLANTE 0x00F

#define NADA 0

// Medidas del MAPA
#define FILAS_MAPA         25
#define COLUMNAS_MAPA      29
#define LIMITE_IZQUIERDO   1
#define LIMITE_SUPERIOR    0

#define LIMITE_DERECHO (LIMITE_IZQUIERDO + COLUMNAS_MAPA - 1)
#define LIMITE_INFERIOR (LIMITE_SUPERIOR + FILAS_MAPA - 1)

// Estados finales de pacman
#define PACMAN_VIVO        10    // Se encuentra jugando
#define PACMAN_MUERTO      20    // Ha muerto
#define PACMAN_GANADOR     30    // Ya ganó

#define VIDAS_PACMAN 4
#define CARACTER_PACMAN 2

// Modos de estado de los fantasmas
#define MODO_CAZADOR 1000     // Este es el modo por defecto de los fantasmas
#define MODO_PRESA   1001     // Este modo es cuando pacman ha comida la píldora mágica
#define MODO_INVISIBLE 1002   // Este modo es cuando el fantasma ha sido comido por pacman y está regresando a su casa

#define TIEMPO_ZOMBIE 50

// Función para el manejo de los colores 
int backcolor = 0; 
void setCColor(int color)
{
   static HANDLE hConsole;
   
   hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
   
   SetConsoleTextAttribute(hConsole, color | (backcolor * 0x10 + 0x100));
}



char mapa[FILAS_MAPA][COLUMNAS_MAPA + 1] = {
   "ahhhhhhhhhhhhhnhhhhhhhhhhhhhb",
   "v@············v············@v",
   "v·ahhb·ahhhhb·v·ahhhhb·ahhb·v",
   "v·v  v·v    v·v·v    v·v  v·v",
   "v·chhd·chhhhd·v·chhhhd·chhd·v",
   "v···························v",
   "v·hhhh·v·hhhhhnhhhhh·v·hhhh·v",
   "v······v······v······v······v",
   "chhhhb·phhhhh·v·hhhhho·ahhhhd",
   "     v·v······i······v·v     ",
   "     v·v·ahhhh2hhhhb·v·v     ", // 2 = para que vaya hacia arriba
   "hhhhhd·v·v    2    v·v·chhhhh",
   "·········v    2    v·········",
   "hhhhhb·v·v    2    v·v·ahhhhh",
   "     v·v·chhhhhhhhhd·v·v     ",
   "     v·v·············v·v     ",
   "ahhhhd·v·hhhhhnhhhhh·v·chhhhb",
   "v·············v·············v",
   "v·hhhb·hhhhhh·v·hhhhhh·ahhh·v",
   "v····v·················v····v",
   "phhh·v·v·hhhhhnhhhhh·v·v·hhho",
   "v······v······v······v······v",
   "v·hhhhhmhhhhh·v·hhhhhmhhhhh·v",
   "v@·························@v",
   "chhhhhhhhhhhhhhhhhhhhhhhhhhhd",
};
/* v : vertical
   h: horizontal
   m: medio y hacia arriba   l
   n: medio y hacia abajo    -j- 
   o: medio y hacia la izquierda -|
   p: medio y hacia la derecha   |-
   a:  esquina superior izquierda 
   b:  esquina superior derecha
   c:  esquina inferior izquierda
   d:  esquina inferior derecha
   s: salida => se transformara a hosizontal
*/

int cantidad_comida = 290;    // Guarda la cantidad de comida que hay en el mapa al principio del juego


// Esta matriz va a guardar todos los moviemientos hacia donde se va pacman. Si es que
// pacman no pasó por algún lugar, simplemente se le asigna el caracter a que significa
// aleatorio
short matriz_movimientos[FILAS_MAPA][COLUMNAS_MAPA];


using namespace std;


/*********** DECLARACIÓN DE CLASES **********/
class PERSONAJE;
class PACMAN;
class FANTASMA;

class PERSONAJE {
   private:
      int x, y;   // Coordenadas absolutas
      int simbolo;
      string nombre;
      int direccion;
      /* La direccion de pacman tiene  posibilidades:
         0 IZQUIERDA      : DIRECCION_IZQUIERDA
         1 DERECHA        : DIRECCION_DERECHA
         2 ARRIBA         : DIRECCION_ARRIBA
         3 ABAJO          : DIRECCION_ABAJO
         -1 NO SE MUEVE   : DIRECCION_INVALIDA
            
      */
      
      int color;
      
      int anteriorX, anteriorY;   // Coordenadas anteriores de x , y
      
   public:
      PERSONAJE(int, int = 1, int = 1, int = 1);
      void setX(int);
      void setY(int);
      void setSimbolo(int);
      void setDireccion(int);
      int getX();
      int getY();
      string getNombre();
      void setNombre(string);
      
      int getColor();
      void setColor(int);
      
      int getAnteriorX();
      int getAnteriorY();
      
      int getXRelativoAlMapa();
      int getYRelativoAlMapa();
      
      int getAnteriorXRelativoAlMapa();   // Obtiene las coordenadas anteriores relativas al mapa
      int getAnteriorYRelativoAlMapa();
      
      int getSimbolo();
      int getDireccion();
      virtual void mover(int);
      virtual void pintar();
      void borrar();
      
      int direccionVerdadera(int);
      
      bool esPosibleIrA(int posible_direccion);
      
}; // Fin de la declaración de la clase PERSONAJE



class PACMAN : public PERSONAJE {
   private:
      int puntaje;
      int vidas;     // Generalmente tiene tres vidas
      bool sigue_jugando;  // Cuando pierde todas sus vidas, ya no sigue jugando
      
   public:
      PACMAN(int color = COLOR_PACMAN, int x = LIMITE_IZQUIERDO + 14, int y = LIMITE_SUPERIOR + 15, int simbolo = CARACTER_PACMAN);
      void setPuntaje(int puntaje);
      int getPuntaje();
      void actualizar_puntaje();
      void setVidas(int vidas);
      int getVidas();
      void setSigueJugando(bool);
      bool getSigueJugando();
      bool choque(vector<FANTASMA> *,int &modo, FANTASMA **fantasma);
      int estadoFinalPacman(); // Retorna PACMAN_VIVO, PACMAN_MUERTO, PACMAN_GANADOR
      // No redefinimos el método pintar() así que se llamará al de la superclase
      void imprimirVidas();
      void reiniciaCoordenadas();
      bool comioPildora();
}; // Fin de la declaración de la clase PACMAN


class FANTASMA : public PERSONAJE {
   private:
      int modo;   // CAZADOR, PRESA
      int temporizador;
      bool temporizador_activado;
   public:
      // FANTASMA(int = (LIMITE_IZQUIERDO + LIMITE_DERECHO) / 2, int = (LIMITE_SUPERIOR + LIMITE_INFERIOR) / 2, int = 1);
      FANTASMA(int color, int = LIMITE_IZQUIERDO + 10, int = LIMITE_SUPERIOR + 12, int = CARACTER_FANTASMA);
      void mover();
      void setModo(int modo);
      int getModo();
      virtual void reiniciar_coordenadas();
      void setColorPredeterminado();
      int getTemporizador();
      void setTemporizador(int temporizador);
}; // Fin de la declaración de la clase FANTASMA


class FANTASMA_ROJO : public FANTASMA {
   private:
      
   public:
      FANTASMA_ROJO();
      void reiniciar_coordenadas();
};


class FANTASMA_ROSADO : public FANTASMA {
   private:
      
   public:
      FANTASMA_ROSADO();
      void reiniciar_coordenadas();
};


class FANTASMA_CELESTE : public FANTASMA {
   private:
      
   public:
      FANTASMA_CELESTE();
      void reiniciar_coordenadas();
};


class FANTASMA_VERDE : public FANTASMA {
   private:
      
   public:
      FANTASMA_VERDE();
      void reiniciar_coordenadas();
};
/******FIN DE LAS DECLARACIONES DE LAS CLASES********/


// Funciones
void pintar_mapa();        // Para pintar el mapa de juego con la comida
void gotoxy2(int, int);    // Similar a gotoxy pero con la API de windows
bool esPared(char);        // Determina si su argumento es un caracter correspondiente a la pared
void setCColor(int color); // Para establecer el color con lo que pintaremos

// Después hacer que esto sea una función miembro de pacman
void puntaje(PACMAN pacman, char mapa[][COLUMNAS_MAPA + 1]);   // Modifica el puntaje de pacman

void inicializaMatrizMovimientos();    // Inicializa el vector que guarda los movimientos de pacman
void imprimir_informacion(class PACMAN, int tecla_presionada);   // Imprime información del juego en tiempo real
void captura_tecla(int &tecla);  // Captura la tecla presionada y la modifica en caso se presione
void pintar_mapa(char mapa[][COLUMNAS_MAPA + 1], int filas, int columnas);
void empezar_juego();      // Función llamada por main para empezar el juego
void reinicia_personajes(PACMAN &pacman, vector<FANTASMA> &fantasmas);   // Regresa a pacman y a los fantasmas a sus coordenadas iniciales
void cuenta_regresiva(int x, int y, int limite_inferior, int limite_superior); // Hace una cuenta regresiva en las coordenadas
void PINTA_MATRIZ_MOVIMIENTOS(); // BORRAR ESTO ES DE PRUEBA

int main() 
{
   empezar_juego();
   return 0;   
}

/---------------------------------------------------------------Parte de Bryan------------------------------------------------------------------/

void empezar_juego() 
{
   system("color");
   // Inicializo la matriz que guarda los movimientos de pacman con -1 (es decir, aleatorio)
   inicializaMatrizMovimientos();
   
   // Pintamos el mapa donde estará pacman y toda la comida actual
   pintar_mapa(mapa, FILAS_MAPA, COLUMNAS_MAPA);
   
   // Creamos al pacman
   PACMAN pacman;
   
   // Creando los objetos FANTASMA en tiempo de compilación
   FANTASMA_ROJO    f1;
   FANTASMA_ROSADO  f2;
   FANTASMA_CELESTE f3;
   FANTASMA_VERDE   f4;

   // Guardamos los fantasmas en un vector de fantasmas
   vector <FANTASMA> fantasmas;
   fantasmas.push_back(f1);
   fantasmas.push_back(f2);
   fantasmas.push_back(f3);
   fantasmas.push_back(f4);
   
   // Pintamos a pacman por primera vez
   pacman.pintar();
   
   // Imprimimos las vida de Pacman
   pacman.imprimirVidas();
   
   // Comienza la cuenta regresiva
   cuenta_regresiva(LIMITE_IZQUIERDO + 14, LIMITE_SUPERIOR + 13, 1, 3);
   
   int tecla = DIRECCION_INVALIDA; ///////////////////// ver si esta línea es necesaria ¿?
   
   int tecla_salir;
   bool repintar = false;
   
   int modo_fantasma = MODO_CAZADOR;   // Guarda el modo en el que estaba el fantasma con el que choqué
   FANTASMA *fantasma = NULL;          // Apunta al fantasma chocado. Si estaba en modo cazador no sirve este dato
   
   reinicia_personajes(pacman, fantasmas);
   do {
      // Esto es antes de actualizar el puntaje porque al actualizar el puntaje se borra las comidas
      if (pacman.comioPildora()) { // Verificamos si pacman comió la píldora mágica)
         // Cambiamos el modo a todos los fantasmas
         for (int i = 0; i < fantasmas.size(); ++i) {
            fantasmas[i].setModo(MODO_PRESA);   // Este método se encarga de cambiar el color
         }
      }
      
      pacman.actualizar_puntaje(); // Actualizamos el puntaje de pacman

      
      captura_tecla(tecla);   // Capturamos la tecla presionada. Si no presiono nada, conserva su valor antes de entrar a la función
      pacman.mover(tecla);    // Movemos a pacman (Lo hago así para que pacman siempre esté pintado detrás de los fantasmas al ser comido)
      imprimir_informacion(pacman, tecla); // Imprimir la información actual: coordenadas, comida, puntaje, etc
      
      // Movemos a los fantasmas
      for (int i = 0; i < fantasmas.size(); ++i) {
         fantasmas[i].mover();
      }

      // Verificamos que Pacman haya chocado o no con algún fantasma
      if (pacman.choque(&fantasmas, modo_fantasma, &fantasma)) {  // Si pacman choca con algún fantasma
      
         cout << "\a"; Sleep(1000); // Se hace una pausita luego del choque
         
         // Veamos si es que pacman muere o se come al fantasma o simplemente lo ignora (cuando el fantasma está muerto)
         
         if (modo_fantasma == MODO_CAZADOR) {  // Si chocaron cuando los fantasmas estaban en modo cazador, pacman pierde
         
            fantasma = NULL;     // No nos sirve la dirección del fantasma chocado
            
            pacman.setVidas(pacman.getVidas() - 1);  // Disminuimos la vida de pacman
            
            if (pacman.getVidas() > 0) {  // Sigue jugando pero disminuye una vida
            
               // Regresamos a pacman y a los fantasmas a su posición inicial
               reinicia_personajes(pacman, fantasmas);
               
               // Pintamos al mapa
               pintar_mapa(mapa, FILAS_MAPA, COLUMNAS_MAPA);
               
               // Pintamos a los personajes
               pacman.pintar();
               for (int i = 0; i < fantasmas.size(); ++i) {
                  fantasmas[i].pintar();
               }
               
               // Imprimimos las vida de Pacman
               pacman.imprimirVidas();
               
               // Empieza la cuenta regresiva
               cuenta_regresiva(LIMITE_IZQUIERDO + 14, LIMITE_SUPERIOR + 13, 1, 3); // Volvemos a empezar

            } else { // Significa que ya murió porque tiene cero vidas
            
               gotoxy2(LIMITE_IZQUIERDO + 10, LIMITE_SUPERIOR + 10);
               cout << "Has perdido xD!!: [s] para salir";
               
               bool sal = false;
               do {
                  char tecla = getch();
                  if (tecla == 's') 
                     sal = true;
               } while (!sal);
               // while ((getch() == 's') ? 0 : 1);
               return;  // Salimos del juego
            
            } // fin de if
            
         } else if (modo_fantasma == MODO_PRESA) { // Cuando pacman choca a un fantasma en modo presa, pacman se lo come
            
            // Cambiar al fantasma a modo MODO_INVISIBLE
            // Entonces dicho fantasma debe regresar a su casa
            
            // Una vez llegado a su casa, debe de volver su estado a modo CAZADOR y empezar con sus coordenadas iniciales
            
            
            
            ///////////////////////////
            // Por ahora simplemente haré que cuando un fantasma sea comido, aparezca inmediatamente en su casa
            // Luego investigaré sobre como hacer para que el fantasma regrese a su casa
            /*********/
            
            
            
            fantasma->reiniciar_coordenadas();  // Regresamos al fantasma a su casa (posición inicial)
            string nombre_fantasma = fantasma->getNombre();
            fantasma->setModo(MODO_CAZADOR); // Regresamos al modo cazador
            
            /*********/
            
            
            // Falta implementar todo esto, cuando los fantasmas regresan a su casa
            // Usar el puntero "fantasma" con el que podemos modificar al fantasma chocado
            
         } else if (modo_fantasma == MODO_INVISIBLE) { // Cuando pacman choca a un fantasma estando en modo invisible, no pasa nada
            
         }
         
      } else {
         if (pacman.estadoFinalPacman() == PACMAN_GANADOR) {
            gotoxy2(LIMITE_IZQUIERDO + 10, LIMITE_SUPERIOR + 10);
            cout << "Has GANADO :D!!: [s] para salir";
            bool sal = false; 
            do {
               char tecla = getch();
               if (tecla == 's') 
                  sal = true;
            } while (!sal);
            return;
         }
      }
      
      
      Sleep(100); // Hace una pausa muy pequeña en milisegundos
      
      /*
      if (tecla == 's') {
         tecla = pacman.getDireccion();
         gotoxy2(LIMITE_IZQUIERDO + 10, LIMITE_SUPERIOR + 10);
         cout << "Seguro que desea salir del juego (y/n)?: ";
         tecla_salir = getche();
         
         if (tecla_salir == 'y') {
            return;
         } else {
            repintar = true;  // para pintar el mapa solo cuando se haya paralizado el juego
         }
      }
      */
      
   } while (tecla_salir != 'y');
}

