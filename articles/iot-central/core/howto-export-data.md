---
title: Exportación de datos desde Azure IoT Central | Microsoft Docs
description: Cómo usar la nueva exportación de datos para exportar los datos de IoT a Azure y a destinos en la nube personalizados.
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 03/24/2021
ms.topic: how-to
ms.service: iot-central
ms.custom: contperf-fy21q1, contperf-fy21q3
ms.openlocfilehash: 7d57f24f8cb4b59ce9b9cd5853be11fb2d104d75
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106277902"
---
# <a name="export-iot-data-to-cloud-destinations-using-data-export"></a>Exportación de datos de IoT a destinos en la nube mediante la característica de exportación de datos

> [!Note]
> En este artículo se describen las características de exportación de datos de IoT Central. Para información sobre las características de exportación de datos heredadas, consulte [Exportación de datos de IoT a destinos en la nube mediante la característica de exportación de datos (versión heredada)](./howto-export-data-legacy.md).

En este artículo se describe cómo usar la nueva característica de exportación de datos de Azure IoT Central. Utilice esta característica para exportar continuamente los datos de IoT filtrados y enriquecidos desde su aplicación de IoT Central. La exportación de datos inserta los cambios casi en tiempo real en otras partes de la solución en la nube para su análisis y almacenamiento, y conocer los detalles de la ruta de acceso activa.

Por ejemplo, puede:

- Exporte continuamente la telemetría, los cambios de propiedad, el ciclo de vida del dispositivo y los datos del ciclo de vida de la plantilla de dispositivo en formato JSON casi en tiempo real.
- Filtrar los flujos de datos para exportar los datos que coincidan con condiciones personalizadas.
- Enriquecer los flujos de datos con valores personalizados y valores de propiedad del dispositivo.
- Enviar los datos a destinos, como Azure Event Hubs, Azure Service Bus, Azure Blob Storage y puntos de conexión de webhook.

> [!Tip]
> Cuando se activa la exportación de datos, solo se obtienen los datos a partir de ese momento. Actualmente, no se pueden recuperar los datos del tiempo durante el que la exportación de datos estaba desactivada. Para conservar más datos históricos, active la exportación de datos al principio.

## <a name="prerequisites"></a>Prerrequisitos

Para usar las características de exportación de datos, debe tener una [aplicación V3](howto-get-app-info.md) y permisos de [exportación de datos](howto-manage-users-roles.md).

Si tiene una aplicación V2, consulte [Migración de la aplicación de IoT Central V2 a V3](howto-migrate.md).

## <a name="set-up-export-destination"></a>Configuración del destino de exportación

El destino de exportación debe existir antes de configurar la exportación de datos. Actualmente están disponibles los siguientes tipos de destino:

- Azure Event Hubs
- Cola de Azure Service Bus
- Tema de Azure Service Bus
- Azure Blob Storage
- webhook

### <a name="create-an-event-hubs-destination"></a>Creación de un destino de Event Hubs

Si no tiene un espacio de nombres existente de Event Hubs al que exportar, siga estos pasos:

1. Cree un [espacio de nombres de Event Hubs en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.EventHub). Puede encontrar más información en los [documentos de Azure Event Hubs](../../event-hubs/event-hubs-create.md).

1. Cree un centro de eventos en el espacio de nombres de Event Hubs Vaya al espacio de nombres y seleccione **+ Centro de eventos** en la parte superior para crear una instancia de centro de eventos.

1. Genere la clave que usará cuando configure la exportación de datos en IoT Central:

    - Seleccione la instancia del centro de eventos que creó.
    - Seleccione **Configuración > Directivas de acceso compartido**.
    - Cree una nueva clave o elija una clave existente que tenga permisos de **envío**.
    - Copie la cadena de conexión principal o la secundaria. Esta cadena de conexión sirve para configurar un nuevo destino en IoT Central.
    - Como alternativa, puede generar una cadena de conexión para todo el espacio de nombres de Event Hubs:
        1. Vaya a su espacio de nombres de Event Hubs en Azure Portal.
        2. En **Configuración**, seleccione **Directivas de acceso compartido**.
        3. Cree una nueva clave o elija una clave existente que tenga permisos de **envío**.
        4. Copie la cadena de conexión principal o la secundaria.
        
