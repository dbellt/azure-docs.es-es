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
ms.openlocfilehash: 1ac7ec3a8883bf2ff1ddc105f3ee7bf68b046e52
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111429599"
---
> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/add-chat)

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar, compruebe lo siguiente:
- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Instalar [Visual Studio](https://visualstudio.microsoft.com/downloads/)
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md). Deberá registrar el **punto de conexión** del recurso para esta guía de inicio rápido.
- Un [Token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en "chat" y anote la cadena del token, así como la cadena userId.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `ChatQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o ChatQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd ChatQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

Instalación de Chat SDK de Azure Communication Services para .NET

```PowerShell
dotnet add package Azure.Communication.Chat --version 1.0.0
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales de Chat SDK de Azure Communication Services para C#.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar subprocesos. |
| ChatThreadClient | Esta clase es necesaria para la funcionalidad de subproceso de chat. Obtiene una instancia a través de la clase ChatClient y la usa para enviar, recibir, actualizar o eliminar mensajes, agregar, quitar u obtener participantes, enviar notificaciones de escritura y confirmaciones de lectura. |

## <a name="create-a-chat-client"></a>Creación de un cliente de chat

Para crear un cliente de chat, usará el punto de conexión de Communication Services y el token de acceso que se generó como parte de los pasos de requisitos previos. Debe usar la clase `CommunicationIdentityClient` de Identity SDK para crear un usuario y emitir un token para pasarlo al cliente de chat.

Obtenga más información sobre los [tokens de acceso de usuario](../../access-tokens.md).

Este inicio rápido no cubre la creación de un nivel de servicio para administrar tokens para la aplicación de chat, aunque se recomienda. [Más información sobre la arquitectura del chat](../../../concepts/chat/concepts.md).

Copie los siguientes fragmentos de código y péguelos en el archivo de código fuente: **Program.cs**
```csharp
using Azure;
using Azure.Communication;
using Azure.Communication.Chat;
using System;

namespace ChatQuickstart
{
    class Program
    {
        static async System.Threading.Tasks.Task Main(string[] args)
        {
            // Your unique Azure Communication service endpoint
            Uri endpoint = new Uri("https://<RESOURCE_NAME>.communication.azure.com");

            CommunicationTokenCredential communicationTokenCredential = new CommunicationTokenCredential(<Access_Token>);
            ChatClient chatClient = new ChatClient(endpoint, communicationTokenCredential);
        }
    }
}
```

## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Use el método `createChatThread` en chatClient para crear una conversación de chat.
- Use `topic` para proporcionar un tema a este chat; el tema puede actualizarse después de crear el subproceso de chat mediante la función `UpdateTopic`.
- Utilice la propiedad `participants` para pasar una lista de objetos `ChatParticipant` que se van a agregar al subproceso de chat. El objeto `ChatParticipant` se inicializó con un objeto `CommunicationIdentifier`. El objeto `CommunicationIdentifier` puede ser de tipo `CommunicationUserIdentifier`, `MicrosoftTeamsUserIdentifier` o `PhoneNumberIdentifier`. Por ejemplo, para obtener un objeto `CommunicationIdentifier`, deberá pasar un identificador de acceso creado siguiendo las instrucciones de [Creación de un usuario](../../access-tokens.md#create-an-identity).

El objeto de respuesta del método `createChatThread` contiene los detalles `chatThread`. Para interactuar con las operaciones de conversaciones de chat, como agregar participantes, enviar un mensaje, eliminar un mensaje, etc., es necesario crear una instancia del cliente `chatThreadClient` con el método `GetChatThreadClient` en el cliente `ChatClient`.

```csharp
var chatParticipant = new ChatParticipant(identifier: new CommunicationUserIdentifier(id: "<Access_ID>"))
{
    DisplayName = "UserDisplayName"
};
CreateChatThreadResult createChatThreadResult = await chatClient.CreateChatThreadAsync(topic: "Hello world!", participants: new[] { chatParticipant });
ChatThreadClient chatThreadClient = chatClient.GetChatThreadClient(threadId: createChatThreadResult.ChatThread.Id);
string threadId = chatThreadClient.Id;
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat
El método `GetChatThreadClient` devuelve un cliente de subproceso para un subproceso que ya existe. Se puede usar para realizar operaciones en el subproceso creado: agregar miembros, enviar un mensaje, etc. thread_id es el id. único del subproceso de chat existente.

```csharp
string threadId = "<THREAD_ID>";
ChatThreadClient chatThreadClient = chatClient.GetChatThreadClient(threadId: threadId);
```

## <a name="list-all-chat-threads"></a>Lista de todas las conversaciones de chat
Use `GetChatThreads` para recuperar todas las conversaciones de chat de las que forma parte el usuario.

```csharp
AsyncPageable<ChatThreadItem> chatThreadItems = chatClient.GetChatThreadsAsync();
await foreach (ChatThreadItem chatThreadItem in chatThreadItems)
{
    Console.WriteLine($"{ chatThreadItem.Id}");
}
```

## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Utilice `SendMessage` para enviar un mensaje a una conversación.

- Utilice `content` para proporcionar el contenido del mensaje; es obligatorio.
- Utilice `type` para indicar el tipo de contenido del mensaje, como "text" o "html". Si no se especifica, se establecerá "text".
- Utilice `senderDisplayName` para especificar el nombre para mostrar del remitente. Si no se especifica, se establecerá una cadena vacía.

```csharp
SendChatMessageResult sendChatMessageResult = await chatThreadClient.SendMessageAsync(content:"hello world", type: ChatMessageType.Text);
string messageId = sendChatMessageResult.Id;
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

También puede recuperar mensajes de chat mediante el sondeo del método `GetMessages` en el cliente de subproceso de chat a intervalos especificados.

```csharp
AsyncPageable<ChatMessage> allMessages = chatThreadClient.GetMessagesAsync();
await foreach (ChatMessage message in allMessages)
{
    Console.WriteLine($"{message.Id}:{message.Content.Message}");
}
```

`GetMessages` toma un parámetro `DateTimeOffset` opcional. Si se especifica ese desplazamiento, recibirá los mensajes recibidos, actualizados o eliminados posteriormente. Tenga en cuenta que los mensajes recibidos antes de la hora de desplazamiento, pero editados o eliminados posteriormente, también se devolverán.

`GetMessages` devuelve la versión más reciente del mensaje, incluidas las modificaciones o eliminaciones que se han producido en este mediante `UpdateMessage` y `DeleteMessage`. En el caso de los mensajes eliminados, `chatMessage.DeletedOn` devuelve un valor de fecha y hora que indica cuándo se eliminó el mensaje. En el caso de los mensajes editados, `chatMessage.EditedOn` devuelve un valor de fecha y hora que indica cuándo se editó el mensaje. Es posible acceder a la hora original de creación del mensaje mediante `chatMessage.CreatedOn` y se puede usar para ordenar los mensajes.

`GetMessages` devuelve distintos tipos de mensajes que se pueden identificar mediante `chatMessage.Type`. Estos tipos son:

- `Text`: mensaje de chat normal enviado por un miembro del subproceso.

- `Html`: un mensaje de texto con formato. Tenga en cuenta que los usuarios de Communication Services actualmente no pueden enviar mensajes de texto enriquecido. Este tipo de mensaje es compatible con los mensajes enviados desde usuarios a de Teams a usuarios de Communication Services en escenarios de interoperabilidad de Teams.

- `TopicUpdated`: mensaje del sistema que indica que el tema se ha actualizado. (solo lectura).

- `ParticipantAdded`: mensaje del sistema que indica que se han agregado uno o varios participantes a la conversación del chat. (solo lectura).

- `ParticipantRemoved`: mensaje del sistema que indica que se ha eliminado un miembro de la conversación del chat.

Para obtener más información, consulte [Tipos de mensajes](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Adición de un usuario como miembro a la conversación del chat

Una vez que se crea un subproceso, puede agregar y quitar usuarios de este. Al agregar usuarios, les concede acceso para poder enviar mensajes a la conversación, y agregar o quitar otros participantes. Antes de llamar a `AddParticipants`, asegúrese de que ha adquirido un token de acceso y una identidad nuevos para ese usuario. El usuario necesitará ese token de acceso para poder inicializar su cliente de chat.

Use `AddParticipants` para agregar uno o varios participantes a la conversación de chat. A continuación se indican los atributos admitidos para cada participante de la conversación:
- `communicationUser` es obligatorio y es la identidad del participante de la conversación.
- `displayName` es opcional y es el nombre para mostrar del participante de la conversación.
- `shareHistoryTime` es opcional y es la hora a partir de la cual el historial de chat se compartió con el participante.

```csharp
var josh = new CommunicationUserIdentifier(id: "<Access_ID_For_Josh>");
var gloria = new CommunicationUserIdentifier(id: "<Access_ID_For_Gloria>");
var amy = new CommunicationUserIdentifier(id: "<Access_ID_For_Amy>");

var participants = new[]
{
    new ChatParticipant(josh) { DisplayName = "Josh" },
    new ChatParticipant(gloria) { DisplayName = "Gloria" },
    new ChatParticipant(amy) { DisplayName = "Amy" }
};

await chatThreadClient.AddParticipantsAsync(participants: participants);
```

## <a name="get-thread-participants"></a>Obtención de los participantes de la conversación

Use `GetParticipants` para recuperar los participantes de la conversación de chat.

```csharp
AsyncPageable<ChatParticipant> allParticipants = chatThreadClient.GetParticipantsAsync();
await foreach (ChatParticipant participant in allParticipants)
{
    Console.WriteLine($"{((CommunicationUserIdentifier)participant.User).Id}:{participant.DisplayName}:{participant.ShareHistoryTime}");
}
```

## <a name="send-read-receipt"></a>Envío de confirmación de lectura

Use `SendReadReceipt` para notificar a otros participantes que el usuario ha leído el mensaje.

```csharp
await chatThreadClient.SendReadReceiptAsync(messageId: messageId);
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
