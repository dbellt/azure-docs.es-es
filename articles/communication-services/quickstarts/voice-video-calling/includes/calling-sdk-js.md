---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 2ecbd207c4b1946a69b01f43ec2bc77d29b1a8c9
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106073568"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un recurso implementado de Communication Services. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un token de acceso de usuario para habilitar el cliente de llamada. Para más información, consulte [Inicio rápido: Creación y administración de tokens de acceso](../../access-tokens.md).
- Opcional: Complete la guía de inicio rápido para [agregar llamadas de voz a la aplicación](../getting-started-with-calling.md).

## <a name="install-the-sdk"></a>Instalar el SDK

> [!NOTE]
> En este documento se usa el SDK web de llamada de ACS.

Use el comando `npm install` para instalar los SDK comunes y de llamada de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-common --save
npm install @azure/communication-calling --save

```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de llamadas de Azure Communication Services:

| Nombre                             | Descripción                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| `CallClient`                      | El punto de entrada principal al SDK de llamadas.                                                                       |
| `CallAgent`                        | Se utiliza para iniciar y administrar las llamadas.                                                                                            |
| `DeviceManager`                    | Se usa para administrar dispositivos multimedia.                                                                                           |
| `AzureCommunicationTokenCredential` | Implementa la interfaz de `CommunicationTokenCredential`, que se usa para crear instancias de `callAgent`. |

## <a name="initialize-a-callclient-instance-create-a-callagent-instance-and-access-devicemanager"></a>Inicialización de una instancia de CallClient, creación de una instancia de CallAgent y acceso a deviceManager

Cree una instancia de `CallClient`. Se puede configurar con opciones personalizadas como una instancia del registrador.

Cuando haya creado una instancia de `CallClient`, puede crear una de `CallAgent` mediante una llamada al método `createCallAgent` en la instancia de `CallClient`. De este modo, se devuelve un objeto de instancia de `CallAgent` de manera asincrónica.

El método `createCallAgent` utiliza `CommunicationTokenCredential` como argumento. Acepta un [token de acceso de usuario](../../access-tokens.md).

Después de crear una instancia de `callAgent`, puede utilizar el método `getDeviceManager` en la instancia de `CallClient` para tener acceso a `deviceManager`.

```js
// Set the logger's log level
setLogLevel('verbose');
// Redirect logger output to wherever desired. By default it logs to console
AzureLogger.log = (...args) => { console.log(...args) };
const userToken = '<user token>';
callClient = new CallClient(options);
const tokenCredential = new AzureCommunicationTokenCredential(userToken);
const callAgent = await callClient.createCallAgent(tokenCredential, {displayName: 'optional ACS user name'});
const deviceManager = await callClient.getDeviceManager()
```

## <a name="place-a-call"></a>Realización de una llamada

Para crear e iniciar una llamada, utilice una de las API en `callAgent` y proporcione un usuario que haya creado mediante el SDK de identidades de Communication Services.

La creación y el inicio de la llamada son sincrónicos. La instancia de llamada le permite suscribirse a eventos de llamada.

### <a name="place-a-1n-call-to-a-user-or-pstn"></a>Realización de una llamada 1:n a un usuario o RTC

Para llamar a otro usuario de Communication Services, invoque el método `startCall` en `callAgent` y pase la instancia de `CommunicationUserIdentifier` que ha [creado con la biblioteca de administración de Communication Services](../../access-tokens.md).

```js
const userCallee = { communicationUserId: '<ACS_USER_ID>' }
const oneToOneCall = callAgent.startCall([userCallee]);
```

Para realizar una llamada a una red telefónica conmutada (RTC) pública, use el método `startCall` en `callAgent` y pase el `PhoneNumberIdentifier` del destinatario. El recurso de Communication Services debe estar configurado para permitir llamadas de RTC.

Cuando llame a un número de RTC, debe especificar su identificador alternativo de autor de llamada. Un identificador del autor de llamada alternativo es un número de teléfono (basado en el estándar E.164), que identifica al autor de llamada en una llamada RTC. Es el número de teléfono que el destinatario de la llamada ve en una llamada entrante.

> [!NOTE]
> Llamadas RTC están actualmente en versión preliminar privada. Para acceder, [suscríbase al programa de usuarios pioneros](https://aka.ms/ACS-EarlyAdopter).

Para una llamada 1:1, utilice el código siguiente:

```js
const pstnCalee = { phoneNumber: '<ACS_USER_ID>' }
const alternateCallerId = {alternateCallerId: '<Alternate caller Id>'};
const oneToOneCall = callAgent.startCall([pstnCallee], {alternateCallerId});
```

Para una llamada 1: n, utilice el código siguiente:

```js
const userCallee = { communicationUserId: <ACS_USER_ID> }
const pstnCallee = { phoneNumber: <PHONE_NUMBER>};
const alternateCallerId = {alternateCallerId: '<Alternate caller Id>'};
const groupCall = callAgent.startCall([userCallee, pstnCallee], {alternateCallerId});

