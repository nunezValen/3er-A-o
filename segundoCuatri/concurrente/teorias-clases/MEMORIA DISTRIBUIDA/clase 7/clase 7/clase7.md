## Clase 7 

PMA aumenta el grado de concurrencia, pero es un quilombo


Patrones de comunicacion:

- Productor/consumidor
- Pares que interactuan
- Cliente/servidor

##### Cliente servidor
Alguien hace un send haciendo un requerimiento(cliente) a un servidor
El servidor atiende ese req y el cliente recibe
Pasaje de mensaje sincronico es la base que se modifica para resolver cliente/servidor

cliente{
send request
receive[i].response
}
Si siempre que hago una request tengo que a la linea siguiente hacer un receive por qué no hacer todo en un mismo llamado

CALL pedido(m,s)

Las adaptaciones del mensaje sincronico son 

RPC Y RENDEZVOUS


- El pasaje de mensajes se ajusta bien a problemas de filtros y pares que interactuan ya que se plantea la comunicacion unidireccional.
- Pero para resolver cliente-servidor la comunicacion bidireccional obliga a especificar 2 tipos de canales
- RPC (Remote procedure call) y rendezvous son tecnicas de comunicacion y sincronizacion entre procesos que suponen un canal BIDIRECCIONAL.


### En que se diferencian?

- En RPC hay un proceso por llamado
- En rendezvous hay un unico proceso que atiende los llamados


## RPC

- Los programas se descomponen en modulos (con procesos y procedures), que pueden residir en espacios de direcciones distintos.
- Los procesos de un modulo pueden compartir variables y llamar a procedures de ese modulo.
- Un proceso de un modulo puede comunicarse con procesos de otro modulo solo invocando procedimientos exportados por ese

module Mname
headers de procedures exportados (visibles)
body
declaraciones de variables
código de inicialización
cuerpos de procedures exportados
procedures y procesos locales
end


- Los procesos locales son llamados background

- Header de un procedure visible :
```
op opname (formales) [returns result]
```

- El cuerpo de un procedure visible es contenido en una declaración proc:
```
proc opname(identif. formales) returns identificador resultado
declaración de variables locales
sentencias
end
```

- Un proceso (o procedure) en un módulo llama a un procedure en otro
ejecutando:
```
call Mname.opname (argumentos)
```

- Por cada call se crea un proceso, el llamador se demora(Hasta que no se responde su call no continua su ejecucion)

- Si el proceso llamador y el procedure estan en el mismo espacio de direcciones es posible evitar crear un nuevo proceso.

- Por sí mismo RPC es solo un mecanismo de comunicación.

## RENDEZVOUS

- Combina comunicacion y sincronizacion: 
- - Como tengo un solo proceso por servidor ese proceso decide a quien deja esperadno. 
- - El servidor hace sentencias de entrada IN, que atiende de a uno por vez

- Si un modulo exporta OPNAME, el proceso server en el modulo realiza rendezvous con un llamador de OPNAME ejecutando la sentencia de entrada: 
```
IN OPNAME (parametros formales) -> S; ni
```

- Las partes entre IN y NI se llaman operaciones de guardas.



