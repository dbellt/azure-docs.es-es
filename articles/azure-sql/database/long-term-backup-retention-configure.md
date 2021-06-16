---
title: 'Azure SQL Database: Administración de la retención de copia de seguridad a largo plazo'
description: Obtenga información sobre cómo almacenar y restaurar copias de seguridad automatizadas para Azure SQL Database en Azure Storage (durante un máximo de 10 años) mediante Azure Portal y PowerShell.
services: sql-database
ms.service: sql-db-mi
ms.subservice: backup-restore
ms.custom: devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: shkale-msft
ms.author: shkale
ms.reviewer: mathoma
ms.date: 12/16/2020
ms.openlocfilehash: d11a9cc597adb7bed4a1c1b235c6d7bf68457f78
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705571"
---
# <a name="manage-azure-sql-database-long-term-backup-retention"></a>Administración de la retención de copias de seguridad a largo plazo de Azure SQL Database
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Con Azure SQL Database puede establecer una directiva de [retención de copias de seguridad a largo plazo](long-term-retention-overview.md) (LTR) para conservar automáticamente las copias de seguridad en contenedores de Azure Blob Storage hasta 10 años. Posteriormente, puede recuperar una base de datos mediante esas copias de seguridad con Azure Portal o PowerShell. Las directivas de retención a largo plazo también se admiten en [Azure SQL Managed Instance](../managed-instance/long-term-backup-retention-configure.md).

## <a name="using-the-azure-portal"></a>Uso de Azure Portal

En las secciones siguientes se muestra cómo usar Azure Portal para establecer directivas de retención a largo plazo, administrar copias de seguridad de retención a largo plazo disponibles y restaurar a partir de una copia de seguridad disponible.

### <a name="configure-long-term-retention-policies"></a>Configuración de directivas de retención a largo plazo

Puede configurar SQL Database para [ conservar las copias de seguridad automatizadas](long-term-retention-overview.md) durante un período superior al período de retención del nivel de servicio.

1. En Azure Portal, vaya al servidor y seleccione **Copias de seguridad**. Seleccione la pestaña **Directivas de retención** para modificar la configuración de retención de copias de seguridad.

   ![Experiencia de directivas de retención](./media/long-term-backup-retention-configure/ltr-policies-tab.png)

2. En la pestaña Directivas de retención, seleccione las bases de datos cuyas directivas de retención de copias de seguridad a largo plazo quiere establecer o modificar. Las bases de datos no seleccionadas no se ven afectadas.

   ![Selección de base de datos para configurar directivas de retención de copias de seguridad](./media/long-term-backup-retention-configure/ltr-policies-tab-configure.png)

3. En el panel **Configurar directivas**, especifique el período de retención que quiera para las copias de seguridad semanales, mensuales o anuales. Seleccione un período de retención de "0" para indicar que no se debe establecer una retención de copias de seguridad a largo plazo.

   ![Panel Configurar directivas](./media/long-term-backup-retention-configure/ltr-configure-policies.png)

4. Seleccione **Aplicar** para aplicar la configuración de retención elegida a todas las bases de datos seleccionadas.

> [!IMPORTANT]
> Cuando se habilita una directiva de retención de copia de seguridad a largo plazo, la primera copia de seguridad puede tardar hasta 7 días en mostrarse y estar disponible para restaurar. Para saber más sobre la cadencia de retención de copia de seguridad a largo plazo, vea [Retención de copia de seguridad a largo plazo](long-term-retention-overview.md).

### <a name="view-backups-and-restore-from-a-backup"></a>Visualización y restauración de copias de seguridad

Vea las copias de seguridad que se han conservado para una base de datos concreta con una directiva LTR y realice la restauración a partir de esas copias de seguridad.

1. En Azure Portal, vaya al servidor y seleccione **Copias de seguridad**. Para ver las copias de seguridad de LTR disponibles para una base de datos concreta, seleccione **Administrar** en la columna de copias de seguridad de LTR disponibles. Aparece un panel con una lista de las copias de seguridad de LTR disponibles para la base de datos seleccionada.

   ![Experiencia de copias de seguridad disponibles](./media/long-term-backup-retention-configure/ltr-available-backups-tab.png)

