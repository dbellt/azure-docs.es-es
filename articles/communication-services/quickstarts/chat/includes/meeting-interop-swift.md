---
title: 'Inicio rápido: unirse a una reunión de Teams'
author: askaur
ms.author: askaur
ms.date: 05/13/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: beb3f6daa577a7505b49faf1acd7cc9fb9a69ff2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110080309"
---
En este inicio rápido aprenderá a chatear en una reunión de Teams mediante el SDK de chat de Azure Communication Services para iOS.

## <a name="prerequisites"></a>Requisitos previos 

- Una  [implementación de Teams](/deployoffice/teams-install). 
- Una [aplicación de llamada](../../voice-video-calling/get-started-teams-interop.md) en funcionamiento. 

## <a name="enable-teams-interoperability"></a>Habilitación de la interoperabilidad de Teams 

Cualquier usuario de Communication Services que se una a una reunión de Teams como invitado solo puede acceder al chat cuando se haya unido a la llamada de la reunión de Teams. Consulte la documentación de la [interoperabilidad de Teams](../../voice-video-calling/get-started-teams-interop.md) para aprender a agregar un usuario de Communication Services a una llamada de reunión de Teams.

Para usar esta característica debe ser miembro de la organización propietaria de ambas entidades.

## <a name="joining-the-meeting-chat"></a>Unión al chat de la reunión 

Una vez que la interoperabilidad de Teams se habilita, cualquier usuario de Communication Services puede unirse a la llamada de Teams como usuario externo mediante el SDK de llamadas. Al unirse a la llamada se le agregará también como participante en el chat de la reunión, donde podrán enviar mensajes a otros usuarios durante la llamada, y recibirlos de ellos. El usuario no tendrá acceso a los mensajes del chat enviados antes de que se haya unido a la llamada. Para unirse a la reunión e iniciar el chat, puede seguir los pasos siguientes.

## <a name="add-chat-to-the-teams-calling-app"></a>Incorporación de chat a la aplicación de llamada de Teams

Elimine `Podfile.lock` y reemplace el contenido de Podfile por lo siguiente:

```
platform :ios, '13.0'
use_frameworks!

target 'AzureCommunicationCallingSample' do
  pod 'AzureCommunicationCalling', '~> 1.0.0-beta.12'
  pod 'AzureCommunicationChat', '~> 1.0.0-beta.11'
end
```

## <a name="install-the-chat-packages"></a>Instalación de los paquetes de chat

Ejecute `pod install` para instalar el paquete AzureCommunicationChat.
Después de la instalación, abra el archivo `.xcworkspace`.

## <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Importe el paquete AzureCommunicationChat en `ContentView.swift` agregando el siguiente fragmento de código:

```
import AzureCommunicationChat
```

## <a name="add-the-teams-ui-controls"></a>Incorporación de los controles de la interfaz de usuario de Teams

En `ContentView.swift`, agregue el siguiente fragmento de código debajo de las variables de estado existentes. 

```
    // Chat
    @State var chatClient: ChatClient?
    @State var chatThreadClient: ChatThreadClient?
    @State var chatMessage: String = ""
    @State var meetingMessages: [MeetingMessage] = []

    let displayName: String = "<YOUR_DISPLAY_NAME_HERE>"
```

Reemplace `<YOUR_DISPLAY_NAME_HERE>` por el nombre para mostrar que le gustaría usar en el chat.

A continuación, modificaremos el formulario `Section` para mostrar los mensajes del chat y agregaremos controles de interfaz de usuario para chatear.

Agregue el siguiente fragmento de código al formulario existente. Justo después de la vista texto `Text(recordingStatus)`, para el estado de la grabación.

```
VStack(alignment: .leading) {
    ForEach(meetingMessages, id: \.id) { message in
        let currentUser: Bool = (message.displayName == self.displayName)
        let foregroundColor = currentUser ? Color.white : Color.black
        let background = currentUser ? Color.blue : Color(.systemGray6)
        let alignment = currentUser ? HorizontalAlignment.trailing : HorizontalAlignment.leading
        VStack {
            Text(message.displayName).font(Font.system(size: 10))
            Text(message.content)
        }
        .alignmentGuide(.leading) { d in d[alignment] }
        .padding(10)
        .foregroundColor(foregroundColor)
        .background(background)
        .cornerRadius(10)
    }
}.frame(minWidth: 0, maxWidth: .infinity)
TextField("Enter your message...", text: $chatMessage)
Button(action: sendMessage) {
    Text("Send Message")
}.disabled(chatThreadClient == nil)
```

