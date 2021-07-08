---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: f1b77e4cee809d7ddb7bbbf7de6c5ac483b150e3
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110060151"
---
<!-- Create a resource group -->

Use el siguiente comando para crear un grupo de recursos. Cambie `your-resource-group-name` por el nombre quiere asignar al grupo de recursos.

Cambie `your-region` por la región geográfica que se usará para almacenar los registros de elementos multimedia y de metadatos de la cuenta de Media Services. Esta región se utilizará para procesar y transmitir contenido multimedia.

```azurecli-interactive
az group create --name <your-resource-group-name> --location <your-region>
```

Ejemplo de respuesta JSON:

```json
{
  "id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/your-resource-group-name",
  "location": "your-region",
  "managedBy": null,
  "name": "your-resource-group-name",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```
