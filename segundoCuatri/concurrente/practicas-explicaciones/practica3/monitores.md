# Practica Monitores:

## Ejercicio 1

Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso
para pasar por el puente, cruza por el mismo y luego sigue su camino. 

a. ¿El código funciona correctamente?
Justifique su respuesta.

b. ¿Se podría simplificar el programa? ¿Sin
monitor? ¿Menos procedimientos? ¿Sin
variable condition? En caso afirmativo,
rescriba el código.

c. ¿La solución original respeta el orden de
llegada de los vehículos? Si rescribió el código
en el punto b), ¿esa solución respeta el orden
de llegada?

## Respuesta: 
### a. No funciona
- primero lo que vemos es como en el proceso al llamar a entrar al puente se envia el id como parametro cuando en el procedure no espera parametros. 


### b. Si se podria:

```python
Monitor Puente{
    Procedure CruzarPuente(){
        "el auto cruza por el puente"
    }
}
```

```python
Process auto[a:1..M]{
    Puente.CruzarPuente()
}
```

### C. La solucion original ni la hecha en el inciso B cumplen con el orden de llegada. 
### Solucion que cumple orden:

```java
Monitor Puente{
    bool libre = true;
    cond cola;
    int esperando = 0;

    Procedure Entrar(){
        if (not libre){
            esperando ++
            wait(cola)
        } else{
            libre = false
        }
    }

    Procedure Salir(){
        if(esperando > 0){
            esperando --
            signal(cola)
        } else {
            libre = true
        }
    }

}
```


```java
Process Auto[id:1..M]{
    Puente.Entrar()
    "Pasando por el puente"
    Puente.Salir()
}
```


## Ejercicio 2:
2- Existen N procesos que deben leer información de una base de datos, la cual es administrada
por un motor que admite una cantidad limitada de consultas simultáneas.

a) Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones
serán necesarios/convenientes para resolverlo.

b) Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de
base de datos puede atender a lo sumo 5 consultas de lectura simultáneas.

## Respuesta:
a- Voy a necesitar un monitor DataBase con los procedure acceder y salir, los procesos procesos


b- 
```java
Monitor DataBase{
    int espacios = 5;
    cond cola;

    Procedure Acceder(){
        while(espacios == 0){
            wait(cola)
        }
            espacios--
    }
    
    Procedure Salir(){
        espacios++
        signal(cola)
    }
}
```

```java
Process proceso[id:1..N] {
    DataBase.Acceder()
    //Leer info
    DataBase.Salir()
}
```


## Ejercicio 4
Existen N vehículos que deben pasar por un puente de acuerdo con el orden de llegada.
Considere que el puente no soporta más de 50000kg y que cada vehículo cuenta con su propio
peso (ningún vehículo supera el peso soportado por el puente). 

## Respuesta
```java
Monitor Puente{
    bool libre = true;
	int esperando = 0;
    bool esperandoPeso = false;
	double pesoTotal = 0;
	cond esperaAutos;
	cond pesoEspera;

    Procedure Pasar (peso: in double) {
        if (not libre) { // Si hay alguien esperando su peso no paso y me duermo en la cola de espera de autos
            esperando++;
            wait (esperaAutos);
        } 
        else libre = false;
        while (pesoTotal + peso > 500000) { // Si peso mas de lo que le queda libre al puente me duermo en una cola aparte
            esperandoPeso = true
            wait (pesoEspera);
        }
        pesoTotal += peso;
        esperandoPeso = false
        if (esperando > 0) {
            esperando–;
            signal(esperaAutos);
        } 
        else libre = true;
    }

    Procedure Salir (peso: in double) {
        pesoTotal -= peso;
        if (esperandoPeso){ // Si hay alguien esperando por su peso lo levanto a él.
            signal (pesoEspera);
        }
    }
}
```

```java
Process Auto[id:1..N]{
    int peso;
    Puente.Pasar(peso)
    "Pasando por el puente"
    Puente.Salir(peso)
}
```


## Ejercicio 5

5- En un corralón de materiales se deben atender a N clientes de acuerdo con el orden de llegada.
Cuando un cliente es llamado para ser atendido, entrega una lista con los productos que
comprará, y espera a que alguno de los empleados le entregue el comprobante de la compra
realizada.

