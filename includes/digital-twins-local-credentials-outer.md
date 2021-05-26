---
author: baanders
description: Archivo de inclusión para configurar la autenticación local de DefaultAzureCredential en los ejemplos de Azure Digital Twins, con introducción
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: eacabf431d396fa042bf4dd60d3b32c688df46bf
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110065136"
---
### <a name="set-up-local-azure-credentials"></a>Configuración de credenciales locales de Azure

En este ejemplo se usa [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet&preserve-view=true) (parte de la biblioteca de `Azure.Identity`) para autenticar a los usuarios mediante la instancia de Azure Digital Twins cuando la ejecuta en la máquina local. Para más información sobre las distintas formas en que una aplicación cliente puede autenticarse mediante Azure Digital Twins, consulte [Procedimiento: Escritura de código de autenticación de aplicación](../articles/digital-twins/how-to-authenticate-client.md).

[!INCLUDE [Azure Digital Twins: local credentials prereq (inner)](digital-twins-local-credentials-inner.md)]