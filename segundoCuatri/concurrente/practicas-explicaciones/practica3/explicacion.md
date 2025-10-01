## Explicacion Practica Monitores:

##### Recordar que:
- No existen variables compartidas
- Las variables permanentes del monitor solo se pueden usar dentro del mismo.
- Si el monitor tiene un codigo de inicializacion hasta que este no termine su ejecucion el monitor no atiende llamados a los procedures

```
Monitor nombre {
 Variables permanentes del monitor

 Procedure uno ()
 Var locales al Procedure
 { ....
 }
 Código de inicialización del monitor
 {
 Inicialización variables.
 }
}
```

```
Process P[id: 0 .. N-1] {
 ….
 nombre.uno();
}

```

- La exclusion mutua es implicita dentro de un monitor al no poder ejecutar mas de un llamado a un procedimiento a la vez hasta que no termina el procedure o no se duerme en una variable condition no se libera el monitor

- La sincronizacion por condicion es explicita por medio de variables conditions usadas en los monitores. Son variables permanentes del monitor(Solo se pueden usar en el monitor que fueron declaradas)
VC = VARIABLE CONDICION.
• wait (vc): duerme al proceso en la cola asociada
a la variable condición (al final de la cola).
• Signal (vc): despierta al primer proceso dormido
en vc (al primero que se había dormido) para
que compita nuevamente para acceder al
monitor, y cuando lo haga continuar con la
instrucción después del wait.
• Signal_all (vc): despierta a todos los procesos
dormidos en vc para que todos pasen a
competir por acceder nuevamente al monitor.

```
Monitor nombre {
 cond vc;

 Procedure uno ()
 { ....
 wait (vc);
 ....
 }
 Procedure dos ()
 { ....
 signal (vc);
 ....
 signal_all (vc);
 ....
 }
}

```


### Ejercicios:
1- Existen N personas que desean utilizar un cajero automático. En este primer caso no se debe tener en cuenta el orden de llegada de las personas (cuando está libre cualquiera lo puede usar). Suponga que hay una función UsarCajero() que simula el uso del cajero

- En este problema lo único que se debe tener en cuenta es usar el Cajero con
Exclusión Mutua, no se requiere otro tipo de sincronización como por ejemplo
para respetar un orden. Por lo tanto alcanza con que el monitor represente al
Cajero Automático y tenga un procedimiento que simule el uso del mismo.

```
Monitor Cajero {
    Procedure PasarAlCajero(){
        UsarCajero();
    }
}
```
```
Process Persona [id:0..N-1]{
    Cajero.PasarAlCajero()
}
```
Como la exlusion mutua con los monitores ya esta implicita solo basta con llamar al pasar al cajero.


2- Existen N personas que desean utilizar un cajero automático. En este segundo
caso se debe tener en cuenta el orden de llegada de las personas. Suponga que
hay una función UsarCajero() que simula el uso del cajero.

- Partiendo de la solucion anterior, se respeta el orden? NO, como el monitor representa al cajero, cuando un proceso entra en él es por competencia no por el orden.

Deberiamos hacer algo del estilo: 

```
Process Persona[id:0..N-1]{
    Cajero.Pasar()
    UsarCajero()
    Cajero.Salir
}
```

- Por lo que debemos tener una variable que indique el estado del recurso(si esta libre u ocupado), de estar ocupado tiene que dormirse


```python
Monitor Cajero{
    bool libre = true;
    cond cola;
    int esperando = 0;

    Procedure Pasar(){
        if(not libre){
            esperando++; wait(cola);
        } else {
            libre = false;
        }
    }

    Procedure Salir(){
        if (esperando > 0) {
            esperando--;
            signal(cola);
        } else {
            libre = true;
        }
    }
}
```

```python
Process Persona[id: 0..N-1]{
    Cajero.Pasar()
    UsarCajero()
    Cajero.Salir()
}
```

### NOTAS:
- No uso empty(cola) ya que no se puede usar empty sobre variables condicion.
- No hago  
```python
Procedure Salir ()
 { libre = true;
 signal (cola);
 }

```
- Ya que al hacer de esta forma estoy marcando el cajero como libre y podria entrar un proceso el cual no estaba en la cola y recien llega. Primero veo si la cola esta vacia, de estarlo libero, si no hago signal, lo cual despierta al proceso que se durmio primero, ahi ejecuta el usar cajero, ya que no necesita preguntar por libre, y repite al salir.

### 3- Partiendo de la solucion anterior agregar si llega una persona con prioridad

