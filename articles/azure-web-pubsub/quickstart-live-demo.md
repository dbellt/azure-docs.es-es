---
title: Demostración en vivo del servicio Azure Web PubSub
description: Inicio rápido para empezar con una demostración en vivo del servicio Azure Web PubSub.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: overview
ms.date: 04/26/2021
ms.openlocfilehash: a516d215a971b2a6cb54d4e73305cc69ce0edf8c
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167080"
---
# <a name="quickstart-get-started-with-live-demo"></a>Inicio rápido: Introducción a una demostración en vivo

El servicio Azure Web PubSub le ayuda a crear aplicaciones web de mensajería en tiempo real mediante WebSockets y el patrón de publicación y suscripción fácilmente. En este inicio rápido, aprenderá los primeros pasos para realizar fácilmente una demostración en vivo.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [create-instance-portal](includes/create-instance-portal.md)]

## <a name="get-started-with-the-live-demo"></a>Primeros pasos con la demostración en vivo

### <a name="get-client-url-with-a-temp-access-token"></a>Obtener la dirección URL del cliente con un token de acceso temporal

Como primer paso, es preciso obtener la dirección URL del cliente de la instancia de Azure Web PubSub. 

- Vaya a Azure Portal y descubra la instancia de Azure Web PubSub.
- Vaya a `Client URL Generator` en la hoja `Key`. 
- Establecer `Roles` correctamente: **Enviar a grupos y** **Unirse o dejar grupos**
- Genere y copie `Client Access URL`. 

### <a name="try-the-live-demo"></a>Prueba de la demostración en vivo 

Con esta demostración en directo, puede unirse a un grupo, o dejarlo, y enviar mensajes a los miembros del grupo fácilmente. 

- Abra la [demostración de Pub/Sub del cliente](https://azure.github.io/azure-webpubsub/demos/clientpubsub.html), pegue `Client Access URL` y seleccione Connect (Conectar). 
- Pruebe diferentes grupos a los que unirse y diferentes grupos a los que enviar mensajes y vea qué mensajes se reciben.
- También puede intentar desactivar `Roles` al generar `Client Access URL` para ver lo que ocurrirá cuando se une a un grupo, o se sale de él, o se envían mensajes a un grupo.