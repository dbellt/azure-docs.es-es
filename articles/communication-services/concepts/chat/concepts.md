---
title: Conceptos de chat en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre los conceptos de chat de Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 05d20b8c07e618c7d8aef8bce7c437c1b3d392ac
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108290081"
---
# <a name="chat-concepts"></a>Conceptos de chat 

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include-chat.md)]

Los SDK de chat de Azure Communication Services se pueden usar para agregar chats de texto en tiempo real a las aplicaciones. En esta página se resumen las funcionalidades y conceptos clave del chat.    

Consulte [Introducción al SDK de chat](./sdk-features.md) para más información sobre las funcionalidades y los lenguajes específicos del SDK.  

## <a name="chat-overview"></a>Introducción al chat    

Las conversaciones de chat se producen dentro de los hilos o **conversaciones de chat**. Las conversaciones de chat tienen las propiedades siguientes:

- Una conversación de chat se identifica de forma única mediante su elemento `ChatThreadId`. 
- Las conversaciones de chat pueden tener uno o varios usuarios como participantes que pueden enviar mensajes. 
- Un usuario puede formar parte de una o varias conversaciones de chat. 
- Solo los participantes de una conversación tienen acceso a una conversación de chat determinada y solo ellos pueden realizar operaciones en la conversación de chat. Estas operaciones incluyen el envío y la recepción de mensajes, la adición de participantes y la eliminación de participantes. 
- Los usuarios se agregan automáticamente como un participante a cualquier conversación de chat que creen.

### <a name="user-access"></a>Acceso de usuarios
Normalmente, el creador de la conversación y los participantes tienen el mismo nivel de acceso a la conversación y pueden ejecutar todas las operaciones relacionadas disponibles en el SDK, incluida la eliminación. Los participantes no tienen acceso de escritura a los mensajes enviados por otros participantes, lo que significa que solo el remitente del mensaje puede actualizar o eliminar los mensajes enviados. Si otro participante intenta hacerlo, obtendrá un error. 

Si desea limitar el acceso a las características del chat para un conjunto de usuarios, puede configurar el acceso como parte del servicio de confianza. El servicio de confianza es el servicio que organiza la autenticación y autorización de los participantes del chat. Lo exploraremos con más detalle a continuación.  

### <a name="chat-data"></a>Datos del chat 
Communication Services almacena el historial de chat hasta que se elimina explícitamente. Los participantes de la conversación de chat pueden usar `ListMessages` para ver el historial de mensajes de una conversación determinada. Los usuarios que se quitaron de una conversación de chat podrán ver el historial de mensajes anterior, pero no podrán enviar ni recibir nuevos mensajes como parte de esa conversación de chat. Una conversación completamente inactiva sin participantes se eliminará automáticamente después de 30 días. Para más información sobre los datos que almacena Communication Services, consulte la documentación sobre [privacidad](../privacy.md).  

### <a name="service-limits"></a>Límites de servicio  
- El número máximo de participantes permitido en una conversación de chat es de 250.   
- El tamaño máximo de mensaje permitido es de aproximadamente 28 KB.  
- En el caso de las conversaciones de chat con más de 20 participantes, no se admiten las características de confirmación de lectura e indicador de escritura.    

## <a name="chat-architecture"></a>Arquitectura del chat    

Hay dos componentes principales en la arquitectura de chat: El 1) servicio de confianza y la 2) aplicación cliente.    

:::image type="content" source="../../media/chat-architecture.png" alt-text="Diagrama que muestra la arquitectura de chat de Communication Services."::: 

 - **Servicio de confianza:** para administrar correctamente una sesión de chat, necesita un servicio que le ayude a conectarse a Communication Services mediante la cadena de conexión del recurso. Este servicio es responsable de la creación de conversaciones de chat, la adición y eliminación de participantes, y la emisión de tokens de acceso a los usuarios. Puede encontrar más información sobre los tokens de acceso en la guía de inicio rápido sobre [tokens de acceso](../../quickstarts/access-tokens.md).  
 - **Aplicación cliente:** la aplicación cliente se conecta al servicio de confianza y recibe los tokens de acceso utilizados por los usuarios para conectarse directamente a Communication Services. Una vez que el servicio de confianza haya creado la conversación de chat y agregado usuarios como participantes, estos pueden usar la aplicación cliente para conectarse a la conversación de chat y enviar mensajes. Use la característica de notificaciones en tiempo real, que se explica a continuación, en la aplicación cliente para suscribirse a las actualizaciones de los mensajes y la conversación por parte de otros participantes.
    
        
## <a name="message-types"></a>Tipos de mensaje    

Como parte del historial de mensajes, el chat comparte los mensajes generados por el usuario, así como los mensajes generados por el sistema. Los mensajes del sistema se generan cuando se actualiza una conversación de chat y pueden ayudar a identificar cuándo se ha agregado o quitado un participante o cuándo se ha actualizado el tema de la conversación. Al llamar a `List Messages` o `Get Messages` en una conversación de chat, el resultado contendrá ambos tipos de mensajes en orden cronológico.

