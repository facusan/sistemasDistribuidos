# Introducción a Sistemas Distribuidos
Resumen de Introducción a Sistemas Distribuidos

#PDU (Protocol Data Unit)
Es la unidad de datos especificada en un protocolo de una determinada capa.

|       CAPA              |        PDU       |
| ----------------------- | :--------------: |
| CAPA 1 Physical Layer   | BIT              |
| CAPA 2 Data Link Layer  | frame ó trama    |
| CAPA 3 Network Layer    | paquete          |
| CAPA 4 Transport Layer  | segmento (TCP) o datagrama (UDP) |
| CAPA 5-6-7 Application Layer | mensaje  |

# CAPA DE TRANSPORTE
La tarea de esta capa es proporcionar un transporte de datos confiable de la máquina de origen a la máquina de destino, independientemente de la red o redes físicas en uso.

### Sockets: process Identification
<IP Adrress>:<Port number>

- Los puertos conocidos son los comprendidos del 0 al 1023.
- Los puertos registrados son aquellos del 1024 al 49151.
- Los puertos dinámicos y/o privados son los de 49152 a 65535.

## User Datagram Protocol (UDP) 

- Protocolo de transporte
- RFC 768, año 1980
- Simple, proporciona solo direccionamiento de capa.
    1. Transferencia de datos de la capa superior: una aplicación envía datos al protocolo UDP.
    2. Encapsulamiento del mensaje UDP: El mensaje de capa superior se encapsula en el campo de datos de un mensaje UDP. Las cabeceras del mensaje UDP se rellenan, incluyendo el puerto de origen de la aplicación que envñia los datos y el puerto de destino del destinatario.
    3. Transferir Mensaje a IP: Se pasa el mensaje UDP a la capa 3 para la transmisión.

### Qué no hace UDP?
- No establece conexiones antes de enviar datos. Simplemente empaqueta y envía.
- No proporciona ninguna garantía de que sus mensajes llegaran.
- No proporciona los reconocimientos que demuestran que se reciben los datos.
- No detecta la pérdida de mensajes ni los retransmite.
- No asegura que los datos se reciban en el mismo orden en que fueron enviados.
- No proporciona ningún mecanismo para gestionar el flujo de datos entre los dispositivos o manejar la congestión.

### Por qué algunas aplicaciones usan UDP?
- El rendimiento es más importante que la fiabilidad.
- Intercambio de datos que son "Cortos y relevantes".
- Tráfico multicast.
- Aplicaciones que utilizan UDP y TCP.

### UDP Message Format
Encabezado solo tiene 8 bytes de longitud.

| Field Name        | Size (bits)   | Description                                           |
| ------------------| ------------- | ----------------------------------------------------- |
| Source port       | 16            | Número de puerto que originó el mensaje UDP           |
| Destination port  | 16            | Número de puerto del proceso que es detinatario       |
| Length            | 16            | Longitud de todo el datagrama, incluyendo la cabecera |
| Checksum          | 16            | Suma de comprobación opcional                         |
| Data              | Variable      | Mensaje de capa superior que se enviará               |

## Transmission Control Protocol (TCP)

- Protocolo de transporte
- RFC 793, septiembre 1981, varias actualizaciones.
- Todas las funciones orientado a la conexión fiable para las aplicaciones.
- Bidireccional.
- Múltiple conexión y dispositivo-identificado.
- Incluye un gestor para el control de flujo.
- Confirmación.

### TCP Message Format
El encabezado TCP tiene una parte fija de 20 bytes de longitud y otra opcional que puede tener hasta 40 bytes.

| Field Name        | Size (bits)   | Description                                           |
| ------------------| ------------- | ----------------------------------------------------- |
| Source port       | 16            | Número de puerto que originó el mensaje TCP           |
| Destination port  | 16            | Número de puerto del proceso que es detinatario       |
| Sequence number   | 32            | Si el flag SYN=1,  es el nro de seq inicial, sino es el nro de seq acumulado del primer byte de datos |
| ACK number        | 32            | Si el flag ACK=1, es el siguiente numero de seq que el receptor está esperando. Esto confirma la recepción de todos los bytes anteriores. La primer ACK enviado por cada extremo reconoce el nro inicial de secuencia del otro, pero no hay datos |
| Data offset       | 4             | Especifica el tamaño de la cabecera TCP en palabras de 32 bits |
| Reserved          | 6             | Para uso futuro, se ajustan a cero |
| Flags             | 6             | URG: Campo urgent pointer, indica que ciertos datos son prioritarios <br/> ACK: acuse de recibo. <br/> PSH: Funcion de PUSH, para enviar el buffer de recepción hacia la aplicación.<br/> RST: aborta la conexión en respuesta a un error<br/> SYN: incia conexión, sincroniza numeros de secuencia <br/> FIN: no hay más datos de remitente |
| Windows size      | 16            | Es el tamaño de la ventana de recepción, que especifica el nro de bytes que el remitente de este segmento está dispuesto a recibir |
| Checksum          | 16            | Control de errores del header y datos |
| Urgetn pointer    | 16            | si el flag URG está establecido, este puntero indica la cantidad de los datos en el segmento, contando desde el primer byte, es urgente.
| Options           | 0-320         | En bloques de 32 bits, ver RFC-793 |

