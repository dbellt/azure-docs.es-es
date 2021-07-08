---
title: 'Inicio rápido: Obtención de un token de retransmisión de red'
description: Obtenga información sobre cómo recuperar un token STUN/TURN mediante Azure Communication Services
author: shahen
manager: anvalent
services: azure-communication-services
ms.author: shahen
ms.date: 05/21/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.custom: tracking-python, devx-track-javascript
zone_pivot_groups: acs-js-csharp
ms.openlocfilehash: 2fc0bfeaef1d3e0e49d1597f41582b6d79d9a3db
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026932"
---
# <a name="quickstart-get-a-network-relay-token"></a>Inicio rápido: Obtención de un token de retransmisión de red

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

En esta guía de inicio rápido se muestra cómo recuperar un token de retransmisión de red para acceder a los servidores TURN de Azure Communication Services

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree su cuenta de forma gratuita](https://azure.microsoft.com/free).
- Un recurso de Azure Communication Services activo, consulte [Creación de un recurso de Communication Services](./create-communication-resource.md) si no tiene uno.

::: zone pivot="programming-language-csharp"
[!INCLUDE [Get a network relay token with .NET](./includes/relay-token-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Get a network relay token with JavaScript](./includes/relay-token-js.md)]
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a un recurso de Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](./create-communication-resource.md#clean-up-resources).
