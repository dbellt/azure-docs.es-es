---
title: Implementación de Azure Spring Cloud con Azure PowerShell
description: Implementación de Azure Spring Cloud con Azure PowerShell
author: bmitchell287
ms.author: brendm
ms.topic: conceptual
ms.service: spring-cloud
ms.devlang: azurepowershell
ms.date: 11/16/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 40c5d720aa3ddf6d38b830c84e326e9cbbf06ed0
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108135192"
---
# <a name="deploy-azure-spring-cloud-with-azure-powershell"></a>Implementación de Azure Spring Cloud con Azure PowerShell

En este artículo se describe cómo crear una instancia de Azure Spring Cloud mediante el módulo de PowerShell [Az.SpringCloud](/powershell/module/Az.SpringCloud).

## <a name="requirements"></a>Requisitos

* Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [azure-powershell-requirements-no-header.md](../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Aunque el módulo **Az.SpringCloud** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

  ```azurepowershell-interactive
  Install-Module -Name Az.SpringCloud
  ```

* Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

  ```azurepowershell-interactive
  Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
  ```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos con el nombre especificado y en la ubicación especificada.

```azurepowershell-interactive
New-AzResourceGroup -Name <resource group name> -Location eastus
```

## <a name="provision-a-new-instance-of-azure-spring-cloud"></a>Aprovisionamiento de una instancia nueva de Azure Spring Cloud

Para crear una nueva instancia de Azure Spring Cloud, use el cmdlet [New-AzSpringCloud](/powershell/module/az.springcloud/new-azspringcloud). En el ejemplo siguiente se crea un servicio de Azure Spring Cloud con el nombre especificado en el grupo de recursos creado anteriormente.

```azurepowershell-interactive
New-AzSpringCloud -ResourceGroupName <resource group name> -name <service instance name> -Location eastus
```

## <a name="create-a-new-azure-spring-cloud-app"></a>Creación de una aplicación de Azure Spring Cloud

Para crear una nueva aplicación, use el cmdlet [New-AzSpringCloudApp](/powershell/module/az.springcloud/new-azspringcloudapp). En el ejemplo siguiente se crea una aplicación de Azure Spring Cloud llamada `gateway`.

```azurepowershell-interactive
New-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

## <a name="create-a-new-azure-spring-cloud-deployment"></a>Creación de una implementación de Azure Spring Cloud

Para crear una nueva implementación, use el cmdlet [New-AzSpringCloudAppDeployment](/powershell/module/az.springcloud/new-azspringcloudappdeployment). En el ejemplo siguiente se crea una implementación de Azure Spring Cloud llamada `default` para la aplicación `gateway`.

```azurepowershell-interactive
New-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -Name <service instance name> -AppName gateway -DeploymentName default
```

## <a name="get-an-azure-spring-cloud-service"></a>Obtención de un servicio de Azure Spring Cloud

Para obtener un servicio de Azure Spring Cloud y sus propiedades, use el cmdlet [Get-AzSpringCloud](/powershell/module/az.springcloud/get-azspringcloud). En el siguiente ejemplo se recupera información sobre el servicio de Azure Spring Cloud especificado.

```azurepowershell-interactive
Get-AzSpringCloud -ResourceGroupName <resource group name> -ServiceName <service instance name>
```

## <a name="get-an-azure-spring-cloud-app"></a>Obtención de una aplicación de Azure Spring Cloud

Para obtener una aplicación de Azure Spring Cloud y sus propiedades, use el cmdlet [Get-AzSpringCloudApp](/powershell/module/az.springcloud/get-azspringcloudapp). En el siguiente ejemplo se recupera información acerca de la aplicación de Spring Cloud `gateway`.

```azurepowershell-interactive
Get-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

## <a name="get-an-azure-spring-cloud-deployment"></a>Obtención de una implementación de Azure Spring Cloud

Para obtener una implementación de Azure Spring Cloud y sus propiedades, use el cmdlet [Get-AzSpringCloudAppDeployment](/powershell/module/az.springcloud/get-azspringcloudappdeployment). En el siguiente ejemplo se recupera información acerca de la implementación de Spring Cloud `default`.

```azurepowershell-interactive
Get-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway -DeploymentName default
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no se necesitan los recursos que se han creado en este artículo, puede eliminarlos con el siguiente comando.

### <a name="delete-an-azure-spring-cloud-deployment"></a>Eliminación de una implementación de Azure Spring Cloud

Para quitar una implementación de Azure Spring Cloud, use el cmdlet [Remove-AzSpringCloudAppDeployment](/powershell/module/az.springcloud/remove-azspringcloudappdeployment). En el ejemplo siguiente se elimina una implementación de aplicación de Azure Spring Cloud denominada `default` para el servicio y la aplicación especificados.

```azurepowershell-interactive
Remove-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway -DeploymentName default
```

### <a name="delete-an-azure-spring-cloud-app"></a>Eliminación de una aplicación de Azure Spring Cloud

Para quitar una aplicación de Spring Cloud, use el cmdlet [Remove-AzSpringCloudApp](/powershell/module/Az.SpringCloud/remove-azspringcloudapp). En el siguiente ejemplo se elimina la aplicación `gateway` en el servicio y el grupo de recursos especificados.

```azurepowershell
Remove-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

### <a name="delete-an-azure-spring-cloud-service"></a>Eliminación de un servicio de Azure Spring Cloud

Para quitar un servicio de Azure Spring Cloud, use el cmdlet [Remove-AzSpringCloud](/powershell/module/Az.SpringCloud/remove-azspringcloud). En el ejemplo siguiente se elimina el servicio de Azure Spring Cloud especificado.

```azurepowershell
Remove-AzSpringCloud -ResourceGroupName <resource group name> -ServiceName <service instance name>
```

### <a name="delete-the-resource-group"></a>Eliminar el grupo de recursos

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurepowershell-interactive
Remove-AzResourceGroup -Name <resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

[Recursos para desarrolladores de Azure Spring Cloud](./resources.md).