---
title: 'PowerShell: Migración de SQL Server a SQL Managed Instance en línea'
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre cómo migrar en línea de SQL Server a Azure SQL Managed Instance mediante Azure PowerShell y Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: devx-track-azurepowershell
ms.topic: how-to
ms.date: 12/16/2020
ms.openlocfilehash: 8947a9bb62306d3497e3888c15ea6f82d82d056e
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702866"
---
# <a name="migrate-sql-server-to-sql-managed-instance-online-with-powershell--azure-database-migration-service"></a>Migración de SQL Server a SQL Managed Instance en línea con PowerShell y Azure Database Migration Service

En este artículo, se va a migrar en línea la base de datos **Adventureworks2016** restaurada en una instancia local de SQL Server 2005, o una versión superior, a una instancia de Azure SQL Managed Instance mediante Microsoft Azure PowerShell. Se pueden migrar bases de datos desde una instancia de SQL Server a una Instancia administrada de SQL mediante el módulo `Az.DataMigration` en Microsoft Azure PowerShell.

En este artículo aprenderá a:
> [!div class="checklist"]
>
> * Cree un grupo de recursos.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración en una instancia de Azure Database Migration Service.
> * Ejecute la migración en línea.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se proporcionan los pasos para realizar una migración en línea, pero también es posible realizarla [sin conexión](howto-sql-server-to-azure-sql-managed-instance-powershell-offline.md).


## <a name="prerequisites"></a>Requisitos previos

Para completar estos pasos, necesitará lo siguiente:

* [SQL Server 2016 o posterior](https://www.microsoft.com/sql-server/sql-server-downloads) (cualquier edición).
* Una copia local de la base de datos **AdventureWorks2016**, que está disponible para su descarga [aquí](/sql/samples/adventureworks-install-configure).
* Para habilitar el protocolo TCP/IP, que se deshabilita de forma predeterminada con la instalación de SQL Server Express. Habilite el protocolo TCP/IP siguiendo el artículo [Habilitar o deshabilitar un protocolo de red de servidor](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
* Configurar [Firewall de Windows para el acceso al motor de base de datos](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una Instancia administrada de SQL. Se puede crear una Instancia administrada de SQL mediante los pasos que se describen en el artículo [Creación de una Instancia administrada de ASQL](../azure-sql/managed-instance/instance-create-quickstart.md).
* Descargar e instalar [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) versión 3.3 o posterior.
* Una instancia de Microsoft Azure Virtual Network creada con el modelo de implementación de Azure Resource Manager, que proporciona a Azure Database Migration Service conectividad de sitio a sitio a los servidores de origen locales a través de [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Una evaluación completada de la base de datos local y la migración del esquema con Data Migration Assistant, como se describe en el artículo [Realizar una evaluación de migración de SQL Server](/sql/dma/dma-assesssqlonprem).
* Descargar e instalar el módulo `Az.DataMigration` (versión 0.7.2 o posterior) de la Galería de PowerShell con el [cmdlet Install-Module de PowerShell](/powershell/module/powershellget/Install-Module).
* Asegurarse de que las credenciales usadas para conectarse a la instancia de SQL Server de origen tengan el permiso [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Asegurarse de que las credenciales usadas para conectarse a la Instancia administrada de SQL de destino tengan el permiso CONTROL DATABASE en las bases de datos de la Instancia administrada de SQL de destino.

    > [!IMPORTANT]
    > Para las migraciones en línea, tendrá que haber configurado las credenciales de Azure Active Directory. Para más información, consulte el artículo [Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos mediante el comando [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup).

En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la región *Este de EE. UU.*

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-dms"></a>Creación de una instancia de DMS

Puede crear una instancia nueva de Azure Database Migration Service mediante el cmdlet `New-AzDataMigrationService`.
Este cmdlet espera los siguientes parámetros requeridos:

* *Nombre del grupo de recursos de Azure*. Puede usar el comando [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup) para crear un grupo de recursos de Azure como se mostró anteriormente y proporcionar su nombre como un parámetro.
* *Nombre del servicio*. Cadena que se corresponde con el nombre de servicio único deseado para Azure Database Migration Service.
* *Ubicación*. Especifica la ubicación del servicio. Especifique una ubicación de centro de datos de Azure, por ejemplo, Oeste de EE. UU. o Sudeste de Asia.
* *SKU*. Este parámetro corresponde al nombre de SKU de DMS. Los nombres de SKU admitidos actualmente son *Basic_1vCore*, *Basic_2vCores* y *GeneralPurpose_4vCores*.
* *Identificador de subred virtual*. Puede usar el cmdlet [`New-AzVirtualNetworkSubnetConfig`](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) para crear una subred.

En el ejemplo siguiente se crea un servicio denominado *MyDMS* en el grupo de recursos *MyDMSResourceGroup* que se encuentra en la región *Este de EE. UU.* con una red virtual denominada *MyVNET* y una subred llamada *MySubnet*.


```powershell
$vNet = Get-AzVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Cree un proyecto de migración después de crear una instancia de Azure Database Migration Service. Un proyecto de Azure Database Migration Service requiere información de conexión para las instancias de origen y de destino, así como una lista de bases de datos que desee migrar como parte del proyecto.
Defina las cadenas de conexión de conectividad de origen y de destino.

El siguiente script define los detalles de conexión de SQL Server de origen: 

```powershell
# Source connection properties
$sourceDataSource = "<mysqlserver.domain.com/privateIP of source SQL>"
$sourceUserName = "domain\user"
$sourcePassword = "mypassword"
```

El siguiente script define los detalles de conexión de SQL Managed Instance de destino: 

```powershell
# Target MI connection properties
$targetMIResourceId = "/subscriptions/<subid>/resourceGroups/<rg>/providers/Microsoft.Sql/managedInstances/<myMI>"
$targetUserName = "<user>"
$targetPassword = "<password>"
```



### <a name="define-source-and-target-database-mapping"></a>Definición de la asignación de bases de datos de origen y de destino

Provisión de las bases de datos que se van a migrar en este proyecto de migración

El siguiente script asigna la base de datos de origen a la nueva base de datos correspondiente en la instancia de SQL Managed Instance de destino con el nombre proporcionado.

```powershell
# Selected databases (Source database name to target database name mapping)
$selectedDatabasesMap = New-Object System.Collections.Generic.Dictionary"[String,String]" 
$selectedDatabasesMap.Add("<source  database name>", "<target database name> ")
```

En el caso de varias bases de datos, agregue la lista de bases de datos al script anterior con el siguiente formato:

```powershell
$selectedDatabasesMap = New-Object System.Collections.Generic.Dictionary"[String,String]" 
$selectedDatabasesMap.Add("<source  database name1>", "<target database name1> ")
$selectedDatabasesMap.Add("<source  database name2>", "<target database name2> ")
```

### <a name="create-dms-project"></a>Creación de proyecto DMS

Puede crear un proyecto de Azure Database Migration Service dentro de la instancia de DMS.

```powershell
# Create DMS project
$project = New-AzDataMigrationProject `
  -ResourceGroupName $dmsResourceGroupName `
  -ServiceName $dmsServiceName `
  -ProjectName $dmsProjectName `
  -Location $dmsLocation `
  -SourceType SQL `
  -TargetType SQLMI `

# Create selected databases object
$selectedDatabases = @();
foreach ($sourceDbName in $selectedDatabasesMap.Keys){
    $targetDbName = $($selectedDatabasesMap[$sourceDbName])
    $selectedDatabases += New-AzDmsSelectedDB -MigrateSqlServerSqlDbMi `
      -Name $sourceDbName `
      -TargetDatabaseName $targetDbName `
      -BackupFileShare $backupFileShare `
}
```



### <a name="create-a-backup-fileshare-object"></a>Creación de un objeto FileShare para copias de seguridad

Cree ahora un objeto FileShare que representa el recurso compartido de red SMB local en el que Azure Database Migration Service puede realizar las copias de seguridad de la base de datos de origen con el cmdlet New-AzDmsFileShare.

```powershell
# SMB Backup share properties
$smbBackupSharePath = "\\shareserver.domain.com\mybackup"
$smbBackupShareUserName = "domain\user"
$smbBackupSharePassword = "<password>"

# Create backup file share object
$smbBackupSharePasswordSecure = ConvertTo-SecureString -String $smbBackupSharePassword -AsPlainText -Force
$smbBackupShareCredentials = New-Object System.Management.Automation.PSCredential ($smbBackupShareUserName, $smbBackupSharePasswordSecure)
$backupFileShare = New-AzDmsFileShare -Path $smbBackupSharePath -Credential $smbBackupShareCredentials
```

### <a name="define-the-azure-storage"></a>Definición de la instancia de Azure Storage 

Seleccione el contenedor de Azure Storage que se va a usar para la migración: 

```powershell
# Storage resource id
$storageAccountResourceId = "/subscriptions/<subscriptionname>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<mystorage>"
```


### <a name="configure-azure-active-directory-app"></a>Configuración de la aplicación de Azure Active Directory

Proporcione los detalles necesarios para Azure Active Directory para una migración de SQL Managed Instance en línea: 

```powershell
# AAD properties
$AADAppId = "<appid-guid>"
$AADAppKey = "<app-key>"

# Create AAD object
$AADAppKeySecure = ConvertTo-SecureString $AADAppKey -AsPlainText -Force
$AADApp = New-AzDmsAadApp -ApplicationId $AADAppId -AppKey $AADAppKeySecure
```


## <a name="create-and-start-a-migration-task"></a>Creación e inicio de una tarea de migración

Después, cree e inicie una tarea de Azure Database Migration Service. Llame al origen y al destino mediante variables y enumere las tablas de la base de datos que se van a migrar: 


```powershell
# Managed Instance online migration properties
$dmsTaskName = "testmigration1"

# Create source connection info
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource $sourceDataSource `
  -AuthType WindowsAuthentication `
  -TrustServerCertificate:$true
$sourcePasswordSecure = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCredentials = New-Object System.Management.Automation.PSCredential ($sourceUserName, $sourcePasswordSecure)

# Create target connection info
$targetConnInfo = New-AzDmsConnInfo -ServerType SQLMI `
    -MiResourceId $targetMIResourceId
$targetPasswordSecure = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCredentials = New-Object System.Management.Automation.PSCredential ($targetUserName, $targetPasswordSecure)
```

En el ejemplo siguiente se crea y se inicia una tarea de migración en línea: 

```powershell
# Create DMS migration task
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMiSync `
  -ResourceGroupName $dmsResourceGroupName `
  -ServiceName $dmsServiceName `
  -ProjectName $dmsProjectName `
  -TaskName $dmsTaskName `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCredentials `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCredentials `
  -SelectedDatabase  $selectedDatabases `
  -BackupFileShare $backupFileShare `
  -AzureActiveDirectoryApp $AADApp `
  -StorageResourceId $storageAccountResourceId
```

Para obtener más información, consulte [New-AzDataMigrationTask](/powershell/module/az.datamigration/new-azdatamigrationtask).

## <a name="monitor-the-migration"></a>Supervisión de la migración

Para supervisar la migración, realice las tareas siguientes.

### <a name="check-the-status-of-task"></a>Comprobación del estado de la tarea

```powershell
# Get migration task status details
$migTask = Get-AzDataMigrationTask `
                    -ResourceGroupName $dmsResourceGroupName `
                    -ServiceName $dmsServiceName `
                    -ProjectName $dmsProjectName `
                    -Name $dmsTaskName `
                    -ResultType DatabaseLevelOutput `
                    -Expand

# Task state will be either of 'Queued', 'Running', 'Succeeded', 'Failed', 'FailedInputValidation' or 'Faulted'
$taskState = $migTask.ProjectTask.Properties.State

# Display task state
$taskState | Format-List
```

Use lo siguiente para obtener una lista de errores:-

```powershell
# Get task errors
$taskErrors = $migTask.ProjectTask.Properties.Errors

# Display task errors
foreach($taskError in $taskErrors){
    $taskError |  Format-List
}


# Get database level details
$databaseLevelOutputs = $migTask.ProjectTask.Properties.Output

# Display database level details
foreach($databaseLevelOutput in $databaseLevelOutputs){

    # This is the source database name.
    $databaseName = $databaseLevelOutput.SourceDatabaseName;

    Write-Host "=========="
    Write-Host "Start migration details for database " $databaseName
    # This is the status for that database - It will be either of:
    # INITIAL, FULL_BACKUP_UPLOADING, FULL_BACKUP_UPLOADED, LOG_FILES_UPLOADING,
    # CUTOVER_IN_PROGRESS, CUTOVER_INITIATED, CUTOVER_COMPLETED, COMPLETED, CANCELLED, FAILED
    $databaseMigrationState = $databaseLevelOutput.MigrationState;

    # Details about last restored backup. This contains file names, LSN, backup date, etc 
    $databaseLastRestoredBackup = $databaseLevelOutput.LastRestoredBackupSetInfo
        
    # Details about last restored backup. This contains file names, LSN, backup date, etc 
    $databaseLastRestoredBackup = $databaseLevelOutput.LastRestoredBackupSetInfo

    # Details about last Currently active/most recent backups. This contains file names, LSN, backup date, etc 
    $databaseActiveBackpusets = $databaseLevelOutput.ActiveBackupSets

    # Display info
    $databaseLevelOutput | Format-List

    Write-Host "Currently active/most recent backupset details:"
    $databaseActiveBackpusets  | select BackupStartDate, BackupFinishedDate, FirstLsn, LastLsn -ExpandProperty ListOfBackupFiles | Format-List

    Write-Host "Last restored backupset details:"
    $databaseLastRestoredBackupFiles  | Format-List

    Write-Host "End migration details for database " $databaseName
    Write-Host "=========="
}
```

## <a name="performing-the-cutover"></a>Realización de la transición 

Con una migración en línea, se lleva a cabo una copia de seguridad y restauración completas de las bases de datos. Después, el trabajo procede a restaurar los registros de transacciones almacenados en BackupFileShare.

Cuando la base de datos de una Instancia administrada de Azure SQL se haya actualizado con los datos más recientes y esté sincronizada con la base de datos de origen, podrá realizar una migración total.

En el ejemplo siguiente se completa la migración total. Los usuarios invocan este comando a su discreción.

```powershell
$command = Invoke-AzDmsCommand -CommandType CompleteSqlMiSync `
                               -ResourceGroupName myResourceGroup `
                               -ServiceName $service.Name `
                               -ProjectName $project.Name `
                               -TaskName myDMSTask `
                               -DatabaseName "Source DB Name"
```

## <a name="deleting-the-instance-of-azure-database-migration-service"></a>Eliminación de la instancia de Azure Database Migration Service

Una vez completada la migración, puede eliminar la instancia de Azure Database Migration Service:

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="additional-resources"></a>Recursos adicionales

Para obtener información sobre escenarios de migración adicionales (pares de origen/destino), consulte la [guía de Database Migration](https://datamigration.microsoft.com/) de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Azure Database Migration Service en el artículo [¿Qué es Azure Database Migration Service?](./dms-overview.md)
