---
title: 'PowerShell: ejecución de una migración sin conexión desde una base de datos de MySQL a Azure Database for MySQL mediante DMS'
titleSuffix: Azure Database Migration Service
description: Aprenda a migrar una base de datos de MySQL local a Azure Database for MySQL mediante Azure Database Migration Service con un script de PowerShell.
services: dms
author: sumitgaurin
ms.author: sgaur
manager: arthiaga
ms.reviewer: arthiaga
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.topic: tutorial
ms.date: 04/11/2021
ms.openlocfilehash: eea4a539c8a2b17a9a6280a8f847e68542d1a8d5
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950636"
---
# <a name="migrate-mysql-to-azure-database-for-mysql-offline-with-powershell--azure-database-migration-service"></a>Migración de MySQL a Azure Database for MySQL sin conexión con PowerShell y Azure Database Migration Service

En este artículo, va a migrar una base de datos de MySQL restaurada a una instancia local a Azure Database for MySQL mediante la funcionalidad de migración sin conexión de Azure Database Migration Service con Microsoft Azure PowerShell. En el artículo se documenta una colección de scripts de PowerShell que se pueden ejecutar en secuencia para realizar la migración sin conexión de la base de datos de MySQL a Azure. Puede descargar el script de PowerShell completo que se describe en este tutorial desde nuestro [repositorio de GitHub](https://github.com/Azure/azure-mysql/tree/master/Azure%20DMS%20-%20MySQL%20Offline%20Migration%20Script).


> [!NOTE]
> Actualmente no es posible ejecutar la migración completa de la base de datos mediante el módulo Az.DataMigration. Mientras tanto, el script de PowerShell de ejemplo se proporciona "tal cual", que usa la [API REST de DMS](/rest/api/datamigration/tasks/get) y permite automatizar la migración. Este script se modificará o dejará de usarse una vez que se haya agregado la compatibilidad oficial en el módulo Az.DataMigration y la CLI de Azure. 

> [!NOTE]
> Los servicios Amazon Relational Database Service (RDS) para MySQL y Amazon Aurora (basado en MySQL) también se admiten como orígenes de la migración.

> [!IMPORTANT]
> Para las migraciones en línea, puede usar herramientas de código abierto, como [MyDumper/MyLoader](https://centminmod.com/mydumper.html), con [replicación de datos de entrada](../mysql/concepts-data-in-replication.md).


El artículo ayuda a automatizar el escenario en el que los nombres de la base de datos de origen y de destino pueden ser iguales o diferentes y como parte de la migración es necesario migrar todas o algunas de las tablas de la base de datos de destino que tengan el mismo nombre y la misma estructura de tabla. Aunque en los artículos se da por supuesto que el origen es una instancia de base de datos de MySQL y el destino es Azure Database for MySQL, se puede usar para migrar de una instancia de Azure Database for MySQL a otra simplemente cambiando el nombre y las credenciales del servidor de origen. También se admite la migración desde servidores MySQL de una versión inferior (v5.6 y versiones posteriores) a versiones posteriores.

[!INCLUDE [preview features callout](../../includes/dms-boilerplate-preview.md)]

En este artículo aprenderá a:
> [!div class="checklist"]
>
> * Migrar el esquema de la base de datos.
> * Cree un grupo de recursos.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración en una instancia de Azure Database Migration Service.
> * Configurar el proyecto de migración para usar la funcionalidad de migración sin conexión para MySQL.
> * Ejecutar la migración.

## <a name="prerequisites"></a>Requisitos previos

Para completar estos pasos, necesitará lo siguiente:

* Disponga de una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free).
* Disponer de una base de datos de MySQL local con la versión 5.6 o posterior. Si no la tiene, descargue e instale [MySQL Community Edition](https://dev.mysql.com/downloads/mysql/) 5.6 o posterior.
* [Creación de una instancia en Azure Database for MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Consulte el artículo [Uso de MySQL Workbench para conectarse y consultar datos en Azure Database for MySQL](../mysql/connect-workbench.md) y obtenga más información sobre cómo conectarse y crear una base de datos mediante la aplicación Workbench. La versión de Azure Database for MySQL debe ser igual o superior a la versión local de MySQL. Por ejemplo, MySQL 5.7 admite la migración a Azure Database for MySQL 5.7 o la actualización a la versión 8.  
* Cree una instancia de Microsoft Azure Virtual Network para Azure Database Migration Service con el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](../virtual-network/index.yml)y, especialmente, los artículos de inicio rápido con detalles paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue el [punto de conexión de servicio de *Microsoft.Sql*](../virtual-network/virtual-network-service-endpoints-overview.md) a la subred en la que se aprovisionará el servicio. Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloqueen el puerto de salida 443 de ServiceTag para Storage y AzureMonitor. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Abra el firewall de Windows para permitir conexiones de la red virtual para que Azure Database Migration Service puede acceder al servidor MySQL de origen que, de manera predeterminada, usa el puerto TCP 3306.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir conexiones desde la red virtual para que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](../azure-sql/database/firewall-configure.md) de nivel de servidor o [configure puntos de conexión de servicio de red virtual](../mysql/howto-manage-vnet-using-portal.md) para que la instancia de Azure Database for MySQL de destino permita la red virtual para que Azure Database Migration Service acceda a las bases de datos de destino.
* La versión de la instancia de MySQL de origen debe ser compatible con MySQL community edition. Para determinar la versión de la instancia de MySQL, en la utilidad de MySQL o en MySQL Workbench, ejecute el siguiente comando:

    ```
    SELECT @@version;
    ```

* Azure Database for MySQL solo es compatible con tablas de InnoDB. Para convertir tablas de MyISAM a InnoDB, consulte el artículo [Converting Tables from MyISAM to InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html) (Cómo convertir tablas de MyISAM a InnoDB)
* El usuario debe tener privilegios para leer datos en la base de datos de origen.
* En la guía se usa PowerShell v7.1 con PSEdition Core, que se puede instalar según la [guía de instalación](/powershell/scripting/install/installing-powershell?view=powershell-7.1&preserve-view=true).
* Descargue e instale los siguientes módulos de la Galería de PowerShell con el [cmdlet Install-Module de PowerShell](/powershell/module/powershellget/Install-Module); asegúrese de abrir la ventana de comandos de PowerShell mediante Ejecutar como administrador:
    * Az.Resources
    * Az.Network
    * Az.DataMigration

```powershell
Install-Module Az.Resources
Install-Module Az.Network
Install-Module Az.DataMigration
Import-Module Az.Resources
Import-Module Az.Network
Import-Module Az.DataMigration
```

## <a name="migrate-database-schema"></a>Migración del esquema de base de datos

Para transferir todos los objetos de base de datos, como esquemas de tablas, índices y procedimientos almacenados, es necesario extraer el esquema de la base de datos de origen y aplicarlo a la base de datos de destino. Para extraer el esquema, puede usar mysqldump con el parámetro `--no-data`. Para ello, necesita una máquina que pueda conectarse a la base de datos de MySQL de origen y a la instancia de Azure Database for MySQL de destino.

Para exportar el esquema mediante mysqldump, ejecute el siguiente comando:

```
mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
```

Por ejemplo:

```
mysqldump -h 10.10.123.123 -u root -p --databases migtestdb --no-data > d:\migtestdb.sql
```

Para importar el esquema a la instancia de Azure Database for MySQL de destino, ejecute el siguiente comando:

```
mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
 ```

Por ejemplo:

```
mysql.exe -h mysqlsstrgt.mysql.database.azure.com -u docadmin@mysqlsstrgt -p migtestdb < d:\migtestdb.sql
 ```

Si su esquema utiliza claves externas, la tarea de migración regulará la carga de datos en paralelo durante el proceso. No es necesario descartar las claves externas durante la migración del esquema.

Si la base de datos utiliza desencadenadores, se aplicará la integridad de datos en el destino antes de llevar a cabo la migración completa de los datos desde el origen. Es recomendable deshabilitar los desencadenadores en todas las tablas del destino durante la migración y, a continuación, habilitar los desencadenadores una vez finalizada esta.

Ejecute el siguiente script en MySQL Workbench en la base de datos de destino para extraer el script del desencadenador de anulación y el script del desencadenador de adición.

```sql
SELECT
    SchemaName,
    GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery,
    Concat('DELIMITER $$ \n\n', GROUP_CONCAT(AddQuery SEPARATOR '$$\n'), '$$\n\nDELIMITER ;') as AddQuery
FROM
(
SELECT 
    TRIGGER_SCHEMA as SchemaName,
    Concat('DROP TRIGGER `', TRIGGER_NAME, "`") as DropQuery,
    Concat('CREATE TRIGGER `', TRIGGER_NAME, '` ', ACTION_TIMING, ' ', EVENT_MANIPULATION, 
            '\nON `', EVENT_OBJECT_TABLE, '`\n' , 'FOR EACH ', ACTION_ORIENTATION, ' ',
            ACTION_STATEMENT) as AddQuery
FROM  
    INFORMATION_SCHEMA.TRIGGERS
ORDER BY EVENT_OBJECT_SCHEMA, EVENT_OBJECT_TABLE, ACTION_TIMING, EVENT_MANIPULATION, ACTION_ORDER ASC
) AS Queries
GROUP BY SchemaName
```

Ejecute la consulta del desencadenador de anulación generada (columna DropQuery) en el resultado para quitar los desencadenadores de la base de datos de destino. La consulta para agregar desencadenadores se puede guardar y usar una vez finalizada la migración de datos.

## <a name="log-in-to-your-microsoft-azure-subscription"></a>Inicio de sesión en la suscripción de Microsoft Azure

Use el [comando Connect-AzAccount de PowerShell](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión en la suscripción de Azure mediante PowerShell, según las instrucciones del artículo [Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps).

El siguiente script establece la suscripción predeterminada para la sesión de PowerShell después del inicio de sesión y crea una función de registro auxiliar para los registros de consola con formato.

```powershell
[string] $SubscriptionName = "mySubscription"
$ErrorActionPreference = "Stop";

