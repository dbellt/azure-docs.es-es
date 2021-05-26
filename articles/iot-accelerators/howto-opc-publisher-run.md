---
title: 'Ejecución de OPC Publisher: Azure | Microsoft Docs'
description: En este artículo se describe cómo ejecutar y depurar OPC Publisher. También se abordan las consideraciones sobre rendimiento y memoria.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 7a37ababcac0208bfa92c2edf221ff5513bcb2e6
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109839318"
---
# <a name="run-opc-publisher"></a>Ejecutar OPC Publisher

> [!IMPORTANT]
> Mientras actualizamos este artículo, consulte el artículo sobre [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/), donde encontrará el contenido más actualizado.

En este artículo se describe cómo ejecutar y depurar OPC Publisher. También se abordan las consideraciones sobre rendimiento y memoria.

## <a name="command-line-options"></a>Opciones de línea de comandos

El uso de la aplicación se muestra mediante la opción de la línea de comandos `--help` como sigue:

```sh/cmd
Current directory is: /appdata
Log file is: <hostname>-publisher.log
Log level is: info

OPC Publisher V2.3.0
Informational version: V2.3.0+Branch.develop_hans_methodlog.Sha.0985e54f01a0b0d7f143b1248936022ea5d749f9

Usage: opcpublisher.exe <applicationname> [<IoT Hubconnectionstring>] [<options>]

OPC Edge Publisher to subscribe to configured OPC UA servers and send telemetry to Azure IoT Hub.
To exit the application, just press CTRL-C while it is running.

applicationname: the OPC UA application name to use, required
                  The application name is also used to register the publisher under this name in the
                  IoT Hub device registry.

IoT Hubconnectionstring: the IoT Hub owner connectionstring, optional

There are a couple of environment variables which can be used to control the application:
_HUB_CS: sets the IoT Hub owner connectionstring
_GW_LOGP: sets the filename of the log file to use
_TPC_SP: sets the path to store certificates of trusted stations
_GW_PNFP: sets the filename of the publishing configuration file

Command line arguments overrule environment variable settings.

Options:
      --pf, --publishfile=VALUE
                              the filename to configure the nodes to publish.
                                Default: '/appdata/publishednodes.json'
      --tc, --telemetryconfigfile=VALUE
                              the filename to configure the ingested telemetry
                                Default: ''
  -s, --site=VALUE           the site OPC Publisher is working in. if specified
                                this domain is appended (delimited by a ':' to
                                the 'ApplicationURI' property when telemetry is
                                sent to IoT Hub.
                                The value must follow the syntactical rules of a
                                DNS hostname.
                                Default: not set
      --ic, --iotcentral     publisher will send OPC UA data in IoTCentral
                                compatible format (DisplayName of a node is used
                                as key, this key is the Field name in IoTCentral)
                                . you need to ensure that all DisplayName's are
                                unique. (Auto enables fetch display name)
                                Default: False
      --sw, --sessionconnectwait=VALUE
                              specify the wait time in seconds publisher is
                                trying to connect to disconnected endpoints and
                                starts monitoring unmonitored items
                                Min: 10
                                Default: 10
      --mq, --monitoreditemqueuecapacity=VALUE
                              specify how many notifications of monitored items
                                can be stored in the internal queue, if the data
                                can not be sent quick enough to IoT Hub
                                Min: 1024
                                Default: 8192
      --di, --diagnosticsinterval=VALUE
                              shows publisher diagnostic info at the specified
                                interval in seconds (need log level info).
                                -1 disables remote diagnostic log and diagnostic
                                output
                                0 disables diagnostic output
                                Default: 0
      --ns, --noshutdown=VALUE
                              same as runforever.
                                Default: False
      --rf, --runforever     publisher can not be stopped by pressing a key on
                                the console, but will run forever.
                                Default: False
      --lf, --logfile=VALUE  the filename of the logfile to use.
                                Default: './<hostname>-publisher.log'
      --lt, --logflushtimespan=VALUE
                              the timespan in seconds when the logfile should be
                                flushed.
                                Default: 00:00:30 sec
      --ll, --loglevel=VALUE the loglevel to use (allowed: fatal, error, warn,
                                info, debug, verbose).
                                Default: info
        --ih, --IoT Hubprotocol=VALUE
                              the protocol to use for communication with IoT Hub (
                                allowed values: Amqp, Http1, Amqp_WebSocket_Only,
                                  Amqp_Tcp_Only, Mqtt, Mqtt_WebSocket_Only, Mqtt_
                                Tcp_Only) or IoT EdgeHub (allowed values: Mqtt_
                                Tcp_Only, Amqp_Tcp_Only).
                                Default for IoT Hub: Mqtt_WebSocket_Only
                                Default for IoT EdgeHub: Amqp_Tcp_Only
      --ms, --IoT Hubmessagesize=VALUE
                              the max size of a message which can be send to
                                IoT Hub. when telemetry of this size is available
                                it will be sent.
                                0 will enforce immediate send when telemetry is
                                available
                                Min: 0
                                Max: 262144
                                Default: 262144
      --si, --IoT Hubsendinterval=VALUE
                              the interval in seconds when telemetry should be
                                send to IoT Hub. If 0, then only the
                                IoT Hubmessagesize parameter controls when
                                telemetry is sent.
                                Default: '10'
      --dc, --deviceconnectionstring=VALUE
                              if publisher is not able to register itself with
                                IoT Hub, you can create a device with name <
                                applicationname> manually and pass in the
                                connectionstring of this device.
                                Default: none
  -c, --connectionstring=VALUE
                              the IoT Hub owner connectionstring.
                                Default: none
      --hb, --heartbeatinterval=VALUE
                              the publisher is using this as default value in
                                seconds for the heartbeat interval setting of
                                nodes without
                                a heartbeat interval setting.
                                Default: 0
      --sf, --skipfirstevent=VALUE
                              the publisher is using this as default value for
                                the skip first event setting of nodes without
                                a skip first event setting.
                                Default: False
      --pn, --portnum=VALUE  the server port of the publisher OPC server
                                endpoint.
                                Default: 62222
      --pa, --path=VALUE     the enpoint URL path part of the publisher OPC
                                server endpoint.
                                Default: '/UA/Publisher'
      --lr, --ldsreginterval=VALUE
                              the LDS(-ME) registration interval in ms. If 0,
                                then the registration is disabled.
                                Default: 0
      --ol, --opcmaxstringlen=VALUE
                              the max length of a string opc can transmit/
                                receive.
                                Default: 131072
      --ot, --operationtimeout=VALUE
                              the operation timeout of the publisher OPC UA
                                client in ms.
                                Default: 120000
      --oi, --opcsamplinginterval=VALUE
                              the publisher is using this as default value in
                                milliseconds to request the servers to sample
                                the nodes with this interval
                                this value might be revised by the OPC UA
                                servers to a supported sampling interval.
                                please check the OPC UA specification for
                                details how this is handled by the OPC UA stack.
                                a negative value will set the sampling interval
                                to the publishing interval of the subscription
                                this node is on.
                                0 will configure the OPC UA server to sample in
                                the highest possible resolution and should be
                                taken with care.
                                Default: 1000
      --op, --opcpublishinginterval=VALUE
                              the publisher is using this as default value in
                                milliseconds for the publishing interval setting
                                of the subscriptions established to the OPC UA
                                servers.
                                please check the OPC UA specification for
                                details how this is handled by the OPC UA stack.
                                a value less than or equal zero will let the
                                server revise the publishing interval.
                                Default: 0
      --ct, --createsessiontimeout=VALUE
                              specify the timeout in seconds used when creating
                                a session to an endpoint. On unsuccessful
                                connection attemps a backoff up to 5 times the
                                specified timeout value is used.
                                Min: 1
                                Default: 10
      --ki, --keepaliveinterval=VALUE
                              specify the interval in seconds the publisher is
                                sending keep alive messages to the OPC servers
                                on the endpoints it is connected to.
                                Min: 2
                                Default: 2
      --kt, --keepalivethreshold=VALUE
                              specify the number of keep alive packets a server
                                can miss, before the session is disconneced
                                Min: 1
                                Default: 5
      --aa, --autoaccept     the publisher trusts all servers it is
                                establishing a connection to.
                                Default: False
      --tm, --trustmyself=VALUE
                              same as trustowncert.
                                Default: False
      --to, --trustowncert   the publisher certificate is put into the trusted
                                certificate store automatically.
                                Default: False
      --fd, --fetchdisplayname=VALUE
                              same as fetchname.
                                Default: False
      --fn, --fetchname      enable to read the display name of a published
                                node from the server. this will increase the
                                runtime.
                                Default: False
      --ss, --suppressedopcstatuscodes=VALUE
                              specifies the OPC UA status codes for which no
                                events should be generated.
                                Default: BadNoCommunication,
                                BadWaitingForInitialData
      --at, --appcertstoretype=VALUE
                              the own application cert store type.
                                (allowed values: Directory, X509Store)
                                Default: 'Directory'
      --ap, --appcertstorepath=VALUE
                              the path where the own application cert should be
                                stored
                                Default (depends on store type):
                                X509Store: 'CurrentUser\UA_MachineDefault'
                                Directory: 'pki/own'
      --tp, --trustedcertstorepath=VALUE
                              the path of the trusted cert store
                                Default: 'pki/trusted'
      --rp, --rejectedcertstorepath=VALUE
                              the path of the rejected cert store
                                Default 'pki/rejected'
      --ip, --issuercertstorepath=VALUE
                              the path of the trusted issuer cert store
                                Default 'pki/issuer'
      --csr                  show data to create a certificate signing request
                                Default 'False'
      --ab, --applicationcertbase64=VALUE
                              update/set this applications certificate with the
                                certificate passed in as bas64 string
      --af, --applicationcertfile=VALUE
                              update/set this applications certificate with the
                                certificate file specified
      --pb, --privatekeybase64=VALUE
                              initial provisioning of the application
                                certificate (with a PEM or PFX fomat) requires a
                                private key passed in as base64 string
      --pk, --privatekeyfile=VALUE
                              initial provisioning of the application
                                certificate (with a PEM or PFX fomat) requires a
                                private key passed in as file
      --cp, --certpassword=VALUE
                              the optional password for the PEM or PFX or the
                                installed application certificate
      --tb, --addtrustedcertbase64=VALUE
                              adds the certificate to the applications trusted
                                cert store passed in as base64 string (multiple
                                strings supported)
      --tf, --addtrustedcertfile=VALUE
                              adds the certificate file(s) to the applications
                                trusted cert store passed in as base64 string (
                                multiple filenames supported)
      --ib, --addissuercertbase64=VALUE
                              adds the specified issuer certificate to the
                                applications trusted issuer cert store passed in
                                as base64 string (multiple strings supported)
      --if, --addissuercertfile=VALUE
                              adds the specified issuer certificate file(s) to
                                the applications trusted issuer cert store (
                                multiple filenames supported)
      --rb, --updatecrlbase64=VALUE
                              update the CRL passed in as base64 string to the
                                corresponding cert store (trusted or trusted
                                issuer)
      --uc, --updatecrlfile=VALUE
                              update the CRL passed in as file to the
                                corresponding cert store (trusted or trusted
                                issuer)
      --rc, --removecert=VALUE
                              remove cert(s) with the given thumbprint(s) (
                                multiple thumbprints supported)
      --dt, --devicecertstoretype=VALUE
                              the IoT Hub device cert store type.
                                (allowed values: Directory, X509Store)
                                Default: X509Store
      --dp, --devicecertstorepath=VALUE
                              the path of the iot device cert store
                                Default Default (depends on store type):
                                X509Store: 'My'
                                Directory: 'CertificateStores/IoT Hub'
  -i, --install              register OPC Publisher with IoT Hub and then exits.
                                Default:  False
  -h, --help                 show this message and exit
      --st, --opcstacktracemask=VALUE
                              ignored, only supported for backward comaptibility.
      --sd, --shopfloordomain=VALUE
                              same as site option, only there for backward
                                compatibility
                                The value must follow the syntactical rules of a
                                DNS hostname.
                                Default: not set
      --vc, --verboseconsole=VALUE
                              ignored, only supported for backward comaptibility.
      --as, --autotrustservercerts=VALUE
                              same as autoaccept, only supported for backward
                                cmpatibility.
                                Default: False
      --tt, --trustedcertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the trusted cert store will always reside in a
                                directory.
      --rt, --rejectedcertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the rejected cert store will always reside in a
                                directory.
      --it, --issuercertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the trusted issuer cert store will always
                                reside in a directory.
```

