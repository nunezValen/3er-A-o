# Practica 5 carajo

# ADA


### Anotaciones importantes
- Los entry call se almacenan en una cola de acuerdo al orden de llegada
- Por estar como primera alternativa en un select NO tiene más prioridad
- Cuando es un solo empleado se pone Task empleado is... Si son varios es Task type empleado is... Lo mismo para cualquier TASK
- Los clientes no conocen sus ids, necesitas que el programa principal se los mande.

# Duda: 
- Si hago lo de delay, el pedido del cliente que se fue, no se queda en la cola? o como funciona?


1 Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso.
El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2
unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de
vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas,
recursos y sincronizaciones serán necesarios/convenientes para resolverlo.

a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.

b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los
vehículos.


a)

```js

Procedure Puente is
    TASK Acceso is
        ENTRY AccesoAuto; 
        ENTRY AccesoCamioneta;
        ENTRY AccesoCamion;
        ENTRY Salir (peso:IN integer);
    End Acceso

    TASK TYPE Auto;
    vecAUTOS : array(1..A) of Auto;

    TASK BODY Auto IS
    BEGIN 
        Acceso.AccesoAuto;
        // Cruzar Puente
        Acceso.Salir(1);
    END Auto;

    TASK TYPE Camioneta;
    vecCamionetas : array (1..B) of Camioneta;

    TASK BODY Camioneta IS
    BEGIN 
        Acceso.AccesoCamioneta;
        // Cruzar Puente
        Acceso.Salir(1);
    END Auto;

    TASK TYPE Camion;
    vecCamion : array (1..C) of Camion;
    
    TASK BODY Camion IS
    BEGIN 
        Acceso.AccesoCamion;
        // Cruzar Puente
        Acceso.Salir(1);
    END Auto;

    TASK BODY Acceso IS
        peso : integer := 0;
    BEGIN

        LOOP
            // Recordar que el select es no deterministico por lo que si hay más de una que cumpla se va a ejecutar cualquiera de ellas.
            SELECT
                // Si mi peso es menor que 5 puedo agarrar un auto
                WHEN (peso < 5) =>
                    ACCEPT AccesoAuto DO
                        peso := peso +1;
                    END AccesoAuto;
            OR  
                // Si mi peso es menor que cuatro entonces tengo al menos 2 unidades disponibles como para meter una camioneta.
                WHEN (peso < 4) =>
                    ACCEPT AccesoCamioneta DO
                        peso := peso + 2;
                    END AccesoCamioneta;
            OR
                // Si me peso es menor que 3 puedo meter un camion
                WHEN (peso < 3) => 
                    ACCEPT AccesoCamion DO 
                        peso := peso + 3
                     END AccesoCamion;
            OR
                ACCEPT Salir(pesoSalida : IN integer) DO
                    peso := peso - pesoSalida;
                END Salir;
            END SELECT;
        END LOOP;
    END Acceso

Begin
    null;
END Puente;


```

 
B) Aca hay dos formas de ver el mayor prioridad y no me queda claro cual seria, tenes opcion A: Si hay camiones esperando para pasar, NO PASA NADIE MÁS, hasta que no pasen ellos. Opcion B: Solo se les da prioridad cuando tienen espacio para pasar, ejemplo si hay 1 espacio, el camion no pasa pero un auto sí, por más que haya camiones esperando, en cambio si hay 3 libres, el camion pasa aunque haya autos esperando. Voy a hacer la opcion A que es más facil.

