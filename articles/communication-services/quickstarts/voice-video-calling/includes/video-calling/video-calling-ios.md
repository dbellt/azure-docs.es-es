---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: a302cdacb7943e81f884b7f83a03c8b72ce3e197
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111560721"
---
Puede empezar a usar Azure Communication Services con Calling SDK de Azure Communication Services para agregar llamadas individuales de vídeo a la aplicación. Aprenderá a iniciar videollamadas y a responder a ellas con Calling SDK de Azure Communication Services para iOS.

> [!NOTE]
> En este documento se usa la versión 1.0.0 de Calling SDK.

> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-ios-quickstarts/tree/main/Add%20Video%20Calling)

## <a name="prerequisites"></a>Requisitos previos
- Obtenga una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Mac con [Xcode](https://developer.apple.com/xcode/), junto con un certificado de desarrollador válido instalado en el Llavero.
- Cree un recurso activo de Communication Services. [Cree un recurso de Communication Services](../../../create-communication-resource.md?tabs=windows&pivots=platform-azp).
- Cree un token de acceso de usuario para crear una instancia del cliente de llamada. [Creación y administración de tokens de acceso](../../../access-tokens.md?pivots=programming-language-csharp).

## <a name="setting-up"></a>Instalación
### <a name="creating-the-xcode-project"></a>Creación del proyecto de Xcode
En Xcode, cree un nuevo proyecto de iOS y seleccione la plantilla Aplicación de una vista. En este tutorial se usa el [marco SwiftUI](https://developer.apple.com/xcode/swiftui/), por lo que debe establecer el lenguaje en Swift y la interfaz de usuario en SwiftUI. Durante este inicio rápido, no va a crear pruebas. No dude en desactivar Incluir pruebas.

:::image type="content" source="../../media/ios/xcode-new-ios-project.png" alt-text="Captura de pantalla que muestra la ventana Nuevo proyecto en Xcode.":::

### <a name="installing-cocoapods"></a>Instalación de CocoaPods
Utilice esta guía para [instalar CocoaPods](https://guides.cocoapods.org/using/getting-started.html) en su Mac. 

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Instalación del paquete y las dependencias con CocoaPods
1. Si desea crear un podfile para la aplicación, abra el terminal, vaya a la carpeta del proyecto y ejecute el archivo init del pod.

2. Agregue el siguiente código al podfile y guárdelo:

```
platform :ios, '13.0'
use_frameworks!

target 'VideoCallingQuickstart' do
  pod 'AzureCommunicationCalling', '~> 1.0.0'
end
```

3. Ejecute la instalación del pod.

4. Abra .xcworkspace con Xcode.


### <a name="request-access-to-the-microphone-and-camera"></a>Solicite acceso al micrófono y a la cámara.
Para acceder al micrófono y a la cámara del dispositivo, debe actualizar la lista de propiedades de información de la aplicación con `NSMicrophoneUsageDescription` y `NSCameraUsageDescription`. Puede establecer el valor asociado a una cadena que se va a incluir en el cuadro de diálogo empleado por el sistema para solicitar acceso al usuario.

Haga clic con el botón derecho en la entrada `Info.plist` del árbol del proyecto y seleccione Open As > Source Code (Abrir como > Código fuente). Agregue las líneas siguientes a la sección `<dict>` de nivel superior y guarde el archivo.

```XML
<key>NSMicrophoneUsageDescription</key>
<string>Need microphone access for VOIP calling.</string>
<key>NSCameraUsageDescription</key>
<string>Need camera access for video calling</string>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación
Abra el archivo `ContentView.swift` del proyecto y agregue una declaración de importación en la parte superior del archivo para importar la biblioteca `AzureCommunicationCalling` y `AVFoundation`. AVFoundation se usa para capturar el permiso de audio del código.

```Swift
import AzureCommunicationCalling
import AVFoundation
```

## <a name="object-model"></a>Modelo de objetos
Las siguientes clases e interfaces controlan algunas de las características principales del SDK de llamadas de Azure Communication Services para iOS.

| Nombre      | Descripción | 
| :---        |    :----   |
| CallClient  | CallClient es el punto de entrada principal al SDK de llamadas.      |
| CallAgent  | CallAgent se usa para iniciar y administrar llamadas.        |
| CommunicationTokenCredential| CommunicationTokenCredential se usa como la credencial de token para crear una instancia de CallAgent.    |
| CommunicationIdentifier | CommunicationIdentifier se usa para representar la identidad del usuario, que puede ser una de las opciones siguientes: CommunicationUserIdentifier/PhoneNumberIdentifier/CallingApplication.        |

## <a name="create-the-call-agent"></a>Creación del agente de llamadas
Reemplace la implementación del struct ContentView por algunos sencillos controles de interfaz de usuario que permitan a los usuarios iniciar y finalizar una llamada. En este inicio rápido, asociaremos la lógica de negocios a estos controles.

```Swift
struct ContentView: View {
    @State var callee: String = ""
    @State var callClient: CallClient?
    @State var callAgent: CallAgent?
    @State var call: Call?
    @State var deviceManager: DeviceManager?
    @State var localVideoStream:[LocalVideoStream]?
    @State var incomingCall: IncomingCall?
    @State var sendingVideo:Bool = false
    @State var errorMessage:String = "Unknown"

    @State var remoteVideoStreamData:[Int32:RemoteVideoStreamData] = [:]
    @State var previewRenderer:VideoStreamRenderer? = nil
    @State var previewView:RendererView? = nil
    @State var remoteRenderer:VideoStreamRenderer? = nil
    @State var remoteViews:[RendererView] = []
    @State var remoteParticipant: RemoteParticipant?
    @State var remoteVideoSize:String = "Unknown"
    @State var isIncomingCall:Bool = false
    
    @State var callObserver:CallObserver?
    @State var remoteParticipantObserver:RemoteParticipantObserver?

    var body: some View {
        NavigationView {
            ZStack{
                Form {
                    Section {
                        TextField("Who would you like to call?", text: $callee)
                        Button(action: startCall) {
                            Text("Start Call")
                        }.disabled(callAgent == nil)
                        Button(action: endCall) {
                            Text("End Call")
                        }.disabled(call == nil)
                        Button(action: toggleLocalVideo) {
                            HStack {
                                Text(sendingVideo ? "Turn Off Video" : "Turn On Video")
                            }
                        }
                    }
                }
                // Show incoming call banner
                if (isIncomingCall) {
                    HStack() {
                        VStack {
                            Text("Incoming call")
                                .padding(10)
                                .frame(maxWidth: .infinity, alignment: .topLeading)
                        }
                        Button(action: answerIncomingCall) {
                            HStack {
                                Text("Answer")
                            }
                            .frame(width:80)
                            .padding(.vertical, 10)
                            .background(Color(.green))
                        }
                        Button(action: declineIncomingCall) {
                            HStack {
                                Text("Decline")
                            }
                            .frame(width:80)
                            .padding(.vertical, 10)
                            .background(Color(.red))
                        }
                    }
                    .frame(maxWidth: .infinity, alignment: .topLeading)
                    .padding(10)
                    .background(Color.gray)
                }
                ZStack{
                    VStack{
                        ForEach(remoteViews, id:\.self) { renderer in
                            ZStack{
                                VStack{
                                    RemoteVideoView(view: renderer)
                                        .frame(width: .infinity, height: .infinity)
                                        .background(Color(.lightGray))
                                }
                            }
                            Button(action: endCall) {
                                Text("End Call")
                            }.disabled(call == nil)
                            Button(action: toggleLocalVideo) {
                                HStack {
                                    Text(sendingVideo ? "Turn Off Video" : "Turn On Video")
                                }
                            }
                        }
                        
                    }.frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .topLeading)
                    VStack{
                        if(sendingVideo)
                        {
                            VStack{
                                PreviewVideoStream(view: previewView!)
                                    .frame(width: 135, height: 240)
                                    .background(Color(.lightGray))
                            }
                        }
                    }.frame(maxWidth:.infinity, maxHeight:.infinity,alignment: .bottomTrailing)
                }
            }
     .navigationBarTitle("Video Calling Quickstart")
        }.onAppear{
            // Authenticate the client
            
            // Initialize the CallAgent and access Device Manager
            
            // Ask for permissions
        }
    }
}

//Functions and Observers

struct PreviewVideoStream: UIViewRepresentable {
    let view:RendererView
    func makeUIView(context: Context) -> UIView {
        return view
    }
    func updateUIView(_ uiView: UIView, context: Context) {}
}

struct RemoteVideoView: UIViewRepresentable {
    let view:RendererView
    func makeUIView(context: Context) -> UIView {
        return view
    }
    func updateUIView(_ uiView: UIView, context: Context) {}
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

### <a name="authenticate-the-client"></a>Autenticar el cliente
Para poder inicializar una instancia de CallAgent, necesitamos un token de acceso de usuario que nos permita realizar y recibir llamadas. Agregue el código siguiente a la devolución de llamada `onAppear` en `ContentView.swift`:

```Swift
var userCredential: CommunicationTokenCredential?
do {
    userCredential = try CommunicationTokenCredential(token: "<USER ACCESS TOKEN>")
} catch {
    print("ERROR: It was not possible to create user credential.")
    return
}
```

Reemplace `<USER ACCESS TOKEN>` por un token de acceso de usuario válido para el recurso. Consulte la documentación relativa al [token de acceso de usuario](../../../access-tokens.md?pivots=programming-language-csharp) si aún no tiene ningún token disponible.

### <a name="initialize-the-callagent-and-access-device-manager"></a>Inicialice CallAgent y acceda al Administrador de dispositivos
Para crear una instancia de CallAgent desde una instancia de CallClient, utilice el método `callClient.createCallAgent` que devuelva asincrónicamente un objeto CallAgent cuando se inicialice. El Administrador de dispositivos permite enumerar los dispositivos locales que se pueden usar en una llamada para transmitir secuencias de audio o vídeo. También permite solicitar permiso a un usuario para acceder al micrófono o a la cámara. 

```Swift
self.callClient = CallClient()
self.callClient?.createCallAgent(userCredential: userCredential!) { (agent, error) in
    if error != nil {
        print("ERROR: It was not possible to create a call agent.")
        return
    }

    else {
        self.callAgent = agent
        print("Call agent successfully created.")
        self.callAgent!.delegate = incomingCallHandler
        self.callClient?.getDeviceManager { (deviceManager, error) in
            if (error == nil) {
                print("Got device manager instance")
                self.deviceManager = deviceManager
            } else {
                print("Failed to get device manager instance")
            }
        }
    }
}
```

### <a name="ask-for-permissions"></a>Solicitud de permisos
Para solicitar permisos de audio y vídeo, es necesario agregar el código siguiente a la devolución de llamada `onAppear`. 

```Swift
AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
    if granted {
        AVCaptureDevice.requestAccess(for: .video) { (videoGranted) in
            /* NO OPERATION */
        }
    }
}
```

## <a name="display-local-video"></a>Visualización de un vídeo local
Antes de iniciar una llamada, puede administrar la configuración relacionada con el vídeo. En este inicio rápido, implementaremos la capacidad de activar o desactivar un vídeo local antes o durante una llamada. 

En primer lugar, tenemos que acceder a las cámaras locales con `deviceManager`. Una vez que hayamos seleccionado la cámara que queramos, podremos construir `LocalVideoStream`, crear un objeto `VideoStreamRenderer` y asociarlo a `previewView`. Durante la llamada, podemos usar `startVideo` o `stopVideo` para iniciar o detener el envío de `LocalVideoStream` a participantes remotos. Esta función también puede utilizarse con el control de llamadas entrantes. 

```Swift
func toggleLocalVideo() {
    // toggling video before call starts
    if (call == nil)
    {
        if(!sendingVideo)
        {
            self.callClient = CallClient()
            self.callClient?.getDeviceManager { (deviceManager, error) in
                if (error == nil) {
                    print("Got device manager instance")
                    self.deviceManager = deviceManager
                } else {
                    print("Failed to get device manager instance")
                }
            }
            guard let deviceManager = deviceManager else {
                return
            }
            let camera = deviceManager.cameras.first
            let scalingMode = ScalingMode.fit
            if (self.localVideoStream == nil) {
                self.localVideoStream = [LocalVideoStream]()
            }
            localVideoStream!.append(LocalVideoStream(camera: camera!))
            previewRenderer = try! VideoStreamRenderer(localVideoStream: localVideoStream!.first!)
            previewView = try! previewRenderer!.createView(withOptions: CreateViewOptions(scalingMode:scalingMode))
            self.sendingVideo = true
        }
        else{
            self.sendingVideo = false
            self.previewView = nil
            self.previewRenderer!.dispose()
            self.previewRenderer = nil
        }
    }
    // toggle local video during the call
    else{
        if (sendingVideo) {
            call!.stopVideo(stream: localVideoStream!.first!) { (error) in
                if (error != nil) {
                    print("cannot stop video")
                }
                else {
                    self.sendingVideo = false
                    self.previewView = nil
                    self.previewRenderer!.dispose()
                    self.previewRenderer = nil
                }
            }
        }
        else {
            guard let deviceManager = deviceManager else {
                return
            }
            let camera = deviceManager.cameras.first
            let scalingMode = ScalingMode.fit
            if (self.localVideoStream == nil) {
                self.localVideoStream = [LocalVideoStream]()
            }
            localVideoStream!.append(LocalVideoStream(camera: camera!))
            previewRenderer = try! VideoStreamRenderer(localVideoStream: localVideoStream!.first!)
            previewView = try! previewRenderer!.createView(withOptions: CreateViewOptions(scalingMode:scalingMode))
            call!.startVideo(stream:(localVideoStream?.first)!) { (error) in
                if (error != nil) {
                    print("cannot start video")
                }
                else {
                    self.sendingVideo = true
                }
            }
        }
    }
}
```

## <a name="place-an-outgoing-call"></a>Realización de una llamada saliente
El método `startCall` se establece como la acción que se llevará a cabo cuando se toque el botón Iniciar llamada. En este inicio rápido, de manera predeterminada, las llamadas salientes son solo de audio. Para iniciar una llamada con vídeo, debemos establecer `VideoOptions` con `LocalVideoStream` y pasarlo con `startCallOptions` para definir las opciones iniciales de la llamada.

```Swift
func startCall() {
        let startCallOptions = StartCallOptions()
        if(sendingVideo)
        {
            if (self.localVideoStream == nil) {
                self.localVideoStream = [LocalVideoStream]()
            }
            let videoOptions = VideoOptions(localVideoStreams: localVideoStream!)
            startCallOptions.videoOptions = videoOptions
        }
        let callees:[CommunicationIdentifier] = [CommunicationUserIdentifier(self.callee)]
        self.callAgent?.startCall(participants: callees, options: startCallOptions) { (call, error) in
            setCallAndObersever(call: call, error: error)
        }
    }
```

`CallObserver` y `RemotePariticipantObserver` se usan para administrar participantes remotos y eventos que tienen lugar en mitad de la llamada. Estableceremos los observadores en la función `setCallAndOberserver`. 

```Swift
func setCallAndObersever(call:Call!, error:Error?) {
    if (error == nil) {
        self.call = call
        self.callObserver = CallObserver(self)
        self.call!.delegate = self.callObserver
        self.remoteParticipantObserver = RemoteParticipantObserver(self)
    } else {
        print("Failed to get call object")
    }
}
```

## <a name="answer-an-incoming-call"></a>Respuesta a una llamada entrante
Para responder a una llamada entrante, implemente un objeto `IncomingCallHandler` para que aparezca el banner de llamada entrante y poder responder o rechazar la llamada. Inserte la siguiente implementación en `IncomingCallHandler.swift`. 

```Swift
final class IncomingCallHandler: NSObject, CallAgentDelegate, IncomingCallDelegate {
    public var contentView: ContentView?
    private var incomingCall: IncomingCall?

    private static var instance: IncomingCallHandler?
    static func getOrCreateInstance() -> IncomingCallHandler {
        if let c = instance {
            return c
        }
        instance = IncomingCallHandler()
        return instance!
    }

    private override init() {}
    
    public func callAgent(_ callAgent: CallAgent, didRecieveIncomingCall incomingCall: IncomingCall) {
        self.incomingCall = incomingCall
        self.incomingCall?.delegate = self
        contentView?.showIncomingCallBanner(self.incomingCall!)
    }
    
    public func callAgent(_ callAgent: CallAgent, didUpdateCalls args: CallsUpdatedEventArgs) {
        if let removedCall = args.removedCalls.first {
            contentView?.callRemoved(removedCall)
            self.incomingCall = nil
        }
    }
}
```

Es necesario crear una instancia de `IncomingCallHandler`. Para ello, agregue el código siguiente a la devolución de llamada `onAppear` en `ContentView.swift`:

```Swift
let incomingCallHandler = IncomingCallHandler.getOrCreateInstance()
incomingCallHandler.contentView = self
```

Una vez que CallAgent se haya creado correctamente, establezca un delegado:

```Swift
self.callAgent!.delegate = incomingCallHandler
```

Cuando haya una llamada entrante, `IncomingCallHandler` llamará a la función `showIncomingCallBanner` para mostrar el botón `answer` y `decline`. 

```Swift
func showIncomingCallBanner(_ incomingCall: IncomingCall?) {
    isIncomingCall = true
    self.incomingCall = incomingCall
}
```

Las acciones asociadas a `answer` y `decline` se implementan tal y como se indica a continuación. Para responder a la llamada con vídeo, es necesario activar el vídeo local y establecer las opciones de `AcceptCallOptions` con `localVideoStream`. 

```Swift
func answerIncomingCall() {
    isIncomingCall = false
    let options = AcceptCallOptions()
    if (self.incomingCall != nil) {
        guard let deviceManager = deviceManager else {
            return
        }        
        if (self.localVideoStream == nil) {
            self.localVideoStream = [LocalVideoStream]()
        }
        if(sendingVideo)
        {
            let camera = deviceManager.cameras.first
            localVideoStream!.append(LocalVideoStream(camera: camera!))
            let videoOptions = VideoOptions(localVideoStreams: localVideoStream!)
            options.videoOptions = videoOptions
        }
        self.incomingCall!.accept(options: options) { (call, error) in
            setCallAndObersever(call: call, error: error)
        }
    }
}

func declineIncomingCall(){
    self.incomingCall!.reject { (error) in }
    isIncomingCall = false
}
```

## <a name="remote-participant-video-streams"></a>Secuencias de vídeo de participantes remotos
Podemos crear una clase `RemoteVideoStreamData` para controlar las secuencias de vídeo de representación de un participante remoto. 

```Swift
public class RemoteVideoStreamData : NSObject, RendererDelegate {
    public func videoStreamRenderer(didFailToStart renderer: VideoStreamRenderer) {
        owner.errorMessage = "Renderer failed to start"
    }
    
    private var owner:ContentView
    let stream:RemoteVideoStream
    var renderer:VideoStreamRenderer? {
        didSet {
            if renderer != nil {
                renderer!.delegate = self
            }
        }
    }
    
    var views:[RendererView] = []
    init(view:ContentView, stream:RemoteVideoStream) {
        owner = view
        self.stream = stream
    }
    
    public func videoStreamRenderer(didRenderFirstFrame renderer: VideoStreamRenderer) {
        let size:StreamSize = renderer.size
        owner.remoteVideoSize = String(size.width) + " X " + String(size.height)
    }
}
```

## <a name="subscribe-to-events"></a>Suscripción a los eventos
Podemos implementar una clase `CallObserver` para suscribirnos a una colección de eventos a los que se enviarán notificaciones si los valores cambian durante la llamada. 

```Swift
public class CallObserver: NSObject, CallDelegate, IncomingCallDelegate {
    private var owner: ContentView
    init(_ view:ContentView) {
            owner = view
    }
        
    public func call(_ call: Call, didChangeState args: PropertyChangedEventArgs) {
        if(call.state == CallState.connected) {
            initialCallParticipant()
        }
    }

    // render remote video streams when remote participant changes
    public func call(_ call: Call, didUpdateRemoteParticipant args: ParticipantsUpdatedEventArgs) {
        for participant in args.addedParticipants {
            participant.delegate = owner.remoteParticipantObserver
            for stream in participant.videoStreams {
                if !owner.remoteVideoStreamData.isEmpty {
                    return
                }
                let data:RemoteVideoStreamData = RemoteVideoStreamData(view: owner, stream: stream)
                let scalingMode = ScalingMode.fit
                data.renderer = try! VideoStreamRenderer(remoteVideoStream: stream)
                let view:RendererView = try! data.renderer!.createView(withOptions: CreateViewOptions(scalingMode:scalingMode))
                data.views.append(view)
                self.owner.remoteViews.append(view)
                owner.remoteVideoStreamData[stream.id] = data
            }
            owner.remoteParticipant = participant
        }
    }

    // Handle remote video streams when the call is connected
    public func initialCallParticipant() {
        for participant in owner.call!.remoteParticipants {
            participant.delegate = owner.remoteParticipantObserver
            for stream in participant.videoStreams {
                renderRemoteStream(stream)
            }
            owner.remoteParticipant = participant
        }
    }
    
    //create render for RemoteVideoStream and attach it to view
    public func renderRemoteStream(_ stream: RemoteVideoStream!) {
        if !owner.remoteVideoStreamData.isEmpty {
            return
        }
        let data:RemoteVideoStreamData = RemoteVideoStreamData(view: owner, stream: stream)
        let scalingMode = ScalingMode.fit
        data.renderer = try! VideoStreamRenderer(remoteVideoStream: stream)
        let view:RendererView = try! data.renderer!.createView(withOptions: CreateViewOptions(scalingMode:scalingMode))
        self.owner.remoteViews.append(view)
        owner.remoteVideoStreamData[stream.id] = data
    }
}
```

## <a name="remote-participant-management"></a>Administración remota de participantes
El tipo `RemoteParticipant` representa a todos los participantes remotos y estos están disponibles mediante la colección `remoteParticipants` en una instancia de llamada. Podemos implementar una clase `RemoteParticipantObserver` para suscribirnos a las actualizaciones de secuencias de vídeo remotas de participantes remotos. 

```Swift
public class RemoteParticipantObserver : NSObject, RemoteParticipantDelegate {
    private var owner:ContentView
    init(_ view:ContentView) {
        owner = view
    }

    public func renderRemoteStream(_ stream: RemoteVideoStream!) {
        let data:RemoteVideoStreamData = RemoteVideoStreamData(view: owner, stream: stream)
        let scalingMode = ScalingMode.fit
        data.renderer = try! VideoStreamRenderer(remoteVideoStream: stream)
        let view:RendererView = try! data.renderer!.createView(withOptions: CreateViewOptions(scalingMode:scalingMode))
        self.owner.remoteViews.append(view)
        owner.remoteVideoStreamData[stream.id] = data
    }
    
    // render RemoteVideoStream when remote participant turns on the video, dispose the renderer when remote video is off
    public func remoteParticipant(_ remoteParticipant: RemoteParticipant, didUpdateVideoStreams args: RemoteVideoStreamsEventArgs) {
        for stream in args.addedRemoteVideoStreams {
            renderRemoteStream(stream)
        }
        for stream in args.removedRemoteVideoStreams {
            for data in owner.remoteVideoStreamData.values {
                data.renderer?.dispose()
            }
            owner.remoteViews.removeAll()
        }
    }
}
```

## <a name="run-the-code"></a>Ejecución del código
Para compilar y ejecutar la aplicación en el simulador de iOS, seleccione Product > Run (Producto > Ejecutar) o use el método abreviado de teclado (⌘-R).

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-ios-quickstarts/tree/main/Add%20Video%20Calling).