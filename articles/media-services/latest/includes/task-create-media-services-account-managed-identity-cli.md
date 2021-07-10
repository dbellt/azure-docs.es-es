---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: 72dc27bf1600d298d41a034b081b216d3d77c94c
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110103249"
---
<!--Create a media services account -->

El siguiente comando de la CLI de Azure crea una nueva cuenta de Media Services. Reemplace los siguientes valores: `your-media-services-account-name`, `your-storage-account-name` y `your-resource-group-name` por los nombres que quiere usar. El comando supone que ya ha creado un grupo de recursos y una cuenta de almacenamiento. 

Proporciona a la cuenta de Media Services una identidad administrada asignada por el sistema con la marca `--mi-system-assigned`.

```azurecli-interactive

az ams account create --name <your-media-services-account-name> --resource-group <your-resource-group-name> --mi-system-assigned --storage-account <your-storage-account-name>

```

Ejemplo de respuesta JSON:

```json
{
  "encryption": {
    "keyVaultProperties": null,
    "type": "SystemKey"
  },
  "id": "/subscriptions/00000000-0000-0000-0000-00000000/resourceGroups/your-resource-group/providers/Microsoft.Media/mediaservices/your-media-services-account-name",
  "identity": {
    "principalId": "00000000-0000-0000-0000-00000000",
    "tenantId": "00000000-0000-0000-0000-00000000",
    "type": "SystemAssigned"
  },
  "location": "your-region",
  "mediaServiceId": "00000000-0000-0000-0000-00000000",
  "name": "your-media-services-account-name",
  "resourceGroup": "your-resource-group",
  "storageAccounts": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-00000000/resourceGroups/mediatest1rg/providers/Microsoft.Storage/storageAccounts/your-storage-account-name",
      "resourceGroup": "your-resource-group",
      "type": "Primary"
    }
  ],
  "storageAuthentication": "System",
  "systemData": {
    "createdAt": "2021-05-14T21:25:12.3492071Z",
    "createdBy": "you@example.com",
    "createdByType": "User",
    "lastModifiedAt": "2021-05-14T21:25:12.3492071Z",
    "lastModifiedBy": "you@example.com",
    "lastModifiedByType": "User"
  },
  "tags": null,
  "type": "Microsoft.Media/mediaservices"
}
```
