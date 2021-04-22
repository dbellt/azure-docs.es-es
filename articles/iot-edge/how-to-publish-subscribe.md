---
title: Publicación y suscripción con Azure IoT Edge | Microsoft Docs
description: Use el agente MQTT de IoT Edge para publicar y suscribirse a mensajes.
services: iot-edge
keywords: ''
author: kgremban
ms.author: kgremban
ms.reviewer: ebertra
ms.date: 11/09/2020
ms.topic: conceptual
ms.service: iot-edge
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: 25d4774144ff4ea601badb1fb71b51c8142def26
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304123"
---
# <a name="publish-and-subscribe-with-azure-iot-edge-preview"></a>Publicación y suscripción con Azure IoT Edge (versión preliminar)

[!INCLUDE [iot-edge-version-202011](../../includes/iot-edge-version-202011.md)]

Puede usar el agente MQTT de Azure IoT Edge para publicar y suscribirse a mensajes. En este artículo se muestra cómo conectarse a este agente, publicar y suscribirse a mensajes a través de temas definidos por el usuario y usar primitivos de mensajería de IoT Hub. El agente MQTT de IoT Edge está integrado en el centro de IoT Edge. Para obtener más información, consulte las [funcionalidades de intermediación del centro de IoT Edge](iot-edge-runtime.md).

> [!NOTE]
> El agente MQTT de IoT Edge se encuentra actualmente en versión preliminar pública.

## <a name="pre-requisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción válida.
- La [CLI de Azure](/cli/azure/) con la extensión de la CLI `azure-iot` instalada. Para obtener más información, consulte los [pasos de instalación de la extensión de Azure IoT para la CLI de Azure](/cli/azure/azure-cli-reference-for-iot).
- **IoT Hub** de SKU F1, S1, S2 o S3.
- Un **dispositivo IoT Edge con la versión 1.2 o superior**. Dado que el agente MQTT de IoT Edge se encuentra actualmente en versión preliminar pública, establezca las siguientes variables de entorno en true en el contenedor edgeHub para habilitar el agente MQTT:

   | Nombre | Value |
   | - | - |
   | `experimentalFeatures__enabled` | `true` |
   | `experimentalFeatures__mqttBrokerEnabled` | `true` |

