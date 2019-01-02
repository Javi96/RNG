# Tema 2: OSPF

## RIP
En un protocolo que usa el vector de distancia y como métrica el número de saltos. Es útil en redes pequeñas y soporta hasta 16 saltos. Además se desaconseja su uso al tener un número de saltos reducido (no escala bien en redes grandes), no gestiona máscaras de subred de longitud variable, las difusiones periódicas de la tabla de rutas completa consumen gran cantidad de ancho de banda. 

A esto añadirmos que las redes son planas (no hay áreas, límites ni ruteo con clases ni agregación), y no tiene en cuenta el coste de los caminos sinó que usa el menor número de saltos (no es óptimo).

## OSPF
Es un protoclo de encaminamiento interno y es el más usado. A diferencia de RIP este se basa en el estado del enlace y divide el SA en áreas, de forma que cada área se conecta a la troncal de dicho SA.

OSPF solventa la mayoría de problemas de RIP:

- no hay número máximo de saltos.
- los cambios en la topología se propagan al instante no periódicamente.
- división en áreas.
- autenticación del ruteo.
- permite indexar rutas extermas al propio SA.

## OSPF - Problema de la inundación

El algoritmo usado en OSPF es SPF y está basado en el algoritmo de Dijkstra, por lo que necesita la información completa de la topología por lo que es necesario controlar el proceso de intercambio de información. 

Cuando se detecta un cambio en la red se lleva a cabo una inundación por la misma con la nueva topología, lo cual puede desembocar en ciclos. Para evitar esto cada nodo compara el router-id del mensaje con la lista de router-id que tiene de otros envíos de forma que si ya estaba en ella se para la inundación por parte de dicho nodo. Además también se tiene en cuenta la edad del anuncio de manera que al detectar un mensaje demasiado viejo se para la inundación.



## OSPF - Establecimiento de vecindad

Los encaminadores que están en el mismo segmento pueden convertirse en vecinos mediante el interecambio de mensajes hello. Los paquetes de saludo se envían
periódicamente fuera de cada interfaz mediante IP Multicast de forma que los encaminadores se convierten en vecinos al enviar y recibir dicho mensaje garantizando así una comunicación bidireccional. 

La negociación entre vecinos se aplica sólo
a la dirección primaria. Las direcciones secundarias se pueden configurar en una interfaz con la
restricción de que deben pertenecer a la misma área que la dirección primaria.
Dos routers no se convertirán en vecinos a menos que coincidan en lo siguiente:

- Id. de área: dos routers que tienen un segmento común; sus interfaces deben pertenecer a la
misma área en ese segmento. Claro que las interfaces deben pertenecer a la misma subred y
deben tener una máscara similar.
- Autenticación: OSPF permite la configuración de una contraseña para un área específica. Los
routers que quieren convertirse en vecinos tienen que intercambiar la misma contraseña en
un segmento determinado.
- Intervalo Hello e Intervalo Muerto: OSPF intercambia paquetes de saludo en cada segmento.
Esta es una forma de keepalive que los routers utilizan para reconocer su existencia en un
segmento y para elegir un router designado (DR) en segmentos de acceso múltiple. El
intervalo Hello especifica el periodo de tiempo, en segundos, entre los paquetes Hello que un
router envía sobre una interfaz OSPF. El intervalo muerto es la cantidad de segundos durante
los cuales los paquetes Hello de un router no han sido vistos, antes de que sus vecinos
declaren desactivado al router OSPF.
El OSPF requiere que estos intervalos sean exactamente los mismos entre dos vecinos. Si
cualquiera de estos intervalos es diferente, estos routers no se convertirán en vecinos en un
segmento particular.
- Indicador de área Stub: Para convertirse en vecinos, dos routers también deben coincidir en
el indicador de área stub de los paquetes Hello. 


## OSPF - Encaminador designado

