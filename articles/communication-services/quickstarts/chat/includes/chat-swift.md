---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: 24a5c92164e0eace41224edfd2153c6142f7ea49
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107251533"
---
[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-chat.md)]

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar, compruebe lo siguiente:

- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Instale [Xcode](https://developer.apple.com/xcode/) y [CocoaPods](https://cocoapods.org/). Va a usar Xcode para crear una aplicación iOS para el inicio rápido y Cocoapods para instalar las dependencias.
- Cree un recurso de Azure Communication Services. Para más información, consulte [Inicio rápido: Creación y administración de recursos de Communication Services](../../create-communication-resource.md). Para esta guía de inicio rápido, tendrá que registrar el punto de conexión del recurso.
- Cree dos usuarios de Azure Communication Services y emítales un [token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en `chat` y anote las cadenas `token` e `userId`. En esta guía de inicio rápido, se creará una conversación con un participante inicial y, después, se agregará un segundo participante a esta.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-ios-application"></a>Creación de una aplicación iOS

Abra Xcode y seleccione **Create a new Xcode project** (Crear un nuevo proyecto en Xcode). A continuación, seleccione **iOS** para la plataforma y **App** (Aplicación) para la plantilla.

Para el nombre del proyecto, escriba **ChatQuickstart**. A continuación, seleccione **Storyboard** como interfaz, **UIKit App Delegate** como ciclo de vida y **Swift** como lenguaje.

Seleccione **Next** (Siguiente) y elija el directorio en el que desea que se cree el proyecto.

### <a name="install-the-libraries"></a>Instalación de las bibliotecas

Utilice Cocoapods para instalar las dependencias de Communication Services necesarias.

Desde la línea de comandos, vaya al directorio raíz del proyecto de iOS `ChatQuickstart`. Cree un archivo Podfile con el siguiente comando: `pod init`.

Abra el archivo Podfile y agregue las siguientes dependencias al destino `ChatQuickstart`:

```
pod 'AzureCommunication', '~> 1.0.0-beta.11'
pod 'AzureCommunicationChat', '~> 1.0.0-beta.11'
```

Instale las dependencias con el comando siguiente: `pod install`. Tenga en cuenta que esto también crea un área de trabajo de Xcode.

Después de ejecutar `pod install`, vuelva a abrir el proyecto en Xcode; para ello, seleccione el área `.xcworkspace` recién creada.

### <a name="set-up-the-placeholders"></a>Configuración de los marcadores de posición

Abra el área de trabajo `ChatQuickstart.xcworkspace` en Xcode y, a continuación, abra `ViewController.swift`.

En este inicio rápido, va a agregar el código a `viewController` y verá la salida en la consola de Xcode. En este inicio rápido no se aborda la creación de una interfaz de usuario en iOS. 

Al principio de `viewController.swift`, importe las bibliotecas `AzureCommunication` y `AzureCommunicatonChat`:

```
import AzureCommunication
import AzureCommunicationChat
```

Copie el código siguiente en el método `viewDidLoad()` de `ViewController`:

```
override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.

        let semaphore = DispatchSemaphore(value: 0)
        DispatchQueue.global(qos: .background).async {
            do {
                // <CREATE A CHAT CLIENT>
                
                // <CREATE A CHAT THREAD>

                // <LIST ALL CHAT THREADS>

                // <GET A CHAT THREAD CLIENT>

                // <SEND A MESSAGE>

                // <SEND A READ RECEIPT >

                // <RECEIVE MESSAGES>

                // <ADD A USER>
                
                // <LIST USERS>
            } catch {
                print("Quickstart failed: \(error.localizedDescription)")
            }
        }
    }
```

Con fines de demostración, usaremos un semáforo para sincronizar el código. En los pasos siguientes, va a reemplazar los marcadores de posición por código de ejemplo mediante la biblioteca de chat de Azure Communication Services.


### <a name="create-a-chat-client"></a>Creación de un cliente de chat

Reemplace el comentario `<CREATE A CHAT CLIENT>` por el código siguiente:

```
let endpoint = "<ACS_RESOURCE_ENDPOINT>"
let credential =
try CommunicationTokenCredential(
    token: "<ACCESS_TOKEN>"
)
let options = AzureCommunicationChatClientOptions()

let chatClient = try ChatClient(
    endpoint: endpoint,
    credential: credential,
    withOptions: options
)
```

Reemplace `<ACS_RESOURCE_ENDPOINT>` por el punto de conexión del recurso de Azure Communication Services. Reemplace `<ACCESS_TOKEN>` por un token de acceso de Communication Services válido.

En este inicio rápido no se aborda la creación de un nivel de servicio para administrar tokens para la aplicación de chat, aunque se recomienda. Para más información, consulte la sección "Arquitectura del chat" de [Conceptos de chat](../../../concepts/chat/concepts.md).

Para más información sobre los tokens de acceso de usuario, consulte [Inicio rápido: Creación y administración de tokens de acceso](../../access-tokens.md).

## <a name="object-model"></a>Modelo de objetos 

Las siguientes clases e interfaces controlan algunas de las características principales de Chat SDK de Azure Communication Services para JavaScript.

| Nombre                                   | Descripción                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ChatClient` | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar conversaciones. |
| `ChatThreadClient` | Esta clase es necesaria para la funcionalidad de conversación de chat. Obtenga una instancia mediante `ChatClient` y úsela para enviar, recibir, actualizar y eliminar mensajes. También puede usarla para agregar, quitar y obtener usuarios, enviar notificaciones de escritura y confirmaciones de lectura, y suscribirse a eventos de chat. |

## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Ahora va a utilizar la instancia de `ChatClient` para crear una conversación con un usuario inicial.

Reemplace el comentario `<CREATE A CHAT THREAD>` por el código siguiente:

```
let request = CreateChatThreadRequest(
    topic: "Quickstart",
    participants: [
        ChatParticipant(
            id: CommunicationUserIdentifier("<USER_ID>"),
            displayName: "Jack"
        )
    ]
)

var threadId: String?
chatClient.create(thread: request) { result, _ in
    switch result {
    case let .success(result):
        threadId = result.chatThread?.id

    case .failure:
        fatalError("Failed to create thread.")
    }
    semaphore.signal()
}
semaphore.wait()
```

Reemplace `<USER_ID>` por un identificador de usuario de Communication Services válido.

Aquí va a usar un semáforo para esperar al controlador de finalización antes de continuar. En pasos posteriores, usará el elemento `threadId` de la respuesta que se devuelve al controlador de finalización.

## <a name="list-all-chat-threads"></a>Lista de todas las conversaciones de chat

Después de crear una conversación de chat, podemos enumerar todas las conversaciones de chat mediante una llamada al método `listChatThreads` en `ChatClient`. Reemplace el comentario `<LIST ALL CHAT THREADS>` por el código siguiente:

```
chatClient.listThreads { result, _ in
    switch result {
    case let .success(chatThreadItems):
        var iterator = chatThreadItems.syncIterator
            while let chatThreadItem = iterator.next() {
                print("Thread id: \(chatThreadItem.id)")
            }
    case .failure:
        print("Failed to list threads")
    }
    semaphore.signal()
}
semaphore.wait()
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat

Ahora que ha creado una conversación de chat, puede obtener un elemento `ChatThreadClient` para realizar operaciones en la conversación.

Reemplace el comentario `<GET A CHAT THREAD CLIENT>` por el código siguiente:

```
let chatThreadClient = try chatClient.createClient(forThread: threadId!)
```

## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Reemplace el comentario `<SEND A MESSAGE>` por el código siguiente:

```
let message = SendChatMessageRequest(
    content: "Hello!",
    senderDisplayName: "Jack"
)

var messageId: String?

chatThreadClient.send(message: message) { result, _ in
    switch result {
    case let .success(result):
        print("Message sent, message id: \(result.id)")
        messageId = result.id
    case .failure:
        print("Failed to send message")
    }
    semaphore.signal()
}
semaphore.wait()
```

En primer lugar, construya el elemento `SendChatMessageRequest`, que contiene el contenido y el nombre para mostrar del remitente. Esta solicitud también puede contener la hora del historial del recurso compartido, si desea incluirla. La respuesta devuelta al controlador de finalización contiene el identificador del mensaje enviado.


## <a name="send-a-read-receipt"></a>Envío de una confirmación de lectura

Puede enviar una confirmación de lectura para un mensaje determinado mediante una llamada al método `sendReadReceipt` de `ChatThreadClients`. Reemplace el comentario `<SEND A READ RECEIPT>` por el código siguiente:

```
if let id = messageId {
    chatThreadClient.sendReadReceipt(forMessage: id) { result, _ in
        switch result {
        case .success:
            print("Read receipt sent")
        case .failure:
            print("Failed to send read receipt")
        }
        semaphore.signal()
    }
    semaphore.wait()
} else {
    print("Cannot send read receipt without a message id")
}
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

Puede recibir mensajes de una conversación de chat mediante una llamada al método `listMessages()` desde `ChatThreadClient`. Los mensajes de la lista incluyen los mensajes del sistema, así como los mensajes enviados por el usuario. Para más información sobre los tipos de mensajes que puede recibir, consulte [Tipos de mensaje](https://docs.microsoft.com/azure/communication-services/concepts/chat/concepts#message-types).

Reemplace el comentario `<RECEIVE MESSAGES>` por el código siguiente:

```
chatThreadClient.listMessages { result, _ in
    switch result {
    case let .success(messages):
        var iterator = messages.syncIterator
        while let message = iterator.next() {
            print("Received message of type \(message.type)")
        }

    case .failure:
        print("Failed to receive messages")
    }
    semaphore.signal()
}
semaphore.wait()
```

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Adición de un usuario como miembro a la conversación del chat

Reemplace el comentario `<ADD A USER>` por el código siguiente:

```
let user = ChatParticipant(
    id: CommunicationUserIdentifier("<USER_ID>"),
    displayName: "Jane"
)

chatThreadClient.add(participants: [user]) { result, _ in
    switch result {
    case let .success(result):
        (result.invalidParticipants != nil) ? print("Added participant") : print("Error adding participant")
    case .failure:
        print("Failed to add the participant")
    }
    semaphore.signal()
}
semaphore.wait()
```

Reemplace `<USER_ID>` por el identificador de usuario de Communication Services del usuario que se va a agregar.

Al agregar un participante a una conversación, la respuesta devuelta puede contener errores. Estos errores indican que no se pueden agregar participantes concretos.

## <a name="list-users-in-a-thread"></a>Enumeración de usuarios en una conversación

Reemplace el comentario `<LIST USERS>` por el código siguiente:

```
chatThreadClient.listParticipants { result, _ in
    switch result {
    case let .success(participants):
        var iterator = participants.syncIterator
        while let participant = iterator.next() {
            let user = participant.id as! CommunicationUserIdentifier
            print("User with id: \(user.identifier)")
        }
    case .failure:
        print("Failed to list participants")
    }
    semaphore.signal()
}
semaphore.wait()
```

## <a name="run-the-code"></a>Ejecución del código

En Xcode, presione el botón Run (Ejecutar) para compilar y ejecutar el proyecto. En la consola, puede ver la salida del código y la salida del registrador de ChatClient.

