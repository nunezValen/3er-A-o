## Práctica 2 – Semáforos

### CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS: 
• Los semáforos deben estar declarados en todos los ejercicios. 
• Los semáforos deben estar inicializados en todos los ejercicios. 
• No se puede utilizar ninguna sentencia para setear o ver el valor de un semáforo. 
• Debe evitarse hacer busy waiting en todos los ejercicios. 
• En todos los ejercicios el tiempo debe representarse con la función delay.


#### 1. Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión.  

a. Analice el problema y defina qué procesos, recursos y semáforos/sincronizaciones serán necesarios/convenientes para resolverlo. 

* Respuesta : 
Necesitaría, un semaforo que me indique si el detector de metales esta ocupado y el proceso persona que esperará a la señal del semaforo para poder acceder al detector de metales. 

b. Implemente una solución que modele el acceso de las personas a un detector (es decir,  si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar).  

```python
sem mutex = 1;
process persona [i:1..N]{
    P(mutex)
    ## usar detector
    V(mutex)
}
```


c. Modifique su solución para el caso que haya tres detectores.

```python
sem mutex = 3; # Lo unico que cambia

process persona[i:1..N]{
    P(mutex)
    # Usar detector
    V(mutex)
}
```

d. Modifique la solución anterior para el caso en que cada persona pueda pasar más de 
una vez, siendo aleatoria esa cantidad de veces.  

```python
sem mutex = 3;

process persona [i: 1..N]{
    int x = random.int();
    while(x>0){
        P(mutex)
        # Usar detector
        V(mutex)
        x--
    }
}
```


#### 2. Un sistema de control cuenta con 4 procesos que realizan chequeos en forma  colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de  tamaño N). De cada fallo, se conoce su número de identificación (ID) y su nivel de  gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Resuelva considerando las siguientes  situaciones: 


a) Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el 
orden). 

```python
sem mutex = 1;
int cantidad = 0;
colaFallos cola[N];

process controlador [i=1..4]{
    Fallo fallo;
    P(mutex)
    while (not cola.isEmpty()){
        fallo = cola.pop;
        V(mutex)
        if(fallo.getNivel() == 3){
            print(fallo.getID())
        }
        P(mutex)
    }
    V(mutex)
    
}
```
// Funciona de la siguiente manera: 
Llega un proceso, bloquea el semaforo para consultar si puede acceder al while, si accede al while hace el pop del elemento y luego del pop libera el semaforo, luego lo vuelve a bloquear al final de la iteracion para consultar si debe acceder nuevamente y de acceder hace pop y libera. Terminado las iteraciones(la cola esta vacia) libera el semaforo.

b) Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los 
resultados en un vector global. 


```python
sem mutex = 1;
sem semaforoNivel[4] = ([4] 1); 
array cantidadFallos[4] = ([4] 0);
colaFallos cola[N];


process controlador [i=1..4]{
    Fallo fallo;
    P(mutex)
    while (not cola.isEmpty()){
        fallo = cola.pop;
        V(mutex)
        if(fallo.getNivel() == 3){
            print(fallo.getID())
        }
        P(semaforoNivel[fallo.getNivel()]) 
        cantidadFallos[fallo.getNivel()] ++ ;
        V(semaforoNivel[fallo.getNivel()])
        P(mutex)
    }
    V(mutex)
}
```





c) Ídem b) pero cada proceso debe ocuparse de contar los fallos de un nivel de 
gravedad determinado.

```python
sem mutex = 1;
sem semaforoNivel[4] = ([4] 1); 
array cantidadFallos[4] = ([4] 0);
colaFallos cola[N];


process controlador [i=1..4]{
    Fallo fallo;
    P(mutex)
    while (not cola.isEmpty()){
        fallo = cola.pop;
        V(mutex)

        if (fallo.getNivel() == id){
            if(nivel == 3){
                print(fallo.getID())
            }
            cantidadFallos[id]++
        }
        else {
            P(mutex)
            cola.push(fallo)
            V(mutex)
        }
        P(mutex)
    }
    V(mutex)
}
```