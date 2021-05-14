---
title: Notificaciones de eventos
titleSuffix: Azure Digital Twins
description: Descubra cómo interpretar distintos tipos de eventos y sus diferentes mensajes de notificación.
author: baanders
ms.author: baanders
ms.date: 4/8/2021
ms.topic: conceptual
ms.service: digital-twins
ms.custom: contperf-fy21q4
ms.openlocfilehash: 625f69b0088ffe6931410afc6f2a825c3156fd70
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209956"
---
# <a name="event-notifications"></a>Notificaciones de eventos

Los distintos eventos de Azure Digital Twins generan **notificaciones**, que permiten que el back-end de solución sepa que están ocurriendo distintas acciones. A continuación, se [enrutan](concepts-route-events.md) a distintas ubicaciones dentro y fuera de Azure Digital Twins que pueden usar esta información para tomar medidas.

Son varios los tipos de notificaciones que se pueden generar y los mensajes de notificación pueden tener un aspecto distinto en función del tipo de evento que los generó. En este artículo se proporcionan detalles sobre los diferentes tipos de mensajes y su aspecto.

En este gráfico se muestran los distintos tipos de notificación:

[!INCLUDE [digital-twins-notifications.md](../../includes/digital-twins-notifications.md)]

## <a name="notification-structure"></a>Estructura de notificaciones

En general, las notificaciones constan de dos partes: el encabezado y el cuerpo. 

### <a name="event-notification-headers"></a>Encabezados de notificaciones de eventos

Los encabezados de los mensajes de notificación se representan con pares clave-valor. En función del protocolo usado (MQTT, AMQP o HTTP), los encabezados del mensaje se serializarán de manera distinta. En esta sección se describe información general de los encabezados de los mensajes de notificación, independientemente del protocolo y la serialización específicos elegidos.

