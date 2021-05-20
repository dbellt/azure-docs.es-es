---
title: Uso de las propiedades en una solución de Azure IoT Central
description: Obtenga información sobre el uso de las propiedades de solo lectura y escritura en una solución de Azure IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 11/06/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: a605046c8cf76a425f7df5b1cdf637f7dc4bfc90
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747258"
---
# <a name="use-properties-in-an-azure-iot-central-solution"></a>Uso de las propiedades en una solución de Azure IoT Central

En esta guía paso a paso se muestra cómo usar las propiedades del dispositivo que se definen en una plantilla de dispositivo en la aplicación de Azure IoT Central.

Las propiedades representan valores de un momento dado. Por ejemplo, un dispositivo puede usar una propiedad para notificar la temperatura objetivo que está intentando alcanzar. De manera predeterminada, las propiedades del dispositivo son de solo lectura en IoT Central. Las propiedades editables permiten sincronizar el estado entre el dispositivo y la aplicación de Azure IoT Central.

También puede definir las propiedades de la nube en una aplicación de Azure IoT Central. Los valores de propiedad de la nube nunca se intercambian con un dispositivo y están fuera del ámbito de este artículo.

## <a name="define-your-properties"></a>Definición de las propiedades

Las propiedades son campos de datos que representan el estado de un dispositivo. Se usan para representar el estado duradero del dispositivo, como el estado de encendido-apagado de un dispositivo. Las propiedades también pueden representar propiedades básicas del dispositivo, como la versión del software del dispositivo. Las propiedades se declaran como de solo lectura o de escritura.

En la captura de pantalla siguiente se muestra una definición de propiedad en la aplicación de Azure IoT Central.

:::image type="content" source="media/howto-use-properties/property-definition.png" alt-text="Captura de pantalla que muestra una definición de propiedad en la aplicación de Azure IoT Central.":::

En la siguiente tabla se muestran las opciones de configuración de una funcionalidad de propiedad.

| Campo           | Descripción                                                                                                                                                                                                                        |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nombre para mostrar    | El nombre para mostrar del valor de propiedad que se usa en los paneles y formularios.                                                                                                                                                              |
| Nombre            | El nombre de la propiedad. Azure IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico.  El código del dispositivo usa este valor de **Nombre**.           |
| Tipo de funcionalidad | Propiedad.                                                                                                                                                                                                                          |
| Tipo semántico   | El tipo semántico de la propiedad, como la temperatura, el estado o el evento. La elección del tipo semántico determina cuál de los campos siguientes está disponible.                                                                       |
| Schema          | El tipo de datos de la propiedad, como doble, cadena o vector. Las opciones disponibles vienen determinadas por el tipo semántico. El esquema no está disponible para los tipos semánticos de evento y estado.                                               |
| Editable       | Si la propiedad no se puede escribir, el dispositivo puede notificar los valores de propiedad a Azure IoT Central. Si la propiedad se puede escribir, el dispositivo puede notificar los valores de propiedad a Azure IoT Central. Después, Azure IoT Central puede enviar actualizaciones de propiedades al dispositivo. |
| severity        | Solo está disponible para el tipo semántico de evento. Los niveles de gravedad son **Error**, **Información** o **Advertencia**.                                                                                                                         |
| Valores de estado    | Solo está disponible para el tipo semántico de estado. Defina los valores de estado posibles, cada uno de los cuales tiene el nombre para mostrar, el nombre, el tipo de enumeración y el valor.                                                                                   |
| Unidad            | Una unidad para el valor de propiedad, como **mph**, **%** o **&deg;C**.                                                                                                                                                              |
| Unidad de visualización    | Una unidad de visualización para su uso en paneles y formularios.                                                                                                                                                                                    |
| Comentario         | Cualquier comentario sobre la funcionalidad de propiedad.                                                                                                                                                                                        |
| Descripción     | Una descripción de la funcionalidad de propiedad.                                                                                                                                                                                          |

