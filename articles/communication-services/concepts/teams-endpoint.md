---
title: Punto de conexión de Teams personalizado
titleSuffix: An Azure Communication Services concept document
description: Creación de un punto de conexión de Teams personalizado
author: tomaschladek
manager: nmurav
services: azure-communication-services
ms.author: tchladek
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: acbd6a332d74a9b244f34230ac4b4eb591a1ab2d
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113108070"
---
# <a name="custom-teams-endpoint"></a>Punto de conexión de Teams personalizado

> [!IMPORTANT]
> Para habilitar o deshabilitar la experiencia del punto de conexión de Teams personalizado, rellene [este formulario](https://forms.office.com/r/B8p5KqCH19).

Se puede usar Azure Communication Services para crear puntos de conexión de Teams personalizados para comunicarse con el cliente de Microsoft Teams u otros puntos de conexión de Teams personalizados. Con el punto de conexión de Teams personalizado, puede personalizar la experiencia de voz, vídeo, chat y uso compartido de pantalla para los usuarios de Teams.

Puede usar el SDK de identidad de Azure Communication Services para intercambiar tokens de usuario de AAD por tokens de acceso de Teams. En los diagramas siguientes, se muestra un caso de uso multiinquilino, en el que Fabrikam es el cliente de la empresa Contoso.

## <a name="calling"></a>Llamar 

Las funcionalidades de voz, vídeo y uso compartido de pantalla se proporcionan mediante los SDK de llamadas de Azure Communication Services. En el diagrama siguiente, se muestra información general del proceso que se va a seguir al integrar las experiencias de llamadas con los puntos de conexión de Teams personalizados.

![Proceso para habilitar la característica de llamadas para la experiencia del punto de conexión de Teams personalizado](./media/teams-identities/teams-identity-calling-overview.png)

## <a name="chat"></a>Chat

También puede usar puntos de conexión de Teams personalizados para integrar opcionalmente las funcionalidades de chat mediante Graph API. Más información sobre Graph API en [la documentación](https://docs.microsoft.com/graph/api/channel-post-messages). 


![Proceso para habilitar la característica de chat para la experiencia del punto de conexión de Teams personalizado](./media/teams-identities/teams-identity-chat-overview.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Generación de un token de acceso de Teams](../quickstarts/manage-teams-identity.md)

Puede que los siguientes documentos le resulten interesantes:

- Más información sobre la [interoperabilidad de Teams](./teams-interop.md)
