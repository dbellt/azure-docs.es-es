---
title: Empleo de la API de REST para administrar dispositivos en Azure IoT Central
description: Uso de la API de REST de IoT Central para controlar dispositivos en una aplicación
author: dominicbetts
ms.author: dobett
ms.date: 06/01/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 56ee24985da57c6da76638b659e5a07e68d0f516
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110788953"
---
# <a name="how-to-use-the-iot-central-rest-api-to-control-devices"></a>Empleo de la API de REST de IoT Central para controlar dispositivos

La API de REST de IoT Central permite desarrollar aplicaciones cliente que se integran con aplicaciones de IoT Central. Puede usar la API de REST para controlar dispositivos de la aplicación de IoT Central. La API de REST permite:

- Leer el último valor de telemetría conocido de un dispositivo.
- Leer los valores de propiedad de un dispositivo.
- Establecer propiedades editables en un dispositivo.
- Llamar a comandos en un dispositivo.

En este artículo se explica cómo usar la API `/devices/{device_id}` para controlar dispositivos individuales. También se pueden usar trabajos para controlar dispositivos en bloque.

Un dispositivo puede agrupar las propiedades, la telemetría y los comandos que admite en _componentes_ y _módulos_.

Cada llamada API de REST de IoT Central requiere un encabezado de autorización. Para obtener más información, vea los [procedimientos de autenticación y autorización de llamadas a la API REST de IoT Central](howto-authorize-rest-api.md).

Para obtener documentación de referencia sobre la API de REST de IoT Central, vea [Referencia de la API de REST de Azure IoT Central](/rest/api/iotcentral/).

## <a name="components-and-modules"></a>Componentes y módulos

Los componentes permiten agrupar y reutilizar capacidades de un dispositivo. Para obtener más información sobre los componentes y los modelos de dispositivo, vea [Guía de modelado de IoT Plug and Play](../../iot-pnp/concepts-modeling-guide.md).