a) Resuelva considerando que el corralón tiene un único empleado.

b) Resuelva considerando que el corralón tiene E empleados (E > 1). Los empleados no
deben terminar su ejecución.

c) Modifique la solución (b) considerando que los empleados deben terminar su ejecución
cuando se hayan atendido todos los clientes.

## Respuesta A 

```java
Monitor Admin{
    Cola c;
    Cond espera;
    Cond HayPedido;,
    text comprobantes[N]

    Procedure Pedido(compra: IN str; Comprobante: OUT str ; IDCli: in int){
        push(C,(IDCli,compra)) // Mando el id del cliente y la compra a la cola
        signal(HayPedido) // Despierto al empleado avisando que hay pedido
        wait(espera) // Me duermo hasta que el empleado termine
        comprobante = comprobantes[IDCli] // Agarro el comprobante y me voy
    }

    Procedure Sig(IDCli: OUT int; compra: OUT text){
        if(empty(C)): wait(HayPedido) // Espero a que haya un pedido
        pop(C(IDCli,compra)) //Como hay pedido tomo el id y la compra de la cola
    } 

    Procedure Resultado(IDcli: IN int; comprobanteGenerado: IN text){
        comprobantes[IDCli] = comprobanteGenerado // Guardo en el arreglo de comprobantes el comprobante del cliente.
        signal(espera) // Despierto al cliente que esta esperando su pedido en espera
    }
    
}
```

```java
Process Cliente[0..N-1]{
    text lista_compra, comprobante;
    Admin.Pedido(lista_compra,comprobante,id);


}


Process Empleado{
    text compras,comprobante
    int IDCli

    while(true){
        Admin.Sig(IDCli,compras) // Recibo id del cliente y sus compras
        comprobante = GenerarComprobante(compras) // Genero su comprobante
        Admin.Resultado(IDCli,comprobante) // Envio el resultado y su id para levantarlo.

        
    }
}

```



## Respuesta B
```java
Monitor Admin{
    Cola c;
    Cond espera[N];
    Cond HayPedido;,
    text comprobantes[N]

    Procedure Pedido(compra: IN str; Comprobante: OUT str ; IDCli: in int){
        push(C,(IDCli,compra)) // Mando el id del cliente y la compra a la cola
        signal(HayPedido) // Despierto al empleado avisando que hay pedido
        wait(espera[IDCli]) // Me duermo hasta que el empleado termine
        comprobante = comprobantes[IDCli] // Agarro el comprobante y me voy
    }

    Procedure Sig(IDCli: OUT int; compra: OUT text){
        if(empty(C)): wait(HayPedido) // Espero a que haya un pedido
        pop(C(IDCli,compra)) //Como hay pedido tomo el id y la compra de la cola
    } 

    Procedure Resultado(IDcli: IN int; comprobanteGenerado: IN text){
        comprobantes[IDCli] = comprobanteGenerado // Guardo en el arreglo de comprobantes el comprobante del cliente.
        signal(espera[IDcli]) // Despierto al cliente que esta esperando su pedido en espera
    }
    
}
```

```java
Process Cliente[0..N-1]{
    text lista_compra, comprobante;
    Admin.Pedido(lista_compra,comprobante,id);


}


Process Empleado[0..E-1]{
    text compras,comprobante
    int IDCli

    while(true){
        Admin.Sig(IDCli,compras) // Recibo id del cliente y sus compras
        comprobante = GenerarComprobante(compras) // Genero su comprobante
        Admin.Resultado(IDCli,comprobante) // Envio el resultado y su id para levantarlo.

        
    }
}

```