Normalmente, la cadena de conexión del propietario de IoT Hub solo se especifica durante la primera ejecución de la aplicación. La cadena de conexión se cifra y se almacena en el almacén de certificados de la plataforma. En las ejecuciones posteriores, la aplicación lee la cadena de conexión del almacén de certificados. Si especifica la cadena de conexión en cada ejecución, el dispositivo que se creó para la aplicación en el registro del dispositivo de IoT Hub se elimina y se vuelve a crear.

## <a name="run-natively-on-windows"></a>Ejecución de forma nativa en Windows

Abra el proyecto **opcpublisher.sln** con Visual Studio, compile la solución y publíquela. Puede iniciar la aplicación en el **directorio de destino** donde ha realizado la publicación de la siguiente manera:

```cmd
dotnet opcpublisher.dll <applicationname> [<IoT Hubconnectionstring>] [options]
```

## <a name="use-a-self-built-container"></a>Uso de un contenedor compilado automáticamente

Compile su propio contenedor e inícielo como sigue:

```sh/cmd
docker run <your-container-name> <applicationname> [<IoT Hubconnectionstring>] [options]
```

## <a name="use-a-container-from-microsoft-container-registry"></a>Uso de un contenedor de Microsoft Container Registry

Hay un contenedor pregenerado disponible en Microsoft Container Registry. Inícielo como sigue:

```sh/cmd
docker run mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

Consulte [Docker Hub](https://hub.docker.com/_/microsoft-iotedge-opc-publisher) para ver los sistemas operativos y las arquitecturas de procesador compatibles. Si el sistema operativo y la arquitectura de CPU son compatibles, Docker selecciona automáticamente el contenedor adecuado.

## <a name="run-as-an-azure-iot-edge-module"></a>Ejecución como módulo de Azure IoT Edge

OPC Publisher está listo para usarse como módulo de [Azure IoT Edge](../iot-edge/index.yml). Al usar OPC Publisher como módulo de IoT Edge, los únicos protocolos de transporte admitidos son **Amqp_Tcp_Only** y **Mqtt_Tcp_Only**.

Para agregar OPC Publisher como módulo a la implementación de IoT Edge, vaya a la configuración de IoT Hub en Azure Portal y complete estos pasos:

1. Vaya a **IoT Edge** y cree o seleccione el dispositivo IoT Edge.
1. Seleccione **Set modules** (Establecer módulos).
1. Seleccione **Agregar** en **Módulos de implementación** e **IoT Edge Module** (Módulo de IoT Edge).
1. En el campo **Nombre**, escriba **publisher**.
1. En el campo **URI de la imagen**, escriba `mcr.microsoft.com/iotedge/opc-publisher:<tag>`.
1. Puede encontrar las etiquetas disponibles en [Docker Hub](https://hub.docker.com/_/microsoft-iotedge-opc-publisher).
1. Pegue el siguiente JSON en el campo **Opciones de creación del contenedor**:

    ```json
    {
        "Hostname": "publisher",
        "Cmd": [
            "--aa"
        ]
    }
    ```

    Esta configuración configura IoT Edge para iniciar un contenedor denominado **publisher** con la imagen de OPC Publisher. El nombre de host del sistema del contenedor está establecido en **publisher**. A OPC Publisher se llama con el siguiente argumento de la línea de comandos: `--aa`. Con esta opción, OPC Publisher confía en los certificados de los servidores OPC UA a los que se conecta. Puede usar cualquiera de las opciones de la línea de comandos de OPC Publisher. La única limitación es el tamaño de las **Opciones de creación del contenedor** que admite IoT Edge.

1. Deje los restantes valores tal cual y seleccione **Guardar**.
1. Si desea procesar la salida de OPC Publisher localmente con otro módulo de IoT Edge, vuelva a la página **Establecer módulos**. A continuación, vaya a la pestaña **Especificar rutas** y agregue una nueva ruta similar al siguiente JSON:

    ```json
    {
      "routes": {
        "processingModuleToIoT Hub": "FROM /messages/modules/processingModule/outputs/* INTO $upstream",
        "opcPublisherToProcessingModule": "FROM /messages/modules/publisher INTO BrokeredEndpoint(\"/modules/processingModule/inputs/input1\")"
      }
    }
    ```

1. De nuevo en la página **Establecer módulos**, seleccione **Siguiente** hasta llegar a la última página de la configuración.
1. Seleccione **Enviar** para enviar la configuración a IoT Edge.
1. Cuando haya iniciado IoT Edge en el dispositivo de Edge y el contenedor de Docker **publisher** se esté ejecutando, puede modificar la salida del registro de OPC Publisher mediante `docker logs -f publisher` o en el archivo de registro. En el ejemplo anterior, el archivo de registro está encima de `d:\iiotegde\publisher-publisher.log`. También puede usar la [herramienta iot-edge-opc-publisher-diagnostics](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics).

### <a name="make-the-configuration-files-accessible-on-the-host"></a>Accesibilidad de los archivos de configuración en el host

Para hacer que los archivos de configuración del módulo de IoT Edge estén accesibles en el sistema de archivos del host, use las siguientes **Opciones de creación del contenedor**. El siguiente ejemplo es de una implementación con contenedores de Linux para Windows:

```json
{
    "Hostname": "publisher",
    "Cmd": [
        "--pf=/appdata/pn.json",
        "--aa"
    ],
    "HostConfig": {
        "Binds": [
            "d:/iiotedge:/appdata"
        ]
    }
}
```

Con estas opciones, OPC Publisher lee los nodos que debe publicar desde el archivo `./pn.json` y el directorio de trabajo del contenedor se establece en `/appdata` al inicio. Con esta configuración, OPC Publisher lee el archivo `/appdata/pn.json` desde el contenedor para obtener su configuración. Sin la opción `--pf`, OPC Publisher intenta leer el archivo de configuración predeterminado `./publishednodes.json`.

El archivo de registro, con el nombre predeterminado `publisher-publisher.log`, se escribe en `/appdata` y el directorio `CertificateStores` también se crea en este directorio.

Para que todos estos archivos estén disponibles en el sistema de archivos del host, la configuración del contenedor requiere un volumen de montaje de enlace. El enlace `d://iiotedge:/appdata` asigna el directorio `/appdata`, que es el directorio de trabajo actual del inicio del contenedor, al directorio del host `d://iiotedge`. Sin esta opción, no persisten los datos de archivo al volver a iniciar el contenedor.