```

### <a name="place-a-11-call-with-video-camera"></a>Realización de una llamada 1:1 con videocámara

> [!IMPORTANT]
> Actualmente, no puede haber más de una secuencia de vídeo local saliente.

Para realizar una llamada de vídeo, tiene que especificar las cámaras mediante el método `getCameras()` de `deviceManager`.

Después de seleccionar una cámara, úsela para construir una instancia de `LocalVideoStream`. Páselo en `videoOptions` como un elemento de la matriz `localVideoStream` al método `startCall`.

```js
const deviceManager = await callClient.getDeviceManager();
const cameras = await deviceManager.getCameras();
const camera = cameras[0]
localVideoStream = new LocalVideoStream(camera);
const placeCallOptions = {videoOptions: {localVideoStreams:[localVideoStream]}};
const call = callAgent.startCall(['acsUserId'], placeCallOptions);

```

Cuando la llamada se conecta, iniciará automáticamente el envío de una secuencia de vídeo desde la cámara seleccionada hasta los demás participantes. Esto también se aplica a las opciones de vídeo de `Call.Accept()` y `CallAgent.join()`.

### <a name="join-a-group-call"></a>Unión a una llamada grupal

> [!NOTE]
> El parámetro `groupId` se considera metadatos del sistema y Microsoft puede usarlo para las operaciones necesarias para ejecutar el sistema. No incluya datos personales en el valor `groupId`. Microsoft no trata este parámetro como datos personales y su contenido puede ser visible para los empleados de Microsoft o almacenarse a largo plazo.
>
> El parámetro `groupId` requiere que los datos estén en formato GUID. Se recomienda usar GUID generados de forma aleatoria que no se consideran datos personales en los sistemas.
>

Para iniciar una llamada grupal nueva o unirse a una en curso, use el método `join` y pase un objeto con una propiedad `groupId`. El valor `groupId` debe ser un GUID.

```js

const context = { groupId: <GUID>}
const call = callAgent.join(context);