- **Clientes de Mosquitto** instalados en el dispositivo IoT Edge. En este artículo se usan los clientes de Mosquitto populares [MOSQUITTO_PUB](https://mosquitto.org/man/mosquitto_pub-1.html) y [MOSQUITTO_SUB](https://mosquitto.org/man/mosquitto_sub-1.html). Se podrían usar otros clientes MQTT en su lugar. Para instalar los clientes de Mosquitto en un dispositivo Ubuntu, ejecute el siguiente comando:

    ```cmd
    sudo apt-get update && sudo apt-get install mosquitto-clients
    ```

    No instale el servidor de Mosquitto, ya que puede provocar el bloqueo de los puertos MQTT (8883 y 1883) y entrar en conflicto con el centro de IoT Edge.

## <a name="connect-to-iot-edge-hub"></a>Conexión al centro de IoT Edge

La conexión a un centro de IoT Edge sigue los mismos pasos que se describen en el [artículo de conexión a IoT Hub con un cliente MQTT genérico](../iot-hub/iot-hub-mqtt-support.md) o en el [artículo de la descripción conceptual de un centro de IoT Edge](iot-edge-runtime.md). Los pasos son los siguientes:

1. De manera opcional, el cliente MQTT establece una *conexión segura* con el centro de IoT Edge mediante Seguridad de la capa de transporte (TLS).
2. El cliente MQTT *se autentica* a sí mismo en el centro de IoT Edge.
3. El centro de IoT Edge *autoriza* al cliente MQTT según su directiva de autorización.

### <a name="secure-connection-tls"></a>Conexión segura (TLS)

La Seguridad de la capa de transporte (TLS) se usa para establecer una comunicación cifrada entre el cliente y el centro de IoT Edge.

Para deshabilitar TLS, use el puerto 1883 (MQTT) y enlace el contenedor edgeHub al puerto 1883.

Para habilitar TLS, si un cliente se conecta en el puerto 8883 (MQTTS) al agente MQTT, se iniciará un canal TLS. El agente envía su cadena de certificados que el cliente debe validar. Para validar la cadena de certificados, el certificado raíz del agente MQTT debe estar instalado como certificado de confianza en el cliente. Si el certificado raíz no es de confianza, el agente MQTT rechazará la biblioteca cliente con un error de comprobación de certificados. Los pasos que deben seguirse para instalar este certificado raíz del agente en el cliente son los mismos que en el caso de [puerta de enlace transparente](how-to-create-transparent-gateway.md) y se describen en la documentación de [preparación de un dispositivo de nivel inferior](how-to-connect-downstream-device.md#prepare-a-downstream-device).

### <a name="authentication"></a>Authentication

Para que un cliente MQTT se autentique a sí mismo, primero debe enviar un paquete CONNECT al agente MQTT para iniciar una conexión en su nombre. Este paquete proporciona tres fragmentos de información de autenticación: `client identifier`, `username` y `password`:

- El campo `client identifier` es el nombre del dispositivo o el nombre del módulo en IoT Hub. Utiliza la siguiente sintaxis:

  - Para un dispositivo: `<device_name>`

  - Para un módulo: `<device_name>/<module_name>`

   Para conectarse al agente MQTT, se debe registrar un dispositivo o módulo en IoT Hub.

   El agente no permitirá conexiones de varios clientes con las mismas credenciales. El agente desconectará al cliente ya conectado si un segundo cliente se conecta con las mismas credenciales.

- El campo `username` se deriva del nombre del dispositivo o del módulo, y del nombre de IoTHub al que pertenece el dispositivo mediante la siguiente sintaxis:

  - Para un dispositivo: `<iot_hub_name>.azure-devices.net/<device_name>/?api-version=2018-06-30`

  - Para un módulo: `<iot_hub_name>.azure-devices.net/<device_name>/<module_name>/?api-version=2018-06-30`

- El campo `password` del paquete CONNECT depende del modo de autenticación:

  - Al usar la [autenticación de claves simétricas](how-to-authenticate-downstream-device.md#symmetric-key-authentication), el campo `password` es un token de SAS.
  - Al usar la [autenticación de X.509 autofirmado](how-to-authenticate-downstream-device.md#x509-self-signed-authentication), el campo `password` no está presente. En este modo de autenticación, se requiere un canal TLS. El cliente debe conectarse al puerto 8883 para establecer una conexión TLS. Durante el protocolo de enlace TLS, el agente MQTT solicita un certificado de cliente. Este certificado se usa para comprobar la identidad del cliente y, por tanto, el campo `password` no es necesario posteriormente cuando se envía el paquete CONNECT. El envío tanto de un certificado de cliente como del campo de contraseña provocará un error y se cerrará la conexión. Las bibliotecas de MQTT y las bibliotecas cliente TLS suelen tener una manera de enviar un certificado de cliente al iniciar una conexión. Puede ver un ejemplo paso a paso en la sección [Uso de un certificado X.509 para la autenticación del cliente](how-to-authenticate-downstream-device.md#x509-self-signed-authentication).

Los módulos implementados por IoT Edge usan la [autenticación de claves simétricas](how-to-authenticate-downstream-device.md#symmetric-key-authentication) y pueden llamar a la [API de carga de trabajo de IoT Edge](https://github.com/Azure/iotedge/blob/40f10950dc65dd955e20f51f35d69dd4882e1618/edgelet/workload/README.md) local mediante programación para obtener un token de SAS, incluso cuando están sin conexión.

### <a name="authorization"></a>Authorization

Una vez que un cliente MQTT se autentica en el centro de IoT Edge, debe estar autorizado para conectarse. Una vez conectado, debe estar autorizado para publicar o suscribirse a temas concretos. El centro de IoT Edge concede estas autorizaciones en función de su directiva de autorización. La directiva de autorización es un conjunto de instrucciones expresadas como estructura JSON que se envía al centro de IoT Edge a través de su gemelo. Edite el gemelo de un centro de IoT Edge para configurar su directiva de autorización.

> [!NOTE]
> Para la versión preliminar pública, la edición de las directivas de autorización del agente MQTT solo está disponible a través de Visual Studio, Visual Studio Code o la CLI de Azure. Azure Portal actualmente no admite la edición del gemelo del centro de IoT Edge ni de su directiva de autorización.

Cada instrucción de la directiva de autorización consta de la combinación de `identities`, los efectos `allow` o `deny`, `operations` y `resources`:

- `identities` describir el asunto de la directiva. Debe asignarse al `client identifier` enviado por los clientes en su paquete CONNECT.
- Los efectos `allow` o `deny` definen si se permiten o deniegan las operaciones.
- `operations` define las acciones que se van a autorizar. `mqtt:connect`, `mqtt:publish` y `mqtt:subscribe` son las tres acciones admitidas hoy en día.
- `resources` define el objeto de la directiva. Puede tratarse de un tema o un patrón de tema definido con [caracteres comodín de MQTT](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718107).

A continuación se muestra un ejemplo de una directiva de autorización que explícitamente no permite que el cliente "rogue_client" se conecte, permite que cualquier cliente de Azure IoT se conecte, y permite que "sensor_1" publique en el tema `events/alerts`.

```json
{
   "$edgeHub":{
      "properties.desired":{
         "schemaVersion":"1.2",
         "routes":{
            "Route1":"FROM /messages/* INTO $upstream"
         },
         "storeAndForwardConfiguration":{
            "timeToLiveSecs":7200
         },
         "mqttBroker":{
            "authorizations":[
               {
                  "identities":[
                     "rogue_client"
                  ],
                  "deny":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               },
               {
                  "identities":[
                     "{{iot:identity}}"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               },
               {
                  "identities":[
                     "sensor_1"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:publish"
                        ],
                        "resources":[
                           "events/alerts"
                        ]
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

Hay un par de cosas que hay que tener en cuenta al escribir la directiva de autorización:

- Requiere el esquema de gemelos `$edgeHub`, versión 1.2.
- De manera predeterminada, se deniegan todas las operaciones.
- Las instrucciones de autorización se evalúan en el orden en que aparecen en la definición JSON. Se inicia examinando `identities` y, a continuación, seleccione las primeras instrucciones allow o deny que coincidan con la solicitud. En caso de conflictos entre las instrucciones allow y deny, gana la instrucción deny.
- Se pueden usar varias variables (por ejemplo, sustituciones) en la directiva de autorización:
    - `{{iot:identity}}` representa la identidad del cliente conectado actualmente. Por ejemplo, una identidad de dispositivo como `myDevice` o una identidad de módulo como `myEdgeDevice/SampleModule`.
    - `{{iot:device_id}}` representa la identidad del dispositivo conectado actualmente. Por ejemplo, una identidad de dispositivo como `myDevice` o la identidad del dispositivo donde se ejecuta un módulo, como `myEdgeDevice`.
    - `{{iot:module_id}}` representa la identidad del módulo conectado actualmente. Esta variable está en blanco para los dispositivos conectados o una identidad de módulo como `SampleModule`.
    - `{{iot:this_device_id}}` representa la identidad del dispositivo IoT Edge que ejecuta la directiva de autorización. Por ejemplo, `myIoTEdgeDevice`.

Las autorizaciones para los temas de IoT Hub se tratan de forma ligeramente diferente que para los temas definidos por el usuario. Estos son los puntos clave para recordar:

- Los dispositivos o módulos de Azure IoT necesitan una regla de autorización explícita para conectarse al agente MQTT del centro de IoT Edge. A continuación se proporciona una directiva de autorización de conexión predeterminada.
- Los dispositivos o módulos Azure IoT pueden acceder a sus propios temas de IoT Hub de manera predeterminada sin ninguna regla de autorización explícita. Sin embargo, las autorizaciones proceden de relaciones de elementos primarios y secundarios en ese caso, y estas relaciones deben estar establecidas. Los módulos de IoT Edge se establecen automáticamente como elementos secundarios de su dispositivo IoT Edge, pero los dispositivos deben establecerse explícitamente como elementos secundarios de su puerta de enlace IoT Edge.

Esta es una directiva de autorización predeterminada que se puede usar para habilitar todos los dispositivos o módulos de Azure IoT para **conectarse** al agente:

```json
{
   "$edgeHub":{
      "properties.desired":{
         "schemaVersion":"1.2",
         "mqttBroker":{
            "authorizations":[
               {
                  "identities": [
                     "{{iot:identity}}"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

Ahora que ya comprende cómo conectarse al agente MQTT de IoT Edge, veamos cómo se puede usar para publicar y suscribirse a mensajes primero en temas definidos por el usuario, luego, en los temas del centro de IoT y, finalmente, en otro agente MQTT.

## <a name="publish-and-subscribe-on-user-defined-topics"></a>Publicación y suscripción a temas definidos por el usuario

En este artículo, usará un cliente denominado **sub_client** que se suscribe a un tema, y otro cliente denominado **pub_client** que publica en un tema. Usaremos la [autenticación de claves simétricas](how-to-authenticate-downstream-device.md#symmetric-key-authentication), pero puede hacer lo mismo con la [autenticación con X.509 autofirmado](how-to-authenticate-downstream-device.md#x509-self-signed-authentication) o con la [autenticación X.509 firmado por una entidad de certificación](./how-to-authenticate-downstream-device.md#x509-ca-signed-authentication).

### <a name="create-publisher-and-subscriber-clients"></a>Creación de los clientes publicador y suscriptor

Cree dos dispositivos IoT en IoT Hub y obtenga sus contraseñas. Use la CLI de Azure desde el terminal para lo siguiente:

1. Cree dos dispositivos IoT en IoT Hub y establézcalos como primarios del dispositivo IoT Edge:

    ```azurecli-interactive
    az iot hub device-identity create --device-id  sub_client --hub-name <iot_hub_name> --pd <edge_device_id>
    az iot hub device-identity create --device-id  pub_client --hub-name <iot_hub_name> --pd <edge_device_id>
    ```

2. Para obtener sus contraseñas, genere un token de SAS:

    - Para un dispositivo:
    
       ```azurecli-interactive
       az iot hub generate-sas-token -n <iot_hub_name> -d <device_name> --key-type primary --du 3600
       ```
    
       donde 3600 es la duración del token de SAS en segundos (por ejemplo, 3600 = 1 hora).
    
    - Para un módulo:
    
       ```azurecli-interactive
       az iot hub generate-sas-token -n <iot_hub_name> -d <device_name> -m <module_name> --key-type primary --du 3600
       ```
    
       donde 3600 es la duración del token de SAS en segundos (por ejemplo, 3600 = 1 hora).

3. Copie el token de SAS, que es el valor correspondiente a la clave "sas" de la salida. A continuación se muestra un ejemplo de salida del comando de la CLI de Azure anterior:

    ```
    {
       "sas": "SharedAccessSignature sr=example.azure-devices.net%2Fdevices%2Fdevice_1%2Fmodules%2Fmodule_a&sig=H5iMq8ZPJBkH3aBWCs0khoTPdFytHXk8VAxrthqIQS0%3D&se=1596249190"
    }
    ```

### <a name="authorize-publisher-and-subscriber-clients"></a>Autorización de los clientes publicador y suscriptor

Para autorizar al editor y suscriptor, edite el gemelo del centro de IoT Edge mediante la creación de una implementación con la CLI de Azure, Visual Studio o Visual Studio Code para incluir la siguiente directiva de autorización:

```json
{
   "$edgeHub":{
      "properties.desired":{
         "schemaVersion":"1.2",
         "mqttBroker":{
            "authorizations":[
               {
                  "identities": [
                     "{{iot:identity}}"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               },
               {
                  "identities": [
                     "<iot_hub_name>.azure-devices.net/sub_client"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:subscribe"
                        ],
                        "resources":[
                           "test_topic"
                        ]
                     }
                  ],
               },
               {
                  "identities": [
                     "<iot_hub_name>.azure-devices.net/pub_client"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:publish"
                        ],
                        "resources":[
                           "test_topic"
                        ]
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

### <a name="symmetric-keys-authentication-without-tls"></a>Autenticación de claves simétricas sin TLS

#### <a name="subscribe"></a>Suscribirse

Conecte el cliente MQTT **sub_client** al agente MQTT y suscríbase a `test_topic` ejecutando el siguiente comando en el dispositivo IoT Edge:

```bash
mosquitto_sub \
    -t "test_topic" \
    -i "sub_client" \
    -u "<iot_hub_name>.azure-devices.net/sub_client/?api-version=2018-06-30" \
    -P "<sas_token>" \
    -h "<edge_device_address>" \
    -V mqttv311 \
    -p 1883
```

donde `<edge_device_address>` = `localhost` en este ejemplo, ya que el cliente se ejecuta en el mismo dispositivo que IoT Edge.

Tenga en cuenta que en este primer ejemplo se usa el puerto 1883 (MQTT), por ejemplo, sin TLS. En la sección siguiente se muestra otro ejemplo con el puerto 8883 (MQTTS), por ejemplo, con TLS habilitado.

El cliente MQTT **sub_client** se ha iniciado y está esperando los mensajes entrantes en `test_topic`.

#### <a name="publish"></a>Publicar

Conecte el cliente MQTT **pub_client** al agente MQTT y publique un mensaje en el mismo `test_topic` que antes mediante la ejecución del siguiente comando en el dispositivo IoT Edge desde otro terminal:

```bash
mosquitto_pub \
    -t "test_topic" \
    -i "pub_client" \
    -u "<iot_hub_name>.azure-devices.net/pub_client/?api-version=2018-06-30" \
    -P "<sas_token>" \
    -h "<edge_device_address>" \
    -V mqttv311 \
    -p 1883 \
    -m "hello"
```

donde `<edge_device_address>` = `localhost` en este ejemplo, ya que el cliente se ejecuta en el mismo dispositivo que IoT Edge.

Al ejecutar el comando, el cliente MQTT **sub_client** recibe el mensaje "hello".

### <a name="symmetric-keys-authentication-with-tls"></a>Autenticación de claves simétricas con TLS

Para habilitar TLS, el puerto debe cambiarse de 1883 (MQTT) a 8883 (MQTTS), y los clientes deben tener el certificado raíz del agente MQTT para poder validar la cadena de certificados enviada por el agente MQTT. Para ello, siga los pasos indicados en la sección [Conexión segura (TLS)](#secure-connection-tls).

Dado que los clientes se ejecutan en el mismo dispositivo que el agente MQTT en el ejemplo anterior, se aplican los mismos pasos para habilitar TLS simplemente cambiando el número de puerto de 1883 (MQTT) a 8883 (MQTTS).

## <a name="publish-and-subscribe-on-iot-hub-topics"></a>Publicación y suscripción en temas de IoT Hub

Los [SDK de dispositivos de Azure IoT](https://github.com/Azure/azure-iot-sdks) ya permiten a los clientes realizar operaciones de IoT Hub, pero no permiten la publicación ni suscripción en temas definidos por el usuario. Las operaciones de IoT Hub se pueden realizar con cualquier cliente MQTT mediante la semántica de publicación y suscripción, siempre que se respeten los protocolos de primitivos de IoT Hub. Revisaremos las características específicas para comprender cómo funcionan estos protocolos.

### <a name="send-telemetry-data-to-iot-hub"></a>Envío de datos de telemetría a IoT Hub

El envío de datos de telemetría a IoT Hub es similar a la publicación en un tema definido por el usuario, pero con un tema de IoT Hub específico:

- En el caso de un dispositivo, los datos de telemetría se envían en el tema: `devices/<device_name>/messages/events/`
- En el caso de un módulo, los datos de telemetría se envían en el tema: `devices/<device_name>/<module_name>/messages/events/`

Además, cree una ruta como `FROM /messages/* INTO $upstream` para enviar los datos de telemetría desde el agente MQTT de IoT Edge a IoT Hub. Para obtener más información sobre el enrutamiento, consulte [Declaración de rutas](module-composition.md#declare-routes).

### <a name="get-twin"></a>Obtención del gemelo

La obtención del dispositivo/módulo gemelo no es un patrón MQTT típico. El cliente debe emitir una solicitud para el gemelo que va a servir IoT Hub.

Para recibir gemelos, el cliente debe suscribirse a un tema específico de IoT Hub `$iothub/twin/res/#`. El nombre de este tema se hereda de IoT Hub, y todos los clientes deben suscribirse al mismo tema. No significa que los dispositivos ni módulos reciban los gemelos unos de otros. IoT Hub y el centro de IoT Edge saben qué gemelo se debe entregar a dónde, incluso si todos los dispositivos escuchan el mismo nombre de tema. 

Una vez que se realice la suscripción, el cliente  debe solicitar el gemelo mediante la publicación de un mensaje en el tema específico de IoT Hub `$iothub/twin/GET/?rid=<request_id>/#`, donde `<request_id>` es un identificador arbitrario. IoT Hub enviará entonces su respuesta con los datos solicitados en el tema `$iothub/twin/res/200/?rid=<request_id>`, al que se suscribe el cliente. Así es como un cliente puede emparejar sus solicitudes con las respuestas.

### <a name="receive-twin-patches"></a>Recepción de revisiones de gemelos

Para recibir las revisiones de gemelos, un cliente debe suscribirse al tema especial de IoTHub `$iothub/twin/PATCH/properties/desired/#`. Una vez que se realiza la suscripción, el cliente recibe las revisiones de gemelos enviadas por IoT Hub en este tema. 

### <a name="receive-direct-methods"></a>Recepción de métodos directos

Recibir un método directo es similar a recibir gemelos completos, con la adición de que el cliente tiene que confirmar que ha recibido la llamada. En primer lugar, el cliente se suscribe al tema especial de IoT Hub `$iothub/methods/POST/#`. Después, una vez que se recibe un método directo en este tema, el cliente debe extraer el identificador de solicitud `rid` del subtema en el que se recibe el método directo y, por último, publicar un mensaje de confirmación en el tema especial de IoT Hub `$iothub/methods/res/200/<request_id>`.

### <a name="send-direct-methods"></a>Envío de métodos directos

El envío de un método directo es una llamada HTTP y, por tanto, no pasa por el agente MQTT. Para enviar un método directo a IoT Hub, consulte [Conocimiento e invocación de los métodos directos](../iot-hub/iot-hub-devguide-direct-methods.md). Para enviar un método directo localmente a otro módulo, consulte este [ejemplo de invocación de método directo del SDK de Azure IoT en C#](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/src/ModuleClient.cs#L597).

## <a name="publish-and-subscribe-between-mqtt-brokers"></a>Publicación y suscripción entre agentes MQTT

Para conectar dos agentes MQTT, el centro de IoT Edge incluye un puente MQTT. Un puente MQTT se usa normalmente para conectar un agente MQTT en ejecución con otro agente MQTT. Por lo general, solo se inserta en otro agente un subconjunto del tráfico local.

> [!NOTE]
> Actualmente, el puente del centro de IoT Edge solo se puede usar entre dispositivos IoT Edge anidados. No se puede usar para enviar datos a IoT Hub, ya que IoT Hub no es un agente MQTT con todas las características. Para obtener más información sobre la compatibilidad de características del agente MQTT de IoT Hub, consulte [Comunicación con la instancia de IoT Hub mediante el protocolo MQTT](../iot-hub/iot-hub-mqtt-support.md). Para obtener más información sobre el anidamiento de dispositivos IoT Edge, consulte [Conexión de un dispositivo IoT Edge de nivel inferior a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-iot-edge-device.md#configure-iot-edge-on-devices) 

En una configuración anidada, el puente MQTT del centro de IoT Edge actúa como cliente del agente MQTT primario, por lo que se deben establecer reglas de autorización en EdgeHub primario para permitir que EdgeHub secundario publique y se suscriba a temas específicos definidos por el usuario para los que está configurado el puente.

El puente MQTT de IoT Edge se configura a través de una estructura JSON que se envía al centro de IoT Edge a través de su gemelo. Edite el gemelo de un centro de IoT Edge para configurar su puente MQTT.

> [!NOTE]
> Para la versión preliminar pública, la configuración del puente MQTT solo está disponible a través de Visual Studio, Visual Studio Code o la CLI de Azure. Azure Portal actualmente no admite la edición del gemelo del centro de IoT Edge ni de su configuración de puente MQTT.

El puente MQTT puede configurarse para conectar un agente MQTT de IoT Edge Hub a varios agentes externos. Para cada agente externo, se requiere la siguiente configuración:

- `endpoint` es la dirección del agente MQTT remoto con el que se va a conectar. Actualmente solo se admiten dispositivos IoT Edge primarios y se definen mediante la variable `$upstream`.
- `settings` define los temas a los que se va a aplicar el puente para un punto de conexión. Puede haber varias configuraciones por punto de conexión, y se usan los valores siguientes para configurarlo:
    - `direction`: Ya sea `in` para suscribirse a los temas del agente remoto, o `out` para publicar en los temas del agente remoto
    - `topic`: Patrón de tema principal con el que se debe asociar. Se pueden usar [caracteres comodín de MQTT](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718107) para definir este patrón. Se pueden aplicar distintos prefijos a este patrón de tema en el agente local y en el agente remoto.
    - `outPrefix`: Prefijo que se aplica a al patrón `topic` en el agente remoto.
    - `inPrefix`: Prefijo que se aplica a al patrón `topic` en el agente local.

A continuación se muestra un ejemplo de configuración de puente MQTT de IoT Edge que vuelve a publicar todos los mensajes recibidos en los temas `alerts/#` de un dispositivo IoT Edge primario en un dispositivo IoT Edge secundario en los mismos temas, y vuelve a publicar todos los mensajes enviados en los temas `/local/telemetry/#` de un dispositivo IoT Edge secundario en un dispositivo IoT Edge primario en los temas `/remote/messages/#`.

```json
{
    "schemaVersion": "1.2",
    "mqttBroker": {
        "bridges": [{
            "endpoint": "$upstream",
            "settings": [{
                    "direction": "in",
                    "topic": "alerts/#"
                },
                {
                    "direction": "out",
                    "topic": "#",
                    "inPrefix": "/local/telemetry/",
                    "outPrefix": "/remote/messages/"
                }
            ]
        }]
    }
}
```
Otras notas sobre el puente MQTT del centro de IoT Edge:
- El protocolo MQTT se usará automáticamente como protocolo ascendente cuando se use el agente MQTT y esa instancia de IoT Edge se use en una configuración anidada, por ejemplo, con un `parent_hostname` especificado. Para obtener más información sobre los protocolos ascendentes, consulte [Comunicación en la nube](iot-edge-runtime.md#cloud-communication). Para obtener más información sobre las configuraciones anidadas, consulte [Conexión de un dispositivo IoT Edge de nivel inferior a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-iot-edge-device.md#configure-iot-edge-on-devices)

## <a name="next-steps"></a>Pasos siguientes

[Información del centro de IoT Edge](iot-edge-runtime.md#iot-edge-hub)
