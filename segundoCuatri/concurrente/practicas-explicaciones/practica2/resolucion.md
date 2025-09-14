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


#### 3. Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola.  Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben  sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente para su reúso.  

```python
sem mutex = 5; sacando = 1;
Cola cola;


process proceso[i:1..P]{
    
    P(mutex)
    P(sacando) # Necesito sacando para que no intenten sacar la misma al mismo tiempo
    instancia  =  cola.pop()
    V(sacando)
    # usar instancia
    cola.push(instancia)
    V(mutex)

}

```


#### 4. Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al  mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y  usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción: 
• no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD. 
• no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD. 
Indique si la solución presentada es la más adecuada. Justifique la respuesta. 

```pascal
Var
sem: semaphoro := 6;
alta: semaphoro := 4;
baja: semaphoro := 5
```
<table style="width:100%">
   <thead>
        <tr>
            <th>
                Process Usuario-Alta [I:1..L]:: 
            </th>
            <th>
                Process Usuario-Baja [I:1..K]:: 
            </th>
        </tr>
   </thead>
<tbody>
<tr>
 <td rowspan=4>

```cpp
 { P (sem);
 P (alta);
 //usa la BD
 V(sem);
 V(alta);
 }
```

</td>
 <td rowspan=4>

```cpp
{ P (sem);
 P (baja);
//usa la BD
 V(sem);
 V(baja);
 }
```
</td>
</td>
</tr>
</tbody>
</table>


##### Respuesta

Estan mal declaradas las variables
```python
sem sem = 6;
sem alta = 4;
sem baja = 5;
```

Y ademas se debería hacer primero p(alta) o p(baja) y luego p(sem), ya que si no podría ocurrir que usuarios estén bloqueando la entrada a la BD sin poder acceder porque su prioridad alcanzo el limite y estarían evitando que otros de distinta prioridad puedan acceder (cuando deberían poder hacerlo porque su prioridad no alcanzo el máximo) Ej.: 6 de alta bloquean la BD pero solo 4 pudieron acceder a ella y los otros 2 no pueden hacerlo pero aun asi tienen la BD bloqueada evitando que entren 2 de prioridad baja (que deberían poder). No se maximiza la concurrencia y hay demora innecesaria.


#### 5. En una empresa de logística de paquetes existe una sala de contenedores donde se  preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con  capacidad para N contenedores. Resuelva considerando las siguientes situaciones: 
a) La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de  preparar los paquetes y dejarlos en los contenedores; un empelado Entregador  que se ocupa de tomar los paquetes de los contenedores y realizar la entregas.  Tanto el Preparador como el Entregador trabajan de a un paquete por vez. 

```python
Paquete paquetes[N]
sem vacio = 1;
sem lleno = 0;
```

```python
Process preparador {
    while (true){
        # Preparar paquete
        P(vacio)
        paquetes.push(paquete)
        V(lleno)
    }
}
```

```python
Process entregador {
    while (true){
        P(lleno)
        paquete = paquetes.pop()
        V(vacio)
        # Entregar paquete
    }
}
```



b) Modifique la solución a) para el caso en que haya P empleados Preparadores. 

```python
Paquete paquetes[N]
sem vacio = 1;
sem lleno = 0;
sem mutexPreparador = 1;
```

```python
Process preparador {
    while (true){
        # Preparar paquete
        P(vacio)
        P(mutexPreparador)
        paquetes.push(paquete)
        P(mutexPreparador)
        V(lleno)
    }
}
```

```python
Process entregador {
    while (true){
        P(lleno)
        paquete = paquetes.pop()
        V(vacio)
        # Entregar paquete
    }
}
```




c) Modifique la solución a) para el caso en que haya E empleados Entregadores. 

```python
Paquete paquetes[N]
sem vacio = 1;
sem lleno = 0;
sem mutexEntregador = 1;
```

```python
Process preparador {
    while (true){
        # Preparar paquete
        P(vacio)
        paquetes.push(paquete)
        V(lleno)
    }
}
```

