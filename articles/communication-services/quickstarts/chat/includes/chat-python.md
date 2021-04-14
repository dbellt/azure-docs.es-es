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
ms.openlocfilehash: 31704e705b828cc0070e3b79f5d527cfa9deb0c3
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107386602"
---
[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-chat.md)]

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar, compruebe lo siguiente:

- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Instalación de [Python](https://www.python.org/downloads/)
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md). Deberá registrar el **punto de conexión** del recurso para esta guía de inicio rápido.
- Un [Token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en "chat" y anote la cadena del token, así como la cadena userId.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir chat-quickstart && cd chat-quickstart
```

Use un editor de texto para crear un archivo denominado **start-chat.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

```python
import os
# Add required SDK components from quickstart here

try:
    print('Azure Communication Services - Chat Quickstart')
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-sdk"></a>Instalación del SDK

```console

pip install azure-communication-chat

```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de Chat SDK de Azure Communication Services para Python.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar subprocesos. |
| ChatThreadClient | Esta clase es necesaria para la funcionalidad de subproceso de chat. Obtiene una instancia de a través de la clase ChatClient y la usa para enviar, recibir, actualizar o eliminar mensajes, agregar, quitar u obtener usuarios, enviar notificaciones de escritura y confirmaciones de lectura. |

## <a name="create-a-chat-client"></a>Creación de un cliente de chat

Para crear un cliente de chat, usará el punto de conexión de Communication Services y el `Access Token` que se generó como parte de los pasos de requisitos previos. Obtenga más información sobre los [tokens de acceso de usuario](../../access-tokens.md).

Este inicio rápido no cubre la creación de un nivel de servicio para administrar tokens para la aplicación de chat, aunque se recomienda. Para más información, consulte [Arquitectura del chat](../../../concepts/chat/concepts.md).

```console
pip install azure-communication-identity
```

```python
from azure.communication.chat import ChatClient, CommunicationTokenCredential

endpoint = "https://<RESOURCE_NAME>.communication.azure.com"
chat_client = ChatClient(endpoint, CommunicationTokenCredential("<Access Token>"))
```

## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Use el método `create_chat_thread` para crear un subproceso de chat.

- Use `topic` para proporcionar un tema de subproceso; el tema puede actualizarse después de crear el subproceso de chat mediante la función `update_thread`.
- Use `thread_participants` para enumerar el `ChatParticipant` que se va a agregar al subproceso de chat, el elemento `ChatParticipant` toma el tipo `CommunicationUserIdentifier` como `user`, que es el valor que obtuvo después de crearlo con [Creación de un usuario](../../access-tokens.md#create-an-identity).

`CreateChatThreadResult` es el resultado devuelto por la creación de un subproceso, puede utilizarlo para capturar `id` del subproceso de chat que se ha creado. Se puede utilizar `id` luego para capturar un objeto `ChatThreadClient` mediante el método `get_chat_thread_client`. Se puede usar `ChatThreadClient` para realizar otras operaciones de chat en este subproceso de chat.

```python
topic="test topic"

create_chat_thread_result = chat_client.create_chat_thread(topic)
chat_thread_client = chat_client.get_chat_thread_client(create_chat_thread_result.chat_thread.id)
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat
El método `get_chat_thread_client` devuelve un cliente de subproceso para un subproceso que ya existe. Se puede usar para realizar operaciones en la conversación creada: agregar participantes, enviar un mensaje, etc. thread_id es el identificador único de la conversación de chat existente.

Se puede usar `ChatThreadClient` para realizar otras operaciones de chat en este subproceso de chat.

```python
thread_id = create_chat_thread_result.chat_thread.id
chat_thread_client = chat_client.get_chat_thread_client(thread_id)
```


## <a name="list-all-chat-threads"></a>Lista de todas las conversaciones de chat
El método `list_chat_threads` devuelve un iterador de tipo `ChatThreadItem`. Se puede usar para enumerar todas las conversaciones de chat.

- Utilice `start_time` para especificar el primer punto en el tiempo a partir del cual obtener las conversaciones de chat.
- Utilice `results_per_page` para especificar el número máximo de conversaciones de chat que se devuelven por página.

Un iterador de `[ChatThreadItem]` es la respuesta que devuelven los subprocesos de lista.

```python
from datetime import datetime, timedelta

start_time = datetime.utcnow() - timedelta(days=2)

chat_threads = chat_client.list_chat_threads(results_per_page=5, start_time=start_time)
for chat_thread_item_page in chat_threads.by_page():
    for chat_thread_item in chat_thread_item_page:
        print(chat_thread_item)
        print('Chat Thread Id: ', chat_thread_item.id)
```


## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Utilice el método `send_message` para enviar un mensaje a una conversación de chat recién creada, identificada mediante thread_id.

- Utilice `content` para proporcionar el contenido del mensaje de chat.
- Utilice `chat_message_type` para especificar el tipo de contenido del mensaje. Los valores posibles son "text" y "html"; si no se especifica, se asigna el valor predeterminado "text".
- Utilice `sender_display_name` para especificar el nombre para mostrar del remitente.

`SendChatMessageResult` es la respuesta que se devuelve al enviar un mensaje. Contiene un identificador que es el identificador único del mensaje.

```python
from azure.communication.chat import ChatMessageType

topic = "test topic"
create_chat_thread_result = chat_client.create_chat_thread(topic)
thread_id = create_chat_thread_result.chat_thread.id
chat_thread_client = chat_client.get_chat_thread_client(create_chat_thread_result.chat_thread.id)


content='hello world'
sender_display_name='sender name'

# specify chat message type with pre-built enumerations
send_message_result_w_enum = chat_thread_client.send_message(content=content, sender_display_name=sender_display_name, chat_message_type=ChatMessageType.TEXT)
print("Message sent: id: ", send_message_result_w_enum.id)
```


## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

Puede recuperar mensajes de chat mediante el sondeo del método `list_messages` a intervalos especificados.

- Utilice `results_per_page` para especificar el número máximo de mensajes que se devuelven por página.
- Utilice `start_time` para especificar el primer punto en el tiempo a partir del cual obtener los mensajes.

Un iterador de `[ChatMessage]` es la respuesta que devuelven los mensajes de lista.

```python
from datetime import datetime, timedelta

start_time = datetime.utcnow() - timedelta(days=1)

chat_messages = chat_thread_client.list_messages(results_per_page=1, start_time=start_time)
for chat_message_page in chat_messages.by_page():
    for chat_message in chat_message_page:
        print("ChatMessage: Id=", chat_message.id, "; Content=", chat_message.content.message)
```

`list_messages` devuelve la versión más reciente del mensaje, incluidas las modificaciones o eliminaciones que se han producido en este mediante `update_message` y `delete_message`. En el caso de los mensajes eliminados, `ChatMessage.deleted_on` devuelve un valor de fecha y hora que indica cuándo se eliminó el mensaje. En el caso de los mensajes editados, `ChatMessage.edited_on` devuelve un valor de fecha y hora que indica cuándo se editó el mensaje. Es posible acceder a la hora original de creación del mensaje mediante `ChatMessage.created_on`, que se puede usar para ordenar los mensajes.

`list_messages` devuelve distintos tipos de mensajes que se pueden identificar mediante `ChatMessage.type`. 

Obtenga más información sobre los tipos de mensaje aquí: [Tipos de mensajes](../../../concepts/chat/concepts.md#message-types).

## <a name="send-read-receipt"></a>Envío de confirmación de lectura
Puede usar el método `send_read_receipt` para publicar un evento de confirmación de lectura en una conversación en nombre de un usuario.

- Use `message_id` para especificar el identificador del último mensaje leído por el usuario actual.

```python
content='hello world'

send_message_result = chat_thread_client.send_message(content)
chat_thread_client.send_read_receipt(message_id=send_message_result.id)
```


## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Adición de un usuario como miembro a la conversación del chat

Una vez que se crea un subproceso de chat, puede agregar y quitar usuarios de este. Al agregar usuarios, les concede acceso para enviar mensajes a la conversación del chat, y para agregar o quitar otros participantes. Antes de llamar al método `add_participants`, asegúrese de que ha adquirido un nuevo token de acceso y una identidad para ese usuario. El usuario necesitará ese token de acceso para poder inicializar su cliente de chat.

Mediante el método `add_participants` se pueden agregar uno o varios usuarios a la conversación de chat con el método, siempre que todos los usuarios dispongan de un nuevo token de acceso e identificación.

Se devuelve `list(tuple(ChatParticipant, CommunicationError))`. Cuando un participante se agrega correctamente, se espera una lista vacía. En caso de que se produzca un error al agregar un participante, la lista se rellena con los participantes con error junto con el error producido.

```python
from azure.communication.identity import CommunicationIdentityClient
from azure.communication.chat import ChatParticipant
from datetime import datetime

# create 2 users
identity_client = CommunicationIdentityClient.from_connection_string('<connection_string>')
new_users = [identity_client.create_user() for i in range(2)]

# # conversely, you can also add an existing user to a chat thread; provided the user_id is known
# from azure.communication.identity import CommunicationUserIdentifier
#
# user_id = 'some user id'
# user_display_name = "Wilma Flinstone"
# new_user = CommunicationUserIdentifier(user_id)
# participant = ChatParticipant(
#     user=new_user,
#     display_name=user_display_name,
#     share_history_time=datetime.utcnow())

participants = []
for _user in new_users:
  chat_thread_participant = ChatParticipant(
    user=_user,
    display_name='Fred Flinstone',
    share_history_time=datetime.utcnow()
  ) 
  participants.append(chat_thread_participant) 

response = chat_thread_client.add_participants(participants)

def decide_to_retry(error, **kwargs):
    """
    Insert some custom logic to decide if retry is applicable based on error
    """
    return True

# verify if all users has been successfully added or not
# in case of partial failures, you can retry to add all the failed participants 
retry = [p for p, e in response if decide_to_retry(e)]
if retry:
    chat_thread_client.add_participants(retry)
```


## <a name="list-thread-participants-in-a-chat-thread"></a>Enumeración de los participantes en una conversación de un chat

En una conversación, los participantes se pueden enumerar de forma similar a como se agregan.

Use `list_participants` para recuperar los participantes de la conversación.
- De forma opcional, use `results_per_page` para establecer el número máximo de participantes que se van a devolver por página.
- De forma opcional, use `skip` para omitir los participantes hasta una posición especificada en la respuesta.

Un iterador de `[ChatParticipant]` es la respuesta que devuelven los mensajes de la lista.

```python
chat_thread_participants = chat_thread_client.list_participants()
for chat_thread_participant_page in chat_thread_participants.by_page():
    for chat_thread_participant in chat_thread_participant_page:
        print("ChatParticipant: ", chat_thread_participant)
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `python`.

```console
python start-chat.py
```
