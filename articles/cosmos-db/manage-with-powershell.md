---
title: Administración de recursos de API de Azure Cosmos DB Core (SQL) mediante PowerShell
description: Administre recursos de API de Azure Cosmos DB Core (SQL) mediante PowerShell.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/25/2021
ms.author: mjbrown
ms.custom: seodec18
ms.openlocfilehash: f967c5b40facbcbfe21ae87ae962241632135c34
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228212"
---
# <a name="manage-azure-cosmos-db-core-sql-api-resources-using-powershell"></a>Administración de recursos de API de Azure Cosmos DB Core (SQL) mediante PowerShell
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En la guía siguiente se describe cómo usar PowerShell para crear scripts y automatizar la administración de los recursos de API de Azure Cosmos DB Core (SQL), incluida la cuenta, la base de datos, el contenedor y el rendimiento de Cosmos. Para obtener cmdlets de PowerShell para otras API, consulte [Ejemplos de PowerShell para Cassandra](powershell-samples-cassandra.md), [Ejemplos de PowerShell para MongoDB API](powershell-samples-mongodb.md), [Ejemplos de PowerShell para Gremlin](powershell-samples-gremlin.md) y [Ejemplos de PowerShell para Table](powershell-samples-table.md).

> [!NOTE]
> En los ejemplos de este artículo, se usan los cmdlets de administración de [Az.CosmosDB](/powershell/module/az.cosmosdb). Consulte la página de referencia de la API de [Az.CosmosDB](/powershell/module/az.cosmosdb) para conocer los cambios más recientes.

