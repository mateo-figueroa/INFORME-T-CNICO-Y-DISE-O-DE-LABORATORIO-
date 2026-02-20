Código Fuente (kernel.c - Simulación de Gestión de Procesos): 
#include <stdio.h> 
#include <stdlib.h> 

// Estructura para un proceso 
struct Proceso { 
int id; 
int prioridad; 
char estado[10];  // "Listo", "Ejecutando", etc. 
}; 
// Nodo para lista enlazada de procesos 
struct NodoProceso { 
struct Proceso *proceso; 
struct NodoProceso *siguiente; 
}; 
// Cola de procesos (lista enlazada) 
struct NodoProceso *colaProcesos = NULL; 
 
// Función para crear un proceso (memoria dinámica) 
void crearProceso(int id, int prioridad) { 
    struct NodoProceso *nuevoNodo = (struct NodoProceso *)malloc(sizeof(struct 
NodoProceso)); 
    if (nuevoNodo == NULL) { 
        printf("Error: No se pudo asignar memoria.\n"); 
        return; 
    } 
    nuevoNodo->proceso = (struct Proceso *)malloc(sizeof(struct Proceso)); 
    if (nuevoNodo->proceso == NULL) { 
        printf("Error: No se pudo asignar memoria para proceso.\n"); 
        free(nuevoNodo); 
        return; 
    } 
    nuevoNodo->proceso->id = id; 
    nuevoNodo->proceso->prioridad = prioridad; 
    snprintf(nuevoNodo->proceso->estado, sizeof(nuevoNodo->proceso->estado), "Listo"); 
    nuevoNodo->siguiente = colaProcesos; 
    colaProcesos = nuevoNodo; 
    printf("Proceso %d creado con prioridad %d.\n", id, prioridad); 
} 
 
// Función para eliminar un proceso por ID 
void eliminarProceso(int id) { 
    struct NodoProceso *actual = colaProcesos; 
    struct NodoProceso *anterior = NULL; 
    while (actual != NULL) { 
        if (actual->proceso->id == id) { 
            if (anterior == NULL) { 
                colaProcesos = actual->siguiente; 
            } else { 
                anterior->siguiente = actual->siguiente; 
            } 
            free(actual->proceso); 
            free(actual); 
            printf("Proceso %d eliminado.\n", id); 
            return; 
        } 
        anterior = actual; 
        actual = actual->siguiente; 
    } 
    printf("Proceso %d no encontrado.\n", id); 
} 

// Función para listar procesos 
void listarProcesos() { 
    struct NodoProceso *actual = colaProcesos; 
    printf("Lista de Procesos:\n"); 
    while (actual != NULL) { 
        printf("ID: %d, Prioridad: %d, Estado: %s\n", actual->proceso->id, actual->proceso
>prioridad, actual->proceso->estado); 
        actual = actual->siguiente; 
    } 
}







Código Fuente (filesystem.c - Simulación de Sistemas de Archivos): 
#include <stdio.h> 
#include <stdlib.h> 
#include <string.h> 
 
// Estructura para un archivo simulado 
struct Archivo { 
    char nombre[50]; 
    int tamano;  // En bloques 
    char **bloques;  // Puntero a array de bloques (cadenas) 
    char **journal;  // Journaling simple para ext4 simulado 
}; 
 
// Función para crear un archivo (dinámico) 
struct Archivo* crearArchivo(char* nombre, int tamanoInicial) { 
    struct Archivo* nuevoArchivo = (struct Archivo*)malloc(sizeof(struct Archivo)); 
    if (nuevoArchivo == NULL) { 
        printf("Error: No se pudo asignar memoria para archivo.\n"); 
        return NULL; 
    } 
    strcpy(nuevoArchivo->nombre, nombre); 
    nuevoArchivo->tamano = tamanoInicial; 
    nuevoArchivo->bloques = (char**)malloc(tamanoInicial * sizeof(char*)); 
    nuevoArchivo->journal = (char**)malloc(tamanoInicial * sizeof(char*));  // Journaling 
    if (nuevoArchivo->bloques == NULL || nuevoArchivo->journal == NULL) { 
        printf("Error: No se pudo asignar memoria para bloques/journal.\n"); 
        free(nuevoArchivo); 
        return NULL; 
    } 
    for (int i = 0; i < tamanoInicial; i++) { 
        nuevoArchivo->bloques[i] = NULL; 
        nuevoArchivo->journal[i] = NULL; 
    } 
    printf("Archivo '%s' creado con %d bloques iniciales.\n", nombre, tamanoInicial); 
    return nuevoArchivo; 
} 
 
