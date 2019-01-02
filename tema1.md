# Tema 1: IP de nueva generación: IPv6
## Introducción
IPv4 viene con varios problemas:

+ el tamaño de las direcciones es de 32 bits, lo cual provoca escasez de direcciones. Para solucionarlo se usan intranets privadas y tablas NAT.
+ las redes se organizan en clases lo cual hace que se desaproveche un gran número de direcciones IP. Para evitarlo se usa la notación CIDR.
+ en IPv4 la longitud de la cabecera (y el formato del datagrama) es variable por lo tanto es más difícil de procesar y hay que recurrir a la fragmentación del mensaje.
+ no es seguro aunque IPSec trata de solucionarlo.
+ tiene multicast pero no es demasiado útil.

IPv6 trata de solucionar estos defectos:

+ direcciones de 128 bits.
+ formato de la cabecera simplificado y fácilmente ampliable.
+ encapsulamiento jerárquico.
+ autoconfiguración de los interfaces.
+ soporte para tráfico en tiempo real (VoIP, vídeo bajo demanda...)
+ opciones de seguridad.

## Direccionamiento IPv6

Tenemos 3 tipos de direcciones:

+ unicast: especifica una interfaz única. Un paquete enviado a una dirección de este tipo viaja de un sistema principal al sistema principal de destino. Estas a su vez tienen definidos 3 ámbitos:
	+ enlace local: solo es válida en el enlace al que está conectado el interfaz de red (un cable al router).
		- dirección: FE8 -- 1111 1110 10
	+ sitio local: la dirección solo es válida dentro de la red de la organización (universidad).
		- dirección: FEC -- 1111 1110 11
	+ global: la dirección es válida en todo internet.
		- dirección: 2-3 -- 001
+ multicast: especifica un conjunto de interfaces, posiblemente en ubicaciones distintas. El prefijo que se utiliza para una dirección de difusión múltiple es ff. Cuando se envía un paquete se manda una copia del mismo a cada miembro del grupo.
	- dirección: FF -- 1111 1111
	- tienen varios ámbitos:
		- nodo local -- FF01
		- enlace local -- FF02
		- sitio local -- FF05
		- organización -- FF08
		- global -- FF0E
	- multicast para encaminadores:
		- FF01::2 -- encaminadores del nodo local
		- FF02::2 -- encaminadores del enlace local
		- FF05::2 -- encaminadores del sitio local
		- FF02::9 -- encaminadores RIP del enlace local
	- multicast para computadores:
		- FF01::1 -- computador del nodo local (todos los interfaces del nodo local)
		- FF02::1 -- computadores del enlace local
+ anycast: especifica un conjunto de interfaces, posiblemente en ubicaciones diferentes, que comparten una única dirección. Un paquete enviado a una dirección anycast se transmite sólo al miembro más próximo del grupo de difusión.

## Datagrama IPv6

La cabecera contiene los siguientes campos:

- (4 bits) version: version de ip (0110).
- (6+2 bits) traffic class: define el tipo de tráfico del datagrama. Permite definir la priorización de paquetes en base a:
	- (6 bits) DSCP (Differentiated Services Code Point): proporciona varios códigos para marcar el comportamiento en cada salto de los paquetes que pertenecen a un servicio en concreto.
	- (2 bits) ECN (Explicit Congestion Notification): permite a los encaminadores poner indicadores de congestión en vez de solo descartar los paquetes.
- (20 bits) flow label: entendemos por flow un conjunto de paquetes con un origen y destino comunes los cuales requieren un manejo específico a la hora de atravesar los encaminadores (como servicio en tiempo real). Esto permite a los encaminadores saber por donde han de mandar el flujo sin tener que examinar el resto del paquete.
- (16 bits) payload length: tamaño del datagrama sin tener en cuenta la cabecera. 0 si hay net header con jumbo payload length.
- (8 bits) next header: indica el tipo de cabecera que viene después de la cabecera IP básica o el protocolo de la capa superior. En estas cabeceras se almacena información adicional y dicho campo puede valer:
	- 0 -- hop-by-hop: es exáminada por todos los nodos de la ruta, incluídos origen y destino. Si está debe ir siempre después de la cabecera IPv6. Esta cabecera cuenta con un campo tipo que puede valer:
		- pad1.
		- padn.
		- router alert: el datagrama contiene datos relevantes para el encaminador.
		- jumbo payload length: indica el tamaño del datagrama contando con los 40 bits de la cabecera estándar. Para paquetes con más de 65536 bits.
	- 43 -- router header: permite al emisor definir una serie de nodos que el datagrama a de atravesar. Puede ser útil a la hora de evitar rutas lentas o más costosas. Tiene varios campos:
		- segments left: número de nodos que quedan por atravesar.
		- direcciones: direcciones de los nodos intermedios. Se cambia la i-ésima conforme se va saltando un nodo a otro can la actual, así se guarda un camino de los encaminadores atravesados.
	- 44 -- fragment header: determina la fragmentación (MTU) que hay que aplicar al datagrama.
	- 51 -- autentication header: hace las veces de elemento de seguridad en el envío del datagrama. Con él se garantiza tanto la integridad como la autenticidad del mismo. Tiene 3 parámetros:
		- Security Parameters Index (SPI): etiqueta identificadora de la cabecera.
		- Sequence Number (SN): contador para evitar ataques de repetición.
		- Integrity Check Value (ICV): resultado de aplicar el alg. de verificación de integridad elegido para el datagrama.