A continuación, cambie el título a `Chat Teams Quickstart`. Modifique la línea siguiente con este título.

```
.navigationBarTitle("Chat Teams Quickstart")
```

## <a name="enable-the-teams-ui-controls"></a>Habilitación de los controles de la interfaz de usuario de Teams

### <a name="initialize-the-chatclient"></a>Inicialización de ChatClient

Cree una instancia de `ChatClient` y habilite las notificaciones en tiempo real. Usaremos notificaciones en tiempo real para recibir los mensajes del chat.

Dentro de `NavigationView` `.onAppear`, agregue el fragmento de código siguiente, después del código existente que inicializa `CallAgent`.

```
// Initialize the ChatClient
do {
    let endpoint = "COMMUNICATION_SERVICES_RESOURCE_ENDPOINT_HERE>"
    let credential = try CommunicationTokenCredential(token: "<USER_ACCESS_TOKEN_HERE>")

    self.chatClient = try ChatClient(
        endpoint: endpoint,
        credential: credential,
        withOptions: AzureCommunicationChatClientOptions()
    )

    self.message = "ChatClient successfully created"

    // Start real-time notifications
    self.chatClient?.startRealTimeNotifications() { result in
        switch result {
        case .success:
            print("Real-time notifications started")
            // Receive chat messages
            self.chatClient?.register(event: ChatEventId.chatMessageReceived, handler: receiveMessage)
        case .failure:
            print("Failed to start real-time notifications")
            self.message = "Failed to enable chat notifications"
        }
    }
} catch {
    print("Unable to create ChatClient")
    self.message = "Please enter a valid endpoint and Chat token in source code"
    return
}
```

Reemplace `<COMMUNICATION_SERVICES_RESOURCE_ENDPOINT_HERE>` por el punto de conexión del recurso de Communication Services.
Reemplace `<USER_ACCESS_TOKEN_HERE>` por un token de acceso que tenga el ámbito del chat. 

Más información sobre los tokens de acceso de usuario: [Token de acceso de usuario](../../access-tokens.md)

### <a name="initialize-the-chatthreadclient"></a>Inicialización de ChatThreadClient

Dentro de la función `joinTeamsMeeting()` existente, inicializaremos `ChatThreadClient` una vez que el usuario se haya unido a la reunión.

Dentro del controlador de finalización de la llamada a `self.callAgent?.join()`, agregue el código debajo del comentario `// Initialize the ChatThreadClient`. El código completo se muestra a continuación.

```
self.callAgent?.join(with: teamsMeetingLinkLocator, joinCallOptions: joinCallOptions) { (call, error) in
    if (error == nil) {
        self.call = call
        self.callObserver = CallObserver(self)
        self.call!.delegate = self.callObserver
        self.message = "Teams meeting joined successfully"

        // Initialize the ChatThreadClient
        do {
            guard let threadId = getThreadId(from: self.meetingLink) else {
                self.message = "Failed to join meeting chat"
                return
            }
            self.chatThreadClient = try chatClient?.createClient(forThread: threadId)
            self.message = "Joined meeting chat successfully"
        } catch {
            print("Failed to create ChatThreadClient")
            self.message = "Failed to join meeting chat"
            return
        }
    } else {
        print("Failed to join Teams meeting")
    }
}
```

Agregue la siguiente función auxiliar a `ContentView`, que se usa para recuperar el id. de subproceso de chat del vínculo de reunión de Teams.

```
func getThreadId(from meetingLink: String) -> String? {
    if let range = self.meetingLink.range(of: "meetup-join/") {
        let thread = self.meetingLink[range.upperBound...]
        if let endRange = thread.range(of: "/")?.lowerBound {
            return String(thread.prefix(upTo: endRange))
        }
    }
    return nil
}
```

### <a name="enable-sending-messages"></a>Habilitación del envío de mensajes

Agregue la función `sendMessage()` a `ContentView`. Esta función usará `ChatThreadClient` para enviar mensajes del usuario.