En OSPF tenemos un gran número de mensajes intercambiados a lo largo de la red. Para evitar la congestión en una red de difusión marcamos uno de los encaminadores como DR, además de otro como BDR o buckup por si el primario se cae, de forma que todos los encaminadores de la red crean adyacencias solo con el DR y el BDR.

La elección del DR y BDR se lleva a cabo a través del protocolo Hello. Los paquetes de saludo se
intercambian a través de los paquetes en cada segmento de forma que el router con la prioridad OSPF más alta en un segmento se convierte en el DR para dicho
segmento. El mismo proceso se repite para BDR mientras que en caso de empate ganará el router con el RID
más alto. El valor predeterminado para la prioridad de interfaz OSPF es uno.


## OSPF - Establecimiento de adyacencias

La adyacencia es el paso siguiente al establecimiento de vecinos. Los routers
adyacentes son routers que van más allá de un simple intercambio de mensajes Hello y actúan en el
proceso de intercambio de base de datos. Para reducir la cantidad de intercambio de información
en un segmento determinado, OSPF selecciona un router como Router designado (DR) y un
router como Router designado de respaldo (BDR) en cada segmento, de forma que se elige
el BDR como mecanismo de respaldo en caso de que falle el DR.

La idea detrás de esto es que
los routers tienen un punto central de contacto para el intercambio de la información. En lugar de
que cada router intercambie actualizaciones con cada router en el segmento, todos los routers
intercambian información con el DR y el BDR. El DR y el BDR confían la información al resto. 





## OSPF - Creación de adyacencias

Para establecer acyacencias en necesario:

- intercambiar la descipcion de datos de enlace, es decir, intercambiar los LSA locales al encaminador.
- cada encaminador solicita a sus vecinos los LSA más recientes.

El proceso de construcción de adyacencias tiene efecto después de que se han completado
varias etapas. Los routers adyacentes tendrán la misma base de datos de estados de link y se precisa de varias fases para ello:

- **Down**: no se ha recibido información por el enlace.
- **Attempt**: en redes sin DR. El vecino parece inactivo y envían mensajes hello para intentar establecer vecindad.
- **Init**: se ha recibido un hello pero el RID del encaminador local no está en la lista.
- **2-ways**: Hay comunicación bidireccional con un vecino. El router se ha visto a sí mismo
en los paquetes de saludo provenientes de un vecino. Al final de esta etapa, se habría
realizado la elección del DR y el BDR.
- **Exstart**: se fija un número de secuencia para que los encaminadores reciban siempre la información más reciente. Uno de ellos se marca como primario y otro como secundario de forma que uno consulta la información del otro.
- **Exchange:**: los encaminadores intercambian sus bases de datos. En este estado, los paquetes se podrían
distribuir en forma de inundaciones a otras interfaces del router.
- **Loading**: los encaminadores están terminando con el intercambio de información.
- **Full**: en este estado, la adyacencia se ha completado. Los encaminadores vecinos son
completamente adyacentes.

## OPSF - Definiciones

- **Sistema autónomo**: conjunto de redes conexas y encaminadores bajo el control de una o más organizaciones que trabajan con un mismo sistema de encaminamiento.
- **Área**: conjunto lógico de redes y encaminadores donde cada área mantiene la información topológica de la misma, luego no se conoce la topología externa pero sí las rutas a destinos externos.
- **Área troncal**: área que contiene toda la información topológica de la red OSPF. Es necesario que haya una por cada red y todas las áreas inyectan su información de encaminamiento en ella.
- **Encaminadores internos**: pertenecen a un sólo área por lo que solo conocen la información topológica propia.
- **Encaminadores troncales**: en el área troncal.
- **Encaminadores frontera de área**: interconectan un área con la troncal y mantienen las bases de datos de la topología de ambas áreas de forma separada.
- **Encaminadores frontera del SA**: intercambian información de encaminamiento entre la propia red OSPF y otros algoritmos (RIP, BGP...). 
- **Base de datos de estado del enlace**: información topológica del área. Cuenta con los dispositivos, los enlaces físicos y las rutas hacia destinos exteriores.
- **Link State Advertisement, LSA (Anuncio de estado del enlace)**: contienen información sobre un elemento de la red (encaminador, enlace o ruta). Son intercambiados entre dos encaminadores a la hora de establecer la adyacencia y cuando se genera o modifica un LSA se debe anunciar dicho cambio por todo el área. Esto es llevado a cabo mediante inundación pero de forma controlada y fiable. Los LSA pueden ser de varios tipos:
	- **encaminador**: describe el estado de los interfaces de cada encaminador.
	- **red**: contiene los encaminadores conectados a una red de difusión. Es generado por el DR.
	- **resumen**: lo genera un ABR y la información es inyectada a la troncal y luego a las demás áreas. Tenemos dos tipos en función de si las rutas que contienen destinos inter-áreas (dentro del SA) o destinos a encaminadores ASBR.
	- **externos**: rutas fuera de la red OSPF. Son generados en los ASBR y se difunden por todas las áreas de la red.

