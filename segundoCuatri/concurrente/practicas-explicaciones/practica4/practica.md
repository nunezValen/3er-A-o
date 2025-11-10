# Practica 4 carajo!



if no deterministico en asincronico: 

```js
if (condicion) -> { bloque }
[] (condicion) -> { bloque }
```

do no determinisitico (se sigue ejecutando siempre y cuando haya almenos 1 verdadera y por repeticion solo ejecuta una)
```js
do (condicion) -> { bloque }
[] (condicion) -> { bloque }
```

## CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS DE PASAJE DE
MENSAJES ASINCRÓNICO (PMA):
- Los canales son compartidos por todos los procesos.
- Cada canal es una cola de mensajes; por lo tanto, el primer mensaje encolado es el
primero en ser atendido.
- Por ser PMA, el send no bloquea al emisor.
- Se puede usar la sentencia empty para saber si hay algún mensaje en el canal, pero no
se puede consultar por la cantidad de mensajes encolados.
- Se puede utilizar el if/do no determinístico donde cada opción es una condición boolena donde se puede preguntar por variables locales y/o por empty de canales.
```
 if (cond 1) -> Acciones 1;
 (cond 2) -> Acciones 2;
 ….
 (cond N) -> Acciones N;
 end if
```
- De todas las opciones cuya condición sea Verdadera elige una en forma no determinística y ejecuta las acciones correspondientes. Si ninguna es verdadera, sale del if/do sin ejecutar acción alguna.
- Se debe evitar hacer busy waiting siempre que sea posible (sólo hacerlo si no hay
otra opción).
- En todos los ejercicios el tiempo debe representarse con la función delay.


1- Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus
empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones
serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes
situaciones:

a. Existe un único empleado, el cual atiende por orden de llegada.

b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe
modificarse en la solución anterior?

c. Ídem b) pero considerando que, si no hay clientes para atender, los empleados
realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar
procesos adicionales? ¿Qué consecuencias implicaría?


### A)
```js
chan atencion(int)

Process Cliente[id:0..N-1]{

    send atencion(id)

    // Aca no se si poner un recieve respuesta[id]() -> siendo que respuesta es el canal para que continue este proceso.
}

Process Empleado{
    
    while true:
        recieve documentos(idCli)
        atender(idCli)
}

```

### B)
```js
chan atencion(int)

Process Cliente[id:0..N-1]{

    send atencion(id)

}

Process Empleado[0..1]{
    
    while true:
        recieve atencion(idCli)
        atender(idCli)
}

```


### C)
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

2 Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos.
Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les
entrega un comprobante. Nota: maximizar la concurrencia.

```js
chan pedido[5](text, int);
chan comprobante[P](text);
chan buscarCaja(int);
chan obtenerCaja[P](int);
chan liberarCaja(int);
chan hayPedido(bool);

Process Cliente [id:0..P-1]{

    send buscarCaja(id)
    send hayPedido(true)
    recieve obtenerCaja[id](IdCaja)

    send pedido[idCaja](comprobante)
    send liberarCaja(idCaja)
    send hayPedido(true)
}


Process Admin{

    int cantEspera[5] = ([5]0)
    int minimo
    int IdAux
    bool pedido

    while true {
        receive hayPedido(pedido)
        if(!empty(buscarCaja) && empty(liberarCaja)){
            receive buscarCaja(idAux)
            min = cajaMasVacia(CantEspera)
            cantEspera[mininimo]++
            send obtenerCaja[IdAux](minimo)
        } else {
            if (!empty (liberarCaja)){
                receive liberarCaja(IdAux)
                cantEspera[idAux]--
            }
        }
    }
}


Process Caja[id:0..4]{
    int IdCli
    text pago
    text comprobante

    while(true){
        recieve pedido[id](pago,IdCli)
        generarComprobante(pago,comprobante)
        send comprobante[idCli](comprobante)
    }
}
```



3- Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2
cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar
que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se
lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender,
los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre
1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo
cocina y se lo entrega directamente al cliente correspondiente.
Nota: maximizar la concurrencia.




