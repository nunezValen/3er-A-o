

# PMA

1 - 

```js

chan llegada()
chan recibirExamen(text)
chan nota[N] (double)
chan entregarExamen(text,id)

Process Profesor {

    for (i : 1 to N) {
        recieve llegada()
    }

    for (i: 1 to N) {
        send recibirExamen(examen)
    }

    for (i : 1 to N){
        receive entregarExamen(examen,idA)
        examen.corregir(nota)
        send nota[idA](nota)
    }

}

Process Alumno [id: 1..N]{
    examen : text
    nota : int


    send llegada()
    receive recibirExamen(examen)

    examen.resolver()
    send entregarExamen(examen,id)
    recieve nota(nota)
}


```


# PMS

2-

```js

Process Paciente[id:1..15]{
    tratamiento : text

    Coordinador!Llegada(id)
    Medico?Tratamiento(tratamiento)


}


Process Coordinador{
    cola pacientes 


    for (i : 1 to 30) {

        if Paciente[*]?Llegada(id) ->{
            pacientes.push(id)
        }
        [] !pacientes.isEmpty() ; Medico?Listo -> {
            Medico!EnviarPaciente(pacientes.pop())
        }
    }
}



Process Medico{

    for (i : 1 to 15) {

        Coordinador!Listo()

        Coordinador?EnviarPaciente(idPaciente)

        tratamiento = atender(idPaciente)

        Paciente[idPaciente]!Tratamiento(tratamiento)

    }

}


```



# ADA

```js
Procedure Competencia IS

    TASK Coordinador IS
        Entry Llegada(problema_participantes : out text)
        Entry Resolucion(problema : in text; resultado : out text; posicion : out int )
    END Coordinador

    TASK TYPE Participante;
    participantes : array(1..P) of Participante;


    TASK BODY Participante IS
        problema : text
        posicion : int
        resultado : text
    BEGIN

        Coordinador.Llegada(problema)

        // Resuelvo el problema
        problema.Resolver()

        // Mando y espero mi resultado
        Coordinador.Resolucion(problema,resultado,posicion)

    END Participante


    TASK BODY Coordinador IS

    podio : int = 0
    problema : text

    BEGIN
        generarProblema(problema)

        FOR (i : 1 to P*2) LOOP
        SELECT 
            ACCEPT Llegada (problema_participantes : out text) DO
                problema_participantes := problema
                END Llegada
            OR 
                ACCEPT Resolucion(problema : in text; resultado : out text; posicion : out int ) DO
                    // Obtenemos su posicion
                    podio := podio + 1
                    posicion := podio
                    // Obtenemos el resultado
                    resultado := problema.Corregir()
                END Resolucion
        END LOOP
    END

BEGIN
    NULL
END Competencia 

```