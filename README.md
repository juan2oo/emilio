#include <stdio.h>
#include <stdlib.h>
#include <time.h>

// Definición de estructura para las cartas
struct Carta {
    int numero; // Valor de la carta (1 a 12)
    int palo;   // Palo de la carta (0: Oro, 1: Copas, 2: Espadas, 3: Bastos)
    int jerarquia; // Jerarquía para el Truco (1 a 13)
};

// Declaraciones de funciones
void inicializarMazo(struct Carta mazo[]);
void barajar(struct Carta mazo[]);
void repartir(struct Carta mazo[], struct Carta mano[], int inicio);
void mostrarCarta(struct Carta carta);
int obtenerGanadorMano(struct Carta carta1, struct Carta carta2);
int obtenerJerarquia(int numero, int palo);
int decidirCarta(int cartas[]);
void mostrarMenu(int vector[]);
int pedirTruco(); // Nueva función para pedir Truco

// Mazo y palos
const char* palos[] = {"Oro", "Copas", "Espadas", "Bastos"};
int puntajeJugador = 0, puntajeComputadora = 0;

// Función principal
int main() {
    struct Carta mazo[40];
    struct Carta manoJugador[3], manoComputadora[3];
    int ronda;

    srand(time(NULL));
    inicializarMazo(mazo);
    barajar(mazo);

    // Juego de 3 rondas (una partida de Truco básica)
    for (ronda = 0; ronda < 3 && puntajeJugador < 30 && puntajeComputadora < 30; ronda++) {
        printf("\n--- Ronda %d ---\n", ronda + 1);
        int cartas[3] = {1, 2, 3};
        repartir(mazo, manoJugador, 0);      // Repartir al jugador
        repartir(mazo, manoComputadora, 3);  // Repartir a la computadora

        printf("Tus cartas:\n");
        for (int i = 0; i < 3; i++) {
            printf("%d. ", i + 1);
            mostrarCarta(manoJugador[i]);
        }

        // Opción de pedir Truco antes de iniciar la ronda
        int resultadoTruco = pedirTruco();
        if (resultadoTruco == -1) {
            puntajeJugador += 2; // Ganaste automáticamente
            continue;
        } else if (resultadoTruco == 2) {
            printf("Esta ronda valdrá más puntos por el Truco!\n");
        }

        int manosGanadasJugador = 0, manosGanadasComputadora = 0;

        for (int mano = 0; mano < 3; mano++) {
            printf("\n--- Mano %d ---\n", mano + 1);

            int eleccionJugador;
            eleccionJugador = decidirCarta(cartas);

            struct Carta cartaJugador = manoJugador[eleccionJugador];
            struct Carta cartaComputadora = manoComputadora[mano];

            printf("Tu carta: ");
            mostrarCarta(cartaJugador);
            printf("Carta de la computadora: ");
            mostrarCarta(cartaComputadora);

            int resultado = obtenerGanadorMano(cartaJugador, cartaComputadora);
            if (resultado == 1) {
                printf("Ganaste esta mano!\n");
                manosGanadasJugador++;
            } else if (resultado == -1) {
                printf("La computadora gana esta mano!\n");
                manosGanadasComputadora++;
            } else {
                printf("Empate en esta mano.\n");
            }
            cartas[eleccionJugador] = 0;

            if (manosGanadasJugador == 2 || manosGanadasComputadora == 2) break;
        }

        if (manosGanadasJugador > manosGanadasComputadora) {
            printf("\nGanaste la ronda %d!\n", ronda + 1);
            puntajeJugador += (resultadoTruco == 2) ? 4 : 2; // Doble puntaje si hay Truco
        } else {
            printf("\nLa computadora gana la ronda %d!\n", ronda + 1);
            puntajeComputadora += (resultadoTruco == 2) ? 4 : 2; // Doble puntaje si hay Truco
        }

        printf("Puntaje: Jugador %d - Computadora %d\n", puntajeJugador, puntajeComputadora);
    }

    // Resultados finales
    if (puntajeJugador >= 30)
        printf("\nFelicitaciones, ganaste el Truco!\n");
    else
        printf("\nLa computadora ganó el Truco. Más suerte la próxima vez!\n");

    return 0;
}