```js
chan pedidos (text)
chan comida[N] = ([N]0)
chan pedidosCocineros (text)
chan respuesta[3](text)
chan pregunta (int)


Process Cliente[id:0..N-1]{
    text pedido = ...
    text plato

    // Mando mi pedido a los cocineros
    send pedidos(id,pedido)
    // Espero a recibir mi plato de comida   
    receive comida[id](plato)
}


// Ya que no puedo checkear por el empty debido a que tengo 3 procesos receptores tengo que hacer un coordinador

Process Coordinador{
    text pedidoCli
    int idCli
    int idEmp

    while (True){
        // Recibo el llamado de pregunta del empleado
        recieve pregunta (idEmp)
        // Checkeo si el canal esta vacio
        if (pedidos.isEmpty()){
            // Si esta vacio le retorno vacio
            pedidoCli = "VACIO"
            idCli = -1
        } else {
            // Si no tomo el pedido del cliente y se lo doy al empleado
            recieve pedidos(pedidoCli,idCli)
        }
        // Importante las posiciones de los arreglos!
        send respuesta[idEmp](pedidoCli,idCli)

    }

}

Process Vendedor[id:0..2]{
    text pedidoCli
    int idCli

    while (True) {
        // Pregunto si esta vacio
        send pregunta(id)
        // recibo si esta vacio o el pedido del cliente
        recieve respuesta[id](pedidoCli,idCli)
        
        if (pedidoCli == "VACIO") {
            // Si esta vacio repongo las bebidas
            reponerBebidas()
        } else {
            // Si no, se lo paso a los cocineros
            send pedidosCocineros(pedidoCli,idCli)
        }
    }
}

Process Cocinero[id:0..1]{
    int idCli
    text pedidoCli
    text comida

    while (True){
        // Recibo el id del cliente y su pedido
        recieve pedidosCocineros(pedidoCli,idCli)

        // Realizo el pedido del cliente
        comida = realizarPedido(pedidoCli)

        // Se lo entrego al cliente
        send comida[idCli](comida)
    }
    
}

```




4- Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado
atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación.

a) Implemente una solución para el problema descrito.

b) Modifique la solución implementada para que el empleado dé prioridad a los queterminaron de usar la cabina sobre los que están esperando para usarla.

Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el
empleado que simula que el empleado le cobra al cliente.



a ) 


```js
chan llegada (int)
chan pasar[N] (int)
chan pago
chan factura[N]


Process Cliente[id:0..N-1]{
    int numero
    text factura
    text pedido

    send llegada(id)
    recieve pasar[id](numero)

    // usar cabina
    cabina[numero].usar()

    send pago(id,pedido)
    recieve factura[id](factura)

}

Process Empleado{
    bool cabinasOcupadas[10] = ([10] false)
    int idCli
    int numCabina
    text ticket
    
    While (true){
        receive llegada(idCli)
        // Uso if no deterministico para que no haya prioridad ni orden.
        // Si hay alguien esperando para pagar
        if(!pago.isEmpty() )-> {
            // Recibo el pago
            receive pago(idCli,numCabina)
            cabinasOcupadas[numCabina] = false
            ticket = cobrar(idCli)
            send factura[idCli](ticket)
        }
        [] (pago.isEmpty()) {
            // Si no hay nadie esperando para pagar, y hay alguien esperando para pasar a una cabina y hay cabinas libres entonces:
            if (!llegada.isEmpty() and hayCabinasClibres(cabinasOcupadas)) {
                // Consigo el id del cliente que llego
                receive llegada(idCli)
                // Consigo el numero de una cabina desocupada, es decir, cualquiera que este en false.
                numCabina = obtenerCabinaLibre(cabinasOcupadas)
                // La pongo en ocupada
                cabinasOcupadas[numCabina] = true
                // Se la mando al cliente
                send pasar[idCli](numCabina)
            }
        }
    }

       
    }
}
```

b)

```js
chan llegada (int)
chan pasar[N] (int)
chan pago
chan factura[N]


Process Cliente[id:0..N-1]{
    int numero
    text factura
    text pedido

    send llegada(id)
    recieve pasar[id](numero)

    // usar cabina
    cabina[numero].usar()

    send pago(id,pedido)
    recieve factura[id](factura)

}

Process Empleado{
    bool cabinasOcupadas[10] = ([10] false)
    int idCli
    int numCabina
    text ticket
    
    While (true){
        // Si hay alguien esperando para pagar
        if(!pago.isEmpty() ){
            // Recibo el pago
            receive pago(idCli,numCabina)
            cabinasOcupadas[numCabina] = false
            ticket = cobrar(idCli)
            send factura[idCli](ticket)
        } else {
            // Si no hay nadie esperando para pagar, y hay alguien esperando para pasar a una cabina y hay cabinas libres entonces:
            if (!llegada.isEmpty() and hayCabinasClibres(cabinasOcupadas)) {
                // Consigo el id del cliente que llego
                receive llegada(idCli)
                // Consigo el numero de una cabina desocupada, es decir, cualquiera que este en false.
                numCabina = obtenerCabinaLibre(cabinasOcupadas)
                // La pongo en ocupada
                cabinasOcupadas[numCabina] = true
                // Se la mando al cliente
                send pasar[idCli](numCabina)
            }
        }
    }

}
```




5- Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada.

a) Implemente una solución para el problema descrito.

b) Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos.

c) Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución.

d) Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.

e) Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.

Nota: ni los administrativos ni el director deben esperar a que se imprima el documento


a)

```js
chan impresora (text)


Process Administrativo[id:0..N-1]{
    text documento

    while true{
        // trabajando...
        send impresora (documento)
    }
}

Process Impresora[0..2]{
    text documento

    while true {
        recieve impresora(documento)
        imprimir(documento)
    }

}
```

b) 
```js
chan impresora(text)
chan impresoraPrioridad(text)

Process Administrativo[id:0..N-1]{
    text documento
    while true {
        // Trabajando
        send impresora(documento)
    }
}


Process Director {
    text documento
    while true {
        // Trabajando
        send impresoraPrioridad(documento)
    }
}

Process Impresora[id:0..2]{
    text documento

    while true {

        if(!impresoraPrioridad.isEmpty()){
            recieve impresoraPrioridad(documento)
        } else {
            recieve impresora(documento)
        }
        imprimir(documento)
    }
}
```



c)

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

// Tengo que usar un proceso admin para manejar y que todos puedan terminar
Process Administrador {
    text documento

    for i: 1 to N*10 {
        // Recibo el documento y se lo mando a la impresora
        recieve impresora(documento)
        send imprmirCanal(documento)
    }
    for i : 1 to 3 {    
        // Una vez no hay más documentos que recibir le mando a la impresora null para que terminen
        send imprimir(null)
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


d)


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


if no deterministico en sincronico: 

```js
// condicion es opcional si no esta es true
if (condicion); nombreProceso?canal(parametro) -> { bloque }
[] (condicion); nombreProceso?canal(parametro) -> { bloque }
...
```

do no determinisitico (se sigue ejecutando siempre y cuando haya almenos 1 verdadera y por repeticion solo ejecuta una)
```js
// Condicion tambien es opcional
do (condicion);nombreProceso?canal(parametro) -> { bloque }
[] (condicion);nombreProceso?canal(parametro) -> { bloque }
...
```


1. Suponga que existe un antivirus distribuido que se compone de R procesos robots
Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando
continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la
dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si
están o no infectados.
a) Analice el problema y defina qué procesos, recursos y comunicaciones serán
necesarios/convenientes para resolverlo.
b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos.
c) Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden
en que se hicieron



b) 

```js

Procedure Examinador[id:0..N]{
    text sitioWeb
    while (True){
        // Consigo web
        sitioWeb = buscarWebInfectada()
        // Le mando al analizador(solo hay 1), mediante el canal aviso el sitio web
        Analizador!aviso(sitioWeb)
    }
}

Procedure Analizador{
    text sitioWeb
    while (True){
        // Recibo de cualquier examinador el sitio web y lo 
        Examinador[*]?aviso(sitioWeb)
        analizar(sitioWeb)
    }
}
```



c) con orden de llegada

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



2- En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos
continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo
empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN
preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para
archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el
resultado al segundo empleado


```js

Process EmpleadoUno{
    text muestraADN
    while true{

        muestraADN = preprar_muestra()
        EmpleadoDos!Muestra(muestraADN)
        // Imagino que aca prepara muestras y cuando termina la ultima le manda null?
    }
}



Process EmpleadoDos{
    text muestraADN
    List set
    text resultado

    while true {

        EmpleadoUno?Muestra(muestraADN)

        if(muestraADN != null){
            // Si no me mandaron null, lo cual significaria que ya termino de hacer el set.
            set.add(muestraADN)
        }
        else {
            EmpleadoTres!Set(set)
            EmpleadoTres?Resultado(resultado)
            resultado.archivar()

        }

    }

}

Process EmpleadoTres{
    List set
    text resultado

    while true {
        EmpleadoDos?Set(set)
        resultado = analizar(set)
        EmpleadoDos!Resultado(resultado)
    }


}

```




