# Tema 4: MPLS

Las redes MPLS cuentan con nodos LSR de forma que cuando un paquete entra a un dominio MPLS es clasificado con una FEC, donde cada una de ellas tiene asociada una etiqueta de manera que el encaminamiento del paquete se hace en base a dicha etiqueta.

Cada FEC se define en función del tráfico del flujo y su asociación se realiza en el LSR de ingreso al dominio MPLS. Las etiquetas tienen significado local.

# MPLS - Funcionamiento

## MPLS - Antes del reenvío

Primero se ha de establecer la ruta LSp para el paquete. Para ello se han de especificar los parámetros de QoS para la ruta (recursos para el flujo así como política de colas y descartes). Se a de indicar el encaminamiento fuera del dominio MPLS (OSPF e iBGP) y se han de distribuir las etiquetas a lo largo del LSP. Para ello tenemos dos opciones, LDP y RSVP-TE.

## MPLS - Ingreso del datagrama en dominio MPLS

Se examinan los requisitos de QoS del paquete y se asocia una FEC (como consecuencia también una LSP, en caso de no existir dicha LSP se crea tomando ancho de banda de las demás) y se añade al datagrama la etiqueta que corresponda con la FEC.

## MPLS - Durante el reenvío

Cada LSR quita la etiqueta entrate, pone la local y reenvía el datagrama al siguiente LSR en función de la tabla de rutas.

## MPLS - Al salir del dominio MPLS

Se elimina la etiqueta y se reenvía el paquete. El encargado de eliminar la última etiqueta es el LSR previo al LER (nodo de salida), es decir, el penúltimo del LSR. Esto se hace así para evitar procesar dos veces el datagrama en el LER, una para desetiquetar y otra para reenviar.

# MPLS - Formato de la etiqueta

Dispone de 32 bits, de los cuales 20 van para el valor de la etiqueta. Tiene otros campos como exp para extender otros, S que vale 1 cuando se trata de la última etiqueta apilada y TTL con 8 bits que es el tiempo de vida del datagrama.

Dentro de la LSR ningún encaminador examina la cabecera IP, por lo tanto los campos TTL y Hop Limit no se modifican en ningún momento, lo cual es un problema ya que no se puede determinar cuando hay que descartar un datagrama. Para ello cuando el datagrama ingresa en el dominio MPLS se copia el campo TTL o Hop Limit de la cabecera IP en el campo TTL de la etiqueta MPLS.

Con cada reenvío se decrementa el TTL y funciona igual que siempre. A la hora de salir del dominio MPLS, si no se ha descartado el datagrama (TTL != 0), se copia el valor del campo TTL de la etiqueta MPLS en la cabecera IP y se reenvía el datagrama sin etiquetar.

# MPLS - Apilado de etiquetas

En MPLS se permite apilar etiquetas cuando un mismo dominio vaya a atravesar varios dominios. Para ello un el primer LSR puede hacer añadir una etiqueta del nuevo dominio al datagrama sobre las ya existentes (de otro dominio distinto) y quitarla en el último LSR. Estas operaciones no alteran las etiquetas externas.

# MPLS - Ingeniería de tráfico

Los algoritmos tradicionales como OPSF ya son capaces de definir rutas de forma dinámica, sin embargo los datagramas siempre siguen la misma ruta aunque esta se congestiones, ni se pueden distingir paquetes en función de sus necesadades. Para ello MPLS introduce varios campos como el ancho de banda necesario o la máxima latencia con el fin de paliar estas situaciones.

Se utiliza RSVP-TE para hacer un encaminamiento basado en restricciones. Primero se elige el camino más corto que permite asignar el ancho de banda necesario a la LSP. Si este camino existe se distribuyen las etiquetas necesarias y se toma el ancho de banda de la LSP del disponible en los enlaces.

También se pueden dar pesos a las LSP de forma que en caso de necesitar más ancho de banda o una LSP se puede expropiar a aquellas con menor peso, de forma que la expropiada debe redirigir su flujo por otro camino.

No obstante el calculo del ancho de banda no es trivial pues al fin y al cabo las redes son conmutación de paquetes y éste puede variar de manera drástica e impredecible. Para ello tenemos dos vías, predicción con modelos o mediciones en tiempo real.

# MPLS - Ajuste dinámico del ancho de banda

El ajuste del ancho de banda sigue un algoritmo sencillo. Primero se toman muestras del ancho de banda requerido cada cierto tiempo (120 s). De nuevo cada cierto tiempo (600 s) se calcula el ancho de banda requerido como la máxima de las muestras, y en caso de que la diferencia entre este valor y el ancho de banda actual supere un umbral se señaliza el nuevo ancho de banda para la LSP.

Este sistema funciona bien siempre y cuando los cambios de ancho de banda sean leves, de forma que si se producen grandes picos en cuanto a subida o bajada es ineficiente.

Podemos evitar ambos casos. Para detectar las situaciones de overflow actualizaremos el ancho de banda antes del intervalo de ajuste en caso de que se hayan leído varias muestras que hayan superado el ancho de banda actual. Esto permite responder a subidas repentinas del ancho de banda como los picos de tráfico o situaciones de congestión por fallo de otros enlaces.

# MPLS - Make before break

Antes de cambiar uns LSP se ha de indicar la nueva manteniendo la original. Para ello hay que reservar ancho de banda de forma simultánea para ambas de forma temporal.

# MPLS - Fast reroute

En caso de un fallo en la topología es necesario buscar otras rutas alternativas para poder continuar la transmisión de paquetes. En OSPF se hace con una inundación controlada y después se aplica SPF sobre la nueva topología, mientras que en BGP se espera recibir un nuevo anuncio para la transmisión o se usan otros que ya se han recibido y sean válidos.

En ambos casos la recuperación es costosa y el tráfico se puede ver interrumpido o pueden aparecer bucles. En MPLS se tienen alternativas antes de que ocurra el fallo, de forma que se almacena en las FIB nuevas rutas LSP de forma que en caso de fallo la recuperación es instantanea.

Este sistema nos permite salvaguardarnos frente a fallos de los enlaces, en este caso se crea una LSP alternativa para cada enlace o para evitar fallos en nodos, de forma que se crea una nueva LSP alternativa entre nodos.

A la hora de crear las LSP podemos crear una para cada LSP activa, esto no supone un incremento de etiquetas ya que bastaría con aumentar el tamaño de las FIB, sin embargo la sobrecarga de las mismas es mayor y no nos interesa. La opción más adecuada es crear una LSP alternativa que sirva para varias LSP activas, con ello aumentamos en uno el nivel de etiquetas (se añade una para la ruta alternativa) sin embargo hay menos sobrecarga en las FIB y se consume menos ancho de banda.

# MPLS - IP sobre MPLS

Para tratar el campo TTL o Hop Limit hacemos una copia en el LER de entrada al dominio y lo copiamos en la etiqueta MPLS. En cada encaminador de la LSR se decrementa la copia en 1 y en el LER de salida de copia de nuevo en el datagrama IP.

En caso de descarte se reenvía el paquete hacia delante y en los LER se manda un ICMP al origin del mismo. Esto es posible gracias a que los encaminadores LER ejecutan BGP y mantienen sesiones iBGP entre ellos sin llegar a inyectar rutas externas dentro de OSPF.