## 1. ¿Qué es una red? ¿Cuál es el principal objetivo para construir una red?
Es un grupo de computadoras o dispositivos interconectados. Su objetivo es compratir recursos(dispositivos, información o servicios).

## 2.¿Qué es Internet? Describa los principales componentes que permiten su funcionamiento.
Es una red de redes de computadoras, descentralizada, pública, que ejecutan el conjunto abierto de protocolos TCP/IP. Integra diferentes protocolos de un nivel más bajo.

### Principales componentes que permiten su funcionamiento

#### Dispositivos finales (Hosts)
Computadoras, laptops, celulares, servidores, tablets, etc. Son los equipos donde se generan o reciben datos.

### Dispositivos de red (Intermediarios)
Routers, switches, gateways, puntos de acceso (AP). Permiten la interconexión de redes y dirigen el tráfico.
**Ejemplo:** los routers de los proveedores de Internet.

### Medios de transmisión
Cables de cobre (UTP), fibra óptica, señales electromagnéticas (WiFi, 4G/5G, satélite).
Son los “canales” por donde viajan los datos.

### Protocolos de comunicación
Conjunto de reglas que especifican cómo se intercambian los datos.
El más importante en Internet es la suite TCP/IP, que define cómo se transmiten y reciben los mensajes. Incluye protocolos como IP (direccionamiento y enrutamiento), TCP/UDP (transporte), HTTP/HTTPS (web), SMTP/IMAP (correo), DNS (nombres de dominio), entre otros.

### Software de red
Aplicaciones que permiten interactuar con Internet: navegadores, clientes de correo, servidores web, sistemas de streaming, etc.

### Estructura jerárquica de Internet
**Capa de acceso (Edge):** conexiones de usuarios y empresas.

**Capa de núcleo (Core):** formada por los grandes proveedores internacionales (Tier 1) y regionales que interconectan redes mediante IXPs (Internet Exchange Points).



## 3. ¿Qué son las RFCs?
Son documentos donde se publican los estándares, protocolos, mejores prácticas y experimentos de Internet, que pueden tener distintos niveles de madurez (Proposed Standard, Internet Standard, Experimental, BCP, Historic, FYI).

## 4. ¿Qué es un protocolo?
El conjunto de conductas y normas a conocer,
respetar y cumplir no s´olo en el medio oficial ya establecido, sino tambi´en en el medio social, laboral, etc.

## 5. ¿Por qué dos máquinas con distintos sistemas operativos pueden formar parte de una misma red?
Dos máquinas con distintos sistemas operativos pueden formar parte de una misma red gracias al modelo en capas (Layering). Este modelo divide la complejidad en componentes reutilizables y asegura la interoperabilidad: las capas inferiores ocultan la complejidad a las superiores y ofrecen servicios a través de interfaces. Así, los cambios en una capa no afectan a las demás siempre que se mantenga la interfaz, lo que permite que sistemas distintos se comuniquen utilizando protocolos estándar comunes.

## 6. ¿Cuáles son las 2 categorías en las que pueden clasificarse a los sistemas finales o End Systems? Dé un ejemplo del rol de cada uno en alguna aplicación distribuida que corra sobre Internet.

### Clientes:

**Rol:** Son los sistemas que solicitan servicios o recursos a otros sistemas. Inician las comunicaciones y consumen la información o funcionalidades proporcionadas por los servidores.
Ejemplo en una aplicación distribuida (Navegación Web): En una aplicación de navegación web, tu computadora personal (PC), laptop, o smartphone actuando como un navegador web (como Chrome, Firefox, Safari) es el cliente. Solicita páginas web, imágenes, videos y otros recursos a un servidor web.

### Servidores:

**Rol:** Son los sistemas que proveen servicios o recursos a otros sistemas. Escuchan las solicitudes de los clientes y responden a ellas, ofreciendo la información o funcionalidades que tienen almacenadas o pueden generar.
Ejemplo en una aplicación distribuida (Navegación Web): En la misma aplicación de navegación web, el servidor web (por ejemplo, un servidor Apache o Nginx) que aloja el sitio web que deseas visitar es el servidor. Recibe la solicitud de tu navegador, procesa la petición y envía la página web solicitada de vuelta al cliente.

## 7. ¿Cuál es la diferencia entre una red conmutada de paquetes de una red conmutada de circuitos?
La conmutación de circuitos establece una conexión dedicada y exclusiva para toda la duración de la comunicación, mientras que la conmutación de paquetes divide los datos en pequeñas unidades que se envían de forma independiente y comparten los recursos de la red.

## 8. Analice qué tipo de red es una red de telefonía y qué tipo de red es Internet.

### Red de Telefonía (Tradicional)
**Tipo de Red:** Red de Conmutación de Circuitos.

**Análisis:**
Establecimiento de Conexión: Antes de que puedas hablar con alguien por teléfono (en una red telefónica tradicional, como la PSTN), se establece un circuito físico o lógico dedicado entre tu teléfono y el de la persona a la que llamas. Este circuito se mantiene abierto y exclusivo para esa conversación durante toda su duración.
Recursos Dedicados: Una vez que el circuito está establecido, los recursos de la red (como el ancho de banda) se reservan para esa llamada, incluso si hay silencios.
Calidad de Servicio: Esto garantiza una calidad de voz constante y predecible, ya que no hay competencia por los recursos una vez que la conexión está activa.
Ineficiencia: Si la conversación tiene pausas, los recursos dedicados se desperdician.
Referencia en el PDF: El PDF menciona "Redes de Conmutación de Circuitos" en la página 22 como una clasificación física de redes. Aunque no lo asocia directamente con la telefonía, este es el ejemplo clásico de este tipo de red.

### Internet
Tipo de Red: Red de Conmutación de Paquetes.
**Análisis:**
División de Datos: Cuando envías información a través de Internet (un correo electrónico, una página web, un video), esta información se divide en pequeños fragmentos llamados "paquetes".
Envío Independiente: Cada paquete se envía de forma independiente a través de la red. Pueden tomar diferentes rutas para llegar a su destino y no hay un circuito dedicado preestablecido.
Recursos Compartidos: Los recursos de la red (routers, enlaces) son compartidos por los paquetes de muchas comunicaciones diferentes. Esto permite una utilización muy eficiente de la infraestructura.
Flexibilidad y Escalabilidad: Esta arquitectura la hace extremadamente flexible y escalable, capaz de manejar una gran variedad de tipos de tráfico y un número masivo de usuarios.
Variabilidad en QoS: Debido a la naturaleza compartida y dinámica, la calidad de servicio puede variar (por ejemplo, el retardo puede fluctuar) dependiendo de la congestión de la red.

## 9. Describa brevemente las distintas alternativas que conoce para acceder a Internet en su hogar.
LAN,MAN,WAN,SAN,PAN???
**DSL (Digital Subscriber Line):** Usa líneas telefónicas de cobre existentes para internet de banda ancha.
**Cable (Coaxial):** Acceso a internet de alta velocidad a través de la infraestructura de TV por cable.
**Fibra Óptica (FTTH):** Conexión de muy alta velocidad y baja latencia usando cables de fibra de luz.
**Internet Móvil (4G/5G):** Acceso a través de redes celulares, ideal para portabilidad o zonas sin infraestructura fija.
**Satélite:** Conexión para áreas remotas sin otras opciones, con latencia alta.
**WISP (Wireless Internet Service Provider):** Internet inalámbrico de banda ancha mediante ondas de radio, común en zonas rurales.