Connect-AzAccount
Set-AzContext -Subscription $SubscriptionName
$global:currentSubscriptionId = (Get-AzContext).Subscription.Id;

function LogMessage([string] $Message, [bool] $IsProcessing = $false) {
    if ($IsProcessing) {
        Write-Host "$(Get-Date -Format "yyyy-MM-dd HH:mm:ss"): $Message" -ForegroundColor Yellow
    }
    else {
        Write-Host "$(Get-Date -Format "yyyy-MM-dd HH:mm:ss"): $Message" -ForegroundColor Green
    }    
}
```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

El registro del proveedor de recursos debe efectuarse en cada suscripción de Azure una sola vez. Sin el registro, no podrá crear una instancia de **Azure Database Migration Service**.

Para registrar el proveedor de recursos, utilice el comando [Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider). El siguiente script registra el proveedor de recursos necesario para **Azure Database Migration Service**.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.DataMigration
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos antes de crear los recursos de DMS.

Cree un grupo de recursos de Azure con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroup* en la región *Oeste de EE. UU. 2* en la suscripción predeterminada *mySubscription*.

```powershell
# Get the details of resource group
[string] $Location = "westus2"
[string] $ResourceGroupName = "myResourceGroup"

$resourceGroup = Get-AzResourceGroup -Name $ResourceGroupName
if (-not($resourceGroup)) {
    LogMessage -Message "Creating resource group $ResourceGroupName..." -IsProcessing $true
    $resourceGroup = New-AzResourceGroup -Name $ResourceGroupName -Location $Location
    LogMessage -Message "Created resource group - $($resourceGroup.ResourceId)."
}
else { LogMessage -Message "Resource group $ResourceGroupName exists." }
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>Creación de una instancia de Azure Database Migration Service

