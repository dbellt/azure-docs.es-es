---
title: Creación y administración de servidores y bases de datos únicas
description: Obtenga información sobre la creación y administración de servidores y bases de datos únicas de Azure SQL Database mediante Azure Portal, PowerShell, la CLI de Azure, Transact-SQL (T-SQL) y la API REST.
services: sql-database
ms.service: sql-database
ms.subservice: deployment-configuration
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: f09bd53a8e98b88bcc42ac085240b45d02671bba
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693099"
---
# <a name="create-and-manage-servers-and-single-databases-in-azure-sql-database"></a>Creación y administración de servidores y bases de datos únicas en Azure SQL Database

Puede crear y administrar servidores y bases de datos únicas en Azure SQL Database con Azure Portal, PowerShell, la CLI de Azure, la API REST y Transact-SQL.

## <a name="the-azure-portal"></a>El Portal de Azure

Puede crear el grupo de recursos para Azure SQL Database con antelación o mientras crea el propio servidor.

### <a name="create-a-server"></a>Creación de un servidor

Para crear un servidor mediante [Azure Portal](https://portal.azure.com), cree un nuevo recurso de [servidor](logical-servers.md) desde Azure Marketplace. Como alternativa, puede crear el servidor al implementar una instancia de Azure SQL Database.

  ![creación de un servidor](./media/single-database-manage/create-logical-sql-server.png)

### <a name="create-a-blank-or-sample-database"></a>Creación de una base de datos de ejemplo o en blanco

Para crear una única instancia de Azure SQL Database mediante [Azure Portal](https://portal.azure.com), elija el recurso de Azure SQL Database en Azure Marketplace. Puede crear el grupo de recursos y el servidor con antelación o mientras crea la propia base de datos única. Puede crear una base de datos en blanco o de ejemplo basada en Adventure Works LT.

  ![create database-1](./media/single-database-manage/create-database-1.png)

> [!IMPORTANT]
> Para obtener información sobre cómo seleccionar el plan de tarifa de la base de datos, consulte el [modelo de compra basado en DTU](service-tiers-dtu.md) y el [modelo de compra basado en núcleo virtual](service-tiers-vcore.md).

## <a name="manage-an-existing-server"></a>Administración de un servidor existente

Para administrar un servidor existente, vaya al servidor mediante una serie de métodos, como la página específica de la base de datos, la página **Servidores SQL Server** o la página **Todos los recursos**.

Para administrar una base de datos existente, vaya a la página de **SQL Database** y seleccione la base de datos que desea administrar. En la captura de pantalla siguiente se muestra cómo empezar a configurar un firewall de nivel de servidor para una base de datos desde la página de **información general** de una base de datos.

   ![regla de firewall del servidor](./media/single-database-manage/server-firewall-rule.png)

> [!IMPORTANT]
> Para configurar las propiedades de rendimiento de una base de datos, consulte el [modelo de compra basado en DTU](service-tiers-dtu.md) y el [modelo de compra basado en núcleo virtual](service-tiers-vcore.md).
> [!TIP]
> Para ver un artículo de inicio rápido de Azure Portal, consulte [Creación de una base de datos de SQL Database en Azure Portal](single-database-create-quickstart.md).

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Para crear y administrar servidores, bases de datos únicas y agrupadas y firewalls de servidor con Azure PowerShell, use los siguientes cmdlets de PowerShell. Si necesita instalar o actualizar PowerShell, consulte [Install and configure Azure PowerShell](/powershell/azure/install-az-ps) (Instalación y configuración de Azure PowerShell).

> [!TIP]
> Para ver scripts de PowerShell de ejemplo, consulte [Uso de PowerShell para crear una base de datos SQL Database y configurar una regla de firewall de servidor](scripts/create-and-configure-database-powershell.md) y [Uso de PowerShell para supervisar y escalar una base de datos en SQL Database](scripts/monitor-and-scale-database-powershell.md).

| Cmdlet | Descripción |
| --- | --- |
|[New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)|Crea una base de datos. |
|[Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase)|Obtiene una o más bases de datos.|
|[Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)|Establece las propiedades de una base de datos, o mueve una base de datos existente en un grupo elástico.|
|[Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase)|Quita una base de datos.|
|[New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)|Crea un grupo de recursos.|
|[New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver)|Crea un servidor.|
|[Get-AzSqlServer](/powershell/module/az.sql/get-azsqlserver)|Devuelve información sobre los servidores.|
|[Set-AzSqlServer](/powershell/module/az.sql/set-azsqlserver)|Modifica las propiedades de un servidor.|
|[Remove-AzSqlServer](/powershell/module/az.sql/remove-azsqlserver)|Quita un servidor.|
|[New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule)|Crea una regla de firewall de nivel de servidor. |
|[Get-AzSqlServerFirewallRule](/powershell/module/az.sql/get-azsqlserverfirewallrule)|Obtiene reglas de firewall para un servidor|
|[Set-AzSqlServerFirewallRule](/powershell/module/az.sql/set-azsqlserverfirewallrule)|Modifica una regla de firewall en un servidor.|
|[Remove-AzSqlServerFirewallRule](/powershell/module/az.sql/remove-azsqlserverfirewallrule)|Obtiene una regla de firewall de un servidor.|
| New-AzSqlServerVirtualNetworkRule | Permite crear una [*regla de red virtual*](vnet-service-endpoint-rule-overview.md) basada en una subred que es un punto de conexión de servicio de red virtual. |

## <a name="the-azure-cli"></a>La CLI de Azure

Para crear y administrar los servidores, bases de datos y firewalls con la [CLI de Azure](/cli/azure), use los siguientes comandos de la [CLI de Azure](/cli/azure/sql/db). Use [Cloud Shell](../../cloud-shell/overview.md) para ejecutar la CLI en el explorador o [instálela](/cli/azure/install-azure-cli) en Windows, Linux o macOS. Para crear y administrar grupos elásticos, consulte [Grupos elásticos](elastic-pool-overview.md).

> [!TIP]
> Para ver un artículo de inicio rápido de la CLI de Azure, consulte [Creación de una base de datos única de Azure SQL Database con la CLI de Azure](az-cli-script-samples-content-guide.md). Para ver los scripts de ejemplo de la CLI de Azure, consulte [Uso de la CLI para crear una base de datos en Azure SQL Database y configurar una regla de firewall de SQL Database](scripts/create-and-configure-database-cli.md) y [Uso de la CLI para supervisar y escalar una base de datos de Azure SQL Database](scripts/monitor-and-scale-database-cli.md).
>

| Cmdlet | Descripción |
| --- | --- |
|[az sql db create](/cli/azure/sql/db#az_sql_db_create) |Crea una base de datos.|
|[az sql db list](/cli/azure/sql/db#az_sql_db_list)|Enumera todas las bases de datos y almacenes de datos de un servidor, o todas las bases de datos de un grupo elástico.|
|[az sql db list-editions](/cli/azure/sql/db#az_sql_db_list_editions)|Enumera los objetivos de servicio y los límites de almacenamiento disponibles.|
|[az sql db list-usages](/cli/azure/sql/db#az_sql_db_list_usages)|Devuelve los usos de la base de datos.|
|[az sql db show](/cli/azure/sql/db#az_sql_db_show)|Obtiene una base de datos o un almacenamiento de datos.|
|[az sql db update](/cli/azure/sql/db#az_sql_db_update)|Actualiza una base de datos.|
|[az sql db delete](/cli/azure/sql/db#az_sql_db_delete)|Quita una base de datos.|
|[az group create](/cli/azure/group#az_group_create)|Crea un grupo de recursos.|
|[az sql server create](/cli/azure/sql/server#az_sql_server_create)|Crea un servidor.|
|[az sql server list](/cli/azure/sql/server#az_sql_server_list)|Enumera los servidores.|
|[az sql server list-usages](/cli/azure/sql/server#az_sql_server_list-usages)|Devuelve los usos del servidor.|
|[az sql server show](/cli/azure/sql/server#az_sql_server_show)|Obtiene un servidor.|
|[az sql server update](/cli/azure/sql/server#az_sql_server_update)|Actualiza un servidor.|
|[az sql server delete](/cli/azure/sql/server#az_sql_server_delete)|Permite eliminar un servidor.|
|[az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_create)|Crea una regla de firewall del servidor.|
|[az sql server firewall-rule list](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_list)|Enumera las reglas de firewall en un servidor.|
|[az sql server firewall-rule show](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_show)|Muestra los detalles de una regla de firewall.|
|[az sql server firewall-rule update](/cli/azure/sql/server/firewall-rule##az_sql_server_firewall_rule_update)|Actualiza una regla de firewall.|
|[az sql server firewall-rule delete](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_delete)|Elimina una regla de firewall.|

## <a name="transact-sql-t-sql"></a>Transact-SQL (T-SQL)

Para crear y administrar los servidores, bases de datos y firewalls con Transact-SQL, use los siguientes comandos de Transact-SQL. Puede emitir estos comandos mediante Azure Portal, [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), [Visual Studio Code](https://code.visualstudio.com/docs) o cualquier otro programa que pueda conectarse a un servidor en SQL Database y pasar comandos de Transact-SQL. Para administrar grupos de bases de datos elásticas, consulte [Grupos elásticos](elastic-pool-overview.md).

> [!TIP]
> Para acceder a un artículo de inicio rápido sobre el uso de SQL Server Management Studio en Microsoft Windows, consulte [Azure SQL Database: uso de SQL Server Management Studio para conectarse a los datos y realizar consultas en ellos](connect-query-ssms.md). Para acceder a un inicio rápido sobre el uso de Visual Studio Code en macOS, Linux o Windows, consulte [Azure SQL Database: uso de Visual Studio Code para conectarse a los datos y realizar consultas en ellos](connect-query-vscode.md).
> [!IMPORTANT]
> No se puede crear ni eliminar un servidor mediante Transact-SQL.

| Get-Help | Descripción |
| --- | --- |
|[CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current&preserve-view=true)|Crea una base de datos. Debe estar conectado a la base de datos maestra para crear una base de datos.|
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true) |Modifica una base de datos o un grupo elástico. |
|[DROP DATABASE](/sql/t-sql/statements/drop-database-transact-sql)|Permite eliminar una base de datos.|
|[sys.database_service_objectives](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Devuelve la edición (nivel de servicio), el objetivo de servicio (plan de tarifa) y el nombre del grupo elástico, si existe, de una instancia de Azure SQL Database o de un grupo de SQL dedicado en Azure Synapse Analytics. Si inició sesión en la base de datos maestra en SQL Database, devuelve información sobre todas las bases de datos. Para Azure Synapse Analytics, debe estar conectado a la base de datos maestra.|
|[sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Devuelve el consumo de CPU, E/S y memoria para una base de datos en Azure SQL Database. Hay una fila para cada 15 segundos, incluso si no hay ninguna actividad en la base de datos.|
|[sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)|Devuelve datos de almacenamiento y uso de CPU para una base de datos de Azure SQL Database. Los datos se recopilan y se agregan en intervalos de cinco minutos.|
|[sys.database_connection_stats](/sql/relational-databases/system-catalog-views/sys-database-connection-stats-azure-sql-database)|Contiene estadísticas de eventos de conectividad de SQL Database, que proporcionan una visión general de los aciertos y errores de conexión a la base de datos. |
|[sys.event_log](/sql/relational-databases/system-catalog-views/sys-event-log-azure-sql-database)|Devuelve las conexiones realizadas correctamente a Azure SQL Database, los errores de conexión y los interbloqueos. Puede usar esta información para realizar el seguimiento de la actividad de base de datos o solucionar problemas con SQL Database.|
|[sp_set_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database)|Crea o actualiza la configuración del firewall de nivel de servidor para su servidor. Este procedimiento almacenado solo está disponible en la base de datos maestra para el inicio de sesión principal de nivel de servidor. Solo se puede crear una regla de firewall de nivel de servidor mediante Transact-SQL después de que un usuario con permisos a nivel de Azure haya creado la primera regla de firewall de nivel de servidor.|
|[sys.firewall_rules](/sql/relational-databases/system-catalog-views/sys-firewall-rules-azure-sql-database)|Devuelve información sobre la configuración del firewall de nivel de servidor asociada a la base de datos de Azure SQL Database.|
|[sp_delete_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-firewall-rule-azure-sql-database)|Quita la configuración del firewall de nivel de servidor de su servidor. Este procedimiento almacenado solo está disponible en la base de datos maestra para el inicio de sesión principal de nivel de servidor.|
|[sp_set_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database)|Crea o actualiza las reglas de firewall de nivel de base de datos para la base de datos en Azure SQL Database. Las reglas de firewall de base de datos pueden configurarse para la base de datos maestra y las bases de datos de usuario de SQL Database. Las reglas de firewall de base de datos son útiles cuando se usan usuarios de base de datos independientes. |
|[sys.database_firewall_rules](/sql/relational-databases/system-catalog-views/sys-database-firewall-rules-azure-sql-database)|Devuelve información sobre la configuración del firewall de nivel de base de datos asociada a la base de datos de Azure SQL Database. |
|[sp_delete_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-database-firewall-rule-azure-sql-database)|Quita la configuración del firewall de nivel de base de datos de una base de datos. |

## <a name="rest-api"></a>API DE REST

Para crear y administrar los servidores, bases de datos y firewalls, use estas solicitudes de la API REST.

| Get-Help | Descripción |
| --- | --- |
|[Servers - Create or update](/rest/api/sql/servers/createorupdate)|Crea o actualiza un nuevo servidor.|
|[Servers - Delete](/rest/api/sql/servers/delete)|Elimina un servidor de SQL Server.|
|[Servers - Get](/rest/api/sql/servers/get)|Obtiene un servidor.|
|[Servers - List](/rest/api/sql/servers/list)|Devuelve una lista de servidores en una suscripción.|
|[Servers - List by resource group](/rest/api/sql/servers/listbyresourcegroup)|Devuelve una lista de servidores en un grupo de recursos.|
|[Servers - Update](/rest/api/sql/servers/update)|Actualiza un servidor existente.|
|[Databases - Create or update](/rest/api/sql/databases/createorupdate)|Crea una nueva base de datos o actualiza una ya existente.|
|[Databases - Delete](/rest/api/sql/databases/delete)|Permite eliminar una base de datos.|
|[Databases - Get](/rest/api/sql/databases/get)|Obtiene una base de datos.|
|[Databases - List by elastic pool](/rest/api/sql/databases/listbyelasticpool)|Devuelve una lista de bases de datos de un grupo elástico.|
|[Databases - List by server](/rest/api/sql/databases/listbyserver)|Devuelve una lista de bases de datos de un servidor.|
|[Databases - Update](/rest/api/sql/databases/update)|Actualiza una base de datos existente.|
|[Firewall rules - Create or update](/rest/api/sql/firewallrules/createorupdate)|Crea o actualiza una regla de firewall.|
|[Firewall rules - Delete](/rest/api/sql/firewallrules/delete)|Elimina una regla de firewall.|
|[Firewall rules - Get](/rest/api/sql/firewallrules/get)|Obtiene una regla de firewall.|
|[Firewall rules - List by server](/rest/api/sql/firewallrules/listbyserver)|Devuelve una lista de reglas de firewall.|

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo migrar una base de datos de SQL Server a Azure, vea [Migración a Azure SQL Database](migrate-to-database-from-sql-server.md).
- Para obtener información sobre las características admitidas, consulte el [artículo que trata sobre dicho tema](features-comparison.md).
