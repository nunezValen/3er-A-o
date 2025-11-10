# REPASO DE ALGORITMOS EN MEMORIA DISTRIBUIDA

# PMA
- Los canales son una cola y garantiza orden de llegada


## Modelos

### Coordinador
- Cada que son varios(empleados) y hay que preguntar por empty
- Cuando son varios y hacen algo más (ejemplo esperar)
- Si son varios y necesitas prioridad

```js
chan atencion(int) // Canal de los clientes para ser atendidos
chan pedido(int) // Canal de los pedidos de los empleados para consultar
chan siguienteCli[2](int) // Canal del admin para mandarle el id del siguiente a los empleados

Process Cliente[id:0..N-1]{

    send atencion(id)

}



Process Admin {

    int IdEmp
    int IdCli

    while true {
        recieve pedido(IdE) // Recibo el pedido de consulta del empleado 
        if (empty(atencion)){ // Si no hay nadie esperando pongo en -1
            IdCli = -1;
        } else {
            recieve atencion(IdCli) // Si no, agarro el del cliente
        }
        send siguiente[IdEmp](IdCli) // Le mando al empleado
    }

}


Process Empleado[id:0..1]{
    
    while true{
        send pedido(id) // Digo al admin que quiero agarrar un pedido
        recieve siguiente[id](idCli) // Recibo el id del cliente
        if (idCli == -1){ // Si es -1(que no hay nadie) me duermo
            sleep(15min)
        } else {
            atender(IdCli) // si no lo atiendo.
        }
    }
}
```


### Prioridad y terminar 



```js


chan impresora(text)
chan impresorCanal(text)
chan imprimir(text)

Process Administrativo[id:0..N-1]{
    text documento
    for i: 1 to 10 {
        // Trabajando
        send impresora(documento)
    }
}

Process Director{
    text documento
    for i: 1 to 10 {
        send impresora(documento)
    }
}

// Tengo que usar un proceso admin para manejar y que todos puedan terminar
Process Administrador {
    text documento

    for i: 1 to N*10 + 10 {
        if(!impresoraPrioridad.isEmpty()){
            recieve impresoraPrioridad(documento)
        } else {
            recieve impresora(documento)
        }
        send imprmirCanal(documento)
    }
    for i : 1 to 3 {    
        // Una vez no hay más documentos que recibir le mando a la impresora null para que terminen
        send imprimirCanal(null)
    }
}


Process Impresora[id:0..2]{
    text documento

    while documento =! null {
        recieve imprimirCanal(documento)
        imprmir(documento)
    }
}

```


# PMS
- Para asegurar orden hay que usar un coordinador que tome los pedidos y los pushee a una cola, luego el empleado solicita a este y el coordinador hace un pop


## Modelos

### Coordinador y en orden. 

```js

Process Admin{
    cola fila;
    text sitioWeb
    int idExaminador

    // Recibo de cualquier examinador una web y la pusheo a la fila
    do Examinador[*]?aviso(sitioWeb) -> {
        fila.push(sitioWeb)
    }
    // Si hay algo esperando en la fila y el analizador ya pidio un sitio entonces se lo mando
    [] not fila.isEmpty(); Analizador?Llegada() -> {
        fila.pop(sitioWeb)
        Analizador!Pedido(sitioWeb)
    }
}

Process Examinador[id:0..N]{
    text sitioWeb
    while (True){
        // Consigo web
        sitioWeb = buscarWebInfectadda()
        // Le mando al admin mi web
        Admin!aviso(sitioWeb)
    }
}

Process Analizador{
    text sitioWeb
    while (True){
        // Le aviso al admin que estoy esperando un sitio
        Admin!Llegada(true)
        // Recibo del admin un sitio web 
        Admin?Pedido(sitioWeb)
        // Analizo
        analizar(sitioWeb)
    }
}

```


### Buffering (cuando tengo un proceso que tiene que mandar y seguir tiene que haber un coordinador que reciba y los meta en una cola)
Igual que arriba

- Para barrera conviene tener un proceso barrera con los dos for. 

```js

Process Alumno[0..N-1]{
    text examen
    
    // Aviso que llegue
    Barrera!Llegada()
    // Espero a poder arrancar
    Barrera!Arrancar()
    // Realizo el examen
    examen.resolver()
    Admin!Examen(examen)
}

// P > 1
Process Profesor[id:0..P-1]{
    text examen

    while examen != null {

        Admin!PedirExamen(id)
        Admin!ConseguirExamen(examen)
        corregir(examen)

    }
}

Process Barrera {

    
    For i : 0 to N{
        Alumno[*]?Llegada()
        cant++
    }
    For i: 0 to cant-1{
        Alumno[*]?Arrancar()
    }


}

Process Admin{
    cola examenes
    text examen
    int idProfesor
    int cant


    while cant > 0 {
        // Si un alumno me mando un examen entonces lo guardo en la cola
        do Alumno[*]?Examen(examen) -> {
            examenes.push(examen)
        }
        // Si tengo examenes en la cola y un profesor me pidio un examen entonces se lo mando
        [] !examenes.isEmpty(); Profesor[*]?PedirExamen(idProfesor){
            fila.pop(examen)
            Profesor[idProfesor]!ConseguirExamen(examen)
        }
        od

        // Resto a cantidad
        cant--
    }

    // Una vez salgo del while pq ya no quedan parciales que mandar le mando null a todos los profesores
    For i: 0 to P-1 {
        Profesor[i]!ConseguirExamen(null)
    }
}
```






