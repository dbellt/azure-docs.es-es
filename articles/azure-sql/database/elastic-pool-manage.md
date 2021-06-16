---
title: Administración de grupos elásticos
description: Cree y administre grupos elásticos de Azure SQL Database mediante Azure Portal, PowerShell, la CLI de Azure, la API REST y Transact-SQL (T-SQL).
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.topic: conceptual
author: arvindshmicrosoft
ms.author: arvindsh
ms.reviewer: mathoma
ms.date: 03/12/2019
ms.custom: seoapril2019 sqldbrb=1, devx-track-azurecli
ms.openlocfilehash: 601ab778d1db7c6f161062ef5034acbc26e41cf7
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708839"
---
# <a name="manage-elastic-pools-in-azure-sql-database"></a>Administración de grupos elásticos en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Con un grupo elástico, se determina la cantidad de recursos que este requiere para controlar la carga de trabajo de sus bases de datos y la cantidad de recursos para cada base de datos agrupada.

## <a name="azure-portal"></a>Azure portal

Todas las configuraciones de grupo pueden encontrarse en un único lugar: la hoja **Configurar grupo**. Para llegar aquí, busque un grupo elástico en Azure Portal y haga clic en **Configurar grupo** en la parte superior de la hoja o desde el menú de recursos de la izquierda.

Desde aquí puede realizar cualquier combinación de los siguientes cambios y guardarlos todos en un lote:

1. Cambiar el nivel de servicio del grupo
2. Escalar o reducir verticalmente el rendimiento (DTU o núcleos virtuales) y el almacenamiento
3. Agregar o quitar bases de datos en el grupo
4. Establecer un límite de rendimiento mínimo (garantizado) y máximo para las bases de datos de los grupos
5. Revisar el resumen de costos para ver los cambios en su factura como resultado de las nuevas selecciones