1. En el panel **Copias de seguridad de retención a largo plazo disponibles**, revise las copias de seguridad disponibles. Puede seleccionar una copia de seguridad a partir de la que restaurar o puede eliminarla.

   ![Visualización de copias de seguridad de LTR disponibles](./media/long-term-backup-retention-configure/ltr-available-backups-manage.png)

1. Para restaurar a partir de una copia de seguridad de LTR disponible, seleccione la copia de seguridad a partir de la que quiere realizar la restauración y luego **Restaurar**.

   ![Restauración a partir de una copia de seguridad de LTR disponible](./media/long-term-backup-retention-configure/ltr-available-backups-restore.png)

1. Seleccione un nombre para la nueva base de datos y luego **Revisar y crear** para revisar los detalles de la restauración. Seleccione **Crear** para restaurar la base de datos a partir de la copia de seguridad elegida.

   ![Configuración de los detalles de la restauración](./media/long-term-backup-retention-configure/restore-ltr.png)

1. En la barra de herramientas, seleccione el icono de notificación para ver el estado del trabajo de restauración.

   ![progreso del trabajo de restauración](./media/long-term-backup-retention-configure/restore-job-progress-long-term.png)

1. Cuando se complete el trabajo de restauración, abra la página **Bases de datos SQL** para ver la base de datos recién restaurada.

