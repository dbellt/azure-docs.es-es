---
title: Sincronización de la configuración de servidores DNS de red virtual en el clúster virtual de SQL Managed Instance
description: Obtenga información sobre cómo sincronizar la configuración de servidores DNS de red virtual en el clúster virtual de SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
author: srdan-bozovic-msft
ms.author: srbozovi
ms.topic: how-to
ms.date: 01/17/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 276b4033dc2a5c157245448453b72a8074b79702
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110707153"
---
# <a name="synchronize-virtual-network-dns-servers-setting-on-sql-managed-instance-virtual-cluster"></a>Sincronización de la configuración de servidores DNS de red virtual en el clúster virtual de SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica cómo sincronizar el establecimiento de servidores DNS de red virtual en el clúster virtual de SQL Managed Instance.

## <a name="when-to-synchronize-the-dns-setting"></a>Cuándo sincronizar la configuración de DNS

Hay algunos escenarios (por ejemplo, correo electrónico de base de datos o servidores vinculados a otras instancias de SQL Server en la nube o el entorno híbrido) que requieren que los nombres de host privados se resuelvan en SQL Managed Instance. En este caso, debe configurar un DNS personalizado dentro de Azure. Consulte [Configuración de un DNS personalizado para Azure SQL Managed Instance](custom-dns-configure.md).

Si este cambio se implementa después de que se cree el [clúster virtual](connectivity-architecture-overview.md#virtual-cluster-connectivity-architecture) que hospeda a la instancia administrada, tendrá que sincronizar la configuración de los servidores DNS en el clúster virtual con la configuración de la red virtual.

> [!IMPORTANT]
> La sincronización de la configuración de los servidores DNS afectará a todas las instancias administradas hospedadas en el clúster virtual.

## <a name="how-to-synchronize-the-dns-setting"></a>Sincronización de la configuración de DNS

### <a name="azure-rbac-permissions-required"></a>Se requieren permisos de RBAC de Azure

La configuración del servidor DNS de sincronización de usuarios debe tener uno de los siguientes roles de Azure:

- Rol de colaborador de la suscripción o
- Rol personalizado con el permiso siguiente:
  - `Microsoft.Sql/virtualClusters/updateManagedInstanceDnsServers/action`

### <a name="use-azure-powershell"></a>Uso de Azure PowerShell

Obtenga la red virtual en la que se ha actualizado la configuración de los servidores DNS.

```PowerShell
$ResourceGroup = 'enter resource group of virtual network'
$VirtualNetworkName = 'enter virtual network name'
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroup $ResourceGroup -Name $VirtualNetworkName
```
Use el comando de PowerShell [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction) para sincronizar la configuración de los servidores DNS de todos los clústeres virtuales de la subred.

```PowerShell
Get-AzSqlVirtualCluster `
    | where SubnetId -match $virtualNetwork.Id `
    | select Id `
    | Invoke-AzResourceAction -Action updateManagedInstanceDnsServers -Force
```
### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

Obtenga la red virtual en la que se ha actualizado la configuración de los servidores DNS.

```Azure CLI
resourceGroup="auto-failover-group"
virtualNetworkName="vnet-fog-eastus"
virtualNetwork=$(az network vnet show -g $resourceGroup -n $virtualNetworkName --query "id" -otsv)
```

Use el comando de la CLI de Azure [az resource invoke-action](/cli/azure/resource#az_resource_invoke_action) para sincronizar la configuración de los servidores DNS de todos los clústeres virtuales de la subred.

```Azure CLI
az sql virtual-cluster list --query "[? contains(subnetId,'$virtualNetwork')].id" -o tsv \
    | az resource invoke-action --action updateManagedInstanceDnsServers --ids @-
```
## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la configuración de un DNS personalizado en [Configuración de un DNS personalizado para Azure SQL Managed Instance](custom-dns-configure.md).
- Para obtener información general, consulte [¿Qué es Azure SQL Managed Instance?](sql-managed-instance-paas-overview.md).