Puede crear una nueva instancia de Azure Database Migration Service mediante el comando [New-AzDataMigrationService](/powershell/module/az.datamigration/new-azdatamigrationservice). Este comando espera los siguientes parámetros requeridos:
* *Nombre del grupo de recursos de Azure*. Puede usar el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) para crear el grupo de recursos de Azure como se mostró anteriormente y proporcionar su nombre como un parámetro.
* *Nombre del servicio*. Cadena que se corresponde con el nombre de servicio único para Azure Database Migration Service. 
* *Ubicación*. Especifica la ubicación del servicio. Especifique una ubicación de centro de datos de Azure, por ejemplo, Oeste de EE. UU. o Sudeste de Asia.
* *SKU*. Este parámetro corresponde al nombre de SKU de DMS. El nombre de las SKU admitidas actualmente es *Standard_1vCore*, *Standard_2vCores*, *Standard_4vCores* y *Premium_4vCores*.
* *Identificador de subred virtual*. Puede usar el comando [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) para obtener la información de una subred. 

El siguiente script espera que la red virtual *myVirtualNetwork* exista con una subred llamada *default* y, a continuación, crea una instancia de Database Migration Service con el nombre *myDmService* en el grupo de recursos creado en el **paso 3** y en la misma región.

```powershell
# Get a reference to the DMS service - Create if not exists
[string] $VirtualNetworkName = "myVirtualNetwork"
[string] $SubnetName = "default"
[string] $ServiceName = "myDmService"

$dmsServiceResourceId = "/subscriptions/$($global:currentSubscriptionId)/resourceGroups/$ResourceGroupName/providers/Microsoft.DataMigration/services/$ServiceName"
$dmsService = Get-AzResource -ResourceId $dmsServiceResourceId -ErrorAction SilentlyContinue

# Create Azure DMS service if not existing
# Possible values for SKU currently are Standard_1vCore,Standard_2vCores,Standard_4vCores,Premium_4vCores
if (-not($dmsService)) {   
    $virtualNetwork = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VirtualNetworkName
    if (-not ($virtualNetwork)) { throw "ERROR: Virtual Network $VirtualNetworkName does not exists" }

    $subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $virtualNetwork -Name $SubnetName
    if (-not ($subnet)) { throw "ERROR: Virtual Network $VirtualNetworkName does not contains Subnet $SubnetName" }

    LogMessage -Message "Creating Azure Data Migration Service $ServiceName..." -IsProcessing $true
    $dmsService = New-AzDataMigrationService `
        -ResourceGroupName $ResourceGroupName `
        -Name $ServiceName `
        -Location $resourceGroup.Location `
        -Sku Premium_4vCores `
        -VirtualSubnetId $Subnet.Id
    
    $dmsService = Get-AzResource -ResourceId $dmsServiceResourceId
    LogMessage -Message "Created Azure Data Migration Service - $($dmsService.ResourceId)."
}
else { LogMessage -Message "Azure Data Migration Service $ServiceName exists." }
```

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Cree un proyecto de migración después de crear una instancia de Azure Database Migration Service. Un proyecto de migración especifica el tipo de migración que debe realizarse.

El siguiente script crea un proyecto de migración llamado *myfirstmysqlofflineproject* para la migración sin conexión de MySQL a Azure Database for MySQL en la instancia de Database Migration Service creada en el **paso 4** y en la misma región.

```powershell
# Get a reference to the DMS project - Create if not exists
[string] $ProjectName = "myfirstmysqlofflineproject"

