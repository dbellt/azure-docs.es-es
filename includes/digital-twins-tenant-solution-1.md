---
author: baanders
description: como incluir el archivo que describe una solución de token para la limitación entre inquilinos con Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/13/2021
ms.author: baanders
ms.openlocfilehash: 5bc7e8af356ebe5968fe8a74783edc215788e5b4
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589364"
---
Una manera de hacerlo es con el siguiente comando de la CLI, donde `<home-tenant-ID>` es el id. del inquilino de Azure AD que contiene la instancia de Azure Digital Twins predeterminada:

```azurecli-interactive
az account get-access-token --tenant <home-tenant-ID> --resource https://digitaltwins.azure.net
```

Después de realizar la solicitud, la identidad recibirá un token emitido para el recurso de Azure AD *https://digitaltwins.azure.net* , que tiene una notificación de id. de inquilino correspondiente a la instancia de Azure Digital Twins predeterminada. El uso de este token en las solicitudes de API o con el código `Azure.Identity`, permitirá que la identidad federada acceda al recurso de Azure Digital Twins.