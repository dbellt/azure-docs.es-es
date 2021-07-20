---
title: Inicio rápido de las API de grabación de llamadas con Azure Communication Services
titleSuffix: An Azure Communication Services quickstart document
description: Proporciona un ejemplo de inicio rápido para las API de grabación de llamadas.
author: ravithanneeru
manager: joseys
services: azure-communication-services
ms.author: joseys
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
zone_pivot_groups: acs-csharp-java
ms.openlocfilehash: f4dfd0701c2eeeaa5ba3b2be20b9448ca2093601
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113111813"
---
# <a name="call-recording-api-quickstart"></a>Inicio rápido de las API de grabación de llamadas
Este inicio rápido le permite empezar a grabar llamadas de voz y vídeo. En este inicio rápido se da por supuesto que ya ha utilizado el [SDK del cliente de llamadas](get-started-with-video-calling.md) para crear la experiencia de llamadas del usuario final. Con las **API y SDK del servidor de llamadas** puede habilitar y administrar las grabaciones. 

::: zone pivot="programming-language-csharp"
[!INCLUDE [Build Call Recording server sample with C#](./includes/call-recording-samples/recording-server-csharp.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Build Call Recording server sample with Java](./includes/call-recording-samples/recording-server-java.md)]
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos.

- Consulte nuestro [ejemplo de elementos principales de una llamada](../../samples/calling-hero-sample.md).
- Más información sobre la [llamada a las funcionalidades de SDK](./calling-client-samples.md)
- Más información sobre [cómo funciona la llamada](../../concepts/voice-video-calling/about-call-types.md)
