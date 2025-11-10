# Practica de repaso mierda

# PMS - PMA

1- En una oficina existen 100 empleados que envían documentos para imprimir en 5 impresoras
compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera
impresora que se encuentre libre:

a- Implemente un programa que permita resolver el problema anterior usando PMA.

b- Resuelva el mismo problema anterior pero ahora usando PMS.

a-
```js
chan imprimir (text)


Process Impresora[id:0..4]{
    documento = text

    while true {
        // Agarro el documento, esto siempre va a agarrar una impresora libre.
        receive imprimir(documento)
        hacerImpresion(documento)
    }
}

Process Empleado[id:0..99]{
    documento = ...

    documento = generarDocumento()
    // Le mando el documento a la cola de impresion y siempre va a ser en orden y lo va a tomar una impresora libre.
    send imprimir(documento)
}
```



b-
```js

Process Admin{
    cola documentos

    while true {
        // Aca lo que hago es, no deterministicamente voy agarrando documentos y pusheandolos a la cola o si la cola no esta vacia   y la impresora me aviso que estaba lista se lo mando a esa impresora para imprimir.
        do Empleado[*]?EnviarDocumento(documento) -> documentos.push(documento);
        [] !documentos.isEmpty(); Impresora[*]?Listo(idImpresora) ->
            documento = documentos.pop()
            Impresora[idImpresora]!Imprimir(documento)
        od
    }
}

Process Impresora[id: 0..4]{
    while true {
        Admin!Listo(id)
        Admin?Imprimir(documento)
        hacerImpresion(documento)
    }
}

Process Empleado[id:0..99]{
    
    documento = generarDocumento()
    Admin!EnviarDocumento(documento)

}

```



2) Resolver el siguiente problema con PMS. En la estación de trenes hay una terminal de SUBE que
debe ser usada por P personas de acuerdo con el orden de llegada. Cuando la persona accede a la
terminal, la usa y luego se retira para dejar al siguiente. Nota: cada Persona usa sólo una vez la
terminal. 


Bueno, es un passing the baton de manual, a ver como lo hacemos

```js

Process Admin{
    cola esperando
    ocupado = false

    // Si NO esta ocupado, y alguien me pide de usar la sube entonces dejo pasar a la persona que llego.
    do !ocupado; Persona[*]UsarSube(idPersona) ->
        ocupado = true;
        Persona[idPersona]!Permiso(sube)
    // Si esta ocupado, pero me piden de usar la sube entonces lo pongo en espera
    [] ocupado; Persona[*]UsarSube(idPersona) -> esperando.push(idPersona)

    // Si me dicen que terminan de usar la sube entonces hago el passing the baton, se lo paso al siguiente sin sacar el ocupado.
    [] Persona[*]DejarSube -> 
        if (!esperando.isEmpty()){
            siguiente = esperando.pop()
            Persona[siguiente]!Permiso(sube)
        } else {
            ocupado = true
        }
    [] // para que no termine
    od
}

Process Persona[0..P-1]{

    Admin!UsarSube(id)
    Admin?Permiso(sube)
    // Usar Sube
    sube.usar()
    // Me void
    Admin!DejarSube()
}
```



3) Resolver el siguiente problema con PMA. En un negocio de cobros digitales hay P personas que
deben pasar por la única caja de cobros para realizar el pago de sus boletas. Las personas son
atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos
de 5 boletas de los que pagan más. Adicionalmente, las personas embarazadas tienen prioridad sobre
los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les
devuelve el vuelto y los recibos de pago.





```js

chan prioridad (boletas,pago)
chan sinPrioridad(boletas,pago)
chan embarazadas(boletas,pago)
chan levantarse

Process Caja {

    while true {

        recieve levantarse()

        if !embarazadas.isEmpty(){
            recieve embarazadas(boletas,pago)
        }
        elif ( !prioridad.isEmpty() ) {
            recieve prioridad(boletas,pago)
        } else {
            recieve sinPrioridad(boletas,pago)
        }


        generarRecibo(boletas,pago)
        send retorno(recibos,vuelto)
    }


}


Process Persona [id:1..P]{

    boletas = ...
    soy_embarazada = ...


    if(soy_embarazada) {
        send embarazadas(boletas,pago)
    } elif (cantBoletas < 5) {
        send prioridad(boletas,pago)
    } else {
        send sinPrioridad(boletas,pago)
    }

    send levantarse()
    recieve retorno(recibos,vuelto)

}
```



# ADA


1) Resolver el siguiente problema. La página web del Banco Central exhibe las diferentes cotizaciones
del dólar oficial de 20 bancos del país, tanto para la compra como para la venta. Existe una tarea
programada que se ocupa de actualizar la página en forma periódica y para ello consulta la cotización
de cada uno de los 20 bancos. Cada banco dispone de una API, cuya única función es procesar las
solicitudes de aplicaciones externas. La tarea programada consulta de a una API por vez, esperando
a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, entonces se mostrará
vacía la información de ese banco.