$dmsProjectResourceId = "/subscriptions/$($global:currentSubscriptionId)/resourceGroups/$($dmsService.ResourceGroupName)/providers/Microsoft.DataMigration/services/$($dmsService.Name)/projects/$projectName"
$dmsProject = Get-AzResource -ResourceId $dmsProjectResourceId -ErrorAction SilentlyContinue

# Create Azure DMS Project if not existing
if (-not($dmsProject)) {
    LogMessage -Message "Creating Azure DMS project $projectName for MySQL migration ..." -IsProcessing $true

    $newProjectProperties = @{"sourcePlatform" = "MySQL"; "targetPlatform" = "AzureDbForMySQL" }
    $dmsProject = New-AzResource `
        -ApiVersion 2018-03-31-preview `
        -Location $dmsService.Location `
        -ResourceId $dmsProjectResourceId `
        -Properties $newProjectProperties `
        -Force

    LogMessage -Message "Created Azure DMS project $projectName - $($dmsProject.ResourceId)."
}
else { LogMessage -Message "Azure DMS project $projectName exists." }
```

## <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>Creación de un objeto de información de conexión de base de datos para las conexiones de origen y de destino

Después de crear el proyecto de migración, creará la información de conexión de la base de datos. Esta información de conexión se usará para conectarse a los servidores de origen y de destino durante el proceso de migración.

El siguiente script toma el nombre del servidor, el nombre de usuario y la contraseña de las instancias de MySQL de origen y destino y crea los objetos de información de conexión. El script solicita al usuario que escriba la contraseña de las instancias de MySQL de origen y de destino. En el caso de los scripts sin interacción, las credenciales se pueden capturar desde Azure Key Vault. 

```powershell
# Initialize the source and target database server connections
[string] $SourceServerName = "13.66.136.192"
[string] $SourceUserName = "docadmin@mysqlserver"
[securestring] $SourcePassword = Read-Host "Enter MySQL Source Server Password" -AsSecureString

[string] $TargetServerName = "migdocdevwus2mysqlsstrgt.mysql.database.azure.com"
[string] $TargetUserName = "docadmin@migdocdevwus2mysqlsstrgt"
[securestring] $TargetPassword = Read-Host "Enter MySQL Target Server Password" -AsSecureString

function InitConnection(
    [string] $ServerName,
    [string] $UserName,
    [securestring] $Password) {
    $connectionInfo = @{
        "dataSource"             = "";
        "serverName"             = "";
        "port"                   = 3306;
        "userName"               = "";
        "password"               = "";
        "authentication"         = "SqlAuthentication";
        "encryptConnection"      = $true;
        "trustServerCertificate" = $true;
        "additionalSettings"     = "";
        "type"                   = "MySqlConnectionInfo" 
    }

    $connectionInfo.dataSource = $ServerName;
    $connectionInfo.serverName = $ServerName;
    $connectionInfo.userName = $UserName;
    $connectionInfo.password = (ConvertFrom-SecureString -AsPlainText $password).ToString();
    $connectionInfo;
}

# Initialize the source and target connections
LogMessage -Message "Initializing source and target connection objects ..." -IsProcessing $true
$sourceConnInfo = InitConnection `
    $SourceServerName `
    $SourceUserName `
    $SourcePassword;

$targetConnInfo = InitConnection `
    $TargetServerName `
    $TargetUserName `
    $TargetPassword;

LogMessage -Message "Source and target connection object initialization complete."
```

## <a name="extract-the-list-of-table-names-from-the-target-database"></a>Extracción de la lista de nombres de tabla de la base de datos de destino

La lista de tablas de la base de datos se puede extraer mediante una tarea de migración y la información de conexión. La lista de tablas se extraerá de la base de datos de origen y la base de datos de destino para que se puedan realizar la asignación y validación adecuadas. 

El siguiente script toma los nombres de las bases de datos de origen y de destino y, a continuación, extrae la lista de tablas de las bases de datos mediante la tarea de migración *GetUserTablesMySql*. 

```powershell
# Run scenario to get the tables from the target database to build
# the migration table mapping
[string] $TargetDatabaseName = "migtargetdb"
[string] $SourceDatabaseName = "migsourcedb"

function RunScenario([object] $MigrationService, 
    [object] $MigrationProject, 
    [string] $ScenarioTaskName, 
    [object] $TaskProperties, 
    [bool] $WaitForScenario = $true) {
    # Check if the scenario task already exists, if so remove it
    LogMessage -Message "Removing scenario if already exists..." -IsProcessing $true
    Remove-AzDataMigrationTask `
        -ResourceGroupName $MigrationService.ResourceGroupName `
        -ServiceName $MigrationService.Name `
        -ProjectName $MigrationProject.Name `
        -TaskName $ScenarioTaskName `
        -Force;

    # Start the new scenario task using the provided properties
    LogMessage -Message "Initializing scenario..." -IsProcessing $true
    New-AzResource `
        -ApiVersion 2018-03-31-preview `
        -Location $MigrationService.Location `
        -ResourceId "/subscriptions/$($global:currentSubscriptionId)/resourceGroups/$($MigrationService.ResourceGroupName)/providers/Microsoft.DataMigration/services/$($MigrationService.Name)/projects/$($MigrationProject.Name)/tasks/$($ScenarioTaskName)" `
        -Properties $TaskProperties `
        -Force | Out-Null;
    
    LogMessage -Message "Waiting for $ScenarioTaskName scenario to complete..." -IsProcessing $true
    if ($WaitForScenario) {
        $progressCounter = 0;
        do {
            if ($null -ne $scenarioTask) {
                Start-Sleep 10;
            }

            # Get calls can time out and will return a cancellation exception in that case
            $scenarioTask = Get-AzDataMigrationTask `
                -ResourceGroupName $MigrationService.ResourceGroupName `
                -ServiceName $MigrationService.Name `
                -ProjectName $MigrationProject.Name `
                -TaskName $ScenarioTaskName `
                -Expand `
                -ErrorAction Ignore;

            Write-Progress -Activity "Scenario Run $ScenarioTaskName  (Marquee Progress Bar)" `
                -Status $scenarioTask.ProjectTask.Properties.State `
                -PercentComplete $progressCounter
            
            $progressCounter += 10;
            if ($progressCounter -gt 100) { $progressCounter = 10 }
        }
        while (($null -eq $scenarioTask) -or ($scenarioTask.ProjectTask.Properties.State -eq "Running") -or ($scenarioTask.ProjectTask.Properties.State -eq "Queued"))
    }
    Write-Progress -Activity "Scenario Run $ScenarioTaskName" `
        -Status $scenarioTask.ProjectTask.Properties.State `
        -Completed
                 
    # Now get it using REST APIs so we can expand the output  
    LogMessage -Message "Getting expanded task results ..." -IsProcessing $true  
    $psToken = (Get-AzAccessToken -ResourceUrl https://management.azure.com).Token;
    $token = ConvertTo-SecureString -String $psToken -AsPlainText -Force;
    $taskResource = Invoke-RestMethod `
        -Method GET `
        -Uri "https://management.azure.com$($scenarioTask.ProjectTask.Id)?api-version=2018-03-31-preview&`$expand=output" `
        -ContentType "application/json" `
        -Authentication Bearer `
        -Token $token;
    
    $taskResource.properties;
}