```python
Process entregador {
    while (true){
        P(lleno)
        P(mutexEntregador)
        paquete = paquetes.pop()
        V(mutexEntregador)
        V(vacio)
        # Entregar paquete
    }
}
```




d) Modifique la solución a) para el caso en que haya P empleados Preparadores y E 
empleadores Entregadores. 

```python
Paquete paquetes[N]
sem vacio = 1;
sem lleno = 0;
sem mutexPreparador = 1;
sem mutexEntregador = 1:
```

```python
Process preparador {
    while (true){
        # Preparar paquete
        P(vacio)
        P(mutexPreparador)
        paquetes.push(paquete)
        P(mutexPreparador)
        V(lleno)
    }
}
```
```python
Process entregador {
    while (true){
        P(lleno)
        P(mutexEntregador)
        paquete = paquetes.pop()
        V(mutexEntregador)
        V(vacio)
        # Entregar paquete
    }
}
```


#### 6. Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos: 
a) Implemente una solución suponiendo que existe una única impresora compartida por  todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar  el orden. Existe una función Imprimir(documento) llamada por la persona que simula el  uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

```python
sem mutex = 1;

Process persona [id:1..N]{
    P(mutex)
    Imprimir(Documento)
    V(mutex)
}
```

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada. 


c) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1). 

d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora. 

e) Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar. 

```cpp
colaLlegada c;
sem espera[P] = ([P] 0);
sem mutex = 1;
bool libre = true;

Process persona [i:0..P-1]{
    Documento documento;
    int aux;
    P(mutex);
    if (libre){
        libre = false;
        V(mutex);
    }
    else {
        c.push(i);
        V(mutex);
        P(espera[i]);
    }
    Imprimir(documento);
    P(mutex);
    if (c.isEmpty()){
        libre = true;
    }
    else {
        aux = c.pop();
        V(espera[aux]);
    }
    V(mutex);
}
```

c)

```cpp
int proximo = 0;
sem espera[P] = ([P] 0);

Process persona [i:0..P-1]{
    Documento documento;
    if (proximo != i){ //se podria sacar este if pero se deberia inicializar el semaforo del proceso con i=0 en 1
        P(espera[i]);
    }
    Imprimir(documento);
    proximo++;
    V(espera[proximo]);
}
```

d)

```cpp
sem espera[P] = ([P] 0);
sem mutex = 1;
sem listo = 0;
sem llena = 0;
colaLlegada c;

Process persona [i:0..P-1]{
    Documento documento;
    P(mutex);
    c.push(i);
    V(mutex);
    V(llena);
    P(espera[i]);
    Imprimir(documento);
    V(listo);
}

Process coordinador{
    int aux;
    for i = 0..P-1{
        P(llena);
        P(mutex);
        aux = c.pop();
        V(mutex);
        V(espera[aux]);
        P(listo);
    }
}
```

e)

```cpp
sem espera[P] = ([P] 0);
sem mutex = 1;
sem llena = 0;
colaLlegada c;
int impresoras[5] = {0,1,2,3,4};
sem cantImpres = 5;
int impersona[P] = ([P] -1);
sem mutexImpresoras = 1;

Process persona[i:0..P-1]{
    Documento documento;
    P(mutex);
    c.push(i);
    V(mutex);
    V(llena);
    P(espera[i]);
    Imprimir(documento, impersona[i]);
    P(mutexImpresoras);
    impresoras.push(impersona[i]);
    V(mutexImpresoras);
    V(impresoras);
}

Process coordinador{
    int aux;
    int impaux;
    for i = 0..P-1{
        P(llena);
        P(mutex);
        aux = c.pop();
        V(mutex);
        P(impresoras);
        P(mutexImpresoras);
        impaux = impresoras.pop();
        V(mutexImpresoras);
        impersona[aux] = impaux;
        V(espera[aux]);
    }
}
```