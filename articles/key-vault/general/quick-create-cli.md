---
title: 'Inicio rápido: Creación de un almacén de claves de Azure Key Vault con la CLI de Azure'
description: Inicio rápido que muestra cómo crear un almacén de claves de Azure Key Vault mediante la CLI de Azure
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: quickstart
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: 588bee2922ee44f3f89b5d252b5b4a6991d26b82
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107815133"
---
# <a name="quickstart-create-a-key-vault-using-the-azure-cli"></a>Inicio rápido: Creación de un almacén de claves mediante la CLI de Azure

Azure Key Vault es un servicio de almacenamiento seguro en la nube para [claves](../keys/index.yml), [secretos](../secrets/index.yml) y [certificados](../certificates/index.yml). Para obtener más información sobre Key Vault, consulte el artículo [Acerca de Azure Key Vault](overview.md); para obtener más información sobre lo que se puede almacenar en un almacén de claves, consulte el artículo [Acerca de las claves, secretos y certificados](about-keys-secrets-certificates.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Para realizar este inicio rápido es necesaria la versión 2.0.4 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group](../../../includes/key-vault-cli-rg-creation.md)]

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-kv-creation.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-delete-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado un almacén de claves y lo ha eliminado. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](overview.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](security-features.md)
- Consulte la referencia de los [comandos az keyvault de la CLI de Azure](/cli/azure/keyvault).