3- En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo
entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los
profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.
a) Considerando que P=1.
b) Considerando que P>1.
c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta
que todos hayan llegado al aula.
Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben
terminar su ejecución



a)

```js

Process Alumno[0..N-1]{
    text examen
    examen.resolver()
    Profesor[0]!Examen(examen)

}

// P = 1 por lo que no necesito un proceso admin que reciba los examenes ya que solo se lo mandan a uno
Process Profesor[0..P-1]{
    while true{
        Alumno[*]?Examen(examen)
        corregir(examen)
    }
}

```

b) Ahora sí voy a necesitar un proceso admin que reciba, guarde en una cola y se los vaya mandando a los profesores.



```js

Process Alumno[0..N-1]{
    text examen
    examen.resolver()
    Admin!Examen(examen)
}

// P > 1
Process Profesor[id:0..P-1]{
    text examen

    while true {

        Admin!PedirExamen(id)
        Admin!ConseguirExamen(examen)
        corregir(examen)

    }
}

Process Admin{
    cola examenes
    text examen
    int idProfesor

    while true {
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
    }

}
```


c)  Ahora los alumnos deben esperar que lleguen los otros para comenzar y todos los procesos deben terminar

```js

Process Alumno[0..N-1]{
    text examen
    
    // Aviso que llegue
    Admin!Llegada()
    // Espero a poder arrancar
    Admin!Arrancar()
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

Process Admin{
    cola examenes
    text examen
    int idProfesor
    int cant

    while cant < N {
        Alumno[*]?Llegada()
        cant++
    }
    For i: 0 to cant-1{
        Alumno[i]!Arrancar()
    }


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



4- En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con
exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que
esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira.
a) Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión
mutua (sin importar el orden).
b) Modifique la solución anterior para que el empleado los deje acceder según el orden de
su identificador (hasta que la persona i no lo haya usado, la persona i+1 debe esperar).
c) Modifique la solución a) para que el empleado considere el orden de llegada para dar
acceso al simulador.
Nota: cada persona usa sólo una vez el simulador. 

a)

```js

Process Cliente[id:0..N-1] {
    Empleado!Llegada(id)
    Empleado?Subir()
    // Esta un rato en el simulador
    Empleado!Bajar()
}

Process Empleado{
    int idCli

    while true{
        // Recibo la llegada de un cliente
        Cliente[*]?Llegada(idCli)
        // Lo dejo subir
        Cliente[idCli]!Subir()
        // Espero que termine
        Cliente[idCli]?Bajar()
    }
}

```



b)

```js

Process Cliente[id:0..N-1] {
    Empleado!Llegada()
    Empleado?Subir()
    // Esta un rato en el simulador
    Empleado!Bajar()
}

Process Empleado{

    for i: 0 to n-N-1{
        // Recibo la llegada de un cliente
        Cliente[i]?Llegada()
        // Lo dejo subir
        Cliente[i]!Subir()
        // Espero que termine
        Cliente[i]?Bajar()
    }
}

```


c) Ahora sí me importa el orden, y como al hacer Cliente[*]Llegada(idCli) hace que agarre a cualquiera, no funciona como una cola, tengo que hacer un proceso admin

```js

Process Cliente[id:0..N-1] {
    
    // Le aviso que llegue al admin
    Admin!Llegada(id)
    // Espero que el empleado me deje subir
    Empleado?Subir()
    // Esta un rato en el simulador
    Empleado!Bajar()
}

Process Admin{
    int idCli
    cola clientes
    // Espero la llegada si es que hay de clientes y me guardo los ids en una cola para el orden
    do Cliente[*]?Llegada(idCli) -> {
        clientes.push(idCli)
    }
    // Si el empleado hizo un pedido y hay clientes en la cola le mando uno.
    [] !clientes.isEmpty(); Empleado?Pedido(){
        clientes.pop(idCliente)
        Empleado!Cliente(idCliente)
    }
}

Process Empleado{
    int idCli

    while true{

        // Le aviso al admin que estoy listo para recibir un cliente
        Admin!Pedido()
        // Recibo la llegada de un cliente por el admin
        Admin?Cliente(idCli)
        // Lo dejo subir
        Cliente[idCli]!Subir()
        // Espero que termine
        Cliente[idCli]?Bajar()
    }
}

```



5)En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por
E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la
máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada
Espectador una sólo una vez la máquina.

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