# Tema 3: BGP

Internet se construye mediante la interconexión de SA. Cada SA está identificado por un número único llamado ASN. Tenemos varios conceptos interesantes:

- **vecino**: aquellos SA y encaminadores que intercambian información de encaminamiento.
- **announce**: envío de información de encaminamiento a un vecino.
- **accept**: uso de la información recibida desde un vecino.
- **peers**: encaminadores de SA vecinos o inter-as que intercambian información de encaminamiento y políticas.

Para poder envíar información de encaminamiento de AS1 a AS2 es necesario que:

- AS2 se anuncie a AS1 (le pasa su ruta).
- que AS1 acepte el anuncio de AS2 para empezar el flujo de datos.

Por lo tanto el flujo de los datos es en sentido opuesto a los anuncios de las rutas. 


## BGP - Tipos de sistemas autónomos

Los sistemas autónomos dentro de BGP pueden ser:

- **multihomed**: está conectado a las de un SA de forma que no cae si falla otro SA al que está conectado. No permite el tráfico de tránsito.
- **transit**: hace de puente entre varios SA de forma que permite el tránsito de tráfico a través de él. Los proveedores de internet son SIEMPRE sistemas autónomos de tráfico al igual que las troncales de internet.
- **stub**: sistemas autónomos clientes que solo están conectados a otro SA. Por definición no permiten el tránsito aunque si pueden haber establecido relación con otro sistema autónomo que no esté en los servidores públicos de rutas.


## BGP - Whois

Es una base de datos que mantiene información sobre las redes de internet que nos permite hacer peticiones para determinar el propietario de un nombre de dominio o una dirección IP.

- whois 200.10.228.135 (ip)
- whois AS27733 (SA)
- whois PY-CNCO-LACNIC (identificador)


## BGP - Encaminamiento inter-AS

IPv4 permite encaminamiento en dos niveles (netid y hostid) de forma que hay que mantener en las tablas de encaminamiento una entrada para cada destino de red, lo cual genera una tabla enorme que se vuelve insostenible.

Para solventar esta situación se usa CIDR de forma que una sola entrada en la tabla de encaminamiento agrupa un gran número de redes e identifica al RIR que maneja dicha red.

En esta situación tenemos que cada región controla varios prefijos de red de forma que en cada una de ellas se distribuyen dichos prefijos a las organizaciones particulares. Esto provoca que el encaminamiento entre sistemas autónomos no sea trivial, de forma que para definir las políticas de encaminamiento de los mismos tenemos que tener en cuenta varios factores:

- las organizaciones que gestionan cada SA deben estaballecer acuerdos particulares.
- hay factores económicos involucrados.
- es necesario que haya conectividad física.
- la política influye.

Esto hace que un encaminamiento basado en el siguiente salto no sea útil ya que es muy probable que nos interese no atravesar ciertos SA.


## BGP - Mensajes

- **open**: es el primer mensaje que se envía tras el establecimiento de la conexión TCP y se encarga de informar a los vecinos sobre la versión del protocolo, el número de SA y el identificador del proceso BGP. Además también indica el tiempo durante el que se va a mantener abierta la sesión BGP (90 seg) de forma que si vale 0 la conexión no caduca. Cuando se ha enviado este mensaje el emisor queda a la espera de un mensaje keepalive.
- **keepalive**: sirve de confirmación para el mensaje open. Si la sesión BGP va a ser limitada es necesario mandar un mensaje keepalive cada cierto tiempo (30 seg) para indicar que la sesión continúa. Con esto se evita la congestión de la red.
- **notification**: cierra la sesión BGP, es decir, se anulan las rutas apredidas durante dicha sesión. Además manda un código de error en caso de que no se haya recibido un mensaje, o faltase algún keepalive dentro del hello time (90 seg).
- **update**: intercambia información de encaminamiento como las rutas a eliminar, los atributos de las mismas o los prefijos de red accesible y su longitud. Este mensaje se envía solo si hay algún cambio en la topología y su recepción provoca que el proceso BGP se active para modificar las tablas de rutas y enviar a su vez otro update a los demás vecinos.