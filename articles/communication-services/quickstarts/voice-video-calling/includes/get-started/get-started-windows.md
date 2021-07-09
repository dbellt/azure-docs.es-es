---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 7589190f29e7d85bfb49a868dde9b261882d28b0
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111594098"
---
En este artículo de inicio rápido aprenderá a iniciar una llamada con Calling SDK de Azure Communication Services para Windows.

> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/VoiceCalling)

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Instale [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) con la carga de trabajo de desarrollo de la Plataforma universal de Windows. 
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../../create-communication-resource.md).
- Un [token de acceso de usuario](../../../access-tokens.md) para su instancia de Azure Communication Services.

## <a name="setting-up"></a>Instalación

### <a name="creating-the-project"></a>Creación del proyecto

En Visual Studio, cree un proyecto con la plantilla **Aplicación vacía (Windows universal)** para configurar una aplicación para la Plataforma universal de Windows (UWP) de una sola página.

:::image type="content" source="../../media/windows/create-a-new-project.png" alt-text="Captura de pantalla que muestra la ventana Nuevo proyecto en Visual Studio.":::

### <a name="install-the-package"></a>Instalar el paquete

Haga clic con el botón derecho en el proyecto y vaya a `Manage Nuget Packages` para instalar `Azure.Communication.Calling`. 

### <a name="request-access"></a>Solicitar acceso

Vaya a `Package.appxmanifest` y haga clic en `Capabilities`.
Compruebe `Internet (Client & Server)` para obtener acceso entrante y saliente a Internet. Compruebe `Microphone` para acceder a la fuente de audio del micrófono. 

:::image type="content" source="../../media/windows/request-access.png" alt-text="Captura de pantalla que muestra la solicitud de acceso a Internet y al micrófono en Visual Studio.":::

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Es necesario configurar un diseño básico para adjuntar la lógica. Para hacer una llamada saliente, se necesita un elemento `TextBox` para proporcionar el identificador de usuario del destinatario. También se necesita un botón `Start Call` y un botón `Hang Up`. Abra `MainPage.xaml` del proyecto y agregue el nodo `StackPanel` a `Page`: 

```C#
<Page
    x:Class="CallingQuickstart.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:CallingQuickstart"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
    <StackPanel>
        <TextBox Text="Who would you like to call?" TextWrapping="Wrap" x:Name="CalleeTextBox" Margin="10,10,10,10"></TextBox>
        <Button Content="Start Call" Click="CallButton_ClickAsync" x:Name="CallButton" Margin="10,10,10,10"></Button>
        <Button Content="Hang Up" Click="HangupButton_Click" x:Name="HangupButton" Margin="10,10,10,10"></Button>
    </StackPanel>
</Page>
```

Abra `MainPage.xaml.cs` y reemplace el contenido por la siguiente implementación: 
```C#
using System;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

using Azure.Communication;
using Azure.Communication.Calling;

namespace CallingQuickstart
{
    /// <summary>
    /// An empty page that can be used on its own or navigated to within a Frame.
    /// </summary>
    public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
            this.InitCallAgent();
        }
        
        private async void InitCallAgent()
        {
            // Create Call Client and initialize Call Agent
        }
        
        private async void CallButton_ClickAsync(object sender, RoutedEventArgs e)
        {
            // Start call
        }

        private async void HangupButton_Click(object sender, RoutedEventArgs e)
        {
            // End the current call
        }

        CallClient call_client_;
        CallAgent call_agent_;
        Call call_;
    }
}
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de llamadas de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient | CallClient es el punto de entrada principal al SDK de llamadas.|
| CallAgent | CallAgent se usa para iniciar y administrar llamadas. |
| CommunicationTokenCredential | CommunicationTokenCredential se usa como la credencial de token para crear una instancia de CallAgent.| 
| CommunicationUserIdentifier | CommunicationUserIdentifier se usa para representar la identidad del usuario, que puede ser una de las opciones siguientes: CommunicationUserIdentifier/PhoneNumberIdentifier/CallingApplication. |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Inicialice una instancia de `CallAgent` con un token de acceso de usuario que nos permita realizar y recibir llamadas. Agregue el siguiente código a la función `InitCallAgent`. 

```C#
CommunicationTokenCredential token_credential = new CommunicationTokenCredential("<USER_ACCESS_TOKEN>");
call_client_ = new CallClient();

CallAgentOptions callAgentOptions = new CallAgentOptions()
{
    DisplayName = "<YOUR_DISPLAY_NAME>"
};
call_agent_ = await call_client_.CreateCallAgent(token_credential, callAgentOptions);
```

## <a name="start-a-call"></a>Iniciar una llamada

Agregue la implementación a `CallButton_ClickAsync` para iniciar una llamada con el `call_agent` creado. 

```C#
StartCallOptions startCallOptions = new StartCallOptions();
ICommunicationIdentifier[] callees = new ICommunicationIdentifier[1]
{
    new CommunicationUserIdentifier(CalleeTextBox.Text)
};
call_ = await call_agent_.StartCallAsync(callees, startCallOptions);
```

## <a name="end-a-call"></a>Finalizar una llamada

Finalice la llamada actual cuando se haga clic en el botón `Hang Up`. 

```C#
private async void HangupButton_Click(object sender, RoutedEventArgs e)
{
    await call_.HangUpAsync(new HangUpOptions());
}
```

## <a name="run-the-code"></a>Ejecución del código

Puede compilar y ejecutar el código en Visual Studio. Tenga en cuenta que para las plataformas de soluciones, se admiten `ARM64`, `x64` y `x86`. 

Para hacer una llamada saliente, proporcione un identificador de usuario en el campo de texto y haga clic en el botón `Start Call`. La llamada a `8:echo123` le conecta a un bot de eco; esto es excelente como introducción y para comprobar que los dispositivos de audio funcionan.

:::image type="content" source="../../media/windows/run-the-app.png" alt-text="Captura de pantalla que muestra la ejecución de la aplicación de inicio rápido":::

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/VoiceCalling).