```js

Procedure Banco IS

    TASK TYPE API IS
        Entry Consultar(respuesta : OUT text);
    END API

    apis : array (1..20) of API;

    TASK BODY API IS
        respuesta_api : ...
    BEGIN
        ACCEPT Consultar(respuesta: out text) DO
            respuesta := respuesta_api
        END Consultar
    END


    TASK Actualizar;

    TASK BODY ACTUALIZAR IS
        respuestas : array(1..20) of text
        respuesta : text
    BEGIN

        for (i: 1 to 20) LOOP
            SELECT apis[i].Consultar(respuesta)
                respuesta[i] = respuesta
            OR DELAY 5.00 
                respuesta[i] = null; 
            END SELECT
        END LOOP

    END ACTUALIZAR

BEGIN
    NULL
END
```

2) Resolver el siguiente problema. En un negocio de cobros digitales hay P personas que deben pasar
por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo
con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que
pagan más. Adicionalmente, las personas ancianas tienen prioridad sobre los dos casos anteriores.
Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los
recibos de pago. 

```js

PROCEDURE NEGOCIO IS

    TASK CAJA IS
        Entry Llegada()
        Entry Anciano(boletas : in text ; pago : in real ; comprobante : OUT text ; vuelto : OUT real)
        Entry prioridad(boletas : in text ; pago : in real ; comprobante : OUT text ; vuelto : OUT real)
        Entry sin_prioridad(boletas : in text ; pago : in real ; comprobante : OUT text ; vuelto : OUT real)
    END CAJA


    TASK BODY CAJA IS

    boletas : text
    pago : real
    vuelto : real
    comprobante : text
    BEGIN

        LOOP 
                SELECT ACCEPT Anciano(boletas : in text ; pago : in real ; comprobante : OUT text ; vuelto : OUT real) DO
                    comprobante := generarComprobante(boletas,pago)
                    vuelto := calcularVuelto(boletas,pago)
                END Anciano

                OR WHEN (Anciano´Count = 0)
                    ACCEPT Prioridad(boletas : in text ; pago : in real ; id : in int) DO
                        comprobante := generarComprobante(boletas,pago)
                        vuelto := calcularVuelto(boletas,pago)
                    END Prioridad
                OR WHEN (Anciano´Count = 0 AND Prioridad´Count = 0)
                    ACCEPT Sin_Prioridad(boletas : in text ; pago : in real ; id : in int) DO
                        comprobante := generarComprobante(boletas,pago)
                        vuelto := calcularVuelto(boletas,pago)
                    END Sin_Prioridad;
                END SELECT
        END LOOP

    END CAJA


    TASK TYPE PERSONA;
    
    Personas : array (1..P) of Persona


    TASK BODY PERSONA IS
        is_anciano := ...
        menos_de_cinco := ...
        boletas : text
        pago : real 
        vuelto : real
        recibos : text
    BEGIN

        ACCEPT ID (id : in int) DO
            id : id;
        END ID

        if (is_anciano) {
            Caja.Anciano(boletas,pago,recibos,vuelto)
        } elif (menos_de_cinco) {
            Caja.Prioridad(boletas,pago,recibos,vuelto)
        } else {
            Caja.Sin_Prioridad(boletas,pago,recibos,vuelto)
        }
    END PERSONA




BEGIN 
    NULL
END NEGOCIO

```


3) Resolver el siguiente problema. La oficina central de una empresa de venta de indumentaria debe
calcular cuántas veces fue vendido cada uno de los artículos de su catálogo. La empresa se compone
de 100 sucursales y cada una de ellas maneja su propia base de datos de ventas. La oficina central
cuenta con una herramienta que funciona de la siguiente manera: ante la consulta realizada para un
artículo determinado, la herramienta envía el identificador del artículo a las sucursales, para que cada
una calcule cuántas veces fue vendido en ella. Al final del procesamiento, la herramienta debe
conocer cuántas veces fue vendido en total, considerando todas las sucursales. Cuando ha terminado
de procesar un artículo comienza con el siguiente (suponga que la herramienta tiene una función
generarArtículo() que retorna el siguiente ID a consultar). Nota: maximizar la concurrencia. Existe
una función ObtenerVentas(ID) que retorna la cantidad de veces que fue vendido el artículo con
identificador ID en la base de la sucursal que la llama.


```js

PROCEDURE OFICINA IS

    TASK TYPE Sucursal
    sucursales : array (1..100) of Sucursal

    TASK Central IS
        Entry QuieroID(id : out int)
        Entry Total(total : in int)
    END Central


    TASK BODY Sucursal IS
    BD : base_datos
    total,id : int
    BEGIN
        LOOP
            // Pido el id
            Central.QuieroID(id)
            // Consigo el total
            total := BD.ObtenerVentas(id)
            // Se lo mando
            Central.Total(total)
            
            // Espero que me acepte el mensaje para poder continuar
            Central.Continuar

        END LOOP
    END

    TASK BODY CENTRAL IS 
    total_sucursales,idArticulo : int
    BEGIN
        LOOP

            idArticulo := generarArticulo();
            total_sucursales := 0
            for (i : 1 to 200 ) LOOP
                // Aca lo que hago es que los que me pidieron el id se los paso y si alguien me manda el total lo voy sumando (eligiendo entre las dos no deterministicamente)
                SELECT 
                    ACCEPT QuieroID(id : out int) DO
                        id := idArticulo;
                    END QuieroID
                OR 
                    ACCEPT Total (total : in int) DO
                        total_sucursales := total_sucursales + total
                    END Total
                END SELECT
            END LOOP
            
            For (1 to 100) LOOP
                ACCEPT Continuar
            END LOOP

        END LOOP
    END
```