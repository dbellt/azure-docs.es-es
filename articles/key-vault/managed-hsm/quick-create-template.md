---
title: 'Inicio rápido de Azure: Creación de un HSM administrado mediante una plantilla de Azure Resource Manager'
description: Inicio rápido que muestra cómo crear un HSM administrado de Azure Key Vault mediante una plantilla de Resource Manager
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.custom: mvc
ms.date: 09/15/2020
ms.author: mbaldwin
ms.openlocfilehash: b2cd8578599fe1e80044f1eb388cddae7a292166
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107376875"
---
# <a name="quickstart-create-an-key-vault-managed-hsm-using-an-azure-resource-manager-template"></a>Inicio rápido: Creación de un HSM administrado de Key Vault mediante una plantilla de Azure Resource Manager

Managed HSM es un servicio en la nube que cumple los estándares totalmente administrado, de alta disponibilidad y de un solo inquilino que le permite proteger las claves criptográficas de las aplicaciones en la nube mediante HSM validados de **FIPS 140-2 de nivel 3**.  

Este inicio rápido se centra en el proceso de implementación de una plantilla de Resource Manager para crear un HSM administrado.  La [plantilla de Resource Manager](../../azure-resource-manager/templates/overview.md) es un archivo JSON (notación de objetos JavaScript) que contiene la infraestructura y la configuración del proyecto. La plantilla usa sintaxis declarativa, lo que permite establecer lo que pretende implementar sin tener que escribir la secuencia de comandos de programación para crearla. Si desea más información sobre el desarrollo de plantillas de Resource Manager, consulte la [documentación de Resource Manager](../../azure-resource-manager/index.yml) y la [referencia de la plantilla](/azure/templates/microsoft.keyvault/allversions).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este artículo, debe tener los siguientes elementos:

- Una suscripción a Microsoft Azure. Si no tiene una, puede registrarse para una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial).
- La CLI de Azure, versión 2.12.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte el artículo sobre la [instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para iniciar sesión en Azure mediante la CLI puede escribir:

```azurecli
az login
```

Para más información sobre las opciones de inicio de sesión mediante la CLI, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

## <a name="create-a-manage-hsm"></a>Creación de una instancia de Managed HSM

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-managed-hsm-create/).

El recurso de Azure definido en la plantilla:

* **Microsoft. KeyVault/managedHSMs**: cree un HSM administrado de Azure Key Vault.

Puede encontrar más ejemplos de plantillas de Azure Key Vault [aquí](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault).

La plantilla requiere el identificador de objeto asociado a la cuenta. Para encontrarlo, use el comando [az ad user show](/cli/azure/ad/user#az_ad_user_show) de la CLI de Azure y pase su dirección de correo electrónico en el parámetro `--id`. Puede limitar la salida al identificador de objeto solo con el parámetro `--query`.

```azurecli-interactive
az ad user show --id <your-email-address> --query "objectId"
```

También puede necesitar su identificador de inquilino. Para encontrarla, use el comando[az ad user show](/cli/azure/account#az_account_show) de la CLI de Azure. Puede limitar la salida al identificador de inquilino solo con el parámetro `--query`.

 ```azurecli-interactive
 az account show --query "tenantId"
 ```

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea un almacén de claves y un secreto.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-managed-hsm-create%2Fazuredeploy.json"><img src="../media/deploy-to-azure.svg" alt="deploy to azure"/></a>

2. Seleccione o escriba los siguientes valores.

    A menos que se especifique, utilice el valor predeterminado para crear el almacén de claves y un secreto.

    - **Suscripción**: Seleccione una suscripción de Azure.
    - **Grupo de recursos**: Seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y haga clic en **Aceptar**.
    - **Ubicación**: Seleccione una ubicación. Por ejemplo, **Centro-sur de EE. UU.**
    - **managedHSMName**: escriba un nombre para el HSM administrado.
    - **Id. de inquilino**: la función de la plantilla recupera automáticamente el identificador del inquilino; no cambie el valor predeterminado.  Si no hay ningún valor, escriba el identificador de inquilino que recuperó en [Requisitos previos](#prerequisites).
    * **initialAdminObjectIds**: escriba el identificador de objeto que recuperó en [Requisitos previos](#prerequisites).

3. Seleccione **Comprar**. Una vez que el almacén de claves se haya implementado correctamente, recibirá una notificación:

Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un HSM administrado. Este HSM administrado no será completamente funcional hasta que se active. Para aprender a activar un HSM, consulte el apartado sobre la [activación del HSM administrado](quick-create-cli.md#activate-your-managed-hsm).

- Lea una [introducción a Managed HSM](overview.md)
- Obtenga información sobre la [administración de claves en un HSM administrado](key-management.md)
- Consulte [Procedimientos recomendados de Managed HSM](best-practices.md)
