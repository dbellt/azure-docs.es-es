---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: c52dfe33de673f6ad529a3065b9567a685acfad5
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110062559"
---
<!--Create a media services account -->

El siguiente comando de la CLI de Azure crea una nueva cuenta de Media Services. Puede reemplazar los valores siguientes: `your-media-services-account-name`, `your-storage-account-name`y `your-resource-group-name`. El comando supone que ya ha creado un grupo de recursos y una cuenta de almacenamiento.

```azurecli-interactive
az ams account create --name <your-media-services-account-name> -g <your-resource-group-name> --storage-account <your-storage-account-name> -l <your-region>
```

Ejemplo de respuesta JSON:

```json
{
  "id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/your-resource-group-name/providers/Microsoft.Media/mediaservices/your-media-services-account-name",
  "location": "your-region",
  "mediaServiceId": "the-media-services-account-id",
  "name": "your-media-services-account-name",
  "resourceGroup": "your-resource-group-name",
  "storageAccounts": [
    {
      "id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/your-resource-group-name/providers/Microsoft.Storage/storageAccounts/your-storage-account-name",
      "resourceGroup": "your-resource-group-name",
      "type": "Primary"
    }
  ],
  "tags": null,
  "type": "Microsoft.Media/mediaservices"
}
```
