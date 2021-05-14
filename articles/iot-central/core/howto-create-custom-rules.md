---
title: Ampliación de Azure IoT Central con notificaciones y reglas personalizadas | Microsoft Docs
description: Como desarrollador de soluciones, configure una aplicación de IoT Central para enviar notificaciones por correo electrónico cuando un dispositivo deja de enviar datos de telemetría. Esta solución utiliza Azure Stream Analytics, Azure Functions y SendGrid.
author: philmea
ms.author: philmea
ms.date: 02/09/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: mvc, devx-track-csharp
manager: philmea
ms.openlocfilehash: a65d9dbaed4d197c2e0843e73ff3f45b8678017e
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107864224"
---
# <a name="extend-azure-iot-central-with-custom-rules-using-stream-analytics-azure-functions-and-sendgrid"></a>Extensión de Azure IoT Central con reglas personalizadas mediante Stream Analytics, Azure Functions y SendGrid

Esta guía paso a paso le muestra, como desarrollador de soluciones, cómo ampliar la aplicación de IoT Central con notificaciones y reglas personalizadas. El ejemplo muestra el envío de una notificación a un operador cuando un dispositivo deja de enviar datos de telemetría. La solución utiliza una consulta de [Azure Stream Analytics](../../stream-analytics/index.yml) para detectar cuándo un dispositivo ha dejado de enviar datos de telemetría. El trabajo de Stream Analytics utiliza [Azure Functions](../../azure-functions/index.yml) para enviar correos electrónicos de notificación mediante [SendGrid](https://sendgrid.com/docs/for-developers/partners/microsoft-azure/).

Esta guía paso a paso le muestra cómo ampliar IoT Central todavía más con las reglas y acciones integradas.

En esta guía paso a paso, aprenderá a:

* Hacer streaming de datos de telemetría desde una aplicación de IoT Central mediante la *exportación de datos continua*.
* Crear una consulta de Stream Analytics que detecta cuándo un dispositivo ha dejado de enviar datos.
* Enviar una notificación por correo electrónico con los servicios de SendGrid y Azure Functions.

## <a name="prerequisites"></a>Prerequisites

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

### <a name="iot-central-application"></a>Aplicación de IoT Central

Cree una aplicación de IoT Central desde el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral) con la configuración siguiente:

| Configuración | Value |
| ------- | ----- |
| Plan de precios | Estándar |
| Plantilla de la aplicación | Análisis en la tienda: supervisión del estado |
| Nombre de la aplicación | Acepte el valor predeterminado o elija su propio nombre. |
| URL | Acepte el valor predeterminado o elija un prefijo de dirección URL único. |
| Directorio | El inquilino de Azure Active Directory |
| Suscripción de Azure | Su suscripción de Azure |
| Region | La región más cercana |

En los ejemplos y las capturas de pantalla de este artículo se usa la región **Estados Unidos**. Elija una ubicación cercana a usted y asegúrese de que crea todos los recursos en la misma región.

Esta plantilla de aplicación incluye dos dispositivos de termostato simulados que envían telemetría.

### <a name="resource-group"></a>Resource group