Algunas notificaciones se ajustan al estándar [CloudEvents](https://cloudevents.io/). La conformidad con CloudEvents es la siguiente.
* Las notificaciones emitidas desde dispositivos continúan siguiendo las especificaciones existentes para las notificaciones.
* Las notificaciones procesadas y emitidas por IoT Hub continúan siguiendo las especificaciones existentes para las notificaciones, excepto en las que IoT Hub opta por admitir CloudEvents, como a través de Event Grid.
* Las notificaciones emitidas desde [gemelos digitales](concepts-twins-graph.md) con un [modelo](concepts-models.md) se ajustan a CloudEvents.
* Las notificaciones procesadas y emitidas por Azure Digital Twins se ajustan a CloudEvents.

Los servicios tienen que agregar un número de secuencia en todas las notificaciones para indicar su orden, o bien mantener su propia ordenación de alguna otra manera. 

Las notificaciones emitidas por Azure Digital Twins a Event Grid reciben automáticamente el formato del esquema de CloudEvents o del esquema de EventGridEvent, en función del tipo de esquema que se define en el tema de Event Grid. 

Los atributos de extensión de los encabezados se agregarán como propiedades en el esquema de Event Grid dentro de la carga. 

### <a name="event-notification-bodies"></a>Cuerpos de las notificaciones de eventos

Los cuerpos de los mensajes de notificación se describen aquí en JSON. En función de la serialización deseada para el cuerpo del mensaje (por ejemplo, con JSON, CBOR, Protobuf, etc.), el cuerpo del mensaje se puede serializar de forma distinta.

El conjunto de campos que contiene el cuerpo varía con los distintos tipos de notificación.

En las siguientes secciones se explican con más detalle los distintos tipos de notificaciones que emite IoT Hub y Azure Digital Twins (u otros servicios de Azure IoT). Aprenderá qué desencadena cada tipo de notificación y el conjunto de campos incluido en cada tipo de cuerpo de notificación.

## <a name="digital-twin-change-notifications"></a>Notificaciones de cambios de gemelo digital

Las **notificaciones de cambio de gemelo digital** se desencadenan cuando se actualiza un gemelo digital, como muestra a continuación:
* Cuando cambian los metadatos o los valores de la propiedad.
* Cuando cambian los metadatos del componente o del gemelo digital. Un ejemplo de este escenario es cambiar el modelo de un gemelo digital.

### <a name="properties"></a>Propiedades

Estos son los campos del cuerpo de una notificación de cambio de gemelo digital.

| Nombre    | Value |
| --- | --- |
| `id` | Identificador de la notificación, como un UUID o un contador mantenido por el servicio. `source` + `id` es único para cada evento distinto |
| `source` | Nombre de la instancia de IoT Hub o de Azure Digital Twins, como *myhub.azure-devices.net* o *mydigitaltwins.westus2.azuredigitaltwins.net* |
| `data` | Un documento de revisión JSON que describe la actualización realizada en el gemelo. Para más información, consulte [Detalles del cuerpo](#body-details) a continuación. |
| `specversion` | *1.0*<br>El mensaje se ajusta a esta versión de la [especificación CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `Microsoft.DigitalTwins.Twin.Update` |
| `datacontenttype` | `application/json` |
| `subject` | Identificador del gemelo digital. |
| `time` | Marca de tiempo para el momento en que se produjo la operación en el gemelo digital. |
| `traceparent` | Un contexto de seguimiento de W3C para el evento. |

### <a name="body-details"></a>Detalles del cuerpo

Dentro del mensaje, el campo `data` contiene un documento de revisión JSON que contiene la actualización del gemelo digital.

Por ejemplo, supongamos que se actualizó un gemelo digital con la revisión siguiente.

:::code language="json" source="~/digital-twins-docs-samples/models/patch-component-2.json":::

Los datos de la notificación correspondiente (si la ejecuta sincrónicamente el servicio, como la actualización de un gemelo digital por parte de Azure Digital Twins) tendría un cuerpo como el siguiente:

```json
{
    "modelId": "dtmi:example:com:floor4;2",
    "patch": [
      {
        "value": 40,
        "path": "/Temperature",
        "op": "replace"
      },
      {
        "value": 30,
        "path": "/comp1/prop1",
        "op": "add"
      }
    ]
  }
```

Esta es la información que aparecerá en el campo `data` del mensaje de notificación del ciclo de vida.

## <a name="digital-twin-lifecycle-notifications"></a>Notificaciones de ciclo de vida de los gemelos digitales

Todos los [gemelos digitales](concepts-twins-graph.md) emiten notificaciones, independientemente de si representan [dispositivos de IoT Hub en Azure Digital Twins](how-to-ingest-iot-hub-data.md) o no. Esto se debe a las **notificaciones del ciclo de vida**, que son sobre el propio gemelo digital.

Las notificaciones del ciclo de vida se desencadenan cuando:
* Se crea un gemelo digital
* Se elimina un gemelo digital

### <a name="properties"></a>Propiedades

Estos son los campos del cuerpo de una notificación del ciclo de vida.

| Nombre | Value |
| --- | --- |
| `id` | Identificador de la notificación, como un UUID o un contador mantenido por el servicio. `source` + `id` es único para cada evento distinto. |
| `source` | Nombre de la instancia de IoT Hub o de Azure Digital Twins, como *myhub.azure-devices.net* o *mydigitaltwins.westus2.azuredigitaltwins.net* |
| `data` | Los datos del gemelo que experimenta el evento de ciclo de vida. Para más información, consulte [Detalles del cuerpo](#body-details-1) a continuación. |
| `specversion` | *1.0*<br>El mensaje se ajusta a esta versión de la [especificación CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `Microsoft.DigitalTwins.Twin.Create`<br>`Microsoft.DigitalTwins.Twin.Delete` |
| `datacontenttype` | `application/json` |
| `subject` | Identificador del gemelo digital. |
| `time` | Marca de tiempo para el momento en que se produjo la operación en el gemelo. |
| `traceparent` | Un contexto de seguimiento de W3C para el evento. |

### <a name="body-details"></a>Detalles del cuerpo

Este es un ejemplo de un mensaje de notificación del ciclo de vida: 

```json
{
  "specversion": "1.0",
  "id": "d047e992-dddc-4a5a-b0af-fa79832235f8",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "contoso-adt.api.wus2.digitaltwins.azure.net",
  "data": {
    "$dtId": "floor1",
    "$etag": "W/\"e398dbf4-8214-4483-9d52-880b61e491ec\"",
    "$metadata": {
      "$model": "dtmi:example:Floor;1"
    }
  },
  "subject": "floor1",
  "time": "2020-06-23T19:03:48.9700792Z",
  "datacontenttype": "application/json",
  "traceparent": "00-18f4e34b3e4a784aadf5913917537e7d-691a71e0a220d642-01"
}
```

Dentro del mensaje, el campo `data` contiene los datos del gemelo digital afectado, representados en formato JSON. El esquema para esto es *Recurso de Digital Twins 7.1*.

En el caso de los eventos de creación, la carga de `data` refleja el estado del gemelo una vez creado el recurso, por lo que debe incluir todos los elementos generados por el sistema tal como una llamada `GET`.

Este es un ejemplo de los datos para un dispositivo [IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md), con componentes y sin propiedades de nivel superior. Se deben omitir las propiedades que no tienen sentido para los dispositivos (como las propiedades notificadas). Esta es la información que aparecerá en el campo `data` del mensaje de notificación del ciclo de vida.

```json
{
  "$dtId": "device-digitaltwin-01",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "thermostat": {
    "temperature": 80,
    "humidity": 45,
    "$metadata": {
      "$model": "dtmi:com:contoso:Thermostat;1",
      "temperature": {
        "desiredValue": 85,
        "desiredVersion": 3,
        "ackVersion": 2,
        "ackCode": 200,
        "ackDescription": "OK"
      },
      "humidity": {
        "desiredValue": 40,
        "desiredVersion": 1,
        "ackVersion": 1,
        "ackCode": 200,
        "ackDescription": "OK"
      }
    }
  },
  "$metadata": {
    "$model": "dtmi:com:contoso:Thermostat_X500;1",
  }
}
```

Este es otro ejemplo de los datos del gemelo digital. Se basa en un [modelo](concepts-models.md) y no admite componentes:

```json
{
  "$dtId": "logical-digitaltwin-01",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "avgTemperature": 70,
  "comfortIndex": 85,
  "$metadata": {
    "$model": "dtmi:com:contoso:Building;1",
    "avgTemperature": {
      "desiredValue": 72,
      "desiredVersion": 5,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "OK"
    },
    "comfortIndex": {
      "desiredValue": 90,
      "desiredVersion": 1,
      "ackVersion": 3,
      "ackCode": 200,
      "ackDescription": "OK"
    }
  }
}
```

## <a name="digital-twin-relationship-change-notifications"></a>Notificaciones de cambio de relación de gemelo digital

Las **notificaciones de cambio de relación** se desencadenan cuando se crea, actualiza o elimina un gemelo digital. 

### <a name="properties"></a>Propiedades

Estos son los campos del cuerpo de una notificación de cambio de relación.

| Nombre    | Value |
| --- | --- |
| `id` | Identificador de la notificación, como un UUID o un contador mantenido por el servicio. `source` + `id` es único para cada evento distinto |
| `source` | Nombre de la instancia de Azure Digital Twins, como *mydigitaltwins.westus2.azuredigitaltwins.net* |
| `data` | La carga de la relación que se cambió. Para más información, consulte [Detalles del cuerpo](#body-details-2) a continuación. |
| `specversion` | *1.0*<br>El mensaje se ajusta a esta versión de la [especificación CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `Microsoft.DigitalTwins.Relationship.Create`<br>`Microsoft.DigitalTwins.Relationship.Update`<br>`Microsoft.DigitalTwins.Relationship.Delete` |
| `datacontenttype` | `application/json` |
| `subject` | Identificador de la relación, como `<twinID>/relationships/<relationshipID>` |
| `time` | Marca de tiempo para el momento en que se produjo la relación. |
| `traceparent` | Un contexto de seguimiento de W3C para el evento. |

### <a name="body-details"></a>Detalles del cuerpo

Dentro del mensaje, el campo `data` contiene la carga de una relación, en formato JSON. Usa el mismo formato como una solicitud `GET` para una relación a través de la [API DigitalTwins](/rest/api/digital-twins/dataplane/twins). 

Este es un ejemplo de los datos de una notificación de relación de actualización. "Actualizar una relación" significa que las propiedades de la relación han cambiado, por lo que los datos muestran la propiedad actualizada y su nuevo valor. Esta es la información que aparecerá en el campo `data` del mensaje de notificación de la relación de los gemelos digitales.

```json
{
    "modelId": "dtmi:example:Floor;1",
    "patch": [
      {
        "value": "user3",
        "path": "/ownershipUser",
        "op": "replace"
      }
    ]
  }
```

Este es un ejemplo de los datos de una notificación de creación o eliminación de una relación. Para `Relationship.Delete`, el cuerpo es el mismo que la solicitud `GET` y obtiene el estado más reciente antes de la eliminación.

```json
{
    "$relationshipId": "device_to_device",
    "$etag": "W/\"72479873-0083-41a8-83e2-caedb932d881\"",
    "$relationshipName": "Connected",
    "$targetId": "device2",
    "connectionType": "WIFI"
}
```

## <a name="digital-twin-telemetry-messages"></a>Mensajes de telemetría de gemelos digitales

Los **mensajes de telemetría** se reciben Azure Digital Twins desde dispositivos conectados que recopilan y envían medidas.

### <a name="properties"></a>Propiedades

Estos son los campos del cuerpo de un mensaje de telemetría.

| Nombre    | Value |
| --- | --- |
| `id` | Identificador de la notificación, que proporciona el cliente al llamar a la API de telemetría. |
| `source` | Nombre completo del gemelo al que se envió el evento de telemetría. Utilice el siguiente formato: `<yourDigitalTwinInstance>.api.<yourRegion>.digitaltwins.azure.net/<twinId>`. |
| `specversion` | *1.0*<br>El mensaje se ajusta a esta versión de la [especificación CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `microsoft.iot.telemetry` |
| `data` | Mensaje de telemetría que se ha enviado a los gemelos. La carga está sin modificar y puede que no se alinee con el esquema del gemelo al que se ha enviado la telemetría. |
| `dataschema` | el esquema de datos es el identificador del modelo del gemelo o el componente que emite la telemetría. Por ejemplo, `dtmi:example:com:floor4;2`. |
| `datacontenttype` | `application/json` |
| `traceparent` | Un contexto de seguimiento de W3C para el evento. |

### <a name="body-details"></a>Detalles del cuerpo

El cuerpo contiene la medida de telemetría junto con información contextual sobre el dispositivo.

Este es un cuerpo del mensaje de telemetría de ejemplo: 

```json
{
  "specversion": "1.0",
  "id": "df5a5992-817b-4e8a-b12c-e0b18d4bf8fb",
  "type": "microsoft.iot.telemetry",
  "source": "contoso-adt.api.wus2.digitaltwins.azure.net/digitaltwins/room1",
  "data": {
    "Temperature": 10
  },
  "dataschema": "dtmi:example:com:floor4;2",
  "datacontenttype": "application/json",
  "traceparent": "00-7e3081c6d3edfb4eaf7d3244b2036baa-23d762f4d9f81741-01"
}
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la entrega de eventos a distintos destinos, mediante puntos de conexión y rutas:
* [Conceptos: Rutas de eventos](concepts-route-events.md)
