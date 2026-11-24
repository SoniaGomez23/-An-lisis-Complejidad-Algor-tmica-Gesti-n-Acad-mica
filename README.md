#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <chrono>

using namespace std;
using namespace std::chrono;

// Estructura para representar cada fila del CSV
struct RegistroNota {
    string claveUnica; // id_estudiante + codigo_asignatura
    string datosCompletos;
};

// =========================================================================
// ESTRUCTURA 1: LISTA ENLAZADA SIMPLE (Estilo del profesor)
// =========================================================================
struct NodoLista {
    RegistroNota dato;
    NodoLista* siguiente = nullptr;
};

struct ListaEnlazada {
    NodoLista* cabeza = nullptr;

    // Inserción al inicio: O(1)
    void insertar(RegistroNota reg) {
        NodoLista* nuevo = new NodoLista();
        nuevo->dato = reg;
        nuevo->siguiente = cabeza;
        cabeza = nuevo;
    }

    // Búsqueda secuencial: O(n)
    bool buscar(string clave) {
        for (NodoLista* actual = cabeza; actual != nullptr; actual = actual->siguiente) {
            if (actual->dato.claveUnica == clave) return true;
        }
        return false;
    }

    // Recorrido completo: O(n)
    int recorrer() {
        int contador = 0;
        for (NodoLista* actual = cabeza; actual != nullptr; actual = actual->siguiente) {
            contador++;
        }
        return contador;
    }

    void liberar() {
        while (cabeza != nullptr) {
            NodoLista* temp = cabeza;
            cabeza = cabeza->siguiente;
            delete temp;
        }
    }
};

// =========================================================================
// ESTRUCTURA 2: COLA / LISTA DOBLE CON PUNTERO AL FINAL (Inserción al final O(1))
// =========================================================================
struct Cola {
    NodoLista* frente = nullptr;
    NodoLista* finalCola = nullptr;

    // Inserción al final (Enqueue): O(1)
    void insertar(RegistroNota reg) {
        NodoLista* nuevo = new NodoLista();
        nuevo->dato = reg;
        if (finalCola == nullptr) {
            frente = finalCola = nuevo;
        } else {
            finalCola->siguiente = nuevo;
            finalCola = nuevo;
        }
    }

    // Búsqueda secuencial: O(n)
    bool buscar(string clave) {
        for (NodoLista* actual = frente; actual != nullptr; actual = actual->siguiente) {
            if (actual->dato.claveUnica == clave) return true;
        }
        return false;
    }

    // Recorrido completo: O(n)
    int recorrer() {
        int contador = 0;
        for (NodoLista* actual = frente; actual != nullptr; actual = actual->siguiente) {
            contador++;
        }
        return contador;
    }

    void liberar() {
        while (frente != nullptr) {
            NodoLista* temp = frente;
            frente = frente->siguiente;
            delete temp;
        }
        finalCola = nullptr;
    }
};

// =========================================================================
// ESTRUCTURA 3: ARREGLO DINÁMICO MANUAL (Puntero y redimensionamiento)
// =========================================================================
struct ArregloDinamico {
    RegistroNota* datos = nullptr;
    int capacidad = 10;
    int tamano = 0;

    ArregloDinamico() {
        datos = new RegistroNota[capacidad];
    }

    // Inserción al final: O(1) amortizado
    void insertar(RegistroNota reg) {
        if (tamano == capacidad) {
            capacidad *= 2;
            RegistroNota* nuevoArreglo = new RegistroNota[capacidad];
            for (int i = 0; i < tamano; i++) {
                nuevoArreglo[i] = datos[i];
            }
            delete[] datos;
            datos = nuevoArreglo;
        }
        datos[tamano++] = reg;
    }

    // Búsqueda secuencial: O(n)
    bool buscar(string clave) {
        for (int i = 0; i < tamano; i++) {
            if (datos[i].claveUnica == clave) return true;
        }
        return false;
    }

    // Recorrido completo: O(n)
    int recorrer() {
        int contador = 0;
        for (int i = 0; i < tamano; i++) {
            contador++;
        }
        return contador;
    }

    void liberar() {
        delete[] datos;
        datos = nullptr;
        tamano = 0;
    }
};

