## Explicación practica semaforos


Declaraciones

```python
sem mutex = 1 // Semaforo normal
sem espera[5] = ([5]1) // Arreglo de semaforos
```

#### Ejercicio 4
En una empresa de genetica hay N clientes que envian secuencias de ADN para que sean analizadas y esperan para poder continuar. Para resolvdr estos analisi la empresa cuenta con un servidor que resuelve los pedidos de acuerdo con la orden de llegada

Procesos: Clientes(envia ADN), Servidor(consumidor)



No es necesario preguntar por la cola vacia, ya que si mi consumidor despierta es porque la cola tiene un dato que ya pasó por el productor. El consumidor debe quedarse dormido a que el productor le avise que hay algo. Una vez arranca el servidor podemos ver como se invierten los roles y el productor pasa a ser consumidor esperando la respuesta del cliente.

```python
sem mutex = 1
sem pedidos = 0

int resultados[N]
sem espera[N] = ([N] 0)

cola c
```


```python
Process cliente [id: 0..n-1]{
    S
    while true {
        P(mutex)
        push(C,(id,S))
        V(mutex)
        V(pedidos)

        P(espera[id])
    }
}
```


```python
Process servidor {
    secuencia sec, int aux
    while true{
        P(peidods)
        P(mutex)
        pop ( C(aux,sec))
        V(mutex)

        resultados[aux] = resolver(sec)
        V(espera[aux])

    }
}
```



#### Ejercicio 5
En una empresa de genética hay N clientes que envían secuencias de ADN para
que sean analizadas y esperan los resultados para poder continuar. Para resolver
estos análisis la empresa cuenta con 2 servidores que van alternando su uso para
no exigirlos de más (en todo momento uno está trabajando y el otro
descansando); cada 5 horas cambia en servidor con el que se trabaja. El servidor
que está trabajando, toma un pedido (de a uno de acuerdo al orden de llegada de
los mismos), lo resuelve y devuelve el resultado al cliente correspondiente.
Cuando terminan las 5 horas se intercambian los servidores que atienden los
pedidos. Si al terminar las 5 horas el servidor se encuentre atendiendo un
pedido, lo termina y luego se intercambian los servidores.


El consumidor(cliente) me queda igual, tengo que tener dos servidores y un proceso reloj el cual tomará el tiempo



```python
sem mutex = 1
sem pedidos = 0
sem espera[N] = ([N] 0)
sem inicio = 0

int resultados[N]

turno[2] = (1,0)

bool finTiempo = false

cola c
```

```python
Process cliente [id: 0..n-1]{
    S
    while true {
        P(mutex)
        push(C,(id,S))
        V(mutex)
        V(pedidos)

        P(espera[id])
    }
}
```


```python
Process servidor[id:0..1] {
    secuencia sec, int aux, bool ok
    while true{
        P(turno[id])
        finTiempo = false
        V(inicio)
        ok = true

        while(ok){
            P(pediods)
            if ( finTiempo){
                ok = false
                V(turno[1-id])
            } else{

                P(mutex)
                pop ( C(aux,sec))
                V(mutex)

                resultados[aux] = resolver(sec)
                V(espera[aux])

            }
        }

    }
}
```

```python
Process reloj{
    while true {
        p(Inicio)
        delay(5 hs)
        finTiempo = true
        V(pedidos)
    }
}

```



#### Ejercicio 6