```
func sendMessage() {
    let message = SendChatMessageRequest(
        content: self.chatMessage,
        senderDisplayName: self.displayName
    )
    
    self.chatThreadClient?.send(message: message) { result, _ in
        switch result {
        case .success:
            print("Chat message sent")
        case .failure:
            print("Failed to send chat message")
        }

        self.chatMessage = ""
    }
}
```

### <a name="enable-receiving-messages"></a>Habilitación de la recepción de mensajes

Para recibir mensajes, implementaremos el controlador para eventos `ChatMessageReceived`. Cuando se envíen nuevos mensajes al subproceso, este controlador agregará los mensajes a la variable `meetingMessages` para que se puedan mostrar en la interfaz de usuario.

En primer lugar, agregue la siguiente estructura a `ContentView.swift`. La interfaz de usuario usará los datos de la estructura para mostrar los mensajes del chat.

```
struct MeetingMessage {
    let id: String
    let content: String
    let displayName: String
}
```

A continuación, agregue la función `receiveMessage()` a `ContentView`. Este es el controlador al que se llama cada vez que se produce un evento `ChatMessageReceived`.

```
func receiveMessage(response: Any, eventId: ChatEventId) {
    let chatEvent: ChatMessageReceivedEvent = response as! ChatMessageReceivedEvent

    let displayName: String = chatEvent.senderDisplayName ?? "Unknown User"
    let content: String = chatEvent.message.replacingOccurrences(of: "<[^>]+>", with: "", options: String.CompareOptions.regularExpression)

    self.meetingMessages.append(
        MeetingMessage(
            id: chatEvent.id,
            content: content,
            displayName: displayName
        )
    )
}
```

### <a name="leave-the-chat"></a>Abandono del chat

Cuando el usuario abandone la reunión de Teams, borraremos el chat de la interfaz de usuario. El código completo se muestra a continuación.

```
func leaveMeeting() {
    if let call = call {
        call.hangUp(options: nil, completionHandler: { (error) in
            if error == nil {
                self.message = "Leaving Teams meeting was successful"
                // Clear the chat
                self.meetingMessages.removeAll()
            } else {
                self.message = "Leaving Teams meeting failed"
            }
        })
    } else {
        self.message = "No active call to hanup"
    }
}
```

## <a name="get-a-teams-meeting-chat-thread-for-a-communication-services-user"></a>Obtención de un subproceso del chat de la reunión para un usuario de Communication Services

El vínculo y el chat de la reunión de Teams se pueden recuperar mediante las instancias de Graph API, que se detallan en la [documentación de Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true). El SDK de llamada de Communication Services acepta un vínculo a toda la reunión de Teams. Este vínculo se devuelve como parte del recurso `onlineMeeting`, accesible desde la [propiedad `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). Con las instancias de [Graph API](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true), también puede obtener `threadId`. La respuesta tendrá un objeto `chatInfo` que contiene el elemento `threadID`. 

También puede obtener la información de la reunión necesaria y el identificador de la conversación en la dirección URL **Unirse a la reunión** de la propia invitación a la reunión de Teams.
Así es el vínculo de una reunión en Teams: `https://teams.microsoft.com/l/meetup-join/meeting_chat_thread_id/1606337455313?context=some_context_here`. El valor de `threadId` estará donde se encuentra el elemento `meeting_chat_thread_id` en el vínculo. Asegúrese de que el elemento `meeting_chat_thread_id` se haya escapado antes de utilizarlo. Debería tener el siguiente formato: `19:meeting_ZWRhZDY4ZGUtYmRlNS00OWZaLTlkZTgtZWRiYjIxOWI2NTQ4@thread.v2`.


## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación. 

Para unirse a la reunión de Teams, escriba el vínculo de la reunión de Teams en la interfaz de usuario.

Después de unirse a la reunión de Teams, deberá admitir al usuario a la reunión en el cliente de Teams. Una vez que el usuario se haya admitido y se haya unido al chat, podrá enviar y recibir mensajes.

:::image type="content" source="../join-teams-meeting-chat-quickstart-ios.png" alt-text="Captura de pantalla de la aplicación iOS completada.":::

> [!NOTE] 
> Actualmente, solo se admite el envío, la recepción y la edición de mensajes para los escenarios de interoperabilidad con Teams. Otras características, como los indicadores de escritura y que los usuarios de Communication Services agreguen o quiten otros usuarios de la reunión de Teams aún no se admiten.