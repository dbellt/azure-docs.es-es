---
title: Uso de Azure PowerShell para configurar la copia de seguridad continua y la restauración a un momento dado en Azure Cosmos DB
description: Aprenda a aprovisionar una cuenta con datos de copia de seguridad continua y restauración mediante Azure PowerShell.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/01/2021
ms.author: govindk
ms.reviewer: sngun
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8f3a99e7dca81e669438498f78447eddfdddd583
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110662129"
---
# <a name="configure-and-manage-continuous-backup-and-point-in-time-restore-preview---using-azure-powershell"></a>Configuración y administración de la copia de seguridad continua y la restauración a un momento dado (versión preliminar) con Azure PowerShell
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> La característica de restauración a un momento dado (modo de copia de seguridad continua) de Azure Cosmos DB está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La característica de restauración a un momento dado (versión preliminar) de Azure Cosmos DB ayuda a recuperarse de un cambio accidental en un contenedor, a restaurar una cuenta, una base de datos o un contenedor eliminados o a realizar una restauración en cualquier región (donde existan copias de seguridad). El modo de copia de seguridad continua permite realizar la restauración en cualquier punto temporal de los últimos 30 días.

En este artículo se explica cómo aprovisionar una cuenta con datos de copia de seguridad continua y restauración mediante Azure PowerShell.

## <a name="install-azure-powershell"></a><a id="install-powershell"></a>Instalar Azure Powershell

1. Ejecute el siguiente comando desde Azure PowerShell para instalar el módulo de versión preliminar `Az.CosmosDB`, que contiene los comandos relacionados con la restauración a un momento dado:

   ```azurepowershell
   Install-Module -Name Az.CosmosDB -AllowPrerelease
   ```

1. Después de instalar los módulos, inicie sesión en Azure con

   ```azurepowershell
   Connect-AzAccount
   ```

1. Seleccione una suscripción concreta con el comando siguiente:

   ```azurepowershell
   Select-AzSubscription -Subscription <SubscriptionName>
   ```

## <a name="provision-a-sql-api-account-with-continuous-backup"></a><a id="provision-sql-api"></a>Aprovisionamiento de una cuenta de API de SQL con copia de seguridad continua

Para aprovisionar una cuenta con copia de seguridad continua, agregue un argumento `-BackupPolicyType Continuous` junto con el comando de aprovisionamiento normal.

El siguiente cmdlet es un ejemplo de una cuenta de escritura de una sola región `pitracct2` con una directiva de copia de seguridad continua creada en la región *Oeste de EE. UU.* en el grupo de recursos *myrg*:

```azurepowershell

New-AzCosmosDBAccount `
  -ResourceGroupName "myrg" `
  -Location "West US" `
  -BackupPolicyType Continuous `
  -Name "pitracct2" `
  -ApiKind "Sql"
      
```

## <a name="provision-a-mongodb-api-account-with-continuous-backup"></a><a id="provision-mongodb-api"></a>Aprovisionamiento de una cuenta de API de MongoDB con copia de seguridad continua

El siguiente cmdlet es un ejemplo de una cuenta de copia de seguridad continua *pitracct2* creada en la región *Oeste de EE. UU.* en el grupo de recursos *myrg*:

```azurepowershell

New-AzCosmosDBAccount `
  -ResourceGroupName "myrg" `
  -Location "West US" `
  -BackupPolicyType Continuous `
  -Name "pitracct3" `
  -ApiKind "MongoDB" `
  -ServerVersion "3.6"

```

## <a name="trigger-a-restore-operation"></a><a id="trigger-restore"></a>Desencadenamiento de una operación de restauración

El siguiente cmdlet es un ejemplo para desencadenar una operación de restauración con el comando restore mediante la cuenta de destino, la cuenta de origen, la ubicación, el grupo de recursos y la marca de tiempo:

```azurepowershell

