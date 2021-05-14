---
title: Información sobre la compatibilidad con AMQP de Azure IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: compatibilidad con dispositivos que se conectan a puntos de conexión orientados a dispositivos y orientados a servicios de IoT Hub mediante el protocolo AMQP. Incluye información sobre la compatibilidad integrada con AMQP de los SDK de dispositivo IoT de Azure.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: f0aaa996d6520a69f5546976d89c7cc4aa802d38
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109481362"
---
# <a name="communicate-with-your-iot-hub-by-using-the-amqp-protocol"></a>Comunicación con el centro de IoT mediante el protocolo AMQP

Azure IoT Hub admite [OASIS Advanced Message Queuing Protocol (AMQP) versión 1.0](https://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf) para ofrecer una variedad de funcionalidades mediante puntos de conexión orientados a dispositivos y orientados a servicios. Este documento describe el uso de clientes de AMQP para conectarse a un centro de IoT para usar la funcionalidad de IoT Hub.

## <a name="service-client"></a>Cliente del servicio

### <a name="connect-and-authenticate-to-an-iot-hub-service-client"></a>Conexión y autenticación en un centro de IoT (cliente del servicio)

Para conectarse a un centro de IoT mediante el uso de AMQP, un cliente puede utilizar la [seguridad basada en notificaciones (CBS)](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc) o la [autenticación de Nivel de seguridad y autenticación simples (SASL)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).

La siguiente información es necesaria para el cliente del servicio:

| Información | Value |
|-------------|--------------|
| Nombre de host del centro de IoT | `<iot-hub-name>.azure-devices.net` |
| Nombre de clave | `service` |
| Clave de acceso | Una clave principal o secundaria asociada al servicio. |
| Firma de acceso compartido | Una firma de acceso compartido de corta duración en el siguiente formato: `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`. Para obtener el código para generar esta firma, consulte [Control del acceso a IoT Hub](./iot-hub-dev-guide-sas.md#security-token-structure).

El siguiente fragmento de código utiliza la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python) para conectarse a un centro de IoT mediante un vínculo de emisor.

```python
import uamqp
import urllib
import time

# Use generate_sas_token implementation available here:
# https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '<operation-link-name>'  # example: '/messages/devicebound'

username = '{policy_name}@sas.root.{iot_hub_name}'.format(
    iot_hub_name=iot_hub_name, policy_name=policy_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

# Create a send or receive client
send_client = uamqp.SendClient(uri, debug=True)
receive_client = uamqp.ReceiveClient(uri, debug=True)
```

### <a name="invoke-cloud-to-device-messages-service-client"></a>Invocación de mensajes de la nube al dispositivo (cliente del servicio)

Para obtener información sobre el intercambio de mensajes de la nube al dispositivo entre el servicio y el centro de IoT y entre el dispositivo y el centro de IoT, consulte [Envío de mensajes de la nube al dispositivo desde IoT Hub](iot-hub-devguide-messages-c2d.md). El cliente del servicio utiliza dos vínculos para enviar mensajes y recibir comentarios sobre los mensajes enviados previamente desde los dispositivos, como se describe en la tabla siguiente:

| Creado por | Tipo de vínculo | Ruta de acceso del vínculo | Descripción |
|------------|-----------|-----------|-------------|
| Servicio | Vínculo de emisor | `/messages/devicebound` | El servicio envía a este vínculo los mensajes de la nube al dispositivo destinados a dispositivos. Los mensajes enviados mediante este vínculo tienen la propiedad `To` establecida en la ruta de acceso del vínculo de receptor del dispositivo de destino, `/devices/<deviceID>/messages/devicebound`. |
| Servicio | Vínculo de receptor | `/messages/serviceBound/feedback` | El servicio recibe en este vínculo los mensajes de comentarios de finalización, rechazo y abandono que proceden de los dispositivos. Para más información acerca de los mensajes de comentarios, consulte [Envío de mensajes de la nube al dispositivo desde IoT Hub](./iot-hub-devguide-messages-c2d.md#message-feedback). |

El fragmento de código siguiente muestra cómo crear un mensaje de la nube al dispositivo y enviarlo a un dispositivo mediante el uso de la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python).

```python
import uuid
# Create a message and set message property 'To' to the device-bound link on device
msg_id = str(uuid.uuid4())
msg_content = b"Message content goes here!"
device_id = '<device-id>'
to = '/devices/{device_id}/messages/devicebound'.format(device_id=device_id)
ack = 'full'  # Alternative values are 'positive', 'negative', and 'none'
app_props = {'iothub-ack': ack}
msg_props = uamqp.message.MessageProperties(message_id=msg_id, to=to)
msg = uamqp.Message(msg_content, properties=msg_props,
                    application_properties=app_props)

# Send the message by using the send client that you created and connected to the IoT hub earlier
send_client.queue_message(msg)
results = send_client.send_all_messages()

# Close the client if it's not needed
send_client.close()
```

Para recibir comentarios, el cliente del servicio crea un vínculo de receptor. El fragmento de código siguiente muestra cómo crear un vínculo mediante el uso de la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python):