![Hoja de configuración del grupo elástico](./media/elastic-pool-manage/configure-pool.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Para crear y administrar grupos elásticos y bases de datos agrupadas de SQL Database con Azure PowerShell, use los siguientes cmdlets de PowerShell. Si necesita instalar o actualizar PowerShell, consulte [Install and configure Azure PowerShell](/powershell/azure/install-az-ps) (Instalación y configuración de Azure PowerShell). Para crear y administrar los servidores para un grupo elástico, consulte el artículo sobre cómo [crear y administrar servidores](logical-servers.md). Para crear y administrar reglas de firewall, consulte la sección [Administración de reglas de firewall mediante Azure PowerShell](firewall-configure.md#use-powershell-to-manage-server-level-ip-firewall-rules).

> [!TIP]
> Para obtener scripts de ejemplo de PowerShell, vea [Creación de grupos elásticos y traslado de bases de datos de un grupo a otro y fuera de un grupo mediante PowerShell](scripts/move-database-between-elastic-pools-powershell.md) y [Uso de PowerShell para supervisar y escalar un grupo elástico de SQL en Azure SQL Database](scripts/monitor-and-scale-pool-powershell.md).
>

| Cmdlet | Descripción |
| --- | --- |
|[New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool)|Crea un grupo elástico.|
|[Get-AzSqlElasticPool](/powershell/module/az.sql/get-azsqlelasticpool)|Obtiene los grupos elásticos y sus valores de propiedad.|
|[Set-AzSqlElasticPool](/powershell/module/az.sql/set-azsqlelasticpool)|Modifica las propiedades de un grupo de bases de datos elásticas. Por ejemplo, use la propiedad **StorageMB** para modificar el almacenamiento máximo de un grupo elástico.|
|[Remove-AzSqlElasticPool](/powershell/module/az.sql/remove-azsqlelasticpool)|Elimina un grupo elástico.|
|[Get-AzSqlElasticPoolActivity](/powershell/module/az.sql/get-azsqlelasticpoolactivity)|Obtiene el estado de las operaciones en un grupo elástico.|
|[New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)|Crea una nueva base de datos en un grupo existente o como una sola base de datos. |
|[Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase)|Obtiene una o más bases de datos.|
|[Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)|Establece las propiedades de una base de datos o mueve una base de datos existente a un grupo elástico, fuera de él o entre grupos elásticos.|
|[Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase)|Quita una base de datos.|

> [!TIP]
> La creación de varias bases de datos en un grupo elástico puede tardar tiempo cuando se realiza mediante el portal o los cmdlets de PowerShell que crean una base de datos única cada vez. Para automatizar la creación en un grupo elástico, vea [CreateOrUpdateElasticPoolAndPopulate](https://gist.github.com/billgib/d80c7687b17355d3c2ec8042323819ae).

## <a name="azure-cli"></a>Azure CLI

Para crear y administrar grupos elásticos de SQL Database con la [CLI de Azure](/cli/azure), use los siguientes comandos de la [CLI de Azure para SQL Database](/cli/azure/sql/db). Use [Cloud Shell](../../cloud-shell/overview.md) para ejecutar la CLI en el explorador o [instálela](/cli/azure/install-azure-cli) en Windows, Linux o macOS.

> [!TIP]
> Para obtener scripts de ejemplo de la CLI de Azure, vea los artículos [Uso de la CLI para mover una base de datos de SQL Database en un grupo elástico de SQL](scripts/move-database-between-elastic-pools-cli.md) y [Uso de la CLI de Azure para escalar un grupo elástico de SQL en Azure SQL Database](scripts/scale-pool-cli.md).
>

| Cmdlet | Descripción |
| --- | --- |
|[az sql elastic-pool create](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_create)|Crea un grupo elástico.|
|[az sql elastic-pool list](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_list)|Devuelve una lista de grupos elásticos de un servidor.|
|[az sql elastic-pool list-dbs](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_list_dbs)|Devuelve una lista de bases de datos de un grupo elástico.|
|[az sql elastic-pool list-editions](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_list_editions)|Además incluye los parámetros disponibles de DTU de grupo, los límites de almacenamiento y la configuración por base de datos. Para reducir el nivel de detalle, los límites de almacenamiento y la configuración por base de datos adicionales están ocultos de forma predeterminada.|
|[az sql elastic-pool update](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_update)|Actualiza un grupo elástico.|
|[az sql elastic-pool delete](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_delete)|Elimina el grupo elástico.|

## <a name="transact-sql-t-sql"></a>Transact-SQL (T-SQL)

Para crear y mover bases de datos en grupos elásticos existentes o para devolver información sobre un grupo elástico de SQL Database con Transact-SQL, use los siguientes comandos de T-SQL. Puede emitir estos comandos mediante Azure Portal, [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), [Visual Studio Code](https://code.visualstudio.com/docs) o cualquier otro programa que pueda conectarse a un servidor y pasar comandos de Transact-SQL. Para crear y administrar reglas de firewall mediante T-SQL, consulte [Administración de reglas de firewall mediante Transact-SQL](firewall-configure.md#use-transact-sql-to-manage-ip-firewall-rules).

> [!IMPORTANT]
> No es posible crear, actualizar ni eliminar un grupo elástico de Azure SQL Database mediante Transact-SQL. Puede agregar o quitar bases de datos de un grupo elástico y puede usar DMV para devolver información sobre grupos elásticos existentes.
>

| Get-Help | Descripción |
| --- | --- |
|[CREATE DATABASE (Azure SQL Database)](/sql/t-sql/statements/create-database-azure-sql-database)|Crea una nueva base de datos en un grupo existente o como una sola base de datos. Debe estar conectado a la base de datos maestra para crear una base de datos.|
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) |Mueve una base de datos a un grupo elástico, fuera de él o entre grupos elásticos.|
|[DROP DATABASE (Transact-SQL)](/sql/t-sql/statements/drop-database-transact-sql)|Permite eliminar una base de datos.|
|[sys.elastic_pool_resource_stats (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database)|Devuelve estadísticas de uso de recursos de todos los grupos elásticos de un servidor. Para cada grupo elástico hay una fila por cada ventana de informe de 15 segundos (cuatro filas por minuto). Esto incluye uso de CPU, E/S, registro, almacenamiento y empleo simultáneo de solicitudes o sesiones por parte de todas las bases de datos del grupo.|
|[sys.database_service_objectives (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Devuelve la edición (nivel de servicio), el objetivo de servicio (plan de tarifa) y el nombre del grupo elástico, si existe, para una base de datos de SQL Database o Azure Synapse Analytics. Si ha iniciado sesión en la base de datos maestra en un servidor, devuelve información sobre todas las bases de datos. Para Azure Synapse Analytics, debe estar conectado a la base de datos maestra.|

## <a name="rest-api"></a>API DE REST

Para crear y administrar grupos elásticos y bases de datos agrupadas de SQL Database, use estas solicitudes de la API REST.

| Get-Help | Descripción |
| --- | --- |
|[Elastic pools - Create or update](/rest/api/sql/elasticpools/createorupdate)|Crea un nuevo grupo elástico o actualiza uno ya existente.|
|[Grupos elásticos: eliminación](/rest/api/sql/elasticpools/delete)|Elimina el grupo elástico.|
|[Grupos elásticos: obtención](/rest/api/sql/elasticpools/get)|Obtiene un grupo elástico.|
|[Elastic pools - List by server](/rest/api/sql/elasticpools/listbyserver)|Devuelve una lista de grupos elásticos de un servidor.|
|[Grupos elásticos: actualización](/rest/api/sql/2020-11-01-preview/elasticpools/update)|Actualiza un grupo elástico ya existente.|
|[Elastic pool activities](/rest/api/sql/elasticpoolactivities)|Devuelve las actividades de los grupos elásticos.|
|[Elastic pool database activities](/rest/api/sql/elasticpooldatabaseactivities)|Devuelve la actividad de las bases de datos de un grupo elástico.|
|[Databases - Create or update](/rest/api/sql/databases/createorupdate)|Crea una nueva base de datos o actualiza una ya existente.|
|[Databases - Get](/rest/api/sql/databases/get)|Obtiene una base de datos.|
|[Databases - List by elastic pool](/rest/api/sql/databases/listbyelasticpool)|Devuelve una lista de bases de datos de un grupo elástico.|
|[Databases - List by server](/rest/api/sql/databases/listbyserver)|Devuelve una lista de bases de datos de un servidor.|
|[Databases - Update](/rest/api/sql/databases/update)|Actualiza una base de datos existente.|

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de los modelos de diseño de las aplicaciones SaaS que usan grupos elásticos, consulte [Modelos de diseño para aplicaciones SaaS multiinquilino con Azure SQL Database](saas-tenancy-app-design-patterns.md).
* Para obtener un tutorial sobre SaaS con grupos elásticos, vea [Introducción a la aplicación de SaaS Wingtip](saas-dbpertenant-wingtip-app-overview.md).
