# PARCIAL 5(CINCOOOOOOOO) ULTIMO QUE HAGO. MÁS VALE APROBAR MAÑANA



## Enunciado 1


```java


cola listaCuits;

sem mutex
sem totalS
sem terminados
int totalGlobal[10] = ([10]0) // Contador de cuits por digito global
int termine = 0


Process Worker[id:0..4]{
    int total[10] = ([10]0)

    P(mutex)
    while (!cola.isEmpty()){
        cuit = cola.pop()
        V(mutex)

        // Sumo que digito me da
        total[obtenerDV(cuit)]++

        P(mutex) // Bloqueo para la siguiente repeticion del while
    }

    // Una vez no hay mas elementos en la cola

    P(totalS) 
    for (i: 0..9){
        
        totalGlobal[i] += total[i] // Sumo los locales al global
    }
    V(totalS)

    P(terminados)
    termine++
    if(termine == 5) {
        print(totalGlobal[]) // Si soy el ultimo en terminar imprimo todos los totales
    }
    V(terminados)

}


```

#### Nota: Ni idea si es lo maximo concurrente posible, pero ya son las 00:00 y quiero dormir

## Enunciado 2 Y ULTIMOO :D

```java

Process Empleado{

    indicaciones = .. 
    tarjeta = ..
    int idCli = -1

    Admin.RecibirIndicaciones(indicaciones,idCli)
    tarjeta = hacerTarjeta(indicaciones)
    Admin.EntregarTarjeta(tarjeta,idCli)

}


Process Cliente[id:0..C-1]{

    indicaciones = ...
    tarjeta

    Admin.EnviarIndicaciones(indicaciones,tarjeta,id)
}


Monitor Admin{

    cond empleado
    cond esperandoTarjeta[C]
    txt tarjetas[C]

    Procedure EnviarIndicaciones(indicaciones : IN txt; tarjeta : OUT txt,id : IN int){

        cola.push(indicaciones,id)
        signal(empleado) // Levanto al empleado, si no llego todavia no pasa nada pq la cola no estará vacia
        wait(esperandoTarjeta)
        tarjeta = tarjetas[id]
    }

    Procedure RecibirIndicaciones(indicaciones: OUT txt; id: OUT int){
        if(cola.isEmpty()){ // si esta vacia es porque no hay nadie
            wait(empleado)
        }
        indicaciones,id = cola.pop() // saco indicaciones e ID

    }

    Procedure EntregarTarjeta(tarjeta: IN txt; idCli: IN int){

        tarjetas[idCli] = tarjeta
        signal(esperandoTarjeta) // Levanto al que espera la tarjeta
    }


}


```


### Bueno ya esta son 00:12. Mañana se riende, exitos a cualquiera que este leyendo esto.