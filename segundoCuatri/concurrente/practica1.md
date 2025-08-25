# Variables compartidas

## Explicacion practica 1

Ejercicios de historias posibles: Es solo ver los distintos resultados y caminos que pueden tener dos procesos

### Forma general del await
La logica del await es uno chequea la condicion, si no se cumple sale y vuelve a chequear. Cuando la condicion es verdadera y pasa a ejecutar las sentencias la condicion no puede pasar a ser falsa hasta que no se completen todas las sentencias. Es **atomico**

### Ejemplo 2
Se tiene un salon con 4 puertas por donde entran alumnos a un examen. Cada puerta lleva la cuenta de los alumnos que entraron por ella y a su vez se lleva cuenta total de personas en el salón. 
**Debemos identificar que son procesos y cuales variables compartidas**
1. La variable que cuenta la cantidad de alumnos que pasa por cada puerta.
2. El proceso de cada puerta

quedaria algo así:

int total = 0;

process puerta[id:0..3]{
    int parcial = 0;
    while true{
        *esperar llegada*
        parcial = parcial +1;
        < total = total +1 > no tengo los simbolitos de menor e igual.
    }
}


<> significa que la operacion es atomica


## Ejemplo 3
Hay un docente que les debe tomar examen oral a 30 alumnos(uno a la vez) de acuerdo al orden dado por el identificador del proceso

**Procesos** : alumnos y profesor. 
**Variables Compartidas** : No hay.


int actual = -1;
Process alumno [id:0..29]
{
    <await ( actual == id )>
    //Rinde
    //Espera a que termine
}

Porcess Docente
{
    for i = 0..29
    {
        actual = i
        //Toma examen
        //Avisa a i que termino
    }
}


#### Ahora debemos sincronizar la salida

bool listo = false;
int actual = -1;

Process alumno [id:0..29]
{
    <await ( actual == id )>
    //Rinde
    <await listo ;>
     listo = false; 
}

Porcess Docente
{
    for i = 0..29
    {
        actual = i
        //Toma examen
        <listo = true;>
        <await (not listo)> //Tengo que esperar que el alumno se vaya. Es necesario hacerlo con <>
    }
}



**Qué pasa si el alumno no llego?**
Tengo que esperar a que el alumno llegue (Barrera entre dos procesos)



bool listo = false;
int actual = -1;
ok = false;

Process alumno [id:0..29]
{
    <await ( actual == id )>
    ok = true;
    //Rinde
    <await listo ;>
     listo = false; 
}

Porcess Docente
{
    for i = 0..29
    {
        actual = i
        <await ok>
        ok=false;
        //toma examen
        <listo = true;>
        <await (not listo)> //Tengo que esperar que el alumno se vaya. Es necesario hacerlo con <>
    }
}




### Ejemplo 4
Un cajero automatico debe ser usado por N personas de a uno a la vez y segun el orden de llegada al mismo. En caso de que llegue una persona anciana, la deben dejar ubicarse al principio de la cola.

Exclusion mutua selectiva.

Process persona [id:0..n-1]{
    //Si el cajero no esta libre debe encolarse. *La primer condicion a chequear es la condicion por la cual me voy a dormir, por eso veo si me tengo que encolar en lugar de enconlarme directamente* 
    //Usar el cajero
    //Liberar el cajero
}


Tengo que tener una cola con prioridad que al hacer pop me de la persona con mas prioridad

colaEspecial C;
int siguiente = -1;

process persona[id:0..n-1]{
    int edad = ... ;
    <If (siguiente == -1 siguiente = id
    else agregar(C,edad,id)>

    <await(siguiente == id)>
    
    //usa cajero
    
    <if empty (C) siguiente = -1
    else siguiente = sacar(C)>
}