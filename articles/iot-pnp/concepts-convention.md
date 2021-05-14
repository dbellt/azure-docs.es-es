---
title: Convenciones de IoT Plug and Play | Microsoft Docs
description: Descripción de las convenciones que IoT Plug and Play espera que los dispositivos usen cuando envían telemetría y propiedades y controlan los comandos y las actualizaciones de propiedades.
author: rido-min
ms.author: rmpablos
ms.date: 07/10/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 88297671dedec5b4001677f88ac5b9f86de107e3
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108325753"
---
# <a name="iot-plug-and-play-conventions"></a>Convenciones de IoT Plug and Play

Los dispositivos de IoT Plug and Play deben seguir un conjunto de convenciones cuando intercambien mensajes con un centro de IoT. Los dispositivos de IoT Plug and Play usan el protocolo MQTT para comunicarse con IoT Hub.

Los dispositivos pueden incluir [módulos](../iot-hub/iot-hub-devguide-module-twins.md) o implementarse en un [módulo de IoT Edge](../iot-edge/about-iot-edge.md) hospedado por el tiempo de ejecución de IoT Edge.

Se describen la telemetría, las propiedades y los comandos que implementa un dispositivo IoT Plug and Play con un _modelo_ de [lenguaje de definición de Digital Twins v2 (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl). En este artículo se hace referencia a dos tipos de modelo:

- **Sin componentes**: un modelo sin componentes. El modelo declara la telemetría, las propiedades y los comandos como propiedades de nivel superior en la sección de contenido de la interfaz principal. En la herramienta de exploración Azure IoT, este modelo aparece como un único _componente predeterminado_.
- **Varios componentes**: modelo compuesto por dos o más interfaces. Una interfaz principal, que aparece como _componente predeterminado_, con telemetría, propiedades y comandos. Una o más interfaces declaradas como componentes con telemetría, propiedades y comandos adicionales.

Para más información, consulte la [Guía de modelado de IoT Plug and Play](concepts-modeling-guide.md).

## <a name="identify-the-model"></a>Identificación del modelo

Para anunciar el modelo que implementa, un dispositivo o módulo IoT Plug and Play incluye el identificador del modelo en el paquete de conexión MQTT agregando `model-id` al campo `USERNAME`.

Para identificar el modelo que implementa un dispositivo o módulo, un servicio puede obtener el identificador de modelo en:

- El campo `modelId` del dispositivo gemelo.
- El campo `$metadata.$model` del gemelo digital.
- Una notificación de cambios de gemelo digital.

## <a name="telemetry"></a>Telemetría

La telemetría enviada desde un dispositivo sin componentes no requiere metadatos adicionales. El sistema agrega la propiedad `dt-dataschema`.

La telemetría enviada desde un dispositivo con varios componentes debe agregar `$.sub` como una propiedad de mensaje. El sistema agrega las propiedades `dt-subject` y `dt-dataschema`.

## <a name="read-only-properties"></a>Propiedades de solo lectura

### <a name="sample-no-component-read-only-property"></a>Propiedad de solo lectura sin componentes de ejemplo

Un dispositivo o módulo puede enviar cualquier JSON válido que sigue las reglas de DTDL v2.

DTDL:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:example: Thermostat;1",
  "@type": "Interface",
  "contents": [
    {
      "@type": "Property",
      "name": "temperature",
      "schema": "double"
    }
  ]
}
```

Ejemplo de carga de propiedad notificada:

```json
"reported" :
{
  "temperature" : 21.3
}
```

### <a name="sample-multiple-components-read-only-property"></a>Propiedad de solo lectura de varios componentes de ejemplo

El dispositivo o módulo debe agregar el marcador `{"__t": "c"}` para indicar que el elemento hace referencia a un componente.

DTDL que hace referencia a un componente:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:TemperatureController;1",
  "@type": "Interface",
  "displayName": "Temperature Controller",
  "contents": [
    {
      "@type" : "Component",
      "schema": "dtmi:com:example:Thermostat;1",
      "name": "thermostat1"
    }
  ]
}
```