No todas las plantillas de dispositivo usan componentes. En la captura de pantalla siguiente se muestra la plantilla de dispositivo de un [termostato](https://github.com/Azure/iot-plugandplay-models/blob/main/dtmi/com/example/thermostat-2.json) simple donde todas las capacidades se definen en una sola interfaz denominada **Componente predeterminado**:

:::image type="content" source="media/howto-control-devices-with-rest-api/thermostat-device.png" alt-text="Captura de pantalla que muestra un dispositivo de termostato simple sin componentes.":::

En la captura de pantalla siguiente se muestra una plantilla de dispositivo de [controlador de temperatura](https://github.com/Azure/iot-plugandplay-models/blob/main/dtmi/com/example/temperaturecontroller-2.json) que usa componentes. El controlador de temperatura tiene dos componentes de termostato y un componente de información del dispositivo:

:::image type="content" source="media/howto-control-devices-with-rest-api/temperature-controller-device.png" alt-text="Captura de pantalla que muestra un dispositivo de controlador de temperatura con dos componentes de termostato y un componente de información del dispositivo.":::

En IoT Central, módulo se refiere a un módulo IoT Edge que se ejecuta en un dispositivo IoT Edge conectado. Un módulo puede tener un modelo simple, como el termostato que no usa componentes. Un módulo también puede usar componentes para organizar un conjunto más complejo de capacidades. En la captura de pantalla siguiente se muestra un ejemplo de una plantilla de dispositivo que usa módulos. El dispositivo de sensor ambiental tiene un módulo de nombre `SimulatedTemperatureSensor` y una interfaz heredada de nombre `management`:

:::image type="content" source="media/howto-control-devices-with-rest-api/environmental-sensor-device.png" alt-text="Captura de pantalla que muestra un dispositivo de sensor ambiental con un módulo.":::

## <a name="get-a-device-component"></a>Obtención de un componente de dispositivo

Use la siguiente solicitud para recuperar los componentes de un dispositivo de nombre `temperature-controller-01`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/components?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente. La matriz `value` contiene detalles de cada componente del dispositivo:

```json
{
  "value": [
    {
      "@type": "Component",
      "name": "thermostat1",
      "displayName": "Thermostat One",
      "description": "Thermostat One of Two."
    },
    {
      "@type": "Component",
      "name": "thermostat2",
      "displayName": "Thermostat Two",
      "description": "Thermostat Two of Two."
    },
    {
      "@type": "Component",
      "name": "deviceInformation",
      "displayName": "Device Information interface",
      "description": "Optional interface with basic device hardware information."
    }
  ]
}
```

## <a name="get-a-device-module"></a>Obtención de un módulo de dispositivo

Use la siguiente solicitud para recuperar una lista de módulos que se ejecutan en un dispositivo IoT Edge conectado de nombre `environmental-sensor-01`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/environmental-sensor-01/modules?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente. La matriz de módulos solo incluye los módulos personalizados que se ejecutan en el dispositivo IoT Edge, no los módulos `$edgeAgent` y `$edgeHub` integrados:

```json
{
  "value": [
    {
      "@type": [
        "Relationship",
        "EdgeModule"
      ],
      "name": "SimulatedTemperatureSensor",
      "displayName": "SimulatedTemperatureSensor"
    }
  ]
}
```

Use la siguiente solicitud para recuperar una lista de los componentes de un módulo de nombre `SimulatedTemperatureSensor`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/environmental-sensor-01/modules?api-version=1.0
```

## <a name="read-telemetry"></a>Lectura de telemetría

Use la siguiente solicitud para recuperar el último valor de telemetría conocido de un dispositivo que no usa componentes. En este ejemplo, el nombre del dispositivo es `thermostat-01` y el de la telemetría `temperature`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/thermostat-01/telemetry/temperature?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "timestamp": "2021-03-24T12:33:15.223Z",
  "value": 40.10993804456927
}
```

Use la siguiente solicitud para recuperar el último valor de telemetría conocido de un dispositivo que sí usa componentes. En este ejemplo, el nombre del dispositivo es `temperature-controller-01`, el del componente es `thermostat2` y el de la telemetría `temperature`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/components/thermostat2/telemetry/temperature?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "timestamp": "2021-03-24T12:43:44.968Z",
  "value": 70.29168040339141
}
```

Si el dispositivo es un dispositivo IoT Edge, use la siguiente solicitud para recuperar el último valor de telemetría conocido de un módulo. En este ejemplo se usa un dispositivo de nombre `environmental-sensor-01` con un módulo de nombre `SimulatedTemperatureSensor` y una telemetría de nombre `ambient`. El tipo de telemetría `ambient` tiene valores de temperatura y humedad:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/environmental-sensor-01/modules/SimulatedTemperatureSensor/telemetry/ambient?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "timestamp": "2021-03-25T15:44:34.955Z",
  "value": {
    "temperature": 21.18032378129676,
    "humidity": 25
  }
}
```

> [!TIP]
> Para acceder a la telemetría desde un componente de un módulo, use `/api/devices/{deviceId}/modules/{moduleName}/components/{componentName}/telemetry/{telemetryName}`.

## <a name="read-properties"></a>Lectura de las propiedades

Use la siguiente solicitud para recuperar los valores de propiedad de un dispositivo que no usa componentes. En este ejemplo, el nombre del dispositivo es `thermostat-01`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/thermostat-01/properties?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente. Muestra que el dispositivo comunica un solo valor de propiedad:

```json
{
  "maxTempSinceLastReboot": 93.95907131817654,
  "$metadata": {
    "maxTempSinceLastReboot": {
      "lastUpdateTime": "2021-03-24T12:47:46.7571438Z"
    }
  }
}
```

Use la siguiente solicitud para recuperar los valores de propiedad de todos los componentes. En este ejemplo, el nombre del dispositivo es `temperature-controller-01`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/properties?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "serialNumber": "Explicabo animi nihil qui facere sit explicabo nisi.",
  "$metadata": {
    "serialNumber": {
      "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
    }
  },
  "thermostat1": {
    "maxTempSinceLastReboot": 79.7290121339184,
    "$metadata": {
      "maxTempSinceLastReboot": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      }
    }
  },
  "thermostat2": {
    "maxTempSinceLastReboot": 54.214860556320424,
    "$metadata": {
      "maxTempSinceLastReboot": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      }
    }
  },
  "deviceInformation": {
    "manufacturer": "Eveniet culpa sed sit omnis.",
    "$metadata": {
      "manufacturer": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "model": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "swVersion": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "osName": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "processorArchitecture": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "processorManufacturer": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "totalStorage": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      },
      "totalMemory": {
        "lastUpdateTime": "2021-03-24T13:58:52.5999859Z"
      }
    },
    "model": "Necessitatibus id ab dolores vel eligendi fuga.",
    "swVersion": "Ut minus ipsum ut omnis est asperiores harum.",
    "osName": "Atque sit omnis eum sapiente eum tenetur est dolor.",
    "processorArchitecture": "Ratione enim dolor iste iure.",
    "processorManufacturer": "Aliquam eligendi sit ipsa.",
    "totalStorage": 36.02825898541592,
    "totalMemory": 55.442695395750505
  }
}
```

Use la siguiente solicitud para recuperar un valor de propiedad de un componente individual. En este ejemplo, el nombre del dispositivo es `temperature-controller-01` y el del componente `thermostat2`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/components/thermostat2/properties?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "maxTempSinceLastReboot": 24.445128131004935,
  "$metadata": {
    "maxTempSinceLastReboot": {
      "lastUpdateTime": "2021-03-24T14:03:53.787491Z"
    }
  }
}
```