Use [Azure Portal para crear un grupo de recursos](https://portal.azure.com/#create/Microsoft.ResourceGroup) llamado **DetectStoppedDevices** que contenga los otros recursos que cree. Cree los recursos de Azure en la misma ubicación que la aplicación de IoT Central.

### <a name="event-hubs-namespace"></a>Espacio de nombres de Event Hubs

Use [Azure Portal para crear un espacio de nombres de Event Hubs](https://portal.azure.com/#create/Microsoft.EventHub) con la siguiente configuración:

| Configuración | Valor |
| ------- | ----- |
| Nombre    | Elija el nombre del espacio de nombres |
| Plan de tarifa | Básica |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| Location | Este de EE. UU. |
| Unidades de procesamiento | 1 |

### <a name="stream-analytics-job"></a>Trabajo de Stream Analytics

Use [Azure Portal para crear un trabajo de Stream Analytics](https://portal.azure.com/#create/Microsoft.StreamAnalyticsJob) con la siguiente configuración:

| Configuración | Valor |
| ------- | ----- |
| Nombre    | Elija el nombre del trabajo |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| Location | Este de EE. UU. |
| Entorno de hospedaje | Nube |
| Unidades de streaming | 3 |

### <a name="function-app"></a>Aplicación de función

Use [Azure Portal para crear una aplicación de función](https://portal.azure.com/#create/Microsoft.FunctionApp) con la siguiente configuración:

| Configuración | Valor |
| ------- | ----- |
| Nombre de la aplicación    | Elija el nombre de la aplicación de función |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| SO | Windows |
| Plan de hospedaje | Plan de consumo |
| Location | Este de EE. UU. |
| Pila de tiempo de ejecución | .NET |
| Storage | Crear nuevo |

### <a name="sendgrid-account-and-api-keys"></a>Cuenta de SendGrid y claves de API

Antes de comenzar, si no tiene una cuenta de SendGrid, cree una [cuenta gratuita](https://app.sendgrid.com/).

1. En la configuración del panel de SendGrid en el menú de la izquierda, seleccione **Claves de API**.
1. Haga clic en **Crear clave de API**.
1. Asigne a la nueva clave de API el nombre **AzureFunctionAccess**.
1. Haga clic en **Create & View** (Crear y ver).

    :::image type="content" source="media/howto-create-custom-rules/sendgrid-api-keys.png" alt-text="Captura de pantalla de creación de clave de API de SendGrid.":::

Después, se le proporcionará una clave de API. Guarde esta cadena para usarla más adelante.

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Puede configurar una aplicación de IoT Central para la exportación continua de datos de telemetría a un centro de eventos. En esta sección, creará un centro de eventos para recibir datos de telemetría de la aplicación de IoT Central. El centro de eventos entrega los datos de telemetría al trabajo de Stream Analytics para su procesamiento.

1. En Azure Portal, vaya al espacio de nombres de Event Hubs y seleccione **+ Centro de eventos**.
1. Asigne al centro de eventos el nombre **centralexport** y seleccione **Crear**.

El espacio de nombres de Event Hubs se parece a la captura de pantalla siguiente: 

```:::image type="content" source="media/howto-create-custom-rules/event-hubs-namespace.png" alt-text="Screenshot of Event Hubs namespace." border="false":::

## Define the function

This solution uses an Azure Functions app to send an email notification when the Stream Analytics job detects a stopped device. To create your function app:

1. In the Azure portal, navigate to the **App Service** instance in the **DetectStoppedDevices** resource group.
1. Select **+** to create a new function.
1. Select **HTTP Trigger**.
1. Select **Add**.

    :::image type="content" source="media/howto-create-custom-rules/add-function.png" alt-text="Image of the Default HTTP trigger function"::: 

## Edit code for HTTP Trigger

The portal creates a default function called **HttpTrigger1**:

```:::image type="content" source="media/howto-create-custom-rules/default-function.png" alt-text="Screenshot of Edit HTTP trigger function.":::

1. Replace the C# code with the following code:

    ```csharp
    #r "Newtonsoft.Json"
    #r "SendGrid"
    using System;
    using SendGrid.Helpers.Mail;
    using Microsoft.Azure.WebJobs.Host;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static SendGridMessage Run(HttpRequest req, ILogger log)
    {
        string requestBody = new StreamReader(req.Body).ReadToEnd();
        log.LogInformation(requestBody);
        var notifications = JsonConvert.DeserializeObject<IList<Notification>>(requestBody);

        SendGridMessage message = new SendGridMessage();
        message.Subject = "Contoso device notification";

        var content = "The following device(s) have stopped sending telemetry:<br/><br/><table><tr><th>Device ID</th><th>Time</th></tr>";
        foreach(var notification in notifications) {
            log.LogInformation($"No message received - Device: {notification.deviceid}, Time: {notification.time}");
            content += $"<tr><td>{notification.deviceid}</td><td>{notification.time}</td></tr>";
        }
        content += "</table>";
        message.AddContent("text/html", content);  

        return message;
    }

    public class Notification
    {
        public string deviceid { get; set; }
        public string time { get; set; }
    }
    ```

    You may see an error message until you save the new code.
1. Select **Save** to save the function.

## Add SendGrid Key

To add your SendGrid API Key, you need to add it to your **Function Keys** as follows:

1. Select **Function Keys**.
1. Choose **+ New Function Key**.
1. Enter the *Name* and *Value* of the API Key you created before.
1. Click **OK.**

    :::image type="content" source="media/howto-create-custom-rules/add-key.png" alt-text="Screenshot of Add Sangrid Key.":::


## Configure HttpTrigger function to use SendGrid

To send emails with SendGrid, you need to configure the bindings for your function as follows:

1. Select **Integrate**.
1. Choose **Add Output** under **HTTP ($return)**.
1. Select **Delete.**
1. Choose **+ New Output**.
1. For Binding Type, then choose **SendGrid**.
1. For SendGrid API Key Setting Type, click New.
1. Enter the *Name* and *Value* of your SendGrid API key.
1. Add the following information:

| Setting | Value |
| ------- | ----- |
| Message parameter name | Choose your name |
| To address | Choose the name of your To Address |
| From address | Choose the name of your From Address |
| Message subject | Enter your subject header |
| Message text | Enter the message from your integration |

1. Select **OK**.

    :::image type="content" source="media/howto-create-custom-rules/add-output.png" alt-text="Screenshot of Add SandGrid Output.":::


### Test the function works

To test the function in the portal, first choose **Logs** at the bottom of the code editor. Then choose **Test** to the right of the code editor. Use the following JSON as the **Request body**:

```json
[{"deviceid":"test-device-1","time":"2019-05-02T14:23:39.527Z"},{"deviceid":"test-device-2","time":"2019-05-02T14:23:50.717Z"},{"deviceid":"test-device-3","time":"2019-05-02T14:24:28.919Z"}]
```

Los mensajes del registro de la función aparecen en el panel **Registros**:

```:::image type="content" source="media/howto-create-custom-rules/function-app-logs.png" alt-text="Function log output":::

After a few minutes, the **To** email address receives an email with the following content:

```txt
The following device(s) have stopped sending telemetry:

Device ID    Time
test-device-1    2019-05-02T14:23:39.527Z
test-device-2    2019-05-02T14:23:50.717Z
test-device-3    2019-05-02T14:24:28.919Z
```

## <a name="add-stream-analytics-query"></a>Incorporación de una consulta de Stream Analytics

Esta solución utiliza una consulta de Stream Analytics para detectar cuándo un dispositivo ha dejado de enviar datos de telemetría durante más de 120 segundos. La consulta usa la telemetría del centro de eventos como entrada. El trabajo envía los resultados de consulta a la aplicación de función. En esta sección configurará un trabajo de Stream Analytics:

1. En Azure Portal, vaya a su trabajo de Stream Analytics, en **Topología de trabajo** seleccione **Entradas**, elija **+ Agregar entrada de flujo** y, luego, **Centro de eventos**.
1. Use la información de la tabla siguiente para configurar la entrada usando el centro de eventos que creó anteriormente, y luego elija **Guardar**:

    | Configuración | Value |
    | ------- | ----- |
    | Alias de entrada | centraltelemetry |
    | Subscription | Su suscripción |
    | Espacio de nombres del Centro de eventos | Espacio de nombres del centro de eventos |
    | Nombre del centro de eventos | Usar existente: **centralexport** |

1. En **Topología de trabajo**, seleccione **Salidas**, elija **+ Agregar** y, luego, **función de Azure**.
1. Use la información de la tabla siguiente para configurar la salida, y luego elija **Guardar**:

    | Configuración | Value |
    | ------- | ----- |
    | Alias de salida | emailnotification |
    | Subscription | Su suscripción |
    | Aplicación de función | Su aplicación de función |
    | Función  | HttpTrigger1 |

1. En **Topología de trabajo**, seleccione **Consulta** y reemplace la consulta existente con el siguiente SQL:

    ```sql
    with
    LeftSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid1, 
        EventEnqueuedUtcTime AS time1
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    ),
    RightSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid2, 
        EventEnqueuedUtcTime AS time2
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    )

    SELECT
        LeftSide.deviceid1 as deviceid,
        LeftSide.time1 as time
    INTO
        [emailnotification]
    FROM
        LeftSide
        LEFT OUTER JOIN
        RightSide 
        ON
        LeftSide.deviceid1=RightSide.deviceid2 AND DATEDIFF(second,LeftSide,RightSide) BETWEEN 1 AND 120
        where
        -- Find records where a device didn't send a message 120 seconds
        RightSide.deviceid2 is NULL
    ```

1. Seleccione **Guardar**.
1. Para iniciar el trabajo de Stream Analytics, elija **Información general**, después **Inicio** y **Ahora** y, a continuación, **Inicio**:

    :::image type="content" source="media/howto-create-custom-rules/stream-analytics.png" alt-text="Captura de pantalla de Stream Analytics.":::

## <a name="configure-export-in-iot-central"></a>Configuración de la exportación en IoT Central 

En el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral), vaya a la aplicación de IOT Central que creó.

En esta sección va a configurar la aplicación para que haga streaming de los datos de telemetría desde sus dispositivos simulados al centro de eventos. Para configurar la exportación:

1. Vaya a la página **Exportación de datos**, seleccione **+ Nuevo** y, después, **Azure Event Hubs**.
1. Utilice los siguientes valores para configurar la exportación y, luego, seleccione **Guardar**: 

    | Configuración | Valor |
    | ------- | ----- |
    | Display Name (Nombre para mostrar) | Exportar a Event Hubs |
    | habilitado | Por |
    | Tipo de datos para exportar | Telemetría |
    | Características enriquecidas | Escriba la clave o el valor deseados de cómo quiere que se organicen los datos exportados. | 
    | Destination | Cree uno y escriba información sobre a dónde se exportarán los datos. |

    :::image type="content" source="media/howto-create-custom-rules/cde-configuration.png" alt-text="Captura de pantalla Exportación de datos.":::

Espere hasta que el estado de la exportación sea **En ejecución** antes de continuar.

## <a name="test"></a>Prueba

Para probar la solución, puede deshabilitar la exportación continua de datos de IoT Central para los dispositivos simulados detenidos:

1. En la aplicación IoT Central, vaya hasta la página **Exportación de datos** y seleccione la configuración de exportación **Exportar a Event Hubs**.
1. Establezca **Habilitado** en **Desactivado** y elija **Guardar**.
1. Después de un mínimo de dos minutos la dirección de correo electrónico de **destino** recibe uno o más correos electrónicos que tienen un aspecto similar al ejemplo siguiente:

    ```txt
    The following device(s) have stopped sending telemetry:

    Device ID         Time
    Thermostat-Zone1  2019-11-01T12:45:14.686Z
    ```

## <a name="tidy-up"></a>Limpieza

Para hacer limpieza después de este artículo de procedimientos y evitar costos innecesarios, elimine el grupo de recursos **DetectStoppedDevices** de Azure Portal.

Puede eliminar la aplicación de IoT Central desde la página **Administración** de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

En esta guía paso a paso, ha aprendido lo siguiente:

* Hacer streaming de datos de telemetría desde una aplicación de IoT Central mediante la *exportación de datos continua*.
* Crear una consulta de Stream Analytics que detecta cuándo un dispositivo ha dejado de enviar datos.
* Enviar una notificación por correo electrónico con los servicios de SendGrid y Azure Functions.

Ahora que sabe cómo crear notificaciones y reglas personalizadas, el siguiente paso que le sugerimos es que aprenda sobre la [Ampliación de Azure IoT Central con análisis personalizados](howto-create-custom-analytics.md).