### <a name="create-a-service-bus-queue-or-topic-destination"></a>Creación de una cola o un tema de Service Bus como destino

Si no tiene un espacio de nombres existente de Service Bus al que exportar, siga estos pasos:

1. Cree un [nuevo espacio de nombres de Service Bus en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.ServiceBus.1.0.5). Puede encontrar más información en los [documentos de Azure Service Bus](../../service-bus-messaging/service-bus-create-namespace-portal.md).

1. Para crear una cola o tema como destino de exportación, vaya al espacio de nombres de Service Bus y seleccione **+ Cola** o **+ Tema**.

1. Genere la clave que usará cuando configure la exportación de datos en IoT Central:

    - Seleccione la cola o el tema que creó.
    - Seleccione **Configuración/Directivas de acceso compartido**.
    - Cree una nueva clave o elija una clave existente que tenga permisos de **envío**.
    - Copie la cadena de conexión principal o la secundaria. Esta cadena de conexión sirve para configurar un nuevo destino en IoT Central.
    - Como alternativa, puede generar una cadena de conexión para todo el espacio de nombres de Service Bus:
        1. Vaya al espacio de nombres de Service Bus en Azure Portal.
        2. En **Configuración**, seleccione **Directivas de acceso compartido**.
        3. Cree una nueva clave o elija una clave existente que tenga permisos de **envío**.
        4. Copie la cadena de conexión principal o la secundaria.

### <a name="create-an-azure-blob-storage-destination"></a>Creación de una instancia de Azure Blob Storage como destino

Si no tiene una cuenta existente de Azure Storage a la que exportar, siga estos pasos:

1. Cree una [cuenta de almacenamiento en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM). Puede obtener más información sobre cómo crear nuevas [cuentas de almacenamiento de Azure Blob ](../../storage/blobs/storage-quickstart-blobs-portal.md) o [cuentas de almacenamiento de Azure Data Lake Storage v2](../../storage/common/storage-account-create.md). La exportación de datos solo puede escribir datos en cuentas de almacenamiento que admiten blobs en bloques. En la lista siguiente se muestran los tipos de cuenta de almacenamiento compatibles conocidos:

    |Nivel de rendimiento|Tipo de cuenta|
    |-|-|
    |Estándar|Uso general V2|
    |Estándar|Uso general V1|
    |Estándar|Blob Storage|
    |Premium|Almacenamiento de blob en bloques|

1. Para crear un contenedor en su cuenta de almacenamiento, vaya a su cuenta de almacenamiento. En **Blob Service** seleccione **Examinar blobs**. Seleccione **+ Contenedor** en la parte superior para crear un contenedor.

1. Genere una cadena de conexión para la cuenta de almacenamiento en **Configuración > Claves de acceso**. Copie una de las dos cadenas de conexión.

### <a name="create-a-webhook-endpoint"></a>Creación de un punto de conexión de webhook