Si ejecuta contenedores de Windows, la sintaxis del parámetro `Binds` es distinta. Al inicio del contenedor, el directorio de trabajo es `c:\appdata`. Para colocar el archivo de configuración en el directorio `d:\iiotedge` en el host, especifique la siguiente asignación en la sección `HostConfig`:

```json
"HostConfig": {
    "Binds": [
        "d:/iiotedge:c:/appdata"
    ]
}
```

Si ejecuta contenedores de Linux, la sintaxis del parámetro `Binds` vuelve a ser distinta. Al inicio del contenedor, el directorio de trabajo es `/appdata`. Para colocar el archivo de configuración en el directorio `/iiotedge` en el host, especifique la siguiente asignación en la sección `HostConfig`:

```json
"HostConfig": {
    "Binds": [
        "/iiotedge:/appdata"
    ]
}
```

## <a name="considerations-when-using-a-container"></a>Consideraciones al usar un contenedor

En las siguientes secciones se enumeran detalles que recordar al usar un contenedor:

### <a name="access-to-the-opc-publisher-opc-ua-server"></a>Acceso al servidor OPC UA de OPC Publisher

De manera predeterminada, el servidor OPC UA de OPC Publisher escucha en el puerto 62222. Para exponer este puerto de entrada en un contenedor, use el siguiente comando:

