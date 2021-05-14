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
ms.openlocfilehash: 7748737450a013fcfe66ac189213b390c2afa7d1
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109627286"
---
## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar, compruebe lo siguiente:

- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Instale las versiones Active LTS y Maintenance LTS de [Node.js](https://nodejs.org/en/download/).
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md). Para este inicio rápido, tendrá que **registrar el punto de conexión del recurso**.
- Cree *tres* usuarios de ACS y genéreles un token de acceso de usuario [Token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en **chat** y **anote la cadena del token, así como la cadena userId**. La demo completa crea una conversación con dos participantes iniciales y, después, agrega un tercer participante a la misma.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-web-application"></a>Creación de una nueva aplicación web

En primer lugar, abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir chat-quickstart && cd chat-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

### <a name="install-the-packages"></a>Instalación de los paquetes

Use el comando `npm install` para instalar los siguientes SDK de Communication Services para JavaScript.

```console
npm install @azure/communication-common --save

npm install @azure/communication-identity --save

npm install @azure/communication-signaling --save

npm install @azure/communication-chat --save

```

La opción `--save` muestra la biblioteca como dependencia en el archivo **package.json**.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Esta guía de inicio rápido usa webpack para agrupar los recursos de la aplicación. Ejecute el siguiente comando para instalar los paquetes de npm webpack, webpack-cli y webpack-dev-server, y mostrarlos como dependencias de desarrollo en el archivo **package.json**:

```console
npm install webpack webpack-cli webpack-dev-server --save-dev
```

Cree un archivo `webpack.config.js` en el directorio raíz. Copie la configuración siguiente en este archivo:

```
module.exports = {
  entry: "./client.js",
  output: {
    filename: "bundle.js"
  },
  devtool: "inline-source-map",
  mode: "development"
}
```

Agregue un script `start` a `package.json`, lo usaremos para ejecutar la aplicación. En la sección `scripts` de `package.json` agregue lo siguiente:

```
"scripts": {
  "start": "webpack serve --config ./webpack.config.js"
}
```

Cree un archivo **index.html** en el directorio raíz del proyecto. Usaremos este archivo como plantilla para agregar la funcionalidad de chat mediante Chat SDK de Azure Communication Services para JavaScript.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Chat Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Chat Quickstart</h1>
    <script src="./bundle.js"></script>
  </body>
</html>
```

Cree un archivo en el directorio raíz del proyecto denominado **client.js** que contendrá la lógica de la aplicación para esta guía de inicio rápido.

### <a name="create-a-chat-client"></a>Creación de un cliente de chat

Para crear un cliente de chat en una aplicación web, usará el **punto de conexión** de Communication Services y el **token de acceso** que se generó como parte de los pasos de los requisitos previos.

Los tokens de acceso de usuario permiten compilar aplicaciones cliente que se autentiquen directamente en Azure Communication Services. En este inicio rápido no se trata la creación de un nivel de servicio para administrar tokens para una aplicación de chat. Consulte [Conceptos de chat](../../../concepts/chat/concepts.md) para más información sobre la arquitectura de chat y [Tokens de acceso de usuario](../../access-tokens.md) para más información sobre los tokens de acceso.

En **client.js**, use el punto de conexión y el token de acceso del código siguiente para agregar la funcionalidad de chat mediante Chat SDK de Azure Communication Services para JavaScript.

```JavaScript

import { ChatClient } from '@azure/communication-chat';
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

// Your unique Azure Communication service endpoint
let endpointUrl = 'https://<RESOURCE_NAME>.communication.azure.com';
// The user access token generated as part of the pre-requisites
let userAccessToken = '<USER_ACCESS_TOKEN>';

let chatClient = new ChatClient(endpointUrl, new AzureCommunicationTokenCredential(userAccessToken));
console.log('Azure Communication Chat client created!');
```
- Reemplace **endpointUrl** por el punto de conexión de recurso de Communication Services y consulte [Creación de un recurso de comunicación de Azure](../../create-communication-resource.md) si aún no lo ha hecho.
- Reemplace **userAccessToken** por el token que ha emitido.


### <a name="run-the-code"></a>Ejecución del código

Ejecute el siguiente comando para agrupar el host de aplicación en un servidor web local:
```console
npm run start
```
Abra el explorador web y vaya a http://localhost:8080/.
En la consola de herramientas de desarrollo en el explorador, debe ver lo siguiente:

```console
Azure Communication Chat client created!
```

## <a name="object-model"></a>Modelo de objetos
Las siguientes clases e interfaces controlan algunas de las características principales de Chat SDK de Azure Communication Services para JavaScript.

| Nombre                                   | Descripción                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar subprocesos. |
| ChatThreadClient | Esta clase es necesaria para la funcionalidad de subproceso de chat. Obtiene una instancia de a través de la clase ChatClient y la usa para enviar, recibir, actualizar o eliminar mensajes, agregar, quitar u obtener usuarios, enviar notificaciones de escritura y confirmaciones de lectura, y suscribirse a eventos de chat. |


## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Use el método `createThread` para crear un subproceso de chat.

`createThreadRequest` se usa para describir la solicitud de subproceso:

- Utilice `topic` para proporcionar un tema a este chat. Los temas se pueden actualizar después de crear el subproceso de chat mediante la función `UpdateThread`.
- Use `participants` para enumerar los participantes que se van a agregar a la conversación del chat.

Cuando se resuelve, el método `createChatThread` devuelve `CreateChatThreadResult`. Este modelo contiene una propiedad `chatThread` donde se puede acceder al `id` del subproceso recién creado. Luego, se puede utilizar `id` para obtener una instancia de `ChatThreadClient`. A continuación, se puede usar `ChatThreadClient` para realizar una operación dentro del subproceso, como enviar mensajes o enumerar participantes.

```JavaScript
async function createChatThread() {
  const createChatThreadRequest = {
    topic: "Hello, World!"
  };
  const createChatThreadOptions = {
    participants: [
      {
        id: { communicationUserId: '<USER_ID>' },
        displayName: '<USER_DISPLAY_NAME>'
      }
    ]
  };
  const createChatThreadResult = await chatClient.createChatThread(
    createChatThreadRequest,
    createChatThreadOptions
  );
  const threadId = createChatThreadResult.chatThread.id;
  return threadId;
}

createChatThread().then(async threadId => {
  console.log(`Thread created:${threadId}`);
  // PLACEHOLDERS
  // <CREATE CHAT THREAD CLIENT>
  // <RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>
  // <SEND MESSAGE TO A CHAT THREAD>
  // <LIST MESSAGES IN A CHAT THREAD>
  // <ADD NEW PARTICIPANT TO THREAD>
  // <LIST PARTICIPANTS IN A THREAD>
  // <REMOVE PARTICIPANT FROM THREAD>
  });
```

Al actualizar la pestaña del explorador, debería ver lo siguiente en la consola:
```console
Thread created: <thread_id>
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat

El método `getChatThreadClient` devuelve un `chatThreadClient` para un subproceso que ya existe. Se puede usar para realizar operaciones en el subproceso creado: agregar participantes, enviar un mensaje, etc. threadId es el identificador único de la conversación del chat existente.

```JavaScript
let chatThreadClient = chatClient.getChatThreadClient(threadId);
console.log(`Chat Thread client for threadId:${threadId}`);

```
Agregue este código en lugar del comentario `<CREATE CHAT THREAD CLIENT>` en **client.js**, actualice la pestaña del explorador y compruebe la consola. Debería ver lo siguiente:
```console
Chat Thread client for threadId: <threadId>
```

## <a name="list-all-chat-threads"></a>Lista de todas las conversaciones de chat

El método `listChatThreads` devuelve un `PagedAsyncIterableIterator` de tipo `ChatThreadItem`. Se puede usar para enumerar todas las conversaciones de chat.
Un iterador de `[ChatThreadItem]` es la respuesta que devuelven los subprocesos de lista.

```JavaScript
const threads = chatClient.listChatThreads();
for await (const thread of threads) {
   // your code here
}
```

## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Utilice el método `sendMessage` para enviar un mensaje a un subproceso identificado mediante threadId.

`sendMessageRequest` se utiliza para describir la solicitud del mensaje:

- Utilice `content` para proporcionar el contenido del mensaje de chat.

`sendMessageOptions` se usa para describir los parámetros opcionales de la operación:

- Utilice `senderDisplayName` para especificar el nombre para mostrar del remitente.
- Use `type` para especificar el tipo de mensaje, como "texto" o "html".

`SendChatMessageResult` es la respuesta que se devuelve al enviar un mensaje. Contiene un identificador que es el identificador único del mensaje.

```JavaScript
const sendMessageRequest =
{
  content: 'Hello Geeta! Can you share the deck for the conference?'
};
let sendMessageOptions =
{
  senderDisplayName : 'Jack',
  type: 'text'
};
const sendChatMessageResult = await chatThreadClient.sendMessage(sendMessageRequest, sendMessageOptions);
const messageId = sendChatMessageResult.id;
```

Agregue este código en lugar del comentario `<SEND MESSAGE TO A CHAT THREAD>` en el archivo **client.js**, actualice la pestaña del explorador y compruebe la consola.
```console
Message sent!, message id:<number>
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

Con la señalización en tiempo real, puede suscribirse para escuchar nuevos mensajes entrantes y actualizar los mensajes actuales en la memoria en consecuencia. Azure Communication Services admite una [lista de eventos a los que se puede suscribir](../../../concepts/chat/concepts.md#real-time-notifications).

```JavaScript
// open notifications channel
await chatClient.startRealtimeNotifications();
// subscribe to new notification
chatClient.on("chatMessageReceived", (e) => {
  console.log("Notification chatMessageReceived!");
  // your code here
});

```
Agregue este código en lugar del comentario `<RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>` en **client.js**.
Actualice la pestaña del explorador; debería ver en la consola un mensaje `Notification chatMessageReceived`.

También puede recuperar mensajes de chat mediante el sondeo del método `listMessages` a intervalos especificados.

```JavaScript

const messages = chatThreadClient.listMessages();
for await (const message of messages) {
   // your code here
}

```
Agregue este código en lugar del comentario `<LIST MESSAGES IN A CHAT THREAD>` en **client.js**.
Actualice la pestaña; en la consola debería encontrar la lista de los mensajes enviados en esta conversación del chat.

`listMessages` devuelve distintos tipos de mensajes que se pueden identificar mediante `chatMessage.type`. 

Para obtener más información, consulte [Tipos de mensajes](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Adición de un usuario como miembro a la conversación del chat

Una vez que se crea un subproceso de chat, puede agregar y quitar usuarios de este. Al agregar usuarios, les concede acceso para enviar mensajes a la conversación del chat, y agregar o quitar otros participantes.

Antes de llamar al método `addParticipants`, asegúrese de que ha adquirido un nuevo token de acceso y una identidad para el usuario. El usuario necesitará ese token de acceso para poder inicializar su cliente de chat.

`addParticipantsRequest` describe el objeto de solicitud, donde `participants` enumera los participantes que se van a agregar a la conversación del chat.
- El elemento `id` obligatorio es el identificador de la comunicación que se va a agregar al subproceso de chat.
- `displayName` es opcional y es el nombre para mostrar del participante de la conversación.
- `shareHistoryTime` es opcional y es la hora a partir de la cual el historial de chat se compartió con el participante. Para compartir el historial desde el inicio del subproceso de chat, establezca esta propiedad en cualquier fecha igual o anterior a la hora de creación del subproceso. Para no compartir ningún historial anterior a la hora en que se agregó el participante, establézcala en la fecha actual. Para compartir el historial parcialmente, establezca la opción en la fecha de su elección.

```JavaScript

const addParticipantsRequest =
{
  participants: [
    {
      id: { communicationUserId: '<NEW_PARTICIPANT_USER_ID>' },
      displayName: 'Jane'
    }
  ]
};

await chatThreadClient.addParticipants(addParticipantsRequest);

```
Reemplace **NEW_PARTICIPANT_USER_ID_USER_ID** por un [identificador de usuario nuevo](../../access-tokens.md). Agregue este código en lugar del comentario `<ADD NEW PARTICIPANT TO THREAD>` en **client.js**

## <a name="list-users-in-a-chat-thread"></a>Enumeración de usuarios en un subproceso de chat
```JavaScript
const participants = chatThreadClient.listParticipants();
for await (const participant of participants) {
   // your code here
}
```
Agregue este código en lugar del comentario `<LIST PARTICIPANTS IN A THREAD>` en **client.js**, actualice la pestaña del explorador y compruebe la consola; debería ver información acerca de los usuarios de un subproceso.

## <a name="remove-user-from-a-chat-thread"></a>Eliminación de un usuario de un subproceso de chat

En una conversación del chat, los participantes se pueden eliminar de forma similar a como se agregan. Para quitarlos, será preciso realizar un seguimiento de los identificadores de los participantes que se han agregado.

Utilice el método `removeParticipant`, donde `participant` es el usuario de la comunicación que se va a quitar del subproceso.

```JavaScript

await chatThreadClient.removeParticipant({ communicationUserId: <PARTICIPANT_ID> });
await listParticipants();
```
Reemplace **PARTICIPANT_ID** por un id. de usuario utilizado en el paso anterior (<NEW_PARTICIPANT_USER_ID>).
Agregue este código en lugar del comentario `<REMOVE PARTICIPANT FROM THREAD>` en **client.js**.
