# Semaforos

## Bolsa limitada

```java
int cant = 0; // cant de caramelos comidos
sem mutex = 1;
Process Chico[id: 0..C-1]
{ P(mutex); // bloqueo para consultar cuantos caramelos quedan
while (cant < N){
     -- tomar caramelo
     cant = cant + 1; // saco el caramelo
     V(mutex); // libero cant
     -- comer caramelo
     P(mutex); // me quedo esperando, o bloqueo cant para tomar el proximo
}
V(mutex); // al no quedar más libero
}
```


## Barrera con coordinador

```java
int contador = 0; bool seguir = true;
sem mutex = 1, espera_abuela = 0, barrera = 0, espera_chicos[C] = ([C] 0), listo = 0;
```
```java
Process Chico[id: 0..C-1]{
    int i;
    P(mutex); // Bloqueo contador
    contador = contador + 1;
    if (contador == C){
        for i = 1..C → V(barrera) // Si soy el ultimo empiezo a liberar a los pibe;
        V(espera_abuela); // Despierto a la abu
    }
    V(mutex); // Libero el contador
    P(barrera); // Consumo 1 de barrera, si todavia no llegamos todos barrera esta en 0 por lo que me quedo esperando
    P(espera_chicos[id]); // Espero que me llame la abuela
    while (seguir){ 
        --tomar caramelo
        V(listo); // Le aviso a la abuela que ya agarre caramelo
        --comer caramelo
        P(espera_chicos[id]); // Espero que me llame nuevamente.
    }
}
```

```java
Process Abuela{
    int i, aux;
    P(espera_abuela); // Espero que me avisen que llegaron todos
    for i = 1..N{
        aux = (rand mod C);
        V(espera_chicos[aux]); // Le aviso a un chico que se despierte
        P(listo); // Espero que me avise que ya agarro el caramelo
    }
    seguir = false; // Pongo el seguir en false por el while
    for aux = 0..C-1 → V(espera_chicos[aux]); // Despierto a todos por si alguno no fue levantado nuevamente y corte su ejecucion por el false de seguir en el while
} 
```

### Si los chicos no le tienen que avisar a la abuela, la abuela deberia hacer un for consumiendo las llegadas de los chicos



## Cliente / Un servidor

```java
sem mutex = 1, pedidos = 0, espera[N] = ([N] 0);
int resultados[N];
cola C;
Process Cliente[id: 0..N-1]{
    secuencia S;
    while (true){
        --generar secuencia S
        P(mutex); // Bloqueo la cola
        push(C, (id, S));
        V(mutex); // Libero la cola
        V(pedidos); // Despierto al servidor
        P(espera[id]); // Espero resultado
        --ver resultado de resultados[id] // Agarrp el resultado
    }
}

Process Servidor{
    secuencia sec; int aux;
    while (true){
        P(pedidos); // Espero que haya un pedido
        P(mutex); // Bloqueo la cola para sacar
        pop(C, (aux, sec));
        V(mutex); // Libero
        resultados[aux] = resolver(sec); // Meto resultado
        V(espera[aux]); // Le aviso que tiene el resultado(por id)
    }
}

```


## Cliente / Dos servidores pero alternando entre servidor cada 5hs

Lo unico que cambia seria tener un proceso reloj y preguntar en cada ejecucion del while si tengo que seguir

```java

bool sigo[2] = ([1]false)
sem arrancar = 1
sem inicio = 0


Process Servidor[id:0..1]{

    secuencia sec; int aux;
    while (true){
        P(sigo)
        if(sigo[id]){
            V(sigo)
            V(inicio) // le aviso al reloj que arranco
            P(pedidos); // Espero que haya un pedido
            P(mutex); // Bloqueo la cola para sacar
            pop(C, (aux, sec));
            V(mutex); // Libero
            resultados[aux] = resolver(sec); // Meto resultado
            V(espera[aux]); // Le aviso que tiene el resultado(por id)    
        }
    }
}


Process Reloj{
    while(true){
        P(inicio) // Espero que arranque el proceso
        delay(5hs) // espero 5 hs
        P(arrancar) // bloqueo la variable sigo 
        for i = 1..2{
            sigo[i] = not sigo[i]
        }
        V(arracar) // Desbloqueo sigo
    }
}

```



## Passing the baton

```java
cola c;
sem mutex = 1, espera[30] = ([30] 0);
boolean libre = true;

Process Escalador[id: 0..29]
    { int aux;
    -- llega al paso
    P (mutex);
    if (libre) { libre = false; V (mutex); }
    else {  push (C, id);
            V (mutex);
            P (espera[id]); };
    //Usa el paso con Exclusión Mutua
    P (mutex);
    if (empty (C)) libre = true
    else { pop (C, aux); V (espera[aux]); };
    V (mutex);
}
```

## Ticket

```java
sem mutex = 1, espera[30] = ([30] 0);
int ticket = 0;
Process Escalador[id: 0..29]
    { int miTurno;
    -- llega al paso
    P (mutex);
    miTurno = ticket;
    ticket++;
    V (mutex);
    if (miTurno > 0) P (espera[miTurno]);
    //Usa el paso con Exclusión Mutua
    if (miTurno < 29) V (espera[miTurno+1]);
}
```


# Monitores

## Orden de llegada