```js
Procedure Puente is
    TASK Acceso is
        ENTRY AccesoAuto; 
        ENTRY AccesoCamioneta;
        ENTRY AccesoCamion;
        ENTRY Salir (peso:IN integer);
    End Acceso

    TASK TYPE Auto;
    vecAUTOS : array(1..A) of Auto;

    TASK BODY Auto IS
    BEGIN 
        Acceso.AccesoAuto;
        // Cruzar Puente
        Acceso.Salir(1);
    END Auto;

    TASK TYPE Camioneta;
    vecCamionetas : array (1..B) of Camioneta;

    TASK BODY Camioneta IS
    BEGIN 
        Acceso.AccesoCamioneta;
        // Cruzar Puente
        Acceso.Salir(1);
    END Auto;

    TASK TYPE Camion;
    vecCamion : array (1..C) of Camion;
    
    TASK BODY Camion IS
    BEGIN 
        Acceso.AccesoCamion;
        // Cruzar Puente
        Acceso.Salir(1);
    END Auto;

    TASK BODY Acceso IS
        peso : integer := 0;
    BEGIN

        LOOP
            // Recordar que el select es no deterministico por lo que si hay más de una que cumpla se va a ejecutar cualquiera de ellas.
            SELECT
                // Si mi peso es menor que 5 puedo agarrar un auto. Y no hay camiones esperando pasar. 
                WHEN (peso < 5 AND AccesoCamion’count != 0) =>
                    ACCEPT AccesoAuto DO
                        peso := peso +1;
                    END AccesoAuto;
            OR  
                // Si mi peso es menor que cuatro entonces tengo al menos 2 unidades disponibles como para meter una camioneta. Y no hay camiones esperando pasar
                WHEN (peso < 4 AND AcessoCamion’count != 0) =>
                    ACCEPT AccesoCamioneta DO
                        peso := peso + 2;
                    END AccesoCamioneta;
            OR
                // Si me peso es menor que 3 puedo meter un camion
                WHEN (peso < 3) => 
                    ACCEPT AccesoCamion DO 
                        peso := peso + 3
                     END AccesoCamion;
            OR
                ACCEPT Salir(pesoSalida : IN integer) DO
                    peso := peso - pesoSalida;
                END Salir;
            END SELECT;
        END LOOP;
    END Acceso

Begin
    null;
END Puente;

```



2- Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar
un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de
acuerdo con el orden de llegada.
a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido
atendidos.
b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para
realizar el pago.
c) Implemente una solución donde los clientes se retiran si no son atendidos
inmediatamente.
d) Implemente una solución donde los clientes esperan a lo sumo 10 minutos para ser
atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más
y se retiran si no son atendidos inmediatamente.



a)
```js
Procedure Banco is

    TASK Empleado is
        Entry pago(pago: IN texto; Comprobante: OUT texto)
    END Empleado

    Task type Cliente;

    arrayClientes : array (1..N) of Cliente;

    TASK BODY Cliente  is
        Comprobante : texto
    
    BEGIN
        // Mando el pago y espero comprobante
        Empleado.Pago("Pago", Comprobante)
    END Cliente

    TASK BODY Empleado is
        Comprobante : texto;
        pago : texto;
    BEGIN
        LOOP
            // Agarro un pedido, genero el comprobante y al terminar se lo manda.
            ACCEPT Pago(pago : IN texto,comprobante : OUT texto)
                comproabante := generarComprobante(pago)
            END Pago
        END LOOP
    END Empleado

BEGIN 
    null;
END Banco;
```



b)

```js
Procedure Banco is

    TASK Empleado is
        Entry pago(pago: IN texto; Comprobante: OUT texto)
    END Empleado

    Task type Cliente;

    arrayClientes : array (1..N) of Cliente;

    TASK BODY Cliente  is
        Comprobante : texto
    
    BEGIN
        // Si mi pedido es atendido en menos de 10 minutos recibo el comprobante y me voy, si no me voy sin nada. 
        SELECT
            Empleado.Pago("Pago", Comprobante)
        OR DELAY 600
            NULL
        END SELECT
    END Cliente

    TASK BODY Empleado is
        Comprobante : texto;
        pago : texto;
    BEGIN
        LOOP
            // Agarro un pedido, genero el comprobante y al terminar se lo manda.
            ACCEPT Pago(pago : IN texto,comprobante : OUT texto)
                comproabante := generarComprobante(pago)
            END Pago
        END LOOP
    END Empleado

BEGIN 
    null;
END Banco;
```



