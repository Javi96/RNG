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


## BGP - Atributos de ruta

Son usados para dirigir el proceso de selección de ruta. Pueden ser:

- **bien conocidos y obligatorios**: el atributo debe ser reconocido en todas las implementaciones de BGP y ha de estar presente en cada mensaje update.
- **bien conocidos y no obligatorios**: estar en el update es opcional.
- **opcionales transitivos**: no tienen que estar en todas las implementaciones BGP pero en caso de estar han de ser reenviados a otros vecinos.
- **opcionales no transitivos**: no necesitan ser reenviados a otros vecinos.

Los atributos más comunes son los siguientes:

- **ORIGIN**: bien conocido y obligatorio. Identifica el origen de una ruta, el cual puede ser iBGP (la ruta se genera en el propio SA), eBGP (la ruta es externa al SA) o incomplete (la ruta es inyectada por un método externo).
- **AS_PATH**: bien conocido y obligatorio. Es una lista con los SA que forman la ruta para alcanzar el destino. (100 - 200 - 300 || el camino para ir es 300 -> 200 -> 100). Permite el control de bucles si el ASN del propio encaminador ya estaba en la lista.
- **NEXT_HOP**: bien conocido y obligatorio. Indica la IP del encaminador frontera que será el próximo salto en eBGP. En iBGP no hay cambios pero el protocolo interno elige el encaminamiento dentro del SA.
- **LOCAL_PREF**: bien conocido no obligatorio. Indica la preferencia local para cada ruta dentro del SA. Solo se tiene en cuenta a nivel de iBGP pero no eBGP.
- **ATOMIC_AGGREGATE**: bien conocido no obligatorio. Este atributo indica agregado de rutas, lo cual ocurre cuando un mismo encaminador recibe varios anuncios para redes de distintas rutas, en este caso puede optar por anunciarlas como tal o por agregarlas con un prefijo más corto.
- **AGGREGATOR**: opcional transitivo. Indica el último ASN que hizo agregado de rutas e indica la ip del encaminador que lo hizo.
- **MULTI_EXIT_DISC**: opcional no transitivo. En caso de que dos SA estén unidos por dos o más rutas este atributo indica cual es preferible para encaminar a través de él.

## BGP - Proceso de decisión

Hay que tener en cuenta que el encaminamiento es a nivel local!!.

1. se elige el anuncio con mayor LOCAL_PREF.
2. si hay empate se elige la ruta originada localmente.
3. si hay empate se usa el AS_PATH más corto.
4. si hay empate se usa el ORIGIN menor, es decir, se prima antes IGP que EGP y que Incomplete.
5. si hay empate se usa el menor MED.
6. si hay empate entonces eBGP antes que iBGP.
7. si hay empate la ruta con coste IGP menor al siguiente salto.
8. si hay empate vecino BGP con id menor.
9. si hay empate vecino BGP con IP menor.