```

### <a name="join-a-teams-meeting"></a>Unión a una reunión de Teams
> [!NOTE]
> Esta API se ofrece a los desarrolladores como versión preliminar y puede cambiar en función de los comentarios que recibamos. No utilice esta API en un entorno de producción. Para usar esta API, utilice la versión beta del SDK web de llamada de ACS.

Para unirse a una reunión de Teams, use el método `join` y pase un vínculo o las coordenadas de la reunión.

Unirse mediante un vínculo de reunión:

```js
const locator = { meetingLink: <meeting link>}
const call = callAgent.join(locator);
```

Unirse mediante las coordenadas de reunión:

```js
const locator = {
    threadId: <thread id>,
    organizerId: <organizer id>,
    tenantId: <tenant id>,
    messageId: <message id>
}
const call = callAgent.join(locator);
```

## <a name="receive-an-incoming-call"></a>Recepción de una llamada entrante

La instancia de `callAgent` emite un evento `incomingCall` cuando la identidad que ha iniciado sesión recibe una llamada entrante. Para escuchar este evento, suscríbase mediante una de estas opciones:

```js
const incomingCallHander = async (args: { incomingCall: IncomingCall }) => {

    //Get incoming call ID
    var incomingCallId = incomingCall.id

    // Get information about caller
    var callerInfo = incomingCall.callerInfo

    // Accept the call
    var call = await incomingCall.accept();

    // Reject the call
    incomingCall.reject();

    // Subscribe to callEnded event and get the call end reason
     incomingCall.on('callEnded', args => {
        console.log(args.callEndReason);
    });

    // callEndReason is also a property of IncomingCall
    var callEndReason = incomingCall.callEndReason;
};
callAgentInstance.on('incomingCall', incomingCallHander);
```

El evento `incomingCall` incluye una instancia de `incomingCall` que puede aceptar o rechazar.

## <a name="manage-calls"></a>Administración de llamadas

Durante una llamada, puede tener acceso a las propiedades de llamada y administrar la configuración de vídeo y audio.

### <a name="check-call-properties"></a>Comprobación de las propiedades de llamada

Obtenga el identificador único (cadena) de una llamada:

   ```js
    const callId: string = call.id;
   ```

Para más información sobre los demás participantes de la llamada, revise la colección `remoteParticipants` de la instancia "call".

   ```js
   const remoteParticipants = call.remoteParticipants;
   ```

Identifique al autor de la llamada de una llamada entrante:

   ```js
   const callerIdentity = call.callerInfo.identifier;
   ```

   `identifier` es uno de los tipos de `CommunicationIdentifier`.

Obtenga el estado de una llamada:

   ```js
   const callState = call.state;
   ```

   Esto devuelve una cadena que representa el estado actual de una llamada:

  - `None`: estado inicial de la llamada.
  - `Connecting`: estado de transición inicial cuando se realiza o acepta la llamada.
  - `Ringing`: en el caso de una llamada saliente, indica que se está llamando a los participantes remotos. Es `Incoming` en su lado.
  - `EarlyMedia`: indica un estado en el que se reproduce un anuncio antes de conectar la llamada.
  - `Connected`: indica que la llamada está conectada.
  - `LocalHold`: indica que un participante local ha puesto en espera la llamada. No hay flujo de medios entre el punto de conexión local y los participantes remotos.
  - `RemoteHold`: indica que el participante remoto ha puesto en espera la llamada. No hay flujo de medios entre el punto de conexión local y los participantes remotos.
  - `Disconnecting`: estado de la transición antes de que la llamada pase a un estado `Disconnected`.
  - `Disconnected`: estado final de la llamada. Si se pierde la conexión de red, el estado cambia a `Disconnected` después de dos minutos.

Averigüe por qué finalizó una llamada mediante la inspección de la propiedad `callEndReason`:

   ```js
   const callEndReason = call.callEndReason;
   const callEndReasonCode = callEndReason.code // (number) code associated with the reason
   const callEndReasonSubCode = callEndReason.subCode // (number) subCode associated with the reason
   ```

Obtenga información sobre si la llamada actual es entrante o saliente mediante la inspección de la propiedad `direction`. Devuelve `CallDirection`.

  ```js
   const isIncoming = call.direction == 'Incoming';
   const isOutgoing = call.direction == 'Outgoing';
   ```

Compruebe si el micrófono actual está silenciado. Devuelve `Boolean`.

   ```js
   const muted = call.isMuted;
   ```

Para averiguar si la secuencia de uso compartido de pantalla se envía desde un punto de conexión determinado, revise la propiedad `isScreenSharingOn`: Devuelve `Boolean`.

   ```js
   const isScreenSharingOn = call.isScreenSharingOn;
   ```

Para inspeccionar las secuencias de vídeo activas, compruebe la colección `localVideoStreams`. Devuelve objetos `LocalVideoStream`.

   ```js
   const localVideoStreams = call.localVideoStreams;
   ```

### <a name="check-a-callended-event"></a>Comprobación de un evento callEnded

La instancia `call` emite un evento `callEnded` cuando finaliza la llamada. Para escuchar este evento, suscríbase mediante el código siguiente:

```js
const callEndHander = async (args: { callEndReason: CallEndReason }) => {
    console.log(args.callEndReason)
};

call.on('callEnded', callEndHander);
```

### <a name="mute-and-unmute"></a>Silencio y reactivación del sonido

Para silenciar o reactivar el sonido del punto de conexión local, puede usar las API asincrónicas `mute` y `unmute`:

```js

//mute local device
await call.mute();

//unmute local device
await call.unmute();

