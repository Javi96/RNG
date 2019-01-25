# Tema 5: Redes definidas por software

En redes segmentadas tenemos que las LAN se comunican de manera jerárquica pero sigue habiendo un único dominio de difusión, no hay restricciones de tráfico y puede haber puertos que no se utilicen.

La solución la encontramos en las VLAN donde las máquinas de un mismo grupo pertenecen a la misma LAN mientras que un conutador separa el tráfico de cada grupo. Además un equipo puede pertenecer a varias LAN y la configuración de las mismas puede cambiar de forma dinámica.

En este punto tenemos dos opciones:

 - VLAN basadas en puertos donde el administrador define que puerto de cada conmutador pertenece a cada LAN (implica integrar comunicación entre varios conmutadores).
 - VLAN basadas en etiquetas: cada trama tiene una etiqueta **tag** que indica a que VLAN (grupo) pertenece. Esto permgite unir varios conmutadores entre sí.

# SDN: Etiquetas 802.1Q

Tienen 32 bits y 2 campos:
 
- tipo (16 bits): 0x8100.
- etiqueta (16 bits):
	- prioridad (4 bits). 
	- VLAN identificador (12 bits).

En SDN tenemos conmutadores, los cuales son gestionables y se encargan de añadir etiquetas (solo el primero y el último) a lo largo del recorrido.

# SDN: Redes definidas por software

Permiten separar la capa de control de la capa de datos para que sea gestionable de forma versátil y estándar.

## SDN: Open VSwitch

Software abierto que implementa un switch virtual entre entornos virtuales y que reenvía tráfico entre las VM que se ejecutan en una misma máquina.

Tenemos una estructura de tres capas:

- Esterna: nodo de control que gestiona tanto el servidor como los switches activos.
- Usuario: contiene ovsdb-server y ovs-vswitchd.
- Kernel: núcleo de control.

## SDN: ovsdb-server

Mantiene la base de datos de Open vSwitch, es decir, contiene toda la información relativa a puentes, interfaces, túneles y direcciones de los controladores. Todo está configurado en memoria no volátil.

## SDN: ovs-vswitchd

Es el gestor de Open vSwitch. Mantiene actualizada la información de la base de datos, configura el módulo OVS del kernel y gestiona los interfaces virtuales.