C)

```js
Procedure Banco is

    TASK Empleado is
        Entry pago(pago: IN texto; Comprobante: OUT texto)
    END Empleado

    Task type Cliente;

    arrayClientes : array (1..N) of Cliente;

    TASK BODY Cliente  is
        Comprobante : texto
    
    BEGIN
        // Si mi pedido no es atendido de inmediato me voy 
        SELECT
            Empleado.Pago("Pago", Comprobante)
        ELSE
            NULL
        END SELECT
    END Cliente

    TASK BODY Empleado is
        Comprobante : texto;
        pago : texto;
    BEGIN
        LOOP
            // Agarro un pedido, genero el comprobante y al terminar se lo manda.
            ACCEPT Pago(pago : IN texto,comprobante : OUT texto)
                comproabante := generarComprobante(pago)
            END Pago
        END LOOP
    END Empleado

BEGIN 
    null;
END Banco;
```


D)

```js
Procedure Banco is

    TASK Empleado is
        Entry pago(pago: IN texto; Comprobante: OUT texto)
    END Empleado

    Task type Cliente;

    arrayClientes : array (1..N) of Cliente;

    TASK BODY Cliente  is
        Comprobante : texto
    
    BEGIN
        // Si mi pedido es atendido en menos de 10 minutos recibo el comprobante y me voy, si no, espero a que sea atendido inmediatamente o me voy 
        SELECT
            Empleado.Pago("Pago", Comprobante)
        OR DELAY 600
            SELECT
                Empleado.Pago("Pago",comprobante);
            ELSE
                NULL;
            END SELECT
        END SELECT
    END Cliente

    TASK BODY Empleado is
        Comprobante : texto;
        pago : texto;
    BEGIN
        LOOP
            // Agarro un pedido, genero el comprobante y al terminar se lo manda.
            ACCEPT Pago(pago : IN texto,comprobante : OUT texto)
                comproabante := generarComprobante(pago)
            END Pago
        END LOOP
    END Empleado

BEGIN 
    null;
END Banco;
```



3- Se dispone de un sistema compuesto por 1 central y 2 procesos periféricos, que se
comunican continuamente. Se requiere modelar su funcionamiento considerando las
siguientes condiciones:
- La central siempre comienza su ejecución tomando una señal del proceso 1; luego
toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una
señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.
- Los procesos periféricos envían señales continuamente a la central. La señal del
proceso 1 será considerada vieja (se deshecha) si en 2 minutos no fue recibida. Si la
señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y
vuelve a mandarla (no se deshecha).


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



4- En una clínica existe un médico de guardia que recibe continuamente peticiones de
atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la
clínica ser atendidos.
Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo
haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del
médico. Si no es atendida tres veces, se enoja y se retira de la clínica.
Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente
le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el
momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la
petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa
trabajando y haciendo más peticiones.
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos.
Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando
está libre aprovecha a procesar las notas dejadas por las enfermeras.