### Establecimiento de conexión de proceso: "Three-way handsahke"
SYN ->
<- SYN + ACK
ACK ->

### Control de flujo
TCP utiliza una variación en el método de ventnaas deslizantes. En lugar de usar un nro de secuencia para dar acuse de recepción, se utiliza cantidad de bytes transmitidos en el segmento.

Se utiliza un generador de nros iniciales de seq (ISN) que selecciona un nuvo ISN de 32 bits, está asociado a un reloj.

**RTO** Retransmition Time OUT (Tiempo de espera de retransmisión)
La RFC 6298 determina la fórmila de cálculo de RTO.
Dos variables de estado, SRTT (smoothed round-trip time) y RTTVAR (round-trip time variation) y la granularidad del reloj en G segundos.

Hasta se haya realizado una medición de tiempo RTT, el emisor debe establecer el RTO en 1 segundo. En la versión anterior de esta RFC (2988) utiliza un RTO inicial de 3 segundos. Cuando se realiza la primer medición R de RTT, se realizan los siguientes cómputos.

SRTT = R
RTTvar = R / 2
RTO = SRTT + max (G,K. RTTVar)

Cuando se realice una subsecuente medición de R' de RTT:
SRTT'= (1-a) SRTT + a R'
RTTvar' = (1-b) RTTvar + b | SRTT - R' |
Los valores sugeridos por la RFC son K=4, a= 1/8 y b= 1/4
Luego de actualizar los valores de SRTT y RTTvar, se recalcula el nuevo RTO.

RTO= SRTT + max ( G, K . RTTvar)
Si RTO es menor a 1 segundo, entonces el valor se debe redondear a 1.

**Algoritmo de Karn**, las mediciones de RTT no deben llevarse a cabo utilizando segmentos que fueron retransmitidos.

### Pérdida de múltiples segmentos
Para habilitar SACK, debe ser negociado en el inicio de la conexión. Dentro del segmento SYN debe enviarse la opción tipo-4 SACK Permitted.
Durante la conexión se enviará en los ACK la opción 5.

### Terminación Normal de la conexión
FIN                                             ->
                                                <- ACK
                                                <- FIN
ACK                                             ->  
wait for double segment life (MSL) Time
CLOSED

### Control de congestión

**TAHOE**
- Slow start
- Congestion Avoidance (CA)
- Fast retransmit

#### Variables del control de congestión
**cwnd (congestion window)**, que controla del lado de la fuente la cantidad de datos que se puede enviar sin haber recibido un ACK.
**ssthresh (slow start threshold)** que indica en qué fase de control de congestión se encuentra el transmiso (slow start  si es mayor que cwnd o congestion avoidance si es menor; de ser iguales, se puede utilizar cualquiera de los dos algoritmos). El valor inicial es 65536 bytes.
**smss (sender maximum segment size)** es el tamaño máximo de segmento de la conexión o en caso de no anunciarse se toma el default de 536 bytes (RFC 879) 
sender Window = MIN (rwnd, cwnd)

#### Slow start - Congestion Avoidance
El algoritmo comienza en la fase de crecimiento exponencial inicialmente co nu n tamaño de ventana de congestion (CWND) de un MSS y lo aumenta en un tamaño de segmento (MSS) para cada nuevo ACK recibido.

cwnd = cwnd + mss

Este comportamiento continúa hasta que el tamaño de la ventana de congestión (CWND) alcanza el tamaño de ssthresh o hasta que se produce una perdida.