Puede exportar los datos a un punto de conexión de webhook HTTP disponible públicamente. Puede crear un punto de conexión de webhook de prueba mediante [RequestBin](https://requestbin.net/). RequestBin limita las solicitudes cuando se alcanza el límite:

1. Abra [RequestBin](https://requestbin.net/).
2. Cree una instancia de RequestBin y copie la **dirección URL de la papelera**. Esta dirección URL se usa para probar la exportación de datos.

## <a name="set-up-data-export"></a>Configuración de la exportación de datos

Ahora que tiene un destino al que exportar los datos, configure la exportación de datos en su aplicación de IoT Central:

1. Inicie sesión en su aplicación de IoT Central.

1. En el panel izquierdo, seleccione **Exportación de datos**.

    > [!Tip]
    > Si no ve **Exportación de datos** en el panel izquierdo, no tiene permisos para configurar la exportación de datos en la aplicación. Hable con un administrador para configurar la exportación de datos.

1. Seleccione **+ New export** (+ Nueva exportación).

1. Escriba un nombre para mostrar para la nueva exportación y asegúrese de que la exportación de datos está **habilitada**.

1. Elija el tipo de datos para exportar. En la tabla siguiente se enumeran los tipos de exportación de datos compatibles:

    | Tipo de datos | Descripción | Formato de datos |
    | :------------- | :---------- | :----------- |
    |  Telemetría | Exporte los mensajes de telemetría de los dispositivos casi en tiempo real. Cada mensaje exportado contiene todo el contenido del mensaje del dispositivo original, normalizado.   |  [Formato del mensaje de telemetría](#telemetry-format)   |
    | Cambios de la propiedad | Exporte los cambios de las propiedades de dispositivos y la nube casi en tiempo real. En el caso de las propiedades de dispositivo de solo lectura, se exportan los cambios realizados en los valores notificados. En el caso de las propiedades de lectura y escritura, se exportan los valores notificados y deseados. | [Formato de los mensajes de cambio de propiedad](#property-changes-format) |
    | Ciclo de vida del dispositivo | Exporte los eventos registrados y eliminados del dispositivo. | [Formato de los mensajes de cambios del ciclo de vida de dispositivo](#device-lifecycle-changes-format) |
    | Ciclo de vida de plantillas de dispositivo | Exporte los cambios de plantilla de dispositivo publicados, incluidos los creados, actualizados y eliminados. | [Formato de los mensajes de cambios del ciclo de vida de plantilla de dispositivo](#device-template-lifecycle-changes-format) | 

1. Opcionalmente, agregue filtros para reducir la cantidad de datos exportados. Hay diferentes tipos de filtros disponibles para cada tipo de exportación de datos: <a name="DataExportFilters"></a>
    
    | Tipo de datos | Filtros disponibles| 
    |--------------|------------------|
    |Telemetría|<ul><li>Filtrar por nombre de dispositivo, id. de dispositivo y plantilla de dispositivo</li><li>Filtrar el flujo para que solo contenga la telemetría que cumpla las condiciones de filtro</li><li>Filtrar el flujo para que solo contenga la telemetría de dispositivos con propiedades que coincidan con las condiciones de filtro</li><li>Filtrar el flujo para que solo contenga la telemetría que tenga *propiedades de mensaje* que cumplan la condición de filtro. Las *propiedades de mensaje* (también denominadas *propiedades de aplicación*) se envían en un contenedor de pares de clave-valor en cada mensaje de telemetría enviado opcionalmente por los dispositivos que usan los SDK de dispositivo. Para crear un filtro de propiedad de mensaje, escriba la clave de la propiedad de mensaje que busca y especifique una condición. Solo se exportarán los mensajes de telemetría con propiedades que coincidan con la condición de filtro especificada. [Obtenga más información sobre las propiedades de aplicaciones en la documentación de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md). </li></ul>|
    |Cambios de la propiedad|<ul><li>Filtrar por nombre de dispositivo, id. de dispositivo y plantilla de dispositivo</li><li>Filtrar el flujo para que contenga solo los cambios de propiedad que cumplan las condiciones de filtro</li></ul>|
    |Ciclo de vida del dispositivo|<ul><li>Filtrar por nombre de dispositivo, id. de dispositivo y plantilla de dispositivo</li><li>Filtrar el flujo para que solo contenga los cambios de dispositivos con propiedades que coincidan con las condiciones de filtro</li></ul>|
    |Ciclo de vida de plantillas de dispositivo|<ul><li>Filtrar por plantilla de dispositivo</li></ul>|
    
1. Opcionalmente, puede enriquecer los mensajes exportados con metadatos adicionales de pares clave-valor. Están disponibles los siguientes enriquecimientos para los tipos de exportación de datos de telemetría y cambios de propiedad: <a name="DataExportEnrichmnents"></a>
    - **Cadena personalizada**: agrega una cadena estática personalizada a cada mensaje. Escriba cualquier clave y especifique cualquier valor de cadena.
    - **Propiedad**: agrega la propiedad notificada de dispositivo actual o el valor de propiedad de la nube a cada mensaje. Escriba cualquier clave y elija una propiedad de dispositivo o nube. Si el mensaje exportado procede de un dispositivo que no tiene la propiedad especificada, no obtendrá el enriquecimiento.

1. Agregue un destino nuevo o uno que ya haya creado. Seleccione el vínculo **Create a new one** (Crear uno nuevo) y agregue la siguiente información:

    - **Destination name** (Nombre del destino): el nombre para mostrar del destino en IoT Central.
    - **Tipo de destino**: elija el tipo de destino. Si todavía no ha configurado el destino de exportación, consulte [Configuración del destino de exportación](#set-up-export-destination).
    - En el caso de Azure Event Hubs o la cola o el tema de Azure Service Bus, pegue la cadena de conexión del recurso y escriba el nombre (distingue mayúsculas de minúsculas) del centro de eventos, la cola o el tema, si es necesario.
    - En Azure Blob Storage, pegue la cadena de conexión del recurso y, si es necesario, escriba el nombre del contenedor (distingue mayúsculas de minúsculas).
    - Si se trata de un webhook, pegue la dirección URL de devolución de llamada para el punto de conexión de webhook. Opcionalmente, puede configurar la autorización de webhook (OAuth 2.0 y el token de autorización) y agregar encabezados personalizados. 
        - En OAuth 2.0, solo se admite el flujo de credenciales de cliente. Cuando el destino se guarde, IoT Central se comunicará con el proveedor de OAuth para recuperar un token de autorización. Este token se adjuntará al encabezado "Authorization" para cada mensaje enviado a este destino.
        - Para el token de autorización, puede especificar un valor de token que se adjunte directamente al encabezado "Authorization" para cada mensaje enviado a este destino.
    - Seleccione **Crear**.

1. Seleccione **+ Destino** y elija un destino en el menú desplegable. Puede agregar hasta cinco destinos a una única exportación.

1. Cuando haya terminado de configurar la exportación, seleccione **Guardar**. Transcurridos unos minutos, los datos aparecen en sus destinos.

## <a name="monitor-your-export"></a>Supervisión de la exportación

Además de ver el estado de las exportaciones en IoT Central, puede usar [Azure Monitor](../../azure-monitor/overview.md) para ver la cantidad de datos que está exportando y los errores de exportación. Puede acceder a las métricas de estado de dispositivos y de exportación en los gráficos de Azure Portal, con una API de REST o con consultas en PowerShell o la CLI de Azure. Actualmente, puede supervisar las siguientes métricas de exportación de datos en Azure Monitor:

- Número de mensajes entrantes para exportar antes de que se apliquen los filtros
- Número de mensajes que pasan a través de filtros
- Número de mensajes exportados correctamente a destinos
- Número máximo de errores detectados

Para más información, vea [Supervisión del estado general de una aplicación de IoT Central](howto-monitor-application-health.md).

## <a name="destinations"></a>Destinations

### <a name="azure-blob-storage-destination"></a>Destino de Azure Blob Storage

Los datos se exportan una vez por minuto, y cada archivo contiene el lote de cambios desde la exportación anterior. Los datos exportados se guardan en formato JSON. Las rutas de acceso predeterminadas a los datos exportados en la cuenta de almacenamiento son:

- Telemetría: _{container}/{app-id}/{partition_id}/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}_
- Cambios de propiedad: _{container}/{app-id}/{partition_id}/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}_

Para examinar los archivos exportados en Azure Portal, vaya al archivo y seleccione **Editar blob**.

### <a name="azure-event-hubs-and-azure-service-bus-destinations"></a>Destinos de Azure Event Hubs y Azure Service Bus

Los datos se exportan casi en tiempo real. Los datos están en el cuerpo del mensaje y están en formato JSON codificados como UTF-8.

Las anotaciones o el contenedor de propiedades del sistema del mensaje contienen los campos `iotcentral-device-id`, `iotcentral-application-id`, `iotcentral-message-source` y `iotcentral-message-type`, que tienen los mismos valores que los campos correspondientes del cuerpo del mensaje.

### <a name="webhook-destination"></a>Destino de webhook

En el caso de los destinos de webhooks, los datos también se exportan casi en tiempo real. Los datos del cuerpo del mensaje están en el mismo formato que en Event Hubs y Service Bus.

## <a name="telemetry-format"></a>Formato de los datos de telemetría

Cada mensaje exportado contiene un formato normalizado del mensaje completo que el dispositivo envió en el cuerpo del mensaje. El mensaje está en formato JSON y codificado como UTF-8. La información de los mensajes incluye:

- `applicationId`: el identificador de la aplicación IoT Central.
- `messageSource`: el origen del mensaje (`telemetry`).
- `deviceId`:  el identificador del dispositivo que envió el mensaje de telemetría.
- `schema`: el nombre y la versión del esquema de carga.
- `templateId`: el identificador de la plantilla de dispositivo asociada al dispositivo.
- `enqueuedTime`: la hora a la que IoT Central recibió este mensaje.
- `enrichments`: cualquier enriquecimiento configurado en la exportación.
- `messageProperties`: propiedades adicionales que el dispositivo envió con el mensaje. A veces, estas propiedades se denominan *propiedades de la aplicación*. [Puede encontrar más información en la documentación de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En el caso de Event Hubs y Service Bus, IoT Central exporta un mensaje nuevo rápidamente después de recibir el mensaje de un dispositivo. En las propiedades de usuario (también conocidas como propiedades de la aplicación) de cada mensaje, `iotcentral-device-id`, `iotcentral-application-id` e `iotcentral-message-source` se incluyen automáticamente.

Para Blob Storage, los mensajes se procesan por lotes y se exportan una vez por minuto.

En el ejemplo siguiente se muestra un mensaje de telemetría exportado:

```json

{
    "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
    "messageSource": "telemetry",
    "deviceId": "1vzb5ghlsg1",
    "schema": "default@v1",
    "templateId": "urn:qugj6vbw5:___qbj_27r",
    "enqueuedTime": "2020-08-05T22:26:55.455Z",
    "telemetry": {
        "Activity": "running",
        "BloodPressure": {
            "Diastolic": 7,
            "Systolic": 71
        },
        "BodyTemperature": 98.73447010562934,
        "HeartRate": 88,
        "HeartRateVariability": 17,
        "RespiratoryRate": 13
    },
    "enrichments": {
      "userSpecifiedKey": "sampleValue"
    },
    "messageProperties": {
      "messageProp": "value"
    }
}
```
### <a name="message-properties"></a>Propiedades del mensaje

Los mensajes de telemetría tienen propiedades de metadatos además de la carga de telemetría. El fragmento de código anterior muestra ejemplos de mensajes del sistema, como `deviceId` y `enqueuedTime`. Para obtener más información sobre las propiedades del mensaje del sistema, consulte [Propiedades del sistema de los mensajes de IoT Hub D2C](../../iot-hub/iot-hub-devguide-messages-construct.md#system-properties-of-d2c-iot-hub-messages).

Puede agregar propiedades a los mensajes de telemetría si necesita agregar metadatos personalizados a los mensajes de telemetría. Por ejemplo, tiene que agregar una marca de tiempo cuando el dispositivo crea el mensaje.

En el fragmento de código siguiente se muestra cómo agregar la propiedad `iothub-creation-time-utc` al mensaje cuando se crea en el dispositivo:

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
async function sendTelemetry(deviceClient, index) {
  console.log('Sending telemetry message %d...', index);
  const msg = new Message(
    JSON.stringify(
      deviceTemperatureSensor.updateSensor().getCurrentTemperatureObject()
    )
  );
  msg.properties.add("iothub-creation-time-utc", new Date().toISOString());
  msg.contentType = 'application/json';
  msg.contentEncoding = 'utf-8';
  await deviceClient.sendEvent(msg);
}
```

# <a name="java"></a>[Java](#tab/java)

```java
private static void sendTemperatureTelemetry() {
  String telemetryName = "temperature";
  String telemetryPayload = String.format("{\"%s\": %f}", telemetryName, temperature);

  Message message = new Message(telemetryPayload);
  message.setContentEncoding(StandardCharsets.UTF_8.name());
  message.setContentTypeFinal("application/json");
  message.setProperty("iothub-creation-time-utc", Instant.now().toString());

  deviceClient.sendEventAsync(message, new MessageIotHubEventCallback(), message);
  log.debug("My Telemetry: Sent - {\"{}\": {}°C} with message Id {}.", telemetryName, temperature, message.getMessageId());
  temperatureReadings.put(new Date(), temperature);
}
```

# <a name="c"></a>[C#](#tab/csharp)

```csharp
private async Task SendTemperatureTelemetryAsync()
{
  const string telemetryName = "temperature";

  string telemetryPayload = $"{{ \"{telemetryName}\": {_temperature} }}";
  using var message = new Message(Encoding.UTF8.GetBytes(telemetryPayload))
  {
      ContentEncoding = "utf-8",
      ContentType = "application/json",
  };
  message.Properties.Add("iothub-creation-time-utc", DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"));
  await _deviceClient.SendEventAsync(message);
  _logger.LogDebug($"Telemetry: Sent - {{ \"{telemetryName}\": {_temperature}°C }}.");
}
```

# <a name="python"></a>[Python](#tab/python)

```python
async def send_telemetry_from_thermostat(device_client, telemetry_msg):
    msg = Message(json.dumps(telemetry_msg))
    msg.custom_properties["iothub-creation-time-utc"] = datetime.now(timezone.utc).isoformat()
    msg.content_encoding = "utf-8"
    msg.content_type = "application/json"
    print("Sent message")
    await device_client.send_message(msg)
```

---

En el fragmento de código siguiente se muestra esta propiedad en el mensaje exportado a Blob Storage:

```json
{
  "applicationId":"5782ed70-b703-4f13-bda3-1f5f0f5c678e",
  "messageSource":"telemetry",
  "deviceId":"sample-device-01",
  "schema":"default@v1",
  "templateId":"urn:modelDefinition:mkuyqxzgea:e14m1ukpn",
  "enqueuedTime":"2021-01-29T16:45:39.143Z",
  "telemetry":{
    "temperature":8.341033560421833
  },
  "messageProperties":{
    "iothub-creation-time-utc":"2021-01-29T16:45:39.021Z"
  },
  "enrichments":{}
}
```

## <a name="property-changes-format"></a>Formato de cambios de propiedades

Cada mensaje o registro representa un cambio en una propiedad de la nube o del dispositivo. En el caso de las propiedades del dispositivo, solo los cambios en el valor notificado se exportan como un mensaje independiente. La información del mensaje exportado incluye:

- `applicationId`: el identificador de la aplicación IoT Central.
- `messageSource`: el origen del mensaje (`properties`).
- `messageType`: `cloudPropertyChange`, `devicePropertyDesiredChange` o `devicePropertyReportedChange`.
- `deviceId`:  el identificador del dispositivo que envió el mensaje de telemetría.
- `schema`: el nombre y la versión del esquema de carga.
- `enqueuedTime`: la hora a la que IoT Central detectó este cambio.
- `templateId`: el identificador de la plantilla de dispositivo asociada al dispositivo.
- `enrichments`: cualquier enriquecimiento configurado en la exportación.

En Event Hubs y Service Bus, IoT Central exporta los nuevos datos de los mensajes al centro de eventos o a la cola o el tema de Service Bus casi en tiempo real. En las propiedades de usuario (también conocidas como propiedades de la aplicación) de cada mensaje, `iotcentral-device-id`, `iotcentral-application-id`, `iotcentral-message-source` e `iotcentral-message-type` se incluyen automáticamente.

Para Blob Storage, los mensajes se procesan por lotes y se exportan una vez por minuto.

En el ejemplo siguiente, se muestra un mensaje de cambio de propiedad exportado recibido en Azure Blob Storage.

```json
{
    "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
    "messageSource": "properties",
    "messageType": "cloudPropertyChange",
    "deviceId": "18a985g1fta",
    "schema": "default@v1",
    "templateId": "urn:qugj6vbw5:___qbj_27r",
    "enqueuedTime": "2020-08-05T22:37:32.942Z",
    "properties": [{
        "name": "MachineSerialNumber",
        "value": "abc"
    }],
    "enrichments": {
        "userSpecifiedKey" : "sampleValue"
    }
}
```

## <a name="device-lifecycle-changes-format"></a>Formato de los cambios de ciclo de vida de dispositivo

Cada mensaje o registro representa un cambio en un único dispositivo. La información del mensaje exportado incluye:

- `applicationId`: el identificador de la aplicación IoT Central.
- `messageSource`: el origen del mensaje (`deviceLifecycle`).
- `messageType`: `registered` o `deleted`.
- `deviceId`: el id. del dispositivo que se ha cambiado.
- `schema`: el nombre y la versión del esquema de carga.
- `templateId`: el identificador de la plantilla de dispositivo asociada al dispositivo.
- `enqueuedTime`: la hora a la que se produjo este cambio en IoT Central.
- `enrichments`: cualquier enriquecimiento configurado en la exportación.

En Event Hubs y Service Bus, IoT Central exporta los nuevos datos de los mensajes al centro de eventos o a la cola o el tema de Service Bus casi en tiempo real. En las propiedades de usuario (también conocidas como propiedades de la aplicación) de cada mensaje, `iotcentral-device-id`, `iotcentral-application-id`, `iotcentral-message-source` e `iotcentral-message-type` se incluyen automáticamente.

Para Blob Storage, los mensajes se procesan por lotes y se exportan una vez por minuto.

En el ejemplo siguiente, se muestra un mensaje del ciclo de vida de dispositivo exportado que se ha recibido en Azure Blob Storage.

```json
{
  "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
  "messageSource": "deviceLifecycle",
  "messageType": "registered",
  "deviceId": "1vzb5ghlsg1",
  "schema": "default@v1",
  "templateId": "urn:qugj6vbw5:___qbj_27r",
  "enqueuedTime": "2021-01-01T22:26:55.455Z",
  "enrichments": {
    "userSpecifiedKey": "sampleValue"
  }
}
```
## <a name="device-template-lifecycle-changes-format"></a>Formato de los cambios del ciclo de vida de plantilla de dispositivo

Cada mensaje o registro representa un cambio en una única plantilla de dispositivo publicada. La información del mensaje exportado incluye:

- `applicationId`: el identificador de la aplicación IoT Central.
- `messageSource`: el origen del mensaje (`deviceTemplateLifecycle`).
- `messageType`: `created`, `updated` o `deleted`.
- `schema`: el nombre y la versión del esquema de carga.
- `templateId`: el identificador de la plantilla de dispositivo asociada al dispositivo.
- `enqueuedTime`: la hora a la que se produjo este cambio en IoT Central.
- `enrichments`: cualquier enriquecimiento configurado en la exportación.

En Event Hubs y Service Bus, IoT Central exporta los nuevos datos de los mensajes al centro de eventos o a la cola o el tema de Service Bus casi en tiempo real. En las propiedades de usuario (también conocidas como propiedades de la aplicación) de cada mensaje, `iotcentral-device-id`, `iotcentral-application-id`, `iotcentral-message-source` e `iotcentral-message-type` se incluyen automáticamente.

Para Blob Storage, los mensajes se procesan por lotes y se exportan una vez por minuto.

En el ejemplo siguiente, se muestra un mensaje del ciclo de vida de dispositivo exportado que se ha recibido en Azure Blob Storage.

```json
{
  "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
  "messageSource": "deviceTemplateLifecycle",
  "messageType": "created",
  "schema": "default@v1",
  "templateId": "urn:qugj6vbw5:___qbj_27r",
  "enqueuedTime": "2021-01-01T22:26:55.455Z",
  "enrichments": {
    "userSpecifiedKey": "sampleValue"
  }
}
```

## <a name="comparison-of-legacy-data-export-and-data-export"></a>Comparación entre la exportación de datos heredada y la exportación de datos

En la siguiente tabla se observan las diferencias entre la [exportación de datos heredada](howto-export-data-legacy.md) y las características de la exportación de datos nueva:

| Capacidades  | Exportación de datos heredada | Exportación de datos nueva |
| :------------- | :---------- | :----------- |
| Tipos de datos disponibles | Telemetría, dispositivos y plantillas de dispositivo | Telemetría, cambios de propiedades, cambios de ciclo de vida de dispositivo, cambios de ciclo de vida de plantilla de dispositivo |
| Filtrado | None | Depende del tipo de datos exportado. Para datos de telemetría, filtrado por telemetría, propiedades de mensaje, valores de propiedad |
| Características enriquecidas | None | Enriquecer con una cadena personalizada o un valor de propiedad en el dispositivo |
| Destinations | Azure Event Hubs, colas y temas de Azure Service Bus, Azure Blob Storage | Igual que para los webhooks y la exportación de datos heredada|
| Versiones de aplicación admitidas | V2, V3 | Solo v3 |
| Límites destacados | 5 exportaciones por aplicación, 1 destino por exportación | 10 conexiones de exportaciones y destinos por aplicación |

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo usar la nueva exportación de datos, puede aprender a [usar el análisis en IoT Central](./howto-create-analytics.md).
