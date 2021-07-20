---
title: Demostración en vivo del servicio Azure Web PubSub
description: Inicio rápido para empezar con una demostración en vivo del servicio Azure Web PubSub.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: overview
ms.date: 04/26/2021
ms.openlocfilehash: e35ef66c038748e31ba218a56adb8111374d2339
ms.sourcegitcommit: 6bd31ec35ac44d79debfe98a3ef32fb3522e3934
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2021
ms.locfileid: "113219613"
---
# <a name="quickstart-get-started-with-chatroom-live-demo"></a>Inicio rápido: Introducción a la demostración en directo de una sala de chat

El servicio Azure Web PubSub le ayuda a crear aplicaciones web de mensajería en tiempo real mediante WebSockets y el patrón de publicación y suscripción fácilmente. La [demostración en directo de una sala de chat](https://azure.github.io/azure-webpubsub/demos/clientpubsub.html) muestra la funcionalidad de mensajería en tiempo real proporcionada por Azure Web PubSub. Con esta demostración en directo, puede unirse fácilmente a un grupo de chat y enviar mensajes en tiempo real a un grupo específico. 

:::image type="content" source="media/quickstart-live-demo/chat-live-demo.gif" alt-text="Uso de la demostración en directo de una sala de chat.":::

En este inicio rápido, aprenderá los primeros pasos para realizar fácilmente una demostración en vivo.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [create-instance-portal](includes/create-instance-portal.md)]

## <a name="get-started-with-the-chatroom-live-demo"></a>Introducción a la demostración en directo de una sala de chat

### <a name="get-client-url-with-a-temp-access-token"></a>Obtener la dirección URL del cliente con un token de acceso temporal

Como primer paso, es preciso obtener la dirección URL del cliente de la instancia de Azure Web PubSub. 

- Vaya a Azure Portal y descubra la instancia de Azure Web PubSub.
- Vaya a `Client URL Generator` en la hoja `Key`. 
- Establecer `Roles` correctamente: **Enviar a grupos y** **Unirse o dejar grupos**
- Genere y copie `Client Access URL`. 

:::image type="content" source="media/quickstart-live-demo/generate-client-url.png" alt-text="Captura de pantalla de la generación de la dirección URL del cliente.":::

### <a name="try-the-live-demo"></a>Prueba de la demostración en vivo 

Con esta demostración en directo, puede unirse a un grupo, o dejarlo, y enviar mensajes a los miembros del grupo fácilmente. 

- Abra la [demostración en directo de una sala de chat](https://azure.github.io/azure-webpubsub/demos/clientpubsub.html), pegue el valor de `Client Access URL` y seleccione Connect (Conectar). 

:::image type="content" source="media/quickstart-live-demo/paste-client-access-url.png" alt-text="Captura de pantalla del pegado de la dirección URL del cliente en la demostración en directo.":::

> [!NOTE]
>  La **dirección URL de acceso de cliente** es una herramienta útil que se proporciona en el portal para simplificar la experiencia de introducción. También puede usar esta dirección URL de acceso de cliente para realizar algunas pruebas rápidas de conexión. Para escribir su propia aplicación, proporcionamos SDK en 4 lenguajes para ayudarle a generar la dirección URL. 

- Pruebe diferentes grupos a los que unirse y diferentes grupos a los que enviar mensajes y vea qué mensajes se reciben. Por ejemplo:
    - Haga que dos clientes se unan al mismo grupo. Verá que el mensaje se difunde a los miembros del grupo. 
    - Haga que dos clientes se unan a grupos diferentes. Verá que el cliente no puede recibir el mensaje si no es miembro del grupo. 
- También puede probar a desactivar `Roles` al generar `Client Access URL` para ver lo que ocurre cuando se une a un grupo o se envían mensajes a un grupo. Por ejemplo:
    - Desactive el permiso `Send to Groups`. Verá que el cliente no puede enviar mensajes al grupo. 
    - Desactive el permiso `Join/Leave Groups`. Verá que el cliente no puede unirse a un grupo. 

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido la funcionalidad de mensajería en tiempo real con la demostración en directo de una sala de chat. Ahora, puede empezar a crear su propia aplicación. 

> [!div class="nextstepaction"]
> [Inicio rápido: Publicación y suscripción de mensajes en Azure Web PubSub](https://azure.github.io/azure-webpubsub/getting-started/publish-messages/js-publish-message)

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de una sala de chat simple con Azure Web PubSub](https://azure.github.io/azure-webpubsub/getting-started/create-a-chat-app/js-handle-events)

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de una aplicación de chat simple sin servidor con Azure Functions y el servicio Azure Web PubSub](./quickstart-serverless.md)

> [!div class="nextstepaction"]
> [Explore más ejemplos de Azure Web PubSub](https://github.com/Azure/azure-webpubsub/tree/main/samples).