# Parcial 3 
#### Ya no quiero más :(


## Enunciado 1



```java

int arreglo[1000000]
double promedio
int terminados
sem mutex= 1
sem terminadosSem = 1
sem esperando = 0

Process Worker[0..9]{
    
    double total

    int base = arreglo.size() div 10 // 100000

    int inicio = id * base 

    int fin = inicio + base - 1

    for (i: inicio to fin){
        total += arreglo[i]
    }

    P(mutex)
    totalGlobal += total
    V(total)

    P(terminadosSem)
    terminados++
    if(terminados == 10){
        promedio = totalGlobal / arreglo.size()
        for(i: 1 to 10){
            V(esperando) // Levanto a los pibe
        }
    }

    P(esperando) // Espero  que los pibe terminen

    P(mutex)
    print(promedio)
    V(mutex)

}

```


## Enunciado 2


```java
sem mutex = 1
sem esperando[20] = ([20]0)
bool libre = true
queque cola

Process Camion[id:0..19]{

    P(mutex)
    if(!libre){
        cola.push(id)
        V(mutex)
        P(esperando[id])
    } else {
        libre = False
    }

    // Descargar
    
    P(mutex)

    if(!cola.isEmpty()){
        V(esperando[cola.pop()])
    } else {
        libre = true
    }

    V(mutex)


}

```


## Enunciado 3


```java

Process Profesor{

    txt consulta, respuesta
    int id

    while (true){
        Admin[0].RecibirConsulta(consulta,id)
        // Hacer la respuesta
        Admin[0].EnviarRespuesta(respuesta,id)
    }

}

Process Auxiliar {

    txt consulta, respuesta
    int id

    while (true){
        Admin[1].RecibirConsulta(consulta,id)
        // Hacer la respuesta
        Admin[1].EnviarRespuesta(respuesta,id)
    }

}

Process Persona[id:0..99] {
    int tipo = ... // 0 o 1 segun si es practica o teorica
    txt consulta,respuesta

    Admin[tipo].Consultar(consulta,respuesta,id)

}

Monitor Admin[id:0..1]{
    cond esperandoRespuesta
    cond esperandoConsulta
    txt respuestas[100] 

    Procedure Consultar(consulta: IN txt, respuesta: OUT txt; id: IN int){
        cola.push(consulta,id)
        signal(esperandoConsulta)
        wait(esperandoRespuesta)
        respuesta = respuestas[id]
    }

    Procedure RecibirConsulta(consulta: OUT txt; id : OUT int){

        if(cola.isEmpty()){
            wait(esperandoConsulta)
        }
        consulta,id = cola.pop()
    }

    Procedure EnviarRespuesta(respuesta: IN txt; id: IN int){

        respuestas[id] = respuesta
        signal(esperandoRespuesta)
    }

}

```