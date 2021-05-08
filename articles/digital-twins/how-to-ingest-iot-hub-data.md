---
title: Ingesta de telemetría desde IoT Hub
titleSuffix: Azure Digital Twins
description: Consulte cómo ingerir los mensajes de telemetría de dispositivos desde IoT Hub.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 9/15/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: b69ba164a7bd0edecf427866cd3e872a65e41355
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108208750"
---
# <a name="ingest-iot-hub-telemetry-into-azure-digital-twins"></a>Ingesta de telemetría de IoT Hub en Azure Digital Twins

Azure Digital Twins se basa en los datos de los dispositivos IoT y otros orígenes. Un origen común de los datos del dispositivo que se va a usar en Azure Digital Twins es [IoT Hub](../iot-hub/about-iot-hub.md).

El proceso de ingesta de datos en Azure Digital Twins consiste en configurar un recurso de proceso externo como, por ejemplo, una función creada con [Azure Functions](../azure-functions/functions-overview.md). La función recibe los datos y usa las [API de DigitalTwins](/rest/api/digital-twins/dataplane/twins) para establecer propiedades o desencadenar eventos de telemetría en los [gemelos digitales](concepts-twins-graph.md) en consecuencia. 

Este documento de procedimientos le guía en el proceso de escritura de una función que pueda ingerir datos de telemetría de IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

Antes de continuar con este ejemplo, debe configurar los siguientes recursos como requisito previo:
* **Una instancia de IoT Hub**. Para instrucciones, consulte la sección *Creación de una instancia de IoT Hub* de [este inicio rápido de IoT Hub](../iot-hub/quickstart-send-telemetry-cli.md).
* **Una instancia de Azure Digital Twins** que recibirá la telemetría del dispositivo. Para instrucciones, consulte [Procedimiento: Configuración de una instancia de Azure Digital Twins y autenticación](./how-to-set-up-instance-portal.md).

### <a name="example-telemetry-scenario"></a>Escenario de telemetría de ejemplo

En este procedimiento se explica cómo enviar mensajes desde IoT Hub a Azure Digital Twins, mediante una función de Azure. Hay muchas configuraciones y estrategias de coincidencia posibles que puede usar para enviar mensajes, pero el ejemplo de este artículo contiene los siguientes elementos:
* Un dispositivo de termómetro en IoT Hub, con un identificador de dispositivo conocido
* Un gemelo digital para representar el dispositivo, con un identificador coincidente

> [!NOTE]
> En este ejemplo se usa una coincidencia de identificador sencillo entre el identificador de dispositivo y el identificador de gemelo digital correspondiente, pero es posible proporcionar asignaciones más sofisticadas del dispositivo a su gemelo (por ejemplo, con una tabla de asignación).

Siempre que el dispositivo de termostato envía un evento de telemetría de temperatura, una función procesa la telemetría y la propiedad *temperatura* del gemelo digital debe actualizarse. Este escenario se describe en un diagrama a continuación:

:::image type="content" source="media/how-to-ingest-iot-hub-data/events.png" alt-text="Diagrama de un dispositivo de IoT Hub envía la telemetría de temperatura con IoT Hub a una función de Azure que actualiza una propiedad de temperatura de un gemelo en Azure Digital Twins." border="false":::

## <a name="add-a-model-and-twin"></a>Adición de un modelo y un gemelo

En esta sección, configurará un [gemelo digital](concepts-twins-graph.md) en Azure Digital Twins que representará el dispositivo de termostato, y se actualizará con información de IoT Hub.

Para crear un gemelo de tipo termostato, primero debe cargar el [modelo](concepts-models.md) de termostato en la instancia, que describe las propiedades de un termostato y que se usará más adelante para crear el gemelo.

[!INCLUDE [digital-twins-thermostat-model-upload.md](../../includes/digital-twins-thermostat-model-upload.md)]

A continuación, deberá **crear un gemelo con este modelo**. Use el comando siguiente para crear un gemelo de termostato llamado **thermostat67** y establezca 0,0 como valor de temperatura inicial.

