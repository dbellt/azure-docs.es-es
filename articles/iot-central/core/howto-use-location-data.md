---
title: Uso de los datos de ubicación en una solución de Azure IoT Central
description: Obtenga información sobre cómo usar los datos de ubicación enviados de un dispositivo conectado a la aplicación de IoT Central. Trace los datos de ubicación en un mapa o cree reglas de geovalla.
author: dominicbetts
ms.author: dobett
ms.date: 01/08/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: e93da2bec3753ba6efda2b4eb16d6ef78b42234b
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747222"
---
# <a name="use-location-data-in-an-azure-iot-central-solution"></a>Uso de los datos de ubicación en una solución de Azure IoT Central

En este artículo se muestra cómo usar los datos de ubicación en una aplicación de IoT Central. Un dispositivo conectado a IoT Central puede enviar datos de ubicación como flujo de telemetría o usar una propiedad de dispositivo para informar sobre los datos de ubicación.

Puede usar los datos de ubicación para:

* Trazar la ubicación indicada en un mapa.
* Trazar el historial de ubicaciones de telemetría en un mapa.
* Crear reglas de geovalla para notificar a un operador cuando un dispositivo entra o sale de un área específica.

## <a name="add-location-capabilities-to-a-device-template"></a>Incorporación de capacidades de ubicación a una plantilla de dispositivo

En la captura de pantalla siguiente se muestra una plantilla de dispositivo con ejemplos de una propiedad de dispositivo y un tipo de telemetría que usan datos de ubicación. Las definiciones usan el tipo semántico **location** y el tipo de esquema **geolocation**:

:::image type="content" source="media/howto-use-location-data/location-device-template.png" alt-text="Captura de pantalla que muestra la definición de la propiedad location en la plantilla de dispositivo":::

Como referencia, las definiciones del [lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) para estas funciones son similares al siguiente fragmento de código:

```json
{
  "@type": [
    "Property",
    "Location"
  ],
  "displayName": {
    "en": "DeviceLocation"
  },
  "name": "DeviceLocation",
  "schema": "geopoint",
  "writable": false
},
{
  "@type": [
    "Telemetry",
    "Location"
  ],
  "displayName": {
    "en": "Tracking"
  },
  "name": "Tracking",
  "schema": "geopoint"
}
```

> [!NOTE]
> El tipo de esquema **geopoint** no es parte de la [especificación de DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). Actualmente, IoT Central admite el tipo de esquema **geopoint** y el tipo semántico **location** para la compatibilidad con versiones anteriores.

## <a name="send-location-data-from-a-device"></a>Envío de datos de ubicación desde un dispositivo

Cuando un dispositivo envía datos para la propiedad **DeviceLocation** que se muestra en la sección anterior, la carga útil es similar al siguiente fragmento de código JSON:

```json
{
  "DeviceLocation": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

Cuando un dispositivo envía datos para la telemetría **Tracking** que se muestra en la sección anterior, la carga útil es similar al siguiente fragmento de código JSON:

```json
{
  "Tracking": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

## <a name="display-device-location"></a>Visualización de la ubicación del dispositivo

Puede mostrar los datos de ubicación en varios lugares de la aplicación de IoT Central. Por ejemplo, en las vistas asociadas a un dispositivo individual o a los paneles.

Al crear una vista para un dispositivo, puede optar por trazar la ubicación en un mapa o mostrar los valores individuales:

:::image type="content" source="media/howto-use-location-data/location-views.png" alt-text="Captura de pantalla que muestra la vista de ejemplo con datos de ubicación":::

Puede agregar mosaicos de mapa a un panel para trazar la ubicación de uno o más dispositivos. Cuando se agrega un mosaico de mapa para mostrar la telemetría de ubicación, se puede trazar la ubicación en un periodo de tiempo. En la siguiente captura de pantalla se muestra la ubicación indicada por un dispositivo simulado durante los últimos 30 minutos:

:::image type="content" source="media/howto-use-location-data/location-dashboard.png" alt-text="Captura de pantalla que muestra el panel de ejemplo con datos de ubicación":::

## <a name="create-a-geofencing-rule"></a>Creación de una regla de geovalla

Puede usar la telemetría de ubicación para crear una regla de geovalla que genera una alerta cuando un dispositivo se mueve dentro o fuera de un área rectangular. En la siguiente captura de pantalla se muestra una regla que usa cuatro condiciones para definir un área rectangular con valores de latitud y longitud. La regla genera un correo electrónico cuando el dispositivo se mueve al área rectangular:

:::image type="content" source="media/howto-use-location-data/geofence-rule.png" alt-text="Captura de pantalla que muestra una definición de regla de geovalla":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar propiedades en una aplicación de Azure IoT Central, consulte:

* [Cargas de telemetría, propiedades y comandos](concepts-telemetry-properties-commands.md)
* [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md)