Las propiedades también se pueden definir en una interfaz en una plantilla de dispositivo como se muestra a continuación:

``` json
{
  "@type": [
    "Property",
    "Temperature"
  ],
  "name": "targetTemperature",
  "schema": "double",
  "displayName": "Target Temperature",
  "description": "Allows to remotely specify the desired target temperature.",
  "unit" : "degreeCelsius",
  "writable": true
},
{
  "@type": [
    "Property",
    "Temperature"
  ],
  "name": "maxTempSinceLastReboot",
  "schema": "double",
  "unit" : "degreeCelsius",
  "displayName": "Max temperature since last reboot.",
  "description": "Returns the max temperature since last device reboot."
}
```

En este ejemplo se muestran dos propiedades. Estas se relacionan con la definición de la propiedad en la interfaz de usuario:

* `@type` especifica el tipo de funcionalidad: `Property`. En el ejemplo anterior también se muestra el tipo semántico `Temperature` para ambas propiedades.
* `name` para la propiedad.
* `schema` especifica el tipo de datos de la propiedad. Este valor puede ser un tipo primitivo, como doble, entero, booleano o cadena. También se admiten tipos de objetos complejos y mapas.
* `writable` De forma predeterminada, las propiedades son de solo lectura. Puede marcar una propiedad como editable utilizando este campo.

Los campos opcionales, como el nombre para mostrar y la descripción, permiten agregar más detalles a la interfaz y las funcionalidades.

Al crear una propiedad, puede especificar tipos de esquema complejos, como **Object** y **Enum**.

![Captura de pantalla que muestra cómo agregar una capacidad.](./media/howto-use-properties/property.png)

Al seleccionar el **esquema** complejo, como **Object**, deberá definir también el objeto.

:::image type="content" source="media/howto-use-properties/object.png" alt-text="Captura de pantalla que muestra cómo definir un objeto":::

El siguiente código muestra la definición de un tipo de propiedad Object. Este objeto tiene dos campos con los tipos cadena y entero.

``` json
{
  "@type": "Property",
  "displayName": {
    "en": "ObjectProperty"
  },
  "name": "ObjectProperty",
  "schema": {
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "displayName": {
          "en": "Field1"
        },
        "name": "Field1",
        "schema": "integer"
      },
      {
        "displayName": {
          "en": "Field2"
        },
        "name": "Field2",
        "schema": "string"
      }
    ]
  }
}
```

## <a name="implement-read-only-properties"></a>Implementación de propiedades de solo lectura

De forma predeterminada, las propiedades son de solo lectura. Las propiedades de solo lectura permiten a un dispositivo informar de las actualizaciones de los valores de las propiedades a la aplicación de Azure IoT Central. La aplicación de Azure IoT Central no puede establecer el valor de una propiedad de solo lectura.

Azure IoT Central usa dispositivos gemelos para sincronizar valores de propiedad entre el dispositivo y la aplicación de Azure IoT Central. Los valores de propiedades del dispositivo usan propiedades notificadas del dispositivo gemelo. Para más información, consulte los detalles sobre [dispositivos gemelos](../../iot-hub/tutorial-device-twins.md).

El siguiente fragmento de código de un modelo de dispositivo muestra la definición de un tipo de propiedad de solo lectura:

``` json
{
  "name": "model",
  "displayName": "Device model",
  "schema": "string",
  "comment": "Device model name or ID. Ex. Surface Book 2."
}
```

Un dispositivo envía las actualizaciones de propiedades como carga JSON. Para más información, consulte [Cargas de telemetría, propiedades y comandos](./concepts-telemetry-properties-commands.md).

Puede usar el SDK de dispositivo IoT de Azure para enviar una actualización de propiedad a la aplicación de Azure IoT Central.

Las propiedades de dispositivo gemelo se pueden enviar a la aplicación de Azure IoT Central mediante la siguiente función:

``` javascript
hubClient.getTwin((err, twin) => {
    const properties = {
        model: 'environmentalSensor1.0'
    };
    twin.properties.reported.update(properties, (err) => {
        console.log(err ? `error: ${err.toString()}` : `status: success` )
    });
});
```

En este artículo se usa Node.js por motivos de simplicidad. Para ver ejemplos de otros lenguajes, consulte el tutorial [Creación y conexión de una aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md).

En la siguiente vista de la aplicación de Azure IoT Central se muestran las propiedades que puede ver. La vista hace que la propiedad **modelo del dispositivo** pase a ser _propiedad de dispositivo de solo lectura_.

:::image type="content" source="media/howto-use-properties/read-only.png" alt-text="Captura de pantalla que muestra la vista de una propiedad de solo lectura":::

## <a name="implement-writable-properties"></a>Implementación de propiedades editables

Los operadores establecen las propiedades editables en la aplicación de Azure IoT Central en un formulario. Azure IoT Central envía la propiedad al dispositivo. Azure IoT Central espera una confirmación del dispositivo.

En el siguiente fragmento de código de un modelo de dispositivo se muestra la definición de un tipo de propiedad editable:

``` json
{
  "@type": "Property",
  "displayName": "Brightness Level",
  "description": "The brightness level for the light on the device. Can be specified as 1 (high), 2 (medium), 3 (low)",
  "name": "brightness",
  "writable": true,
  "schema": "long"
}
```

Para definir y controlar las propiedades editables a las que responde el dispositivo, puede usar el código siguiente:

``` javascript
hubClient.getTwin((err, twin) => {
    twin.on('properties.desired.brightness', function(desiredBrightness) {
        console.log( `Received setting: ${desiredBrightness.value}` );
        var patch = {
            brightness: {
                value: desiredBrightness.value,
                ad: 'success',
                ac: 200,
                av: desiredBrightness.$version
            }
        }
        twin.properties.reported.update(patch, (err) => {
            console.log(err ? `error: ${err.toString()}` : `status: success` )
        });
    });
});
```

El mensaje de respuesta debe incluir los campos `ac` y `av`. El campo `ad` es opcional. Consulte los fragmentos de código siguientes para ver ejemplos:

* `ac` es un campo numérico que usa los valores de la tabla siguiente.
* `av` es el número de versión enviado al dispositivo.
* `ad` es una descripción de cadena de opción.

| Value | Etiqueta | Descripción |
| ----- | ----- | ----------- |
| `'ac': 200` | Completed | La operación de cambio de propiedad se ha completado correctamente. |
| `'ac': 202` o `'ac': 201` | Pending | La operación de cambio de propiedad está pendiente o en curso. |
| `'ac': 4xx` | Error | El cambio de propiedad solicitado no era válido o tenía un error. |
| `'ac': 5xx` | Error | El dispositivo experimentó un error inesperado al procesar el cambio solicitado. |

Para obtener más información sobre los dispositivos gemelos, consulte [Configuración de dispositivos desde un servicio back-end](../../iot-hub/tutorial-device-twins.md).

Cuando el operador establece una propiedad editable en la aplicación de Azure IoT Central, la aplicación utiliza la propiedad especificada en el dispositivo gemelo para enviar el valor al dispositivo. A continuación, el dispositivo responde con una propiedad notificada del dispositivo gemelo. Cuando Azure IoT Central recibe el valor de la propiedad notificada, actualiza la vista de la propiedad con el estado **Aceptada**.

En la vista siguiente se muestran las propiedades editables. Al escribir el valor y seleccionar **Guardar**, el estado inicial es **Pendiente**. Cuando el dispositivo acepta el cambio, el estado cambia a **Aceptado**.

![Captura de pantalla que muestra el estado Pendiente.](./media/howto-use-properties/status-pending.png)

![Captura de pantalla que muestra la propiedad Aceptado.](./media/howto-use-properties/accepted.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar propiedades en una aplicación de Azure IoT Central, consulte:

* [Cargas de telemetría, propiedades y comandos](concepts-telemetry-properties-commands.md)
* [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md)