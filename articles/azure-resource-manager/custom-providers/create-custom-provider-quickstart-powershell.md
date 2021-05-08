---
title: Creación de un proveedor de recursos personalizado de Azure con Azure PowerShell
description: Describe cómo crear un proveedor de recursos personalizado de Azure con Azure PowerShell
author: MSEvanhi
ms.author: evanhi
ms.date: 09/22/2020
ms.topic: quickstart
ms.devlang: azurepowershell
ms.custom: devx-track-azurepowershell - devx-track-azurepowershell - mode-api
ms.openlocfilehash: 41510a547c7a3a19af5e3691647352efcbf40950
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318214"
---
# <a name="quickstart-create-an-azure-custom-resource-provider-with-azure-powershell"></a>Inicio rápido: Creación de un proveedor de recursos personalizado de Azure con Azure PowerShell

En esta guía de inicio rápido, aprenderá a crear su propio proveedor de recursos personalizado de Azure mediante el módulo [Az.CustomProviders](/powershell/module/az.customproviders) de PowerShell.

> [!CAUTION]
> La característica Proveedores personalizados de Azure está actualmente en versión preliminar pública. Esta versión preliminar se proporciona sin un acuerdo de nivel de servicio. No se recomienda para las cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Requisitos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps). Si decide usar Cloud Shell, consulte [Introducción a Azure Cloud Shell](../../cloud-shell/overview.md) para más información.

> [!IMPORTANT]
> Aunque el módulo **Az.CustomProviders** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.CustomProviders
```

Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](../../azure-resource-manager/management/overview.md) con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos con el nombre especificado y en la ubicación especificada.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location westus2
```

## <a name="create-a-custom-resource-provider"></a>Creación de un proveedor de recursos personalizado

Para crear o actualizar un proveedor de recursos personalizado, use el cmdlet [New-AzCustomProvider](/powershell/module/az.customproviders/new-azcustomprovider) como se muestra en el ejemplo siguiente.

```azurepowershell-interactive
New-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type -Location westus2 -ResourceType @{Name='CustomRoute1'; Endpoint='https://www.contoso.com/'}
```

## <a name="get-the-custom-resource-provider-manifest"></a>Obtención del manifiesto del proveedor de recursos personalizado

Para recuperar información sobre el manifiesto del proveedor de recursos personalizado, use el cmdlet [Get-AzCustomProvider](/powershell/module/az.customproviders/get-azcustomprovider) como se muestra en el ejemplo siguiente.

```azurepowershell-interactive
Get-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type | Format-List
```

## <a name="create-an-association"></a>Creación de una asociación

Para crear o actualizar una asociación, use el cmdlet [New-AzCustomProviderAssociation](/powershell/module/az.customproviders/new-azcustomproviderassociation) como se muestra en el ejemplo siguiente.

```azurepowershell-interactive
$provider = Get-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type
New-AzCustomProviderAssociation -Scope $resourceId -Name MyAssoc -TargetResourceId $provider.Id
```

## <a name="get-an-association"></a>Obtención de una asociación

Para recuperar información sobre una asociación, use el cmdlet [Get-AzCustomProviderAssociation](/powershell/module/az.customproviders/get-azcustomproviderassociation) como se muestra en el ejemplo siguiente.

```azurepowershell-interactive
Get-AzCustomProviderAssociation -Scope $resourceId -Name MyAssoc
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no se necesitan los recursos que se han creado en este artículo, puede eliminarlos con el siguiente comando.

### <a name="delete-an-association"></a>Eliminación de una asociación

Para quitar una asociación, use el cmdlet [Remove-AzCustomProviderAssociation](/powershell/module/az.customproviders/remove-azcustomproviderassociation). En el ejemplo siguiente se elimina una asociación.

```azurepowershell
Remove-AzCustomProviderAssociation -Scope $id -Name Namespace.Type
```

### <a name="delete-a-custom-resource-provider"></a>Eliminación de un proveedor de recursos personalizado

Para quitar un proveedor de recursos personalizado, use el cmdlet [Remove-AzCustomProvider](/powershell/module/az.customproviders/remove-azcustomprovider). En el ejemplo siguiente se elimina un proveedor de recursos personalizado.

```azurepowershell-interactive
Remove-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type
```

### <a name="delete-the-resource-group"></a>Eliminar el grupo de recursos

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los [proveedores de recursos personalizados de Azure](overview.md).