```python
import json

operation = '/messages/serviceBound/feedback'

# ...
# Re-create the URI by using the preceding feedback path and authenticate it
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
batch = receive_client.receive_message_batch(max_batch_size=10)
for msg in batch:
    print('received a message')
    # Check content_type in message property to identify feedback messages coming from device
    if msg.properties.content_type == 'application/vnd.microsoft.iothub.feedback.json':
        msg_body_raw = msg.get_data()
        msg_body_str = ''.join(msg_body_raw)
        msg_body = json.loads(msg_body_str)
        print(json.dumps(msg_body, indent=2))
        print('******************')
        for feedback in msg_body:
            print('feedback received')
            print('\tstatusCode: ' + str(feedback['statusCode']))
            print('\toriginalMessageId: ' + str(feedback['originalMessageId']))
            print('\tdeviceId: ' + str(feedback['deviceId']))
            print
    else:
        print('unknown message:', msg.properties.content_type)
```

Como se muestra en el código anterior, un mensaje de comentarios de la nube al dispositivo tiene un tipo de contenido *application/vnd.microsoft.iothub.feedback.json*. Puede usar las propiedades del cuerpo JSON del mensaje para deducir el estado de entrega del mensaje original:

* La clave `statusCode` del cuerpo de comentarios tiene uno de los siguientes valores: *Success*, *Expired*, *DeliveryCountExceeded*, *Rejected* o *purged*.

* La clave `deviceId` del cuerpo de los comentarios tiene el identificador del dispositivo de destino.

* La clave `originalMessageId` del cuerpo de los comentarios tiene el identificador del mensaje de la nube al dispositivo original enviado por el servicio. Puede usar este estado de entrega para correlacionar los comentarios y los mensajes de la nube al dispositivo.

### <a name="receive-telemetry-messages-service-client"></a>Recepción de mensajes de telemetría (cliente del servicio)

De forma predeterminada, el centro de IoT almacena los mensajes de telemetría del dispositivo ingeridos en un centro de eventos integrado. El cliente del servicio puede usar el protocolo AMQP para recibir los eventos almacenados.

Para ello, el cliente del servicio primero debe conectarse al punto de conexión del centro de IoT y recibir una dirección de redireccionamiento a los centros de eventos integrados. El cliente del servicio, a continuación, usa la dirección proporcionada para conectarse al centro de eventos integrado.

En cada paso, el cliente debe presentar las siguientes piezas de información:

* Credenciales de servicio válidas (token de firma de acceso compartido del servicio).

* Una ruta de acceso con el formato correcto para la partición del grupo de consumidores desde el que intenta recuperar mensajes. Para un grupo de consumidores y un identificador de partición determinados, la ruta de acceso tiene el formato siguiente: `/messages/events/ConsumerGroups/<consumer_group>/Partitions/<partition_id>` (el grupo de consumidores predeterminado es `$Default`).

* Un predicado de filtro opcional para designar un punto de partida en la partición. Este predicado puede venir en forma de un número de secuencia, un desplazamiento o una marca de tiempo de puesta en cola.

El siguiente fragmento de código utiliza la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python) para mostrar los pasos anteriores:

```python
import json
import uamqp
import urllib
import time

# Use the generate_sas_token implementation that's available here: https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '/messages/events/ConsumerGroups/{consumer_group}/Partitions/{p_id}'.format(
    consumer_group='$Default', p_id=0)

username = '{policy_name}@sas.root.{iot_hub_name}'.format(
    policy_name=policy_name, iot_hub_name=iot_hub_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

# Optional filtering predicates can be specified by using endpoint_filter
# Valid predicates include:
# - amqp.annotation.x-opt-sequence-number
# - amqp.annotation.x-opt-offset
# - amqp.annotation.x-opt-enqueued-time
# Set endpoint_filter variable to None if no filter is needed
endpoint_filter = b'amqp.annotation.x-opt-sequence-number > 2995'

# Helper function to set the filtering predicate on the source URI


def set_endpoint_filter(uri, endpoint_filter=''):
    source_uri = uamqp.address.Source(uri)
    source_uri.set_filter(endpoint_filter)
    return source_uri


receive_client = uamqp.ReceiveClient(
    set_endpoint_filter(uri, endpoint_filter), debug=True)
try:
    batch = receive_client.receive_message_batch(max_batch_size=5)
except uamqp.errors.LinkRedirect as redirect:
    # Once a redirect error is received, close the original client and recreate a new one to the re-directed address
    receive_client.close()

    sas_auth = uamqp.authentication.SASTokenAuth.from_shared_access_key(
        redirect.address, policy_name, access_key)
    receive_client = uamqp.ReceiveClient(set_endpoint_filter(
        redirect.address, endpoint_filter), auth=sas_auth, debug=True)

# Start receiving messages in batches
batch = receive_client.receive_message_batch(max_batch_size=5)
for msg in batch:
    print('*** received a message ***')
    print(''.join(msg.get_data()))
    print('\t: ' + str(msg.annotations['x-opt-sequence-number']))
    print('\t: ' + str(msg.annotations['x-opt-offset']))
    print('\t: ' + str(msg.annotations['x-opt-enqueued-time']))
```

Para un identificador de dispositivo determinado, el centro de IoT usa un hash del identificador de dispositivo para determinar en qué partición almacenar sus mensajes. El fragmento de código anterior muestra cómo se reciben eventos de una sola partición de este tipo. Sin embargo, tenga en cuenta que una aplicación típica a menudo necesita recuperar los eventos que se almacenan en todas las particiones del centro de eventos.

## <a name="device-client"></a>Cliente del dispositivo

### <a name="connect-and-authenticate-to-an-iot-hub-device-client"></a>Conexión y autenticación en un centro de IoT (cliente del dispositivo)

Para conectarse a un centro de IoT mediante el uso de AMQP, un dispositivo puede utilizar la [seguridad basada en notificaciones (CBS)](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc) o la [autenticación de Nivel de seguridad y autenticación simples (SASL)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).

La siguiente información es necesaria para el cliente del dispositivo:

| Información | Value |
|-------------|--------------|
| Nombre de host del centro de IoT | `<iot-hub-name>.azure-devices.net` |
| Clave de acceso | Una clave principal o secundaria asociada al dispositivo. |
| Firma de acceso compartido | Una firma de acceso compartido de corta duración en el siguiente formato: `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`. Para obtener el código para generar esta firma, consulte [Control del acceso a IoT Hub](./iot-hub-dev-guide-sas.md#security-token-structure).

El siguiente fragmento de código utiliza la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python) para conectarse a un centro de IoT mediante un vínculo de emisor.

```python
import uamqp
import urllib
import uuid

# Use generate_sas_token implementation available here:
# https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
device_id = '<device-id>'
access_key = '<primary-or-secondary-key>'
username = '{device_id}@sas.{iot_hub_name}'.format(
    device_id=device_id, iot_hub_name=iot_hub_name)
sas_token = generate_sas_token('{hostname}/devices/{device_id}'.format(
    hostname=hostname, device_id=device_id), access_key, None)

# e.g., '/devices/{device_id}/messages/devicebound'
operation = '<operation-link-name>'
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
send_client = uamqp.SendClient(uri, debug=True)
```

Se admiten las siguientes rutas de acceso de vínculo como operaciones de dispositivo:

| Creado por | Tipo de vínculo | Ruta de acceso del vínculo | Descripción |
|------------|-----------|-----------|-------------|
| Dispositivos | Vínculo de receptor | `/devices/<deviceID>/messages/devicebound` | Cada dispositivo de destino recibe en este vínculo los mensajes de la nube al dispositivo destinados a dispositivos. |
| Dispositivos | Vínculo de emisor | `/devices/<deviceID>/messages/events` | Los mensajes del dispositivo a la nube que se envían desde un dispositivo se envían mediante este vínculo. |
| Dispositivos | Vínculo de emisor | `/messages/serviceBound/feedback` | Comentarios de mensajes de la nube al dispositivo enviados al servicio mediante este vínculo por los dispositivos. |

### <a name="receive-cloud-to-device-commands-device-client"></a>Recepción de comandos de la nube al dispositivo (cliente del dispositivo)

Los comandos de la nube al dispositivo que se envían a los dispositivos llegan a un vínculo `/devices/<deviceID>/messages/devicebound`. Los dispositivos pueden recibir estos mensajes por lotes y usar la carga de datos del mensaje, las propiedades del mensaje, las anotaciones o las propiedades de la aplicación del mensaje según sea necesario.