Cuando se produce una perdida (timeout de un segmento enviado), se guarda la mitad del valor de la CWND actual como un nuevo umbral de inicio lento (ssthresh) y slow start comienza de nuevo desde su CWND inicial (1MSS)
.
Una vez que el CWND alcanza el ssthresh, TCP entra en modo CA, de prevención de congestión, en cada nuevo ACK aumenta la CWND:
cwnd = cwnd + mss * mss/cwnd
Esta fase continua hasta que se alcanza la congestión nuevamente. Esto resulta en un aumento lineal de CWND.

#### Fast Retransmit
El algoritmo fast retransmit es utilizado para detectar y reparar pérdidas basado en la recepción de ACK duplicados.
El algoritmo emplea la reccepción de 3 ACK duplicados (4 ACKs idénticos) para concluir que un segmento se ha perdido.
El transmiso envía el segmento que deduce se ha perdido sin esperar que expire el timer de retransmisión.
El umbral ssthresh se fija a la mitad del valor de cwnd antes de la pérdida y se arranca slow start con cwnd = 1 x MSS

#### METODO RENO

- Slow Start (SS)
- Congestion Avoidance (CA)
- Fast Retransmit
- Fast Recovery

#### Fast Recovery
- Congestión severa: detectada por timeout de retransmisión.
    - ssthresh a la mitad del valor de cwnd
    - cwnd = 1 MSS
    - Se aplica slow start
- Congesti´´on leve: detectada por fast retransmit (3 ack duplicados)
    - ssthresh a la mitad del valor de cwnd
    - cwnd = ssthresh
    - Se aplica congestión avoidance.

#### METODO VEGA

TCP Vegas controla su tamaño de ventana observando el RTT de los paquetes que ha enviado previamente.
Si el RTT observado se incrementa, TCP Vegas reconoce que la red comienza a congestionarse, y reduce el tamaño de la ventana.
Si el RTT disminuye, el host emisor determina que la red se alivia de la congestión, y aumenta el tamaño de la ventana de nuevo.

# CAPA DE RED

El servicio que brinda la capa de red es hacer que los datos lleguen desde el origen al destino, aun cuando ambos no estén conectados directamente.

- Version: 4 bits
- IHL: 4 bits
- Tipo de servicio: 8 bits. De tipo y de prioridad. Flash, Intermediate, Routine, etc.
- Longitud Total: 16 bits
- Identificación: 16 bits
- DF: 1 bit
- MF: 1 bit
- Desplazamiento del fragmento: 13 bits
- Tiempo de vida: 8 bits
- Protocolo 8 bits
- Suma de verificación del encabezado: 16 bits.
- Dirección de origen : 32 bits
- Dirección de destino: 32 bits
Hasta acá son 20 bytes de encabezado.
- Opciones: 0 o más palabras, 0 a 40 bytes.

## Direcciones IP v4

Clase A: 1.0.0.0   a 127.255.255.255
Clase B: 128.0.0.0 a 191.255.255.255
Clase C: 192.0.0.0 a 223.255.255.255
Clase D: 224.0.0.0 a 239.255.255.255
Clase E: 240.0.0.0 a 255.255.255.255

**Direcciones privadas**
- 10.0.0.0    a 10.255.255.255
- 172.16.0.0  a 172.31.255.255
- 192.168.0.0 a 192.168.255.255

**NAT (Network Address Translation)**
**PAT (Port Address Translation)**

**CIDR - Enrutamiento interdominios sin clases**
El concepto básico del CDIR es asignar las direcciones IP en bloques de tamaño variable, independientemente de las clases. Si un sitio necesita, 2000 direcciones, se le da un bloque de 2048 direcciones.

### ARP - Protocolo de Resolución de Direcciones

| SOURCE        | DESTINATION       |
| ------------- | ----------------- |
| 1. Check ARP Cache for Destination Device Hardware Addres <br/> 2.Generate ARP Request Frame <br/> 3. Broadcast ARP Request Frame |  |
|  | 4. Process ARP Request Frame <br/> 5. Generate ARP Reply Frame <br/> 6. Update ARP Cache <br/> 7. Send ARP Reply Frame |
| 8. Process ARP Reply Frame <br/> 9. Update ARP Cache |  |


## IP V6
La principal motivación es la limitación impuesta por el campo de direcciones de 32 bits en IP v4.

## Direcciones IP v6
- Consta de 128 bits
- Se representa como ocho grupos de cuatro digitos hexadecimales.
- Cada grupo representa a 16 bits (dos octetos)
- Los grupos están separados por os puntos(:)