// Función para cargar el CSV (detecta ',' o ';')
int cargarCSV(string ruta, RegistroNota* buffer, int maxRegistros) {
    ifstream archivo(ruta);
    if (!archivo.is_open()) {
        cout << "Error: No se pudo abrir el archivo CSV.\n";
        return 0;
    }

    string linea;
    getline(archivo, linea); // Saltar encabezado
    char delim = (linea.find(';') != string::npos) ? ';' : ',';

    int total = 0;
    while (getline(archivo, linea) && total < maxRegistros) {
        if (linea.empty()) continue;
        stringstream ss(linea);
        string col1, col2;
        getline(ss, col1, delim);
        getline(ss, col2, delim);
        
        buffer[total].claveUnica = col1 + "_" + col2;
        buffer[total].datosCompletos = linea;
        total++;
    }
    archivo.close();
    return total;
}

int main() {
    const int MAX_DATOS = 3000;
    RegistroNota* todosLosDatos = new RegistroNota[MAX_DATOS];

    // Ruta de tu archivo CSV
    string rutaCSV = "D:\\Tercer semestre 2026-2\\ESTRUCTURA DE DATOS\\notas.csv\\notas_estudiantes_ingenieria.csv";
    int totalCargados = cargarCSV(rutaCSV, todosLosDatos, MAX_DATOS);

    if (totalCargados == 0) return 1;
    cout << "Registros cargados exitosamente: " << totalCargados << "\n";

    // Archivo de salida para graficar en Excel
    ofstream salida("D:\\Tercer semestre 2026-2\\ESTRUCTURA DE DATOS\\notas.csv\\resultados_tiempos.csv");
    salida << "N;Ins_Lista;Ins_Cola;Ins_Arreglo;Bus_Lista;Bus_Cola;Bus_Arreglo;Rec_Lista;Rec_Cola;Rec_Arreglo\n";

    // Tamaños solicitados: 10, 50, 100, 200 ... 3000
       int tamanos[32] = {10, 50};
    for (int i = 2; i < 32; i++) {
        tamanos[i] = (i - 1) * 100;
    }
    for (int i = 2; i < 32; i++) {
        tamanos[i] = (i - 1) * 100;
    }

    for (int t = 0; t < 32; t++) {
        int n = tamanos[t];
        if (n > totalCargados) break;

        // Buscamos un elemento del medio/final para medir tiempo real
        string claveObjetivo = todosLosDatos[n / 2].claveUnica;

        // 1. Medir Lista Enlazada
        ListaEnlazada lista;
        auto inicio = high_resolution_clock::now();
        for (int i = 0; i < n; i++) lista.insertar(todosLosDatos[i]);
        double tInsLista = duration<double, micro>(high_resolution_clock::now() - inicio).count();

        inicio = high_resolution_clock::now();
        lista.buscar(claveObjetivo);
        double tBusLista = duration<double, micro>(high_resolution_clock::now() - inicio).count();

        inicio = high_resolution_clock::now();
        lista.recorrer();
        double tRecLista = duration<double, micro>(high_resolution_clock::now() - inicio).count();
        lista.liberar();

        // 2. Medir Cola
        Cola cola;
        inicio = high_resolution_clock::now();
        for (int i = 0; i < n; i++) cola.insertar(todosLosDatos[i]);
        double tInsCola = duration<double, micro>(high_resolution_clock::now() - inicio).count();

        inicio = high_resolution_clock::now();
        cola.buscar(claveObjetivo);
        double tBusCola = duration<double, micro>(high_resolution_clock::now() - inicio).count();

        inicio = high_resolution_clock::now();
        cola.recorrer();
        double tRecCola = duration<double, micro>(high_resolution_clock::now() - inicio).count();
        cola.liberar();

        // 3. Medir Arreglo Dinámico
        ArregloDinamico arreglo;
        inicio = high_resolution_clock::now();
        for (int i = 0; i < n; i++) arreglo.insertar(todosLosDatos[i]);
        double tInsArreglo = duration<double, micro>(high_resolution_clock::now() - inicio).count();

        inicio = high_resolution_clock::now();
        arreglo.buscar(claveObjetivo);
        double tBusArreglo = duration<double, micro>(high_resolution_clock::now() - inicio).count();

        inicio = high_resolution_clock::now();
        arreglo.recorrer();
        double tRecArreglo = duration<double, micro>(high_resolution_clock::now() - inicio).count();
        arreglo.liberar();

        salida << n << ";" << tInsLista << ";" << tInsCola << ";" << tInsArreglo << ";"
               << tBusLista << ";" << tBusCola << ";" << tBusArreglo << ";"
               << tRecLista << ";" << tRecCola << ";" << tRecArreglo << "\n";
    }

    salida.close();
    delete[] todosLosDatos;
    cout << "Medicion completada. Archivo 'resultados_tiempos.csv' generado con exito.\n";
    return 0;
}
