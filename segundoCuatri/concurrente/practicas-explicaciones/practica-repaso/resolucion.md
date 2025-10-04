# Practica de repaso

# Semaforos

## Ejercicio 1

1- Resolver los problemas siguientes:

a- En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE
en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo
con el orden de llegada. Implemente una solución utilizando únicamente procesos Persona. Nota:
la función UsarTerminal() le permite cargar la SUBE en la terminal disponible.

b- Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles.
Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera.
Recuerde que sólo debe emplear procesos Persona. Nota: la función UsarTerminal(t) le permite
cargar la SUBE en la terminal t


### Resolucion a

```java
sem mutex = 1
sem espera[N] = ([N] 0)
int siguiente = -1
cola orden
bool libre = true

Process Persona[id:0..N-1]{
    P(mutex) // Bloqueo la variable libre ya que la tengo que consultar y/o modificar
    if(libre){
        libre = false // Si esta libre lo uso
        V(mutex) // Libero el semaforo porque ya cambie la variable libre
    } else {
        cola.push(id) // si no esta libre encolo mi id
        V(mutex)
        P(espera[id]) // y espero a que en el arreglo en la posicion de mi id se libere
    }


    UsarTerminal()


    P(mutex) // Tengo que consultar la cola asi que bloqueo
    if (!cola.empty()){
        V(espera[cola.pop()]) // libero el proximo
    } else{
        libre = true
    }
    V(mutex)

}

```


### Resolucion b

```java
sem semCOla = 1
sem mutex = 1
sem espera[N] = ([N] 0)
int cantLibres

cola terminales
cola orden


Process Persona[id:0..N-1]{
    P(mutex) // Bloqueo la variable cantidadLibres ya que la tengo que consultar y/o modificar
    if(cantLibre > 0){
        cantLibres-- // Si esta libre lo uso
        V(mutex) // Libero el semaforo porque ya cambie la variable libre
    } else {
        orden.push(id) // si no esta libre encolo mi id
        V(mutex)
        P(espera[id]) // y espero a que en el arreglo en la posicion de mi id se libere
    }

    // Saco la proxima terminal libre de la cola para usar
    P(semCola)
    t = terminales.pop()
    V(semCola)
    UsarTerminal(t)

    // Devuelvo la terminal usada para que la use el siguiente
    P(semCola)
    termines.push(t)
    V(semCola)


    P(mutex) // Tengo que consultar la cola asi que bloqueo
    if (!orden.empty()){
        V(espera[orden.pop()]) // libero el proximo
    } else{
        cantLibres++
    }
    V(mutex)

}

```





### Resolucion b alternativa

```java

terminal terminalAsignada[N]

Semterminales = 1
sem semCOla = 1
sem mutex = 1
sem espera[N] = ([N] 0)
int cantLibres

cola terminales
cola orden


Process Persona[id:0..N-1]{
    P(mutex) // Bloqueo la variable cantidadLibres ya que la tengo que consultar y/o modificar
    if(cantLibre > 0){
        cantLibres-- // Si esta libre lo uso
        
        P(Semterminales) // Bloqueo la variable de la cola de terminales
        t = terminales.pop() // Saco una terminal
        V(Semterminales) // Desbloqueo
        
        V(mutex) // Libero el semaforo porque ya cambie la variable libre
    } else {
        orden.push(id) // si no esta libre encolo mi id
        V(mutex)
        P(espera[id]) // y espero a que en el arreglo en la posicion de mi id se libere

        t = terminalAsignada[id] // Saco mi terminal asignada
    }

    UsarTerminal(t) // Uso la terminal


    P(mutex) // Tengo que consultar la cola asi que bloqueo
    if (!orden.empty()){
        siguiente = orden.pop() // Me guardo el id del siguiente en la fila
        terminalAsignada[siguiente] = t // Le asigno mi terminal ya que es la proxima en ser liberada que va a ser la primera en liberarse 
        V(espera[siguiente]) // libero el proximo
    } else{
        terminales.push(t) // Si no hay nadie esperando terminal la pusheo a la cola de todas las terminales
        cantLibres++
    }
    V(mutex) // Desbloqueo el semaforo de la cola orden

}

```

