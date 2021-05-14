---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 03/27/2019
ms.author: tamram
ms.openlocfilehash: 9a60c624b181a1efd2f6deebd349daa82214a8a4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185990"
---
<!--created by Robin Shahan to go in the articles for table storage w/powershell.
    There is one for Azure Table Storage and one for Azure Cosmos DB Table API -->

## <a name="managing-table-entities"></a>Administrar entidades de tabla

Ya tenemos la tabla. Pasemos a ver ahora cómo administrar las entidades (o filas) de esa tabla. 

Las entidades pueden tener hasta 255 propiedades, incluidas tres propiedades del sistema: **PartitionKey**, **RowKey** y **Timestamp**. Usted será responsable de insertar y actualizar los valores de **PartitionKey** y **RowKey**. El servidor se encargará de administrar el valor **Timestamp**, así que no podrá modificarlo. Tanto **PartitionKey** como **RowKey** identifican de forma exclusiva todas las entidades de una tabla.

* **PartitionKey**: determina la partición en la que se almacena la entidad.
* **RowKey**: identifica de forma única la entidad dentro de la partición.

Puede definir hasta 252 propiedades personalizadas para una entidad. 

### <a name="add-table-entities"></a>Agregar entidades de tabla

Agregue entidades a una tabla con **Add-AzTableRow**. En estos ejemplos se usan claves de partición con los valores `partition1` y `partition2`, y las claves de fila equivalen a abreviaturas de estados americanos. Las propiedades de cada entidad son `username` y `userid`. 

```powershell
$partitionKey1 = "partition1"
$partitionKey2 = "partition2"

# add four rows 
Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey1 `
    -rowKey ("CA") -property @{"username"="Chris";"userid"=1}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey2 `
    -rowKey ("NM") -property @{"username"="Jessie";"userid"=2}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey1 `
    -rowKey ("WA") -property @{"username"="Christine";"userid"=3}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey2 `
    -rowKey ("TX") -property @{"username"="Steven";"userid"=4}
```

### <a name="query-the-table-entities"></a>Consultar las entidades de tabla

Puede consultar las entidades de una tabla mediante el comando **Get AzTableRow**.

> [!NOTE]
> Los cmdlets **Get-AzureStorageTableRowAll**, **Get-AzureStorageTableRowByPartitionKey**, **Get AzureStorageTableRowByColumnName** y **Get-AzureStorageTableRowByCustomFilter** están en desuso y se eliminarán en una actualización de una versión futura.

#### <a name="retrieve-all-entities"></a>Recuperar todas las entidades

```powershell
Get-AzTableRow -table $cloudTable | ft
```

Este comando devuelve un valor similar a la siguiente tabla:

| userid | username | partición | clave de fila |
|----|---------|---------------|----|
| 1 | Chris | partition1 | CA |
| 3 | Christine | partition1 | WA |
| 2 | Jessie | partition2 | NM |
| 4 | Steven | partition2 | TX |

#### <a name="retrieve-entities-for-a-specific-partition"></a>Recuperar entidades de una partición específica

```powershell
Get-AzTableRow -table $cloudTable -partitionKey $partitionKey1 | ft
```

El resultado es similar a la siguiente tabla:

| userid | username | partición | clave de fila |
|----|---------|---------------|----|
| 1 | Chris | partition1 | CA |
| 3 | Christine | partition1 | WA |

#### <a name="retrieve-entities-for-a-specific-value-in-a-specific-column"></a>Recuperar entidades con un valor específico en una columna específica

```powershell
Get-AzTableRow -table $cloudTable `
    -columnName "username" `
    -value "Chris" `
    -operator Equal
```

Con esta consulta se recupera un único registro.

|campo|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | CA |

#### <a name="retrieve-entities-using-a-custom-filter"></a>Recuperar entidades con un filtro personalizado 

```powershell
Get-AzTableRow `
    -table $cloudTable `
    -customFilter "(userid eq 1)"
```

Con esta consulta se recupera un único registro.

|campo|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | CA |

### <a name="updating-entities"></a>Actualización de entidades 

La actualización de las entidades es un proceso de tres pasos: Primero se recupera la entidad que se va a cambiar, después se realiza el cambio y, finalmente, se confirma el cambio con **Update-AzTableRow**.

Actualice la entidad con el nombre de usuario = "Jessie" para que refleje el nombre de usuario = "Jessie2". En este ejemplo se indica también otra forma de crear un filtro personalizado con tipos de .NET.

```powershell
# Create a filter and get the entity to be updated.
[string]$filter = `
    [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("username",`
    [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Jessie")
$user = Get-AzTableRow `
    -table $cloudTable `
    -customFilter $filter

# Change the entity.
$user.username = "Jessie2"

# To commit the change, pipe the updated record into the update cmdlet.
$user | Update-AzTableRow -table $cloudTable

# To see the new record, query the table.
Get-AzTableRow -table $cloudTable `
    -customFilter "(username eq 'Jessie2')"
```

En los resultados aparece el registro Jessie2.

|campo|value|
|----|----|
| userid | 2 |
| username | Jessie2 |
| PartitionKey | partition2 |
| RowKey      | NM |

### <a name="deleting-table-entities"></a>Eliminar entidades de tabla

Se puede eliminar una o todas las entidades de la tabla.

#### <a name="deleting-one-entity"></a>Eliminar una entidad

Para eliminar solo una entidad, obtenga una referencia a esa entidad y canalícela en **Remove-AzTableRow**.

```powershell
# Set filter.
[string]$filter = `
  [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("username",`
  [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Jessie2")

# Retrieve entity to be deleted, then pipe it into the remove cmdlet.
$userToDelete = Get-AzTableRow `
    -table $cloudTable `
    -customFilter $filter
$userToDelete | Remove-AzTableRow -table $cloudTable

# Retrieve entities from table and see that Jessie2 has been deleted.
Get-AzTableRow -table $cloudTable | ft
```

#### <a name="delete-all-entities-in-the-table"></a>Eliminar todas las entidades de la tabla

Para eliminar todas las entidades de la tabla, recupérelas y canalice los resultados al cmdlet de eliminación. 

```powershell
# Get all rows and pipe the result into the remove cmdlet.
Get-AzTableRow `
    -table $cloudTable | Remove-AzTableRow -table $cloudTable 

# List entities in the table (there won't be any).
Get-AzTableRow -table $cloudTable | ft
```