En el caso de los mensajes generados por el usuario, el tipo de mensaje se puede establecer en `SendMessageOptions` al enviar un mensaje a la conversación de chat. Si no se proporciona ningún valor, Communication Services establecerá como valor predeterminado el tipo `text`. Es importante establecer este valor al enviar código HTML. Cuando se especifica `html`, Communication Services corregirá el contenido para asegurarse de que se represente de forma segura en los dispositivos cliente.
 - `text`: mensaje de texto sin formato creado y enviado por el usuario como parte de una conversación de chat. 
 - `html`: mensaje con formato HTML creado y enviado por el usuario como parte de una conversación de chat. 

Tipos de mensajes del sistema: 
 - `participantAdded`: mensaje del sistema que indica que se han agregado uno o varios miembros a la conversación del chat.
 - `participantRemoved`: mensaje del sistema que indica que se ha eliminado un miembro de la conversación del chat.
 - `topicUpdated`: mensaje del sistema que indica que el tema del subproceso se ha actualizado.

## <a name="real-time-notifications"></a>Notificaciones en tiempo real  

Algunos SDK (como el SDK de chat para JavaScript) admiten notificaciones en tiempo real. Esta característica permite a los clientes escuchar actualizaciones y mensajes entrantes en tiempo real de Communication Services en una conversación de chat sin tener que sondear las API. La aplicación cliente se puede suscribir a los siguientes eventos:
 - `chatMessageReceived`: cuando un participante envía un mensaje nuevo a una conversación de chat.
 - `chatMessageEdited`: cuando se edita un mensaje en una conversación de chat. 
 - `chatMessageDeleted`: cuando se elimina un mensaje en una conversación de chat.   
 - `typingIndicatorReceived`: cuando otro participante envía un indicador de escritura a la conversación de chat.    
 - `readReceiptReceived`: cuando otro participante envía una confirmación de lectura de un mensaje que ha leído.  
 - `chatThreadCreated`: cuando un usuario de Communication Services crea una conversación de chat.    
 - `chatThreadDeleted`: cuando un usuario de Communication Services elimina una conversación de chat.    
 - `chatThreadPropertiesUpdated`: cuando se actualizan las propiedades de la conversación de chat; actualmente, solo se admite la actualización del tema de la conversación. 
 - `participantsAdded`: cuando se agrega un usuario como participante a una conversación de chat.     
 - `participantsRemoved`: cuando se quita un participante existente de la conversación de chat.

Las notificaciones en tiempo real se pueden usar para proporcionar a los usuarios una experiencia de chat en tiempo real. Para enviar notificaciones de inserción de los mensajes que los usuarios perdieron mientras estaban fuera, Communication Services se integra con Azure Event Grid para publicar eventos relacionados con el chat (después de la operación) que se pueden conectar al servicio de notificaciones personalizado de la aplicación. Para más información, consulte [Control de eventos en Azure Communication Services](../../../event-grid/event-schema-communication-services.md?bc=https%3a%2f%2fdocs.microsoft.com%2fen-us%2fazure%2fbread%2ftoc.json&toc=https%3a%2f%2fdocs.microsoft.com%2fen-us%2fazure%2fcommunication-services%2ftoc.json).


## <a name="build-intelligent-ai-powered-chat-experiences"></a>Creación de experiencias de chat inteligentes basadas en inteligencia artificial   

Puede usar las [API de Azure Cognitive Services](../../../cognitive-services/index.yml) con el SDK de chat para crear casos de uso como:

- Permitir a los usuarios conversar entre sí en distintos idiomas.  
- Ayudar a un agente de soporte técnico a clasificar por prioridades los vales mediante la detección de una opinión negativa en un nuevo mensaje entrante de un cliente. 
- Analizar los mensajes entrantes para detectar claves reconocer entidades, así como solicitar información pertinente al usuario en la aplicación en función del contenido del mensaje.

Una manera de lograrlo es hacer que su servicio de confianza actúe como participante de una conversación de chat. Supongamos que desea quiere la traducción de idiomas. Este servicio será responsable de escuchar los mensajes que intercambian otros participantes [1], llamar a Cognitive Services APIs para traducir el contenido al idioma deseado [2, 3] y enviar el resultado traducido como mensaje en la conversación de chat [4].

De este modo, el historial de mensajes contendrá los mensajes originales y los traducidos. En la aplicación cliente, puede agregar lógica para mostrar el mensaje original o el traducido. Consulte [esta guía de inicio rápido](../../../cognitive-services/translator/quickstart-translator.md) para más información sobre cómo usar Cognitive Services APIs para traducir texto a diferentes idiomas. 
    
:::image type="content" source="../media/chat/cognitive-services.png" alt-text="Diagrama que muestra a Cognitive Services interactuando con Communication Services."::: 

## <a name="next-steps"></a>Pasos siguientes   

> [!div class="nextstepaction"] 
> [Introducción al chat](../../quickstarts/chat/get-started.md)    

Puede que los siguientes documentos le resulten interesantes:  
- Familiarícese con el [SDK de chat](sdk-features.md).