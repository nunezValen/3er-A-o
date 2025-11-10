# PMA


```js

chan discapacitados(int,int)
chan ancianos(int,int)
chan normales(int,int)
chan llegada()
chan RyC[N](remera,chip)

Process Persona(id:0..N-1){
    
    
    dni = obtenerDNI()
    prioridad = obtener(prioridad)


    if (prioridad == 0){
        send discapacitados(dni,id)
    }
    elif (prioridad == 1 ){
        send ancianos(dni,id)
    }
    else {
        send normales(dni,id)
    }
    send llegada()
    receive RyC[id](remera,chip)

}

Process Organizador{

    while true {
        recieve llegada() // para evitar el busy waiting

        if(!discapacitados.isEmpty()){
            recieve discapacitados(dni,id)
        }
        elif(!ancianos.isEmpty()){
            recieve ancianos(dni,id)
        }
        else {
            recieve normales(dni,id)
        }
        // Le mando su remera y chip
        send RyC[id](obtenerRyC(dni))
    }
}
```



# ADA


```js

PROCEDURE UPS IS

    TASK TYPE ClienteAnsioso IS
    clientesAnsiosos : array(1..N) of ClienteAnsioso
    
    TASK TYPE ClientePaciente IS
    clientesPacientes : array (1..N2) of ClientePaciente

    TASK Empleado IS
        Entry Atencion( qr : in text; paquete : out paquete)
    END Empleado

    TASK BODY Empleado IS
    BEGIN
        For (i:  1 to N1+N2) LOOP

            ACCEPT Atencion( qr : in text; paquete : out paquete) DO
                paquete := obtenerPaquete(qr)
            END Atencion
        END LOOP
    END
    

    TASK BODY ClienteAnsioso IS
    qr : text
    paquete : paquete
    BEGIN
        for (1 to 3) LOOP
            SELECT Empleado.Atencion(obtenerQR(),paquete)
                break
            OR caminata()
            END SELECT
        END LOOP
    END Cliente

    TASK BODY ClientePaciente IS
    qr : text
    paquete : paquete
    BEGIN
        for (1 to 3) LOOP
            SELECT Empleado.Atencion(obtenerQR(),paquete)
                break
            OR Delay 600.00
                caminata()
            END SELECT
        END LOOP

    /*
    sin break para ambos clientes seria :

        while (intentos < 3 and !atendido) LOOP
            SELECT Empleado.Atencion(obtenerQR(),paquete)
                atendido := true
            OR DELAY 600.00
                caminata()
                intentos ++
            END SELECT
        END LOOP
    */
    END Cliente

BEGIN

    NULL

END
```