Restore-AzCosmosDBAccount `
  -TargetResourceGroupName <resourceGroupName> `
  -TargetDatabaseAccountName <restored-account-name> `
  -SourceDatabaseAccountName <sourceDatabaseAccountName> `
  -RestoreTimestampInUtc <UTC time> `
  -Location <Azure Region Name>

```

**Ejemplo 1:** restauración de toda la cuenta:

```azurepowershell

Restore-AzCosmosDBAccount `
  -TargetResourceGroupName "rg" `
  -TargetDatabaseAccountName "pitrbb-ps-1" `
  -SourceDatabaseAccountName "source-sql" `
  -RestoreTimestampInUtc "2021-01-05T22:06:00" `
  -Location "West US"

```

**Ejemplo 2:** restauración de colecciones y bases de datos concretas. En este ejemplo se restauran las colecciones myCol1, myCol2 de myDB1 y toda la base de datos myDB2, que incluye todos los contenedores que hay en ella.

```azurepowershell
$datatabaseToRestore1 = New-AzCosmosDBDatabaseToRestore -DatabaseName "myDB1" -CollectionName "myCol1", "myCol2"
$datatabaseToRestore2 = New-AzCosmosDBDatabaseToRestore -DatabaseName "myDB2"

Restore-AzCosmosDBAccount `
  -TargetResourceGroupName "rg" `
  -TargetDatabaseAccountName "pitrbb-ps-1" `
  -SourceDatabaseAccountName "source-sql" `
  -RestoreTimestampInUtc "2021-01-05T22:06:00" `
  -DatabasesToRestore $datatabaseToRestore1, $datatabaseToRestore2 `
  -Location "West US"

```

## <a name="enumerate-restorable-resources-for-sql-api"></a><a id="enumerate-sql-api"></a>Enumeración de los recursos restaurables de la API de SQL

Los cmdlets de enumeración ayudan a detectar los recursos que están disponibles para restaurar en varias marcas de tiempo. Además, también proporcionan una fuente de eventos clave en los recursos restaurables de cuenta, base de datos y contenedor.

**Enumeración de todas las cuentas que se pueden restaurar en la suscripción actual**

Ejecute el comando `Get-AzCosmosDBRestorableDatabaseAccount` de PowerShell para enumerar todas las cuentas que se pueden restaurar en la suscripción actual.

La respuesta incluye todas las cuentas de base de datos (tanto activas como eliminadas) que se pueden restaurar y las regiones desde las que se puede hacer.

```console
{
    "accountName": "sampleaccount",
    "apiType": "Sql",
    "creationTime": "2020-08-08T01:04:52.070190+00:00",
    "deletionTime": null,
    "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.DocumentDB/locations/West US/restorableDatabaseAccounts/23e99a35-cd36-4df4-9614-f767a03b9995",
    "identity": null,
    "location": "West US",
    "name": "23e99a35-cd36-4df4-9614-f767a03b9995",
    "restorableLocations": [
      {
        "creationTime": "2020-08-08T01:04:52.945185+00:00",
        "deletionTime": null,
        "locationName": "West US",
        "regionalDatabaseAccountInstanceId": "30701557-ecf8-43ce-8810-2c8be01dccf9"
      },
      {
        "creationTime": "2020-08-08T01:15:43.507795+00:00",
        "deletionTime": null,
        "locationName": "East US",
        "regionalDatabaseAccountInstanceId": "8283b088-b67d-4975-bfbe-0705e3e7a599"
      }
    ],
    "tags": null,
    "type": "Microsoft.DocumentDB/locations/restorableDatabaseAccounts"
  },
