---
title: Consulta de bases de datos con particiones
description: Ejecute consultas a través de particiones con la biblioteca de cliente de bases de datos elásticas.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.topic: how-to
ms.custom: sqldbrb=1
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: 2b632e156d51b7e6985ab183bdadbb2da5da75b3
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699506"
---
# <a name="multi-shard-querying-using-elastic-database-tools"></a>Consulta de varias particiones mediante las herramientas de Elastic Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="overview"></a>Información general

Con las [herramientas de Elastic Database](elastic-scale-introduction.md), puede crear soluciones de base de datos particionadas. **Consultas a través de particiones múltiples** se usa para tareas como informes y recopilación de datos que requieren ejecutar una consulta que abarca varias particiones. (Compare esto con el [enrutamiento dependiente de los datos](elastic-scale-data-dependent-routing.md), que ejecuta todo el trabajo en una sola partición).

1. Obtenga una clase **RangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) o **ListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) mediante los métodos **TryGetRangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetrangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap)), **TryGetListShardMap** ([ Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetlistshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap)) o **GetShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap)). Consulte [Construcción de un ShardMapManager](elastic-scale-shard-map-management.md#constructing-a-shardmapmanager) y [Obtención de las clases RangeShardMap o ListShardMap](elastic-scale-shard-map-management.md#get-a-rangeshardmap-or-listshardmap).
2. Cree un objeto **MultiShardConnection** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardconnection), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardconnection)).
3. Cree un objeto **MultiShardStatement o MultiShardCommand** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
4. Establezca la **propiedad CommandText** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)) en un comando T-SQL.
5. Ejecute el comando mediante una llamada a los métodos **ExecuteQueryAsync o ExecuteReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement.executeQueryAsync), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
6. Para ver los resultados use las clases **MultiShardResultSet o MultiShardDataReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardresultset), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multisharddatareader)).

## <a name="example"></a>Ejemplo

El código siguiente ilustra el uso de consultas a través de particiones múltiples con un **ShardMap** llamado *myShardMap*.

```csharp
using (MultiShardConnection conn = new MultiShardConnection(myShardMap.GetShards(), myShardConnectionString))
{
    using (MultiShardCommand cmd = conn.CreateCommand())
    {
        cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable";
        cmd.CommandType = CommandType.Text;
        cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn;
        cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults;

        using (MultiShardDataReader sdr = cmd.ExecuteReader())
        {
            while (sdr.Read())
            {
                var c1Field = sdr.GetString(0);
                var c2Field = sdr.GetFieldValue<int>(1);
                var c3Field = sdr.GetFieldValue<Int64>(2);
            }
        }
    }
}
```

Una diferencia clave es la construcción de conexiones entre particiones múltiples. Donde **SqlConnection** opera en una base de datos individual, **MultiShardConnection** toma una **_colección de particiones_*como entrada. Complete la recopilación de particiones a partir de un mapa de particiones. Luego la consulta se ejecuta en la recopilación de particiones usando la semántica* UNION ALL** para ensamblar un solo resultado global. De manera opcional, el nombre de la partición donde se origina la fila se puede agregar a la salida usando la propiedad **ExecutionOptions** en el comando.

Observe la llamada a **myShardMap.GetShards()** . Este método recupera todas las particiones desde el mapa de particiones y brinda una manera fácil de ejecutar una consulta a través de todas las bases de datos importantes. La colección de particiones para una consulta a través de particiones múltiples se puede restringir aún más si se realiza una consulta LINQ sobre la colección devuelta desde la llamada a **myShardMap.GetShards()** . En combinación con la directiva de resultados parciales, la actual capacidad en las consultas a través de particiones múltiples se diseñó para funcionar bien con decenas, y hasta centenas, de particiones.

Una limitación con las consultas a través de particiones múltiples actualmente es la falta de validación de las particiones y de los shardlets que se consultan. Mientras que el enrutamiento dependiente de los datos comprueba que una partición determinada sea parte del mapa de particiones en el momento de la consulta, las consultas a través de particiones múltiples no realizan esta comprobación. Esto puede llevar a que las consultas a través de particiones múltiples se ejecuten en bases de datos que se han quitado del mapa de particiones.

## <a name="multi-shard-queries-and-split-merge-operations"></a>Consultas a través de particiones múltiples y operaciones de división y combinación

Las consultas a través de particiones múltiples no comprueban si los shardlets en la base de datos consultada forman parte de las operaciones de división y combinación en curso. (Consulte el artículo sobre [escalado con la herramienta de división y combinación de Elastic Database](elastic-scale-overview-split-and-merge.md)). Esto puede llevar a incoherencias donde las filas del mismo shardlet se muestran para bases de datos múltiples en la misma consulta a través de particiones múltiples. Tenga en cuenta estas limitaciones y considere la posibilidad de agotar las operaciones de división y combinación y los cambios en el mapa de particiones mientras se realizan consultas a través de particiones múltiples.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]