## Respuesta C
```java
Monitor Admin{
    Cola c;
    Cond espera[N];
    Cond HayPedido;,
    text comprobantes[N]
    int ClientesRestantes = N;

    Procedure Pedido(compra: IN str; Comprobante: OUT str ; IDCli: in int){
        clientesRestantes--
        push(C,(IDCli,compra)) // Mando el id del cliente y la compra a la cola
        signal(HayPedido) // Despierto al empleado avisando que hay pedido
        wait(espera[IDCli]) // Me duermo hasta que el empleado termine
        comprobante = comprobantes[IDCli] // Agarro el comprobante y me voy
    }

    Procedure Sig(IDCli: OUT int; compra: OUT text; Seguir: OUT bool){
        if(clientesRestantes>0){
            if(empty(C)): wait(HayPedido) // Espero a que haya un pedido
            pop(C(IDCli,compra)) //Como hay pedido tomo el id y la compra de la cola
        } else {
            seguir = false
        }
    } 

    Procedure Resultado(IDcli: IN int; comprobanteGenerado: IN text){
        comprobantes[IDCli] = comprobanteGenerado // Guardo en el arreglo de comprobantes el comprobante del cliente.
        signal(espera[IDcli]) // Despierto al cliente que esta esperando su pedido en espera
    }
    
}
```

```java
Process Cliente[0..N-1]{
    text lista_compra, comprobante;
    Admin.Pedido(lista_compra,comprobante,id);


}


Process Empleado[0..E-1]{
    text compras,comprobante
    int IDCli
    bool seguir = true

    while(seguir){
        Admin.Sig(IDCli,compras,seguir) // Recibo id del cliente y sus compras
        if (seguir){
            comprobante = GenerarComprobante(compras) // Genero su comprobante
            Admin.Resultado(IDCli,comprobante) // Envio el resultado y su id para levantarlo.
        }
    }
}

```



## Ejercicio 6
6- Existe una comisión de 50 alumnos que deben realizar tareas de a pares, las cuales son
corregidas por un JTP. Cuando los alumnos llegan, forman una fila. Una vez que están todos
en fila, el JTP les asigna un número de grupo a cada uno. Para ello, suponga que existe una
función AsignarNroGrupo() que retorna un número “aleatorio” del 1 al 25. Cuando un alumno ha recibido su número de grupo, comienza a realizar su tarea. Al terminarla, el alumno le avisa
al JTP y espera por su nota. Cuando los dos alumnos del grupo completaron la tarea, el JTP
les asigna un puntaje (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así
sucesivamente hasta el último que tendrá nota 1). Nota: el JTP no guarda el número de grupo
que le asigna a cada alumno




## Respuesta:

```java
Process Alumno[id:0..50]{
    int nro;
    int nota;

    Tarea.EsperarFila(id);
    Tarea.RecibirNro(id,nro);
    //Realizar Tarea
    Tarea.ObtenerNota(nro,nota);

}

Process JTP{
    int i,nroGrupo;
    int puntaje = 25;
    int tareasContador[25] = ([25]0)


    Tarea.EsperarAlumnos();

    For I: 1..50 {
        Tarea.AsignarNumero(AsignarNroGrupo()) // Le paso la funcion que genera el numero aleatorio.
    }

    For I: 1..50{
        Tarea.Recibir(nroGrupo); // Recibo El numero del grupo del alumno que entrega la tarea.
        tareasContador[nroGrupo] ++; // Sumo uno al contador
        if (tareasContador[nroGrupo] == 2){ // Si el contador es dos en ese grupo significa que ya entregaron ambos por lo que puedo corregir
            Tarea.Corregir(nroGrupo,puntaje)
            puntaje--
        }
    }

}

```



```java
Monitor Tarea{


    Cond profe;
    Cond grupoEspera[25]
    Int grupoNotas[25] = ([25] -1)
    Int nroAsignado[50] = ([50] -1)
    Cola fila;
    int cantidad;

    Procedure EsperarFila(ID: IN int){
        cant++
        push(fila(ID))
        if (cant == 50){ // Si soy el ultimo en la fila soy el nro 50 por lo que levanto al profesor
            signal(profesor)
        }
    }

    Procedure EsperarAlumnos(){
        if (Cant < 50){ // Si no llegaron los 50 alumnos me duermo hasta que me levanten
            wait(profe);
        }
    }

    Procedure RecibirNumero(ID: IN int; Nro: OUT int){
        wait(esperarNumero[ID]); // Espero a que sea mi turno en la fila
        Nro = nroAsignado[ID];
    }

    Procedure AsignarNumero(nro: IN int){
        ID = pop(fila) // Voy sacando de la fila por orden de llegada y asignando numeros
        nroAsignado[ID] = nro;
    }

    Procedure RecibirNota(nro: IN int; nota: OUT int){
        wait(grupoEspera[nro]); // Espero a que este la nota de mi grupo
        nota = grupoNotas[nro];
    }

    Procedure Corregir(nroGrupo: IN int; nota: IN int){
        grupoNotas[nroGrupo] = nota; 
        signal(grupoEspera[nroGrupo]) // Despierto al grupo que se le corrigió 
    }

}

```

