---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: 26ae372b6e431247d2038445daafcb3c997a232d
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107512274"
---
<!--Create a media services account -->

El siguiente comando de la CLI de Azure crea una nueva cuenta de Media Services. Puede reemplazar los valores siguientes: `amsaccount`, `storageaccountforams` (deben coincidir con el valor que especificó para la cuenta de almacenamiento) y `amsResourceGroup` (debe coincidir con el valor que especificó para el grupo de recursos).

```azurecli
az ams account create --name amsaccount -g amsResourceGroup --storage-account storageaccountforams -l westus2
```
