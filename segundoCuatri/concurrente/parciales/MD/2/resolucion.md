# SEGUNDO PARCIAL

# PMA


```js

chan ObtenerPaquete[id](paquete)
chan atencion(qr,id)

Process Empleado [id:1..3]{
    qr

    while true {

        // Recibo a un cliente
        receive atencion(qr,id)
        // Obtengo su paquete
        paquete := obtenerPaquete(qr)

        // Se lo mando
        send ObtenerPaquete[idCli](paquete)
    }
}

Process Cliente[id:0..N-1]{
    // Me pongo en la fila
    send atencion(obtenerQr(idCli),id)
    // Espero mi paquete
    receive ObtenerPaqueta[id](paquete)
} 
```



# PMS

```js


Process Empleado[id:1..3]{

    while true {
        // Le avisso al coordinador que estoy listo
        Coordinador!Listo(id)
        // Espero que me diga cual es el siguiente
        Coordinador?EnviarSiguiente(idCli)

        Cliente[idCli]!Pasar(id)
        Cliente[idCli]?DarQr(qr)

        paquete = obtenerPaquete(qr)

        Cliente[idCli]!Paquete(paquete)

    }

}


Process Coordinador{
    cola clientes
    cola empleados

    while true {
        
        if Clientes[*]?Atencion(id) {
            clientes.push(id)
        }
        [] !clientes.isEmpty(); Empleado[*]?Listo(idEmp){
            Empleado[idEmp]!EnviarSiguiente(clientes.pop())
        }
        
        /*
        if  empleados.isEmpty(); Cliente[*]?Atencion(id)-> {
            clientes.push(id)
        }
        [] !empleados.isEmpty(); Cliente[*]?Atencion(id) -> {
            Empleado[empleados.pop()]!EnviarSiguiente(id)
        }
        [] !clientes.isEmpty(); Empleado[*]?Listo(idEmp)  ->{
            Empleado[idEmp]!EnviarSiguiente(clientes.pop())
        }
        [] Empleado[*]?Listo(idEmp) -> {
            empleados.push(idEmp)
        }
        */
    }


}


Process Cliente[id:0..N-1]{

    Admin!Atencion(obtener)
    Empleado[*]?Pasar(idEmp)
    
    QR = obtenerQR(id)

    Empleado[idEmp]!DarQR(QR)
    Empleado[idEmp]?Paquete(paquete)

}


```




# ADA

```js

Procedure Calculo IS

    TASK Administrador IS
        Entry QuieroPalabra(palabra : out text)
        Entry Total (total : in int)
    END Administador

    TASK TYPE Contador;
    contadores : array(1..N) of Contador;


    TASK BODY Contador IS
    palabra : text
    total : int = 0
    bd : base_datos
    BEGIN
        // Le pido la palabra al admin
        Administrador.QuieroPalabra(palabra)
        // Cuento
        total = bd.contarPalabras(palabra)
        // Se la mando al admin
        Administrador.Total(total)
    END Contador


    TASK BODY Administrador IS

    total_global : int = 0
    palabra_enviar : text
    BEGIN
        palabra_enviar = obtenerPalabra()
        FOR (I : 1 to N*2) LOOP

            SELECT 
                ACCEPT QuieroPalabra(palabra : out text) DO
                    palabra := palabra_enviar
                END QuieroPalabra
            OR
                ACCEPT Total(total : in int) DO
                    total_global := total + total_global;
                END Total
            END SELECT
        END LOOP
    END Administrador
         
BEGIN
    NULL
END Calculo

```


Otra solucion podria ser con un proceso contador_total capaz

```js

Procedure Calculo IS

    TASK Administrador IS
        Entry QuieroPalabra(palabra : out text)
        Entry Total (total : in int)
    END Administador

    TASK Contador_total IS
        Entry Total (total : in int)
    END Contador_total

    TASK TYPE Contador;
    contadores : array(1..N) of Contador;


    TASK BODY Contador IS
    palabra : text
    total : int = 0
    bd : base_datos
    BEGIN
        // Le pido la palabra al admin
        Administrador.QuieroPalabra(palabra)
        // Cuento
        total = bd.contarPalabras(palabra)
        // Se la mando al contador de total
        Contador_total.Total(total)
    END Contador

    TASK BODY Contador_total IS
    total_global : int = 0
    BEGIN
        // Voy aceptando el total de cada worker contador
        for (i : 1 to N) loop
            ACCEPT Total(total : in int) DO
                total_global := total_global + total;
            END Total
        end loop
        // Una vez termina se lo paso al admin
        Administrador.Total(total_global)
    END Contador_total


    TASK BODY Administrador IS
    total_global : int = 0
    palabra_enviar : text
    BEGIN
        palabra_enviar = obtenerPalabra()
        FOR (I : 1 to N) LOOP
        // El admin solo manda las palabras a cada worker contador
            ACCEPT QuieroPalabra(palabra : out text) DO
                palabra := palabra_enviar
            END QuieroPalabra
        END LOOP

        // El admin se queda esperando el total
        ACCEPT Total(total : in int) DO
            total_global := total
        END Total
    END Administrador
         
BEGIN
    NULL
END Calculo

```

Siento que esta solucion es más concurrente, ya que mientras el admin manda las palabras ya se puede ir sumando el total y no se quedan esperando, porque si bien no deterministicamente hace que se elija sin prioridad, siempre una va luego de la otra. 