```

Al igual que `CreationTime` o `DeletionTime` para la cuenta, también hay `CreationTime` o `DeletionTime` para la región. Estas horas permiten elegir la región correcta y un intervalo de tiempo válido para restaurar en esa región.

**Enumeración de todas las versiones de bases de datos SQL de una cuenta de base de datos activa**

La enumeración de todas las versiones de bases de datos permite elegir la base de datos correcta en un escenario en el que se desconoce la hora real de existencia de la base de datos.

Ejecute el siguiente comando de PowerShell para enumerar todas las versiones de bases de datos. Este comando solo funciona con cuentas activas. Los parámetros `DatabaseAccountInstanceId` y `LocationName` se obtienen de las propiedades `name` y `location` en la respuesta del cmdlet `Get-AzCosmosDBRestorableDatabaseAccount`. El atributo `DatabaseAccountInstanceId` hace referencia a la propiedad `instanceId` de la cuenta de base de datos de origen que se está restaurando:


```azurepowershell

Get-AzCosmosdbSqlRestorableDatabase `
  -LocationName "East US" `
  -DatabaseAccountInstanceId <DatabaseAccountInstanceId>

```

**Enumeración de todas las versiones de contenedores SQL de una base de datos de una cuenta de base de datos activa**

Use el siguiente comando para enumerar todas las versiones de contenedores SQL. Este comando solo funciona con cuentas activas. El parámetro `DatabaseRid` es el elemento `ResourceId` de la base de datos que se quiere restaurar. Es el valor del atributo `ownerResourceid` que se encuentra en la respuesta del cmdlet `Get-AzCosmosdbSqlRestorableDatabase`. La respuesta también incluye una lista de las operaciones realizadas en todos los contenedores de esta base de datos.

```azurepowershell

Get-AzCosmosdbSqlRestorableContainer `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -DatabaseRid "AoQ13r==" `
  -LocationName "West US"

```

**Búsqueda de bases de datos o contenedores que se pueden restaurar en cualquier marca de tiempo**

Use el comando siguiente para obtener la lista de bases de datos o contenedores que se pueden restaurar en una marca de tiempo determinada. Este comando solo funciona con cuentas activas.

```azurepowershell

Get-AzCosmosdbSqlRestorableResource `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -LocationName "West US" `
  -RestoreLocation "East US" `
  -RestoreTimestamp "2020-07-20T16:09:53+0000"

```

## <a name="enumerate-restorable-resources-for-mongodb"></a><a id="enumerate-mongodb-api"></a>Enumeración de los recursos restaurables de MongoDB

Los comandos de enumeración que se describen a continuación ayudan a detectar los recursos que están disponibles para restaurar en varias marcas de tiempo. Además, también proporcionan una fuente de eventos clave en los recursos restaurables de cuenta, base de datos y contenedor. Estos comandos solo funcionan en cuentas activas y son similares a los comandos de la API de SQL, aunque con `MongoDB` en el nombre de comando en lugar de `sql`.

**Enumeración de todas las versiones de bases de datos MongoDB de una cuenta de base de datos activa**

```azurepowershell

Get-AzCosmosdbMongoDBRestorableDatabase `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -LocationName "West US"

```

**Enumeración de todas las versiones de colecciones MongoDB de una base de datos de una cuenta de base de datos activa**

```azurepowershell

Get-AzCosmosdbMongoDBRestorableCollection `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -DatabaseRid "AoQ13r==" `
  -LocationName "West US"
```

**Enumeración de todos los recursos de una cuenta de base de datos MongoDB que están disponibles para restaurar en una determinada marca de tiempo y región**

```azurepowershell

Get-AzCosmosdbMongoDBRestorableResource `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -LocationName "West US" `
  -RestoreLocation "West US" `
  -RestoreTimestamp "2020-07-20T16:09:53+0000"
```

## <a name="next-steps"></a>Pasos siguientes

* Configure y administre la copia de seguridad continua mediante la [CLI de Azure](continuous-backup-restore-command-line.md), [Resource Manager](continuous-backup-restore-template.md) o [Azure Portal](continuous-backup-restore-portal.md).
* [Modelo de recursos del modo de copia de seguridad continua](continuous-backup-restore-resource-model.md)
* [Administre los permisos](continuous-backup-restore-permissions.md) necesarios para restaurar datos con el modo de copia de seguridad continua.