```js

Procedure Hospital is

    TASK Medico is
        Entry AtencionEnfermeras(pedido: in text)
        Entry AtencionPacientes(pedido: in text)
    END Medico

    TASK Escritorio is
        entry DejarNota(pedido: in text)
        Entry Notas(nota: out text)
    END Escritorio

    TASK TYPE Enfermera;
    enfermeras : array(1..E) of Enfermera;

    TASK TYPE Persona;
    personas : array(1..P) of Persona;

    TASK BODY Persona is
        pedido : text
    BEGIN
        FOR (1 to 3) LOOP
            SELECT Medico.AtencionPacientes(pedido)
                break
            OR DELAY 300
                DELAY 600
        END LOOP
    END


    TASK BODY Enfermera
        pedido : text
    BEGIN 
        // Llamo al medico
        SELECT Medico.AtencionEnfermeras(pedido)
        // Si no me atiende al instante le dejo el pedido
        else Escritorio.DejarNota(pedido)
    END Enfermera

    TASK BODY Escritorio
        notas: cola;
        nota: text
    BEGIN
        LOOP
            SELECT
                ACCEPT DejarNota(nota: IN text) DO
                    notas.push(nota);
                END DejarNota
            OR 
                WHEN (!notas.IsEmpty()) => 
                    ACCEPT Notas(nota : OUT Text) DO
                        nota = notas.pop()
                    END Notas;
            END SELECT  
        END LOOP
    END Escritorio



    TASK BODY Medico IS
        pedido: text
    BEGIN
        LOOP
            SELECT 
                ACCEPT AtencionEnfermeras(pedido: in text) DO 
                    procesar(pedido);
                END AtencionEnfermeras;
            OR 
                WHEN (AtencionEnfermeras'Count = 0) =>
                    ACCEPT AtencionPacientes(pedido: in text) DO
                        procesar(pedido)
                    END AtencionPacientes
            ELSE
                Escritorio.Notas(nota)
                procesar(nota)
            END SELECT
        END LOOP
    END Medico

BEGIN

    null

END Hospital

```



5- En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos
de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos.
Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la
respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error,
vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde
que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo
2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a
intentarlo (usando el mismo documento).



```js

Procedure Sistema is

    TASK Servidor is
        Entry Pedidos(trabajo : IN text ;is_okay : out bool);
    END Servidor

    Task type Usuario;

    usuarios: array(1..U) of Usuario;

    TASK BODY Usuario IS
        trabajo : text
        is_okay: bool := False
    BEGIN

        While (!is_okay) LOOP // ARREGLAR PARA QUE PONGA QUE ARREGLA EL TRABAJO
            SELECT 
                Servidor.Pedidos(trabajo,is_okay);
                // Si el trabajo estaba mal lo arreglo
                if(!is_okay) 
                    arreglar(trabajo)
            OR 
                DELAY 120
                    DELAY 60
            END SELECT
        END LOOP
    END Usuario


    TASK BODY Servidor IS
        trabajo : text
        is_okay : bool
    BEGIN

        LOOP
            ACCEPT Pedidos(trabajo,is_okay) DO
                corregir_trabajo(trabajo,is_okay)
            END Pedidos
        END LOOP
    END Servior

BEGIN 
    null
END

```

6- En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada
una conoce previamente a que equipo pertenece). Cuando las personas van llegando
esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4
integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que
cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser
de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al
finalizar cada persona debe conocer el grupo que más dinero junto. Nota: maximizar la
concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una
persona existe una función Moneda() que retorna el valor de la moneda encontrada.