## OSPF - Redistribución de rutas

Las rutas externas se incluyen en dos categorías, tipo externo 1 y tipo externo 2. La diferencia
entre ellas es la forma en que se calcula el costo (métrica) de la ruta. El costo de una ruta tipo 2
es siempre el costo externo sin importar el costo interno para alcanzar esa ruta. Un costo tipo 1 es
la suma del costo externo y del costo interno que se utilizó para alcanzar esa ruta. Una ruta tipo 1
siempre es preferible sobre una ruta tipo 2 para el mismo destino.

Si las rutas externas son ambas de tipo 2 y los costos externos a la red de destino son iguales,
entonces se selecciona como mejor trayecto el que presenta un menor costo hacia el ASBR.

## OSPF - Áreas stub

No se permite que las redes
externas, como aquellas redistribuidas desde otros protocolos a OSFP, inunden el área stub. El
ruteo desde estas áreas al mundo exterior se basa en una ruta predeterminada.

Un área puede calificarse como stub cuando hay un único punto de salida desde ese área o si el
ruteo hacia afuera del área no tiene que tomar un trayecto óptimo. 

Otras restricciones del área stub es que no se puede usar como un área de tránsito para los links
virtuales. Además, un ASBR no puede ser interno a un área stub. Se realizan estas restricciones
porque un área fragmentada está configurada principalmente para no transportar rutas externas y
cualquiera de las situaciones anteriores hace que links externos sean inyectados a dicha área. El
backbone, por supuesto, no puede configurarse como un área stub.

Todos los routers OSPF dentro de un área stub deben configurarse como routers stub. Esto es
así porque cuando se configura un área como stub, todas las interfaces que pertenecen a esa
área comienzan a intercambiar paquetes de saludo con un indicador que señala que la interfaz es
stub. 

En realidad, esto es solo un bit en el paquete Hello (bit E) que se fija en 0. Todos los routers
que tienen un segmento común deben coincidir en dicho indicador. Si no coinciden, no se
convertirán en vecinos y el ruteo no tendrá efecto.

## OSPF - Áreas totally stub
Se denomina "zonas totalmente stub" a una extensión hacia las zonas stub. Cisco indica esto al
agregar una palabra clave “no-summary” a la configuración del área Stub. Un área totalmente
congestionada es aquélla que bloquea el paso de las rutas externas y las rutas de resumen (rutas
intra-área) al área. De esta manera, las rutas dentro del área y el valor predeterminado de 0.0.0.0
son las únicas rutas inyectadas en esa área.

## OSPF - Áreas NSSA

Un área NSSA (not-so-stubby area) es un área stub que
contiene un ASBR. El ABR que la une con la troncal no inyecta rutas externas al
área, pero propaga las generadas por el ASBR dentro de la
NSSA en la troncal.

## OSPF - Agregado de rutas

Es el proceso de resumir varias redes consecutivas en una
sola. Dos tipos de agregados:

- Inter-área: se realiza en el ABR para agregar los anuncios desde
el área.
- Externo: se realiza en el ASBR.