> [!NOTE]
> Desde aquí, puede conectarse a la base de datos restaurada mediante SQL Server Management Studio para realizar las tareas necesarias, como [extraer un bit de datos de la base de datos restaurada para copiarlo en la base de datos existente o para eliminar la base de datos existente y cambiar el nombre de la base de datos restaurada por el nombre de la base de datos existente](recovery-using-backups.md#point-in-time-restore).

## <a name="using-powershell"></a>Usar PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

En las siguientes secciones se explica cómo usar PowerShell para configurar la retención de copias de seguridad a largo plazo, ver las copias de seguridad en el almacenamiento de Azure y realizar la restauración a partir de una copia de seguridad en el almacenamiento de Azure.

### <a name="azure-roles-to-manage-long-term-retention"></a>Roles de Azure para administrar la retención a largo plazo

Para **Get-AzSqlDatabaseLongTermRetentionBackup** y **Restore-AzSqlDatabase**, deberá tener uno de los siguientes roles:

- Rol de propietario de la suscripción o
- Rol de colaborador de SQL Server o
- Rol personalizado con los permisos siguientes:

   Microsoft.Sql/locations/longTermRetentionBackups/read  Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionBackups/read  Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/read

Para **Remove-AzSqlDatabaseLongTermRetentionBackup**, deberá tener uno de los roles siguientes:

- Rol de propietario de la suscripción o
- Rol personalizado con el permiso siguiente:

   Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/delete

> [!NOTE]
> El rol de colaborador de SQL Server no tiene permiso para eliminar las copias de seguridad de LTR.

Se pueden conceder permisos de RBAC de Azure en el ámbito de la *suscripción* o del *grupo de recursos*. Sin embargo, para acceder a las copias de seguridad de LTR que pertenecen a un servidor descartado, se debe conceder el permiso en el ámbito de la *suscripción* de dicho servidor.

- Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/delete

### <a name="create-an-ltr-policy"></a>Creación de una directiva LTR

```powershell
# get the SQL server
$subId = "<subscriptionId>"
$serverName = "<serverName>"
$resourceGroup = "<resourceGroupName>"
$dbName = "<databaseName>"

Connect-AzAccount
Select-AzSubscription -SubscriptionId $subId

$server = Get-AzSqlServer -ServerName $serverName -ResourceGroupName $resourceGroup

# create LTR policy with WeeklyRetention = 12 weeks. MonthlyRetention and YearlyRetention = 0 by default.
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -WeeklyRetention P12W

# create LTR policy with WeeklyRetention = 12 weeks, YearlyRetention = 5 years and WeekOfYear = 16 (week of April 15). MonthlyRetention = 0 by default.
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -WeeklyRetention P12W -YearlyRetention P5Y -WeekOfYear 16
```

### <a name="view-ltr-policies"></a>Visualización de directivas de LTR

Este ejemplo muestra cómo enumerar las directivas de LTR de un servidor

```powershell
# get all LTR policies within a server
$ltrPolicies = Get-AzSqlDatabase -ResourceGroupName $resourceGroup -ServerName $serverName | `
    Get-AzSqlDatabaseLongTermRetentionPolicy

# get the LTR policy of a specific database
$ltrPolicies = Get-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup
```

### <a name="clear-an-ltr-policy"></a>Borrado de una directiva LTR

Este ejemplo muestra cómo borrar una directiva LTR desde una base de datos

```powershell
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -RemovePolicy
```

### <a name="view-ltr-backups"></a>Visualización de copias de seguridad de LTR

Este ejemplo muestra cómo enumerar las copias de seguridad de LTR de un servidor.

```powershell
# get the list of all LTR backups in a specific Azure region
# backups are grouped by the logical database id, within each group they are ordered by the timestamp, the earliest backup first
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location

# get the list of LTR backups from the Azure region under the named server
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -ServerName $serverName

# get the LTR backups for a specific database from the Azure region under the named server
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -ServerName $serverName -DatabaseName $dbName

# list LTR backups only from live databases (you have option to choose All/Live/Deleted)
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -DatabaseState Live

# only list the latest LTR backup for each database
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -ServerName $serverName -OnlyLatestPerDatabase
```

### <a name="delete-ltr-backups"></a>Eliminación de copias de seguridad de LTR

En este ejemplo se muestra cómo eliminar una copia de seguridad de LTR desde la lista de copias de seguridad.

```powershell
# remove the earliest backup
$ltrBackup = $ltrBackups[0]
Remove-AzSqlDatabaseLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId
```

> [!IMPORTANT]
> La eliminación de la copia de seguridad de LTR no es reversible. Para eliminar una copia de seguridad de LTR una vez eliminado el servidor, debe tener el permiso del ámbito de la suscripción. Puede configurar notificaciones sobre cada eliminación en Azure Monitor filtrando por la operación "Elimina una copia de seguridad de retención a largo plazo". El registro de actividad contiene información sobre quién y cuándo realizó la solicitud. Consulte [Creación de alertas del registro de actividad](../../azure-monitor/alerts/alerts-activity-log.md) para obtener instrucciones detalladas.

### <a name="restore-from-ltr-backups"></a>Restauración desde copias de seguridad de LTR

Este ejemplo muestra cómo restaurar desde una copia de seguridad de LTR. Observe que aunque esta interfaz no cambió, el parámetro de id. de recurso requiere ahora el id. de recurso de la copia de seguridad de LTR.

```powershell
# restore a specific LTR backup as an P1 database on the server $serverName of the resource group $resourceGroup
Restore-AzSqlDatabase -FromLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId -ServerName $serverName -ResourceGroupName $resourceGroup `
    -TargetDatabaseName $dbName -ServiceObjectiveName P1
```

> [!IMPORTANT]
> Para restaurar a partir de una copia de seguridad de LTR una vez eliminado el servidor o grupo de recursos, debe tener los permisos de la suscripción del servidor y dicha suscripción debe estar activa. También debe omitir el parámetro -ResourceGroupName opcional.

> [!NOTE]
> Desde aquí, puede conectarse a la base de datos restaurada mediante SQL Server Management Studio para realizar las tareas necesarias, como extraer un bit de datos de la base de datos restaurada para copiarlo en la base de datos existente o para eliminar la base de datos existente y cambiar el nombre de la base de datos restaurada por el nombre de la base de datos existente. Consulte la [restauración a un momento dado](recovery-using-backups.md#point-in-time-restore).

## <a name="limitations"></a>Limitaciones
- Al restaurar a partir de una copia de seguridad de LTR, la propiedad de escalado de lectura está deshabilitada. Para habilitarla en la base de datos restaurada, actualice la base de datos después de crearla.
- Debe especificar el objetivo de nivel de servicio de destino, cuando se restaura a partir de una copia de seguridad de LTR, que se creó cuando la base de datos estaba en un grupo elástico. 

## <a name="next-steps"></a>Pasos siguientes

- Para aprender sobre las copias de seguridad automáticas generadas por el servicio, consulte [copias de seguridad automáticas](automated-backups-overview.md)
- Para más información sobre la retención de copia de seguridad a largo plazo, consulte sobre la [retención de copia de seguridad a largo plazo](long-term-retention-overview.md).
