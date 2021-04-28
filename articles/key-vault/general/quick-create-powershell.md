---
title: 'Guía de inicio rápido: creación de una instancia de Azure Key Vault con Azure PowerShell'
description: Guía de inicio rápido que muestra cómo crear una instancia de Azure Key Vault mediante Azure PowerShell
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: quickstart
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: cc6d9ca2621a56242d7472a088e55651f5502c9c
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107814809"
---
# <a name="quickstart-create-a-key-vault-using-powershell"></a>Inicio rápido: Creación de un almacén de claves mediante PowerShell

Azure Key Vault es un servicio de almacenamiento seguro en la nube para [claves](../keys/index.yml), [secretos](../secrets/index.yml) y [certificados](../certificates/index.yml). Para obtener más información sobre Key Vault, consulte el artículo [Acerca de Azure Key Vault](overview.md); para obtener más información sobre lo que se puede almacenar en un almacén de claves, consulte el artículo [Acerca de las claves, secretos y certificados](about-keys-secrets-certificates.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

En esta guía de inicio rápido, creará un almacén de claves con [Azure PowerShell](/powershell/azure/). Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0.0 del módulo de Azure PowerShell, o cualquier versión posterior. Escriba `$PSVersionTable.PSVersion` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group](../../../includes/key-vault-powershell-rg-creation.md)]

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-kv-creation.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-delete-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Key Vault mediante Azure PowerShell. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](overview.md).
- Consulte la referencia para los [cmdlets de Key Vault de Azure PowerShell](/powershell/module/az.keyvault/).
- Consulte [Introducción a la seguridad de Azure Key Vault](security-features.md)

