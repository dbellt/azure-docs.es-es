---
title: 'Inicio rápido: Creación y administración de tokens de acceso'
titleSuffix: An Azure Communication Services quickstart
description: Aprenda a administrar identidades y tokens de acceso mediante el SDK de identidades de Azure Communication Services.
author: tomaschladek
manager: nmurav
services: azure-communication-services
ms.author: tchladek
ms.date: 06/30/2021
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: d4836609aac24a414cb112988a0524de1e7a71c1
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113113293"
---
# <a name="quickstart-create-and-manage-access-tokens"></a>Inicio rápido: Creación y administración de tokens de acceso

Para empezar a usar Azure Communication Services, utilice el SDK de identidades de Communication Services. Permite crear identidades y administrar los tokens de acceso. La identidad representa la entidad de la aplicación en Azure Communication Services (por ejemplo, usuario o dispositivo). Los tokens de acceso permiten que los SDK de chat y de llamadas se autentiquen directamente en Azure Communication Services. Se recomienda generar tokens de acceso en un servicio del lado servidor. Los tokens de acceso se usan para inicializar los SDK de Communication Services en los dispositivos cliente.

Los precios que se muestran en las imágenes de este tutorial son ejemplos solamente.

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/user-access-token-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/user-access-token-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/user-access-token-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/user-access-token-java.md)]
::: zone-end

La salida de la aplicación describe cada acción que se completa:
<!---cSpell:disable --->
```console
Azure Communication Services - Access Tokens Quickstart

Created an identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502

Issued an access token with 'voip' scope that expires at 30/03/21 08:09 09 AM:
<token signature here>

Created an identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-1ce9-31b4-54b7-a43a0d006a52

Issued an access token with 'voip' scope that expires at 30/03/21 08:09 09 AM:
<token signature here>

Successfully revoked all access tokens for identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502

Deleted the identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502
```
<!---cSpell:enable --->

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](./create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a:

> [!div class="checklist"]
> * Administrar identidades
> * Emitir tokens de acceso
> * Usar el SDK de identidades de Communication Services


> [!div class="nextstepaction"]
> [Adición de la llamada de voz a una aplicación](./voice-video-calling/getting-started-with-calling.md)

Puede que también le interese:

 - [Información sobre la autenticación](../concepts/authentication.md)
 - [Incorporación de chat a una aplicación](./chat/get-started.md)
 - [Información sobre la arquitectura de cliente y servidor](../concepts/client-and-server-architecture.md)
