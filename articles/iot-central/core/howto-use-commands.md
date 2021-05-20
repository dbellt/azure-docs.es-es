---
title: Uso de los comandos de dispositivo en una solución de Azure IoT Central
description: Uso de los comandos de dispositivo en una solución de Azure IoT Central. En este tutorial se muestra cómo usar los comandos de dispositivo en la aplicación cliente para la aplicación de Azure IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 01/07/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: a0a7b24c2d28e27c44a130383e852838950abe49
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108733200"
---
# <a name="how-to-use-commands-in-an-azure-iot-central-solution"></a>Uso de los comandos en una solución de Azure IoT Central

En esta guía paso a paso se muestra cómo usar los comandos que se definen en una plantilla de dispositivo.

Un operador puede usar la interfaz de usuario de IoT Central para llamar a un comando para un dispositivo. Los comandos controlan el comportamiento de un dispositivo. Por ejemplo, un operador puede llamar a un comando para reiniciar un dispositivo o recopilar datos de diagnóstico.

Un dispositivo puede:

* Responder inmediatamente a un comando.
* Responder a IoT Central cuando reciba el comando y después notificar a IoT Central cuando se complete el *comandos de larga duración*.

De manera predeterminada, los comandos esperan que un dispositivo se conecte y generan un error si no pueden comunicarse con el dispositivo. Si selecciona la opción **Queue if offline** (poner en cola si no hay conexión) en la interfaz de usuario de la plantilla de dispositivo, se puede poner en cola un comando hasta que un dispositivo se conecte. Estos *comandos sin conexión* se describen en una sección independiente más adelante en este artículo.

## <a name="define-your-commands"></a>Definición de los comandos

Los comandos estándar se envían a un dispositivo para indicar al mismo que haga algo. Un comando puede incluir parámetros con información adicional. Por ejemplo, un comando para abrir una válvula en un dispositivo podría incluir un parámetro que especifique cuánto se debe abrir la válvula. Los comandos también pueden recibir un valor devuelto cuando el dispositivo completa el comando. Por ejemplo, un comando que solicita a un dispositivo que ejecute algunos diagnósticos podría recibir un informe de diagnóstico como valor devuelto.

Los comandos se definen como parte de una plantilla de dispositivo. En la captura de pantalla siguiente se muestra la definición del comando **Get Max-Min report** en la plantilla de dispositivo **Thermostat**. Este comando tiene parámetros de solicitud y de respuesta: 

:::image type="content" source="media/howto-use-commands/command-definition.png" alt-text="Captura de pantalla que muestra el comando Get Max Min Report en la plantilla de dispositivo Thermostat":::

En la tabla siguiente se muestran las opciones de configuración de una funcionalidad de comando:

| Campo             |Descripción|
|-------------------|-----------|
|Display Name (Nombre para mostrar)       |Valor de comando usado en los paneles y formularios.|
| Nombre            | El nombre del comando. IoT Central genera un valor para este campo a partir del nombre para mostrar, pero puede elegir su propio valor si es necesario. Este campo debe ser alfanumérico. El código del dispositivo usa este valor de **Name**.|
| Capability Type (Tipo de funcionalidad) | Comando.|
| Queue if offline (poner en cola si no hay conexión) | Indica si se va a convertir este comando en uno *sin conexión*. |
| Descripción     | Una descripción de la funcionalidad del comando:|
| Comentario     | Cualquier comentario sobre la funcionalidad del comando.|
| Solicitud     | Carga útil del comando de dispositivo.|
| Response     | Carga útil de la respuesta del comando de dispositivo.|

En el fragmento de código siguiente se muestra la representación JSON del comando en el modelo de dispositivo. En este ejemplo, el valor de la respuesta es de tipo **Object** complejo con varios campos:

