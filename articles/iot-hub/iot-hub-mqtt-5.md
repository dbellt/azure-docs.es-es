---
title: Compatibilidad con MQTT 5 de Azure IoT Hub (versión preliminar)
description: Más información sobre la compatibilidad con MQTT 5 de IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 11/19/2020
ms.author: jlian
ms.openlocfilehash: fbe51d51da55d6708bfae96d0fec15eec9896602
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109484854"
---
# <a name="iot-hub-mqtt-5-support-overview-preview"></a>Información general sobre la compatibilidad con MQTT 5 de IoT Hub (versión preliminar)

**Versión**: 2.0 **api-version:** 2020-10-01-preview

En este documento se define la API del plano de datos de IoT Hub a través del protocolo MQTT versión 5.0. Consulte [Referencia de API](iot-hub-mqtt-5-reference.md) para obtener definiciones completas de esta API.

## <a name="prerequisites"></a>Requisitos previos

- [Habilite el modo de versión preliminar](iot-hub-preview-mode.md) en una nueva instancia de IoT Hub para probar MQTT 5.
- Se requiere un conocimiento previo de la [especificación MQTT 5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html).

## <a name="level-of-support-and-limitations"></a>Nivel de compatibilidad y limitaciones

La compatibilidad de IoT Hub para MQTT 5 está en versión preliminar y está limitada de las siguientes maneras (se comunica al cliente a través de `CONNACK` propiedades, a menos que se indique explícitamente lo contrario):

- Todavía no se admite el [SDK de dispositivo Azure IoT Hub](iot-hub-devguide-sdks.md) oficial.
- No se admiten los identificadores de suscripción.
- No se admiten la suscripciones compartidas.
- `RETAIN` no se admite.
- `Maximum QoS` es `1`.
- `Maximum Packet Size` es `256 KiB` (sujeto a más restricciones por operación).
- No se admiten los identificadores de cliente asignados.
- `Keep Alive` está limitado a `19 min` (retraso máximo para la comprobación de ejecución: `28.5 min`).
- `Topic Alias Maximum` es `10`.
- `Response Information` no se admite; `CONNACK` no devuelve la propiedad `Response Information` aunque `CONNECT` contenga la propiedad `Request Response Information`.
- `Receive Maximum` [número máximo permitido de paquetes `PUBLISH` no reconocidos pendientes (en la dirección del cliente-servidor) con `QoS: 1`] es `16`.
- Un solo cliente no puede tener más de `50` suscripciones.
  Cuando se alcance el límite, `SUBACK` devolverá el código de motivo `0x97` (cuota superada) para las suscripciones.

## <a name="connection-lifecycle"></a>Ciclo de vida de las conexiones

### <a name="connection"></a>Conexión

Para conectar un cliente a IoT Hub mediante esta API, establezca la conexión de acuerdo con la especificación MQTT 5.
El cliente debe enviar el paquete `CONNECT` en un plazo de 30 segundos después del protocolo de enlace TLS correcto; de lo contrario, el servidor cerrará la conexión.
A continuación se muestra un ejemplo de paquete `CONNECT`:

```yaml
-> CONNECT
    Protocol_Version: 5
    Clean_Start: 0
    Client_Id: D1
    Authentication_Method: SAS
    Authentication_Data: {SAS bytes}
    api-version: 2020-10-10
    host: abc.azure-devices.net
    sas-at: 1600987795320
    sas-expiry: 1600987195320
    client-agent: artisan;Linux
```

