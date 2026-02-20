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
