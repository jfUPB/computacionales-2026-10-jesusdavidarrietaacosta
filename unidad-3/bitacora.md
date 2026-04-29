Nota del profesor: sin evidencia en la fase de aplicación

# Unidad 3
## Bitácora de aplicación 
<img width="1675" height="1000" alt="Captura de pantalla 2026-04-15 090536" src="https://github.com/user-attachments/assets/398f05cf-73a8-4f4e-ade8-31e66ccb2175" />

Para el primer problema podemos apreciar aqui, al momento de crear algo con un new y no borrarlo con un delete este genera un gasto memoria inecesaria ya que las variables creadas por new se guardan en el heap asi sea que se hayan terminado de ejecutar las funciones que la crearon.

<img width="1677" height="937" alt="Captura de pantalla 2026-04-15 101308" src="https://github.com/user-attachments/assets/b5860ca4-b167-4d00-b5c4-cdb3676a4f28" />
Para el segundo error podemos ver como esta el problema de como la copia del hero apunta al mismo puntero que apunta el hero, esto es un error ya que al cambiar las estadisticas del hero tambien se cambian las de la copiahero y eso no es lo que se pretende en este caso.

Para poder ver el error de la memoria se modifico el codigode la siguiente manera:

```
#include <iostream>
#include <string>

class Personaje {
public:
    std::string nombre;
    int* estadisticas;

    Personaje(std::string n, int vida, int ataque, int defensa) {
        nombre = n;
        estadisticas = new int[3]; // ← memoria en heap (NUNCA se libera)
        estadisticas[0] = vida;
        estadisticas[1] = ataque;
        estadisticas[2] = defensa;
    }

    void imprimir() {
        std::cout << "Personaje " << nombre
            << " [Vida: " << estadisticas[0]
            << ", ATK: " << estadisticas[1]
            << ", DEF: " << estadisticas[2]
            << "]" << std::endl;
    }
};

void simularEncuentro() {
    Personaje heroe("Aragorn", 100, 20, 15);

    Personaje copiaHeroe = heroe; // copia superficial
    copiaHeroe.nombre = "Copia de Aragorn";
}

int main() {
    std::cout << "Iniciando prueba de fuga de memoria..." << std::endl;

    for (int i = 0; i < 10000000; i++) {
        simularEncuentro();

        // Para ver progreso sin saturar consola
        if (i % 1000000 == 0) {
            std::cout << "Iteracion: " << i << std::endl;
        }
    }

    std::cout << "Fin del programa" << std::endl;
    return 0;
}

```
Se agrego un ciclo for para repetir varias veces el proceso para poder apreciar como poco a poco la memoria aumenta y no se libera
<img width="1675" height="1003" alt="image" src="https://github.com/user-attachments/assets/9c544756-378f-4907-b8c8-2b8cce437557" />


Codigo corregido
```
#include <iostream>
#include <string>
#include <vector> // Añadimos la librería vector

class Personaje {
public:
    std::string nombre;
    // 1. Reemplazamos el puntero crudo (int*) por un std::vector<int>
    std::vector<int> estadisticas; 

    Personaje(std::string n, int vida, int ataque, int defensa) {
        nombre = n;
        // 2. Inicializamos el vector directamente, eliminando el uso de 'new'
        estadisticas = {vida, ataque, defensa}; 
        std::cout << "Constructor: nace " << nombre << std::endl;
    }

    void imprimir() {
        std::cout << "Personaje " << nombre
            << " [Vida: " << estadisticas[0]
            << ", ATK: " << estadisticas[1]
            << ", DEF: " << estadisticas[2]
            << "]" << std::endl;
    }
};

void simularEncuentro() {
    std::cout << "\n--- Iniciando encuentro ---" << std::endl;
    Personaje heroe("Aragorn", 100, 20, 15);
    
    // Esta copia ahora es completamente segura
    Personaje copiaHeroe = heroe;
    copiaHeroe.nombre = "Copia de Aragorn";
    
    std::cout << "Saliendo del encuentro..." << std::endl;
}

int main() {
    simularEncuentro();
    std::cout << "\nSimulación terminada." << std::endl;
    return 0;
}

```

los cambios se solucionan gracias a que se estaba usando new para pedir memoria al sistema, pero nunca se agrego un delete[] para devolverla. Al terminar la función simularEncuentro(), los objetos se destruían, pero la memoria que guardaba los enteros quedaba ocupada permanentemente.
El otro problema era que al hacer Personaje copiaHeroe = heroe;, el compilador copiaba el puntero. Como resultado, ambos personajes apuntaban exactamente a la misma dirección de memoria. esto hacia que cuando se cambiaba las estadisticas de la copia o el hero tambien se cambiaba el otro


Este codigo soluciona el error ya que: 
Resuelve la fuga de memoria: El contenedor std::vector tiene su propio destructor interno permitiendo que la función simularEncuentro() terminara y el objeto Personaje se destruyera, su miembro estadisticas también se destruye, liberando la memoria automáticamente.
Resuelve la copia superficial: std::vector sabe cómo copiarse a sí mismo correctamente (copia profunda o deep copy). Cuando el compilador evalúa Personaje copiaHeroe = heroe;, ve que el miembro es un vector e invoca la lógica de copia del vector. Esto crea un nuevo espacio en memoria exclusivo para la copia con los mismos valores, separando los destinos de ambos personajes sin que tengas que programar un constructor de copia complejo (evitando la Regla de los Tres).