# create the get table task properties by initializing the connection and 
# database name
$getTablesTaskProperties = @{
    "input"    = @{
        "connectionInfo"    = $null;
        "selectedDatabases" = $null;
    };
    "taskType" = "GetUserTablesMySql";
};

LogMessage -Message "Running scenario to get the list of tables from the target database..." -IsProcessing $true
$getTablesTaskProperties.input.connectionInfo = $targetConnInfo;
$getTablesTaskProperties.input.selectedDatabases = @($TargetDatabaseName);
# Create a name for the task
$getTableTaskName = "$($TargetDatabaseName)GetUserTables"
# Get the list of tables from the source
$getTargetTablesTask = RunScenario -MigrationService $dmsService `
    -MigrationProject $dmsProject `
    -ScenarioTaskName $getTableTaskName `
    -TaskProperties $getTablesTaskProperties;

if (-not ($getTargetTablesTask)) { throw "ERROR: Could not get target database $TargetDatabaseName table information." }
LogMessage -Message "List of tables from the target database acquired."

LogMessage -Message "Running scenario to get the list of tables from the source database..." -IsProcessing $true
$getTablesTaskProperties.input.connectionInfo = $sourceConnInfo;
$getTablesTaskProperties.input.selectedDatabases = @($SourceDatabaseName);
# Create a name for the task
$getTableTaskName = "$($SourceDatabaseName)GetUserTables"
# Get the list of tables from the source
$getSourceTablesTask = RunScenario -MigrationService $dmsService `
    -MigrationProject $dmsProject `
    -ScenarioTaskName $getTableTaskName `
    -TaskProperties $getTablesTaskProperties;

