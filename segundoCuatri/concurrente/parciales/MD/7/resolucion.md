# PMA 

```js

chan discapacitados(idPersona : int)
chan normal(idPersona : int)
chan pedido()

chan maquina_D[R](idMaquina : int)
chan maquina_N[E](idMaquina : int)
chan termine()


Procedure Admin{
    cola pri
    cola no_pri
    maquinas : array[1..3] maquinas
    cant_libres : 3

    while true {
        recieve pedido()
        // Si discapacitados no esta vacia y hay libres, le mando una maquina libre y resto en la cantidad
        if (!discapacitados.isEmpty() and cant_libres>0){
            recieve discapacitados(idPersona)
            send maquina_D[idPersona](cant_libres)
            cant_libres--
        }
        // Esta guarda dice, si discpacitados esta vacia y cant libres es mayor que 0, le mando a una persona no discapacitada una maquina. 
        [] (discapcitados.isEmpty) and cant_libres>0 -> {
            recieve normal(idPersona)
            send maquina_N[idPersona](cant_libres)
            cant_libres--
        }
        // Esta guarda es para ir agarrando si hay gente que termino y sumo a cant libres
        [] !termine.isEmpty() ->{
            recieve termine()
            cant_libres++
        }

    }


}



Procedure Discapacitado[id:1..R]{
    // Mando a la cola con prioridad
    send discapacitados(id)
    
    // Espero a recibir el id de la maquina a usar
    recieve maquina_D[id](idMaquina)
    // usar maquina
    maquina[idMaquina].usar()
    // Mando que termine de usarla
    send termine()
}

Procedure Persona[id:1..E]{

    // Mando a la cola normal sin prioridad
    send normal(id)
    // Aviso que hay un pedido
    send pedido()
    
    // Espero a recibir el id de la maquina a usar
    recieve maquina_N[id](idMaquina)
    // Usar maquina
    maquina[idMaquina].usar()
    // Mando que termine de usarla
    send termine()
}
```