```sh/cmd
docker run -p 62222:62222 mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="enable-intercontainer-name-resolution"></a>Habilitar la resolución de nombres entre contenedores

Para habilitar la resolución de nombres desde el contenedor a otros contenedores, cree una red de puente de Docker definida por el usuario y conecte el contenedor a esta red mediante la opción `--network`. Asígnele también un nombre al contenedor mediante la opción `--name`, como sigue:

```sh/cmd
docker network create -d bridge iot_edge
docker run --network iot_edge --name publisher mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

El contenedor ahora será accesible mediante el nombre `publisher` para otros contenedores de la misma red.

### <a name="access-other-systems-from-within-the-container"></a>Acceso a otros sistemas desde dentro del contenedor

Se puede acceder a otros contenedores mediante los parámetros descritos en la sección anterior. Si el sistema operativo donde se hospeda Docker tiene DNS habilitado, el acceso a todos los sistemas que conozcan DNS funcionará.

En redes que usen la resolución de nombres NetBIOS, habilite el acceso a otros sistemas mediante el inicio del contenedor con la opción `--add-host`. Esta opción agrega una entrada al archivo de host del contenedor:

```cmd/sh
docker run --add-host mydevbox:192.168.178.23  mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="assign-a-hostname"></a>Asignación de un nombre de host

OPC Publisher usará el nombre de host de la máquina en la que se está ejecutando para la generación del certificado y del punto de conexión. Docker elegirá un nombre de host aleatorio si no hay ninguno definido por la opción `-h`. En el siguiente ejemplo se muestra cómo establecer el nombre de host interno del contenedor en `publisher`:

```sh/cmd
docker run -h publisher mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="use-bind-mounts-shared-filesystem"></a>Uso de montajes de enlaces (sistema de archivos compartido)