Para la administración multiplataforma de Azure Cosmos DB, puede usar los cmdlets `Az` y `Az.CosmosDB` con [versiones multiplataforma de PowerShell](/powershell/scripting/install/installing-powershell), así como la [CLI de Azure](manage-with-cli.md), la [API REST][rp-rest-api] o [Azure Portal](create-sql-api-dotnet.md#create-account).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="getting-started"></a>Introducción

Siga las instrucciones de [Instalación y configuración de Azure PowerShell][powershell-install-configure] para instalar e iniciar sesión en la cuenta de Azure en PowerShell.

> [!IMPORTANT]
> No se puede cambiar el nombre de los recursos de Azure Cosmos DB, ya que esto infringe la forma de funcionar de Azure Resource Manager con los URI de recursos.

## <a name="azure-cosmos-accounts"></a>Cuentas de Azure Cosmos

En las secciones siguientes se muestra cómo administrar la cuenta de Azure Cosmos, entre ellas:

* [Creación de una cuenta de Azure Cosmos](#create-account)
* [Actualización de una cuenta de Azure Cosmos](#update-account)
* [Enumeración de todas las cuentas de Azure Cosmos DB de una suscripción](#list-accounts)
* [Obtención de una cuenta de Azure Cosmos](#get-account)
* [Eliminación de una cuenta de Azure Cosmos](#delete-account)
* [Actualización de las etiquetas de una cuenta de Azure Cosmos](#update-tags)
* [Enumeración de las claves para una cuenta de Azure Cosmos](#list-keys)
* [Regeneración de las claves para una cuenta de Azure Cosmos](#regenerate-keys)
* [Enumeración de las cadenas de conexión para una cuenta de Azure Cosmos](#list-connection-strings)
* [Modificación de la prioridad de conmutación por error de una cuenta de Azure Cosmos](#modify-failover-priority)
* [Desencadenamiento de una conmutación por error manual en una cuenta de Azure Cosmos](#trigger-manual-failover)
* [Enumerar los bloqueos de recursos en una cuenta de Azure Cosmos DB](#list-account-locks)

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a> Creación de una cuenta de Azure Cosmos

Este comando crea una cuenta de base de datos de Azure Cosmos DB con una [directiva de coherencia][distribute-data-globally] en [varias regiones](how-to-manage-database-account.md#automatic-failover), con [conmutación automática por error](consistency-levels.md) y obsolescencia limitada.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "BoundedStaleness"
$maxStalenessInterval = 300
$maxStalenessPrefix = 100000
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0

New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -LocationObject $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$true `
    -DefaultConsistencyLevel $consistencyLevel `
    -MaxStalenessIntervalInSeconds $maxStalenessInterval `
    -MaxStalenessPrefix $maxStalenessPrefix
```

* `$resourceGroupName` El grupo de recursos de Azure en el que se implementa la cuenta de Cosmos. Debe existir.
* `$locations` Las regiones de la cuenta de base de datos, la región con `FailoverPriority 0` es la región de escritura.
* `$accountName` El nombre de la cuenta de Azure Cosmos. Debe ser único, en minúsculas, incluir solo caracteres alfanuméricos y "-", y tener una longitud entre 3 y 31 caracteres.
* `$apiKind` El tipo de cuenta de Cosmos que se crea. Para más información, consulte [API en Cosmos DB](introduction.md#simplified-application-development).
* `$consistencyPolicy`, `$maxStalenessInterval` y `$maxStalenessPrefix` El nivel de coherencia predeterminado y la configuración de la cuenta de Azure Cosmos. Para más información, consulte [Niveles de coherencia en Azure Cosmos DB](consistency-levels.md).

Las cuentas de Azure Cosmos se pueden configurar con firewall de IP, puntos de conexión de servicio de red virtual y puntos de conexión privados. Para más información sobre cómo configurar el firewall de IP para Azure Cosmos DB, consulte [Configuración del firewall de IP](how-to-configure-firewall.md). Para más información sobre cómo habilitar los puntos de conexión de servicio para Azure Cosmos DB, consulte [Configuración del acceso desde redes virtuales](how-to-configure-vnet-service-endpoint.md). Para más información sobre cómo habilitar los puntos de conexión privados para Azure Cosmos DB, consulte [Configuración del acceso desde puntos de conexión privados](how-to-configure-private-endpoints.md).

### <a name="list-all-azure-cosmos-accounts-in-a-resource-group"></a><a id="list-accounts"></a> Enumeración de todas las cuentas de Azure Cosmos de un grupo de recursos

Este comando enumera todas las cuentas de Azure Cosmos de un grupo de recursos.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName
```

### <a name="get-the-properties-of-an-azure-cosmos-account"></a><a id="get-account"></a> Obtención de las propiedades de una cuenta de Azure Cosmos

Este comando le permite obtener las propiedades de una cuenta de base de datos de Azure Cosmos.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName
```

### <a name="update-an-azure-cosmos-account"></a><a id="update-account"></a> Actualización de una cuenta de Azure Cosmos

Este comando permite actualizar las propiedades de la cuenta de base de datos de Azure Cosmos DB. Las propiedades que se pueden actualizar incluyen las siguientes:

* Adición o eliminación de registros
* Cambio de la directiva de coherencia predeterminada
* Cambio del filtro del intervalo de direcciones IP
* Cambio de las configuraciones de la red virtual
* Habilitación de escrituras en varias regiones

> [!NOTE]
> En una cuenta de Azure Cosmos no es posible agregar o quitar regiones (`locations`) y, al mismo tiempo, cambiar otras propiedades. La modificación de regiones debe realizarse como una operación independiente de cualquier otro cambio en la cuenta.
> [!NOTE]
> Este comando permite agregar y quitar regiones, pero no permite modificar las prioridades de conmutación por error ni desencadenar una conmutación por error manual. Consulte [Modificación de la prioridad de conmutación por error](#modify-failover-priority) y [Desencadenamiento de una conmutación por error manual](#trigger-manual-failover).
> [!TIP]
> Cuando se agrega una nueva región, todos los datos deben replicarse por completo y estar confirmados en la nueva región antes de que la región se marque como disponible. La cantidad de tiempo que tarde esta operación dependerá de la cantidad de datos almacenados en la cuenta.

```azurepowershell-interactive
# Create account with two regions
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "Session"
$enableAutomaticFailover = $true
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0

# Create the Cosmos DB account
New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -LocationObject $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$enableAutomaticFailover `
    -DefaultConsistencyLevel $consistencyLevel

# Add a region to the account
$locationObject2 = @()
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "South Central US" -FailoverPriority 2 -IsZoneRedundant 0

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObject2

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
Write-Host "When region was added, press any key to continue."
$HOST.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown") | OUT-NULL
$HOST.UI.RawUI.Flushinputbuffer()

# Remove West US region from the account
$locationObject3 = @()
$locationObject3 += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locationObject3 += New-AzCosmosDBLocationObject -LocationName "South Central US" -FailoverPriority 1 -IsZoneRedundant 0

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObject3

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
```

### <a name="enable-multiple-write-regions-for-an-azure-cosmos-account"></a><a id="multi-region-writes"></a> Habilitación de varias regiones de escritura para una cuenta de Azure Cosmos

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $false
$enableMultiMaster = $true

# First disable automatic failover - cannot have both automatic
# failover and multi-region writes on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover

# Now enable multi-region writes
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster
```

### <a name="delete-an-azure-cosmos-account"></a><a id="delete-account"></a> Eliminación de una cuenta de Azure Cosmos

Este comando elimina una cuenta de Azure Cosmos existente.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Remove-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -PassThru:$true
```

### <a name="update-tags-of-an-azure-cosmos-account"></a><a id="update-tags"></a> Actualización de las etiquetas de una cuenta de Azure Cosmos

Este comando establece las [etiquetas de recursos de Azure][azure-resource-tags] para una cuenta de Azure Cosmos. Las etiquetas se pueden establecer tanto en la creación de cuentas mediante `New-AzCosmosDBAccount`, como en la actualización de cuentas mediante `Update-AzCosmosDBAccount`.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$tags = @{dept = "Finance"; environment = "Production";}

Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Tag $tags
```

### <a name="list-account-keys"></a><a id="list-keys"></a> Enumerar claves de cuenta

Cuando se crea una cuenta de Azure Cosmos, el servicio genera dos claves de acceso primarias que se pueden usar para la autenticación cuando se accede a dicha cuenta. También se generan claves de solo lectura para autenticar las operaciones de solo lectura.
Al proporcionar dos claves de acceso, Azure Cosmos DB permite regenerar y rotar las claves de una en una sin interrupción en la cuenta de Azure Cosmos.
Las cuentas de Cosmos DB tienen dos claves de lectura y escritura (principal y secundaria) y dos claves de solo lectura (principal y secundaria).

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "Keys"
```

### <a name="list-connection-strings"></a><a id="list-connection-strings"></a> Enumeración de cadenas de conexión

El comando siguiente recupera las cadenas de conexión para conectar las aplicaciones a la cuenta de Cosmos DB.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "ConnectionStrings"
```

### <a name="regenerate-account-keys"></a><a id="regenerate-keys"></a> Regeneración de claves de cuenta

Las claves de acceso a una cuenta de Azure Cosmos se deben regenerar periódicamente para ayudar a mantener las conexiones seguras. Una clave de acceso principal y otra secundaria se asignan a la cuenta. Esto permite a los clientes mantener el acceso mientras se regenera la otra.
Hay cuatro tipos de claves para una cuenta de Azure Cosmos (primaria, secundaria, primera de solo lectura y secundaria de solo lectura)

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "mycosmosaccount" # Must be all lower case
$keyKind = "primary" # Other key kinds: secondary, primaryReadOnly, secondaryReadOnly

New-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -KeyKind $keyKind
```

### <a name="enable-automatic-failover"></a><a id="enable-automatic-failover"></a> Habilitación de la conmutación automática por error

El siguiente comando establece que una cuenta de Cosmos DB realice la conmutación por error en su región secundaria si la región primaria deja de estar disponible.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $true
$enableMultiMaster = $false

# First disable multi-region writes - cannot have both automatic
# failover and multi-region writes on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster

# Now enable automatic failover
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover
```

### <a name="modify-failover-priority"></a><a id="modify-failover-priority"></a> Modificación de la prioridad de conmutación por error

En el caso de las cuentas configuradas con conmutación automática por error, puede cambiar el orden en el que Cosmos promoverá las réplicas secundarias a principales si las principales dejan de estar disponibles.

Para el ejemplo siguiente, asuma que la prioridad de la conmutación por error actual es `West US 2 = 0`, `East US 2 = 1` y `South Central US = 2`. El comando cambiará esto a `West US 2 = 0`, `South Central US = 1` y `East US 2 = 2`.

> [!CAUTION]
> El cambio de la ubicación de `failoverPriority=0` desencadenará una conmutación por error manual en una cuenta de Azure Cosmos. Cualquier otro cambio de prioridad no desencadenará ninguna conmutación por error.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("West US 2", "South Central US", "East US 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

### <a name="trigger-manual-failover"></a><a id="trigger-manual-failover"></a>Desencadenamiento de una conmutación por error manual

En el caso de las cuentas configuradas con conmutación por error manual, puede realizar la conmutación y promocionar cualquier réplica secundaria a principal mediante la modificación de `failoverPriority=0`. Esta operación se puede usar para iniciar una exploración de recuperación ante desastres para probar la planeación de esta última.

En el ejemplo siguiente, suponga que la cuenta tiene una prioridad de conmutación por error actual de `West US 2 = 0` y `East US 2 = 1`, y que se invierten las regiones.

> [!CAUTION]
> Cambiar `locationName` por `failoverPriority=0` desencadenará una conmutación por error manual de una cuenta de Azure cosmos. Los restantes cambios de prioridad no desencadenarán ninguna conmutación por error.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("East US 2", "West US 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

### <a name="list-resource-locks-on-an-azure-cosmos-db-account"></a><a id="list-account-locks"></a> Enumerar los bloqueos de recursos en una cuenta de Azure Cosmos DB

Se pueden establecer bloqueos de recursos en recursos de Azure Cosmos DB, incluidas bases de datos y colecciones. En el ejemplo siguiente se muestra cómo enumerar todos los bloqueos de recursos de Azure en una cuenta de Azure Cosmos DB.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceTypeAccount = "Microsoft.DocumentDB/databaseAccounts"
$accountName = "mycosmosaccount"

Get-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeAccount `
    -ResourceName $accountName
```

## <a name="azure-cosmos-db-database"></a>Base de datos de Azure Cosmos DB

En las siguientes secciones se muestra cómo se administra la base de datos de Azure Cosmos DB, lo que incluye:

* [Creación de una base de datos de Azure Cosmos DB](#create-db)
* [Creación de una base de datos de Azure Cosmos DB con capacidad de proceso compartido](#create-db-ru)
* [Obtención de la capacidad de proceso de una base de datos de Azure Cosmos DB](#get-db-ru)
* [Migración del rendimiento de base de datos a escalabilidad automática](#migrate-db-ru)
* [Enumeración de todas las bases de datos de Azure Cosmos DB en una cuenta](#list-db)
* [Obtención de una sola base de datos de Azure Cosmos DB](#get-db)
* [Eliminación de una base de datos de Azure Cosmos DB](#delete-db)
* [Creación de un bloqueo de recursos en una base de datos de Azure Cosmos DB para evitar la eliminación](#create-db-lock)
* [Eliminación de un bloqueo de recursos en una base de datos de Azure Cosmos DB](#remove-db-lock)

### <a name="create-an-azure-cosmos-db-database"></a><a id="create-db">Creación de una base de datos de Azure Cosmos DB</a>

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="create-an-azure-cosmos-db-database-with-shared-throughput"></a><a id="create-db-ru"></a>Creación de una base de datos de Azure Cosmos DB con capacidad de proceso compartido

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$databaseRUs = 400

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -Throughput $databaseRUs
```

### <a name="get-the-throughput-of-an-azure-cosmos-db-database"></a><a id="get-db-ru"></a>Obtención de la capacidad de proceso de una base de datos de Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabaseThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

## <a name="migrate-database-throughput-to-autoscale"></a><a id="migrate-db-ru"></a>Migración del rendimiento de base de datos a escalabilidad automática

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Invoke-AzCosmosDBSqlDatabaseThroughputMigration `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -ThroughputType Autoscale
```

### <a name="get-all-azure-cosmos-db-databases-in-an-account"></a><a id="list-db"></a>Obtención de todas las bases de datos de Azure Cosmos DB de una cuenta

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName
```

### <a name="get-a-single-azure-cosmos-db-database"></a><a id="get-db"></a>Obtención de una sola base de datos de Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="delete-an-azure-cosmos-db-database"></a><a id="delete-db"></a>Eliminación de una base de datos de Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Remove-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="create-a-resource-lock-on-an-azure-cosmos-db-database-to-prevent-delete"></a><a id="create-db-lock"></a>Creación de un bloqueo de recursos en una base de datos de Azure Cosmos DB para evitar la eliminación

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$resourceName = "$accountName/$databaseName"
$lockName = "myResourceLock"
$lockLevel = "CanNotDelete"

New-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName `
    -LockLevel $lockLevel
```

### <a name="remove-a-resource-lock-on-an-azure-cosmos-db-database"></a><a id="remove-db-lock"></a>Eliminación de un bloqueo de recursos en una base de datos de Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$resourceName = "$accountName/$databaseName"
$lockName = "myResourceLock"

Remove-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName
```

## <a name="azure-cosmos-db-container"></a>Contenedor de Azure Cosmos DB

En las siguientes secciones se muestra cómo administrar el contenedor de Azure Cosmos DB, lo que incluye:

* [Creación de un contenedor de Azure Cosmos DB](#create-container)
* [Creación de un contenedor de Azure Cosmos DB con escalabilidad automática](#create-container-autoscale)
* [Creación de un contenedor de Azure Cosmos DB con una clave de partición grande](#create-container-big-pk)
* [Obtención de la capacidad de proceso de un contenedor de Azure Cosmos DB](#get-container-ru)
* [Migración del rendimiento de contenedor a escalabilidad automática](#migrate-container-ru)
* [Creación de un contenedor de Azure Cosmos DB con indexación personalizada](#create-container-custom-index)
* [Creación de un contenedor de Azure Cosmos DB con la indexación desactivada](#create-container-no-index)
* [Creación de un contenedor de Azure Cosmos DB con una clave única y TTL](#create-container-unique-key-ttl)
* [Creación de un contenedor de Azure Cosmos DB con resolución de conflictos](#create-container-lww)
* [Enumeración de todos los contenedores de Azure Cosmos DB de una base de datos](#list-containers)
* [Obtención de un contenedor individual de Azure Cosmos DB en una base de datos](#get-container)
* [Eliminación de un contenedor de Azure Cosmos DB](#delete-container)
* [Creación de un bloqueo de recursos en un contenedor de Azure Cosmos DB para evitar la eliminación](#create-container-lock)
* [Eliminación de un bloqueo de recursos en un contenedor de Azure Cosmos DB](#remove-container-lock)

### <a name="create-an-azure-cosmos-db-container"></a><a id="create-container"></a>Creación de un contenedor de Azure Cosmos DB

```azurepowershell-interactive
# Create an Azure Cosmos DB container with default indexes and throughput at 400 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$throughput = 400 #minimum = 400

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -Throughput $throughput
```

### <a name="create-an-azure-cosmos-db-container-with-autoscale"></a><a id="create-container-autoscale"></a>Creación de un contenedor de Azure Cosmos DB con escalabilidad automática

```azurepowershell-interactive
# Create an Azure Cosmos DB container with default indexes and autoscale throughput at 4000 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$autoscaleMaxThroughput = 4000 #minimum = 4000

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -AutoscaleMaxThroughput $autoscaleMaxThroughput
```

### <a name="create-an-azure-cosmos-db-container-with-a-large-partition-key-size"></a><a id="create-container-big-pk"></a>Creación de un contenedor de Azure Cosmos DB con una clave de partición grande

```azurepowershell-interactive
# Create an Azure Cosmos DB container with a large partition key value (version = 2)
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -PartitionKeyVersion 2
```

### <a name="get-the-throughput-of-an-azure-cosmos-db-container"></a><a id="get-container-ru"></a>Obtención de la capacidad de proceso de un contenedor de Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainerThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

### <a name="migrate-container-throughput-to-autoscale"></a><a id="migrate-container-ru"></a>Migración del rendimiento de contenedor a escalabilidad automática

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Invoke-AzCosmosDBSqlContainerThroughputMigration `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -ThroughputType Autoscale
```

### <a name="create-an-azure-cosmos-db-container-with-custom-index-policy"></a><a id="create-container-custom-index"></a>Creación de un contenedor de Azure Cosmos DB con una directiva de indexación personalizada

```azurepowershell-interactive
# Create a container with a custom indexing policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$indexPathIncluded = "/*"
$indexPathExcluded = "/myExcludedPath/*"

$includedPathIndex = New-AzCosmosDBSqlIncludedPathIndex -DataType String -Kind Range
$includedPath = New-AzCosmosDBSqlIncludedPath -Path $indexPathIncluded -Index $includedPathIndex

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IncludedPath $includedPath `
    -ExcludedPath $indexPathExcluded `
    -IndexingMode Consistent `
    -Automatic $true

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

### <a name="create-an-azure-cosmos-db-container-with-indexing-turned-off"></a><a id="create-container-no-index"></a>Creación de un contenedor de Azure Cosmos DB con la indexación desactivada

```azurepowershell-interactive
# Create an Azure Cosmos DB container with no indexing
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IndexingMode None

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

### <a name="create-an-azure-cosmos-db-container-with-unique-key-policy-and-ttl"></a><a id="create-container-unique-key-ttl"></a>Creación de un contenedor de Azure Cosmos DB con una directiva de clave única y TTL

```azurepowershell-interactive
# Create a container with a unique key policy and TTL of one day
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$uniqueKeyPath = "/myUniqueKeyPath"
$ttlInSeconds = 86400 # Set this to -1 (or don't use it at all) to never expire

$uniqueKey = New-AzCosmosDBSqlUniqueKey `
    -Path $uniqueKeyPath

$uniqueKeyPolicy = New-AzCosmosDBSqlUniqueKeyPolicy `
    -UniqueKey $uniqueKey

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -UniqueKeyPolicy $uniqueKeyPolicy `
    -TtlInSeconds $ttlInSeconds
```

### <a name="create-an-azure-cosmos-db-container-with-conflict-resolution"></a><a id="create-container-lww"></a>Creación de un contenedor de Azure Cosmos DB con resolución de conflictos

Para escribir todos los conflictos en ConflictsFeed y controlarlos de manera independiente, pase `-Type "Custom" -Path ""`.

```azurepowershell-interactive
# Create container with last-writer-wins conflict resolution policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionPath = "/myResolutionPath"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type LastWriterWins `
    -Path $conflictResolutionPath

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```

Para crear una directiva de resolución de conflictos para utilizar un procedimiento almacenado, llame a `New-AzCosmosDBSqlConflictResolutionPolicy` y pase los parámetros `-Type` y `-ConflictResolutionProcedure`.

```azurepowershell-interactive
# Create container with custom conflict resolution policy using a stored procedure
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionSprocName = "mysproc"

$conflictResolutionSproc = "/dbs/$databaseName/colls/$containerName/sprocs/$conflictResolutionSprocName"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type Custom `
    -ConflictResolutionProcedure $conflictResolutionSproc

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```


### <a name="list-all-azure-cosmos-db-containers-in-a-database"></a><a id="list-containers"></a>Enumeración de todos los contenedores de Azure Cosmos DB de una base de datos

```azurepowershell-interactive
# List all Azure Cosmos DB containers in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName
```

### <a name="get-a-single-azure-cosmos-db-container-in-a-database"></a><a id="get-container"></a>Obtención de un contenedor individual de Azure Cosmos DB en una base de datos

```azurepowershell-interactive
# Get a single Azure Cosmos DB container in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

### <a name="delete-an-azure-cosmos-db-container"></a><a id="delete-container"></a>Eliminación de un contenedor de Azure Cosmos DB

```azurepowershell-interactive
# Delete an Azure Cosmos DB container
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Remove-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```
### <a name="create-a-resource-lock-on-an-azure-cosmos-db-container-to-prevent-delete"></a><a id="create-container-lock"></a>Creación de un bloqueo de recursos en un contenedor de Azure Cosmos DB para evitar la eliminación

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$resourceName = "$accountName/$databaseName/$containerName"
$lockName = "myResourceLock"
$lockLevel = "CanNotDelete"

New-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName `
    -LockLevel $lockLevel
```

### <a name="remove-a-resource-lock-on-an-azure-cosmos-db-container"></a><a id="remove-container-lock"></a>Eliminación de un bloqueo de recursos en un contenedor de Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$resourceName = "$accountName/$databaseName/$containerName"
$lockName = "myResourceLock"

Remove-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName
```

## <a name="next-steps"></a>Pasos siguientes

* [Todos los ejemplos de PowerShell](powershell-samples.md)
* [Administración de la cuenta de Azure Cosmos](how-to-manage-database-account.md)
* [Creación de un contenedor de Azure Cosmos DB](how-to-create-container.md)
* [Configuración del período de vida en Azure Cosmos DB](how-to-time-to-live.md)

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[powershell-install-configure]: /powershell/azure/
[scaling-globally]: distribute-data-globally.md#EnableGlobalDistribution
[distribute-data-globally]: distribute-data-globally.md
[azure-resource-groups]: ../azure-resource-manager/management/overview.md#resource-groups
[azure-resource-tags]: ../azure-resource-manager/management/tag-resources.md
[rp-rest-api]: /rest/api/cosmos-db-resource-provider/