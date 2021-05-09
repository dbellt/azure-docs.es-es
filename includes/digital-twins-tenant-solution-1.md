---
author: baanders
description: como incluir el archivo que describe una solución de token para la limitación entre inquilinos con Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/13/2021
ms.author: baanders
ms.openlocfilehash: 08c48b3600a975f43a5ad94e2cc6efd88047d0d8
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205670"
---
Una manera de hacerlo es con el siguiente comando de la CLI, donde `<home-tenant-ID>` es el id. del inquilino de Azure AD que contiene la instancia de Azure Digital Twins predeterminada:

```azurecli-interactive
az account get-access-token --tenant <home-tenant-ID> --resource https://digitaltwins.azure.net
```

Después de realizar la solicitud, la identidad recibirá un token emitido para el recurso de Azure AD `https://digitaltwins.azure.net` , que tiene una notificación de id. de inquilino correspondiente a la instancia de Azure Digital Twins predeterminada. El uso de este token en las solicitudes de API o con el código `Azure.Identity`, permitirá que la identidad federada acceda al recurso de Azure Digital Twins.