## Ejercicio 2
2- Implemente una solución para el siguiente problema. Un sistema debe validar un conjunto de 10000
transacciones que se encuentran disponibles en una estructura de datos. Para ello, el sistema dispone
de 7 workers, los cuales trabajan colaborativamente validando de a 1 transacción por vez cada uno.
Cada validación puede tomar un tiempo diferente y para realizarla los workers disponen de la
función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el
procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada
resultado de la función de validación. Nota: maximizar la concurrencia



```java
sem suma
sem transaccionSem
cola transacciones[10000]
int resultados[0..9] = ([9]0)



Process Worker[id:0..7-1]{
    transaccion t;

    P(cola) // Bloqueo la cola porque quiero ver si esta vacia
    while(!transacciones.empty()){    
        V(transaccionSem) // Libero

        P(transaccionSem) // Bloqueo para hacer pop
        t = transacciones.pop()
        V(transaccionSem) // Libero

        P(suma) // Bloqueo para sumar
        resultados[Validar(t)]++
        V(suma) // Libero

        P(transaccionSem)
        if(transacciones.empty()){ // Si la cola esta vacia es porque saque el ultimo por lo que debo imprimir
            print(resultados)
        }
        V(transaccionSem)    // Libero
        
        P(transaccionSem) // Bloqueo para volver a preguntar en el while
    }
}

```

### Solucion alternativa ya que no me gusto que para cada cosa tenga que bloquear




```java
sem suma
array transacciones[10000];
int resultados[0..9] = ([9]0)

Process Worker[id:0..7]{

    int base  = 10000 div 7       // = 1428
    int resto = 10000 mod 7       // = 4
    int inicio, fin
    int termine = 0
    sem mutex = 1;
    // calcular inicio
    inicio = id * base // Ejemplo para 0 va a arrancar desde 0 hasta 1428 
    if (id < resto){
        inicio = inicio + id // Si mi id es menor que el resto le sumo al inicio mi id, ya que para calcular el fin a los 4 primeros procesos(los que tienen id mas chico que el resto) les agrego una posicion más al arreglo
    }
    else{
        inicio = inicio + resto // Si no mi inicio se mantiene normal
    }
        

    // calcular fin
    fin = inicio + base - 1 // El fin va a ser mi inicio + mi base - 1, en el caso del 0 hasta 1428
    if (id < resto) {
        fin = fin + 1 // Como el id es menor va hasta 1429 y el id 1 arranca en 1430 
    }


    for i: inicio to base {
        resultados[transaccions[i]] ++ // Sumo los resultados de las transacciones
    }
    P(mutex)
    if (termine++ == 7){ // si es igual a 7 es pq fui el ultimo
        print(resultados)
    }
    V(mutex)




}

```


# Monitores


## Ejercicio 1

1- Resolver el siguiente problema. En una elección estudiantil, se utiliza una máquina para voto
electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina
de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto.
La máquina de voto sólo puede ser usada por una persona a la vez. Nota: la función Votar() permite
usar la máquina.


```java
Process Persona[0..N-1]{
    bool prioridad = ... // False o true dependiendo si es embarazada o no

    Mesa.Llegada(prioridad)
    Votar()
    Mesa.Salir()


}

Process Autoridad(){
    while true{
 
    Mesa.EsperarLlegada()
    Mesa.DarAcceso()
    }   
}

```


```java
Monitor Mesa{
    cond esperando
    cond autoridad
    cond prioridad
    cond confirmacion
    int totalPri
    int totalEsp

    Procedure Llegada(esPrioritario: IN bool){

        if(!libre){ 
            if(esPrioritario){  
                totalPri++
                wait(prioridad) // Si esta ocupado y tengo prioridad me agrego a una cola aparte
            } else {
                totalEsp++
                wait(esperando) // Si no tengo prioridad me agrego a la comun
            }
        }
        libre = false
        signal(autoridad)
        wait(confirmacion)
    }

    Procedure EsperarLlegada(){
        if(libre){
            wait(autoridad) // Si no llego nadie me duermo
        }
    }

    Procedure DarAcceso(){
        signal(confirmacion)
    }

    Procedure Salir(){
        if(totalPri>0){
            signal(prioridad)
        } elif(totalEsp>0){
            signal(esperando)
        } else {
            libre = true
        }
    }

}

```



## Ejercicio 2