if (-not ($getSourceTablesTask)) { throw "ERROR: Could not get source database $SourceDatabaseName table information." }
LogMessage -Message "List of tables from the source database acquired."

```

## <a name="build-table-mapping-based-on-user-configuration"></a>Creación de una asignación de tablas basada en la configuración del usuario

Como parte de la configuración de la tarea de migración, creará una asignación entre las tablas de origen y de destino. La asignación se realiza en el nivel de nombre de tabla, pero se supone que la estructura de la tabla (número de columnas, nombres de columna, tipos de datos, etc.) de las tablas asignadas es exactamente la misma.

El siguiente script crea una asignación basada en la lista de tablas de origen y de destino extraída en el **paso 7**. Para la carga parcial de datos, el usuario puede proporcionar una lista de tablas para filtrar las tablas. Si no se proporciona ninguna entrada de usuario, se asignan todas las tablas de destino. El script también comprueba si existe o no una tabla con el mismo nombre en el origen. Si el nombre de tabla no existe en el origen, la tabla de destino se omite para la migración. 

```powershell
# Create the source to target table map
# Optional table settings
# DEFAULT: $IncludeTables = $null => include all tables for migration
# DEFAULT: $ExcludeTables = $null => exclude no tables from migration
# Exclude list has higher priority than include list
# Array of qualified source table names which should be migrated
[string[]] $IncludeTables = @("migsourcedb.coupons", "migsourcedb.daily_cash_sheets");
[string[]] $ExcludeTables = $null;

LogMessage -Message "Creating the table map based on the user input and database table information ..." `
    -IsProcessing $true

$targetTables = $getTargetTablesTask.Output.DatabasesToTables."$TargetDatabaseName";
$sourceTables = $getSourceTablesTask.Output.DatabasesToTables."$SourceDatabaseName";
$tableMap = New-Object 'system.collections.generic.dictionary[string,string]';

$schemaPrefixLength = $($SourceDatabaseName + ".").Length;
$tableMappingError = $false
foreach ($srcTable in $sourceTables) {
    # Removing the database name prefix from the table name so that comparison
    # can be done in cases where database name given are different
    $tableName = $srcTable.Name.Substring($schemaPrefixLength, `
            $srcTable.Name.Length - $schemaPrefixLength)

    # In case the table is part of exclusion list then ignore the table
    if ($null -ne $ExcludeTables -and $ExcludeTables -contains $srcTable.Name) {
        continue;
    }

    # Either the include list is null or the table is part of the include list then add it in the mapping
    if ($null -eq $IncludeTables -or $IncludeTables -contains $srcTable.Name) {
        # Check if the table exists in the target. If not then log TABLE MAPPING ERROR
        if (-not ($targetTables | Where-Object { $_.name -ieq "$($TargetDatabaseName).$tableName" })) {
            $tableMappingError = $true
            Write-Host "TABLE MAPPING ERROR: $($targetTables.name) does not exists in target." -ForegroundColor Red
            continue;
        }  

        $tableMap.Add("$($SourceDatabaseName).$tableName", "$($TargetDatabaseName).$tableName");
    }     
}

