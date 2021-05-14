---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 27516e5f0a75fa02804c8865c876044b48c4e729
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108313621"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Un recurso de Azure Communication Services implementado. [Creación de un recurso de Communication Services](../../create-communication-resource.md).
- Un token de acceso de usuario para habilitar el cliente de llamada. [Obtenga un token de acceso de usuario](../../access-tokens.md).
- Opcional: siga la guía de inicio rápido [Incorporación de llamadas de voz a la aplicación](../getting-started-with-calling.md).

## <a name="set-up-your-system"></a>Configuración del sistema

### <a name="create-the-xcode-project"></a>Creación del proyecto de Xcode

En Xcode, cree un proyecto de iOS nuevo y seleccione la plantilla **Aplicación de vista única**. En este inicio rápido se usa el [marco SwiftUI](https://developer.apple.com/xcode/swiftui/), por lo que debe establecer el **lenguaje** en **Swift** y la **interfaz de usuario** en **SwiftUI**. 

No va a crear pruebas unitarias ni pruebas de interfaz de usuario durante este inicio rápido. No dude en borrar los cuadros de texto para **incluir pruebas unitarias** y para **incluir pruebas de IU**.

:::image type="content" source="../media/ios/xcode-new-ios-project.png" alt-text="Captura de pantalla que muestra la ventana para crear un proyecto en Xcode.":::

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Instalación del paquete y las dependencias con CocoaPods

1. Cree un podfile para la aplicación, de la siguiente manera:

   ```
   platform :ios, '13.0'
   use_frameworks!
   target 'AzureCommunicationCallingSample' do
     pod 'AzureCommunicationCalling', '~> 1.0.0-beta.8'
     pod 'AzureCommunication', '~> 1.0.0-beta.8'
     pod 'AzureCore', '~> 1.0.0-beta.8'
   end
   ```

2. Ejecute `pod install`.
3. Abra el archivo `.xcworkspace` con Xcode.

### <a name="request-access-to-the-microphone"></a>Solicitud de acceso al micrófono

Para acceder al micrófono del dispositivo, debe actualizar la lista de propiedades de información de la aplicación con el elemento `NSMicrophoneUsageDescription`. Establezca el valor asociado al elemento `string` que se incluirá en el cuadro de diálogo que el sistema usa para solicitar acceso al usuario.

Haga clic con el botón derecho en la entrada `Info.plist` del árbol del proyecto y seleccione **Abrir como** > **Código fuente**. Agregue las líneas siguientes a la sección `<dict>` de nivel superior y guarde el archivo.

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Need microphone access for VOIP calling.</string>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Abra el archivo *ContentView.swift* del proyecto y agregue una declaración `import` en la parte superior del archivo para importar la biblioteca `AzureCommunicationCalling`. Además, importe `AVFoundation`. Lo necesitará para las solicitudes de permiso de audio en el código.

```swift
import AzureCommunicationCalling
import AVFoundation
```

## <a name="learn-the-object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de llamadas de Azure Communication Services para iOS.

> [!NOTE]
> En esta guía de inicio rápido se usa la versión 1.0.0-beta.8 del SDK de llamadas.


| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| `CallClient` | `CallClient` es el punto de entrada principal al SDK de llamadas.|
| `CallAgent` | `CallAgent` se utiliza para iniciar y administrar las llamadas. |
| `CommunicationTokenCredential` | `CommunicationTokenCredential` se usa como la credencial del token para crear una instancia de `CallAgent`.| 
| `CommunicationIdentifier` | `CommunicationIdentifier` se usa para representar la identidad del usuario. La identidad puede ser `CommunicationUserIdentifier`, `PhoneNumberIdentifier` o `CallingApplication`. |

> [!NOTE]
> Cuando la aplicación implementa delegados de eventos, tiene que mantener una referencia segura a los objetos que requieren suscripciones a eventos. Por ejemplo, cuando se devuelve un objeto `RemoteParticipant` al invocar el método `call.addParticipant` y la aplicación establece que el delegado escuche en `RemoteParticipantDelegate`, la aplicación debe contener una referencia segura al objeto `RemoteParticipant`. De lo contrario, si se recopila este objeto, el delegado generará una excepción irrecuperable cuando el SDK de llamadas intente invocar al objeto.

## <a name="initialize-callagent"></a>Inicialización de CallAgent

Para crear una instancia de `CallAgent` a partir de `CallClient`, debe usar el método `callClient.createCallAgent`, que devuelve de manera asincrónica un objeto `CallAgent` después de que se inicializa.

Para crear un cliente de llamada, debe pasar un objeto `CommunicationTokenCredential`.

```swift

import AzureCommunication

let tokenString = "token_string"
var userCredential: CommunicationTokenCredential?
var userCredential: CommunicationTokenCredential?
   do {
       userCredential = try CommunicationTokenCredential(with: CommunicationTokenRefreshOptions(initialToken: token, 
                                                                     refreshProactively: true,
                                                                     tokenRefresher: self.fetchTokenSync))
   } catch {
       return
}

// tokenProvider needs to be implemented by Contoso, which fetches a new token
public func fetchTokenSync(then onCompletion: TokenRefreshOnCompletion) {
    let newToken = self.tokenProvider!.fetchNewToken()
    onCompletion(newToken, nil)
}
```

Pase el objeto `CommunicationTokenCredential` que ha creado a `CallClient` y establezca el nombre para mostrar.

```swift

callClient = CallClient()
let callAgentOptions:CallAgentOptions = CallAgentOptions()!
options.displayName = " iOS User"

callClient?.createCallAgent(userCredential: userCredential!,
    options: callAgentOptions) { (callAgent, error) in
        if error == nil {
            print("Create agent succeeded")
            self.callAgent = callAgent
        } else {
            print("Create agent failed")
        }
})

```

## <a name="place-an-outgoing-call"></a>Realización de una llamada saliente

Para crear e iniciar una llamada, debe llamar a una de las API en `CallAgent` y proporcionar la identidad de Communication Services de un usuario que haya aprovisionado mediante el SDK de administración de Communication Services.

La creación y el inicio de la llamada son sincrónicos. Recibirá una instancia de la llamada que le permitirá suscribirse a todos los eventos de la llamada.

### <a name="place-a-11-call-to-a-user-or-a-1n-call-with-users-and-pstn"></a>Realice una llamada de uno a uno a un usuario o una llamada de uno a varios con usuarios y RTC

```swift

let callees = [CommunicationUser(identifier: 'UserId')]
let oneToOneCall = self.callAgent.call(participants: callees, options: StartCallOptions())

```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Realice una llamada de uno a varios con usuarios y RTC
Para realizar la llamada con RTC, debe especificar el número de teléfono adquirido con Communication Services.

```swift

let pstnCallee = PhoneNumberIdentifier(phoneNumber: '+1999999999')
let callee = CommunicationUserIdentifier(identifier: 'UserId')
let groupCall = self.callAgent.call(participants: [pstnCallee, callee], options: StartCallOptions())

```

### <a name="place-a-11-call-with-video"></a>Realización de una llamada 1:1 con vídeo
Para obtener una instancia del administrador de dispositivos, consulte la sección acerca de la [administración de dispositivos](#manage-devices).

```swift

let camera = self.deviceManager!.cameras!.first
let localVideoStream = LocalVideoStream(camera: camera)
let videoOptions = VideoOptions(localVideoStream: localVideoStream)

let startCallOptions = StartCallOptions()
startCallOptions?.videoOptions = videoOptions

let callee = CommunicationUserIdentifier(identifier: 'UserId')
let call = self.callAgent?.call(participants: [callee], options: startCallOptions)

```

### <a name="join-a-group-call"></a>Unión a una llamada grupal
Para unirse a una llamada, debe llamar a una de las API en `CallAgent`.

```swift

let groupCallLocator = GroupCallLocator(groupId: UUID(uuidString: "uuid_string"))!
let call = self.callAgent?.join(with: groupCallLocator, joinCallOptions: JoinCallOptions())

```

### <a name="subscribe-to-an-incoming-call"></a>Suscripción a una llamada entrante
Suscríbase a un evento de llamada entrante.

```
final class IncomingCallHandler: NSObject, CallAgentDelegate, IncomingCallDelegate
{
    // Event raised when there is an incoming call
    public func onIncomingCall(_ callAgent: CallAgent!, incomingcall: IncomingCall!) {
        self.incomingCall = incomingcall
        // Subscribe to get OnCallEnded event
        self.incomingCall?.delegate = self
    }

    // Event raised when incoming call was not answered
    public func onCallEnded(_ incomingCall: IncomingCall!, args: PropertyChangedEventArgs!) {
        self.incomingCall = nil
    }
}
```

### <a name="accept-an-incoming-call"></a>Aceptar una llamada entrante
Para aceptar una llamada, llame al método `accept` en un objeto de llamada. Establezca un delegado para `CallAgent`.

```swift
final class CallHandler: NSObject, CallAgentDelegate
{
    public var incomingCall: Call?
 
    public func onCallsUpdated(_ callAgent: CallAgent!, args: CallsUpdatedEventArgs!) {
        if let incomingCall = args.addedCalls?.first(where: { $0.isIncoming }) {
            self.incomingCall = incomingCall
        }
    }
}

let firstCamera: VideoDeviceInfo? = self.deviceManager!.cameras!.first
let localVideoStream = LocalVideoStream(camera: firstCamera)
let acceptCallOptions = AcceptCallOptions()
acceptCallOptions!.videoOptions = VideoOptions(localVideoStream:localVideoStream!)
if let incomingCall = CallHandler().incomingCall {
   incomingCall.accept(options: acceptCallOptions) { (call, error) in
               if error == nil {
                   print("Incoming call accepted")
               } else {
                   print("Failed to accept incoming call")
               }
           }
} else {
   print("No incoming call found to accept")
}
```

## <a name="set-up-push-notifications"></a>Configuración de las notificaciones de inserción

Una notificación de inserción en el dispositivo móvil es la notificación emergente que recibe en el dispositivo móvil. En el caso de las llamadas, nos centraremos en las notificaciones de inserción de VoIP (voz sobre IP). 

En las secciones siguientes se describe cómo registrarse, controlar y anular el registro de las notificaciones de inserción. Antes de iniciar estas tareas, complete estos requisitos previos:

1. En Xcode, vaya a **Signing & Capabilities** (Firma y funcionalidades). Para agregar una funcionalidad, seleccione **+ Capability** (+ Funcionalidad) y, a continuación, seleccione **Push Notifications** (Notificaciones de inserción).
2. Para agregar otra funcionalidad, seleccione **+ Capability** (+ Funcionalidad) y, a continuación, seleccione **Background Modes** (Modos en segundo plano).
3. En **Background Modes** (Modos en segundo plano), seleccione las casillas **Voice over IP** (Voz sobre IP) y **Remote notifications** (Notificaciones remotas).

:::image type="content" source="../media/ios/xcode-push-notification.png" alt-text="Captura de pantalla que muestra cómo agregar funcionalidades en Xcode." lightbox="../media/ios/xcode-push-notification.png":::

### <a name="register-for-push-notifications"></a>Registro de notificaciones push

A fin de registrarse para recibir notificaciones de inserción, llame a `registerPushNotification()` en una instancia de `CallAgent` con un token de registro de dispositivo.

El registro de las notificaciones de inserción se debe producir después de una inicialización correcta. Cuando se destruya el objeto `callAgent`, se llamará a `logout`, lo que anulará automáticamente el registro para notificaciones de inserción.


```swift

let deviceToken: Data = pushRegistry?.pushToken(for: PKPushType.voIP)
callAgent.registerPushNotifications(deviceToken: deviceToken) { (error) in
    if(error == nil) {
        print("Successfully registered to push notification.")
    } else {
        print("Failed to register push notification.")
    }
}

```

### <a name="handle-push-notifications"></a>Control de las notificaciones de inserción
Para recibir notificaciones de inserción de llamadas entrantes, llame a `handlePushNotification()` en una instancia de `CallAgent` con una carga de diccionario.

```swift

let callNotification = IncomingCallInformation.from(payload: pushPayload?.dictionaryPayload)

callAgent.handlePush(notification: callNotification) { (error) in
    if (error != nil) {
        print("Handling of push notification failed")
    } else {
        print("Handling of push notification was successful")
    }
}

```
### <a name="unregister-push-notifications"></a>Anulación del registro de notificaciones push

Las aplicaciones pueden anular el registro de notificaciones push en cualquier momento. Solo debe llamar al método `unregisterPushNotification` en `CallAgent`.

> [!NOTE]
> El registro para que las aplicaciones dejen de recibir notificaciones de inserción no se anula de manera automática al cerrar la sesión.

```swift

callAgent.unregisterPushNotifications { (error) in
    if (error != nil) {
        print("Unregister of push notification failed, please try again")
    } else {
        print("Unregister of push notification was successful")
    }
}

```

## <a name="perform-mid-call-operations"></a>Realización de operaciones durante la llamada

Durante una llamada, puede realizar varias operaciones para administrar la configuración relacionada con el vídeo y el audio.

### <a name="mute-and-unmute"></a>Silencio y reactivación del sonido

Para silenciar o reactivar el sonido del punto de conexión local, puede usar las API asincrónicas `mute` y `unmute`.

```swift
call!.mute { (error) in
    if error == nil {
        print("Successfully muted")
    } else {
        print("Failed to mute")
    }
}

```

Use el código siguiente para silenciar el punto de conexión local de forma asincrónica.

```swift
call!.unmute { (error) in
    if error == nil {
        print("Successfully un-muted")
    } else {
        print("Failed to unmute")
    }
}
```

### <a name="start-and-stop-sending-local-video"></a>Inicio y detención del envío de vídeo local

Para empezar a enviar vídeo local a otros participantes de la llamada, use la API `startVideo` y pase el elemento `localVideoStream` con el elemento `camera`.

```swift

let firstCamera: VideoDeviceInfo? = self.deviceManager!.cameras!.first
let localVideoStream = LocalVideoStream(camera: firstCamera)

call!.startVideo(stream: localVideoStream) { (error) in
    if (error == nil) {
        print("Local video started successfully")
    } else {
        print("Local video failed to start")
    }
}

```

Una vez que se inicia el envío de vídeo, la instancia de `LocalVideoStream` se agrega a la colección `localVideoStreams` en una instancia de llamada.

```swift

call.localVideoStreams[0]

```

Para detener el vídeo local, pase la instancia de `localVideoStream` devuelta a partir de la invocación de `call.startVideo`. Se trata de una acción asincrónica.

```swift

call!.stopVideo(stream: localVideoStream) { (error) in
    if (error == nil) {
        print("Local video stopped successfully")
    } else {
        print("Local video failed to stop")
    }
}

```

## <a name="manage-remote-participants"></a>Administración de participantes remotos

El tipo `RemoteParticipant` representa a todos los participantes remotos y estos están disponibles mediante la colección `remoteParticipants` en una instancia de llamada.

### <a name="list-participants-in-a-call"></a>Lista de los participantes en una llamada

```swift

call.remoteParticipants

```

### <a name="get-remote-participant-properties"></a>Obtención de las propiedades de los participantes remotos

```swift

// [RemoteParticipantDelegate] delegate - an object you provide to receive events from this RemoteParticipant instance
var remoteParticipantDelegate = remoteParticipant.delegate

// [CommunicationIdentifier] identity - same as the one used to provision a token for another user
var identity = remoteParticipant.identity

// ParticipantStateIdle = 0, ParticipantStateEarlyMedia = 1, ParticipantStateConnecting = 2, ParticipantStateConnected = 3, ParticipantStateOnHold = 4, ParticipantStateInLobby = 5, ParticipantStateDisconnected = 6
var state = remoteParticipant.state

// [Error] callEndReason - reason why participant left the call, contains code/subcode/message
var callEndReason = remoteParticipant.callEndReason

// [Bool] isMuted - indicating if participant is muted
var isMuted = remoteParticipant.isMuted

// [Bool] isSpeaking - indicating if participant is currently speaking
var isSpeaking = remoteParticipant.isSpeaking

// RemoteVideoStream[] - collection of video streams this participants has
var videoStreams = remoteParticipant.videoStreams // [RemoteVideoStream, RemoteVideoStream, ...]

```

### <a name="add-a-participant-to-a-call"></a>Incorporación de un participante a una llamada

Para agregar un participante a una llamada (ya sea un usuario o un número de teléfono), puede invocar a `addParticipant`. Este comando devolverá de manera sincrónica la instancia de un participante remoto.

```swift

let remoteParticipantAdded: RemoteParticipant = call.add(participant: CommunicationUserIdentifier(identifier: "userId"))

```

### <a name="remove-a-participant-from-a-call"></a>Eliminación de un participante de una llamada
Para quitar un participante de una llamada (ya sea un usuario o un número de teléfono), puede invocar a la API `removeParticipant`. Esto se resolverá de manera asincrónica.

```swift

call!.remove(participant: remoteParticipantAdded) { (error) in
    if (error == nil) {
        print("Successfully removed participant")
    } else {
        print("Failed to remove participant")
    }
}

```

## <a name="render-remote-participant-video-streams"></a>Representación de secuencias de vídeo de participantes remotos

Los participantes remotos pueden iniciar el uso compartido de pantalla o vídeo durante una llamada.

### <a name="handle-video-sharing-or-screen-sharing-streams-of-remote-participants"></a>Control de las transmisiones de uso compartido de vídeo o de pantalla de participantes remotos

Para enumerar las transmisiones de los participantes remotos, inspeccione las colecciones `videoStreams`.

```swift

var remoteParticipantVideoStream = call.remoteParticipants[0].videoStreams[0]

```

### <a name="get-remote-video-stream-properties"></a>Obtención de las propiedades de las transmisiones de vídeo remotas

```swift

var type: MediaStreamType = remoteParticipantVideoStream.type // 'MediaStreamTypeVideo'

var isAvailable: Bool = remoteParticipantVideoStream.isAvailable // indicates if remote stream is available

var id: Int = remoteParticipantVideoStream.id // id of remoteParticipantStream

```

### <a name="render-remote-participant-streams"></a>Representación de las transmisiones de participantes remotos

Para iniciar la representación de las transmisiones de los participantes remotos, use el código siguiente.

```swift

let renderer: Renderer? = Renderer(remoteVideoStream: remoteParticipantVideoStream)
let targetRemoteParticipantView: RendererView? = renderer?.createView(with: RenderingOptions(scalingMode: ScalingMode.crop))
// To update the scaling mode later
targetRemoteParticipantView.update(scalingMode: ScalingMode.fit)

```

### <a name="get-remote-video-renderer-methods-and-properties"></a>Obtención de los métodos y propiedades del representador de vídeo

```swift
// [Synchronous] dispose() - dispose renderer and all `RendererView` associated with this renderer. To be called when you have removed all associated views from the UI.
remoteVideoRenderer.dispose()
```

## <a name="manage-devices"></a>Administración de dispositivos

`DeviceManager` permite enumerar los dispositivos locales que se pueden usar en una llamada para transmitir secuencias de audio o vídeo. También permite solicitar permiso a un usuario para acceder al micrófono o a la cámara. Puede acceder a `deviceManager` en el objeto `callClient`.

```swift

self.callClient!.getDeviceManager { (deviceManager, error) in
        if (error == nil) {
            print("Got device manager instance")
            self.deviceManager = deviceManager
        } else {
            print("Failed to get device manager instance")
        }
    }
```

### <a name="enumerate-local-devices"></a>Enumeración de los dispositivos locales

Para acceder a los dispositivos locales, puede usar métodos de enumeración en el administrador de dispositivos. La enumeración es una acción sincrónica.

```swift
// enumerate local cameras
var localCameras = deviceManager.cameras! // [VideoDeviceInfo, VideoDeviceInfo...]
// enumerate local cameras
var localMicrophones = deviceManager.microphones! // [AudioDeviceInfo, AudioDeviceInfo...]
// enumerate local cameras
var localSpeakers = deviceManager.speakers! // [AudioDeviceInfo, AudioDeviceInfo...]
``` 

### <a name="set-the-default-microphone-or-speaker"></a>Establecimiento del micrófono o el altavoz predeterminados

Puede usar el administrador de dispositivos para establecer un dispositivo predeterminado que se usará cuando se inicie una llamada. Si no se establecen los valores predeterminados de la pila, Communication Services revertirá a los valores predeterminados del sistema operativo.

```swift
// get first microphone
var firstMicrophone = self.deviceManager!.cameras!.first
// [Synchronous] set microphone
deviceManager.setMicrophone(microphoneDevice: firstMicrophone)
// get first speaker
var firstSpeaker = self.deviceManager!.speakers!
// [Synchronous] set speaker
deviceManager.setSpeaker(speakerDevice: firstSpeaker)
```

### <a name="get-a-local-camera-preview"></a>Obtención de una vista previa de la cámara local

Puede usar `Renderer` para empezar a representar una secuencia desde su cámara local. Esta secuencia no se enviará a los demás participantes, porque es una fuente de vista previa local. Se trata de una acción asincrónica.

```swift

let camera: VideoDeviceInfo = self.deviceManager!.getCameraList()![0]
let localVideoStream: LocalVideoStream = LocalVideoStream(camera: camera)
let renderer: Renderer = Renderer(localVideoStream: localVideoStream)
self.view = try renderer!.createView()

```

### <a name="get-local-camera-preview-properties"></a>Obtención de las propiedades de la vista previa de la cámara local

El representador tiene un conjunto de propiedades y métodos que le permiten controlar la representación.

```swift

// Constructor can take in LocalVideoStream or RemoteVideoStream
let localRenderer = Renderer(localVideoStream:localVideoStream)
let remoteRenderer = Renderer(remoteVideoStream:remoteVideoStream)

// [StreamSize] size of the rendering view
localRenderer.size

// [RendererDelegate] an object you provide to receive events from this Renderer instance
localRenderer.delegate

// [Synchronous] create view
try! localRenderer.createView()

// [Synchronous] create view with rendering options
try! localRenderer.createView(with: RenderingOptions(scalingMode: ScalingMode.fit))

// [Synchronous] dispose rendering view
localRenderer.dispose()

```

## <a name="subscribe-to-notifications"></a>Suscribirse a notificaciones

Puede suscribirse a la mayoría de las propiedades y colecciones que se van a notificar al cambiar los valores.

### <a name="properties"></a>Propiedades
Para suscribirse a eventos `property changed`, use el código siguiente.

```swift
call.delegate = self
// Get the property of the call state by getting on the call's state member
public func onCallStateChanged(_ call: Call!,
                               args: PropertyChangedEventArgs!)
{
    print("Callback from SDK when the call state changes, current state: " + call.state.rawValue)
}

 // to unsubscribe
 self.call.delegate = nil

```

### <a name="collections"></a>Colecciones
Para suscribirse a eventos `collection updated`, use el código siguiente.

```swift
call.delegate = self
// Collection contains the streams that were added or removed only
public func onLocalVideoStreamsChanged(_ call: Call!,
                                       args: LocalVideoStreamsUpdatedEventArgs!)
{
    print(args.addedStreams.count)
    print(args.removedStreams.count)
}
// to unsubscribe
self.call.delegate = nil
```