```js

Procedure Juego IS

    TASK TYPE Equipo IS 
        Entry Llegada
        Entry Comenzar
        Entry id (id : IN int)
        Entry Monedas(valor : IN int)
        Entry ganador(equipoGanador : OUT int)
        Entry ConseguirGanador(equipo : out int)
    END Equipo
    equipos : array(1..5) of Equipo;

    TASK TYPE Persona;
    personas : array(1..20) of Persona;

    TASK Admin IS
        Entry TotalEquipo(total : IN int,numEquipo : IN int )
    END Admin


    TASK BODY Admin IS
        resultados : array (1..5) of int;
        ganador : int
    BEGIN

        FOR (i : 1 to 5) LOOP
            ACCEPT TotalEquipo(total : in INT , numEquipo : in int) DO // Aca podria llevar dos variables tal vez y hacer if total > maxActual actualizar el max y el equipo pero seria agregar logica al pedo a mi parecer. Es decir, hay otros equipos esperando mientras hago esos ifs...
                resultados[numEquiipo] := total
            END TotalEquipo
        END LOOP

        // Para no tener que hacer todo lo del maximo hago así y fue
        ganador := resultados.getMaximaPos()

        // Le mando los ganadores a cada equipo
        FOR (i : 1 to 5) LOOP
            Equipo[i].Ganador(ganador)
        END LOOP
    END Admin

    TASK BODY Equipo IS
        totalEquipo : int = 0
        equipoGanador : int = -1
        ID : int 
    BEGIN
        // Consigo mi id
        ACCEPT ID (id : in int) DO
            ID := id
        END ID

        // Recibo la llegada de mi equipo y les digo que comiencen.
        FOR (i : 1 to 4) LOOP
            ACCEPT Llegada
        END LOOP
        For (i: 1 to 4) LOOP
            ACCEPT Comenzar
        END LOOP

        // Recibo las monedas del equipo
        FOR (i : 1 to 4) LOOP
            ACCEPT Monedas(valor : IN int) DO
                totalEquipo := totalEquipo + valor
            END Monedas
        END LOOP

        // Se las paso al admin para hacer el recuento

        Admin.TotalEquipo(totalEquipo,id)
        ACCEPT Ganador (equipoGanador : IN int) DO
            equipoGanador := equipoGanador;
        END Ganador

        // Se paso a las personas quien ganó.
        For (i: 1 to 5 ) LOOP
            ACCEPT ConseguirGanador(equipo : out int) DO
                equipo := equipoGanador
            END ConseguirGanador
        END LOOP
    END Equipo


    TASK BODY Persona IS
        equipo : int = ...
        equipoGanador : int;
        total : 0;
    BEGIN
        // Aviso que llegue y espero que acepten el comenzar
        Equipo[equipo].Llegada;
        Equipo[equipo].Comenzar;

        // Junto mis 15 monedas
        for ( i : 1 to 15) DO
            total := total + EncontrarMoneda() 
        END FOR

        // Se las paso al equipo 
        Equipo[equipo].Monedas(total);
        // Me entero quien ganó
        Equipo[equipo].ConseguirGanador(equipoGanador);
    END Persona


BEGIN
    // Le doy a los equipos sus ids.
    FOR (i: 1 to 5) LOOP
        Equipo[i].id(i)
    END LOOP

END Juego


```






7- Se debe calcular el valor promedio de un vector de 1 millón de números enteros que se
encuentra distribuido entre 10 procesos Worker (es decir, cada Worker tiene un vector de
100 mil números). Para ello, existe un Coordinador que determina el momento en que se
debe realizar el cálculo de este promedio y que, además, se queda con el resultado. Nota:
maximizar la concurrencia; este cálculo se hace una sola vez.

```js

Procedure Calculo IS

    TASK TYPE Worker;
        Entry Vector(desde,hasta : in int; vector : array)
    END Worker
    workers : array(0..9) of Worker


    TASK Coordinador IS
        ENTRY Resultado (total : IN int) 
    END Coordinador


    TASK BODY Coordinador IS
        arreglo : array (0..999999) of int
        promedio : int = 0
        base,inicio,fin,i : int = 0
    BEGIN

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

        promedio := promedio / 10
    END Coordinador


    TASK BODY Worker IS
        desde,hasta,i,total : int = 0
        vector : array;

    BEGIN

        ACCEPT Vector( desde,hasta : in int; vector : in array) DO
            desde := desde;
            hasta := hasta;
            vector := vector;
        END Vector

        For (i : desde to hasta) LOOP
            total := total + vector[i] 
        END LOOP

        Coordinador.Resultado(total);
    END Worker


BEGIN

    null

END Calculo
```


8- Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores
para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia;
a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente
manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores
para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más
se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el
código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores.
Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. Nota:
suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde
recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el
código y el valor de similitud de la huella más parecida a test en la BD correspondiente.
Maximizar la concurrencia y no generar demora innecesaria.