# In case of any table mapping errors identified, throw an error and stop the process
if ($tableMappingError) { throw "ERROR: One or more table mapping errors were identified. Please see previous messages." }
# In case no tables are in the mapping then throw error
if ($tableMap.Count -le 0) { throw "ERROR: Could not create table mapping." }
LogMessage -Message "Migration table mapping created for $($tableMap.Count) tables."
```

## <a name="create-and-configure-the-migration-task-inputs"></a>Creación y configuración de las entradas de la tarea de migración

Después de crear la asignación de tablas, creará las entradas de la tarea de migración de tipo *Migrate.MySql.AzureDbForMySql* y configurará las propiedades.

El siguiente script crea la tarea de migración y establece las conexiones, los nombres de base de datos y la asignación de tablas.

```powershell
# Create and configure the migration scenario based on the connections
# and the table mapping
$offlineMigTaskProperties = @{
    "input"    = @{
        "sourceConnectionInfo"  = $null;
        "targetConnectionInfo"  = $null;
        "selectedDatabases"     = $null;
        "optionalAgentSettings" = @{
            "EnableCacheBatchesInMemory"         = $true;
            "DisableIncrementalRowStatusUpdates" = $true;
        };
        "startedOn"             = $null;
    };
    "taskType" = "Migrate.MySql.AzureDbForMySql";
};
$offlineSelectedDatabase = @{
    "name"               = $null;
    "targetDatabaseName" = $null;
    "tableMap"           = $null;
};

LogMessage -Message "Preparing migration scenario configuration ..." -IsProcessing $true

# Select the database to be migrated
$offlineSelectedDatabase.name = $SourceDatabaseName;
$offlineSelectedDatabase.tableMap = New-Object PSObject -Property $tableMap;
$offlineSelectedDatabase.targetDatabaseName = $TargetDatabaseName;

# Set connection info and the database mapping
$offlineMigTaskProperties.input.sourceConnectionInfo = $sourceConnInfo;
$offlineMigTaskProperties.input.targetConnectionInfo = $targetConnInfo;
$offlineMigTaskProperties.input.selectedDatabases = @($offlineSelectedDatabase);
$offlineMigTaskProperties.input.startedOn = [System.DateTimeOffset]::UtcNow.ToString("O");
```

## <a name="configure-performance-tuning-parameters"></a>Configuración de los parámetros de ajuste del rendimiento

Como parte del módulo de PowerShell, hay algunos parámetros opcionales disponibles que se pueden ajustar en función del entorno. Estos parámetros se pueden usar para mejorar el rendimiento de la tarea de migración. Todos estos parámetros son opcionales y su valor predeterminado es NULL.

> [!NOTE]
> Las siguientes configuraciones de rendimiento han mostrado un mayor rendimiento durante la migración en una SKU Premium.
> * WriteDataRangeBatchTaskCount = 12
> * DelayProgressUpdatesInStorageInterval = 30 segundos
> * ThrottleQueryTableDataRangeTaskAtBatchCount = 36

El siguiente script toma los valores de usuario de los parámetros y establece los parámetros en las propiedades de la tarea de migración.

```powershell
# Setting optional parameters from fine tuning the data transfer rate during migration
# DEFAULT values for all the configurations is $null
LogMessage -Message "Adding optional migration performance tuning configuration ..." -IsProcessing $true
# Partitioning settings
# Optional setting that configures the maximum number of parallel reads on tables located on the source database.
[object] $DesiredRangesCount = 4
# Optional setting that configures that size of the largest batch that will be committed to the target server.
[object] $MaxBatchSizeKb = 4096
# Optional setting that configures the minimum number of rows in each batch written to the target.
[object] $MinBatchRows = $null
# Task count settings
# Optional setting that configures the number of databases that will be prepared for migration in parallel.
[object] $PrepareDatabaseForBulkImportTaskCount = $null
# Optional setting that configures the number of tables that will be prepared for migration in parallel.
[object] $PrepareTableForBulkImportTaskCount = $null
# Optional setting that configures the number of threads available to read ranges on the source.
[object] $QueryTableDataRangeTaskCount = 8
# Optional setting that configures the number of threads available to write batches to the target.
[object] $WriteDataRangeBatchTaskCount = 12
# Batch cache settings
# Optional setting that configures how much memory will be used to cache batches in memory before reads on the source are throttled.
[object] $MaxBatchCacheSizeMb = $null
# Optional setting that configures the amount of available memory at which point reads on the source will be throttled.
[object] $ThrottleQueryTableDataRangeTaskAtAvailableMemoryMb = $null
# Optional setting that configures the number of batches cached in memory that will trigger read throttling on the source.
[object] $ThrottleQueryTableDataRangeTaskAtBatchCount = 36
# Performance settings
# Optional setting that configures the delay between updates of result objects in Azure Table Storage.
[object] $DelayProgressUpdatesInStorageInterval = "00:00:30"

function AddOptionalSetting($optionalAgentSettings, $settingName, $settingValue) {
    # If no value specified for the setting, don't bother adding it to the input
    if ($null -eq $settingValue) {
        return;
    }

    # Add a new property to the JSON object to capture the setting which will be customized
    $optionalAgentSettings | add-member -MemberType NoteProperty -Name $settingName -Value $settingValue
}

