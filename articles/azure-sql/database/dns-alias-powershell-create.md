---
title: Alias DNS (PowerShell y CLI de Azure)
description: Los cmdlets de PowerShell y de la CLI de Azure permiten redirigir las nuevas conexiones de cliente a otro servidor SQL de Azure sin tener que tocar la configuración de cliente.
keywords: dns sql database
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli, devx-track-azurepowershell
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.devlang: PowerShell
ms.topic: how-to
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma, amagarwa, maboja, jrasnick, vanto
ms.date: 05/14/2019
ms.openlocfilehash: 58f1738b790202f699893b51247af66f80977d75
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690356"
---
# <a name="powershell-for-dns-alias-to-azure-sql-database"></a>PowerShell con alias DNS para Azure SQL Database
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

En este artículo se proporciona un script de PowerShell que muestra cómo puede administrar un alias DNS para el [servidor SQL](logical-servers.md) que hospeda su instancia de Azure SQL Database.

> [!NOTE]
> Este artículo se ha actualizado para usar el nuevo módulo Az de Azure PowerShell o la CLI de Azure. Aún puede usar el módulo de AzureRM que continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.
>
> Para obtener más información acerca del módulo Az y la compatibilidad con AzureRM, vea [Presentación del módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az). Para obtener instrucciones sobre la instalación, vea [Instalación de Azure PowerShell](/powershell/azure/install-az-ps) o [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="dns-alias-in-connection-string"></a>Alias DNS en la cadena de conexión

Para conectarse a un [servidor SQL lógico](logical-servers.md), un cliente como SQL Server Management Studio (SSMS) puede proporcionar el nombre del alias DNS en lugar del nombre verdadero del servidor. En la siguiente cadena de servidor de ejemplo, el alias *any-unique-alias-name* reemplaza al primer nodo delimitado por puntos en la cadena de servidor de cuatro nodos:

   `<yourServer>.database.windows.net`

## <a name="prerequisites"></a>Requisitos previos

Si quiere ejecutar el script de PowerShell dado en este artículo, se aplican los siguientes requisitos previos:

- Una suscripción y una cuenta de Azure, para una evaluación gratuita, vea [pruebas de Azure](https://azure.microsoft.com/free/).
- Dos servidores

## <a name="example"></a>Ejemplo

El ejemplo de código siguiente se inicia mediante la asignación de valores literales a varias variables.

Para ejecutar el código, edite los valores de marcador de posición para que coincidan con los valores reales en el sistema.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Los cmdlets que se usan son los siguientes:

- [New-AzSqlServerDNSAlias](/powershell/module/az.Sql/New-azSqlServerDnsAlias): crea un alias DNS en el sistema del servicio Azure SQL Database. El alias hace referencia al servidor 1.
- [Get-AzSqlServerDNSAlias](/powershell/module/az.Sql/Get-azSqlServerDnsAlias): obtiene y enumera todos los alias asignados al servidor 1.
- [Set-AzSqlServerDNSAlias](/powershell/module/az.Sql/Set-azSqlServerDnsAlias): modifica el nombre del servidor con el que el alias está configurado para hacer referencia, del servidor 1 al servidor 2.
- [Remove-AzSqlServerDNSAlias](/powershell/module/az.Sql/Remove-azSqlServerDnsAlias): quita el alias del servidor 2 mediante el nombre del alias.

Si necesita instalarla o actualizarla, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-az-ps).

Use `Get-Module -ListAvailable Az` en *powershell\_ise.exe* para encontrar la versión.

```powershell
$subscriptionName = '<subscriptionName>';
$sqlServerDnsAliasName = '<aliasName>';
$resourceGroupName = '<resourceGroupName>';  
$sqlServerName = '<sqlServerName>';
$resourceGroupName2 = '<resourceGroupNameTwo>'; # can be same or different than $resourceGroupName
$sqlServerName2 = '<sqlServerNameTwo>'; # must be different from $sqlServerName.

# login to Azure
Connect-AzAccount -SubscriptionName $subscriptionName;
$subscriptionId = Get-AzSubscription -SubscriptionName $subscriptionName;

Write-Host 'Assign an alias to server 1...';
New-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName -ServerName $sqlServerName `
    -Name $sqlServerDnsAliasName;

Write-Host 'Get the aliases assigned to server 1...';
Get-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName -ServerName $sqlServerName;

Write-Host 'Move the alias from server 1 to server 2...';
Set-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName2 -TargetServerName $sqlServerName2 `
    -Name $sqlServerDnsAliasName `
    -SourceServerResourceGroup $resourceGroupName -SourceServerName $sqlServerName `
    -SourceServerSubscriptionId $subscriptionId.Id;

Write-Host 'Get the aliases assigned to server 2...';
Get-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName2 -ServerName $sqlServerName2;

Write-Host 'Remove the alias from server 2...';
Remove-AzSqlServerDnsAlias –ResourceGroupName $resourceGroupName2 -ServerName $sqlServerName2 `
    -Name $sqlServerDnsAliasName;
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Los comandos utilizados son los siguientes:

- [az sql server dns-alias create](/powershell/module/az.Sql/New-azSqlServerDnsAlias): crea un alias DNS para un servidor. El alias hace referencia al servidor 1.
- [az sql server dns-alias show](/powershell/module/az.Sql/Get-azSqlServerDnsAlias): obtiene y enumera todos los alias asignados al servidor 1.
- [az sql server dns-alias set](/powershell/module/az.Sql/Set-azSqlServerDnsAlias): modifica el nombre del servidor con el que el alias está configurado para hacer referencia, del servidor 1 al servidor 2.
- [az sql server dns-alias delete](/powershell/module/az.Sql/Remove-azSqlServerDnsAlias): quita el alias del servidor 2 mediante el nombre del alias.

Para la instalación o la actualización, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

```azurecli-interactive
$subscriptionName = '<subscriptionName>';
$sqlServerDnsAliasName = '<aliasName>';
$resourceGroupName = '<resourceGroupName>';  
$sqlServerName = '<sqlServerName>';
$resourceGroupName2 = '<resourceGroupNameTwo>'; # can be same or different than $resourceGroupName
$sqlServerName2 = '<sqlServerNameTwo>'; # must be different from $sqlServerName.

# login to Azure
az login -SubscriptionName $subscriptionName;
$subscriptionId = az account list[0].i -SubscriptionName $subscriptionName;

Write-Host 'Assign an alias to server 1...';
az sql server dns-alias create –-resource-group $resourceGroupName --server $sqlServerName `
    --name $sqlServerDnsAliasName;

Write-Host 'Get the aliases assigned to server 1...';
az sql server dns-alias show –-resource-group $resourceGroupName --server $sqlServerName;

Write-Host 'Move the alias from server 1 to server 2...';
az sql server dns-alias set –-resource-group $resourceGroupName2 --server $sqlServerName2 `
    --name $sqlServerDnsAliasName `
    --original-resource-group $resourceGroupName --original-server $sqlServerName `
    --original-subscription-id $subscriptionId.Id;

Write-Host 'Get the aliases assigned to server 2...';
az sql server dns-alias show –-resource-group $resourceGroupName2 --server $sqlServerName2;

Write-Host 'Remove the alias from server 2...';
az sql server dns-alias delete –-resource-group $resourceGroupName2 --server $sqlServerName2 `
    --name $sqlServerDnsAliasName;
```

* * *

## <a name="next-steps"></a>Pasos siguientes

Para obtener una explicación completa de la característica de alias DNS para SQL Database, consulte [Alias DNS para Azure SQL Database](./dns-alias-overview.md).