```

### <a name="start-and-stop-sending-local-video"></a>Inicio y detención del envío de vídeo local

Para iniciar un vídeo, tiene que especificar las cámaras mediante el método `getCameras` en el objeto `deviceManager`. Después, cree una instancia de `LocalVideoStream` pasando la cámara deseada al método `startVideo` como argumento:

```js
const deviceManager = await callClient.getDeviceManager();
const cameras = await deviceManager.getCameras();
const camera = cameras[0]
const localVideoStream = new LocalVideoStream(camera);
await call.startVideo(localVideoStream);
```

Una vez que logre empezar a enviar vídeo, se agregará una instancia de `LocalVideoStream` a la colección `localVideoStreams` en una instancia de llamada.

```js
call.localVideoStreams[0] === localVideoStream;
```

Para detener el vídeo local, pase la instancia de `localVideoStream` disponible en la colección `localVideoStreams`:

```js
await call.stopVideo(localVideoStream);
```

Puede cambiar a un dispositivo de cámara distinto mientras se envía vídeo si invoca `switchSource` en una instancia de `localVideoStream`:

```js
const cameras = await callClient.getDeviceManager().getCameras();
const camera = cameras[1];
localVideoStream.switchSource(camera);
```

## <a name="manage-remote-participants"></a>Administración de participantes remotos

El tipo `RemoteParticipant` representa a todos los participantes remotos, los que están disponibles a través de la colección `remoteParticipants` en una instancia de llamada.

### <a name="list-the-participants-in-a-call"></a>Enumeración de los participantes en una llamada

La colección `remoteParticipants` devuelve una lista de los participantes remotos en una llamada:

```js
call.remoteParticipants; // [remoteParticipant, remoteParticipant....]
```

### <a name="access-remote-participant-properties"></a>Acceso a las propiedades de los participantes remotos

Los participantes remotos tienen un conjunto de propiedades y colecciones asociadas:

- `CommunicationIdentifier`: obtenga el identificador de este participante remoto. "Identity" es uno de los tipos de `CommunicationIdentifier`:

  ```js
  const identifier = remoteParticipant.identifier;
  ```

  Puede ser uno de los siguientes tipos de `CommunicationIdentifier`:

  - `{ communicationUserId: '<ACS_USER_ID'> }`: objeto que representa el usuario de ACS.
  - `{ phoneNumber: '<E.164>' }`: objeto que representa el número de teléfono en formato E.164.
  - `{ microsoftTeamsUserId: '<TEAMS_USER_ID>', isAnonymous?: boolean; cloud?: "public" | "dod" | "gcch" }`: objeto que representa al usuario de Teams.
  - `{ id: string }`: objeto que representa el identificador que no se ajusta a ninguno de los otros tipos de identificadores.

- `state`: obtenga el estado de este participante remoto.

  ```js
  const state = remoteParticipant.state;
  ```

  El estado puede ser:

  - `Idle`: estado inicial.
  - `Connecting`: estado de transición mientras el participante se conecta a la llamada.
  - `Ringing`: el participante está sonando.
  - `Connected`: el participante se conecta a la llamada.
  - `Hold`: el participante está en espera.
  - `EarlyMedia`: anuncio que se reproduce antes de que un participante se conecte a la llamada.
  - `Disconnected`: estado final. El participante se desconecta de la llamada. Si el participante remoto pierde la conectividad de red, su estado cambia a `Disconnected` tras dos minutos.

- `callEndReason`: para saber por qué el participante dejó la llamada, compruebe la propiedad `callEndReason`:

  ```js
  const callEndReason = remoteParticipant.callEndReason;
  const callEndReasonCode = callEndReason.code // (number) code associated with the reason
  const callEndReasonSubCode = callEndReason.subCode // (number) subCode associated with the reason
  ```

- Estado `isMuted`: para averiguar si un participante remoto está silenciado, compruebe la propiedad `isMuted`. Devuelve `Boolean`.

  ```js
  const isMuted = remoteParticipant.isMuted;
  ```

- Estado `isSpeaking`: para averiguar si un participante remoto está hablando, compruebe la propiedad `isSpeaking`. Devuelve `Boolean`.

  ```js
  const isSpeaking = remoteParticipant.isSpeaking;
  ```

- `videoStreams`: para revisar todas las secuencias de vídeo que un participante determinado envía en esta llamada, compruebe la colección `videoStreams`: Contiene objetos `RemoteVideoStream`.

  ```js
  const videoStreams = remoteParticipant.videoStreams; // [RemoteVideoStream, ...]
  ```
- `displayName`: para obtener el nombre para mostrar de este participante remoto, revise la propiedad `displayName` que devuelve la cadena. 

  ```js
  const displayName = remoteParticipant.displayName;
  ```

### <a name="add-a-participant-to-a-call"></a>Incorporación de un participante a una llamada

Para agregar un participante (ya sea un usuario o un número de teléfono) a una llamada, puede usar `addParticipant`. Proporcione uno de los tipos de `Identifier`. Devuelve la instancia de `remoteParticipant`.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
const remoteParticipant = call.addParticipant(userIdentifier);
const remoteParticipant = call.addParticipant(pstnIdentifier, {alternateCallerId: '<Alternate Caller ID>'});
```