# Set any optional settings in the input based on parameters to this cmdlet
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "DesiredRangesCount" $DesiredRangesCount;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "MaxBatchSizeKb" $MaxBatchSizeKb;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "MinBatchRows" $MinBatchRows;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "PrepareDatabaseForBulkImportTaskCount" $PrepareDatabaseForBulkImportTaskCount;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "PrepareTableForBulkImportTaskCount" $PrepareTableForBulkImportTaskCount;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "QueryTableDataRangeTaskCount" $QueryTableDataRangeTaskCount;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "WriteDataRangeBatchTaskCount" $WriteDataRangeBatchTaskCount;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "MaxBatchCacheSizeMb" $MaxBatchCacheSizeMb;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "ThrottleQueryTableDataRangeTaskAtAvailableMemoryMb" $ThrottleQueryTableDataRangeTaskAtAvailableMemoryMb;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "ThrottleQueryTableDataRangeTaskAtBatchCount" $ThrottleQueryTableDataRangeTaskAtBatchCount;
AddOptionalSetting $offlineMigTaskProperties.input.optionalAgentSettings "DelayProgressUpdatesInStorageInterval" $DelayProgressUpdatesInStorageInterval;
```

## <a name="creating-and-running-the-migration-task"></a>Creación y ejecución de la tarea de migración

Después de configurar la entrada de la tarea, ahora la tarea se creará y ejecutará en el agente. El script desencadena la ejecución de la tarea y espera a que se complete la migración.

El siguiente script invoca la tarea de migración configurada y espera a que se complete.

```powershell
# Running the migration scenario
[string] $TaskName = "mysqlofflinemigrate"

LogMessage -Message "Running data migration scenario ..." -IsProcessing $true
$summary = @{
    "SourceServer"   = $SourceServerName;
    "SourceDatabase" = $SourceDatabaseName;
    "TargetServer"   = $TargetServerName;
    "TargetDatabase" = $TargetDatabaseName;
    "TableCount"     = $tableMap.Count;
    "StartedOn"      = $offlineMigTaskProperties.input.startedOn;
}

Write-Host "Job Summary:" -ForegroundColor Yellow
Write-Host $(ConvertTo-Json $summary) -ForegroundColor Yellow

$migrationResult = RunScenario -MigrationService $dmsService `
    -MigrationProject $dmsProject `
    -ScenarioTaskName $TaskName `
    -TaskProperties $offlineMigTaskProperties

LogMessage -Message "Migration completed with status - $($migrationResult.state)"
#Checking for any errors or warnings captured by the task during migration
$dbLevelResult = $migrationResult.output | Where-Object { $_.resultType -eq "DatabaseLevelOutput" } 
$migrationLevelResult = $migrationResult.output | Where-Object { $_.resultType -eq "MigrationLevelOutput" }
if ($dbLevelResult.exceptionsAndWarnings) {
    Write-Host "Following database errors were captured: $($dbLevelResult.exceptionsAndWarnings)" -ForegroundColor Red
}
if ($migrationLevelResult.exceptionsAndWarnings) {
    Write-Host "Following migration errors were captured: $($migrationLevelResult.exceptionsAndWarnings)" -ForegroundColor Red
}
if ($migrationResult.errors.details) {
    Write-Host "Following task level migration errors were captured: $($migrationResult.errors.details)" -ForegroundColor Red
}
```

## <a name="deleting-the-database-migration-service"></a>Eliminación de la instancia de Database Migration Service

Se puede usar la misma instancia de Database Migration Service para varias migraciones, por lo que se puede volver a usar la instancia una vez creada. Si no va a seguir usando la instancia de Database Migration Service, puede eliminar el servicio mediante el comando [Remove-AzDataMigrationService](/powershell/module/az.datamigration/remove-azdatamigrationservice?).

El siguiente script elimina la instancia de Azure Database Migration Service y sus proyectos asociados.

```powershell
Remove-AzDataMigrationService -ResourceId $($dmsService.ResourceId)
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre los problemas conocidos y las limitaciones al llevar a cabo migraciones mediante DMS, consulte el artículo [Solucionar problemas y errores comunes de Azure Database Migration Service](./known-issues-troubleshooting-dms.md).
* Para solucionar problemas de conectividad de la base de datos de origen al usar DMS, consulte el artículo [Solución de errores de DMS al conectarse a las bases de datos de origen](./known-issues-troubleshooting-dms-source-connectivity.md).
* Para más información acerca de Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](./dms-overview.md)
* Para información sobre Azure Database for MySQL, consulte el artículo [¿Qué es Azure Database for MySQL?](../mysql/overview.md)
* Para ver un tutorial sobre el uso de DMS mediante el portal, consulte el artículo [Tutorial: Migración de MySQL a Azure Database for MySQL sin conexión mediante DMS](./tutorial-mysql-azure-mysql-offline-portal.md).