// Inicializa el mazo de cartas con valores y jerarquías
void inicializarMazo(struct Carta mazo[]) {
    int index = 0;
    for (int palo = 0; palo < 4; palo++) {
        for (int numero = 1; numero <= 12; numero++) {
            if (numero != 8 && numero != 9) { // Omitimos 8 y 9 para el mazo español
                mazo[index].numero = numero;
                mazo[index].palo = palo;
                mazo[index].jerarquia = obtenerJerarquia(numero, palo);
                index++;
            }
        }
    }
}

// Baraja el mazo de cartas
void barajar(struct Carta mazo[]) {
    for (int i = 0; i < 40; i++) {
        int j = rand() % 40;
        struct Carta temp = mazo[i];
        mazo[i] = mazo[j];
        mazo[j] = temp;
    }
}

// Reparte 3 cartas desde un índice dado
void repartir(struct Carta mazo[], struct Carta mano[], int inicio) {
    for (int i = 0; i < 3; i++) {
        mano[i] = mazo[inicio + i];
    }
}

// Muestra una carta
void mostrarCarta(struct Carta carta) {
    printf("%d de %s\n", carta.numero, palos[carta.palo]);
}

// Determina el ganador de una mano (1: jugador, -1: computadora, 0: empate)
int obtenerGanadorMano(struct Carta carta1, struct Carta carta2) {
    if (carta1.jerarquia > carta2.jerarquia) return 1;
    if (carta1.jerarquia < carta2.jerarquia) return -1;
    return 0;
}

// Define la jerarquía de las cartas en el Truco
int obtenerJerarquia(int numero, int palo) {
    if (numero == 1 && palo == 2) return 13; // 1 de Espadas
    if (numero == 1 && palo == 3) return 12; // 1 de Bastos
    if (numero == 7 && palo == 2) return 11; // 7 de Espadas
    if (numero == 7 && palo == 0) return 10; // 7 de Oro
    if (numero == 3) return 9;
    if (numero == 2) return 8;
    if (numero == 1) return 7;
    if (numero == 12) return 6;
    if (numero == 11) return 5;
    if (numero == 10) return 4;
    if (numero == 7) return 3;
    if (numero == 6) return 2;
    if (numero == 5) return 1;
    if (numero == 4) return 0;
    return -1;
}

int decidirCarta(int cartas[]) {
    int cant, i, valido = 0;
    while (!valido) {
        printf("Elige una carta: ");
        scanf("%d", &cant);
        if (cant >= 1 && cant <= 3 && cartas[cant - 1] != 0) {
            valido = 1;
        } else {
            printf("Carta inválida. Intenta nuevamente.\n");
        }
    }
    return cant - 1;
}

void mostrarMenu(int vector[]) {
    printf("Cartas disponibles: ");
    for (int i = 0; i < 3; i++) {
        if (vector[i] != 0) {
            printf("%d ", vector[i]);
        }
    }
    printf("\n");
}
int pedirTruco() {
    int respuesta;
    printf("Deseas pedir Truco? 1: si, 2: No ");
    scanf("%d", &respuesta);

    if (respuesta == 1) {
        // La computadora responde aleatoriamente
        int respuestaComputadora = rand() % 2;
        if (respuestaComputadora == 1) {
            printf("La computadora aceptó el Truco ");
            return 2; // Puntaje adicional por Truco
        } else {
            printf("La computadora rechazó el Truco. Ganaste  esta ronda\n");
            return -1; // Computadora pierde automáticamente
        }
    }

    printf("No pediste Truco.La partida sigue normalmente.\n");
    return 0; // No se pidió Truco, todo normal
}
