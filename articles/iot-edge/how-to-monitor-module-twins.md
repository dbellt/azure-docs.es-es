---
title: 'Supervisión de módulos gemelos: Azure IoT Edge'
description: Interpretación de dispositivos y módulos gemelos para determinar la conectividad y su estado.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 05/29/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: a5a31e15c88cef588c93f44c8fe5303d930b5b2c
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479379"
---
# <a name="monitor-module-twins"></a>Supervisión de módulos gemelos

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Los módulos gemelos de Azure IoT Hub permiten supervisar la conectividad y el estado de las implementaciones de IoT Edge. Los módulos gemelos almacenan información útil en su centro de IoT sobre el rendimiento de los módulos en ejecución. Los módulos del entorno de ejecución [Agente de IoT Edge](iot-edge-runtime.md#iot-edge-agent) y [Centro de IoT Edge](iot-edge-runtime.md#iot-edge-hub) mantienen cada uno sus módulos gemelos, `$edgeAgent` y `$edgeHub`, respectivamente:

* `$edgeAgent` contiene datos de estado y conectividad de los módulos Agente de IoT Edge y Centro de IoT Edge, y de los módulos personalizados. El agente de IoT Edge es responsable de implementar los módulos, supervisarlos y enviar notificaciones sobre el estado de la conexión a Azure IoT Hub.
* `$edgeHub` contiene datos sobre las comunicaciones entre el centro de IoT Edge que se ejecuta en un dispositivo y Azure IoT Hub. Esto incluye el procesamiento de mensajes entrantes desde dispositivos de nivel inferior. El centro de IoT Edge es responsable del procesamiento de las comunicaciones entre Azure IoT Hub y los dispositivos y módulos de IoT Edge.

Los datos se organizan en metadatos y etiquetas, junto con los conjuntos de propiedades notificadas y deseadas en las estructuras JSON de los módulos gemelos. Las propiedades deseadas que especificó en el archivo deployment.json se copian en los módulos gemelos. El agente de IoT Edge y el centro de IoT Edge actualizan las propiedades notificadas de los módulos.

Del mismo modo, las propiedades deseadas especificadas para los módulos personalizados en el archivo deployment.json se copian en su módulo gemelo, pero la solución es responsable de proporcionar los valores de sus propiedades notificadas.

En este artículo se describe cómo revisar los módulos gemelos en Azure Portal, la CLI de Azure y en Visual Studio Code. Para más información sobre cómo supervisar el modo en que los dispositivos reciben las implementaciones, consulte [Supervisión de las implementaciones de IoT Edge](how-to-monitor-iot-edge-deployments.md). Para obtener información general sobre el concepto de módulos gemelos, consulte [Uso de módulos gemelos en IoT Hub](../iot-hub/iot-hub-devguide-module-twins.md).

> [!TIP]
> Las propiedades notificadas del entorno de ejecución podrían quedarse obsoletas si un dispositivo IoT Edge se desconecta de su centro de IoT. Puede hacer [ping](how-to-edgeagent-direct-method.md#ping) en el módulo `$edgeAgent` para determinar si se ha perdido la conexión.

## <a name="monitor-runtime-module-twins"></a>Supervisión de los módulos gemelos del entorno de ejecución

Para solucionar los problemas de conectividad de la implementación, revise los módulos gemelos de Agente de IoT Edge y Centro de IoT Edge y, después, explore en profundidad otros módulos.

### <a name="monitor-iot-edge-agent-module-twin"></a>Supervisión del módulo gemelo de Agente de IoT Edge

El siguiente JSON muestra el módulo gemelo `$edgeAgent` en Visual Studio Code con la mayoría de las secciones JSON contraídas.

```json
{
  "deviceId": "Windows109",
  "moduleId": "$edgeAgent",
  "etag": "AAAAAAAAAAU=",
  "deviceEtag": "NzgwNjA1MDUz",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00Z",
  "connectionState": "Disconnected",
  "lastActivityTime": "0001-01-01T00:00:00Z",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "version": 53,
  "properties": {
    "desired": { "···" },
    "reported": {
      "schemaVersion": "1.0",
      "version": { "···" },
      "lastDesiredStatus": { "···" },
      "runtime": { "···" },
      "systemModules": {
        "edgeAgent": { "···" },
        "edgeHub": { "···" }
      },
      "lastDesiredVersion": 5,
      "modules": {
        "SimulatedTemperatureSensor": { "···" }
      },
      "$metadata": { "···" },
      "$version": 48
    }
  }
}
```

El JSON se puede describir en las secciones siguientes, empezando por la parte superior:

* Metadata: contiene los datos de conectividad. Curiosamente, el estado de conexión del agente de IoT Edge aparece siempre como desconectado: `"connectionState": "Disconnected"`. El motivo es que el estado de conexión pertenece a los mensajes del dispositivo a la nube (D2C) y el agente de IoT Edge no envía mensajes de este tipo.
* Properties: contiene las subsecciones `desired` y `reported`.
* Properties.desired: (se muestra contraído) los valores de propiedad esperados que establece el operador en el archivo deployment.json.
* Properties.reported: los valores de propiedad más recientes notificados por el agente de IoT Edge.

Las secciones `properties.desired` y `properties.reported` tienen una estructura similar y contienen metadatos adicionales con la información del esquema, la versión y el entorno de ejecución. También se incluye la sección `modules` para todos los módulos personalizados (como, por ejemplo, `SimulatedTemperatureSensor`) y la sección `systemModules` para los módulos del entorno de ejecución `$edgeAgent` y `$edgeHub`.

Mediante la comparación de los valores de las propiedades notificadas con los valores deseados, puede determinar las discrepancias e identificar las desconexiones que pueden ayudarle a solucionar los problemas. Al hacer estas comparaciones, compruebe el valor notificado `$lastUpdated` en la sección `metadata` de la propiedad que está investigando.

Es importante examinar las siguientes propiedades durante la solución de problemas:

* **exitcode**: cualquier valor distinto de cero indica que el módulo se detuvo con un error. Sin embargo, se usarán los códigos de error 137 o 143 si un módulo se estableció intencionadamente en un estado detenido.

* **lastStartTimeUtc**: muestra la **fecha y hora** en que se inició por última vez el contenedor. Este valor será 0001-01-01T00:00:00Z si el contenedor no se ha iniciado.

* **lastExitTimeUtc**: muestra la **fecha y hora** en la que finalizó el contenedor por última vez. Este valor será 0001-01-01T00:00:00Z si el contenedor se está ejecutando y nunca se detuvo.

* **runtimeStatus**: puede ser uno de los siguientes valores:

    | Value | Descripción |
    | --- | --- |
    | unknown | Estado predeterminado hasta que se crea la implementación. |
    | backoff | El módulo está programado para iniciarse pero no está en ejecución actualmente. Este valor es útil para un módulo que experimenta cambios de estado en el reinicio. Cuando un módulo con errores está esperando el reinicio durante el período de enfriamiento, el módulo se encuentra en un estado de interrupción. |
    | en ejecución | Indica que el módulo está actualmente en ejecución. |
    | unhealthy | Indica un error en la comprobación del sondeo de estado o que se ha agotado el tiempo de espera. |
    | stopped | Indica que el módulo salió correctamente (con un código de salida cero). |
    | con errores | Indica que el módulo salió con un código de salida de error (distinto de cero). El módulo puede volver a pasar al estado de interrupción desde este estado, según la directiva de reinicio en vigor. Este estado puede indicar que se ha producido un error irrecuperable en el módulo. El error se produce cuando Microsoft Monitoring Agent ya no puede recuperar el módulo y será necesaria una nueva implementación. |

Consulte [Propiedades notificadas de EdgeAgent](module-edgeagent-edgehub.md#edgeagent-reported-properties) para más información.

### <a name="monitor-iot-edge-hub-module-twin"></a>Supervisión del módulo gemelo de Centro de IoT Edge

El siguiente JSON muestra el módulo gemelo `$edgeHub` en Visual Studio Code con la mayoría de las secciones JSON contraídas.

```json
{
  "deviceId": "Windows109",
  "moduleId": "$edgeHub",
  "etag": "AAAAAAAAAAU=",
  "deviceEtag": "NzgwNjA1MDU2",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00Z",
  "connectionState": "Connected",
  "lastActivityTime": "0001-01-01T00:00:00Z",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "version": 102,
    "properties": {
      "desired": { "···" },
      "reported": {
        "schemaVersion": "1.0",
        "version": { "···" },
      "lastDesiredVersion": 5,
      "lastDesiredStatus": { "···" },
      "clients": {
        "Windows109/SimulatedTemperatureSensor": {
          "status": "Disconnected",
          "lastConnectedTimeUtc": "2020-04-08T21:42:42.1743956Z",
          "lastDisconnectedTimeUtc": "2020-04-09T07:02:42.1398325Z"
        }
      },
      "$metadata": { "···" },
      "$version": 97
    }
  }
}

```

El JSON se puede describir en las secciones siguientes, empezando por la parte superior:

* Metadata: contiene los datos de conectividad.

* Properties: contiene las subsecciones `desired` y `reported`.
* Properties.desired: (se muestra contraído) los valores de propiedad esperados que establece el operador en el archivo deployment.json.
* Properties.reported: los valores de propiedad más recientes notificados por el centro de IoT Edge de IoT Edge.

Si tiene problemas con los dispositivos de nivel inferior, el análisis de estos datos sería un buen punto de partida.

## <a name="monitor-custom-module-twins"></a>Supervisión de módulos gemelos personalizados

La información sobre la conectividad de los módulos personalizados se mantiene en el módulo gemelo de Agente de IoT Edge. El módulo gemelo del módulo personalizado se usa principalmente para mantener los datos de la solución. Las propiedades deseadas que definió en el archivo deployment.json se reflejan en el módulo gemelo y el módulo puede actualizar los valores de las propiedades notificadas según sea necesario.

Puede usar el lenguaje de programación que prefiera con los [SDK de dispositivo de Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks) para actualizar los valores de las propiedades notificadas del módulo gemelo, en función del código de aplicación de su módulo. En el procedimiento siguiente se usa el SDK de Azure para .NET con este fin y emplea el código del módulo [SimulatedTemperatureSensor](https://github.com/Azure/iotedge/blob/dd5be125df165783e4e1800f393be18e6a8275a3/edge-modules/SimulatedTemperatureSensor/src/Program.cs):

1. Cree una instancia de [ModuleClient](/dotnet/api/microsoft.azure.devices.client.moduleclient) con el método [CreateFromEnvironmentAysnc](/dotnet/api/microsoft.azure.devices.client.moduleclient.createfromenvironmentasync).

1. Obtenga una recopilación de las propiedades del módulo gemelo con el método [GetTwinAsync](/dotnet/api/microsoft.azure.devices.client.moduleclient.gettwinasync).

1. Cree un agente de escucha (pasando una devolución de llamada) para detectar los cambios en las propiedades deseadas con el método [SetDesiredPropertyUpdateCallbackAsync](/dotnet/api/microsoft.azure.devices.client.deviceclient.setdesiredpropertyupdatecallbackasync).

1. En el método de devolución de llamada, actualice las propiedades notificadas del módulo gemelo con el método [UpdateReportedPropertiesAsync](/dotnet/api/microsoft.azure.devices.client.moduleclient), pasando una instancia de [TwinCollection](/dotnet/api/microsoft.azure.devices.shared.twincollection) de los valores de propiedad que desea establecer.

## <a name="access-the-module-twins"></a>Acceso a los módulos gemelos

Puede revisar el código JSON de los módulos gemelos en Azure IoT Hub, en Visual Studio Code y con la CLI de Azure.

### <a name="monitor-in-azure-iot-hub"></a>Supervisión en Azure IoT Hub

Para ver el código JSON del módulo gemelo:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a IoT Hub.
1. Seleccione **IoT Edge** en el menú del panel de la izquierda.
1. En la pestaña **Dispositivos IoT Edge**, seleccione el **identificador de dispositivo** del dispositivo con los módulos que desea supervisar.
1. Seleccione el nombre del módulo en la pestaña **Módulos** y, después, seleccione **Identidad de módulo gemela** en la barra de menús superior.

  ![Selección de un módulo gemelo para verlo en Azure Portal](./media/how-to-monitor-module-twins/select-module-twin.png)

Si aparece el mensaje "No existe una identidad de módulo para este módulo", este error indica que la solución de back-end que originalmente creó la identidad ya no está disponible.

### <a name="monitor-module-twins-in-visual-studio-code"></a>Supervisión de módulos gemelos en Visual Studio Code

Para revisar y editar un módulo gemelo:

1. Si no está instalada, instale la [extensión Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) para Visual Studio Code.
1. En el **Explorador**, expanda **Azure IoT Hub** y, a continuación, expanda el dispositivo con el módulo que desea supervisar.
1. Haga clic con el botón derecho en el módulo y seleccione **Editar módulo gemelo**. Un archivo temporal del módulo gemelo se descarga en el equipo y se muestra en Visual Studio Code.

  ![Obtención de un módulo gemelo para su edición en Visual Studio Code](./media/how-to-monitor-module-twins/edit-module-twin-vscode.png)

Si realiza cambios, seleccione **Update Module Twin** (Actualizar módulo gemelo) por encima del código del editor para guardar los cambios en el centro de IoT.

  ![Actualizar un módulo gemelo en Visual Studio Code](./media/how-to-monitor-module-twins/update-module-twin-vscode.png)

### <a name="monitor-module-twins-in-azure-cli"></a>Supervisión de módulos gemelos en la CLI de Azure

Para ver si IoT Edge se está ejecutando, use el comando [az iot hub invoke-module-method](how-to-edgeagent-direct-method.md#ping) para hacer ping al agente de IoT Edge.

La estructura [az iot hub module-twin](/cli/azure/iot/hub/module-twin) proporciona estos comandos:

* **az iot hub module-twin show**: muestra una definición del módulo gemelo.
* **az iot hub module-twin update**: actualiza la definición de un módulo gemelo.
* **az iot hub module-twin replace**: reemplaza la definición de un módulo gemelo con un JSON de destino.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [comunicarse con EdgeAgent mediante métodos directos integrados](how-to-edgeagent-direct-method.md).