```azurecli-interactive
az dt twin create --dtmi "dtmi:contosocom:DigitalTwins:Thermostat;1" --twin-id thermostat67 --properties '{"Temperature": 0.0,}' --dt-name {digital_twins_instance_name}
```

> [!Note]
> Si usa Cloud Shell en el entorno de PowerShell, puede que deba colocar el carácter de escape delante de las comillas en los campos de JSON insertado para que sus valores se analicen correctamente. Este es el comando para crear el gemelo con esta modificación:
>
> Crear el gemelo:
> ```azurecli-interactive
> az dt twin create --dtmi "dtmi:contosocom:DigitalTwins:Thermostat;1" --twin-id thermostat67 --properties '{\"Temperature\": 0.0,}' --dt-name {digital_twins_instance_name}
> ```

Si el gemelo se crea correctamente, la salida de la CLI del comando debe ser similar a la siguiente:
```json
{
  "$dtId": "thermostat67",
  "$etag": "W/\"0000000-9735-4f41-98d5-90d68e673e15\"",
  "$metadata": {
    "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
    "Temperature": {
      "ackCode": 200,
      "ackDescription": "Auto-Sync",
      "ackVersion": 1,
      "desiredValue": 0.0,
      "desiredVersion": 1
    }
  },
  "Temperature": 0.0
}
```

## <a name="create-a-function"></a>Creación de una función

En esta sección, creará una función de Azure para acceder a Azure Digital Twins y actualizar los gemelos en función de los eventos de telemetría de IoT que reciba. Siga los pasos que se indican a continuación para crear y publicar la función.

#### <a name="step-1-create-a-function-app-project"></a>Paso 1: Creación de un proyecto de aplicación de funciones

