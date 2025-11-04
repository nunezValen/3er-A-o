# Clase 9 : El final del recorrido

## Sistemas paralelos

### Sistemas

- Concurrente: Cuando dos o más cosas suceden simultaneamente pero no necesariamente a la vez. Son varios procesos e hilos ejecutandose simultaneamente.

- Paralelo: Cuando dos o más tareas suceden al mismo tiempo, cada tarea en una unidad de procesamiento distinta para mejorar el rendimiento(reducir el tiempo de ejecución). Es un programa concurrente donde los procesos o hilos se ejecutan en una unidad de procesamiento independiete.

- Distribuido: Conjunto de nodos independientes, cada uno con su propio sistema, memoria, etc. Los cuales colaboran mediante el envio y recepción de mensajes para lograr un objetivo común y aparece al usuario como un sistema unico y coherente. Es un programa concurrente que implica comunicación por mensajes. Tiene la imposibilidad de compartir variables(memoria). Dos maquinas virtuales puede considerarse distribuido ya que si bien usan la misma memoria y recursos no se pueden comunicar mediante ella.

### Por qué es importante usar sistemas paralelos?
- Son la base de la inteligencia artificial, sirve para la ciencia y calculos pesados

- El profe mostró un ejemplo de la multiplicación de una matriz de 1024 x 1024 y secuencialmente se tarda 2.90 segundos(Optimizado) mientras que usando sistemas paralelos y gpu se tarda 0.005 segundos. Haciendo escala, si fueses a ejecutar un algoritmo paralelo 5 años, en secuencial te tardarias 2900 AÑOS(A LA MIERDA)

### Arquitecturas de sistemas paralelos
Se clasifican segun su espacio de direcciones:

- Memoria compartida

- Memoria distribuida(clusters) -> y se tienen que mandar mensajeitos 

### Esquemas de multiprocesadores de memoria compartida(una maquina):
- Esquemas uma: todos los cpus comparten memoria pero tienen como intermediarios sus ram

- Esquemas NUMA: cada cpu tiene su memoria y son intermediados por sus propias cache. El tiempo de cualquier cpu a cualquier moria RAM no es tan tardio como se piensa, lo que es tardado es el guardar/acceder a cache. Cualquier procesador puede acceder a cualquier memoria, solo que es más costoso acceder a las que no son de ellas(por lo de la cache)

### Esquemas de multiprocesadores con memoria distribuida(varias maquinas)
- Procesadores conectados por una red
- Cada maquina tiene su memoria local
- Interacción SOLO por pasaje de mensajes

### Diseño de algoritmos paralelos
- La mejor solucion puede diferir totalmente de la sugerida por algoritmos secuenciales. 
- Puede darse un enfoque metódico para maximizar el rango de opciones consideradas, brindar mecanismos para evaluar las alternativas, y reducir el costo de backtracking por malas elecciones.

Para diseñar un algoritmo paralelo se deben realizar alguno de los siguientes pasos:
- identificar porciones de trabajo(tareas) concurrentes. 
- Consumo energetico
- Me cambio la diapositiva el hdp

#### Descomposicion de tareas
- 