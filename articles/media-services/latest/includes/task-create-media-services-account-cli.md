---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: 07b0897de3fce6a108836f1e72ce956a3a8092f1
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107893181"
---
<!--Create a media services account -->

El siguiente comando de la CLI de Azure crea una nueva cuenta de Media Services. Puede reemplazar los valores siguientes: `amsaccount`, `storageaccountforams` (deben coincidir con el valor que especificó para la cuenta de almacenamiento) y `amsResourceGroup` (debe coincidir con el valor que especificó para el grupo de recursos).

```azurecli
az ams account create --name amsaccount -g amsResourceGroup --storage-account storageaccountforams -l westus2
```
