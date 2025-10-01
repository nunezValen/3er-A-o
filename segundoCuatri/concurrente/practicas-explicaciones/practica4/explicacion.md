## Mensajes Asincronicos

- No tengo variables compartidas.
- Los canales actuan como colas.
- Cualquier proceso puede mandar mensajes al canal y cualquier proceso puede recibir de cualquier canal.
- No se requiere sincronizacion por exclusion mutua ya que no existe la variable compartida


### Sintaxis
```
chan
```
send nombrecanal

empty (nombrecanal) 
- Cuidado con el empty, si tengo más de un proceso(Multiples receptores) me da demoras innecesarias.
- - Formas de solucionar: No usar empty, que el canal no tenga multiples receptores O AGREGAR MULTIPLES RECEPTORES.
