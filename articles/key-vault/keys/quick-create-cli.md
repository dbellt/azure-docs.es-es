---
title: Creación y recuperación de los atributos de una clave en Azure Key Vault mediante la CLI de Azure
description: Inicio rápido en el que se muestra cómo establecer y recuperar una clave de Azure Key Vault mediante la CLI de Azure
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.date: 01/27/2021
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: f5281c02f68598b1cf121a6ab8a574f0694f5ea2
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813459"
---
# <a name="quickstart-set-and-retrieve-a-key-from-azure-key-vault-using-azure-cli"></a>Inicio rápido: Establecimiento y recuperación de una clave desde Azure Key Vault mediante la CLI de Azure

En este inicio rápido, creará un almacén de claves en Azure Key Vault con la CLI de Azure. Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). La CLI de Azure se usa para crear y administrar recursos de Azure mediante comandos o scripts. Una vez que haya terminado, almacenará una clave.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Para realizar este inicio rápido es necesaria la versión 2.0.4 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group](../../../includes/key-vault-cli-rg-creation.md)]

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-kv-creation.md)]

## <a name="add-a-key-to-key-vault"></a>Incorporación de una clave a Key Vault

Para agregar una clave al almacén, no tiene más que realizar un par de pasos adicionales. Esta clave la podría usar una aplicación. 

Escriba los siguientes comandos para crear una clave llamada **ExampleKey**:

```azurecli
az keyvault key create --vault-name "<your-unique-keyvault-name>" -n ExampleKey --protection software
```

Ahora puede hacer referencia a esta clave que agregó a Azure Key Vault utilizando su URI. Use **"https://<su-nombre-de-almacén-de-claves-único>.vault.azure.net/keys/ExampleKey"** para obtener la versión actual. 

Para ver la clave previamente almacenada:

```azurecli

az keyvault key show --name "ExampleKey" --vault-name "<your-unique-keyvault-name>"
```

Ya ha creado una instancia de Key Vault, ha almacenado una clave y, posteriormente, la ha recuperado.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-delete-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Key Vault y ha almacenado una clave en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la referencia de los [comandos az keyvault de la CLI de Azure](/cli/azure/keyvault).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-features.md)