En lugar de usar el sistema de archivos del contenedor, puede elegir el sistema de archivos del host para almacenar la información de configuración y los archivos de registro. Para configurar esta opción, use la opción `-v` de `docker run` en el modo de montaje de enlace.

## <a name="opc-ua-x509-certificates"></a>Certificados X.509 OPC UA

OPC UA usa certificados X.509 para autenticar el cliente de OPC UA y el servidor al establecer una conexión y para cifrar la comunicación entre ellos. OPC Publisher usa almacenes de certificados que mantiene la pila de OPC UA para administrar todos los certificados. Al inicio, OPC Publisher comprueba si hay un certificado para sí mismo. Si hay ninguno en el almacén de certificados y no se pasa uno en la línea de comandos, OPC Publisher crea un certificado autofirmado. Para más información, consulte el método **InitApplicationSecurityAsync** en `OpcApplicationConfigurationSecurity.cs`.

Los certificados autofirmados no proporcionan seguridad, ya que no están firmados por una entidad de certificación de confianza.

OPC Publisher proporciona opciones de la línea de comandos para:

- Recuperar la información de CSR del certificado de aplicación actual que utiliza OPC Publisher.
- Aprovisionar OPC Publisher con un certificado firmado por una entidad de certificación.
- Aprovisionar OPC Publisher con un nuevo par de claves y el certificado firmado por una entidad de certificado correspondiente.
- Agregar certificados del mismo nivel de confianza o al almacén de certificados del emisor de confianza.
- Agregar una lista de revocación de certificados.
- Eliminar certificados del mismo nivel de confianza o del almacén de certificados del emisor de confianza.

Todas estas opciones permiten pasar parámetros con archivos o cadenas con codificación base64.

El tipo predeterminado de almacén de certificados es el sistema de archivos, que se puede cambiar mediante las opciones de la línea de comandos. Puesto que el contenedor no proporciona un almacenamiento persistente en su sistema de archivos, debe elegir un tipo de almacén diferente. Use la opción `-v` de Docker para conservar los almacenes de certificados en el sistema de archivos del host o en un volumen de Docker. Si usa un volumen de Docker, puede pasar certificados mediante cadenas con codificación base64.

El entorno en tiempo de ejecución afecta a cómo se conservan los certificados. Evite crear nuevos almacenes de certificados cada vez que ejecute la aplicación:

- Con la ejecución nativa en Windows no se puede usar un almacén de certificados de la aplicación del tipo `Directory` porque se produce un error de acceso a la clave privada. En este caso, use la opción `--at X509Store`.
- Con la ejecución como contenedor de Docker de Linux puede asignar los almacenes de certificados al sistema de archivos del host con la opción de ejecución de Docker `-v <hostdirectory>:/appdata`. Esta opción hace que el certificado persista al ejecutar la aplicación.
- Si realiza la ejecución como contenedor de Docker de Linux y desea usar un almacén X509 para el certificado de la aplicación, use la opción de ejecución de Docker `-v x509certstores:/root/.dotnet/corefx/cryptography/x509stores` y la opción `--at X509Store` de la aplicación.

## <a name="performance-and-memory-considerations"></a>Consideraciones sobre rendimiento y memoria

En esta sección se describen las opciones para administrar la memoria y el rendimiento:

### <a name="command-line-parameters-to-control-performance-and-memory"></a>Parámetros de la línea de comandos para controlar el rendimiento y la memoria