### Passing the baton

```js

Process Espectador[id:0..E-1]{

    // Llego a la maquina
    Admin!Llegada(id)

    // Espero que me dejen usarla
    Admin?Usar(Maquina)

    Maquina.Usar()

    // Dejo la maquina
    Admin!DejarMaquina()


}

Process Admin{
    cola espectadores
    int idEsp
    bool MaquinaLibre = true
    while true{

        // Cuando llega un nuevo espectador
        do Espectador[*]?Llegada(idEsp) -> {

            // Si la maquina esta ocupada lo pongo en la cola
            if (!MaquinaLibre){
            espectadores.push(idEsp)
            }
            else { // Si esta libre la usa
                MaquinaLibre = false
                Espectador[idEsp]!Usar(maquina)
            }
        }
        [] Espectador[*]?DejarMaquina(); { // Cuando me avisa que termino de usarla
            
            if (!espectadores.isEmpty()){ // Si la cola no esta vacia significa que hay espectadores esperando para usarla, por lo que hago tipo un passing the baton y la usa el proximo
                espectadores.pop(idEsp)
                Espectador[idEsp]!Usar(maquina)
            } else {
                // Si esta vacia dejo la maquina libre para el proximo.
                MaquinaLibre = true
            }
        } 
    }
}
```



# ADA


- Se asegura orden de llegada!


### Timer: Se usa cuando necesito hacer algo durante X tiempo 

```js

Procedure Sistema is

    TASK Central is
        Entry SignalP1(signal : IN texto)
        Entry SignalP2(Signal : IN texto)
        Entry Aviso;
    END Central

    Task Timer is
        Entry Iniciar;
    END Central

    Task body Timer is
        LOOP
            // Agarro el mensaje de iniciar
            Accept Iniciar;
            // Espero 3 minutos
            delay 180
            // Aviso que pare
            Central.Aviso;
        END LOOP
    END Timer


    Task ProcesoUno;

    Task Body ProcesoUno is
        signal: texto
    BEGIN
        LOOP
            // Si no me atienden en 120 segundos (2 minutos) creo una nueva señal e intento de nuevo.
            SELECT Central.SignalP1(signal)
            OR DELAY 120
                Signal = newSignal()
            END SELECT
        END LOOP
    END ProcesoUno

    Task ProcesoDos;

    Task Body ProcesoDos is
        signal : Texto
    BEGIN
        LOOP
            // Si no me atienden al instante espero 1 minuto e intento de nuevo
            SELECT Central.SignalP2(signal)
            ELSE
                DELAY 60
            END SELECT
        END LOOP
    END ProcesoDos

    Task Body Central is
        signal : texto
        time_out = false
    BEGIN
        // Acepto la señal del p1
        Accept SignalP1(signal)
        
        LOOP
            SELECT 
                ACCEPT SignalP1(signal)
            OR
                ACCEPT SignalP2(signal)
                Timer.Iniciar; // Le aviso al timer que arranque el tiempo.
                // Mientras no me avise el timer que se termino el tiempo sigo aceptando mensajes de P2.
                while (Aviso'Count = 0) LOOP
                    Accept SignalP2(signal)
                END LOOP
                // Una vez termino el tiempo agarro el mensaje del timer para volver a poner en 0 el contador de aviso
                Accept Aviso
            END SELECT
        END LOOP
    END Central
```




### Selects y entrys


NO SE PUEDE
SELECT Admin.Enviar
OR Mesa.Pedir


SI SE PUEDE

SELECT Admin.Enviar
OR DELAY 
    MESA.PEDIR
Puedo hacer uno u otro pero no elegir no deterministicamente entre dos entry

No se puede mezclar un aceptar y enviar con ors, pero si con else o con or delay. 

- Los que mandan mensaje son los que esperan. 





```js
    For (i : 0 to 9) LOOP
        // Calculo Inicio y fin para cada worker
        base = 1000000 / 10
        inicio = i * base
        fin = inicio + base - 1
        Worker[i].Vector(inicio,fin,arreglo)            
    END LOOP

    For (i: 0 to 9) LOOP
        ACCEPT Resultado(total : in INT) DO
            promedio = total + promedio
    END LOOP
```

Se puede cambiar por 

```js
// Esto es mejor ya que mientras que estoy pasandoles para arrancar puede que el primero o alguno de ellos ya tenga su resultado y entonces no deterministicamente puedo aceptarlo
For (i: 1 to 20) LOOP´

    SELECT ACCEPT Arrancar()
    OR ACCEPT Resultado(parametros) DO
            // calcula
        END Resultado
    END SELECT
END LOOP

```