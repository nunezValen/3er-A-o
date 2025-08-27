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


### 5.En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.

#### a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas. 
#### b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
#### c) Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).
#### d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora. 

#### a)

```python
boolean usandoImpresora = false
```

```python
process persona[id:1..n]{
    <await (not usandoImpresora); usandoImpresora = true>
    imprimir(documento);
    usandoImpresora = false;
}
```

#### b)

Es igual al ejemplo 4

```python
int siguiente = -1
Queue cola
```

```python
process persona[id: 1..n]{

    // Si siguiente es -1 es porque no hay nadie usando la impresora, entonces la uso y pongo mi id como siguiente. Si no me encolo

    <if (siguiente == -1) then siguiente = id else cola.queue(id)>

    // Espero que siguiente sea mi id

    <await (siguiente == id)>
    
    imprimir(documento)

    // Si no hay nada más en la cola entonces queda libre, le pongo -1, si no hago pop del siguiente.
    
    <if (cola.isEmpty()) then siguiente = -1; else siguiente = cola.dequeue(id)>

}
```



#### c) No se que cambiaria en la explicacion practica muestran que usan una cola especial, nada más. Entiendo que al usar el dequeue se saca el elemento de menor id

```python
int siguiente = -1
Queue colaPrioridad;
```

```python

process persona[id:1..n]{

    // Si siguiente es -1 es porque no hay nadie usando la impresora, entonces la uso y pongo mi id como siguiente. Si no me encolo

    <if (siguiente == -1) then siguiente = id else cola.queue(id)>

    // Espero que siguiente sea mi id

    <await (siguiente == id)>
    
    imprimir(documento)

    // Si no hay nada más en la cola entonces queda libre, le pongo -1, si no hago pop del siguiente.
    
    <if (cola.isEmpty()) then siguiente = -1; else siguiente = cola.dequeue(id)>

}


```



#### Otra solucion que se me ocurre para no depender de una cola de prioridad. Revisar con el ayudante

//Supongo que los array arrancan en false

```python
int siguiente = -1
boolean esperando[1..n]
totalEsperando = 0
```

```python
process persona[id:1..n]{

    <if (siguiente == -1) then siguiente = id else esperando[id] = true; totalEsperando++>

    <await (siguiente == id)>
    
    imprimir(documento)
    
    <if (totalEsperando == 0 then siguiente = -1; else {
        for i = 0..n {
            if(esperando[i]){
                esperando[i] = false
                siguiente = i;
                totalEsperando--
                break;
            }
        }
    }>

}
```


#### D)

```python
queue cola
int siguiente
boolean termine = true
```


```python

process coordinador{
    for i = 1..n {
        <await termine; if (cola.isEmpty) then siguiente = -1; else siguiente = cola.dequeue()>
    }
}

```

```python
process persona[id: 1..n]{


    <if (siguiente == -1) then siguiente = id else cola.queue(id)>

    <await (siguiente == id)>
    
    imprimir(documento)

    termine = true

}
```



### 6. Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:

```python
int turno = 1;
```

```python
Process SC1::
{ while (true)
 { while (turno == 2) skip;
 SC;
 turno = 2;
 SNC;
 }
}

```

```python
Process SC2::
{ while (true)
 { while (turno == 1) skip;
 SC;
 turno = 1;
 SNC;
 }
}

```

### Propiedades:

#### Exclusión mutua: A lo sumo un proceso está en su SC

#### Ausencia de Deadlock (Livelock): si 2 o más procesos tratan de entrar a sus SC, al menos uno tendrá éxito.

#### Ausencia de Demora Innecesaria: si un proceso trata de entrar a su SC y los otros están en sus SNC o terminaron, el primero no está impedido de entrar a su SC.

#### Eventual Entrada: un proceso que intenta entrar a su SC tiene posibilidades de hacerlo (eventualmente lo hará)

##### Respuesta: podemos ver que se cumple con la propiedad 1,2 y 4. Pero la 3(Ausencia de demora innecesaria) no se cumple, ya que se le impide al proceso ejecutar su seccion no critica a no ser que sea su turno. Una forma de solucionarlo seria quitando la SNC y ponerla antes de la condicion del while, en ambos procesos.

```python
Process SC1::
{ while (true)
 { 
    SNC;
    while (turno == 2) skip;
    SC;
    turno = 2;
 }
}

```

```python
Process SC2::
{ while (true)
 { 
    SNC;
    while (turno == 1) skip;
    SC;
    turno = 1;
 }
}

```


### 7. Todavia no puedo, no dimos teoria 3