### <a name="remove-a-participant-from-a-call"></a>Eliminación de un participante de una llamada

Para quitar un participante (ya sea un usuario o un número de teléfono) de una llamada, puede invocar `removeParticipant`. Tiene que pasar uno de los tipos de `Identifier`. Esto se resolverá de manera asincrónica una vez que el participante se quite de la llamada. El participante también se quitará de la colección `remoteParticipants`.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
await call.removeParticipant(userIdentifier);
await call.removeParticipant(pstnIdentifier);
```

## <a name="render-remote-participant-video-streams"></a>Representación de secuencias de vídeo de participantes remotos

Para una lista de las secuencias de vídeo y de las secuencias de uso compartido de pantalla de participantes remotos, revise las colecciones `videoStreams`:

```js
const remoteVideoStream: RemoteVideoStream = call.remoteParticipants[0].videoStreams[0];
const streamType: MediaStreamType = remoteVideoStream.mediaStreamType;
```

Para representar `RemoteVideoStream`, se debe suscribir a un evento `isAvailableChanged`. Si la propiedad `isAvailable` cambia a `true`, un participante remoto envía una secuencia. Cuando esto suceda, cree una instancia de `VideoStreamRenderer` y, luego, cree una instancia de `VideoStreamRendererView` nueva a través del método `createView` asincrónico.  Después puede adjuntar `view.target` a cualquier elemento de la interfaz de usuario.

Cada vez que cambie la disponibilidad de una secuencia remota, puede elegir destruir todo el elemento `VideoStreamRenderer`, un elemento `VideoStreamRendererView` específico o mantenerlos, pero esto hará que el fotograma del vídeo se vea en blanco.

```js
function subscribeToRemoteVideoStream(remoteVideoStream: RemoteVideoStream) {
    let videoStreamRenderer: VideoStreamRenderer = new VideoStreamRenderer(remoteVideoStream);
    const displayVideo = () => {
        const view = await videoStreamRenderer.createView();
        htmlElement.appendChild(view.target);
    }
    remoteVideoStream.on('isAvailableChanged', async () => {
        if (remoteVideoStream.isAvailable) {
            displayVideo();
        } else {
            videoStreamRenderer.dispose();
        }
    });
    if (remoteVideoStream.isAvailable) {
        displayVideo();
    }
}
```

### <a name="remote-video-stream-properties"></a>Propiedades de secuencias de vídeo remotas

Las secuencias de vídeo remotas tienen las propiedades siguientes:

- `id`: el identificador de una secuencia de vídeo remota.

  ```js
  const id: number = remoteVideoStream.id;
  ```

- `mediaStreamType`: puede ser `Video` o `ScreenSharing`.

  ```js
  const type: MediaStreamType = remoteVideoStream.mediaStreamType;
  ```

- `isAvailable`: si el punto de conexión del participante remoto envía secuencias de manera activa.

  ```js
  const type: boolean = remoteVideoStream.isAvailable;
  ```

### <a name="videostreamrenderer-methods-and-properties"></a>Métodos y propiedades de VideoStreamRenderer

Cree una instancia de `VideoStreamRendererView` que se pueda adjuntar en la interfaz de usuario de la aplicación para representar la secuencia de vídeo remota, usar el método `createView()` asincrónico, se resuelve cuando la secuencia está lista para representarse y devuelve un objeto con la propiedad `target` que representa el elemento `video` que se puede anexar en cualquier parte del árbol DOM.

  ```js
  videoStreamRenderer.createView()
  ```

Elimine el `videoStreamRenderer` y todas las instancias de `VideoStreamRendererView` asociadas.

  ```js
  videoStreamRenderer.dispose()
  ```

### <a name="videostreamrendererview-methods-and-properties"></a>Métodos y propiedades de VideoStreamRenderer

Cuando crea un objeto `VideoStreamRendererView`, puede especificar las propiedades `scalingMode` y `isMirrored`. `scalingMode` puede ser `Stretch`, `Crop` o `Fit`. Si se especifica `isMirrored`, la secuencia representada se voltea verticalmente.

```js
const videoStreamRendererView: VideoStreamRendererView = await videoStreamRenderer.createView({ scalingMode, isMirrored });
```

Toda instancia de `VideoStreamRendererView` determinada tiene una propiedad `target` que representa la superficie de representación. Adjunte esta propiedad en la interfaz de usuario de la aplicación:

```js
htmlElement.appendChild(view.target);
```

Puede actualizar `scalingMode` mediante la llamada al método `updateScalingMode`:

```js
view.updateScalingMode('Crop')
```

## <a name="device-management"></a>Administración de dispositivos

En `deviceManager`, puede especificar dispositivos locales que puedan transmitir secuencias de audio y vídeo en una llamada. También le ayuda a solicitar permiso para acceder al micrófono y la cámara de otro usuario mediante la API nativa del explorador.

Puede llamar al método `callClient.getDeviceManager()` para acceder a `deviceManager`.

```js
const deviceManager = await callClient.getDeviceManager();
```

### <a name="get-local-devices"></a>Obtención de dispositivos locales

Para acceder a los dispositivos locales, puede usar métodos de enumeración en `deviceManager`.

```js
//  Get a list of available video devices for use.
const localCameras = await deviceManager.getCameras(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
const localMicrophones = await deviceManager.getMicrophones(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
const localSpeakers = await deviceManager.getSpeakers(); // [AudioDeviceInfo, AudioDeviceInfo...]
```

### <a name="set-the-default-microphone-and-speaker"></a>Establecimiento del micrófono o el altavoz predeterminados

En `deviceManager`, puede establecer un dispositivo predeterminado que usará para iniciar una llamada. Si no se establecen los valores predeterminados del cliente, Communication Services utiliza los valores predeterminados del sistema operativo.

```js
// Get the microphone device that is being used.
const defaultMicrophone = deviceManager.selectedMicrophone;

// Set the microphone device to use.
await deviceManager.selectMicrophone(localMicrophones[0]);

// Get the speaker device that is being used.
const defaultSpeaker = deviceManager.selectedSpeaker;

// Set the speaker device to use.
await deviceManager.selectSpeaker(localSpeakers[0]);
```

### <a name="local-camera-preview"></a>Vista previa de la cámara local

Puede usar `deviceManager` y `VideoStreamRenderer` para empezar a representar secuencias desde la cámara local. Esta secuencia no se enviará a los demás participantes, porque es una fuente de vista previa local.

```js
const cameras = await deviceManager.getCameras();
const camera = cameras[0];
const localCameraStream = new LocalVideoStream(camera);
const videoStreamRenderer = new VideoStreamRenderer(localCameraStream);
const view = await videoStreamRenderer.createView();
htmlElement.appendChild(view.target);

```

### <a name="request-permission-to-camera-and-microphone"></a>Solicitud de permiso para acceder a la cámara o micrófono

Pida al usuario que conceda permisos de cámara y micrófono:

```js
const result = await deviceManager.askDevicePermission({audio: true, video: true});
```

Esto se resuelve con un objeto que indica si se concedieron los permisos de `audio` y `video`:

```js
console.log(result.audio);
console.log(result.video);
```

## <a name="record-calls"></a>Registro de llamadas
> [!NOTE]
> Esta API se ofrece a los desarrolladores como versión preliminar y puede cambiar en función de los comentarios que recibamos. No utilice esta API en un entorno de producción. Para usar esta API, utilice la versión beta del SDK web de llamada de ACS.

La grabación de llamadas es una característica extendida de la API `Call` principal. Primero debe obtener el objeto de API de la característica de grabación:

```js
const callRecordingApi = call.api(Features.Recording);
```

Después, para comprobar si se está grabando la llamada, inspeccione la propiedad `isRecordingActive` de `callRecordingApi`. Devuelve `Boolean`.

```js
const isResordingActive = callRecordingApi.isRecordingActive;
```

También puede suscribirse a los cambios de grabación:

```js
const isRecordingActiveChangedHandler = () => {
  console.log(callRecordingApi.isRecordingActive);
};

callRecordingApi.on('isRecordingActiveChanged', isRecordingActiveChangedHandler);

```

## <a name="transfer-calls"></a>Transferencia de llamadas
> [!NOTE]
> Esta API se ofrece a los desarrolladores como versión preliminar y puede cambiar en función de los comentarios que recibamos. No utilice esta API en un entorno de producción. Para usar esta API, utilice la versión beta del SDK web de llamada de ACS.

La trasferencia de llamadas es una característica extendida de la API `Call` principal. Primero debe obtener el objeto de API de la característica de transferencia:

```js
const callTransferApi = call.api(Features.Transfer);
```

Las transferencias de llamadas implican tres entidades:

- *Transmitente*: la persona que inicia la solicitud de transferencia.
- *Adquirente*: la persona que se va a transferir.
- *Destino de la transferencia*: la persona a la que se transfiere.

Las transferencias siguen estos pasos:

1. Ya hay una llamada conectada entre el *transmitente* y el *adquirente*. El *transmitente* decide transferir la llamada del *adquirente* al *destino de la transferencia*.
1. El *transmitente* llama a `transfer` API.
1. El *adquirente* decide si `accept` o `reject` la solicitud de transferencia al *destino de la transferencia* mediante un evento `transferRequested`.
1. El *destino de la transferencia* recibe una llamada entrante solo si el *adquirente* acepta la solicitud de transferencia.

Para transferir la llamada actual, puede usar `transfer` API. `transfer` toma el objeto `transferCallOptions` opcional, que permite establecer la marca `disableForwardingAndUnanswered`:

- `disableForwardingAndUnanswered = false`: si el *destino de la transferencia* no responde a la llamada de transferencia, la transferencia sigue las configuraciones de reenvío y sin respuesta del *destino de la transferencia*.
- `disableForwardingAndUnanswered = true`: si el *destino de la transferencia* no responde a la llamada de transferencia, el intento de transferencia finalizará.

```js
// transfer target can be an ACS user
const id = { communicationUserId: <ACS_USER_ID> };
```

```js
// call transfer API
const transfer = callTransferApi.transfer({targetParticipant: id});
```

`transfer` API le permite suscribirse a los eventos `transferStateChanged` y `transferRequested`. Un evento `transferRequested` procede de una instancia de `call`, un evento `transferStateChanged` y la transferencia `state` y `error` provienen de una instancia de `transfer`.

```js
// transfer state
const transferState = transfer.state; // None | Transferring | Transferred | Failed

// to check the transfer failure reason
const transferError = transfer.error; // transfer error code that describes the failure if a transfer request failed
```

El *adquirente* puede aceptar o rechazar la solicitud de transferencia iniciada por el *transmitente* en el evento `transferRequested` a través de `accept()` o `reject()` en `transferRequestedEventArgs`. Puede acceder a la información de `targetParticipant` y a los métodos `accept` y `reject` en `transferRequestedEventArgs`.

```js
// Transferee to accept the transfer request
callTransferApi.on('transferRequested', args => {
  args.accept();
});

// Transferee to reject the transfer request
callTransferApi.on('transferRequested', args => {
  args.reject();
});
```

## <a name="learn-about-eventing-models"></a>Más información sobre los modelos de eventos

Inspeccione los valores actuales y suscríbase a los eventos de actualización para los valores futuros.

### <a name="properties"></a>Propiedades

```js
// Inspect the current value
console.log(object.property);

// Subscribe to value updates
object.on('propertyChanged', () => {
    // Inspect new value
    console.log(object.property)
});

// Unsubscribe from updates:
object.off('propertyChanged', () => {});



// Example for inspecting a call state
console.log(call.state);
call.on('stateChanged', () => {
    console.log(call.state);
});
call.off('stateChanged', () => {});
```

### <a name="collections"></a>Colecciones

```js
// Inspect the current collection
object.collection.forEach(v => {
    console.log(v);
});

// Subscribe to collection updates
object.on('collectionUpdated', e => {
    // Inspect new values added to the collection
    e.added.forEach(v => {
        console.log(v);
    });
    // Inspect values removed from the collection
    e.removed.forEach(v => {
        console.log(v);
    });
});

// Unsubscribe from updates:
object.off('collectionUpdated', () => {});

// Example for subscribing to remote participants and their video streams
call.remoteParticipants.forEach(p => {
    subscribeToRemoteParticipant(p);
})

call.on('remoteParticipantsUpdated', e => {
    e.added.forEach(p => { subscribeToRemoteParticipant(p) })
    e.removed.forEach(p => { unsubscribeFromRemoteParticipant(p) })
});

function subscribeToRemoteParticipant(p) {
    console.log(p.state);
    p.on('stateChanged', () => { console.log(p.state); });
    p.videoStreams.forEach(v => { subscribeToRemoteVideoStream(v) });
    p.on('videoStreamsUpdated', e => { e.added.forEach(v => { subscribeToRemoteVideoStream(v) }) })
}
```
