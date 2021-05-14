---
title: Configuración de puntos de conexión web
titleSuffix: Azure Cognitive Services
description: Configuración de puntos de conexión web para Comandos personalizados
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 5f1d5318140dd14c5024e8dd3ad0def0afc7f378
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107725915"
---
# <a name="set-up-web-endpoints"></a>Configuración de puntos de conexión web

En este artículo, aprenderá a configurar los puntos de conexión web en una aplicación de Comandos personalizados que le permitirá realizar solicitudes HTTP desde una aplicación cliente. Deberá completar las siguientes tareas:

- Configurar los puntos de conexión web en la aplicación de Comandos personalizados.
- Llamar a puntos de conexión web en la aplicación de Comandos personalizados.
- Recibir la respuesta de los puntos de conexión web.
- Integrar la respuesta de los puntos de conexión web en una carga JSON personalizada, enviarla y visualizarla desde una aplicación cliente UWP en C# del SDK de Voz.

## <a name="prerequisites"></a>Requisitos previos

> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
> * Una clave de suscripción de Azure para el servicio de voz: [Obtenga una gratis](overview.md#try-the-speech-service-for-free) o créela en [Azure Portal](https://portal.azure.com).
> * Una aplicación de Comandos personalizados (consulte [Creación de un asistente para voz con Comandos personalizados](quickstart-custom-commands-application.md))
> * Una aplicación cliente habilitada para el SDK de voz (consulte [Integración con una aplicación cliente mediante el SDK de voz](how-to-custom-commands-setup-speech-sdk.md)).

## <a name="deploy-an-external-web-endpoint-using-azure-function-app"></a>Implementación de un punto de conexión web externo mediante una aplicación de funciones de Azure

En este tutorial, se necesita un punto de conexión HTTP que mantenga los estados de todos los dispositivos configurados en el comando **TurnOnOff** de la aplicación de Comandos personalizados.

Si ya tiene un punto de conexión web al que quiera llamar, puede ir directamente a la [sección siguiente](#setup-web-endpoints-in-custom-commands). También, en la sección siguiente se proporcionan detalles sobre un punto de conexión web hospedado predeterminado que puede usar si quiere omitir esta sección.

### <a name="input-format-of-azure-function"></a>Formato de entrada de la función de Azure

A continuación, implementará un punto de conexión mediante [Azure Functions](../../azure-functions/index.yml).
Este es el formato de un evento de Comandos personalizados que se pasa a la función de Azure. Use esta información al escribir la aplicación de funciones de Azure.

```json
{
  "conversationId": "string",
  "currentCommand": {
    "name": "string",
    "parameters": {
      "SomeParameterName": "string",
      "SomeOtherParameterName": "string"
    }
  },
  "currentGlobalParameters": {
      "SomeGlobalParameterName": "string",
      "SomeOtherGlobalParameterName": "string"
  }
}
```

    
En la tabla siguiente se describen los atributos clave de esta entrada:
        
| Atributo | Explicación |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **conversationId** | El identificador único de la conversación. Tenga en cuenta que este identificador se puede generar desde la aplicación cliente. |
| **currentCommand** | El comando que está activo actualmente en la conversación. |
| **name** | El nombre del comando. El atributo `parameters` es un mapa con los valores actuales de los parámetros. |
| **currentGlobalParameters** | Un mapa como `parameters`, pero se usa para los parámetros globales. |


En el caso de la función de Azure **DeviceState**, un evento de Comandos personalizados de ejemplo tendrá el siguiente aspecto. Esta función servirá como **entrada** a la aplicación de funciones.
    
```json
{
  "conversationId": "someConversationId",
  "currentCommand": {
    "name": "TurnOnOff",
    "parameters": {
      "item": "tv",
      "value": "on"
    }
  }
}
```

### <a name="azure-function-output-for-a-custom-command-app"></a>Salida de la función de Azure en una aplicación de Comandos personalizados

Si una aplicación de Comandos personalizados usa la salida de la función de Azure, debería aparecer en el formato siguiente. Para más información, consulte [Actualizar un comando desde un punto de conexión web](./how-to-custom-commands-update-command-from-web-endpoint.md).

```json
{
  "updatedCommand": {
    "name": "SomeCommandName",
    "updatedParameters": {
      "SomeParameterName": "SomeParameterValue"
    },
    "cancel": false
  },
  "updatedGlobalParameters": {
    "SomeGlobalParameterName": "SomeGlobalParameterValue"
  }
}
```

### <a name="azure-function-output-for-a-client-application"></a>Salida de la función de Azure en una aplicación cliente

Si una aplicación cliente consume la salida de la función de Azure, la salida puede tener el formato que requiera la aplicación cliente.

En el caso del punto de conexión **DeviceState**, una aplicación cliente, y no la aplicación de Comandos personalizados, consume la salida de la función de Azure. La salida de ejemplo de la función de Azure debe parecerse a esta:
    
```json
{
  "TV": "on",
  "Fan": "off"
}
``` 

Esta salida debe escribirse en un almacenamiento externo para que se pueda mantener el estado de los dispositivos. El estado de almacenamiento externo se usará en la sección [Integración con la aplicación cliente](#integrate-with-client-application) que viene a continuación.


### <a name="deploy-azure-function"></a>Implementar la función de Azure

Se proporciona un ejemplo que puede configurar e implementar como una aplicación de Azure Functions. Para crear una cuenta de almacenamiento en nuestro ejemplo, siga estos pasos:
 
1. Cree el almacenamiento de tabla para guardar el estado del dispositivo. En Azure Portal, cree un recurso de tipo **Cuenta de almacenamiento** con el nombre **devicestate**.
1. Copie el valor de **Cadena de conexión** de **devicestate -> Claves de acceso**. Deberá agregar este secreto de cadena al código de la aplicación de funciones de ejemplo descargado.
1. Descargue el [código de la aplicación de funciones](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/main/custom-commands/quick-start) de ejemplo.
1. Abra la solución descargada en Visual Studio 2019. En **Connections.jsen**, reemplace **STORAGE_ACCOUNT_SECRET_CONNECTION_STRING** por el secreto del paso 2.
1.  Descargue el código de **DeviceStateAzureFunction**.

Para implementar la aplicación de ejemplo en Azure Functions, siga estos pasos.

1. [Implemente](../../azure-functions/index.yml) la aplicación de Azure Functions.
1. Espere a que la implementación se realice correctamente y vaya al recurso implementado en Azure Portal. 
1. Seleccione **Funciones** en el panel izquierdo y, luego, elija **DeviceState**.
1.  En la nueva ventana, seleccione **Code + Test** (Código y prueba) y, luego, elija **Obtener la dirección URL de la función**.
 
## <a name="setup-web-endpoints-in-custom-commands"></a>Configuración de puntos de conexión web en Comandos personalizados

Vamos a enlazar la función de Azure con la aplicación de Comandos personalizados existente.
En esta sección, usará un punto de conexión **DeviceState** predeterminado ya existente. Si creó su propio punto de conexión web mediante Azure Functions o de otro modo, úselo en lugar del predeterminado`https://webendpointexample.azurewebsites.net/api/DeviceState`.

1. Abra la aplicación de Comandos personalizados que creó anteriormente.
1. Vaya a **Puntos de conexión web** y haga clic en **New web endpoint** (Nuevo punto de conexión web).

   > [!div class="mx-imgBorder"]
   > ![Nuevo punto de conexión web](media/custom-commands/setup-web-endpoint-new-endpoint.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Nombre | UpdateDeviceState | Nombre del punto de conexión web. |
   | URL | https://webendpointexample.azurewebsites.net/api/DeviceState | Dirección URL del punto de conexión con el que quiere que se comunique la aplicación de comandos personalizados. |
   | Método | POST | Interacciones permitidas (como GET, POST) con el punto de conexión.|
   | encabezados | Clave: app. Valor: tomar los primeros 8 dígitos de applicationId | Parámetros de encabezado que se incluirán en el encabezado de solicitud.|

    > [!NOTE]
    > - El punto de conexión web de ejemplo que se creó con [Azure Functions](../../azure-functions/index.yml), que establece un enlace con la base de datos que guarda el estado del dispositivo tanto del televisor como del ventilador.
    > - El encabezado sugerido solo es necesario para el punto de conexión de ejemplo.
    > - Para tener la seguridad de que el valor del encabezado es único en nuestro punto de conexión de ejemplo, tome los primeros 8 dígitos de su **applicationId**.
    > - En realidad, el punto de conexión web puede ser el punto de conexión al [centro de IoT](../../iot-hub/about-iot-hub.md) que administra los dispositivos.

1. Haga clic en **Save**(Guardar).

## <a name="call-web-endpoints"></a>Llamada a los puntos de conexión web

1. Diríjase al comando **TurnOnOff**, seleccione **ConfirmationResponse** en regla de finalización y, a continuación, seleccione **Agregar una acción.**
1. En **Nuevo tipo de acción**, seleccione **Call web endpoint** (llamar a un punto de conexión web).
1. En **Editar acción: puntos de conexión**, seleccione **UpdateDeviceState**, que es el punto de conexión web que hemos creado.  
1. En **Configuración**, introduzca los siguientes valores:
   > [!div class="mx-imgBorder"]
   > ![Parámetros de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-edit-action-parameters.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Puntos de conexión | UpdateDeviceState | Punto de conexión web que quiere llamar en esta acción. |
   | Parámetros de consulta | item={SubjectDevice}&&value={OnOff} | Parámetros de consulta que se anexarán a la dirección URL del punto de conexión web.  |
   | Contenido del cuerpo | N/D | Contenido del cuerpo de la solicitud. |

    > [!NOTE]
    > - Los parámetros de consulta sugeridos solo son necesarios para el punto de conexión de ejemplo.

1. Desde **En caso de éxito: acción para ejecutar**, seleccione **Send speech response** (enviar respuesta de voz).

    En **Simple editor** (Editor sencillo), escriba `{SubjectDevice} is {OnOff}`.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra la pantalla Acierto - Acción para ejecutar.](media/custom-commands/setup-web-endpoint-edit-action-on-success-send-response.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Acción para ejecutar | Enviar respuesta de voz | Acción que se va a ejecutar si la solicitud al punto de conexión web se realiza correctamente. |

   > [!NOTE]
   > - También puede acceder directamente a los campos de la respuesta HTTP mediante `{YourWebEndpointName.FieldName}`. Por ejemplo: `{UpdateDeviceState.TV}`

1. Desde **En caso de error: acción para ejecutar**, seleccione **Send speech response** (enviar respuesta de voz).

    En **Simple editor** (Editor sencillo), escriba `Sorry, {WebEndpointErrorMessage}`.

   > [!div class="mx-imgBorder"]
   > ![En caso de error de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-edit-action-on-fail.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Acción para ejecutar | Enviar respuesta de voz | Acción que se va a ejecutar si hay un error en la solicitud al punto de conexión web. |

   > [!NOTE]
   > - `{WebEndpointErrorMessage}` es opcional. Puede quitarlo si no quiere exponer ningún mensaje de error.
   > - En nuestro punto de conexión de ejemplo, se devuelve una respuesta HTTP con mensajes de error detallados para los errores comunes, tales como los parámetros de encabezado faltantes.

### <a name="try-it-out-in-test-portal"></a>Prueba en el portal de pruebas
- Si la respuesta es correcta, guarde, pruebe y entrene.
   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra la respuesta de Acierto.](media/custom-commands/setup-web-endpoint-on-success-response.png)
- Si la respuesta es incorrecta, quite uno de los parámetros de consulta y guarde, vuelva a entrenar y pruebe.
   > [!div class="mx-imgBorder"]
   > ![En caso de error de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-on-fail-response.png)

## <a name="integrate-with-client-application"></a>Integración con la aplicación cliente

En [Envío de una actividad de Comandos personalizados a la aplicación cliente](./how-to-custom-commands-send-activity-to-client.md), agregó una acción **Send activity to client** (Enviar actividad al cliente). La actividad se envía a la aplicación cliente, independientemente de si la acción **Call web endpoint** (llamar a un punto de conexión web) se realiza correctamente o no.
Sin embargo, normalmente solo querrá enviar la actividad a la aplicación cliente cuando la llamada al punto de conexión web se realice correctamente. En este ejemplo, es cuando el estado del dispositivo se actualizó correctamente.

1. Elimine la acción **Send activity to client** (enviar una actividad al cliente) que agregó anteriormente.
1. Edite la acción de llamada a un punto de conexión web:
    1. En **Configuración**, asegúrese de que **Parámetros de consulta** sea `item={SubjectDevice}&&value={OnOff}`
    1. Desde **En caso de éxito**, cambie **Acción para ejecutar** por **Send activity to client** (enviar una actividad al cliente).
    1. Copie el código JSON siguiente en el campo **Activity content** (contenido de la actividad).
   ```json
   {
      "type": "event",
      "name": "UpdateDeviceState",
      "value": {
        "state": "{OnOff}",
        "device": "{SubjectDevice}"
      }
    }
   ```
Ahora, solo envía la actividad al cliente cuando la solicitud al punto de conexión web se realiza correctamente.

### <a name="create-visuals-for-syncing-device-state"></a>Creación de objetos visuales para sincronizar el estado del dispositivo

Agregue el siguiente código XML a `MainPage.xaml` antes del bloque **EnableMicrophoneButton**.

```xml
<Button x:Name="SyncDeviceStateButton" Content="Sync Device State"
        Margin="0,10,10,0" Click="SyncDeviceState_ButtonClicked"
        Height="35"/>
<Button x:Name="EnableMicrophoneButton" ......
        .........../>
```

### <a name="sync-device-state"></a>Sincronización del estado de dispositivo

En `MainPage.xaml.cs`, agregue la referencia `using Windows.Web.Http;`. Agregue el siguiente código a la clase `MainPage` . Este método enviará una solicitud GET al punto de conexión de ejemplo y extraerá el estado actual del dispositivo para la aplicación. Asegúrese de cambiar `<your_app_name>` por el valor que usó en el **encabezado** del punto de conexión web de Comandos personalizados.

```C#
private async void SyncDeviceState_ButtonClicked(object sender, RoutedEventArgs e)
{
    //Create an HTTP client object
    var httpClient = new HttpClient();

    //Add a user-agent header to the GET request.
    var your_app_name = "<your-app-name>";

    Uri endpoint = new Uri("https://webendpointexample.azurewebsites.net/api/DeviceState");
    var requestMessage = new HttpRequestMessage(HttpMethod.Get, endpoint);
    requestMessage.Headers.Add("app", $"{your_app_name}");

    try
    {
        //Send the GET request
        var httpResponse = await httpClient.SendRequestAsync(requestMessage);
        httpResponse.EnsureSuccessStatusCode();
        var httpResponseBody = await httpResponse.Content.ReadAsStringAsync();
        dynamic deviceState = JsonConvert.DeserializeObject(httpResponseBody);
        var TVState = deviceState.TV.ToString();
        var FanState = deviceState.Fan.ToString();
        await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
            CoreDispatcherPriority.Normal,
            () =>
            {
                State_TV.Text = TVState;
                State_Fan.Text = FanState;
            });
    }
    catch (Exception ex)
    {
        NotifyUser(
            $"Unable to sync device status: {ex.Message}");
    }
}
```

## <a name="try-it-out"></a>Prueba

1. Inicie la aplicación.
1. Haga clic en Sync Device State (sincronizar estado del dispositivo).\
Si probó la aplicación con `turn on tv` en la sección anterior, notará que el televisor aparece como **encendido**.
    > [!div class="mx-imgBorder"]
    > ![Sincronización del estado de dispositivo](media/custom-commands/setup-web-endpoint-sync-device-state.png)
1. Seleccione **Habilitar micrófono**.
1. Seleccionar el botón **Hablar**.
1. Diga `turn on the fan`. El estado visual del ventilador debe cambiar a **encendido**.
    > [!div class="mx-imgBorder"]
    > ![Encender ventilador](media/custom-commands/setup-web-endpoint-turn-on-fan.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Exportación de la aplicación Comandos personalizados como una aptitud remota](./how-to-custom-commands-integrate-remote-skills.md)
