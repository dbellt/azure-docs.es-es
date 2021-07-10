---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: 4dae6dc7702781231965f7dc66423fde941f4185
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110103314"
---
<!-- ### Create a Storage Blob Contributor role -->

El siguiente comando crea un rol de Colaborador de Storage Blob. 

Cambie `assignee` por el valor de `principalId`. El comando supone que ya ha creado un grupo de recursos y una cuenta de almacenamiento.  Use `your-resource-group-name` y `your-storage-account-name` como parte del valor de `scope`, como se muestra en el comando siguiente:

```azurecli-interactive
az role assignment create --assignee 00000000-0000-0000-000000000000 --role "Storage Blob Data Contributor" --scope "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/<your-resource-group-name>/providers/Microsoft.Storage/storageAccounts/<your-storage-account-name>"
```

Ejemplo de respuesta JSON:

```json
{
  "canDelegate": null,
  "condition": null,
  "conditionVersion": null,
  "description": null,
  "id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/your-resource-group-name/providers/Microsoft.Storage/storageAccounts/your-storage-account-name/providers/Microsoft.Authorization/roleAssignments/00000000-0000-0000-000000000000",
  "name": "00000000-0000-0000-000000000000",
  "principalId": "00000000-0000-0000-000000000000",
  "principalType": "ServicePrincipal",
  "resourceGroup": "your-resource-group-name",
  "roleDefinitionId": "/subscriptions/00000000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/00000000-0000-0000-000000000000",
  "scope": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/your-resource-group-name/providers/Microsoft.Storage/storageAccounts/your-storage-account-name",
  "type": "Microsoft.Authorization/roleAssignments"
}

```