## 10. ¿Qué ventajas tiene una implementación basada en capas o niveles?
**Divide la Complejidad:**
Reduce la complejidad general del sistema al dividirla en componentes más pequeños y manejables.
**Reusabilidad de Componentes:**
Permite la creación de componentes reusables.
**Abstracción (Oculta la Complejidad):**
Las capas inferiores "ocultan la complejidad" a las capas superiores, proporcionando un nivel de abstracción.
**Uso de Servicios a Través de Interfaces:**
Las capas superiores "utilizan servicios" de las capas inferiores a través de interfaces (similar a las APIs).
**Facilita el Desarrollo y la Evolución:**
Los cambios en una capa no deberían afectar a las demás si la interfaz se mantiene, lo que facilita el desarrollo y la evolución de los componentes de red.
Asegura la interoperabilidad entre diferentes sistemas y tecnologías.
Facilita el Aprendizaje, Diseño y **Administración:**
Simplifica el aprendizaje, el diseño y la administración de las redes.

## 11. ¿Cómo se llama la PDU de cada una de las siguientes capas: Aplicación, Transporte, Red y Enlace?
Aplicacion: Datos
Transporte: Segmento
Red: Paquete
Enlace de datos: Trama/frame

## 12. ¿Qué es la encapsulación? Si una capa realiza la encapsulación de datos, ¿qué capa del nodo receptor realizará el proceso inverso?
Encapsulación es el proceso en el que se agrega informacion de control especifica a los datos que va a recibir la capa superior. Esta informacion se utiliza para gestionar la comunicacion entre capas.
Siempre la capa de arriba es aquella que realiza lo inverso de encapsulacion para poder utilizar la informacion de la de abajo.

**Ejemplo:**

La Capa Física recibe los bits del medio y los reconstruye en una Trama para pasarla a la Capa de Enlace.

La Capa de Enlace recibe la Trama, remueve su encabezado y final, y extrae el Paquete para pasarlo a la Capa de Red.

La Capa de Red recibe el Paquete, remueve su encabezado, y extrae el Segmento para pasarlo a la Capa de Transporte.

La Capa de Transporte recibe el Segmento, remueve su encabezado, y entrega los Datos originales a la Capa de Aplicación.

## 13. Describa cuáles son las funciones de cada una de las capas del stack TCP/IP o protocolo de Internet.
**Capa de Aplicación:** Interfaz entre la red y las aplicaciones del usuario. Provee los servicios de red directamente al usuario final.

**Capa de Transporte:** Establece una comunicación confiable de extremo a extremo entre aplicaciones. Gestiona el control de flujo y la corrección de errores.

**Capa de Internet:** Encamina los paquetes a través de múltiples redes. Se encarga del direccionamiento lógico (IP) y la determinación de la mejor ruta.

**Capa de Enlace de Datos:** Gestiona la comunicación entre dispositivos en la misma red local. Se encarga del direccionamiento físico (MAC) y el acceso al medio.

**Capa Física:** Transmite la secuencia de bits brutos (señales) a través del medio físico (cable, aire, fibra).

## 14. Compare el modelo OSI con la implementación TCP/IP.

### Similitudes
Ambos se dividen en capas.
Ambos tienen capas de aplicaci´on, aunque incluyen servicios
distintos.
Ambos tienen capas de transporte similares.
Ambos tienen capa de red similar pero con distinto nombre.
Se supone que la tecnolog´ıa es de conmutaci´on de paquetes
(no de conmutaci´on de circuitos).
Es importante conocer ambos modelos.

### Diferencias
TCP/IP combina las funciones de la capa de presentación y de sesión en la capa de aplicación.
TCP/IP combina la capas de enlace de datos y la capa física del modelo OSI en una sola capa.
TCP/IP más simple porque tiene menos capas.
Los protocolos TCP/IP son los est´andares en torno a los cuales se desarrolló Internet, de modo que la credibilidad del modelo
TCP/IP se debe en gran parte a sus protocolos.
El modelo OSI es un modelo “más” de referencia, teórico, aunque hay implementaciones.

