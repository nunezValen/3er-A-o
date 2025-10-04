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