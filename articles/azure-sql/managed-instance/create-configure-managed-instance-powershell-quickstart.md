---
title: 'Creación de una instancia de Azure SQL Managed Instance: inicio rápido'
description: Cree una instancia de Azure SQL Managed Instance mediante Azure PowerShell.
services: sql-managed-instance
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: contperf-fy21q1, devx-track-azurecli, devx-track-azurepowershell
ms.devlang: ''
ms.topic: quickstart
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 06/25/2021
ms.openlocfilehash: 6697cb9b89d3cb960f2def9ff4dcf478080b7b07
ms.sourcegitcommit: 6bd31ec35ac44d79debfe98a3ef32fb3522e3934
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2021
ms.locfileid: "113221057"
---
# <a name="quickstart-create-a-managed-instance-using-azure-powershell"></a>Inicio rápido: Creación de una instancia administrada mediante Azure PowerShell

En este inicio rápido, aprenderá a crear una instancia de [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md) mediante Azure PowerShell. 


## <a name="prerequisite"></a>Requisito previo

- Una suscripción de Azure activa. En caso de no tener ninguna, [cree una cuenta gratuita](https://azure.microsoft.com/free/).
- La versión más reciente de [Azure PowerShell](/powershell/azure/install-az-ps). 

## <a name="set-variables"></a>Configuración de variables

La creación de una instancia de SQL Managed Instance requiere la creación de varios recursos en Azure y, por tanto, los comandos de Azure PowerShell se basan en variables para simplificar la experiencia. Defina las variables y, a continuación, ejecute los cmdlets de cada sección dentro de la misma sesión de PowerShell. 

```azurepowershell-interactive
$NSnetworkModels = "Microsoft.Azure.Commands.Network.Models"
$NScollections = "System.Collections.Generic"
# The SubscriptionId in which to create these objects
$SubscriptionId = ''
# Set the resource group name and location for your managed instance
$resourceGroupName = "myResourceGroup-$(Get-Random)"
$location = "eastus2"
# Set the networking values for your managed instance
$vNetName = "myVnet-$(Get-Random)"
$vNetAddressPrefix = "10.0.0.0/16"
$miSubnetName = "myMISubnet-$(Get-Random)"
$miSubnetAddressPrefix = "10.0.0.0/24"
#Set the managed instance name for the new managed instance
$instanceName = "myMIName-$(Get-Random)"
# Set the admin login and password for your managed instance
$miAdminSqlLogin = "SqlAdmin"
$miAdminSqlPassword = "ChangeYourAdminPassword1"
# Set the managed instance service tier, compute level, and license mode
$edition = "General Purpose"
$vCores = 4
$maxStorage = 128
$computeGeneration = "Gen5"
$license = "LicenseIncluded" #"BasePrice" or LicenseIncluded if you have don't have SQL Server licence that can be used for AHB discount
```

## <a name="create-resource-group"></a>Creación de un grupo de recursos 

En primer lugar, conéctese a Azure, establezca el contexto de suscripción y cree el grupo de recursos. 

Para ello, ejecute este script de PowerShell: 

```azurepowershell-interactive

## Connect to Azure
Connect-AzAccount

# Set subscription context
Set-AzContext -SubscriptionId $SubscriptionId 

# Create a resource group
$resourceGroup = New-AzResourceGroup -Name $resourceGroupName -Location $location -Tag @{Owner="SQLDB-Samples"}
```

## <a name="configure-networking"></a>Configuración de las redes

Una vez creado el grupo de recursos, configure los recursos de redes, como la red virtual, las subredes, el grupo de seguridad de red y la tabla de enrutamiento. En este ejemplo se muestra el uso del script de **implementación de la subred delegada para Instancia administrada**, que está disponible en GitHub como [delegate-subnet.ps1](https://github.com/microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-managed-instance/delegate-subnet).

Para ello, ejecute este script de PowerShell: 

```azurepowershell-interactive

# Configure virtual network, subnets, network security group, and routing table
$virtualNetwork = New-AzVirtualNetwork `
                      -ResourceGroupName $resourceGroupName `
                      -Location $location `
                      -Name $vNetName `
                      -AddressPrefix $vNetAddressPrefix

                  Add-AzVirtualNetworkSubnetConfig `
                      -Name $miSubnetName `
                      -VirtualNetwork $virtualNetwork `
                      -AddressPrefix $miSubnetAddressPrefix |
                  Set-AzVirtualNetwork
                  
$scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/delegate-subnet'

$parameters = @{
    subscriptionId = $SubscriptionId
    resourceGroupName = $resourceGroupName
    virtualNetworkName = $vNetName
    subnetName = $miSubnetName
    }

Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/delegateSubnet.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters

$virtualNetwork = Get-AzVirtualNetwork -Name $vNetName -ResourceGroupName $resourceGroupName
$miSubnet = Get-AzVirtualNetworkSubnetConfig -Name $miSubnetName -VirtualNetwork $virtualNetwork
$miSubnetConfigId = $miSubnet.Id
```

## <a name="create-managed-instance"></a>Crear una instancia administrada 

Para mayor seguridad, cree una contraseña compleja y aleatoria para las credenciales de SQL Managed Instance: 

```azurepowershell-interactive
# Create credentials
$secpassword = ConvertTo-SecureString $miAdminSqlPassword -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential ($miAdminSqlLogin, $secpassword)
```

A continuación, cree la instancia de SQL Managed Instance: 

```azurepowershell-interactive
# Create managed instance
New-AzSqlInstance -Name $instanceName `
                      -ResourceGroupName $resourceGroupName -Location $location -SubnetId $miSubnetConfigId `
                      -AdministratorCredential $credential `
                      -StorageSizeInGB $maxStorage -VCore $vCores -Edition $edition `
                      -ComputeGeneration $computeGeneration -LicenseType $license
```

Esta operación puede tardar un tiempo en completarse. Para más información, consulte [Introducción a las operaciones de administración de Azure SQL Managed Instance](management-operations-overview.md).


## <a name="clean-up-resources"></a>Limpieza de recursos

Mantenga el grupo de recursos y la instancia administrada para continuar con los pasos siguientes y aprender a conectarse a la instancia de SQL Managed Instance mediante una máquina virtual cliente. 

Cuando haya terminado de usar estos recursos, puede eliminar el grupo de recursos que creó, lo que también eliminará el servidor y la base de datos única que se encuentran del grupo.

```azurepowershell-interactive
# Clean up deployment 
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```


## <a name="next-steps"></a>Pasos siguientes

Después de crear la instancia de SQL Managed Instance, implemente una máquina virtual cliente para conectarse a la instancia de SQL Managed Instance y restaurar una base de datos de ejemplo. 

> [!div class="nextstepaction"]
> [Crear una máquina virtual cliente](connect-vm-instance-configure.md)
> [Restaurar base de datos](restore-sample-database-quickstart.md)