Si el dispositivo es un dispositivo IoT Edge, use la siguiente solicitud para recuperar los valores de propiedad de un módulo. En este ejemplo se usa un dispositivo de nombre `environmental-sensor-01` con un módulo de nombre `SimulatedTemperatureSensor`:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/environmental-sensor-01/modules/SimulatedTemperatureSensor/properties?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "$metadata": {
    "SendData": {
      "desiredValue": true,
      "desiredVersion": 1
    },
    "SendInterval": {
      "desiredValue": 10,
      "desiredVersion": 1
    }
  }
}
```

> [!TIP]
> Para acceder a las propiedades de un componente de un módulo, use `/devices/{deviceId}/modules/{moduleName}/components/{componentName}/properties`.

## <a name="write-properties"></a>Escritura de propiedades

Algunas propiedades son editables. Por ejemplo, en el modelo de termostato, la propiedad `targetTemperature` es una propiedad editable.

Use la siguiente solicitud para escribir un valor de propiedad individual en un dispositivo que no usa componentes. En este ejemplo, el nombre del dispositivo es `thermostat-01`:

```http
PATCH https://{your app subdomain}.azureiotcentral.com/api/devices/thermostat-01/properties?api-version=1.0
```

El cuerpo de la solicitud se parece al siguiente ejemplo:

```json
{
  "targetTemperature": 65.5
}
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "$metadata": {
    "targetTemperature": {
      "desiredValue": 65.5
    }
  }
}
```

> [!TIP]
> Para actualizar todas las propiedades de un dispositivo, use `PUT` en lugar de `PATCH`.

Use la siguiente solicitud para escribir un valor de propiedad individual en un dispositivo que sí usa componentes. En este ejemplo, el nombre del dispositivo es `temperature-controller-01` y el del componente `thermostat2`:

```http
PATCH https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/components/thermostat2/properties?api-version=1.0
```

El cuerpo de la solicitud se parece al siguiente ejemplo:

```json
{
  "targetTemperature": 65.5
}
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "$metadata": {
    "targetTemperature": {
      "desiredValue": 65.5
    }
  }
}
```

> [!TIP]
> Para actualizar todas las propiedades de un componente, use `PUT` en lugar de `PATCH`.

Si el dispositivo es un dispositivo IoT Edge, use la siguiente solicitud para escribir un valor de propiedad individual en un módulo. En este ejemplo se usan un dispositivo de nombre `environmental-sensor-01`, un módulo de nombre `SimulatedTemperatureSensor` y una propiedad de nombre `SendInterval`:

```http
PUT https://{your app subdomain}.azureiotcentral.com/api/devices/environmental-sensor-01/modules/SimulatedTemperatureSensor/properties?api-version=1.0
```

El cuerpo de la solicitud se parece al siguiente ejemplo:

```json
{
  "SendInterval": 20
}
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "$metadata": {
    "SendInterval": {
      "desiredValue": 20
    }
  }
}
```

> [!TIP]
> Para actualizar todas las propiedades de un módulo, use `PUT` en lugar de `PATCH`.

### <a name="update-module-properties"></a>Actualización de las propiedades del módulo

Si está utilizando un dispositivo IoT Edge, utilice la siguiente solicitud para recuperar los valores de propiedad de un módulo:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/{deviceId}/modules/{moduleName}/properties?api-version=1.0
```

