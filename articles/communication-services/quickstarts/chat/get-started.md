---
title: 'Inicio rápido: Incorporación de chat a una aplicación'
titleSuffix: An Azure Communication Services quickstart
description: En esta guía de inicio rápido se muestra cómo agregar un chat de Communication Services a la aplicación.
author: fanche
manager: phans
services: azure-communication-services
ms.author: mikben
ms.date: 06/30/2021
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python-swift-android
ms.openlocfilehash: dfe1e6d0d357eb77a8000123c6e9d5de606dafe9
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113113374"
---
# <a name="quickstart-add-chat-to-your-app"></a>Inicio rápido: Incorporación de chat a una aplicación

Comience a usar Azure Communication Services mediante el SDK de chat de Communication Services para agregar chat en tiempo real a la aplicación. En esta guía de inicio rápido, usaremos el SDK de chat para crear conversaciones de chat que permitan a los usuarios mantener conversaciones entre sí. Para obtener más información acerca de los conceptos de chat, visite la [documentación conceptual de chat](../../concepts/chat/concepts.md).

::: zone pivot="programming-language-javascript"
[!INCLUDE [Chat with JavaScript SDK](./includes/chat-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Chat with Python SDK](./includes/chat-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Chat with Java SDK](./includes/chat-java.md)]
::: zone-end

::: zone pivot="programming-language-android"
[!INCLUDE [Chat with Android SDK](./includes/chat-android.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [Chat with C# SDK](./includes/chat-csharp.md)]
::: zone-end

::: zone pivot="programming-language-swift"
[!INCLUDE [Chat with iOS SDK](./includes/chat-swift.md)]
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Creación de un cliente de chat
> * Creación de un subproceso con dos usuarios
> * Envío de un mensaje al subproceso
> * Recepción de mensajes de un subproceso
> * Eliminación de usuarios de un subproceso

> [!div class="nextstepaction"]
> [Prueba de la aplicación de elemento principal de chat](../../samples/chat-hero-sample.md)

Puede que también le interese:

 - Obtener más información sobre los [conceptos de chat](../../concepts/chat/concepts.md).
 - Familiarícese con el [SDK de chat](../../concepts/chat/sdk-features.md).