- Ceros iniciales: Ceros a la izquierda en un grupo pueden ser omitidos.
- Grupos de ceros: uno o más grupos conecutivos de valor cero pueden ser sustituidos por un solo grupo vacío utilizando (::). Este reemplazo solo se puede aplciar una vez en una dirección.

## Tipos de direcciones en IPv6
### Direcciones Unicast
Un identificador para una sola interfaz.
**Direcciones globales unicast RFC3587**: son direcciones convencionales, enrutable públicaemnte, al igual que las direcciones IPv4 enrutables públicaemnte convencionales.

Utiliza el rango de direccione que comienzan con valor binario 001.
Todas las direcciones unicast globales tinen un ID de interfaz de 64 bits.
Site-Level Aggregator (SLA) es un ID de subred de 16 bits para que la organización pueda establecer subredes.

**Direcciones de enlace local (local-link) RFC7404**: recomendad para redes pequeñas o para links entre routers. No estan destinados a ser enrutados.
Es una dirección IPV6 unicast que se puede configurar de forma automática en cualquier interaz usando el prefijo de enlace local FE80:: y el identificador de interfaz en el formato EUI-64.

**Direcciones locales únicas RFC 4193**: Una dirección local única es una dirección unicast IPv6 que es única a nivel mundial, que está previsto para las comunicaciones locales (No son encaminadas en la Internet global) y son enrutables dentro de un área limitada, tal como una organización.
- Tiene un prefijo unico global
- Cuesta con un prefijo conocido para permitir un fácil filtrado en los límites del sitio
- Es ISP-independiente y se puede utilizar para las cmunicaciones dentro de un sitio sin tener coenctividad a internet.
- Las aplicaciones pueden trat a las direcciones locales únicas como direcciones con ámbito global.

### Direcciones Anycast RFC 2526
Son direcciones que se asignan a más de una interaz (tipicamente pertenecientes a diferentes nodos), con la propiedad de que un paquete enviado a una dirección anycast se dirige a la interfaz "más cercana" que tenga en esa dirección, de acuerdo con la medida de la distancia de los protocolos de enrutamiento.

## Estructura IPv6
IPv6 header: 40 octetos.
- Version: 4 bits
- DS: 6 bits
- ECN: 2 bits
- Flow Label: 20 bits
- Payload length: 16 bits
- Next header: 8 bits
- Hop Limit: 8 bits
- Source address: 128 bits
- Destination address: 128 bits
Extension Header:
- hop by hop option header
- destination Options Header
- Routing header
- Fragment header, en IPv6 la fragmentacion solamente se puede realizar por los nodos de origen, no por los routers a lo largo de la trayectoria de entrega de un paquete.
- Authentication header
- Encapsulating Security payload header
- Destination Options Header

# Internet Control Message Protocol (ICMP)
ICMP es un "asistente administrativo". Proporciona un apoyo fundamental al protocolo IP en forma de mensaje ICM que permiten diferentes tipo de comunicación que se producen entre los dispositivos IP.
Clases de mensajes ICMP:
- Mensajes de error
- Mensaje Informativos ( o consulta)

###Formato
- TYPE: 1 byte. Identifica el tipo de mensaje ICMP. Para ICMPv6 los valores de 0 a 127 son mensajes de error y los valores 128 a 255 son mensajes informativos. Para ICMPv4 los mensajes de error y los informacionales se enceuntran relacionados.
- CODE: 1 byte. Identifica el subtipo de mensaje dentro de cada valor de mensaje de tipo ICMP. Se pueden definir hasta 256 "subtipos" para cada tipo de mensaje. Los valores de este campo se muestran en los temas individuales de tipo de mensake. ICMP.
- Checksum: 2 bytes. 
- Message: tamaño variable.

## RUTEO
- Entrega directa de datagramas
- Entrega indirecta de datagramas

El ruteo es el proceso de seleccionar la mejor ruta en una red.
El proceso de enrutamiento generalmente se realiza sobre la base de tablas que mantienen un registro de las rutas a los diferentes destinos de la red.

- La tabla de enrutamiento puede ser completada de dos formas diferentes:
    - Ruteo no adaptativo o estático.
    - Ruteo adaptativo, o dinámico.

- La mayoria de los algoritmos de enrutamiento utilizan sólo una ruta de red a la vez.
- Tecnicas de enrutamiento de trayectos múltiples permiten el uso de mñultiples caminos alternativos.
- En caso de rutas superpuestas/iguales, se considerarán los siguientes elementos para decidir qué rutas se instalan en la tabla de enrutamiento (ordenados por orden de prioridad):
    - Prefijo-Longitud
    - Métrica
    - Distancia admisnitrativa

