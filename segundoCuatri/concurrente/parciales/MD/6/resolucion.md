# PMA

```js

chan pedidos(pedido: text; idCli : int)
chan pedidosEmp[V](pedido : text; idCli : int; okay : bool)
chan entrega (repuesto : repuesto)

Process clientes[1..C]{
    send pedidos(pedido,id)
    recieve entrega[id](repuesto)
}


Process Admin{

    while true{
        recieve listo(idEmp)
        if (!pedidos.isEmpty){
            accept pedidos(pedido,idCli)
            send pedidosEmp[idEmp](pedido,idCli,true)
        } else {
            send pedidosEmp[idEmp](null,null,false)
        }
    }
}

Process Vendedor[id:1..V]{

    while true{
        // Aviso que estoy listo
        send listo(id)
        // Recibo pedido
        recieve aceptarPedido[idEmp](pedido,idCli,okay)
        // Si habia pedido entro el repuesto, si no controlo el stock
        if(okay){
            repuesto = pedido.GetRepuesto()
            send entrega[idCli](repuesto)
        } else {
            // controlar stock
            stock.controlar()
        }
    }
}


```




# ADA

```js

Procedure Banco IS

    TASK Empleado IS
        Entry AtencionPremium(pago : in int ; comprobante : out text)
        Entry AtencionRegular(pago : in int ; comprobante : out text)
    END Empleado

    TASK TYPE ClienteRegular
    clientesRegulares : array (1..R) of ClienteRegular

    TASK TYPE ClientePremium
    clientesPremiums : array (1..P) of ClientePremium

    TASK BODY ClienteRegular IS
    pago : ...
    comprobante : ...
    BEGIN
        SELECT Empleado.AtencionPremium(pago,comprobante)
        OR DELAY 1800.00
            NULL
    END ClienteRegular

    TASK BODY ClientePremium IS
    pago : ...
    comprobante : ...
    BEGIN
        Empleado.AtencionPremium(pago,comprobante)
    END ClientePremium

    TASK BODY Empleado IS

    BEGIN
        FOR (I : 1 to R+P) LOOP
            SELECT 
                ACCEPT AtencionPremium(pago : in int; comprobante : out text) DO
                    comprobante := generarComprobante(pago)
                END AtencionPremium
            OR WHEN (AtencionPremium´Count = 0) ->
                ACCEPT AtencionRegular(pago : in int; comprobante : out text) DO
                    comprobante := generarComprobante(pago)
                END AtencionRegular
            END SELECT
        END LOOP
    END
```