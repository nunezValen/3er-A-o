# PMA
El ejercicio no pide que el documento tiene que ser recibido por el empleado pero bueno, supongo que seria correcto que si lo fuera
```js

chan imprimir(documento : text; idEmp : int)
chan listo(impresion : text)


Process Empleado[id:1..100]{
    send imprimir(documento,id)
}

Process Impresora[id:1..5]{
    while true{
        recieve imprimir(documento,idEmp)
        impresion = documento.imprimir()
        send listo[idEmp](impresion)
    }
}
```

# PMS

```js

Process Empleado[id:1..100]{
    
    Admin!Imprimir(id,documento)
    Impresora[*]?Recibir(impresion)

}


Process Impresora[id: 1..5]{

    while true {
        Admin!Lista(id)

        Admin?Imprimime(idEmp,documento)

        impresion := documento.imprimir()

        Empleado[idEmp]!Recibir(impresion)
    }

}

Process Admin{
    cola esperando

    while true { // Hay busy waiting?? NO
        if Empleado[*]?Imprimir(id,documento) -> {
            esperando.push(id,documento)
        }
        [] !esperando.isEmpty(); Impresora[*]?Lista(id) -> {
            impresora[id]!Imprimime(esperando.pop())
        }
    }
}
```


# ADA

```js

PROCEDURE WEB IS


    TASK Type Banco IS
        Entry Cotizacion(cot : out real)
        Entry ID (id : in int)
    END Banco
    bancos : array (1..20) of Banco

    TASK Tarea IS
        Entry Listo(id : in int)
    END Tarea


    TASK BODY Banco IS
        mi_id: ... ;
        cotizacion : ..
    BEGIN
        // Acepto mi id
        ACCEPT ID(id : in int) DO
            mi_id := id
        END ID

        // Aviso que estoy listo
        Tarea.Listo(mi_id)

        // Acepto la solicitud de la cotizacion
        ACCEPT Cotizacion (cot : out real) DO
            cot := cotizacion
        END Cotizacion
    END



    TASK BODY Tarea IS
        cotizaciones : array(1..20) of real
    BEGIN
        for (1 to 20) loop
            ACCEPT Listo (id : in int) DO
                idAPI : id
            END Listo

            SELECT Banco[idAPI].Cotizacion(cot)
                cotizaciones[idAPI] := cot
            OR Delay 5.00
                Cotizaciones[idAPI] := 0
            END SELECT
        end loop
    END



BEGIN
    FOR (I: 1 to 20) DO
        Banco.ID(i)
    END LOOP
END WEB

```


Capaz es mejor hacer :
```js

PROCEDURE WEB IS

    TASK Type Banco IS
        Entry Cotizacion(cot : out real)
    END Banco
    bancos : array (1..20) of Banco

    TASK Tarea

    TASK BODY Banco IS
        cotizacion : ..
    BEGIN
        // Acepto la solicitud de la cotizacion
        ACCEPT Cotizacion (cot : out real) DO
            cot := cotizacion
        END Cotizacion
    END

    TASK BODY Tarea IS
        cotizaciones : array(1..20) of real
    BEGIN
        for (1 to 20) loop
            SELECT Banco[idAPI].Cotizacion(cot)
                cotizaciones[idAPI] := cot
            OR Delay 5.00
                Cotizaciones[idAPI] := 0
            END SELECT
        end loop
    END

BEGIN
 null
END WEB

```