En primer lugar, cree un proyecto de aplicación de funciones en Visual Studio. Para instrucciones sobre cómo hacerlo, consulte la sección [Creación de una aplicación de funciones en Visual Studio](how-to-create-azure-function.md#create-a-function-app-in-visual-studio) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

#### <a name="step-2-fill-in-function-code"></a>Paso 2: Relleno del código de función

Agregue los siguientes paquetes al proyecto:
* [Azure.DigitalTwins.Core](https://www.nuget.org/packages/Azure.DigitalTwins.Core/)
* [Azure.Identity](https://www.nuget.org/packages/Azure.Identity/)
* [Microsoft.Azure.WebJobs.Extensions.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid/)

Cambie el nombre de la función de ejemplo *Function1.cs* que Visual Studio ha generado con el nuevo proyecto a *IoTHubtoTwins.cs*. Reemplace el código de este archivo por el código siguiente:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/IoTHubToTwins.cs":::

Guarde el código de función.

#### <a name="step-3-publish-the-function-app-to-azure"></a>Paso 3: Publicación de la aplicación de funciones en Azure

Publique el proyecto con la función *IoTHubtoTwins. CS* en una aplicación de función en Azure.

Para instrucciones sobre cómo hacerlo, consulte la sección [Publicación de la aplicación de funciones en Azure](how-to-create-azure-function.md#publish-the-function-app-to-azure) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

#### <a name="step-4-configure-the-function-app"></a>Paso 4: Configuración de la aplicación de funciones

A continuación, **asigne un rol de acceso** para la función y **configure las opciones de la aplicación** para que pueda acceder a la instancia de Azure Digital Twins. Para instrucciones sobre cómo hacerlo, consulte la sección [Configuración del acceso de seguridad para la aplicación de funciones](how-to-create-azure-function.md#set-up-security-access-for-the-function-app) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

## <a name="connect-your-function-to-iot-hub"></a>Conexión de la función a IoT Hub

En esta sección, configurará la función como un destino de evento para los datos del dispositivo de IoT Hub. De esta forma, se garantiza que los datos del dispositivo de termostato de IoT Hub se enviarán a la función de Azure para procesarlos.

En [Azure Portal](https://portal.azure.com/), navegue a la instancia de IoT Hub que creó en la sección [Requisitos previos](#prerequisites). En **Eventos**, cree una suscripción para su función.

:::image type="content" source="media/how-to-ingest-iot-hub-data/add-event-subscription.png" alt-text="Captura de pantalla de Azure Portal que muestra la adición de una suscripción de evento.":::

En la página **Crear suscripción de eventos**, rellene los campos como se indica a continuación:
  1. En **Nombre**, elija el nombre que desee para la suscripción de eventos.
  2. En **Esquema de eventos**, elija _Esquema de Event Grid_.
  3. En **System Topic Name** (Nombre de tema del sistema), elija el nombre que prefiera.
  1. En **Filtro para tipos de evento**, active la casilla _Device Telemetry_ (Telemetría de dispositivo) y desactive otros tipos de eventos.
  1. En **Tipo de punto de conexión**, seleccione _Función de Azure_.
  1. En **Punto de conexión**, use el vínculo _Seleccionar un extremo_ para elegir qué función de Azure se usará para el punto de conexión.
    
:::image type="content" source="media/how-to-ingest-iot-hub-data/create-event-subscription.png" alt-text="Captura de pantalla de Azure Portal que muestra los detalles de una suscripción a eventos":::

En la página _Seleccionar la función de Azure_ que se abre, compruebe los detalles siguientes.
 1. **Suscripción**: Su suscripción de Azure.
 2. **Grupo de recursos**: su grupo de recursos.
 3. **Aplicación de funciones**: nombre de la aplicación de funciones.
 4. **Ranura**: _producción_
 5. **Función**: seleccione la función anterior, *IoTHubtoTwins*, en la lista desplegable.

Guarde los detalles con el botón _Confirmar selección_.            
      
:::image type="content" source="media/how-to-ingest-iot-hub-data/select-azure-function.png" alt-text="Captura de pantalla de Azure Portal para seleccionar la función.":::

Seleccione el botón _Crear_ para crear una suscripción de eventos.

## <a name="send-simulated-iot-data"></a>Envío de datos de IoT simulados

Para probar la nueva función de entrada, use el simulador de dispositivos de [Tutorial: Conexión de una solución de un extremo a otro](./tutorial-end-to-end.md). El tutorial utiliza un proyecto de ejemplo escrito en C#. El código de ejemplo se encuentra aquí: [Ejemplos de Azure Digital Twins de un extremo a otro](/samples/azure-samples/digital-twins-samples/digital-twins-samples). Usará el proyecto **DeviceSimulator** en ese repositorio.

En este tutorial integral, va a completar los siguientes pasos:
1. [Registro del dispositivo simulado en el centro de IoT](./tutorial-end-to-end.md#register-the-simulated-device-with-iot-hub)
2. [Configuración y ejecución de la simulación](./tutorial-end-to-end.md#configure-and-run-the-simulation)

## <a name="validate-your-results"></a>Validación de los resultados

Mientras se ejecuta el simulador de dispositivos anterior, cambiará el valor de temperatura del gemelo digital. En la CLI de Azure, ejecute el siguiente comando para ver el valor de temperatura.

```azurecli-interactive
az dt twin query -q "select * from digitaltwins" -n {digital_twins_instance_name}
```

La salida debe contener un valor de temperatura similar al siguiente:

```json
{
  "result": [
    {
      "$dtId": "thermostat67",
      "$etag": "W/\"0000000-1e83-4f7f-b448-524371f64691\"",
      "$metadata": {
        "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
        "Temperature": {
          "ackCode": 200,
          "ackDescription": "Auto-Sync",
          "ackVersion": 1,
          "desiredValue": 69.75806974934324,
          "desiredVersion": 1
        }
      },
      "Temperature": 69.75806974934324
    }
  ]
}
```

Para ver el cambio de valor, ejecute repetidamente el comando de consulta anterior.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la entrada y salida de datos con Azure Digital Twins:
* [Conceptos: Integración con otros servicios](concepts-integration.md)
