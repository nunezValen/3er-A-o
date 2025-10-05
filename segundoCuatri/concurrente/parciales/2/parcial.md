# Parcial 2

## Enunciado 1


```java
sem mutex = 1
sem esperando[20] = ([20] 0)
cola esperandoIDS
bool libre = true


Process Empleado[id:0..19]{
    P(mutex)
    if(!libre){
        esperandoIDS.push(id)
        V(mutex)
        P(esperando[id])
    } else {
        libre = false
        V(mutex)
    }

    pirosecuenciador.usar()

    P(mutex)
    if(!esperandoIDS.isEmpty()){
        V(esperando[esperandoIDS.pop()])
    } else {
        libre = true
    }
    V(mutex)
}
```


## Enunciado 2

```java

int arreglo[1000000] 
int total = 0
int terminados = 0

sem mutex = 1

sem terminadosSem = 1

sem esperarTerminar = 0


Process worker[id = 0..4]{

    int total = 0

    base =  arreglo.size() div 5 // 200.000 

    inicio = base * id // Ejemplo con 0, arranca en la pos 0

    fin = inicio + base -1 // termina en la poss 199.999

    for (i = inicio to fin){
        total += arreglo[i]
    }

    P(mutex)
    total += this.total // Actualizo el global
    V(mutex)


    P(terminadosSem)
    terminados ++
    if(terminados == 5){ // Si soy el ultimo en terminar tengo que levantar al resto
        for i (1..5) {
            V(esperarTerminar) // Despierto al resto y me dejo una para mi
        }
    }
    V(terminadosSem) // Desbloqueo la variable terminados

    P(esperarTerminar) // Espero a que termine el resto

    P(mutex)
    print (total)
    V(mutex)
}
```

## Enunciado 3


```java

Process Persona[id:0..49]{
    int tipoConsulta = ...

    txt consulta = ...

    Mail[tipoConsulta].Enviar(consulta,resultado)

}


Process Empleado{
     txt consulta

    
    while(true){

       Mail[1].AtenderConsulta(consulta)
       // Conseguir resultado
       Mail[1].DevolverResultado(resultado)

    }

}

Process Supervisor {

    txt consulta

    
    while(true){

       Mail[0].AtenderConsulta(consulta)
       // Conseguir resultado
       Mail[0].DevolverResultado(resultado)

    }
}



Monitor Mail[0..1]{


    txt res 
    cola consultas
    cond esperandoConsultas
    cond esperandoResultado

    Procedure Enviar(consulta : IN txt; resultado : OUT txt){

        consultas.push(consulta)
        signal(esperandoConsultas)
        wait(esperandoResultado)
        resultado = res
    
    }

    AtenderConsulta(consulta: OUT txt){

        if(consultas.isEmpty()){
            wait(esperandoConsultas)
        }
        consulta = consultas.pop()
    }

    DevolverResultado(resultado: IN txt){

        res = resultado
        signal(esperandoResultado)
    }

}

```