---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 2bbff7c68b76b82bb834528124271eff69b8ad87
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111896383"
---
Puede empezar a usar Azure Communication Services con Calling SDK de Azure Communication Services para agregar llamadas individuales de vídeo a la aplicación. Aprenderá a iniciar videollamadas y a responder a ellas con Calling SDK de Azure Communication Services para JavaScript.


> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/add-1-on-1-video-calling)


## <a name="prerequisites"></a>Requisitos previos
- Obtenga una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Versiones Active LTS y Maintenance LTS de [Node.js](https://nodejs.org/en/) (8.11.1 y 10.14.1)
- Cree un recurso activo de Communication Services. [Cree un recurso de Communication Services](../../../create-communication-resource.md?pivots=platform-azp&tabs=windows).
- Cree un token de acceso de usuario para crear una instancia del cliente de llamada. [Creación y administración de tokens de acceso](../../../access-tokens.md?pivots=programming-language-csharp).

## <a name="setting-up"></a>Instalación
### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js
Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.
```console
mkdir calling-quickstart && cd calling-quickstart
```
### <a name="install-the-package"></a>Instalar el paquete
Use el comando `npm install` para instalar el SDK de llamadas de Azure Communication Services para JavaScript.

> [!IMPORTANT]
> En este inicio rápido se usa la versión `1.1.0-beta.1` del SDK de llamadas de Azure Communication Services. 


```console
npm install @azure/communication-common --save
npm install @azure/communication-calling@1.1.0-beta.1 --save
```
### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Esta guía de inicio rápido usa webpack para agrupar los recursos de la aplicación. Ejecute el siguiente comando para instalar los paquetes npm `webpack`, `webpack-cli` y `webpack-dev-server`, y los enumera como dependencias de desarrollo en el archivo `package.json`:

```console
npm install webpack@4.42.0 webpack-cli@3.3.11 webpack-dev-server@3.10.3 --save-dev
```
Cree un archivo `index.html` en el directorio raíz del proyecto. Este archivo lo usaremos para configurar un diseño básico que permitirá al usuario realizar una llamada de video 1:1.

Este es el código:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Communication Client - 1:1 Video Calling Sample</title>
</head>

<body>
    <h4>Azure Communication Services</h4>
    <h1>1:1 Video Calling Quickstart</h1>
    <input 
    id="callee-id-input"
    type="text"
    placeholder="Who would you like to call?"
    style="margin-bottom:1em; width: 200px;"
    />

    <div>
    <button id="call-button" type="button" disabled="true">
        start call
    </button>
        &nbsp;
    <button id="hang-up-button" type="button" disabled="true">
        hang up
    </button>
        &nbsp;
    <button id="start-Video" type="button" disabled="true">
        start video
    </button>
        &nbsp;
    <button id="stop-Video" type="button" disabled="true">
        stop video
    </button>     
    </div>

    <div>Local Video</div>
    <div style="height:200px; width:300px; background-color:black; position:relative;">
      <div id="myVideo" style="background-color: black; position:absolute; top:50%; transform: translateY(-50%);">
      </div>     
    </div>
    <div>Remote Video</div>
    <div style="height:200px; width:300px; background-color:black; position:relative;"> 
      <div id="remoteVideo" style="background-color: black; position:absolute; top:50%; transform: translateY(-50%);">
      </div>
    </div>

    <script src="./bundle.js"></script>
</body>
</html>
```

Cree un archivo en el directorio raíz del proyecto denominado `client.js` que contendrá la lógica de la aplicación para esta guía de inicio rápido. Agregue el siguiente código para importar el cliente que realiza la llamada y obtener referencias a los elementos del DOM.

```JavaScript
import { CallClient, CallAgent, VideoStreamRenderer, LocalVideoStream } from "@azure/communication-calling";
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

let call;
let callAgent;
const calleeInput = document.getElementById("callee-id-input");
const callButton = document.getElementById("call-button");
const hangUpButton = document.getElementById("hang-up-button");
const stopVideoButton = document.getElementById("stop-Video");
const startVideoButton = document.getElementById("start-Video");

let placeCallOptions;
let deviceManager;
let localVideoStream;
let rendererLocal;
let rendererRemote;
```
## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de llamadas de Azure Communication Services:

| Nombre      | Descripción | 
| :---        |    :----   |
| CallClient  | CallClient es el punto de entrada principal al SDK de llamadas.      |
| CallAgent  | CallAgent se usa para iniciar y administrar llamadas.        |
| DeviceManager | DeviceManager se usa para administrar dispositivos multimedia.    |
| AzureCommunicationTokenCredential | La clase AzureCommunicationTokenCredential implementa la interfaz CommunicationTokenCredential, que se usa para crear una instancia de CallAgent.        |

## <a name="authenticate-the-client-and-access-devicemanager"></a>Autenticación del cliente y acceso a DeviceManager

Debe reemplazar <[USER_ACCESS_TOKEN](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/access-tokens-quickstart)> por un token de acceso de usuario válido para el recurso. Consulte la documentación relativa al token de acceso de usuario si aún no tiene ningún token disponible. Con el `CallClient`, inicialice una instancia de `CallAgent` con un `CommunicationUserCredential` que nos permita realizar y recibir llamadas. Para acceder a `DeviceManager`, primero se debe crear una instancia de callAgent. Luego, puede usar el método `getDeviceManager` en la instancia de `CallClient` para obtener el elemento `DeviceManager`.

Agregue el siguiente código a `client.js`:

```JavaScript
async function init() {
    const callClient = new CallClient();
    const tokenCredential = new AzureCommunicationTokenCredential("<USER ACCESS TOKEN>");
    callAgent = await callClient.createCallAgent(tokenCredential, { displayName: 'optional ACS user name' });
    
    // Receive an incoming call
    // To handle incoming calls you need to listen to the `incomingCall` event of `callAgent`. Once there is an incoming call, you need to enumerate local cameras and construct 
    // a `LocalVideoStream` instance to send a video stream to the other participant. You also need to subscribe to `remoteParticipants` to handle remote video streams. You can 
    // accept or reject the call through the `incomingCall` instance. 
    callAgent.on('incomingCall', async e => {
        const videoDevices = await deviceManager.getCameras();
        const videoDeviceInfo = videoDevices[0];
        localVideoStream = new LocalVideoStream(videoDeviceInfo);
        localVideoView();

        stopVideoButton.disabled = false;
        callButton.disabled = true;
        hangUpButton.disabled = false;

        const addedCall = await e.incomingCall.accept({videoOptions: {localVideoStreams:[localVideoStream]}});
        call = addedCall;

        subscribeToRemoteParticipantInCall(addedCall);  
    });
    
    // Subscribe to call updates
    // You need to subscribe to the event when the remote participant ends the call to dispose of video renderers and toggle button states. 
    callAgent.on('callsUpdated', e => {
        e.removed.forEach(removedCall => {
            // dispose of video renders
            rendererLocal.dispose();
            rendererRemote.dispose();
            // toggle button states
            hangUpButton.disabled = true;
            callButton.disabled = false;
            stopVideoButton.disabled = true;
        })
    })

    deviceManager = await callClient.getDeviceManager();
    callButton.disabled = false;
}

init();
```
## <a name="place-a-11-outgoing-video-call-to-a-user"></a>Realización de una llamada saliente de vídeo 1:1 a un usuario

Agregue una escucha de eventos para iniciar una llamada cuando se haga clic en `callButton`:

En primer lugar, debe enumerar las cámaras locales mediante la API `getCameraList` de deviceManager. En esta guía de inicio rápido usamos la primera cámara de la colección. Una vez seleccionada la cámara deseada, se creará una instancia de LocalVideoStream y se pasará al método de llamada dentro de `videoOptions` como un elemento de la matriz localVideoStream. Una vez que la llamada se conecta, iniciará automáticamente el envío de una secuencia de vídeo al otro participante. 

```JavaScript
callButton.addEventListener("click", async () => {
    const videoDevices = await deviceManager.getCameras();
    const videoDeviceInfo = videoDevices[0];
    localVideoStream = new LocalVideoStream(videoDeviceInfo);
    placeCallOptions = {videoOptions: {localVideoStreams:[localVideoStream]}};

    localVideoView();
    stopVideoButton.disabled = false;
    startVideoButton.disabled = true;

    const userToCall = calleeInput.value;
    call = callAgent.startCall(
        [{ communicationUserId: userToCall }],
        placeCallOptions
    );

    subscribeToRemoteParticipantInCall(call);

    hangUpButton.disabled = false;
    callButton.disabled = true;
});
```  
Para representar un elemento `LocalVideoStream`, debe crear una nueva instancia de `VideoStreamRenderer` y, a continuación, crear una nueva instancia de `VideoStreamRendererView` con el método asincrónico `createView`. Después puede adjuntar `view.target` a cualquier elemento de la interfaz de usuario. 

```JavaScript
async function localVideoView() {
    rendererLocal = new VideoStreamRenderer(localVideoStream);
    const view = await rendererLocal.createView();
    document.getElementById("myVideo").appendChild(view.target);
}
```
Todos los participantes remotos están disponibles mediante la colección `remoteParticipants` de una instancia de la llamada. Debe escuchar el evento `remoteParticipantsUpdated` para que le notifiquen cuando se agregue un nuevo participante remoto a la llamada. También debe recorrer en iteración la colección `remoteParticipants` para suscribirse a cada uno de ellos con el fin de suscribirse a sus transmisiones de vídeo. 

```JavaScript
function subscribeToRemoteParticipantInCall(callInstance) {
    callInstance.on('remoteParticipantsUpdated', e => {
        e.added.forEach( p => {
            subscribeToParticipantVideoStreams(p);
        })
    }); 
    callInstance.remoteParticipants.forEach( p => {
        subscribeToParticipantVideoStreams(p);
    })
}
```
Debe suscribirse al evento `videoStreamsUpdated` para administrar las transmisiones de vídeo agregadas de los participantes remotos. Puede inspeccionar la colección `videoStreams` para enumerar las secuencias de cada participante mientras recorre la colección `remoteParticipants` de la llamada actual.

```JavaScript
function subscribeToParticipantVideoStreams(remoteParticipant) {
    remoteParticipant.on('videoStreamsUpdated', e => {
        e.added.forEach(v => {
            handleVideoStream(v);
        })
    });
    remoteParticipant.videoStreams.forEach(v => {
        handleVideoStream(v);
    });
}
```
Para representar el elemento `remoteVideoStream`, se debe suscribir a un evento `isAvailableChanged`. Si la propiedad `isAvailable` cambia a `true`, un participante remoto envía una secuencia. Cada vez que cambie la disponibilidad de una secuencia remota, puede elegir destruir todo el elemento `Renderer`, un elemento `RendererView` específico o mantenerlos, pero esto hará que el fotograma del vídeo se vea en blanco.
```JavaScript
function handleVideoStream(remoteVideoStream) {
    remoteVideoStream.on('isAvailableChanged', async () => {
        if (remoteVideoStream.isAvailable) {
            remoteVideoView(remoteVideoStream);
        } else {
            rendererRemote.dispose();
        }
    });
    if (remoteVideoStream.isAvailable) {
        remoteVideoView(remoteVideoStream);
    }
}
```
Para representar un elemento `RemoteVideoStream`, debe crear una nueva instancia de `VideoStreamRenderer` y, a continuación, crear una nueva instancia de `VideoStreamRendererView` con el método asincrónico `createView`. Después puede adjuntar `view.target` a cualquier elemento de la interfaz de usuario. 

```JavaScript
async function remoteVideoView(remoteVideoStream) {
    rendererRemote = new VideoStreamRenderer(remoteVideoStream);
    const view = await rendererRemote.createView();
    document.getElementById("remoteVideo").appendChild(view.target);
}
```

## <a name="end-the-current-call"></a>Finalización de la llamada actual
Agregue un cliente de escucha de eventos para finalizar la llamada actual cuando se haga clic en el `hangUpButton`:
```JavaScript
hangUpButton.addEventListener("click", async () => {
    // dispose of the renderers
    rendererLocal.dispose();
    rendererRemote.dispose();
    // end the current call
    await call.hangUp();
    // toggle button states
    hangUpButton.disabled = true;
    callButton.disabled = false;
    stopVideoButton.disabled = true;
});
```

## <a name="start-and-end-video-during-the-call"></a>Inicio y finalización del vídeo durante la llamada
Puede detener el vídeo durante la llamada actual mediante la incorporación de una escucha de eventos al botón para detener el vídeo para desechar el representador de `localVideoStream`. 
 ```JavaScript       
stopVideoButton.addEventListener("click", async () => {
    await call.stopVideo(localVideoStream);
    rendererLocal.dispose();
    startVideoButton.disabled = false;
    stopVideoButton.disabled = true;
});
```
Puede agregar una escucha de eventos al botón para iniciar el vídeo para volver a activar el vídeo cuando se detuvo durante la llamada actual. 
```JavaScript
startVideoButton.addEventListener("click", async () => {
    await call.startVideo(localVideoStream);
    localVideoView();
    stopVideoButton.disabled = false;
    startVideoButton.disabled = true;
});
```
## <a name="run-the-code"></a>Ejecución del código
Utilice `webpack-dev-server` para compilar y ejecutar la aplicación. Ejecute el siguiente comando para agrupar el host de aplicación en un servidor web local:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```
Abra el explorador web y vaya a http://localhost:8080/. Verá lo siguiente:

:::image type="content" source="../../media/javascript/1-on-1-video-calling.png" alt-text="Página para llamada de vídeo 1:1":::

Para hacer una llamada de vídeo saliente 1:1, proporcione un identificador de usuario en el campo de texto y haga clic en el botón para iniciar llamada. 

## <a name="sample-code"></a>Código de ejemplo
Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/add-1-on-1-video-calling).