DTDL que define el componente:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "contents": [
    {
      "@type": "Property",
      "name": "temperature",
      "schema": "double"
    }
  ]
}
```

Ejemplo de carga de propiedad notificada:

```json
"reported": {
  "thermostat1": {
    "__t": "c",
    "temperature": 21.3
  }
}
```

## <a name="writable-properties"></a>Propiedades editables

El dispositivo o módulo debe confirmar que recibió la propiedad mediante el envío de una propiedad notificada. La propiedad notificada debe incluir:

- `value`: el valor real de la propiedad (normalmente el valor recibido, pero el dispositivo puede decidir informar de un valor diferente).
- `ac`: un código de confirmación que usa un código de estado HTTP.
- `av`: una versión de confirmación que hace referencia a la `$version` de la propiedad deseada. Puede encontrar este valor en la carga de JSON de la propiedad deseada.
- `ad` : una descripción de confirmación opcional.

Cuando se inicia un dispositivo, debe solicitar el dispositivo gemelo y comprobar si hay actualizaciones de propiedades que se puedan escribir. Si la versión de una propiedad grabable aumenta mientras el dispositivo estaba sin conexión, el dispositivo debe enviar una respuesta de propiedad notificada para confirmar que recibió la actualización.

Cuando un dispositivo se inicia por primera vez, puede enviar un valor inicial para una propiedad notificada si no recibe una propiedad inicial deseada del centro. En este caso, el dispositivo debe establecer `av` como `1`. Por ejemplo:

```json
"reported": {
  "targetTemperature": {
    "value": 20.0,
    "ac": 200,
    "av": 1,
    "ad": "initialize"
  }
}
```

Un dispositivo puede usar la propiedad notificada para proporcionar otra información al centro. Por ejemplo, el dispositivo puede responder con una serie de mensajes en curso, como:

```json
"reported": {
  "targetTemperature": {
    "value": 35.0,
    "ac": 202,
    "av": 3,
    "ad": "In-progress - reporting current temperature"
  }
}
```

Cuando el dispositivo alcanza la temperatura de destino, envía el siguiente mensaje:

```json
"reported": {
  "targetTemperature": {
    "value": 20.0,
    "ac": 200,
    "av": 3,
    "ad": "Reached target temperature"
  }
}
```

Un dispositivo podría informar de un error como:

```json
"reported": {
  "targetTemperature": {
    "value": 120.0,
    "ac": 500,
    "av": 3,
    "ad": "Target temperature out of range. Valid range is 10 to 99."
  }
}
```

### <a name="sample-no-component-writable-property"></a>Propiedad editable sin componentes de ejemplo

Cuando un dispositivo recibe varias propiedades comunicadas en una sola carga útil, puede enviar las respuestas de propiedad notificada mediante varias cargas.

Un dispositivo o módulo puede enviar cualquier JSON válido que siga las reglas de DTDL v2:

DTDL:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:example: Thermostat;1",
  "@type": "Interface",
  "contents": [
    {
      "@type": "Property",
      "name": "targetTemperature",
      "schema": "double",
      "writable": true
    }
  ]
}
```

Carga de propiedades deseadas de ejemplo:

```json
"desired" :
{
  "targetTemperature" : 21.3,
  "targetHumidity" : 80,
  "$version" : 3
}
```

Carga de primera propiedad notificada de ejemplo:

```json
"reported": {
  "targetTemperature": {
    "value": 21.3,
    "ac": 200,
    "av": 3,
    "ad": "complete"
  }
}
```

Carga de segunda propiedad notificada de ejemplo:

```json
"reported": {
  "targetHumidity": {
    "value": 80,
    "ac": 200,
    "av": 3,
    "ad": "complete"
  }
}
```

### <a name="sample-multiple-components-writable-property"></a>Propiedad editable de varios componentes de ejemplo

El dispositivo o módulo debe agregar el marcador `{"__t": "c"}` para indicar que el elemento hace referencia a un componente.

El marcador solo se envía para las actualizaciones de las propiedades definidas en un componente. Las actualizaciones de las propiedades definidas en el componente predeterminado no incluyen el marcador; consulte [Propiedad editable sin componentes de ejemplo](#sample-no-component-writable-property).

Cuando un dispositivo recibe varias propiedades comunicadas en una sola carga útil, puede enviar las respuestas de propiedad notificada mediante varias cargas.

El dispositivo o módulo debe confirmar que recibió las propiedades mediante el envío de propiedades notificadas:

DTDL que hace referencia a un componente:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:TemperatureController;1",
  "@type": "Interface",
  "displayName": "Temperature Controller",
  "contents": [
    {
      "@type" : "Component",
      "schema": "dtmi:com:example:Thermostat;1",
      "name": "thermostat1"
    }
  ]
}
```

DTDL que define el componente:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "contents": [
    {
      "@type": "Property",
      "name": "targetTemperature",
      "schema": "double",
      "writable": true
    }
  ]
}
```

Carga de propiedades deseadas de ejemplo:

```json
"desired": {
  "thermostat1": {
    "__t": "c",
    "targetTemperature": 21.3,
    "targetHumidity": 80,
    "$version" : 3
  }
}
```

Carga de primera propiedad notificada de ejemplo:

```json
"reported": {
  "thermostat1": {
    "__t": "c",
    "targetTemperature": {
      "value": 23,
      "ac": 200,
      "av": 3,
      "ad": "complete"
    }
  }
}
```

Carga de segunda propiedad notificada de ejemplo:

```json
"reported": {
  "thermostat1": {
    "__t": "c",
    "targetHumidity": {
      "value": 80,
      "ac": 200,
      "av": 3,
      "ad": "complete"
    }
  }
}
```

## <a name="commands"></a>Comandos:

Las interfaces sin componentes usan el nombre del comando sin prefijo.

En un dispositivo o módulo, las interfaces con varios componentes usan nombres de comando con el siguiente formato: `componentName*commandName`.

## <a name="next-steps"></a>Pasos siguientes

Ahora que aprendió sobre las convenciones de IoT Plug and Play, estos son algunos recursos adicionales:

- [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl)
- [SDK para dispositivos C](/azure/iot-hub/iot-c-sdk-ref/)
- [API REST de IoT](/rest/api/iothub/device)
- [Guía de modelado de IoT Plug and Play](concepts-modeling-guide.md)
