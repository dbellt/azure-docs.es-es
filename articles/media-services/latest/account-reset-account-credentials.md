---
title: 'Restablecimiento de las credenciales de la cuenta: CLI'
description: Use el script de la CLI de Azure para restablecer las credenciales de su cuenta y regresar a la configuración del archivo app.config.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: d5604f177484d33255d2923d72b00fae124c0f9a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783600"
---
# <a name="azure-cli-example-reset-the-account-credentials"></a>Ejemplo de la CLI de Azure: restablecimiento de las credenciales de la cuenta

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

El script de la CLI de Azure de este artículo le muestra cómo restablecer las credenciales de su cuenta y regresar a la configuración del archivo app.config.

## <a name="prerequisites"></a>Prerrequisitos

[Cree una cuenta de Media Services](./create-account-howto.md).

## <a name="example-script"></a>Script de ejemplo

```azurecli-interactive
# Update the following variables for your own settings:
resourceGroup=amsResourceGroup
amsAccountName=amsmediaaccountname

az ams account sp reset-credentials \
  --account-name $amsAccountName \
  --resource-group $resourceGroup
 ```

## <a name="next-steps"></a>Pasos siguientes

* [az ams](/cli/azure/ams)
* [Restablecimiento de las credenciales](/cli/azure/ams/account/sp#az_ams_account_sp_reset_credentials)