El siguiente fragmento de código utiliza la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python)) para recibir mensajes de la nube al dispositivo por un dispositivo.

```python
# ...
# Create a receive client for the cloud-to-device receive link on the device
operation = '/devices/{device_id}/messages/devicebound'.format(
    device_id=device_id)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
while True:
    batch = receive_client.receive_message_batch(max_batch_size=5)
    for msg in batch:
        print('*** received a message ***')
        print(''.join(msg.get_data()))

        # Property 'to' is set to: '/devices/device1/messages/devicebound',
        print('\tto:                     ' + str(msg.properties.to))

        # Property 'message_id' is set to value provided by the service
        print('\tmessage_id:             ' + str(msg.properties.message_id))

        # Other properties are present if they were provided by the service
        print('\tcreation_time:          ' + str(msg.properties.creation_time))
        print('\tcorrelation_id:         ' +
              str(msg.properties.correlation_id))
        print('\tcontent_type:           ' + str(msg.properties.content_type))
        print('\treply_to_group_id:      ' +
              str(msg.properties.reply_to_group_id))
        print('\tsubject:                ' + str(msg.properties.subject))
        print('\tuser_id:                ' + str(msg.properties.user_id))
        print('\tgroup_sequence:         ' +
              str(msg.properties.group_sequence))
        print('\tcontent_encoding:       ' +
              str(msg.properties.content_encoding))
        print('\treply_to:               ' + str(msg.properties.reply_to))
        print('\tabsolute_expiry_time:   ' +
              str(msg.properties.absolute_expiry_time))
        print('\tgroup_id:               ' + str(msg.properties.group_id))

        # Message sequence number in the built-in event hub
        print('\tx-opt-sequence-number:  ' +
              str(msg.annotations['x-opt-sequence-number']))
```

### <a name="send-telemetry-messages-device-client"></a>Envío de mensajes de telemetría (cliente del dispositivo)

También puede enviar mensajes de telemetría desde un dispositivo mediante el uso de AMQP. El dispositivo puede proporcionar opcionalmente un diccionario de propiedades de la aplicación o varias propiedades del mensajes, como el identificador del mensaje.

El siguiente fragmento de código utiliza la [biblioteca uAMQP de Python](https://github.com/Azure/azure-uamqp-python) para enviar mensajes del dispositivo a la nube desde un dispositivo.

```python
# ...
# Create a send client for the device-to-cloud send link on the device
operation = '/devices/{device_id}/messages/events'.format(device_id=device_id)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username), urllib.quote_plus(sas_token), hostname, operation)

send_client = uamqp.SendClient(uri, debug=True)

# Set any of the applicable message properties
msg_props = uamqp.message.MessageProperties()
msg_props.message_id = str(uuid.uuid4())
msg_props.creation_time = None
msg_props.correlation_id = None
msg_props.content_type = None
msg_props.reply_to_group_id = None
msg_props.subject = None
msg_props.user_id = None
msg_props.group_sequence = None
msg_props.to = None
msg_props.content_encoding = None
msg_props.reply_to = None
msg_props.absolute_expiry_time = None
msg_props.group_id = None

# Application properties in the message (if any)
application_properties = { "app_property_key": "app_property_value" }

# Create message
msg_data = b"Your message payload goes here"
message = uamqp.Message(msg_data, properties=msg_props, application_properties=application_properties)

send_client.queue_message(message)
results = send_client.send_all_messages()

for result in results:
    if result == uamqp.constants.MessageState.SendFailed:
        print result
```

## <a name="additional-notes"></a>Notas adicionales

* Las conexiones AMQP se podrían interrumpir debido a un problema temporal de red o a la expiración del token de autenticación (generado en el código). El cliente del servicio debe controlar estas circunstancias y restablecer la conexión y los vínculos, si es necesario. Si expira un token de autenticación, el cliente puede evitar una caída de la conexión al renovar el token de forma proactiva antes de su expiración.

* Ocasionalmente, el cliente debe ser capaz de controlar correctamente el redireccionamiento del vínculo. Para comprender este tipo de operación, consulte la documentación del cliente de AMQP.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el protocolo AMQP, consulte la [especificación de AMQP v1.0](https://www.amqp.org/sites/amqp.org/files/amqp.pdf).

Para más información acerca de la mensajería de IoT Hub, consulte:

* [Mensajes de nube a dispositivo](./iot-hub-devguide-messages-c2d.md)
* [Compatibilidad con protocolos adicionales](iot-hub-protocol-gateway.md)
* [Compatibilidad con el protocolo Message Queuing Telemetry Transport (MQTT)](./iot-hub-mqtt-support.md)
