---
title: Creación de un grupo de SQL dedicado (antes SQL DW) mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un grupo de SQL de Azure Synapse Analytics mediante una plantilla de Azure Resource Manager.
services: azure-resource-manager
author: julieMSFT
ms.service: azure-resource-manager
ms.topic: quickstart
ms.author: jrasnick
ms.date: 06/09/2020
ms.custom: devx-track-azurepowershell - subject-armqs - mode-arm
ms.openlocfilehash: 02eb32fc66b034f7480b9bf2a5319c29c05c94a4
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111902778"
---
# <a name="quickstart-create-an-azure-synapse-analytics-dedicated-sql-pool-formerly-sql-dw-by-using-an-arm-template"></a>Inicio rápido: Creación de un grupo de SQL dedicado de Azure Synapse Analytics (antes SQL DW) mediante una plantilla de Resource Manager

Esta plantilla de Azure Resource Manager creará un grupo de SQL dedicado (antes SQL DW) con la tecnología Cifrado de datos transparente habilitada. Un grupo de SQL dedicado (antes SQL DW) hace referencia a las características de almacenamiento de datos empresariales que están disponibles con carácter general en Azure Synapse.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.sql%2Fsql-data-warehouse-transparent-encryption-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/sql-data-warehouse-transparent-encryption-create/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.sql/sql-data-warehouse-transparent-encryption-create/azuredeploy.json":::

La plantilla define un recurso:

- [Microsoft.Sql/servers](/azure/templates/microsoft.sql/servers)

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir la plantilla. Esta plantilla crea un grupo de SQL dedicado (antes SQL DW).
   
   [![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.sql%2Fsql-data-warehouse-transparent-encryption-create%2Fazuredeploy.json)

1. Escriba o actualice los siguientes valores:

   * **Suscripción**: Seleccione una suscripción de Azure.
   * **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y seleccione **Aceptar**. Un grupo de recursos nuevo facilitará la limpieza de los recursos.
   * **Región**: Seleccione una región.  Por ejemplo, **Centro de EE. UU**.
   * **Nombre del servidor de SQL Server**: acepte el predeterminado o escriba un nombre nuevo para el servidor de SQL Server.
   * **Inicio de sesión de administrador de SQL**: escriba el nombre de usuario del administrador del servidor de SQL Server.
   * **Contraseña de administrador de SQL**: escriba la contraseña del administrador del servidor de SQL Server.
   * **Nombre del almacenamiento de datos**: Escriba un nombre de grupo de SQL dedicado.
   * **Cifrado de datos transparente**: acepte el valor predeterminado, habilitado. 
   * **Objetivo de nivel de servicio**: acepte el valor predeterminado, DW400c.
   * **Ubicación**: acepte la ubicación predeterminada del grupo de recursos.
   * **Revisar y crear**: Seleccionar.
   * **Crear**: Seleccionar.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar los recursos implementados, o bien usar un script de la CLI de Azure o Azure PowerShell script para enumerar los recursos implementados.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the resource group where your dedicated SQL pool (formerly SQL DW) exists:" &&
read resourcegroupName &&
az resource list --resource-group $resourcegroupName 
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name where your dedicated SQL pool (formerly SQL DW) account exists"
(Get-AzResource -ResourceType "Microsoft.Sql/servers/databases" -ResourceGroupName $resourceGroupName).Name
 Write-Host "Press [ENTER] to continue..."
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no lo necesite, elimine el grupo de recursos mediante la CLI de Azure o Azure PowerShell:

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de SQL dedicado (antes SQL DW) mediante una plantilla de Resource Manager y ha validado la implementación. Para más información sobre Azure Synapse Analytics y Azure Resource Manager, consulte los artículos siguientes.

- Lea la [introducción a Azure Synapse Analytics](sql-data-warehouse-overview-what-is.md)
- Obtenga más información sobre [Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- [Creación e implementación de la primera plantilla de Resource Manager](../../azure-resource-manager/templates/template-tutorial-create-first-template.md)
