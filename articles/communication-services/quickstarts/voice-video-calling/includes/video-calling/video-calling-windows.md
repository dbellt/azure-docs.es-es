---
ms.openlocfilehash: 05d7ac0fc46ddbe279208e9d60fb9f039985ad06
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111560740"
---
En este artículo de inicio rápido aprenderá a iniciar una llamada de vídeo 1:1 con el SDK de llamada de Azure Communication Services para Windows.

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
Compruebe `Internet (Client & Server)` para obtener acceso entrante y saliente a Internet. Compruebe `Microphone` para acceder a la fuente de audio del micrófono. Compruebe `WebCam` para acceder a la cámara del dispositivo. 

Agregue el siguiente código a su `Package.appxmanifest`. 
```
<Extensions>
<Extension Category="windows.activatableClass.inProcessServer">
<InProcessServer>
<Path>RtmMvrUap.dll</Path>
<ActivatableClass ActivatableClassId="VideoN.VideoSchemeHandler" ThreadingModel="both" />
</InProcessServer>
</Extension>
</Extensions>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Es necesario configurar un diseño básico para adjuntar la lógica. Para hacer una llamada saliente, se necesita un elemento `TextBox` para proporcionar el id. de usuario del destinatario. También se necesita un botón `Start Call` y un botón `Hang Up`. También es necesario obtener una vista previa del vídeo local y representar el vídeo remoto del otro participante. Por lo tanto, necesitamos dos elementos para mostrar las secuencias de vídeo.

Abra `MainPage.xaml` del proyecto y reemplace el contenido por la siguiente implementación. 

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
    <StackPanel Orientation="Vertical" HorizontalAlignment="Center">
        <MediaElement x:Name="RemoteVideo" AutoPlay="True" Stretch="UniformToFill"/>
        <MediaElement x:Name="LocalVideo" AutoPlay="True"  Stretch="UniformToFill" HorizontalAlignment="Right"  VerticalAlignment="Bottom"/>
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
            this.InitCallAgentAndDeviceManager();
        }
        
        private async void InitCallAgentAndDeviceManager()
        {
            // Initialize call agent and Device Manager
        }

        private async void CallButton_ClickAsync(object sender, RoutedEventArgs e)
        {
            // Authenticate the client and start call
        }
        
        private async void Agent_OnIncomingCall(object sender, IncomingCall incomingcall)
        {
            // Accept an incoming call
        }

        private async void HangupButton_Click(object sender, RoutedEventArgs e)
        {
            // End the current call
        }

        CallClient callClient;
        CallAgent callAgent;
        Call call;
        DeviceManager deviceManager;
        LocalVideoStream[] localVideoStream;
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

Inicialice una instancia de `CallAgent` con un token de acceso de usuario que nos permita realizar y recibir llamadas. Para acceder a las cámaras del dispositivo, también es necesario obtener una instancia de Administrador de dispositivos. 

```C#
private async void InitCallAgentAndDeviceManager()
{
    CallClient callClient = new CallClient();
    deviceManager = await callClient.GetDeviceManager();

    CommunicationTokenCredential token_credential = new CommunicationTokenCredential("<USER_ACCESS_TOKEN>");
    callClient = new CallClient();

    CallAgentOptions callAgentOptions = new CallAgentOptions()
    {
        DisplayName = "<DISPLAY_NAME>"
    };
    callAgent = await callClient.CreateCallAgent(token_credential, callAgentOptions);
    callAgent.OnCallsUpdated += Agent_OnCallsUpdated;
    callAgent.OnIncomingCall += Agent_OnIncomingCall;
}
```

## <a name="start-a-call-with-video"></a>Inicio de una llamada con vídeo

Para iniciar una llamada con vídeo. Es necesario enumerar las cámaras con la instancia del administrador de dispositivos y construir `LocalVideoStream`. Es necesario establecer `VideoOptions` con `LocalVideoStream` y pasarlo con `startCallOptions` para establecer las opciones iniciales de la llamada. Al adjuntar `LocalVideoStream` a `MediaElement`, podemos ver la vista previa del vídeo local. 

```C#
private async void CallButton_ClickAsync(object sender, RoutedEventArgs e)
{
    Debug.Assert(deviceManager.Microphones.Count > 0);
    Debug.Assert(deviceManager.Speakers.Count > 0);
    Debug.Assert(deviceManager.Cameras.Count > 0);

    if (deviceManager.Cameras.Count > 0)
    {
        VideoDeviceInfo videoDeviceInfo = deviceManager.Cameras[0];
        localVideoStream = new LocalVideoStream[1];
        localVideoStream[0] = new LocalVideoStream(videoDeviceInfo);

        Uri localUri = await localVideoStream[0].CreateBindingAsync();

        await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
        {
            LocalVideo.Source = localUri;
            LocalVideo.Play();
        });

    }

    StartCallOptions startCallOptions = new StartCallOptions();
    startCallOptions.VideoOptions = new VideoOptions(localVideoStream);
    ICommunicationIdentifier[] callees = new ICommunicationIdentifier[1]
    {
        new CommunicationUserIdentifier(CalleeTextBox.Text)
    };

    call = await callAgent.StartCallAsync(callees, startCallOptions);
}
```

## <a name="accept-an-incoming-call"></a>Aceptar una llamada entrante

Para responder a una llamada entrante con vídeo, pase `LocalVideoStream` a `acceptCallOptions`. 

```C#
private async void Agent_OnIncomingCall(object sender, IncomingCall incomingcall)
{
    Debug.Assert(deviceManager.Microphones.Count > 0);
    Debug.Assert(deviceManager.Speakers.Count > 0);
    Debug.Assert(deviceManager.Cameras.Count > 0);

    if (deviceManager.Cameras.Count > 0)
    {
        VideoDeviceInfo videoDeviceInfo = deviceManager.Cameras[0];
        localVideoStream = new LocalVideoStream[1];
        localVideoStream[0] = new LocalVideoStream(videoDeviceInfo);

        Uri localUri = await localVideoStream[0].CreateBindingAsync();

        await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
        {
            LocalVideo.Source = localUri;
            LocalVideo.Play();
        });

    }
    AcceptCallOptions acceptCallOptions = new AcceptCallOptions();
    acceptCallOptions.VideoOptions = new VideoOptions(localVideoStream);

    call = await incomingcall.Accept(acceptCallOptions);
}
```

## <a name="remote-participant-and-remote-video-streams"></a>Secuencias de vídeo remotas y participantes remotos

Todos los participantes remotos están disponibles mediante la colección `RemoteParticipants` de una instancia de la llamada. Una vez conectada la llamada, podemos acceder a los participantes remotos de la llamada y controlar las secuencias de vídeo remotas. 

```C#
private async void Agent_OnCallsUpdated(object sender, CallsUpdatedEventArgs args)
{
    foreach (var call in args.AddedCalls)
    {
        foreach (var remoteParticipant in call.RemoteParticipants)
        {
            await AddVideoStreams(remoteParticipant.VideoStreams);
            remoteParticipant.OnVideoStreamsUpdated += async (s, a) => await AddVideoStreams(a.AddedRemoteVideoStreams);
        }
        call.OnRemoteParticipantsUpdated += Call_OnRemoteParticipantsUpdated;
        call.OnStateChanged += Call_OnStateChanged;
    }
}

