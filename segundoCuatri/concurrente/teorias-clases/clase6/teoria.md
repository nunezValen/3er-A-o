## Clase 6 teoria

esperas innecesarias, bloqueos, etc son motivos de desaprobacion

### Programacion distribuida

No puedo definir una variable sola, los procesos se comunican atraves de mensaje. 

Asincronicos: Hay un canal que dejan mensajes y continuan. Para sacar los mensajes es en orden FIFO

#### Pasaje de mensajes: Mensajes sincrónicos

Los canales son tipo link o punto a punto, 1 emisor y 1 receptor.

Hay un canal por cada proceso. 

El send por ahora lo llamamos sync_send.

##### Propiedades del envio de mensajes sincronicos

- El emisor espera a que el mensaje sea recibido por el receptor
- La cola de mensajes asociada con un send sobre un canal se reduce a 1 mensaje -> menos memoria
- El grado de concurrencia se reduce. 

En productor y consumidor, que pasa con los buffer si tengo varios consumidores y varios productores? Voy a necesitar un proceso llamado buffer que es un intermediario. 


##### En qué casos se reduce la concurrencia? 
- Cuando un proceso esta liberando un recurso, debe esperar que el servidor atienda su pedido.
- Cuando un cliente quiere escribir, debe esperar a ser atendido, en lugar de enviar la escritura y continuar.


El send: ! y el recive: ?

Ejemplo

```python
Process A
    B ! e # A le manda e(mensaje) a B

Process B 
    A ? e # B espera recibir e(mensaje) de A
```


##### Sintaxis:

Para recibir de cualquiera

Fuente[*] ? port(x1,..,xn)

Ejemplo

Persona[*] ? port(x1,...,xn) // Espero recibir de cualquier persona un mensaje.


##### Comunicacion guardada

La sentencia de comunicacion es solo recepcion, NO envio. 

if b1; comunicaion -> s1;
   b2; comunicacion2 -> s2;
fi
El if falla cuando todas las condiciones booleanas sean falsas.
Si las dos guardas se cumplen elige una de las dos y termina. El que vuelve a ejecutarse es el Do, que se ejecuta hasta que todas las condiciones booleanas sean falsas.


##### Cosas que se confunden en los exmenes 
?