```java
Process Persona [id: 0..N-1]
{ Cajero.Pasar ();
 UsarCajero();
 Cajero.Salir();
}

Monitor Cajero{
    bool libre = true;
    cond cola;
    int esperando = 0;

    Procedure Pasar ()
    { if (not libre) { esperando ++;
    wait (cola);
    }
    else libre = false;
    }
    Procedure Salir ()
    { if (esperando > 0 ) { esperando --;
    signal (cola);
    }
    else libre = true;
    }
}

```

## Por prioridad
```java

Process Persona [id: 0..N-1]
{ bool edad = ….;

 Cajero.Pasar (id, edad);
 UsarCajero();
 Cajero.Salir();
}


Monitor Cajero{
    bool libre = true;
    cond espera[N];
    int idAux, esperando = 0; colaOrdenada fila;

    Procedure Pasar (idP, edad: in int)
    { if (not libre) { push(fila, idP, edad);
    esperando ++;
    wait (espera[idP]); // Espero que me levanten a mi unicamente
    }
    else libre = false;
    }
    Procedure Salir ()
    { if (esperando > 0 ) { esperando --;
    pop (fila, idAux);
    signal (espera[idAux]);
    }
    else libre = true;
    }
}

```

## Cliente - Servidor(ejercicio 4) que dios se apiade

```java

Process Cliente[id: 0..N-1]
{ int idE;
 text papel, res;

 Banco.llegada(idE);
 Escritorio[idE].atención(papel, res);
}

Process Empleado[id: 0..2]
{ text datos;

 while (true)
 { Banco.próximo(id);
 Escritorio[id].esperarDatos(datos);
 res = resolver solicitud en base a datos
 Escritorio[id].enviarResultado(res);
 }
}


Monitor Banco {
 cola elibres;
 cond esperaC;
 int esperando = 0, cantLibres = 0;

 Procedure Llegada(idE: out int)
 { if (cantLibres == 0)
 { esperando ++;
 wait (esperaC);
 }
 else cantLibres--;
 pop(elibres, idE);
 }
Procedure Próximo(idE: in int)
 { push(elibres, idE);
 if (esperando > 0 )
 { esperando --;
 signal (esperaC);
 }
 else cantLibres++;
 }
}



Monitor Escritorio[id: 0..2] {
 cond vcCliente, vcEmpleado;
 text datos, resultados;
 boolean listo = false;

 Procedure Atención(D: in text; R: out text)
 { datos = D;
 listo = true;
 signal (vcEmpleado);
 wait (vcCliente);
 R = resultados;
 signal (vcEmpleado);
 }
Procedure Esperardatos(D: out text)
 { if (not listo) wait (vcEmpleado);
 D = datos;
 }
Procedure EnviarResultados(R: in text)
 { resultados = R;
 signal (vcCliente);
 wait (vcEmpleado);
 listo = false;
 }
}


```


## Barreras

```java
Process Jugador[id: 0..21]
{ Cancha.llegada(); 
}

Process Partido
{ Cancha. Iniciar();
 delay (90minutos); //se juega el partido
 Cancha.Terminar();
}

Monitor Cancha
{ int cant = 0;
 cond espera, inicio;
 Procedure llegada ()
 { cant ++;
 if (cant == 22) signal (inicio);
 wait (espera);
 }
 Procedure Iniciar ()
 { if (cant < 22) wait (inicio);
 }
 Procedure Terminar ()
 { signal_all(espera);
 }
}


```


## Cliente servidor pero con un solo servidor

```java
Process Cliente [id: 0..N-1]
{ text S, res;
while (true)
{ --generar secuencia S
Admin.Pedido(id, S, res);
}
}

Process Servidor
{ text sec, res;
int aux;
while (true)
{ Admin.Sig(aux, sec);
res = AnalizarSec(sec);
Admin.Resultado(aux, res);
}
}


Monitor Admin {
 Cola C;
 Cond espera;
 Cond HayPedido;
 text res[N];
 Procedure Pedido (IdC: in int; S: in text; R: out text)
 { push (C, (idC,S) );
 signal (HayPedido);
 wait (espera);
 R = res[idC];
 }
 Procedure Sig (IdC: out int; S: out text)
 { if (empty (C)) wait (HayPedido);
 pop (C, (IdC, S));
 }
 Procedure Resultado (IdC: in int; R: in text)
 { res[IdC] = R;
 signal (espera);
 }
}
```


## Cliente servidor pero con varios servidores

```java
Process Cliente [id: 0..N-1]
{ text S, res;
while (true)
{ --generar secuencia S
Admin.Pedido(id, S, res);
}
}

Process Servidor [id: 0..1]
{ text sec, res;
int aux;
while (true)
{ Admin.Sig(aux, sec);
res = AnalizarSec(sec);
Admin.Resultado(aux, res);
}
}


Monitor Admin {
 Cola C;
 Cond espera[N]; // Lo unico que cambia es esto
 Cond HayPedido;
 text res[N];
 Procedure Pedido (IdC: in int; S: in text; R: out text)
 { push (C, (IdC,S) );
 signal (HayPedido);
 wait (espera[IdC]); // Aca
 R = res[IdC];
 }
 Procedure Sig (IdC: out int; S: out text)
 { while (empty (C)) wait (HayPedido);
 pop (C, (IdC, S));
 }
 Procedure Resultado (IdC: in int; R: in text)
 { res[IdC] = R;
 signal (espera[IdC]); // Y aca
 }
}

```