### Algoritmos de enrutamiento
#### Enrutamiento por vector distancia
Basado en el algoritmo de camino mínimo de Bellman-Ford.
Hace que cada nodo de un grafo mantenga un vector que da la mejor distancia conocida a cada destino y el arco que se puede usar para llegar ahí.
Si el grafo tiene n nodos, en n-1 iteraciones del algoritmo se consigue procesar el grafo completo.
- **Split Horizon route advertisement**, prohíbe que se publique una ruta de vuelta por la interfaz desde que se conoció.
- **Route Poisoning**, El router anuncia rutas como inalcanzables por la interfaz sobre la que se aprendieron, mediante el establecimiento de la métrica a infinito.

OPERACIÓN
1. Si no hay entrada de ruta que coincida con la recibida, entonces se añade la entrada de ruta a la tabla de enrutamiento de forma automatica, junto con la información sobre el router desde donde ha recibido la tabla de enrutamiento.
2. Si hay entrada de ruta que coincide, pero la métrica de número de saltos es menor que el que está en su tabla de enrutamiento, la tabla de enrutamiento se actualiza con la nueva ruta.
3. Si hay entrada de ruta que coincide, pero la métrica del número de saltos es mayor que el que está en su tabla de enrutamiento, entonces la entrada de enrutameinto se actualiza con el número de saltos de 16(infinit hop)
4. Los paquetes todavia se remiten por la vieja ruta. Un temporizador de espera (hold-down) se inicia y todas las actualizaciones desde otros routers se ignoran.
5. Si despúes de que el temporizador de espera expira y todavía el router está haciendo anuncios con el mismo número de saltos más altos, entonces el valor se actualiza en su tabla de enrutamiento. Solo despues de que expire el temporizador, las actualizaciones de otros routers son aceptados para esa ruta.

TIMERS
- Update timer: intervalo dentre dos mensajes. Por defecto, 30 segundos.
- Invalid timer: tiempo que una entrada puede estar en la tabla sin ser actualizada. Debe ser por lo menos tres veces el valor del Update timer. Por defecto, es de 180 segundos. Despúes que el tiempo se agote el número de saltos se establecerá en 16.
- Hold-down Timer: se inicia cuando el número de saltos cambia en un valor inferior a un valor más alto. Esto permite que la ruta llegue a estabilizarse. Durante este tiempo no hay ninguna actualizacion de enrutamiento. Valor por defecto 180 segundos.
- Flush Timer: tiempo entre que la ruta se invalida o es marcada como inalcanzable y la eliminacion de la entrada de la tabla de enrutamiento. Por defecto, el valor es 240 segundos. Este es 60 segundos mayor que el INvalid Timer. Así durante 60 segundos, el router publicara esta ruta inalcanzable para todos su vecinos. Este temporizador se debe establecer en un valor más alto que el temporizador invalido.

#### Enrutamiento por la ruta más corta
La idea es armar un grafo de la subred en el que cada nodo representa un enrutador y cada arco del grafo un enlace.
Dijkstra (1959), algoritmo de claculo de la ruta mas corta entre dos nodos de un grafo.

## ARQUITECTURA DE SISTEMA AUTONOMO (AS)
- **Internal Router (IR)** todas las interfaces pertenecen al misma área.
- **Area Border Router (ABR)** coencta una o más areass al backbone. Se considera router de todas las areas que conecta.
- **Backbone Router (BR)** son routers que tienen una interfaz en el area backbone. Pueden ser también ABR.
- **Autonomous system boundary router (ASBR)** router frontera con otros AS's.

**Open Shortest Path First(OSPF)**
LSA= Link State Advertisement

- Message types
    - Hello
    - Database Description
    - LSA Request
    - LSA State Update
    - LSA Acknowledgment

- Timers
    - Hello Interval: intervalo entre dos mensajes Hello. Por defecto, 10 segundos.
    - Dead interval: tiempo durante el cual no se reciben mensajes Hello. Debe ser 4 veces el tiempo de Hello, por defecto 40 segundos.
    - LSA retransmission interval: cuando se transmite un LSA a un router vecino, se espera recibir un ACK dentro de ese tiempo. Caso contrario se retransmite.

OPERACION