Al ejecutar OPC Publisher, debe tener en cuenta los requisitos de rendimiento y los recursos de memoria disponibles en el host.

La memoria y el rendimiento son interdependientes y ambos dependen del número de nodos que configure para publicar. Asegúrese de que los parámetros siguientes cumplen sus requisitos:

- Intervalo de envío de IoT Hub: `--si`
- Tamaño de mensaje de IoT Hub (valor predeterminado `1`): `--ms`
- Capacidad de cola de los elementos supervisados: `--mq`

El parámetro `--mq` controla el límite superior de capacidad de la cola interna, que almacena en búfer todas las notificaciones de cambio de valor de los nodos de OPC. Si OPC Publisher no puede enviar mensajes a IoT Hub lo suficientemente rápido, esta cola almacena en búfer las notificaciones. El parámetro establece el número de notificaciones que se pueden almacenar en búfer. Si ve que el número de elementos de esta cola aumenta en la serie de pruebas, para evitar perder mensajes debe:

- Reducir el intervalo de envío de IoT Hub
- Aumentar el tamaño de los mensajes de IoT Hub

El parámetro `--si` fuerza el envío de mensajes de OPC Publisher a IoT Hub con el intervalo especificado. OPC Publisher envía un mensaje en cuanto se alcanza el tamaño del mensaje especificado por el parámetro `--ms` o el intervalo especificado por el parámetro `--si`. Para deshabilitar la opción de tamaño de mensaje, utilice `--ms 0`. En este caso, OPC Publisher usa el mayor tamaño de mensaje de IoT Hub posible de 256 kB para los datos por lotes.

El parámetro `--ms` permite procesar por lotes los mensajes enviados a IoT Hub. El protocolo que use determinará si la sobrecarga que supone enviar un mensaje a IoT Hub es alta en comparación con el tiempo de envío de la carga real. Si su escenario permite la latencia cuando IoT Hub ingiere datos, configure OPC Publisher para usar el mayor tamaño de mensaje (256 kB).

Antes de usar OPC Publisher en escenarios de producción, pruebe el rendimiento y el uso de memoria en condiciones de producción. Puede usar el parámetro `--di` para especificar el intervalo de escritura de información de diagnóstico de OPC Publisher en segundos.

### <a name="test-measurements"></a>Medidas de prueba

Los diagnósticos de ejemplo siguientes muestran las medidas con distintos valores para los parámetros `--si` y `--ms`, que publican en 500 nodos con un intervalo de publicación de OPC de 1 segundo.  La prueba utilizó una compilación de depuración de OPC Publisher en Windows 10 de forma nativa durante 120 segundos. El protocolo de IoT Hub era el protocolo MQTT predeterminado.

#### <a name="default-configuration---si-10---ms-262144"></a>Configuración predeterminada (--si 10 --ms 262144)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:33:05 (started @ 26.10.2017 15:31:09)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54363
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54363
---------------------------------
messages sent to IoT Hub: 109
last successful msg sent @: 26.10.2017 15:33:04
bytes sent to IoT Hub: 12709429
avg msg size: 116600
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 10
--ms setting: 262144
--ih setting: Mqtt
==========================================================================
```

La configuración predeterminada envía datos a IoT Hub cada 10 segundos o cuando hay 256 kB de datos disponibles para que IoT Hub los ingiera. Esta configuración agrega una latencia moderada de unos 10 segundos, pero tiene menos probabilidad de perder datos debido al gran tamaño de los mensajes. La salida del diagnóstico muestra que no se han perdido actualizaciones de nodo de OPC: `monitored item notifications enqueue failure: 0`.

#### <a name="constant-send-interval---si-1---ms-0"></a>Intervalo de envío constante (--si 1 --ms 0)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:35:59 (started @ 26.10.2017 15:34:03)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54243
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54243
---------------------------------
messages sent to IoT Hub: 109
last successful msg sent @: 26.10.2017 15:35:59
bytes sent to IoT Hub: 12683836
avg msg size: 116365
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 1
--ms setting: 0
--ih setting: Mqtt
==========================================================================
```

Cuando el tamaño del mensaje se establece en 0, OPC Publisher crea lotes de datos internamente con el mayor tamaño de mensaje de IoT Hub compatible, 256 kB. La salida del diagnóstico muestra que el tamaño medio de mensaje es 115 019 bytes. En esta configuración, OPC Publisher no pierde actualizaciones del valor de nodo de OPC y, en comparación con la predeterminada, la latencia es menor.

