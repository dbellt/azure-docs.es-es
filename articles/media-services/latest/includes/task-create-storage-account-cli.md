---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: 6f6209eaee172ece5000d48c48b6cc38e4385017
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110060146"
---
<!-- ### Create a storage account -->

El siguiente comando crea una cuenta de almacenamiento asociada a una cuenta de Media Services. 

Cambie `your-storage-account-name` por un nombre único con una longitud inferior a 24 caracteres. El comando supone que ya ha creado un grupo de recursos.  Use ese nombre de grupo de recursos para `your-resource-group-name`. Use el nombre de la región de su elección para `your-region`.

```azurecli-interactive
az storage account create --name <your-storage-account-name> --kind StorageV2 --sku Standard_LRS -l <your-region> -g <your-resource-group-name>
```

Ejemplo de respuesta JSON:

```json
{
  "accessTier": "Hot",
  "allowBlobPublicAccess": null,
  "azureFilesIdentityBasedAuthentication": null,
  "blobRestoreStatus": null,
  "creationTime": "2021-05-12T22:10:22.058640+00:00",
  "customDomain": null,
  "enableHttpsTrafficOnly": true,
  "encryption": {
    "keySource": "Microsoft.Storage",
    "keyVaultProperties": null,
    "requireInfrastructureEncryption": null,
    "services": {
      "blob": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2021-05-12T22:10:22.152394+00:00"
      },
      "file": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2021-05-12T22:10:22.152394+00:00"
      },
      "queue": null,
      "table": null
    }
  },
  "failoverInProgress": null,
  "geoReplicationStats": null,
  "id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/your-resource-group-name/providers/Microsoft.Storage/storageAccounts/your-storage-account-name",
  "identity": null,
  "isHnsEnabled": null,
  "kind": "StorageV2",
  "largeFileSharesState": null,
  "lastGeoFailoverTime": null,
  "location": "your-region",
  "minimumTlsVersion": null,
  "name": "your-storage-account-name",
  "networkRuleSet": {
    "bypass": "AzureServices",
    "defaultAction": "Allow",
    "ipRules": [],
    "virtualNetworkRules": []
  },
  "primaryEndpoints": {
    "blob": "https://your-storage-account-name.blob.core.windows.net/",
    "dfs": "https://your-storage-account-name.dfs.core.windows.net/",
    "file": "https://your-storage-account-name.file.core.windows.net/",
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://your-storage-account-name.queue.core.windows.net/",
    "table": "https://your-storage-account-name.table.core.windows.net/",
    "web": "your-storage-account-name.z5.web.core.windows.net/"
  },
  "primaryLocation": "your-region",
  "privateEndpointConnections": [],
  "provisioningState": "Succeeded",
  "resourceGroup": "your-resource-group-name",
  "routingPreference": null,
  "secondaryEndpoints": null,
  "secondaryLocation": null,
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": null,
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}
```