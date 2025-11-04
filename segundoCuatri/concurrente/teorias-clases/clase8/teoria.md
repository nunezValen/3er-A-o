# Clase 8 (creo)

## Pthreads (Api de C para crear procesos)

```C
llegue tarde
```

### Pthreads soporta 3 tipos de mutexs, normal|recursive|Error check

- Normal: Fucniona como el de toda la vida. Puede haber dead lock. NO permite que un thread que lo tienen bloqueado vuelva a hacer un lock sobre él.
- Recursivo: Permite que un thread que lo tienen bloqueado vuelva a hacer un lock sobre él(Reentrante). Luego de hacer lock puede volver a lockearlo. Ya que puede pasar que haga un lock, llame a una funcion y quiera re hacer un lock sobre la misma variable. 
- Error Check: Responde con un error al un thread intentar lockear el mismo elemento.

### Variables condición: 
```C
int pthread_cond_wait(pthread_cond_t *cond,pthread_mutex_t *mutex )
```
Es un wait. Todas las variables condicion estan asociadas a un mutex. En la ppt hay muchos ejemplos.

Codigo de productor consumidor: 


```C
pthread_cond_t vacio, lleno;
pthread_mutex_t mutex;
int hayElemento;
tipo_elemento Buffer;
...
main()
 { ...
hayElemento= 0;
pthread_init();
pthread_cond_init(&vacio, NULL);
pthread_cond_init(&lleno, NULL);
pthread_mutex_init(&mutex, NULL);
...
 }
```

```C
void *productor(void *datos){
    tipo_element elem;

    while (true){
        generar_elemento(elem);
        pthread_mutex_lock(&mutex);
        while (hayElemento == 1){
            pthread_cond_wait(&vacio, &mutex);
        }
        Buffer = elem;
        hayElemento = 1;
        pthread_cond_signal(&lleno);
        pthread_mutex_unlock(&mutex);
    }
}
```

```C
void *consumidor(void *datos)
 { tipo_element elem;

 while (true))
 { pthread_mutex_lock (&mutex);
 while (hayElemento = = 0)
 pthread_cond_wait (&lleno, &mutex);
 elem= Buffer;
 hayElemento = 0;
 pthread_cond_signal (&vacio);
 pthread_mutex_unlock (&mutex);
 procesar_elemento(elem);
 }
 } 

```

### Semaforos 
Es una biblioteca adicional

sem_t semaforo→ se declaran globales a los threads.
- sem_init (&semaforo, alcance, inicial) → en esta operación se inicializa el
semáforo semaforo. Inicial es el valor con que se inicializa el semáforo.
Alcance indica si es compartido por los hilos de un único proceso (0) o por
los de todos los procesos ( ≠ 0).
- sem_wait(&semaforo) → equivale al P.
- sem_post(&semaforo) → equivale al V.

### Monitores
- Se simula con un mutex la exclusión mutua, ya que no posee monitores. Hay que hacer una funcion/clase que represente al monitor y lockee y libere al inicio y final.

