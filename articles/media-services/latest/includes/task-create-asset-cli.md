---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/18/2020
ms.author: inhenkel
ms.custom: CLI, devx-track-azurecli
ms.openlocfilehash: e16e3dc6788fb30a87a78fd6d6077087f461cfa7
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107512767"
---
<!--Create a media services asset CLI-->

El siguiente comando de la CLI de Azure crea un nuevo recurso de Media Services. Reemplace los valores `assetName`, `amsAccountName` y `resourceGroup` por los valores con los que esté trabajando actualmente.

```azurecli
az ams asset create -n assetName -a amsAccountName -g resourceGroup
```