Si está utilizando un dispositivo IoT Edge, use la siguiente solicitud para recuperar los valores de propiedad de un componente en un módulo:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/{deviceId}/modules/{moduleName}/components/{componentName}/properties?api-version=1.0
```

## <a name="call-commands"></a>Comandos de llamada

Puede usar la API de REST para llamar a comandos de un dispositivo y recuperar el historial de este.

Use la siguiente solicitud para llamar a un comando de un dispositivo que no usa componentes. En este ejemplo, el nombre del dispositivo es `thermostat-01` y el del comando `getMaxMinReport`:

```http
POST https://{your app subdomain}.azureiotcentral.com/api/devices/thermostat-01/commands/getMaxMinReport?api-version=1.0
```

El cuerpo de la solicitud se parece al siguiente ejemplo:

```json
{
  "since": "2021-03-24T12:55:20.789Z"
}
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "response": {
    "maxTemp": 21.002000799562367,
    "minTemp": 73.09674605264892,
    "avgTemp": 59.54553991653756,
    "startTime": "2022-02-28T15:02:56.789Z",
    "endTime": "2021-05-05T03:50:56.412Z"
  },
  "responseCode": 200
}
```

Para ver el historial de este comando, use la siguiente solicitud:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/thermostat-01/commands/getMaxMinReport?api-version=1.0
```

La respuesta a esta solicitud es similar al ejemplo siguiente:

```json
{
  "value": [
    {
      "response": {
        "maxTemp": 71.43744908819954,
        "minTemp": 51.29986610160005,
        "avgTemp": 39.577384387771744,
        "startTime": "2021-06-20T00:38:17.620Z",
        "endTime": "2022-01-07T22:30:41.104Z"
      },
      "responseCode": 200
    }
  ]
}
```

Use la siguiente solicitud para llamar a un comando de un dispositivo que sí usa componentes. En este ejemplo, el nombre del dispositivo es `temperature-controller-01`, el del componente es `thermostat2` y el del comando `getMaxMinReport`:

```http
POST https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/components/thermostat2/commands/getMaxMinReport?api-version=1.0
```

Los formatos de la carga de la solicitud y la respuesta son iguales que para un dispositivo que no usa componentes.

Para ver el historial de este comando, use la siguiente solicitud:

```http
GET https://{your app subdomain}.azureiotcentral.com/api/devices/temperature-controller-01/components/thermostat2/commands/getMaxMinReport?api-version=1.0
```

> [!TIP]
> Para llamar a comandos de un componente de un módulo, use `/devices/{deviceId}/modules/{moduleName}/components/{componentName}/commands/{commandName}`.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a controlar dispositivos con la API de REST, un próximo paso sugerido es la [Administración de aplicaciones de IoT Central con la API de REST](/learn/modules/manage-iot-central-apps-with-rest-api/).