// Función para escribir en un bloque (con realloc para expansión) 
void escribirBloque(struct Archivo* archivo, int indice, char* datos) { 
    if (indice >= archivo->tamano) { 
        // Expansión dinámica (simulando crecimiento de archivo) 
        archivo->bloques = (char**)realloc(archivo->bloques, (indice + 1) * sizeof(char*)); 
        archivo->journal = (char**)realloc(archivo->journal, (indice + 1) * sizeof(char*)); 
        if (archivo->bloques == NULL || archivo->journal == NULL) { 
            printf("Error: No se pudo expandir bloques/journal.\n"); 
            return; 
        } 
        for (int i = archivo->tamano; i <= indice; i++) { 
            archivo->bloques[i] = NULL; 
            archivo->journal[i] = NULL; 
        } 
        archivo->tamano = indice + 1; 
    } 
    // Journaling: Guardar estado anterior 
    if (archivo->bloques[indice] != NULL) { 
        archivo->journal[indice] = strdup(archivo->bloques[indice]);  // Copia para journaling 
    } 
    free(archivo->bloques[indice]);  // Liberar anterior 
    archivo->bloques[indice] = strdup(datos); 
    printf("Escrito en bloque %d de '%s': %s\n", indice, archivo->nombre, datos); 
} 
 
// Función para leer un bloque 
void leerBloque(struct Archivo* archivo, int indice) { 
    if (indice < archivo->tamano && archivo->bloques[indice] != NULL) { 
        printf("Bloque %d de '%s': %s\n", indice, archivo->nombre, archivo->bloques[indice]); 
    } else { 
        printf("Bloque %d no existe o está vacío.\n", indice); 
    } 
} 
 
// Liberar memoria del archivo 
void liberarArchivo(struct Archivo* archivo) { 
    for (int i = 0; i < archivo->tamano; i++) { 
        free(archivo->bloques[i]); 
        free(archivo->journal[i]); 
    } 
    free(archivo->bloques); 
    free(archivo->journal); 
    free(archivo); 
} 
 
Código Fuente (main.c - Integración): 
#include <stdio.h> 
// Incluir headers de kernel.c y filesystem.c (asumir que están en el mismo directorio) 
 
extern void crearProceso(int id, int prioridad); 
extern void eliminarProceso(int id); 
extern void listarProcesos(); 
extern struct Archivo* crearArchivo(char* nombre, int tamanoInicial); 
extern void escribirBloque(struct Archivo* archivo, int indice, char* datos); 
extern void leerBloque(struct Archivo* archivo, int indice); 
extern void liberarArchivo(struct Archivo* archivo); 
 
int main() { 
    // Simulación Kernel (basado en monolítico/híbrido del Grupo 4) 
    crearProceso(1, 5);  // Proceso Linux-like 
    crearProceso(2, 3);  // Proceso Windows-like 
    listarProcesos(); 
    eliminarProceso(1); 
    listarProcesos(); 
 
    // Simulación File System (ext4/NTFS) 
struct Archivo* miArchivo = crearArchivo("datos.ext4", 2);  // Simulando ext4 
escribirBloque(miArchivo, 0, "Datos de prueba en bloque 0"); 
escribirBloque(miArchivo, 2, "Expansión dinámica en bloque 2");  // Realloc 
leerBloque(miArchivo, 0); 
leerBloque(miArchivo, 2); 
liberarArchivo(miArchivo); 
return 0; 
} 


