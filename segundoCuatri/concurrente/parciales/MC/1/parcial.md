# Parcial 1

## Enunciado 1



```java

char vector[1000000]
resultadosTotales[2] = ([2]0)
sem mutex = 1
sem terminado = 1
int cantTerminados =0 
sem esperando = 0

Process worker[id=0..3]{

    resultados[2] = [(2) 0]

    base = vector.size div 4 // 250.000 
    
    inicio = id * base // Para el caso de id 0 arranca el la pos 0

    fin = inicio + base -1 // hasta la pos 249.000
    // Si fuese el 1 arranca en la 250.000 hasta la 499.999. Todos recorriendo 250.000 posiciones


    for (i: inicio to fin){
        if(vector[i] == "F"){
            resultados[0]++
        } elif (vector[i] == "C"){
            resultados[1]++
        }
    }

    
    P(mutex)
    resultadosTotales[0] += resultados[0]
    resultadosTotales[1] += resultados[1]
    V(mutex)


    P(terminado)
    cantTerminados++
    if(cantTerminados == 4){
        for (1 to 4){
            V(esperando)
        }
    }

    P(esperando)
    
    P(mutex)
    print("cantidad de veces que aparece F = " resultadosTotales[0] "Cantidad de veces que aparece C = "resultadosTotales[0])
    V(mutex)

}
```


## Enunciado 2


```java

sem esperandoTurno[B] = ([B] 0)
sem mutex = 1
bool libre = true
Queue cola

Process Barco[id:0..B-1]{

    P(mutex)
    if(!libre){
        cola.push(id)
        V(mutex)
        P(esperandoTurno[id])
    } else {
        libre = false
        V(mutex)
    }

    DESCARGAR()

    P(mutex)
    if(!cola.isEmpty()){
        V(esperandoTurno[cola.pop()])
    } else {
        libre = true
    }
    V(mutex)
}

```



## Enunciado 3

```java

Process Empleado[id:0..1]{
    str tipo = ...

    for (1 to 15){
        acopiadora[tipo].EsperarCamion()
    }


}


Process Camion[id:0..29]{
    str tipo = ... // Tipo 0 maiz, tipo 1 girasol

    acopiadora[tipo].llegar()
    descargar()
    acopiadora[tipo].salir()
    

}



Monitor Acopiadora[id:0..1] {

    cond empleado
    cond esperandoAviso
    cond camion

    int esperandoAviso = 0
    

    Procedure Llegar(){
        esperandoAviso++ // Llego, levanto al empleado y me duermo
        signal(empleado)
        wait(esperandoAviso)

    }

    Procedure EsperarCamion(){
        if(esperandoAviso == 0){
            wait(empleado) // Si no llego camion me duermo
        }
        esperandoAviso-- // Una vez llega me duermo y espero a que termine de descargar
        signal(esperandoAviso)
        wait(camion)
    }


    Procedure Salir (){
        signal(camion) // Cuando se va el camion despierta al empleado y agarra al proximo
    }
}

```