```json
{
  "@type": "Command",
  "name": "getMaxMinReport",
  "displayName": "Get Max-Min report.",
  "description": "This command returns the max, min and average temperature from the specified time to the current time.",
  "request": {
    "name": "since",
    "displayName": "Since",
    "description": "Period to return the max-min report.",
    "schema": "dateTime"
  },
  "response": {
    "name" : "tempReport",
    "displayName": "Temperature Report",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "maxTemp",
          "displayName": "Max temperature",
          "schema": "double"
        },
        {
          "name": "minTemp",
          "displayName": "Min temperature",
          "schema": "double"
        },
        {
          "name" : "avgTemp",
          "displayName": "Average Temperature",
          "schema": "double"
        },
        {
          "name" : "startTime",
          "displayName": "Start Time",
          "schema": "dateTime"
        },
        {
          "name" : "endTime",
          "displayName": "End Time",
          "schema": "dateTime"
        }
      ]
    }
  }
}
```

> [!TIP]
> Puede exportar un modelo de dispositivo desde la página de la plantilla de dispositivo.

Puede relacionar esta definición de comando con la captura de pantalla de la interfaz de usuario mediante los siguientes campos:

* `@type` para especificar el tipo de funcionalidad: `Command`.
* `name` para el valor del comando.

Los campos opcionales, como el nombre para mostrar y la descripción, permiten agregar más detalles a la interfaz y las funcionalidades.

## <a name="standard-commands"></a>Comandos estándar

En esta sección se muestra cómo un dispositivo envía un valor de respuesta en cuanto recibe el comando.

En el siguiente fragmento de código se muestra cómo un dispositivo puede responder a un comando que envía inmediatamente un código de correcto:

> [!NOTE]
> En este artículo se usa Node.js por motivos de simplicidad. Para ver ejemplos de otros lenguajes, consulte el tutorial [Creación y conexión de una aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md).

```javascript
client.onDeviceMethod('getMaxMinReport', commandHandler);

// ...

const commandHandler = async (request, response) => {
  switch (request.methodName) {
  case 'getMaxMinReport': {
    console.log('MaxMinReport ' + request.payload);
    await sendCommandResponse(request, response, 200, deviceTemperatureSensor.getMaxMinReportObject());
    break;
  }
  default:
    await sendCommandResponse(request, response, 404, 'unknown method');
    break;
  }
};

const sendCommandResponse = async (request, response, status, payload) => {
  try {
    await response.send(status, payload);
    console.log('Response to method \'' + request.methodName +
              '\' sent successfully.' );
  } catch (err) {
    console.error('An error ocurred when sending a method response:\n' +
              err.toString());
  }
};
```

La llamada a `onDeviceMethod` configura el método `commandHandler`. Este controlador de comandos:

1. Comprueba el nombre del comando.
1. En el caso del comando `getMaxMinReport`, llama a `getMaxMinReportObject` para recuperar los valores que se incluirán en el objeto devuelto.
1. Llama a `sendCommandResponse` para devolver la respuesta a IoT Central. Esta respuesta incluye el código de respuesta `200` para indicar que la operación se realizó correctamente.

En la siguiente captura de pantalla se muestra el aspecto de la respuesta de correcto del comando en la interfaz de usuario de IoT Central:

:::image type="content" source="media/howto-use-commands/simple-command-ui.png" alt-text="Captura de pantalla que muestra cómo ver la carga útil de comando de un comando estándar":::

## <a name="long-running-commands"></a>Comandos de ejecución prolongada

En esta sección se muestra cómo un dispositivo puede retrasar el envío de la confirmación de que el comando se completó.

En el siguiente fragmento de código se muestra cómo un dispositivo puede implementar un comando de larga duración:

> [!NOTE]
> En este artículo se usa Node.js por motivos de simplicidad. Para ver ejemplos de otros lenguajes, consulte el tutorial [Creación y conexión de una aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md).

