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

- **Transmission Control Protocol (TCP)**: 
    - Protocolo de transporte
    - Todas las funciones orientado a la conexión fiable para las aplicaciones.
    - Incluye un gestor para el control de flujo.
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