- (8 bits) hop limit: número máximo de encaminadores que el paquete puede atravesar. Cómo TTL en IPv4 pero no se basa en segundos.
- (128 bits cada una) IPv6 origen e IPv6 destino.


## IMCPv6 - descubrimiento de vecinos

En IPv4 se emplea ARP con el objetivo de asociar una dirección IPv4 con direcciones MAC. Esto en IPv6 es substituído por el descubrimiento de vecinos, donde se usa el protocolo ICPv6.

El descubrimiento de vecinos es una función de ICMPv6 que permite identificar otros hosts o encaminadores. Supongamos un caso práctico donde un host quiere mandar un mensaje a otro equipo. Para ello mandará un mensaje a FF02::2 (todos los host, enlace local) con una cabecera de extensión con un mensaje ICMPv6. Este mensaje (neighbour solicitation) llegará a todos los equipos a los que la máquina origen esté conectado, pero solo responderá aquella máquina con la IPv6 indicada en el cuerpo de la cabecera ICMPv6. Dicho equipo responderá con otro mensaje (neighbour advertisement) a la máquina en cuestión.

## IMCPv6 - descubrimiento de encaminadores

Se emplea un procedimiento similar al caso anterior. Ahora el host envía un mensaje (router solicitation) a la dirección FF02:1 (todos los encaminadores, enlace local) el cual es recibido por todos los encaminadores a los que esté conectado el host de forma que responderá (router advertisement) algún encaminador si es que había alguno disponible.

## IMCPv6 - autoconfiguración

En IPv6 no es necesario un protocolo como DHCP en IPv4. Aquí basta con el la máquina conozca el prefijo de red. Para ello es necesario dicho prefijo (64 bits) y la MAC del host (48 bits). Para alcanzar los 128 bits de una dirección IPv6 son necesarios otros 16 bits, los cuales se consiguen añadiendo a la MAC FFFE. Para ello se parte la MAC en dos y se añade dicho elemento y además se invierte el séptimo bit de la misma. Por último se concatenan el prefijo y el resultado de operar con la MAC. 

Esto genera problemas de privacidad pues sabiendo el prefijo de red y con ingeniería inversa en muy fácil localizar los equipos que usan dichas direcciones IPv6.

Para ello surgen las extensiones de privacidad donde el identificador de la interfaz es generado de forma pseudoaleatoria. En este caso se combina la MAC con una semilla aleatoria y se les aplica un algoritmo de hash (MD5), aunque esto genera direcciones inconsistentes que no son fáciles de trabajar al ser cambiantes.

Esto se soluciona con RID el cual es generado con una función criptográfica usando como base el prefijo, la interfaz de la máquina, el identificador de la red, la clave secreta y un contador. Esto proporciona direcciones IPv6 pseudoaleatorias pero estables, facilitando la gestión de la red y asegurando la privacidad.

## Secure Neighbour Discovery

Es una extensión de seguridad para el protocolo de descubrimiento de vecinos, en concreto se encarga de mitigar los ataques de denegación de servicio y la suplantación de identidad (tanto para nodos como encaminadores). Este protocolo se encarga de asegurar que quien dice poseer una drección IPv6 realmente la posee y para ello emplea parejas de claves pública-privada. 

De este modo cada encaminador debe tener un certificado válido y cada nodo una lista de autoridadez reconocidas, de forma que los mensajes de descubrimiento y de anuncio van firmados. Esto se implementa con RA-Guard de forma que un switch se encarga de filtrar ambos mensajes con el fin de evitar que haya nodos que se hagan pasar por encaminadores a la hora de mandar Router Advertisement. Pueden hacerlo manejando la dirección de origen o el puerto.

## Mecanismos de transición

Hasta el despliegue de IPv6 han de convivir ambos protocolos. Por ello se han propuesto varias opciones:

- dual stack: es la forma más sencilla para comunicar IPv6 con IPv4. Para ello un nodo puede mardar y recibir tanto paquetes IPv6 como datagramas IPv4 en base al equipo con el que se esté comunicando. Este método sin embargo no resuelve la falta de direcciones IPv4.
- túneles IPv6 sobre IPv4: en este caso los paquetes desde host con IPv6 o IPv6/IPv4 son enviados a través de redes con IPv4. Para ello se encapsula el paquete IPv6 en un datagrama IPv4 y se indica IPv6 en el campo protocol value con valor 41 en la cabecera IPv4. Hay dos tipos de túneles:
	- automáticos: comunicación entre nodos IPv6 a través de redes IPv4. Presentan problemas de bajo rendimiento y seguridad. Los 6to4 necesitan un 6to4-relay entre cada red para reenvío.
	- manuales: son configurados por el admin de la red y necesitan un tunnel broker para conectarse con Internet6.


## Plan de despliegue

En IPv6 cada enlace debe tener asignado un prefijo /64 o al menos un /48 (65536 redes internas) para el usuario final. Aquí no nos centramos en como ahorrar direcciones si no en como facilitar el agregado (ampliar segmentos cecidos). Para ello pasamos de asignar segmentos continuados si no que asignamos segmentos mediante un árbol binario de manera que siempre habrá espacio para expandir alguna red si fuese necesario.

Aun así esto nos da direcciones asignadas a las sedes las cuales no son consecutivas, de forma que hacemos como en SDN: direcciones pseudoaleatorias.