1. Cada router envia los llamados paquetes de saludo hello por todas las interfaces habilitadas para OSPF. De esta manera, el router descubre routers directamente cconectados que tambien ejecutan OSPF. Si ciertos paramentros en los paquetes de saludo concuerdan, forman una relacion de adyacencia.
2. Luego, cada router intercambia paquetes especiales llamados LINK State Advertisments (LSA) con sus vecinos. EN OSPF la palabra enlace es la misma que la interfaz. Las LSAs contienen detalles tales como: direcciones/mascaras de red configurados en los enlaces (interfaces que ejecutan OSPF), la métrica, el estado del enlace ( que es su relacion con el resto de la red), la lista de vecinos conectados al enlace, etc.
3. Cada router almacena los LSAs en su base de datos de estado de enlace (LSDB). Estos LSAs son entonces anunciados a todos los vecinos OSPF. Como resultado de la inundacion de LSA, todos los routers en el area tienen LSDBs identicos.
4. Cada router se ejecuta el algoritmo de Dijstra para seleccionar la mejor ruta de esa base de datos topologica (LSDB). De esta manera, cada router crea grafico libre de bucles indicando el (mejor) camino mas corto para cada red/ subred anunciado. Las mejores rutas terminan en la tabla de enrutamiento.

##BGP (BORDER GATEWAY PROTOCOL)
- Es el protocolode ruteo mas utilizado en internet
- Se utiliza para comunicar sistemas autonomos.
- Es un protocolo del tipo path-vector: secuenia de AS's y direccion de next hop que conforman una ruta a un cierto destino.
- Utiliza atributos en lugar de una metrica para escoger la mejor ruta a un destino.
- Utiliza unicast para establecer una adyacenciaa (TCP 179)
- Cuando intercambia rutas entre sistemas autonomos diferentes se llama eBGP
- Cuando intercambia rutas dentro del mismo sistema autonomo se llama iBGP

### Sesiones BGP
En una sesion BGP participan solo dos routers (peers).
En la sesion BGP se lleva a cabo el proceso denominado peering, que consite en que un AS informa a otro sobre las rede que puede alcanzar a partir de este.
Ademas de las sesiones inter-AS, los routers de borde de un mismo AS deben intercambiar tambien informaciones BGP para conocer las misma rutas externas e internas. Para ello se utiliza el protocolo I-BGP.






# Cableado estructurado

Modelo jerarquico de 3 capas:
- el cableado horizontal
- el cableado troncal edificio
- el cableado troncal del campus

## Distribuidores
Los paneles de conexión son conocidos como distribuidores
- el **distribuido piso** es el vínculo entre el cableado y la troncal del edificio.
- el **distribuidos del edificio** es el vínculo entre la troncal del edificio y la del campus.
- el **distribuidor de campus** es donde todo el cableado de campus se reúne.

**Enlace permanente:** Es el cableado entre el distribuidor de piso y la toma de telecomunicaciones (PL, permanent link).

La adición de los cables de conexión en cada extremo de la PL forma el **cableado horizontal**.
Los cables de conexión de la TO a la vinculación de los equipos activos son los cables del **área de trabajo**.

Asociado con el distribuidor de campus, se encuentra la **sala principal de equipos**.

|       ISO               |        ANSI       |
| ----------------------- | :--------------:  |
| Campus distributor      | Main Crossconnect |
| Building distributor    | Intermediate CrossConnect|
| Floor distributor       | Horizontal Crossconnect|

## Longitudes máximas de cableado
La distancia máxima del cableado horizontal es de 100 metros. El enlace permaenente, PL, no puede exceder los 90 metros.

ISO: La distancia total compuesta por la suma del cableado horizontal más el backbone del edificio, más el backbone del campus no debe superar los 2000 m.

ANSI:Las distancias de backbone vienen definidas:
Intermediate- Horizontal: 300 m sin importar el medio.
Main- intermediate: desde 500 a 2700 mestros dependdiendo del medio.

COA (Centralised optical architecture): Desde El campus distributor se puede ir directamente a las terminales o desde los building distributors.

CP (Consoloidation Point).
MUTOA (Multi User Telecommunications Outlet Assembly)

## Espacios y montantes
- Building entrance facility
- Equipment room
- Telecommunicattions room

### Consideraciones de los espacios
- Distancia de la zona de trabajo a ser servida.
- Posición vertical relativa de las salas de telecomunicaciones.
- Accesibilidad.
- Dimensiones adecuadas.
- Dedicación exclusiva.
- No deben contener tuberías.
- Ventilación adecuada y aire acondicionado.
- Energía.
- Fuentes de interferencia.
- Fortaleza de piso adecuada.
- Control de acceso.

## Separación de cables de energías y de datos.








