## Clase 3

### Semaforos

El semaforo siempre se inicializa en la declaración:

```python
sem mutex = 1
sem fork[5] = ([5]1)
```

#### Semaforo general

P(s) : <await(s>0) s = s-1>
V(s) : <s = s+1>

Ejemplo de exclusion mutua con semaforo: Utilizamos en vez de un booleano un int llamado free que si vale 1 es free si vale 0 esta ocupado:

```python
int free = 1

process SC[i=1 to n]{
    while(true){
        <await (free > 0) free = free -1>
        // Seccion critica
        <free = free + 1>
        // Seccion no critica 
    }
}
```

El funcionamiento es igual que con un booleano, espero a que free sea mayor a 0, una vez detecto que es mayor lo pongo en 0, es decir le resto uno, realizo mi seccion critica y al terminar le sumo uno para dejarlo libre.


En la ppt estaba escrito de la siguiente manera:


```python
int free = 1

process SC[i=1 to n]{
    while(true){
        P(free)
        // Seccion critica
        V(free)
        // Seccion no critica 
    }
}
```
Recordemos que el semaforo general:
P(s) : <await(s>0) s = s-1>
V(s) : <s = s+1>


#### Barreras

Con dos procesos:
Debo tener dos variables, una para indicar la llegada de cada proceso.


```python
sem llega1 = 0, llega2 = 0;
```

```python
process worker1{
    V(llega1)
    P(llega2)
}
```

```python
process worker2{
    V(llega2)
    P(llega1)
}
```

El proceso 1 avisa que llego y espera que llegue el 2, el 2 hace lo mismo pero al contrario.


#### Contadores de recursos
Cada semaforo cuenta el numero de unidades libres de un recurso determinado. Esta forma de utilizacion es adecuada cuando los procesos compiten por los recursos de multiples unidades.


Ejemplo

```python
typeT buff[n]; int ocupado = 0; libre = 0;
sem vacio = n; lleno = 0;
```

```python
process productor{
    while(true){
        producir mensaje datos
        P(vacio) // Resto uno a los lugares vacios
        buff[libre] = datos; // En el lugar libre coloco el dato
        libre = (libre +1) mod n
        V(lleno) // Aumento la cantidad de llenos
    }
}
```

```python
process consumidor{
    while (true){
        P(lleno) // le saco del contador a llenos
        resultado = buf[ocupado]
        ocupado = (ocupado +1) mod n
        V(vacio) // le saco uno a vacio
    }
}
```

• vacio cuenta los lugares libres, y lleno los ocupados.
• depositar y retirar se pudieron asumir atómicas pues sólo hay un productor y un consumidor.


#### Con más procesos

```python
 typeT buf[n];  int ocupado = 0, libre = 0;
 sem  vacio = n, lleno = 0;
 sem mutexD = 1, mutexR = 1;
```
```python
 process Productor [i = 1..M]{
    while(true){ 
        producir mensaje datos
        P(vacio); 
        P(mutexD);
        buf[libre] = datos;
        libre = (libre+1) mod n;
        V(mutexD); 
        V(lleno); 
      }
 }
```

```python 
process Consumidor [i = 1..N]{ 
    while(true){ 
        P(lleno); 
        P(mutexR);
        resultado = buf[ocupado];
        ocupado = (ocupado+1) mod n; V(mutexR);
        V(vacio);
        consumir mensaje resultado
    }
 }
```