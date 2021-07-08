---
title: Preguntas frecuentes sobre el servicio Azure Web PubSub
description: Obtenga respuestas a las preguntas más frecuentes sobre el servicio Azure Web PubSub.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: overview
ms.date: 04/26/2021
ms.openlocfilehash: b16c69d4a51026d3eafcb6449dd3042703269118
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111371625"
---
# <a name="azure-web-pubsub-service-faq"></a>Preguntas frecuentes sobre el servicio Azure Web PubSub

Estas son las preguntas más frecuentes sobre el servicio Azure Web PubSub. 

## <a name="is-azure-web-pubsub-service-ready-for-production-use"></a>¿Está listo el servicio Azure Web PubSub para usarlo en el entorno de producción?

El servicio Azure Web PubSub está en estado de versión preliminar pública y no tiene un Acuerdo de Nivel de Servicio vigente. 

## <a name="how-do-i-choose-between-azure-signalr-service-and-azure-web-pubsub-service"></a>Elección entre Azure SignalR Service y el servicio Azure Web PubSub

[Azure SignalR Service](https://azure.microsoft.com/services/signalr-service) y el [servicio Azure Web PubSub](https://azure.microsoft.com/services/web-pubsub) ayudan a los clientes a crear fácilmente aplicaciones web en tiempo real a gran escala y con alta disponibilidad, y permiten a los clientes centrarse en su lógica de negocios en lugar de en administrar la infraestructura de mensajería. En general, puede elegir Azure SignalR Service si ya usa la biblioteca de SignalR para compilar la aplicación en tiempo real. Si lo que busca es una solución genérica para crear aplicaciones en tiempo real basadas en WebSocket y en el patrón de publicación-suscripción, puede elegir el servicio Azure Web PubSub. El servicio Azure Web PubSub **no** es un sustituto de Azure SignalR Service. Van dirigidos a distintos escenarios.

Azure SignalR Service es más adecuado en los siguientes casos:  

- Ya usa ASP.NET o ASP.NET Core SignalR, principalmente mediante .NET, o necesita integrarse con el ecosistema .NET (como Blazor).
- Hay un cliente de SignalR disponible para su plataforma. 
- Necesita un protocolo establecido que admita una amplia variedad de patrones de llamada (RPC y streaming), transportes (WebSocket, eventos enviados por el servidor y sondeo largo) y con un cliente que administre la duración de la conexión en su nombre. 

El servicio Azure Web PubSub es más adecuado en las siguientes situaciones:  

- Debe crear aplicaciones en tiempo real basadas en la tecnología WebSocket o en la publicación y la suscripción a través de WebSocket.
- Quiere crear su propio subprotocolo o usar protocolos avanzados existentes a través de WebSocket (por ejemplo, MQTT, AMQP sobre WebSocket). 
- Busca un servidor ligero, por ejemplo, que envíe mensajes al cliente sin pasar por el back-end configurado.  

##  <a name="where-does-my-data-reside"></a>¿Dónde residen mis datos?

El servicio Azure Web PubSub funciona como un servicio de procesador de datos. No almacenará ningún contenido del cliente y la residencia de datos se incluye intencionadamente. Si usa el servicio Azure Web PubSub junto con otros servicios de Azure, como Azure Storage para diagnósticos, consulte [estas notas del producto](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/) para obtener instrucciones sobre cómo mantener la residencia de los datos en las regiones de Azure.