## Ejercicio 7

7- Se debe simular una maratón con C corredores donde en la llegada hay UNA máquina
expendedoras de agua con capacidad para 20 botellas. Además, existe un repositor encargado
de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza
la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera
su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina
sin botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas;
espera a que se haga la recarga; saca una botella y se retira. Nota: mientras se reponen las
botellas se debe permitir que otros corredores se encolen. 


```java
Process Corredor[1..C]{
    Maraton.Inicio();
    // Corre la carrera
    // Termina la carrera
    Maquina.usar()
}

Process Encargado{
    Maquina.Recargar()
}
```

```java
Monitor Maraton{

    Process Inicio(){
        if(!cantidad == C){
            wait(preparados); // Si llegue y no estan todos me duermo
        } else {
            signal_all(preparados); // Si llegue y estan todos los levanto
        }
    }
}
```

```java
Monitor Maquina{
    bool libre = false;
    cond esperandoUsar;
    cond repositor;
    cond corredorEsperando;
    int unidades = 20;
    int esperando = 0


    Procedure Usar(){
        if (!libre){
            esperando++
            wait(esperandoUsar)
        } else {
            libre = false;
            if (unidades > 0){
                unidades--
            } 
            else {
                signal(repositor)
                wait(corredorEsperando)
                unidades--
            }
            if (esperando > 0){
                signal(esperandoUsar)
            } else {
                libre = true;
            }
        }
    }
    


    Procedure Recargar(){
        if(unidades > 0){
            wait(repositor);
        } 
        unidades = 20;
        signal(corredorEsperando)
        
    }
}
```



## Ejercicio 8
8- En un entrenamiento de fútbol hay 20 jugadores que forman 4 equipos (cada jugador conoce
el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo
(han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también
esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos
juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores
se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido,
juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es
necesario que se esperen para salir).

```java

Process Jugador[Id:1..20]{
    int equipo = DarEquipo();
    int cancha;

    Equipo.Llegar(equipo,cancha);
    Cancha[cancha].Entrar();
}

```


```java
Process Partido[id:1..2]{
    Cancha[id].IniciarPartido() // Arranco el partido en la cancha 1
    delay(90min)
    Cancha[id].TerminarPartido() // Arranco el partido en la cancha 2
}
```


```java
Monitor Equipo[1..4]{
    int cant = 0;
    cond esperando;

    Procedure Llegar(cancha: OUT int){
        cant++
        if (!cant == 5){
            wait(espernado) // Espero que llegue todo mi equipo
        } 
        else{
            Admin.ConseguirCancha(cancha) // Consigo numero de cancha
            signal_all(esperando) // Despierto a todos
        }
    }
}
```

```java
Monitor Admin{
    int cancha1 = 2;

    Procedure ConseguirCancha(cancha: OUT int){
        if (cancha1 > 0){
            cancha1-- ;
            cancha = 1;
        } else{
            cancha = 2;
        }
    }
}
```


```java
Monitor Cancha[id: 1..2]{
    int cantidadJugadores
    cond esperando
    cond partido

    Procedure Entrar(){
        cantidadJugadores++
        if(!cantidadJugadores == 10){
            wait(esperando) // Si no llegaron todos me duermo
        } else {
            signal(partido); // Una vez llegaron todos arranco el partido
        }
    }    

    Procedure IniciarPartido{
        if(cantidadJugadores<10){ // Si no llegaron todos me duermo
            wait(partido)
        }
    }

    Procedure TerminarPartido{
        signal_all(esperando) 
    }
}

```