- La propiedad `Authentication Method` es necesaria e identifica el método de autenticación que se utiliza. Para obtener más información sobre el método de autenticación, consulte [Autenticación](#authentication).
- El control de la propiedad `Authentication Data` depende de `Authentication Method`. Si `Authentication Method` se establece en `SAS`, la propiedad `Authentication Data` es necesaria y debe contener una firma válida. Para obtener más información sobre los datos de autenticación, consulte [Autenticación](#authentication).
- La propiedad `api-version` es necesaria y debe establecerse en el valor de la versión de API proporcionada en el encabezado de esta especificación para que la especificación se aplique.
- La propiedad `host` define el nombre de host del inquilino. Es necesario a menos que se haya presentado la extensión SNI en el registro de Hello del cliente durante el protocolo de enlace TLS.
- `sas-at` define la hora de conexión.
- `sas-expiry` define la hora de expiración de la firma SAS proporcionada.
- De manera opcional, `client-agent` comunica información sobre el cliente que crea la conexión.

> [!NOTE]
> `Authentication Method` y otras propiedades de la especificación con nombres en mayúsculas son propiedades de primera clase en MQTT 5: se describen con detalle en la especificación MQTT 5. `api-version` y otras propiedades con guiones son propiedades de usuario específicas de IoT Hub API.

IoT Hub responde con el paquete `CONNACK` una vez que termina con la autenticación y la recuperación de datos para admitir la conexión. Si la conexión se establece correctamente, `CONNACK` tendrá el siguiente aspecto:

```yaml
<- CONNACK
    Session_Present: 1
    Reason_Code: 0x00
    Session_Expiry_Interval: 0xFFFFFFFF # included only if CONNECT specified value less than 0xFFFFFFFF and more than 0x00
    Receive_Maximum: 16
    Maximum_QoS: 1
    Retain_Available: 0
    Maximum_Packet_Size: 262144
    Topic_Alias_Maximum: 10
    Subscription_Identifiers_Available: 0
    Shared_Subscriptions_Available: 0
    Server_Keep_Alive: 1140 # included only if client did not specify Keep Alive or if it specified a bigger value
```

Estas propiedades de paquete `CONNACK` siguen al [especificación MQTT 5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901080). Reflejan las capacidades de IoT Hub.

### <a name="authentication"></a>Authentication

La propiedad `Authentication Method` en el cliente `CONNECT` define el tipo de autenticación que usa para esta conexión:

- `SAS`: la Firma de acceso compartido se proporciona en la propiedad `Authentication Data` de `CONNECT`.
- `X509`: el cliente se basa en la autenticación de certificados de cliente.

 Se produce un error de autenticación si el método de autenticación no coincide con el método configurado del cliente en IoT Hub.

> [!NOTE]
> Esta API requiere que la propiedad `Authentication Method` se establezca en el paquete `CONNECT`. Si no se proporciona la propiedad `Authentication Method`, se produce un error de conexión con la respuesta `Bad Request`.

No se admite la autenticación de nombre de usuario/contraseña usada en versiones anteriores de la API.

#### <a name="sas"></a>SAS

Con la autenticación basada en SAS, el cliente debe proporcionar la firma del contexto de conexión. Esto demuestra la autenticidad de la conexión MQTT. La firma debe basarse en una de las dos claves de autenticación de la configuración del cliente en IoT Hub o en una de las dos claves de acceso compartido de una [directiva de acceso compartido](iot-hub-dev-guide-sas.md).

La cadena que se va a firmar debe tener el siguiente formato:

```text
{host name}\n
{Client Id}\n
{sas-policy}\n
{sas-at}\n
{sas-expiry}\n
```

- `host name` se deriva de la extensión SNI (que presenta el cliente en el registro de Hello del cliente durante el protocolo de enlace TLS) o de la propiedad de usuario `host` del paquete `CONNECT`.
- `Client Id` es el identificador de cliente del paquete `CONNECT`.
- `sas-policy`: si está presente, define la directiva de acceso de IoT Hub usada para la autenticación. Se codifica como propiedad de usuario en el paquete `CONNECT`. Opcional: si se omite, se usará la configuración de autenticación del registro de dispositivos en su lugar.
- `sas-at`: si está presente, especifica la hora de conexión (hora actual). Se codifica como propiedad de usuario de tipo `time` en el paquete `CONNECT`.
- `sas-expiry` define la hora de expiración de la autenticación. Se trata de una propiedad de usuario de tipo `time` en el paquete `CONNECT`. Esta propiedad es obligatoria.

En el caso de los parámetros opcionales, si se omite, se debe usar una cadena vacía en lugar de la cadena para firmar.

HMAC-SHA256 se usa para aplicar un algoritmo hash a la cadena basado en una de las claves simétricas del dispositivo. A continuación, el valor hash se establece como el valor de la propiedad `Authentication Data`.

#### <a name="x509"></a>X509

Si la propiedad `Authentication Method` se establece en `X509`, IoT Hub autentica la conexión de acuerdo con el certificado de cliente proporcionado.

#### <a name="reauthentication"></a>Reautenticación

Si se usa la autenticación basada en SAS, se recomienda usar tokens de autenticación de corta duración. Para mantener la conexión autenticada y evitar la desconexión debido a la expiración, el cliente debe volver a autenticarse mediante el envío del paquete `AUTH` con `Reason Code: 0x19` (reautenticación):

```yaml
-> AUTH
    Reason_Code: 0x19
    Authentication_Method: SAS
    Authentication_Data: {SAS bytes}
    sas-at: {current time}
    sas-expiry: {SAS expiry time}
```

Reglas:

- El valor de `Authentication Method` debe coincidir con el usado para la autenticación inicial.
- Si la conexión se autenticó originalmente mediante SAS basado en la directiva de acceso compartido, la firma utilizada en la reautenticación debe basarse en la misma directiva.

Si la reautenticación se realiza correctamente, IoT Hub envía el paquete `AUTH` con `Reason Code: 0x00` (correcto). De lo contrario, IoT Hub envía el paquete `DISCONNECT` con `Reason Code: 0x87` (no autorizado) y cierra la conexión.

### <a name="disconnection"></a>Desconexión

El servidor puede desconectar el cliente por varios motivos:

- El comportamiento indebido del cliente impide responder y se genera un reconocimiento (o una respuesta) negativo directamente.
- El servidor no puede mantener actualizado el estado de la conexión.
- Se ha conectado un cliente con la misma identidad.

El servidor puede desconectarse con cualquier código de motivo definido en la especificación MQTT 5.0. Menciones destacadas:

- `135` (no autorizado) cuando se produce un error en la reautenticación, el token de SAS actual expira o las credenciales del dispositivo cambian.
- `142` (sesión relegada) cuando se ha abierto una nueva conexión con la misma identidad de cliente.
- `159` (velocidad de conexión superada) cuando se supera la velocidad de conexión de IoT Hub.  
- `131` (error específico de la implementación) se utiliza para cualquier error personalizado definido en esta API. Las propiedades `status` y `reason` se usarán para comunicar más detalles sobre la causa de la desconexión (consulte [Respuesta](#response) para obtener más detalles).

## <a name="operations"></a>Operations

Todas las funcionalidades de esta API se expresan como operaciones. Este es un ejemplo de la operación Enviar telemetría:

```yaml
-> PUBLISH
    QoS: 1
    Packet_Id: 3
    Topic: $iothub/telemetry
    Payload: Hello

<- PUBACK
    Packet_Id: 3
    Reason_Code: 0
```

Para obtener una especificación completa de las operaciones de esta API, consulte [Referencia de API](iot-hub-mqtt-5-reference.md).

> [!NOTE]
> Todos los ejemplos de esta especificación se muestran desde la perspectiva del cliente. El signo `->` indica que el cliente envía paquetes, y `<-` que los recibe.

### <a name="message-topics-and-subscriptions"></a>Temas de los mensajes y suscripciones

Los temas usados en los mensajes de las operaciones de esta API comienzan por `$iothub/`.
La semántica del agente de MQTT no se aplica a estas operaciones. Consulte "[Topics beginning with \$](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901246)" (Temas que comienzan por $) para obtener más detalles.
No se admiten los temas que comienzan por `$iothub/` que no están definidos en esta API:

- El envío de mensajes a un tema sin definir da como resultado la respuesta `Not Found` (consulte [Respuesta](#response) para obtener más información a continuación).
- La suscripción a un tema sin definir da como resultado `SUBACK` con `Reason Code: 0x8F` (filtro de tema no válido).

Los nombres de tema y los nombres de propiedad distinguen mayúsculas de minúsculas y deben coincidir exactamente. Por ejemplo, `$iothub/telemetry/` no se admite, pero `$iothub/telemetry` sí.

> [!NOTE]
> No se admiten los caracteres comodín en las suscripciones de `$iothub/..`. Es decir, un cliente no puede suscribirse a `$iothub/+` ni a `$iothub/#`. Si intenta hacerlo, obtendrá `SUBACK` con `Reason Code: 0xA2` (no se admiten suscripciones de comodín). Solo se admiten caracteres comodín de un solo segmento (`+`), en lugar de parámetros de ruta de acceso en el nombre de tema para las operaciones que los incluyen.

### <a name="interaction-types"></a>Tipos de interacción

Todas las operaciones de esta API se basan en uno de los dos tipos de interacción:

- Mensaje con confirmación opcional (MessageAck)
- Solicitud-respuesta (ReqRep)

Las operaciones también varían según la dirección (determinada por la dirección del mensaje inicial que se intercambia):

- Cliente a servidor (c2s)
- Servidor a cliente (s2c)

Por ejemplo, el envío de telemetría es una operación de cliente a servidor de tipo "mensaje con reconocimiento", mientras que el método de control directo es una operación de servidor a cliente de tipo solicitud-respuesta.

#### <a name="message-acknowledgement-interactions"></a>Interacciones de reconocimiento de mensajes

El mensaje con la interacción de reconocimiento opcional (MessageAck) se expresa como un intercambio de paquetes `PUBLISH` y `PUBACK` en MQTT. El reconocimiento es opcional y el remitente puede optar por no solicitarlo mediante el envío del paquete `PUBLISH` con `QoS: 0`.

> [!NOTE]
> Si las propiedades del paquete `PUBACK` se deben truncar debido al valor de `Maximum Packet Size` declarado por el cliente, IoT Hub conservará tantas propiedades de usuario como admita el límite especificado. Las propiedades de usuario que se enumeran en primer lugar tienen más posibilidades de enviarse que las que se muestran más adelante. La propiedad `Reason String` tiene la menor prioridad.

##### <a name="example-of-simple-messageack-interaction"></a>Ejemplo de interacción MessageAck simple

Mensaje:

```yaml
PUBLISH
    QoS: 1
    Packet_Id: 34
    Topic: $iothub/{request.path}
    Payload: <any>
```

Reconocimiento (correcto):

```yaml
PUBACK
    Packet_Id: 34
    Reason_Code: 0
```

#### <a name="request-response-interactions"></a>Interacciones de respuesta a solicitud

En las interacciones de respuesta a solicitud (ReqRep), las solicitudes y las respuestas se traducen en paquetes `PUBLISH` con `QoS: 0`.

La propiedad `Correlation Data` se debe establecer en ambas y se usa para hacer coincidir el paquete de respuesta con el de solicitud.

Esta API usa el tema de respuesta única `$iothub/responses` para todas las operaciones ReqRep. No es necesario suscribirse a este tema ni cancelar la suscripción para las operaciones de cliente a servidor: el servidor supone que todos los clientes están suscritos.

##### <a name="example-of-simple-reqrep-interaction"></a>Ejemplo de interacción ReqRep simple

Solicitud:

```yaml
PUBLISH
    QoS: 0
    Topic: $iothub/{request.path}
    Correlation_Data: 0x01 0xFA
    Payload: ...
```

Respuesta (correcto):

```yaml
PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x01 0xFA
    Payload: ...
```

Las interacciones ReqRep no admiten paquetes `PUBLISH` con `QoS: 1` como mensajes de solicitud o de respuesta. El envío de la solicitud `PUBLISH` da como resultado la respuesta `Bad Request`.

La longitud máxima admitida de la propiedad `Correlation Data` es de 16 bytes. Si la propiedad `Correlation Data` en el paquete `PUBLISH` se establece en un valor de más de 16 bytes, IoT Hub envía `DISCONNECT` con el resultado `Bad Request` y cierra la conexión. Este comportamiento solo se aplica a los paquetes intercambiados dentro de esta API.

> [!NOTE]
> Los datos de correlación son una secuencia de bytes arbitraria, es decir, no se garantiza que sea una cadena UTF-8.
>
> ReqRep usa temas predefinidos para la respuesta; Se omite la propiedad del tema de respuesta en el paquete `PUBLISH` de la solicitud (si la estableció el remitente).

IoT Hub suscribe automáticamente el cliente a los temas de respuesta para todas las operaciones ReqRep de cliente a servidor. Aunque el cliente cancele la suscripción explícitamente del tema de respuesta, IoT Hub restablece automáticamente la suscripción. En el caso de las interacciones ReqRep de servidor a cliente, sigue siendo necesario que el dispositivo se suscriba.

### <a name="message-properties"></a>Message (propiedades)

Las propiedades de la operación, definidas por el sistema o el usuario, se expresan como propiedades de paquete en MQTT 5.

Los nombres de propiedad de usuario distinguen mayúsculas de minúsculas y se deben escribir exactamente según su definición. Por ejemplo, `Trace-ID` no es compatible, pero sí lo es `trace-id`.

Las solicitudes con propiedades de usuario fuera de la especificación y sin el prefijo `@` producen un error.

Las propiedades del sistema se codifican como propiedades de primera clase (por ejemplo, `Content Type`) o como propiedades de usuario. La especificación proporciona una lista exhaustiva de las propiedades del sistema admitidas.
Todas las propiedades de primera clase se omiten a menos que la compatibilidad con estas se indique explícitamente en la especificación.

Si se permiten las propiedades definidas por el usuario, sus nombres deben seguir el formato `@{property name}`. Las propiedades definidas por el usuario solo admiten valores de cadena UTF-8 válidos. Por ejemplo, la propiedad `MyProperty1` con el valor `15` se debe codificar como una propiedad de usuario con el nombre `@MyProperty` y el valor `15`.

Si IoT Hub no reconoce la propiedad del usuario, se considera un error y IoT Hub responde con `PUBACK`, con `Reason Code: 0x83` (error específico de la implementación) y con `status: 0100` (solicitud incorrecta). Si no se solicitó la confirmación (QoS: 0), se devolverá el paquete `DISCONNECT` con el mismo error y se terminará la conexión.

Esta API define los siguientes tipos de datos además de `string`:

- `time`: número de milisegundos desde `1970-01-01T00:00:00.000Z`. Por ejemplo, `1600987195320` para `2020-09-24T22:39:55.320Z`.
- `u32`: valor entero de 32 bits sin signo.
- `u64`: valor entero de 64 bits sin signo.
- `i32`: valor entero de 32 bits con signo.

### <a name="response"></a>Response

Las interacciones pueden dar lugar a resultados diferentes: `Success`, `Bad Request`, `Not Found`, etc.
Los resultados se diferencian entre sí por la propiedad de usuario `status`. `Reason Code` en los paquetes `PUBACK` (para interacciones MessageAck) coincide con `status` en el significado siempre que es posible.

> [!NOTE]
> Si el cliente especifica `Request Problem Information: 0` en el paquete CONNECT, no se enviarán propiedades de usuario en los paquetes `PUBACK` para cumplir con la especificación MQTT 5, incluida la propiedad `status`. En este caso, el cliente todavía se puede basar en `Reason Code` para determinar si el reconocimiento es positivo o negativo. 

Cada interacción tiene un valor predeterminado (o correcto). Tiene `Reason Code` establecido en `0` y la propiedad `status` "not set". De lo contrario:

- En el caso de las interacciones MessageAck, `PUBACK` obtiene un valor de `Reason Code` distinto de 0x0 (correcto). la propiedad `status` puede estar presente para aclarar mejor el resultado.
- En las interacciones de ReqRep, la respuesta `PUBLISH` obtiene el conjunto de propiedades `status`.
- Dado que no hay ninguna manera de responder a las interacciones MessageAck con `QoS: 0` directamente, el paquete `DISCONNECT` se envía en su lugar con la información de la respuesta y luego se cierra la conexión.

Ejemplos:

Solicitud incorrecta (MessageAck):

```yaml
PUBACK
    Reason_Code: 131
    status: 0100
    reason: Unknown property `test`
```

No autorizado (MessageAck):

```yaml
PUBACK
    Reason_Code: 135
    status: 0101
```

No autorizado (ReqRep):

```yaml
PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: ...
    status: 0101
```

Cuando es necesario, IoT Hub establece las siguientes propiedades de usuario:

- `status`: código extendido de IoT Hub para el estado de la operación. Este código se puede utilizar para diferenciar los resultados.
- `trace-id`: identificador de seguimiento de la operación; IoT Hub puede mantener diagnósticos adicionales relativos a la operación que se podría usar para la investigación interna.
- `reason`: mensaje legible que proporciona más información sobre por qué la operación finalizó con el estado que indicaba la propiedad `status`.

> [!NOTE]
> Si el cliente establece la propiedad `Maximum Packet Size` del paquete CONNECT en un valor muy pequeño, es posible que no todas las propiedades del usuario encajen y podrían no aparecer en el paquete.
> 
> `reason` está pensado únicamente para personas y no debe usarse en la lógica de cliente. Esta API permite cambiar los mensajes en cualquier momento sin previo aviso y sin cambio de versión.
>
> Si el cliente envía `RequestProblemInformation: 0` del paquete CONNECT, las propiedades de usuario no se incluirán en los reconocimientos de acuerdo con la [especificación MQTT 5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901053).

#### <a name="status-code"></a>status code

La propiedad `status` contiene el código de estado de la operación. Está optimizado para la eficacia de la lectura automatizada.
Consta de un entero sin signo de dos bytes codificado como hexadecimal en una cadena como `0501`.
Estructura del código (mapa de bits):

```text
7 6 5 4 3 2 1 0 | 7 6 5 4 3 2 1 0
0 0 0 0 0 R T T | C C C C C C C C
```

El primer byte se usa para las marcas:

- Los bits 0 y 1 indican el tipo de resultados:
  - `00`: correcto
  - `01`: error de cliente
  - `10`: error de servidor
- El bit 2: `1` indica que el error se puede reintentar.
- Los bits 3 a 7 están reservados y deben establecerse en `0`.

El segundo byte contiene el código de respuesta distintivo real. Los códigos de error con marcas diferentes pueden tener el mismo valor del segundo byte. Por ejemplo, pueden existir los códigos de error `0001`, `0101`, `0201` y `0301` con un significado distinto.

Por ejemplo, `Too Many Requests` es un cliente, un error que se puede reintentar con código propio de `1`. Su valor es `0000 0101 0000 0001` o `0x0501`.

Los clientes pueden utilizar bits de tipo para identificar si la operación finalizó correctamente. Los clientes también pueden usar el bit que se puede reintentar para decidir si es prudente reintentar la operación.

## <a name="recommendations"></a>Recomendaciones

### <a name="session-management"></a>Administración de sesiones

El paquete `CONNACK` contiene la propiedad `Session Present` para indicar si el servidor restauró la sesión creada anteriormente. Use esta propiedad para averiguar si se va a suscribir a los temas o a omitir la suscripción, ya que la suscripción se realizó anteriormente.

Para basarse en `Session Present`, el cliente debe realizar un seguimiento de las suscripciones que ha realizado (es decir, envido el paquete `SUBSCRIBE` y recibido `SUBACK` con el código de motivo correcto), o bien asegurarse de suscribirse a todos los temas de un solo intercambio de `SUBSCRIBE`/`SUBACK`. De lo contrario, si el cliente envía dos paquetes `SUBSCRIBE` y el servidor solo procesa uno de estos correctamente, el servidor comunicará `Session Present: 1` en `CONNACK` y solo se aceptará una parte de las suscripciones del cliente.

Para evitar el caso en que una versión anterior del cliente no se suscribió a todos los temas, es mejor suscribirse de forma incondicional cuando el comportamiento del cliente cambia (por ejemplo, como parte de la actualización del firmware). Además, para asegurarse de que no quedan suscripciones obsoletas (tomando el número máximo permitido de suscripciones), cancele de forma explícita las suscripciones que ya no estén en uso.

### <a name="batching"></a>Lotes

No hay ningún formato especial para enviar un lote de mensajes. Para reducir la sobrecarga de las operaciones que consumen muchos recursos en TLS y redes, agrupe paquetes (`PUBLISH`, `PUBACK`, `SUBSCRIBE`, etc.) antes de entregarlos a la pila de TLS/TCP subyacente. Además, el cliente puede facilitar el alias de tema en el "lote":

- Coloque el nombre completo del tema en el primer paquete `PUBLISH` para la conexión y asóciele el alias del tema.
- Coloque los siguientes paquetes para el mismo tema con la propiedad de nombre de tema y alias de tema vacía.

## <a name="migration"></a>Migración

En esta sección se enumeran los cambios en la API en comparación con la [API de MQTT anterior](iot-hub-mqtt-support.md).

- El protocolo de transporte es MQTT 5. El anterior era MQTT 3.1.1.
- La información de contexto para la autenticación de SAS se incluye en el paquete `CONNECT` directamente en lugar de codificarse junto con la firma.
- La opción Método de autenticación se usa para indicar el método de autenticación utilizado.
- La firma de acceso compartido se coloca en la propiedad Authentication Data. Anteriormente se usaba el campo Contraseña.
- Los temas de las operaciones son diferentes:
  - Telemetría: `$iothub/telemetry` en lugar de `devices/{Client Id}/messages/events`.
  - Comandos: `$iothub/commands` en lugar de `devices/{Client Id}/messages/devicebound`.
  - Operación Patch Twin notificada: `$iothub/twin/patch/reported` en lugar de `$iothub/twin/PATCH/properties/reported`.
  - Notificación de cambio de estado deseado de gemelo: `$iothub/twin/patch/desired` en lugar de `$iothub/twin/PATCH/properties/desired`.
- No se requiere una suscripción para el tema de respuesta de las operaciones de respuesta a solicitud de cliente-servidor.
- Las propiedades de usuario se usan en lugar de las propiedades de codificación en el segmento de nombre de tema.
- Los nombres de propiedad se escriben según la convención de nomenclatura con guiones en lugar de las abreviaturas con prefijo especial. Las propiedades definidas por el usuario ahora requieren el prefijo en su lugar. Por ejemplo, `$.mid` ahora es `message-id`, mientras `myProperty1` se convierte en `@myProperty1`.
- La propiedad Correlation Data se usa para correlacionar los mensajes de solicitud y respuesta para las operaciones de respuesta a solicitud en lugar de la propiedad `$rid` codificada en el tema.
- La propiedad `iothub-connection-auth-method` ya no se marca en los eventos de telemetría.
- Los comandos C2D no se purgarán en ausencia de suscripción del dispositivo. Permanecerán en cola hasta que el dispositivo se suscriba o expiren.

## <a name="examples"></a>Ejemplos

### <a name="send-telemetry"></a>Envío de datos de telemetría

Mensaje:

```yaml
-> PUBLISH
    QoS: 1
    Packet_Id: 31
    Topic: $iothub/telemetry
    @myProperty1: My String Value # optional
    creation-time: 1600987195320 # optional
    @ No_Rules-ForUser-PROPERTIES: Any UTF-8 string value # optional
    Payload: <data>
```

Confirmación:

```yaml
<- PUBACK
    Packet_Id: 31
    Reason_Code: 0
```

Reconocimiento alternativo (limitado):

```yaml
<- PUBACK
    Packet_Id: 31
    Reason_Code: 151
    status: 0501
```

---

### <a name="send-get-twins-state"></a>Envío de estado de Get Twin

Solicitud:

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/get
    Correlation_Data: 0x01 0xFA
    Payload: <empty>
```

Respuesta (correcta):

```yaml
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x01 0xFA
    Payload: <twin/desired state>
```

Respuesta (no permitida):

```yaml
<- PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x01 0xFA
    status: 0102
    reason: Operation not allowed for `B2` SKU
    Payload: <empty>
```

---

### <a name="handle-direct-method-call"></a>Control de llamada de método directo

Solicitud:

```yaml
<- PUBLISH
    QoS: 0
    Topic: $iothub/methods/abc
    Correlation_Data: 0x0A 0x10
    Payload: <data>
```

Respuesta (correcta):

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x0A 0x10
    response-code: 200 # user defined response code
    Payload: <data>
```

> [!NOTE]
> `status` no establecido: es una respuesta correcta.

Respuesta de dispositivo no disponible:

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/responses
    Correlation_Data: 0x0A 0x10
    status: 0603
```

---

### <a name="error-while-using-qos-0-part-1"></a>Error al usar QoS 0, parte 1

Solicitud:

```yaml
-> PUBLISH
    QoS: 0
    Topic: $iothub/twin/gett # misspelled topic name - server won't recognize it as Request-Response interaction
    Correlation_Data: 0x0A 0x10
    Payload: <data>
```

Respuesta:

```yaml
<- DISCONNECT
    Reason_Code: 144
    reason: "Unsupported topic: `$iothub/twin/gett`"
```

---

### <a name="error-while-using-qos-0-part-2"></a>Error al usar QoS 0, parte 2

Solicitud:

```yaml
-> PUBLISH # missing Correlation Data
    QoS: 0
    Topic: $iothub/twin/get
    Payload: <data>
```

Respuesta:

```yaml
<- DISCONNECT
    Reason_Code: 131
    status: 0100
    reason: "`Correlation Data` property is missing"
```
## <a name="next-steps"></a>Pasos siguientes

- Para revisar la referencia de la API de la versión preliminar de MQTT 5, consulte [Referencia de la API de MQTT 5 del plano de datos de IoT Hub](iot-hub-mqtt-5-reference.md).
- Para seguir un ejemplo de C#, consulte el [repositorio de ejemplos de GitHub](https://github.com/Azure-Samples/iot-hub-mqtt-5-preview-samples-csharp).