## Practica 1

### 1. Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas: 
#### a) En algún caso el valor de x al terminar el programa es 56.
#### b) En algún caso el valor de x al terminar el programa es 22.
#### c) En algún caso el valor de x al terminar el programa es 23.

### P1:
```python
if (x = 0){
   1.1 y:= 4*2; 
    1.2 x:= y + 2; 
}
```

### P2:
```python
If (x > 0){
    2.1 x:= x + 1
} 
```

### P3:
```python 3.1     3.2    3.3
x:= (x*3) + (x*2) + 1;
```

#### Respuesta: 

##### Caso 1:
p1,p2,p3: X=56
a = verdadero

##### Caso 2:

para que de 22 hay que hacer
3.1, 1.1,1.2,3.2,3.3, 2.1 =  x=22

calculo: 
3*0, y=8, x=10, x=21, x=22

entonces b = verdadero

##### Caso 3:
C : verdadero

3.1, 1.1, 1.2, 2.1, 3.2, 3.3 = x=23




### 2. Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.


Idea: La idea principal seria subdividir el arreglo por la cantidad de procesos y darle una parte del arreglo a cada proceso para que realice su busqueda, por cada elemento que encuentre igual a N lo sume a una variable compartida, por lo cual necesitaremos atomicidad.

Pre condiciones: El arreglo sera divisible por la cantidad de procesos. 


##### Solucion (Se la mostré al ayudante y le gustó más está y dijo que estaba perfect :D )

```python
int total = 0;
int vector[0..M];
boolean libre = true; 
int N = 33;

chunks = M div J


process buscador[id:0..J]{

    inicio = chunks * id - chunks;
    fin = chunks * id - 1;


    int total_proc = 0;
    for i : inicio...fin{
        if (vector[i] == N){
            total_proc := total_proc+1
        }
    }
    <await libre; libre = false>
    //Seccion critica
    total = total + total_proc
    libre = true
}
```


##### Otra solucion (Tambien esta bien pero la otra es mejor)
```python
int total = 0;
int vector[0..M];
boolean libre = true; 
int N = 33;

chunks = M div J

process buscador[id:0..J]{

    inicio = chunks * id - chunks;
    fin = chunks * id - 1;

    for i : inicio...fin{
        if (vector[i] == N){
            <total = total +1>
        }
    }
}
```

### 3. Dada la siguiente solución de grano grueso: #### a) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias.

```python
int cant = 0; 
int pri_ocupada = 0;
int pri_vacia = 0; 
int buffer[N];
```
```python
Process Productor::
{ while (true)
 { produce elemento
 <await (cant < N); cant++>
 buffer[pri_vacia] = elemento;
 pri_vacia = (pri_vacia + 1) mod N;
 }
}
```
```python
Process Consumidor::
{ while (true)
 { <await (cant > 0); cant-- >
 elemento = buffer[pri_ocupada];
 pri_ocupada = (pri_ocupada + 1) mod N;
 consume elemento
 }
}
```


a) El codigo no funciona para resolver el problema ya que no es atomico, puede suceder que entre al productor, sume uno en la cantidad, se corte la ejecucion y pase al consumidor, este leera que la cantidad es mayor a 0 por lo que la restará e intentará acceder a la pri_ocupada pero no hay nada cargado ya que el productor no llego a cargarlo.
Tambien puede suceder que este lleno, el consumidor consuma y antes que lo saque se corte su ejecucion lo que hace que el productor produzca con el vector lleno y pise un valor

##### Solucion 1: Solo mover el piquito ">"

```python
int cant = 0; 
int pri_ocupada = 0;
int pri_vacia = 0; 
int buffer[N];
```
```python
Process Productor::
{ while (true)
 { produce elemento
 <await (cant < N); cant++
 buffer[pri_vacia] = elemento;>
 pri_vacia = (pri_vacia + 1) mod N;
 }
}
```
```python
Process Consumidor::
{ while (true)
 { <await (cant > 0); cant-- 
 elemento = buffer[pri_ocupada];>
 pri_ocupada = (pri_ocupada + 1) mod N;
 consume elemento
 }
}
```


##### Otra solucion(boolean), más quilombo pero el profe dijo que estaba bien:


```python
int cant = 0; 
int pri_ocupada = 0;
int pri_vacia = 0; 
int buffer[N];
boolean puede = true;
```
```python
Process Productor::
{ while (true)
 { produce elemento
 <await (cant < N) and puede;puede = false; cant++>
 buffer[pri_vacia] = elemento;
 pri_vacia = (pri_vacia + 1) mod N;
 puede = true;
 }
}
```
```python
Process Consumidor::
{ while (true)
 { <await (cant > 0) and puede;puede = false; cant--> 
 elemento = buffer[pri_ocupada];
 pri_ocupada = (pri_ocupada + 1) mod N;
 consume elemento
 puede = true;
 }
}
```


#### b) Modificar el código para que funcione para C consumidores y P productores


##### Solucion (Esta le gusto al profe)

``` python
int cant = 0; 
int pri_ocupada = 0;
int pri_vacia = 0; 
int buffer[N];
boolean puede = true;
```
```python
Process Productor[id:0..C]
{ while (true)
 { produce elemento
 <await (cant < N) and puede;puede = false; cant++>
 buffer[pri_vacia] = elemento;
 pri_vacia = (pri_vacia + 1) mod N;
 puede = true;
 }
}
```
```python
Process Consumidor[id:0..P]
{ while (true)
 { <await (cant > 0) and puede;puede = false; cant--> 
 elemento = buffer[pri_ocupada];
 pri_ocupada = (pri_ocupada + 1) mod N;
 consume elemento
 puede = true;
 }
}
```



##### Otra solucion (Me la dijo el profe pero siento que es re chancha porque haces atomico todo)


``` python
int cant = 0; 
int pri_ocupada = 0;
int pri_vacia = 0; 
int buffer[N];
```
``` python
Process Productor[id:0..C]
{ while (true)
 { produce elemento
 <await (cant < N); cant++>
 buffer[pri_vacia] = elemento;
 pri_vacia = (pri_vacia + 1) mod N;>
 }
}
```
``` python
Process Consumidor[id:0..P]
{ while (true)
 { <await (cant > 0); cant-- 
 elemento = buffer[pri_ocupada];
 pri_ocupada = (pri_ocupada + 1) mod N;>
 consume elemento
 }
}
```





### 4. Resolver con SENTENCIAS AWAIT (<> y <await B; S>). Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar. 

#### Mi solucion, el profe dijo que estaba bien pero que es arriesgado, ya que dependes en la implementacion de la cola.

```python
Queue cola;
int libres = 5;
```


```python
process instancia[id:0..N]{ 
    <await libres > 0; libres-->
    instancia = cola.pop;
    //usa la instancia
    <cola.push(instancia); libre++>
}
```

##### Otra solucion, el profe dijo que es mejor esta ya que no tenes el riesgo de que sucedan 2 pop a la vez, ya que al haber dos pop a la vez depende de como el lenguaje implemente la cola es si sacan el mismo elemento, diferente o ninguno.

```python
process instancia[id:0..N]{ 
    <await cola.length > 0; instancia = cola.pop;>
    //usa la instancia
    <cola.push(instancia);>
}
```


### 5.