# PMS


```js

Process Paciente[id:1..15] {
    estudio = obtenerEstudio()
    Medico!Turno(estudio)
    Medico?Resultado(res)
}

Process Medico{

    For (i : 1 to 15) {
        Paciente[i]?Turno(estudio)
        res = revisarEstudio(estudio)
        Paciente[i]!Resultado(res)
    }
}
```


# Ada

```js
Procedure Maraton IS

    TASK TYPE Persona;
    Personas : array(1..P) of Persona


    TASK Organizador IS
        Entry dispacitados(dni : in int; r : out remera; c : out chip)
        Entry prioridad(dni : in int; r : out remera; c : out chip)
        Entry normal(dni : in int; r : out remera; c : out chip)
    End Organizador

    TASK BODY Persona IS
        prioridad : int = obtenerPrioridad()
        dni : int = obtenerDni()
    BEGIN
        if(prioridad == 0) then
            Organizador.Discapacitados(dni,r,c)
        elif (prioridad == 1) then
            organiazdor.Prioridad(dni,r,c)
        else 
            Organizador.Normal(dni,r,c)
    END Persona


    TASK BODY Organizador IS

    BEGIN
        FOR (i : 1 to P) LOOP
            SELECT
                ACCEPT Discapacitados(dni : in int; r : out remera; c : out chip) DO
                    r = obtenerRemera(dni)
                    c = obtenerChip(dni)
                END Discapacitados
            OR WHEN(Discapacitados´Count = 0) -> 
                ACCEPT Prioridad(dni : in int; r : out remera; c : out chip) DO
                    r = obtenerRemera(dni)
                    c = obtenerChip(dni)
                END Prioridad
            OR WHEN (Discapacitados´Count = 0 AND Prioridad = 0) ->
                ACCEPT Normal(dni : in int; r : out remera; c : out chip) DO
                    r = obtenerRemera(dni)
                    c = obtenerChip(dni)
                END Normal
            END SELECT
        END LOOP
    END

BEGIN

    NULL

END Maraton
```