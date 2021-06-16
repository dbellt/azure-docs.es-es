---
title: Configuración de conectividad para Azure SQL Database y Azure Synapse Analytics
description: En este artículo se explica por qué debe elegir la versión de seguridad de la capa de transporte (TLS) y el proxy frente a la configuración de redireccionamiento para Azure SQL Database y Azure Synapse Analytics.
services: sql-database
ms.service: sql-database
ms.subservice: connect
titleSuffix: Azure SQL Database and Azure Synapse Analytics
ms.topic: how-to
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: mathoma, vanto
ms.date: 07/06/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4de2ad883e0daa4a0be0e1da8060b783af5713ff
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699868"
---
# <a name="azure-sql-connectivity-settings"></a>Configuración de la conectividad de Azure SQL
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Este artículo presenta los valores que controlan la conectividad con el servidor para Azure SQL Database y [grupo de SQL dedicado (anteriormente, SQL DW)](../../synapse-analytics\sql-data-warehouse\sql-data-warehouse-overview-what-is.md) en Azure Synapse Analytics. Estos valores se aplican a todas las bases de datos de SQL Database y de grupo de SQL dedicado (anteriormente, SQL DW) asociadas al servidor.

> [!IMPORTANT]
> Este artículo no se aplica a Azure SQL Managed Instance.

Se puede acceder a la configuración de conectividad desde la pantalla **Firewalls y redes virtuales**, como se indica en la siguiente captura de pantalla:

 ![Captura de pantalla de la ventana de configuración de conectividad.][1]

> [!NOTE]
> Estos valores surten efecto inmediatamente después de aplicarse. Los clientes pueden experimentar una pérdida de conexión si no cumplen los requisitos de cada opción de configuración.

## <a name="deny-public-network-access"></a>Denegación del acceso a una red pública

Cuando la opción **Denegar acceso desde red pública** está establecida en **Sí**, solo se permiten las conexiones mediante puntos de conexión privados. Cuando esta opción está establecida en **No** (valor predeterminado), los clientes pueden conectarse mediante puntos de conexión públicos (con reglas de firewall basadas en IP o reglas de firewall basadas en redes virtuales) o puntos de conexión privados (mediante Azure Private Link), tal como se describe en [Introducción al acceso de red](network-access-controls-overview.md).

 ![Diagrama que muestra la conectividad cuando la opción Denegar acceso desde red pública está establecida en Sí frente a cuando está establecida en No.][2]

Cualquier intento de establecer la opción **Denegar acceso desde red pública** en **Sí** sin puntos de conexión privados existentes en el servidor lógico producirá un error con un mensaje similar al siguiente:  

```output
Error 42102
Unable to set Deny Public Network Access to Yes since there is no private endpoint enabled to access the server.
Please set up private endpoints and retry the operation.
```

> [!NOTE]
> Para definir las reglas de firewall de red virtual en un servidor lógico que ya está configurado con puntos de conexión privados, establezca **Denegar acceso a la red pública** en **No**.

Cuando la opción **Denegar acceso desde red pública** está establecida en **Sí**, solo se permiten las conexiones mediante puntos de conexión privados. Se denegarán todas las conexiones a través de puntos de conexión públicos con un mensaje de error similar al siguiente:  

```output
Error 47073
An instance-specific error occurred while establishing a connection to SQL Server. 
The public network interface on this server is not accessible. 
To connect to this server, use the Private Endpoint from inside your virtual network.
```

Cuando la opción **Denegar acceso desde red pública** está establecida en **Sí**, se denegará cualquier intento de agregar o actualizar las reglas de firewall con un mensaje de error similar al siguiente:

```output
Error 42101
Unable to create or modify firewall rules when public network interface for the server is disabled. 
To manage server or database level firewall rules, please enable the public network interface.
```

## <a name="change-public-network-access-via-powershell"></a>Cambio del acceso a la red pública mediante PowerShell

> [!IMPORTANT]
> Azure SQL Database todavía es compatible con el módulo de Azure Resource Manager para PowerShell, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. El script siguiente requiere el [módulo de Azure PowerShell](/powershell/azure/install-az-ps).

El siguiente script de PowerShell le muestra cómo aplicar las operaciones `Get` y `Set` a la propiedad **Public Network Access** (Acceso a la red pública) en el nivel del servidor:

```powershell
# Get the Public Network Access property
(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).PublicNetworkAccess

# Update Public Network Access to Disabled
$SecureString = ConvertTo-SecureString "password" -AsPlainText -Force

Set-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group -SqlAdministratorPassword $SecureString -PublicNetworkAccess "Disabled"
```

## <a name="change-public-network-access-via-cli"></a>Cambio del acceso a la red pública mediante la CLI