```javascript
client.onDeviceMethod('rundiagnostics', commandHandler);

// ...

const commandHandler = async (request, response) => {
  switch (request.methodName) {
  case 'rundiagnostics': {
    console.log('Starting long-running diagnostics run ' + request.payload);
    await sendCommandResponse(request, response, 202, 'Diagnostics run started');

    // Long-running operation here
    // ...

    const patch = {
      rundiagnostics: {
        value: 'Diagnostics run complete at ' + new Date().toLocaleString()
      }
    };

    deviceTwin.properties.reported.update(patch, function (err) {
      if (err) throw err;
      console.log('Properties have been reported for component');
    });
    break;
  }
  default:
    await sendCommandResponse(request, response, 404, 'unknown method');
    break;
  }
};
```

La llamada a `onDeviceMethod` configura el método `commandHandler`. Este controlador de comandos:

1. Comprueba el nombre del comando.
1. Llama a `sendCommandResponse` para devolver la respuesta a IoT Central. Esta respuesta incluye el código de respuesta `202` para indicar que hay resultados pendientes.
1. Completa la operación de ejecución prolongada.
1. Usa una propiedad notificada con el mismo nombre que el comando para indicar a IoT Central que el comando se completó.

En la siguiente captura de pantalla se muestra el aspecto de la respuesta del comando en la interfaz de usuario de IoT Central cuando recibe el código de respuesta 202:

:::image type="content" source="media/howto-use-commands/long-running-start.png" alt-text="Captura de pantalla que muestra la respuesta inmediata del dispositivo":::

En la captura de pantalla siguiente se muestra la interfaz de usuario de IoT Central cuando recibe la actualización de propiedad que indica que el comando se completó:

:::image type="content" source="media/howto-use-commands/long-running-finish.png" alt-text="Captura de pantalla que muestra el comando de larga duración finalizado":::

## <a name="offline-commands"></a>Comandos sin conexión

En esta sección se muestra cómo un dispositivo administra un comando sin conexión. Si un dispositivo está en línea, puede controlar el comando sin conexión en cuanto lo recibe. Si un dispositivo está sin conexión, se encarga del comando sin conexión cuando vuelve a conectarse a IoT Central. Los dispositivos no pueden enviar un valor devuelto en respuesta a un comando sin conexión.

> [!NOTE]
> En este artículo se usa Node.js por motivos de simplicidad.

En la captura de pantalla siguiente se muestra un comando sin conexión denominado **GenerateDiagnostics**. El parámetro de solicitud es un objeto con una propiedad datetime denominada **StartTime** y una propiedad de enumeración de enteros denominada **Bank**:

:::image type="content" source="media/howto-use-commands/offline-command.png" alt-text="Captura de pantalla que muestra la interfaz de usuario de un comando sin conexión":::

En el siguiente fragmento de código se muestra cómo un cliente puede escuchar comandos sin conexión y mostrar el contenido del mensaje:

```javascript
client.on('message', function (msg) {
  console.log('Body: ' + msg.data);
  console.log('Properties: ' + JSON.stringify(msg.properties));
  client.complete(msg, function (err) {
    if (err) {
      console.error('complete error: ' + err.toString());
    } else {
      console.log('complete sent');
    }
  });
});
```

La salida del fragmento de código anterior muestra la carga útil con los valores **StartTime** y **Bank**. La lista de propiedades incluye el nombre del comando en el elemento **method-name** de la lista:

```output
Body: {"StartTime":"2021-01-06T06:00:00.000Z","Bank":2}
Properties: {"propertyList":[{"key":"iothub-ack","value":"none"},{"key":"method-name","value":"GenerateDiagnostics"}]}
```

> [!NOTE]
> El período de vida predeterminado para los comandos sin conexión es de 24 horas, después del cual expira el mensaje.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar comandos en su aplicación de Azure IoT Central, consulte [Cargas](concepts-telemetry-properties-commands.md) para obtener más información sobre los parámetros de comando y [Creación y conexión de un aplicación cliente a la aplicación de Azure IoT Central](tutorial-connect-device.md) para ver ejemplos de código completos en distintos lenguajes.