private async void Call_OnRemoteParticipantsUpdated(object sender, ParticipantsUpdatedEventArgs args)
{
    foreach (var remoteParticipant in args.AddedParticipants)
    {
        await AddVideoStreams(remoteParticipant.VideoStreams);
        remoteParticipant.OnVideoStreamsUpdated += async (s, a) => await AddVideoStreams(a.AddedRemoteVideoStreams);
    }
}
```

### <a name="render-remote-videos"></a>Representación de vídeos remotos

Adjunte a `MediaElement` cada secuencia de vídeo remota. 

```C#
private async Task AddVideoStreams(IReadOnlyList<RemoteVideoStream> streams)
{

    foreach (var remoteVideoStream in streams)
    {
        var remoteUri = await remoteVideoStream.CreateBindingAsync();

        await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
        {
            RemoteVideo.Source = remoteUri;
            RemoteVideo.Play();
        });
        remoteVideoStream.Start();
    }
}
```

## <a name="call-state-update"></a>Actualización del estado de la llamada
Es necesario limpiar los representadores de vídeo una vez desconectada la llamada. 

```C#
private async void Call_OnStateChanged(object sender, PropertyChangedEventArgs args)
{
    switch (((Call)sender).State)
    {
        case CallState.Disconnected:
            await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
            {
                LocalVideo.Source = null;
                RemoteVideo = null;
            });
            break;
        default:
            System.Console.WriteLine(((Call)sender).State);
            break;
    }
}
```

## <a name="end-a-call"></a>Finalizar una llamada

Finalice la llamada actual cuando se haga clic en el botón `Hang Up`. 

```C#
private async void HangupButton_Click(object sender, RoutedEventArgs e)
{
    var hangUpOptions = new HangUpOptions();
    await call.HangUp(hangUpOptions);
}
```

## <a name="run-the-code"></a>Ejecución del código

Puede compilar y ejecutar el código en Visual Studio. Tenga en cuenta que para las plataformas de soluciones, se admiten `ARM64`, `x64` y `x86`. 

Para hacer una llamada de vídeo saliente, proporcione un id. de usuario en el campo de texto y haga clic en el botón `Start Call`. 