> [!IMPORTANT]
> Todos los scripts de esta sección requieren la [CLI de Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>CLI de Azure en un shell de Bash

El siguiente script de la CLI muestra cómo cambiar la opción **Acceso a la red pública** en un shell de Bash:

```azurecli-interactive

# Get current setting for Public Network Access
az sql server show -n sql-server-name -g sql-server-group --query "publicNetworkAccess"

# Update setting for Public Network Access
az sql server update -n sql-server-name -g sql-server-group --set publicNetworkAccess="Disabled"
```

## <a name="minimal-tls-version"></a>Versión mínima de TLS 

La opción de versión mínima de [Seguridad de la capa de transporte (TLS)](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server) permite a los clientes elegir la versión de TLS que usa su base de datos SQL.

Actualmente, se admite TLS 1.0, 1.1 y 1.2. Establecer una versión mínima de TLS garantiza que se admitan las versiones más recientes de TLS. Por ejemplo, la selección de una versión 1.1 de TLS significa que solo se aceptan las conexiones con TLS 1.1 y 1.2, y que se rechazan las conexiones con TLS 1.0. Después de realizar pruebas para confirmar que las aplicaciones la admiten, se recomienda establecer la versión mínima de TLS en 1.2. Esta versión incluye correcciones para vulnerabilidades de las versiones anteriores y es la versión más reciente de TLS que se admite en Azure SQL Database.

> [!IMPORTANT]
> El valor predeterminado para la versión mínima de TLS es Permitir todas las versiones. Después de aplicar una versión de TLS, no es posible revertir al valor predeterminado.

En el caso de los clientes con aplicaciones que se basan en versiones anteriores de TLS, se recomienda establecer la versión mínima de TLS en función de los requisitos de las aplicaciones. En el caso de los clientes necesitan aplicaciones para conectarse mediante una conexión sin cifrar, se recomienda no establecer ninguna versión mínima de TLS.

Para obtener más información, consulte [Consideraciones de TLS para la conectividad de SQL Database](connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

Después de establecer la versión mínima de TLS, se producirá un error en los intentos de inicio de sesión de los clientes que usan una versión de TLS inferior a la versión mínima de TLS del servidor, con el siguiente error:

```output
Error 47072
Login failed with invalid TLS version
```

## <a name="set-the-minimal-tls-version-via-powershell"></a>Establecimiento de la versión mínima de TLS mediante PowerShell

> [!IMPORTANT]
> Azure SQL Database todavía es compatible con el módulo de Azure Resource Manager para PowerShell, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. El script siguiente requiere el [módulo de Azure PowerShell](/powershell/azure/install-az-ps).

El siguiente script de PowerShell le muestra cómo `Get` y `Set` la propiedad **Minimal TLS Version** (Versión mínima de TLS) en el nivel del servidor lógico:

```powershell
# Get the Minimal TLS Version property
(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).MinimalTlsVersion

# Update Minimal TLS Version to 1.2
$SecureString = ConvertTo-SecureString "password" -AsPlainText -Force

Set-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group -SqlAdministratorPassword $SecureString  -MinimalTlsVersion "1.2"
```

## <a name="set-the-minimal-tls-version-via-the-azure-cli"></a>Establecimiento de la versión mínima de TLS mediante la CLI de Azure

> [!IMPORTANT]
> Todos los scripts de esta sección requieren la [CLI de Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>CLI de Azure en un shell de Bash

El siguiente script de la CLI muestra cómo cambiar la opción **Minimal TLS Version** (versión mínima de TLS) en un shell de Bash:

```azurecli-interactive
# Get current setting for Minimal TLS Version
az sql server show -n sql-server-name -g sql-server-group --query "minimalTlsVersion"

# Update setting for Minimal TLS Version
az sql server update -n sql-server-name -g sql-server-group --set minimalTlsVersion="1.2"
```

## <a name="change-the-connection-policy"></a>Cambio de la directiva de conexión

La [directiva de conexión](connectivity-architecture.md#connection-policy) determina cómo se conectan los clientes a Azure SQL Database.

## <a name="change-the-connection-policy-via-powershell"></a>Cambio de la directiva de conexión mediante PowerShell

> [!IMPORTANT]
> Azure SQL Database todavía es compatible con el módulo de Azure Resource Manager para PowerShell, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. El script siguiente requiere el [módulo de Azure PowerShell](/powershell/azure/install-az-ps).

El siguiente script muestra cómo cambiar la directiva de conexión mediante PowerShell:

```powershell
# Get SQL Server ID
$sqlserverid=(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).ResourceId

# Set URI
$id="$sqlserverid/connectionPolicies/Default"

# Get current connection policy
(Get-AzResource -ResourceId $id -ApiVersion 2014-04-01 -Verbose).Properties.ConnectionType

# Update connection policy
Set-AzResource -ResourceId $id -Properties @{"connectionType" = "Proxy"} -f
```

## <a name="change-the-connection-policy-via-the-azure-cli"></a>Cambio de la directiva de conexión mediante la CLI de Azure

> [!IMPORTANT]
> Todos los scripts de esta sección requieren la [CLI de Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>CLI de Azure en un shell de Bash

El siguiente script de CLI muestra cómo cambiar la directiva de conexión en un shell de Bash:

```azurecli-interactive
# Get SQL Server ID
sqlserverid=$(az sql server show -n sql-server-name -g sql-server-group --query 'id' -o tsv)

# Set URI
ids="$sqlserverid/connectionPolicies/Default"

# Get current connection policy
az resource show --ids $ids

# Update connection policy
az resource update --ids $ids --set properties.connectionType=Proxy
```

### <a name="azure-cli-from-a-windows-command-prompt"></a>CLI de Azure desde un símbolo del sistema de Windows

En el siguiente script de la CLI se muestra cómo cambiar la directiva de conexión desde un símbolo del sistema de Windows (con la CLI de Azure instalada):

```azurecli
# Get SQL Server ID and set URI
FOR /F "tokens=*" %g IN ('az sql server show --resource-group myResourceGroup-571418053 --name server-538465606 --query "id" -o tsv') do (SET sqlserverid=%g/connectionPolicies/Default)

# Get current connection policy
az resource show --ids %sqlserverid%

# Update connection policy
az resource update --ids %sqlserverid% --set properties.connectionType=Proxy
```

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo funciona la conectividad en Azure SQL Database, consulte [Arquitectura de conectividad](connectivity-architecture.md).
- Para información sobre cómo cambiar la directiva de conexión para un servidor, consulte [conn-policy](/cli/azure/sql/server/conn-policy).

<!--Image references-->
[1]: media/single-database-create-quickstart/manage-connectivity-settings.png
[2]: media/single-database-create-quickstart/manage-connectivity-flowchart.png
