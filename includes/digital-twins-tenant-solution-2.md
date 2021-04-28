---
author: baanders
description: cómo incluir el archivo que describe una solución de código para la limitación entre inquilinos con Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/13/2021
ms.author: baanders
ms.openlocfilehash: 2702f3c70d9e6f1a0bdad8695414e2d5fab02411
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589383"
---
En el ejemplo siguiente se muestra cómo establecer un valor de identificador de inquilino para `InteractiveBrowserTenantId` en las opciones de `DefaultAzureCredential`:

:::image type="content" source="../articles/digital-twins/media/troubleshoot-error-404/defaultazurecredentialoptions.png" alt-text="Captura de pantalla del código que muestra el método DefaultAzureCredentialOptions. El valor de InteractiveBrowserTenantId se establece en un valor de identificador de inquilino de ejemplo.":::

Hay opciones similares disponibles para establecer un inquilino para la autenticación con Visual Studio y Visual Studio Code. Para obtener más información sobre las opciones disponibles, consulte la [documentación de DefaultAzureCredentialOptions](/dotnet/api/azure.identity.defaultazurecredentialoptions?view=azure-dotnet&preserve-view=true).