```js

Procedure Sistema IS

    TASK TYPE Servidor IS
        Entry EncontrarHuella(test : IN text)
    END Servidor

    TASK Especialista IS
        Entry Valor(valor,codigo : IN int)
    END Servidor



    servidores : array (1..8) of Servidor

    TASK BODY Especialista IS
        huella : imagen
        codigo_max,valor_max : int := -1
    BEGIN

        LOOP
            huella = getTest()
            for (i : 1 to 8 ) LOOP
                Servidor[i].EncontarHuella(huella)
            END LOOP

            for (i : 1 to 8) LOOP
                ACCEPT Valor(valor,codigo : IN INT) DO
                    if (valor > valor_max) then
                        valor_max := valor
                        codigo_max := codigo
                    fi
                END Valor
            END LOOP

            print("el codigo es" + codigo)
        END LOOP
    END Especialista


    TASK BODY SERVIDOR IS
        bd : base_datos
        test : imagen
        valor, codigo : int
    BEGIN

        LOOP
            // Agarro la huella, no calculo dentro del do porque generaria demora innesaria ya que el resto de servidores no podria agarrar la huella hasta que yo termine.    
            ACCEPT EncontarHuella(test : in int) DO
                test := test;
            END EncontrarHuella

            // Busco la huella
            bd.Buscar(test,codigo,valor)

            // Se la mando
            Especialista.Valor(valor,codigo)

        END LOOP
    END Servidor

BEGIN

    NULL

END Sistema



```

9- Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3
camiones. Hay P personas que hacen reclamos continuamente hasta que uno de los 
camiones pase por su casa. Cada persona hace un reclamo y espera a lo sumo 15 minutos a
que llegue un camión; si no pasa, vuelve a hacer el reclamo y a esperar a lo sumo 15
minutos a que llegue un camión; y así sucesivamente hasta que el camión llegue y recolecte
los residuos. Sólo cuando un camión llega, es cuando deja de hacer reclamos y se retira.
Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos
ha hecho sin ser atendido. Nota: maximizar la concurrencia.



```js

Procedure Ciudad IS

    TASK TYPE Camion IS
        Entry Atender(idPersona : in INT)
        Entry ConseguirID(id: in int)
    END Camion
    camiones : array (1..8) of Camion

    TASK TYPE Persona IS
        Entry Fin 
        Entry ID (id : in int)
    END Persona
    personas : arrray (1..P) of Persona;


    TASK Coordinador IS
        ENTRY Libre (idCamion : in int)
    END Coordinador

    TASK BODY Coordinador IS
        camionLibre,camionLibre : in int
    BEGIN
        LOOP
            // Agarro o espero que un camion me diga que esta libre
            Accept Libre(idCamion : in int) DO
                camionLibre := idCamion;
            END Libre

            // Acepto el reclamo de una persona
            Accept Reclamo(idPersona : in INT) DO
                // Le mando el camion
                camiones[camionLibre].Atender(idPersona);
            END Reclamo
        END LOOP
    END Coordinador

    TASK BODY Persona IS
        id : int
    BEGIN
        // Acepto el id
        ACCEPT ID (id : in INT) DO
            id := id
        END ID
        // Mientras no se atienda mi llamado sigo haciendo reclamos cada 15 min
        WHILE (seguir) LOOP
            SELECT Coordinador.Reclamo(id)
                ACCEPT Fin DO
                    seguir := false;
            OR  DELAY 900
            END SELECT
        END LOOP
    END Persona

    TASK BODY Camion IS
    id,idPersona : int
    BEGIN
        // Conseguimos mi id
        ACCEPT ConseguirID(id : in int) DO
            id:=id;
        END ConseguirID
        LOOP
            // Le aviso al coordinador que estoy libre
            Coordinador.Libre(id)
            ACCEPT Atender (idPersona : in INT) DO
                idPersona := idPersona
            END Atender

            // Atendemos el reclamo
            personas[idPersona].atenderReclamo() // Funcion fictica

            // Le avisamos que terminamos
            personas[idPersona].Fin;
        END LOOP
    END Camion

BEGIN
    FOR (i: 1 to P) LOOP
        personas[i].ID(i)
    END LOOP

    FOR (i : 1 to 8) LOOP
        camiones[i].ConseguirID(i)
    END LOOP
END Ciudad
```