### <a name="send-each-opc-node-value-update---si-0---ms-0"></a>Envío de cada actualización del valor de nodo de OPC (--si 0 --ms 0)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:39:33 (started @ 26.10.2017 15:37:37)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 8184
monitored item notifications enqueued: 54232
monitored item notifications enqueue failure: 44624
monitored item notifications dequeued: 1424
---------------------------------
messages sent to IoT Hub: 1423
last successful msg sent @: 26.10.2017 15:39:33
bytes sent to IoT Hub: 333046
avg msg size: 234
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 96
--si setting: 0
--ms setting: 0
--ih setting: Mqtt
==========================================================================
```

Esta configuración envía un mensaje a IoT Hub por cada cambio de valor del nodo de OPC. El diagnóstico muestra que el tamaño medio de los mensajes es 234 bytes, pequeño. La ventaja de esta configuración es que OPC Publisher no agrega latencia. El número de actualizaciones del valor de nodo de OPC perdidas (`monitored item notifications enqueue failure: 44624`) es alto, por lo que esta configuración no es apta para escenarios con grandes volúmenes de datos de telemetría que publicar.

### <a name="maximum-batching---si-0---ms-262144"></a>Procesamiento por lotes máximo (--si 0 --ms 262144)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:42:55 (started @ 26.10.2017 15:41:00)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54137
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54137
---------------------------------
messages sent to IoT Hub: 48
last successful msg sent @: 26.10.2017 15:42:55
bytes sent to IoT Hub: 12565544
avg msg size: 261782
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 0
--ms setting: 262144
--ih setting: Mqtt
==========================================================================
```

Esta configuración procesa por lotes tantas actualizaciones del valor de nodo de OPC como sea posible. El tamaño máximo de los mensajes de IoT Hub es 256 kB, que es lo que se ha configurado. No se han solicitado intervalos de envío, lo que significa que la cantidad de datos que ingiera IoT Hub determina la latencia. Esta configuración es la que menos probabilidad tiene de perder valores del nodo de OPC y es adecuada para publicar un gran número de nodos. Al usar esta configuración, asegúrese de que el escenario no tiene condiciones que introduzcan latencia alta si no se alcanza el tamaño de mensaje de 256 kB.

## <a name="debug-the-application"></a>Depurar la aplicación

Para depurar la aplicación, abra el archivo de la solución **opcpublisher.sln** con Visual Studio y use las herramientas de depuración de allí.

Si necesita acceder al servidor OPC UA de OPC Publisher UA Publisher, asegúrese de que el firewall permite el acceso al puerto que escucha el servidor. El puerto predeterminado es 62222.

## <a name="control-the-application-remotely"></a>Control de la aplicación de forma remota

Los nodos se pueden configurar para la publicación mediante métodos directos de IoT Hub.

OPC Publisher implementa unas llamadas a métodos directos de IoT Hub adicionales para la lectura de:

- Información general.
- Información de diagnóstico sobre las sesiones de OPC, las suscripciones y los elementos supervisados.
- Información de diagnóstico sobre los eventos y los mensajes de IoT Hub.
- El registro de inicio.
- Las últimas 100 líneas del registro.
- El apagado de la aplicación.

Los siguientes repositorios de GitHub contienen herramientas para [configurar los nodos para la publicación](https://github.com/Azure-Samples/iot-edge-opc-publisher-nodeconfiguration) y para [leer la información de diagnóstico](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics). Ambas herramientas también están disponibles como contenedores de Docker Hub.

## <a name="use-a-sample-opc-ua-server"></a>Uso de un servidor OPC UA de ejemplo

Si no tiene un servidor OPC UA real, puede usar el [ejemplo de PLC OPC UA](https://github.com/Azure-Samples/iot-edge-opc-plc) para empezar a trabajar. Este ejemplo de PLC también está disponible en Docker Hub.

Implementa varias etiquetas que generan datos aleatorios y etiquetas con anomalías. Si tiene que simular valores de etiqueta adicionales, puede ampliar el ejemplo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a ejecutar OPC Publisher, los siguientes pasos recomendados son obtener información sobre [OPC Twin](overview-opc-twin.md) y [OPC Vault](overview-opc-vault.md).
