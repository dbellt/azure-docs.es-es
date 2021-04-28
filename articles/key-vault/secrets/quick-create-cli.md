---
title: 'Inicio rápido: Establecimiento y recuperación de un secreto desde Azure Key Vault'
description: Guía de inicio rápido que muestra cómo establecer y recuperar un secreto de Azure Key Vault mediante la CLI de Azure
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-azurecli
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: dc5fa0eeabc437f16ca64c34b4bb72e62c0c3b21
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107815043"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-azure-cli"></a>Inicio rápido: Establecimiento y recuperación de un secreto desde Azure Key Vault mediante la CLI de Azure

En este inicio rápido, creará un almacén de claves en Azure Key Vault con la CLI de Azure. Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). La CLI de Azure se usa para crear y administrar recursos de Azure mediante comandos o scripts. Una vez que haya terminado, almacenará un secreto.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Para realizar este inicio rápido es necesaria la versión 2.0.4 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group](../../../includes/key-vault-cli-rg-creation.md)]

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-kv-creation.md)]

## <a name="add-a-secret-to-key-vault"></a>Incorporación de un secreto a Key Vault

Para agregar un secreto al almacén, simplemente debe realizar un par de pasos adicionales. Esta contraseña la podría usar una aplicación. La contraseña se denominará **ExamplePassword** y almacenará el valor **hVFkk965BuUv**.

Use el comando [az keyvault secret set](/cli/azure/keyvault/secret#az_keyvault_secret_set) de la CLI de Azure que se muestra a continuación para crear un secreto en Key Vault denominado **ExamplePassword** que almacenará el valor **hVFkk965BuUv**:

```azurecli
az keyvault secret set --vault-name "<your-unique-keyvault-name>" --name "ExamplePassword" --value "hVFkk965BuUv"
```

## <a name="retrieve-a-secret-from-key-vault"></a>Recuperar un secreto del almacén de claves

Ahora puede hacer referencia a esta clave que agregó a Azure Key Vault utilizando su URI. Use **'https://<nombre-del-almacén-de-claves>.vault.azure.net/secrets/ExamplePassword'** para obtener la versión actual.

Para ver el valor contenido en el secreto como texto sin formato:

```azurecli
az keyvault secret show --name "ExamplePassword" --vault-name "<your-unique-keyvault-name>" --query "value"
```

Ya ha creado una instancia de Key Vault, ha almacenado un secreto y, posteriormente, lo ha recuperado.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-delete-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Key Vault y ha almacenado un secreto en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Aprenda a [almacenar secretos multilínea en Key Vault](multiline-secrets.md)
- Consulte la referencia de los [comandos az keyvault de la CLI de Azure](/cli/azure/keyvault).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-features.md)
