---
title: 'Inicio rápido: Creación de una instancia de Azure Dedicated HSM con Azure PowerShell'
description: Creación de una instancia de Azure Dedicated HSM con Azure PowerShell
services: dedicated-hsm
author: msmbaldwin
ms.author: mbaldwin
ms.date: 11/13/2020
ms.topic: quickstart
ms.service: key-vault
ms.devlang: azurepowershell
ms.custom: devx-track-azurepowershell - devx-track-azurepowershell - mode-api
ms.openlocfilehash: 07c3e6124bf6c2494f944bf955533df2f8528e84
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110703776"
---
# <a name="quickstart-create-an-azure-dedicated-hsm-with-azure-powershell"></a>Inicio rápido: Creación de una instancia de Azure Dedicated HSM con Azure PowerShell

En este artículo se describe cómo crear una instancia de Azure Dedicated HSM mediante el módulo de PowerShell [AZ.DedicatedHsm](/powershell/module/az.dedicatedhsm).

## <a name="requirements"></a>Requisitos

* Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [azure-powershell-requirements-no-header.md](../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Aunque el módulo **Az.DedicatedHsm** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

  ```azurepowershell-interactive
  Install-Module -Name Az.DedicatedHsm
  ```

* Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

  ```azurepowershell-interactive
  Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
  ```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos con el nombre especificado y en la ubicación especificada.

```azurepowershell-interactive
New-AzResourceGroup -Name myRG -Location westus
```

## <a name="create-a-dedicated-hsm"></a>Creación de un HSM dedicado

Para crear un HSM dedicado, use el cmdlet [New-AzDedicatedHsm](/powershell/module/az.dedicatedhsm/new-azdedicatedhsm). En el ejemplo siguiente se crea un HSM dedicado en la suscripción especificada.

```azurepowershell-interactive
$Params = @{
  Name  = 'MyHSM'
  ResourceGroupName = 'myRG'
  Location = 'westus'
  Sku = 'SafeNet Luna Network HSM A790'
  StampId = 'stamp1'
  SubnetId = '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myrg/providers/Microsoft.Network/virtualNetworks/myhsm-vnet/subnets/hsmsubnet'
  NetworkInterface = @{PrivateIPAddress = '10.2.1.120'}
}
New-AzDedicatedHsm @Params
```

```Output
Name       Provisioning State SKU                           Location
----       ------------------ ---                           --------
myhsm      Succeeded          SafeNet Luna Network HSM A790 westus
```

## <a name="get-a-dedicated-hsm"></a>Obtención de un HSM dedicado

Para recuperar información sobre un HSM dedicado existente, use el cmdlet [Get-AzDedicatedHsm](/powershell/module/az.dedicatedhsm/get-azdedicatedhsm). En el ejemplo siguiente se obtiene el HSM dedicado especificado.

```azurepowershell-interactive
Get-AzDedicatedHsm -Name MyHSM -ResourceGroupName myRG
```

```Output
Name       Provisioning State SKU                           Location
----       ------------------ ---                           --------
myhsm      Succeeded          SafeNet Luna Network HSM A790 westus
```

## <a name="update-a-dedicated-hsm"></a>Actualización de un HSM dedicado

Para actualizar un HSM dedicado, use el cmdlet [Update-AzDedicatedHsm](/powershell/module/az.dedicatedhsm/update-azdedicatedhsm). En el ejemplo siguiente se actualiza un HSM dedicado en la suscripción especificada.

```azurepowershell-interactive
Update-AzDedicatedHsm -Name MyHSM -ResourceGroupName myRG -Tag @{'key1' = '1'; 'key2' = 2; 'key3' = 3}
```

```Output
PS C:\>Update-AzDedicatedHsm -Name  hsm-n7wfxi -ResourceGroupName dedicatedhsm-rg-n359cz -Tag @{'key1' = '1';
'key2' = 2; 'key3' = 3}

Name       Provisioning State SKU                           Location
----       ------------------ ---                           --------
myhsm      Succeeded          SafeNet Luna Network HSM A790 westus
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no se necesitan los recursos que se han creado en este artículo, puede eliminarlos con el siguiente comando.

### <a name="remove-a-dedicated-hsm"></a>Eliminación de un HSM dedicado

Para quitar un HSM dedicado, use el cmdlet [Remove-AzDedicatedHsm](/powershell/module/az.dedicatedhsm/remove-azdedicatedhsm). En el ejemplo siguiente se elimina el HSM dedicado especificado.

```azurepowershell-interactive
Remove-AzDedicatedHsm -Name hsm-7t2xaf -ResourceGroupName lucas-manual-test
```

### <a name="delete-the-resource-group"></a>Eliminar el grupo de recursos

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myRG
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre [Azure Dedicated HSM](overview.md).
