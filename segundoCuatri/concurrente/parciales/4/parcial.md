# Parcial 4

#### nota: Son las 22:45 cuando resuelvo esto, tengo sueño ya hice como 40 ejercicios, no quiero más, asi que no confiar en la resolucion


## Enunciado 1




```java

sem arrancar = 0
sem esperandoRespuesta[N] = ([N]0)
sem mutex = 1
sem entradasS = 1
int entradas = E
queue cola 

Process Persona[id:0..N-1]{
    comprobante = ..
    datos  = ..
    bool pude = ...
    
    P(mutex)
    cola.push(id,datos) // Me encolo
    V(mutex)
    P(esperandoRespuesta[id]) // Espero la respuesta
    
    pude = estados[id] // si pudo o no
    comprobante = comprobante[id] // recibo mi comprobante
    

}

Process Cajero[0..C-1]{
    id,datos


    P(arrancar) // Espero que el timer me diga que arranque

    while (true){
        if(!cola.isEmpty){
            // Si hay gente en la cola, saco sus datos
            P(mutex)
            id,datos = cola.pop()
            v(mutes)

            P(entradasS)
            if(entradas>0){
                entradas--
                V(entradas)
                realizarPago(comprobantes[id],pudo[id])   // Devuelve si pudo realizar el pago junto a su comprobante
            } else {
                print ("No hay entradas")
            }

            signal(esperandoRespuesta[id])


        }
    }
}


Process Timer{
    // Esperar a que sea la hora
    for(i : 1.. to C){

         V(arrancar) // les digo que arranquen

    }
}

```


### Que ejercicio paja, es para hacerlo con monitores NO CON SEMAFOROS



## Enunciado 2 ya son las 23:12

### Voy a hacer solo el inciso B pq si no es lo mismo siempre




```java

Process Persona[0..N-1]{
    int edad = ...

    Mirador.Pasar(edad,id)

    // Mirando por el mirador

    Mirador.Salir()

}


Monitor Mirador{

    ColaOrdenada cola
    cond esperandoPasar[N] = ([N] 0)
    bool libre = true


    Procedure Pasar(edad : IN int,id: IN int){

        if(!libre){
            cola.insertarOrdenado(edad)
            wait(esperandoPasar[id])
        } else {
            libre = false
        } 
    }

    Procedure Salir(){
        if(!cola.isEmpty()){
            signal(esperandoPasar[cola.pop()]) // Si no hay nadie esperando saco el id del mayor para que sea el siguiente
        } else {
            libre = true // si no hay nadie esperando dejo libre para el proximo
        }
    }
}

```


#### Otra solucion, ya que en la anterior ejercicio veo el siguiente problema: Si tengo 20 años por qué le tengo que dejar el lugar a alguien con 21? Seria mejor tener dos colas condicion, una por prioridad y otra sin, entonces todos los mayores a 60 tienen la misma prioridad y lo mismo para los menores a 60  


```java

Process Persona[0..N-1]{
    int edad = ...

    Mirador.Pasar(edad) // solo le paso edad

    // Mirando por el mirador

    Mirador.Salir()

}


Monitor Mirador{
    cond sinPrioridad
    int cantSin
    cond prioridad
    int cantPri
    bool libre = true


    Procedure Pasar(edad : IN int){
        if(!libre){
            if (edad > 60){
                cantPri++
                wait(prioridad) // si no esta libre y tengo mas de 60 espero en prioridad
            } else {
                cantSin++
                wait(sinPrioridad) // Si no tengo +60 entonces espero en la otra
            }
        } else {
            libre = false // si esta libre entonces paso yo
        }
    }

    Procedure Salir(){
        if(cantPri > 0){ // Si hay alguien en prioridad
            cantPri--
            signal(prioridad)
        } else if ( cantSin > 0) { // Si no hay gente con prioridad pero sí sin prioridad
            cantSin--
            signal(sinPrioridad)
        } else {
            libre = true // Si no hay nadie en ninguna entonces dejo libre
        }
    }
}

```


### Son 23:30 y mañana rindo a las 8 pero ya fue, uno más

