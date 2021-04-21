---
title: Características y sintaxis que se admiten en la API de Azure Cosmos DB para la versión 4.0 de servidor de MongoDB
description: Obtenga más información sobre las características y sintaxis que se admiten en la API de Azure Cosmos DB para la versión 4.0 de servidor de MongoDB. Obtenga información sobre los comandos de base de datos, la compatibilidad con los lenguajes de consulta, los tipos de datos, los comandos de canalización de agregación y los operadores admitidos.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 03/02/2021
author: gahl-levy
ms.author: gahllevy
ms.openlocfilehash: 74e08cce381d173e0c5e1458ae99167a0c8c697f
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107504517"
---
# <a name="azure-cosmos-dbs-api-for-mongodb-40-server-version-supported-features-and-syntax"></a>API de Azure Cosmos DB para MongoDB (versión 4.0 de servidor): características y sintaxis que se admiten
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Azure Cosmos DB es un servicio de base de datos con varios modelos y de distribución global de Microsoft. Puede comunicarse con la API de Azure Cosmos DB para MongoDB mediante cualquiera de los [controladores](https://docs.mongodb.org/ecosystem/drivers) del cliente de MongoDB de código abierto. La API de Azure Cosmos DB para MongoDB permite usar los controladores de cliente existentes mediante la adhesión al [protocolo de conexión](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol) de MongoDB.

Con la API de Azure Cosmos DB para MongoDB, puede disfrutar de las ventajas de MongoDB a las que está acostumbrado, con todas las funcionalidades empresariales que ofrece Cosmos DB: [distribución global](distribute-data-globally.md), [particionamiento automático](partitioning-overview.md), garantías de disponibilidad y latencia, cifrado en reposo, copias de seguridad y mucho más.

## <a name="protocol-support"></a>Compatibilidad con protocolos

A continuación se enumeran los operadores admitidos y las limitaciones o excepciones. Cualquier controlador de cliente que reconozca estos protocolos podrá conectarse a la API de Azure Cosmos DB para MongoDB. Cuando se usa la API de Azure Cosmos DB para las cuentas de MongoDB, las versiones 3.6 y posteriores de las cuentas tienen el punto de conexión con el formato `*.mongo.cosmos.azure.com`, mientras que la versión 3.2 de las cuentas tiene el punto de conexión con el formato `*.documents.azure.com`.

> [!NOTE]
> En este artículo solo se enumeran los comandos de servidor admitidos y se excluyen las funciones contenedoras del lado cliente. Las funciones contenedoras del lado cliente, como `deleteMany()` y `updateMany()` usan internamente los comandos de servidor `delete()` y `update()`. Las funciones que usan comandos de servidor admitidos son compatibles con la API de Azure Cosmos DB para MongoDB.

## <a name="query-language-support"></a>Compatibilidad con lenguajes de consulta

La API de Azure Cosmos DB para MongoDB proporciona una compatibilidad completa con las construcciones del lenguaje de consulta de MongoDB. A continuación, encontrará una lista detallada de las opciones, comandos, fases, operadores y operaciones admitidos actualmente.

## <a name="database-commands"></a>Comandos de base de datos

La API de Azure Cosmos DB para MongoDB admite los siguientes comandos de base de datos:

### <a name="query-and-write-operation-commands"></a>Comandos de operación de consulta y escritura

| Get-Help | Compatible |
|---------|---------|
| [Flujos de cambio](mongodb-change-streams.md) | Sí |
| delete | Sí |
| eval | No |
| find | Sí |
| findAndModify | Sí |
| getLastError | Sí |
| getMore | Sí |
| getPrevError | No |
| insert | Sí |
| parallelCollectionScan | No |
| resetError | No |
| update | Sí |

### <a name="transaction-commands"></a>Comandos de transacción

| Get-Help | Compatible |
|---------|---------|
| abortTransaction | Sí |
| commitTransaction | Sí |

### <a name="authentication-commands"></a>Comandos de autenticación

| Get-Help | Compatible |
|---------|---------|
| authenticate | Sí |
| getnonce | Sí |
| logout | Sí |

### <a name="administration-commands"></a>Comandos de administración

| Get-Help | Compatible |
|---------|---------|
| cloneCollectionAsCapped | No |
| collMod | No |
| connectionStatus | No |
| convertToCapped | No |
| copydb | No |
| create | Sí |
| createIndexes | Sí |
| currentOp | Sí |
| drop | Sí |
| dropDatabase | Sí |
| dropIndexes | Sí |
| filemd5 | Sí |
| killCursors | Sí |
| killOp | No |
| listCollections | Sí |
| listDatabases | Sí |
| listIndexes | Sí |
| reIndex | Sí |
| renameCollection | No |

### <a name="diagnostics-commands"></a>Comandos de diagnóstico

| Get-Help | Compatible |
|---------|---------|
| buildInfo | Sí |
| collStats | Sí |
| connPoolStats | No |
| connectionStatus | No |
| dataSize | No |
| dbHash | No |
| dbStats | Sí |
| explain | Sí |
| features | No |
| hostInfo | Sí |
| listDatabases | Sí |
| listCommands | No |
| profiler | No |
| serverStatus | No |
| top | No |
| whatsmyuri | Sí |

## <a name="aggregation-pipeline"></a>Canalización de agregación<a name="aggregation-pipeline"></a>

### <a name="aggregation-commands"></a>Comandos de agregación

| Get-Help | Compatible |
|---------|---------|
| aggregate | Sí |
| count | Sí |
| distinct | Sí |
| mapReduce | No |

### <a name="aggregation-stages"></a>Fases de agregación

| Get-Help | Compatible |
|---------|---------|
| $addFields | Sí |
| $bucket | No |
| $bucketAuto | No |
| $changeStream | Sí |
| $collStats | No |
| $count | Sí |
| $currentOp | No |
| $facet | Sí |
| $geoNear | Sí |
| $graphLookup | Sí |
| $group | Sí |
| $indexStats | No |
| $limit | Sí |
| $listLocalSessions | No |
| $listSessions | No |
| $lookup | Parcial |
| $match | Sí |
| $out | Sí |
| $project | Sí |
| $redact | Sí |
| $replaceRoot | Sí |
| $replaceWith | No |
| $sample | Sí |
| $skip | Sí |
| $sort | Sí |
| $sortByCount | Sí |
| $unwind | Sí |

> [!NOTE]
> `$lookup` todavía no admite la característica de [subconsultas no correlacionadas](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#join-conditions-and-uncorrelated-sub-queries) introducida en la versión 3.6 del servidor. Recibirá un error con un mensaje que contiene el texto `let is not supported` si intenta utilizar el operador `$lookup` con los campos `let` y `pipeline`.

### <a name="boolean-expressions"></a>Expresiones booleanas

| Get-Help | Compatible |
|---------|---------|
| $and | Sí |
| $not | Sí |
| $or | Sí |

### <a name="conversion-expressions"></a>Expresiones de conversión

| Get-Help | Compatible |
|---------|---------|
| $convert | Sí |
| $toBool | Sí |
| $toDate | Sí |
| $toDecimal | Sí |
| $toDouble | Sí |
| $toInt | Sí |
| $toLong | Sí |
| $toObjectId | Sí |
| $toString | Sí |

### <a name="set-expressions"></a>Expresiones de conjunto

| Get-Help | Compatible |
|---------|---------|
| $setEquals | Sí |
| $setIntersection | Sí |
| $setUnion | Sí |
| $setDifference | Sí |
| $setIsSubset | Sí |
| $anyElementTrue | Sí |
| $allElementsTrue | Sí |

### <a name="comparison-expressions"></a>Expresiones de comparación

| Get-Help | Compatible |
|---------|---------|
| $cmp | Sí |
| $eq | Sí | 
| $gt | Sí | 
| $gte | Sí | 
| $lt | Sí |
| $lte | Sí | 
| $ne | Sí | 
| $in | Sí | 
| $nin | Sí | 

### <a name="arithmetic-expressions"></a>Expresiones aritméticas

| Get-Help | Compatible |
|---------|---------|
| $abs | Sí |
| $add | Sí |
| $ceil | Sí |
| $divide | Sí |
| $exp | Sí |
| $floor | Sí |
| $ln | Sí |
| $log | Sí |
| $log10 | Sí |
| $mod | Sí |
| $multiply | Sí |
| $pow | Sí |
| $sqrt | Sí |
| $subtract | Sí |
| $trunc | Sí |

### <a name="string-expressions"></a>Expresiones de cadena

| Get-Help | Compatible |
|---------|---------|
| $concat | Sí |
| $indexOfBytes | Sí |
| $indexOfCP | Sí |
| $ltrim | Sí |
| $rtrim | Sí |
| $trim | Sí |
| $split | Sí |
| $strLenBytes | Sí |
| $strLenCP | Sí |
| $strcasecmp | Sí |
| $substr | Sí |
| $substrBytes | Sí |
| $substrCP | Sí |
| $toLower | Sí |
| $toUpper | Sí |

### <a name="text-search-operator"></a>Operador de búsqueda de texto

| Get-Help | Compatible |
|---------|---------|
| $meta | No |

### <a name="array-expressions"></a>Expresiones de matriz

| Get-Help | Compatible |
|---------|---------|
| $arrayElemAt | Sí |
| $arrayToObject | Sí |
| $concatArrays | Sí |
| $filter | Sí |
| $indexOfArray | Sí |
| $isArray | Sí |
| $objectToArray | Sí |
| $range | Sí |
| $reverseArray | Sí |
| $reduce | Sí |
| $size | Sí |
| $slice | Sí |
| $zip | Sí |
| $in | Sí |

### <a name="variable-operators"></a>Operadores de variable

| Get-Help | Compatible |
|---------|---------|
| $map | Sí |
| $let | Sí |

### <a name="system-variables"></a>Variables del sistema

| Get-Help | Compatible |
|---------|---------|
| $$CURRENT | Sí |
| $$DESCEND | Sí |
| $$KEEP | Sí |
| $$PRUNE | Sí |
| $$REMOVE | Sí |
| $$ROOT | Sí |

### <a name="literal-operator"></a>Operador literal

| Get-Help | Compatible |
|---------|---------|
| $literal | Sí |

### <a name="date-expressions"></a>Expresiones de fecha

| Get-Help | Compatible |
|---------|---------|
| $dayOfYear | Sí |
| $dayOfMonth | Sí |
| $dayOfWeek | Sí |
| $year | Sí |
| $month | Sí | 
| $week | Sí |
| $hour | Sí |
| $minute | Sí | 
| $second | Sí |
| $millisecond | Sí | 
| $dateToString | Sí |
| $isoDayOfWeek | Sí |
| $isoWeek | Sí |
| $dateFromParts | No | 
| $dateToParts | No |
| $dateFromString | No |
| $isoWeekYear | Sí |

### <a name="conditional-expressions"></a>Expresiones condicionales

| Get-Help | Compatible |
|---------|---------|
| $cond | Sí |
| $ifNull | Sí |
| $switch | Sí |

### <a name="data-type-operator"></a>Operador de tipo de datos

| Get-Help | Compatible |
|---------|---------|
| $type | Sí |

### <a name="accumulator-expressions"></a>Expresiones de acumulador

| Get-Help | Compatible |
|---------|---------|
| $sum | Sí |
| $avg | Sí |
| $first | Sí |
| $last | Sí |
| $max | Sí |
| $min | Sí |
| $push | Sí |
| $addToSet | Sí |
| $stdDevPop | Sí |
| $stdDevSamp | Sí |

### <a name="merge-operator"></a>Operador de combinación

| Get-Help | Compatible |
|---------|---------|
| $mergeObjects | Sí |

## <a name="data-types"></a>Tipos de datos

La API de Azure Cosmos DB para MongoDB admite documentos codificados en formato BSON de MongoDB. La versión 4.0 de la API mejora el uso interno de este formato, lo que supone mejorar el rendimiento y reducir los costos. Los documentos escritos o actualizados con un punto de conexión que ejecute la versión 4.0 se beneficiarán de esta mejora.
 
En un [escenario de actualización](mongodb-version-upgrade.md), los documentos escritos antes de la actualización a la versión 4.0 no se beneficiarán del rendimiento mejorado hasta que se actualicen por medio de una operación de escritura que pase por el punto de conexión 4.0.

| Get-Help | Compatible |
|---------|---------|
| Double | Sí |
| String | Sí |
| Object | Sí |
| Array | Sí |
| Binary Data | Sí | 
| ObjectId | Sí |
| Boolean | Sí |
| Date | Sí |
| Null | Sí |
| Entero de 32 bits (int) | Sí |
| Timestamp | Sí |
| Entero de 64 bits (long) | Sí |
| MinKey | Sí |
| MaxKey | Sí |
| Decimal128 | Sí | 
| Expresión regular | Sí |
| JavaScript | Sí |
| JavaScript (con ámbito)| Sí |
| No definido | Sí |

## <a name="indexes-and-index-properties"></a>Índices y propiedades de índice

### <a name="indexes"></a>Índices

| Get-Help | Compatible |
|---------|---------|
| Índice de campo único | Sí |
| Índice compuesto | Sí |
| Índice de varias claves | Sí |
| Índice de texto | No |
| 2dsphere | Sí |
| Índice 2d | No |
| Índice de hash | Sí |

### <a name="index-properties"></a>Propiedades de índice

| Get-Help | Compatible |
|---------|---------|
| TTL | Sí |
| Único | Sí |
| Parcial | No |
| No distingue mayúsculas de minúsculas | No |
| Dispersos | No |
| Información previa | Sí |

## <a name="operators"></a>Operadores

### <a name="logical-operators"></a>Operadores lógicos

| Get-Help | Compatible |
|---------|---------|
| $or | Sí |
| $and | Sí |
| $not | Sí |
| $nor | Sí | 

### <a name="element-operators"></a>Operadores de elementos

| Get-Help | Compatible |
|---------|---------|
| $exists | Sí |
| $type | Sí |

### <a name="evaluation-query-operators"></a>Operadores de consulta de evaluación

| Get-Help | Compatible |
|---------|---------|
| $expr | No |
| $jsonSchema | No |
| $mod | Sí |
| $regex | Sí |
| $text | No (No admitido. Utilice $regex en su lugar).| 
| $where | No | 

En las consultas de $regex, las expresiones ancladas a la izquierda permiten la búsqueda de índice. Sin embargo, si utiliza el modificador 'i' (no distingue mayúsculas y minúsculas) y el modificador 'm' modificador (multilínea), se realiza el examen de colección de todas las expresiones.

Cuando es necesario incluir '$' o '|', es mejor crear dos (o más) consultas regex. Por ejemplo, dada la siguiente consulta original: `find({x:{$regex: /^abc$/})`, tiene que modificarse de la siguiente forma:

`find({x:{$regex: /^abc/, x:{$regex:/^abc$/}})`

La primera parte utilizará el índice para restringir la búsqueda a esos documentos que empiezan por ^ abc y la segunda parte buscará coincidencias con los datos exactos. El operador de barra '|' actúa como una función "or": la consulta `find({x:{$regex: /^abc |^def/})` coincide con los documentos con los que el campo "x" tiene un valor que comienza por "abc" o "def". Para utilizar el índice, se recomienda dividir la consulta en dos consultas distintas combinadas mediante el operador $or: `find( {$or : [{x: $regex: /^abc/}, {$regex: /^def/}] })`.

### <a name="array-operators"></a>Operadores de matriz

| Get-Help | Compatible | 
|---------|---------|
| $all | Sí | 
| $elemMatch | Sí | 
| $size | Sí | 

### <a name="comment-operator"></a>Operador de comentario

| Get-Help | Compatible | 
|---------|---------|
| $comment | Sí | 

### <a name="projection-operators"></a>Operadores de proyección

| Get-Help | Compatible |
|---------|---------|
| $elemMatch | Sí |
| $meta | No |
| $slice | Sí |

### <a name="update-operators"></a>Operadores de actualización

#### <a name="field-update-operators"></a>Operadores de actualización de campo

| Get-Help | Compatible |
|---------|---------|
| $inc | Sí |
| $mul | Sí |
| $rename | Sí |
| $setOnInsert | Sí |
| $set | Sí |
| $unset | Sí |
| $min | Sí |
| $max | Sí |
| $currentDate | Sí |

#### <a name="array-update-operators"></a>Operadores de actualización de matriz

| Get-Help | Compatible |
|---------|---------|
| $ | Sí |
| $[]| Sí |
| $[<identifier>]| Sí |
| $addToSet | Sí |
| $pop | Sí |
| $pullAll | Sí |
| $pull | Sí |
| $push | Sí |
| $pushAll | Sí |

#### <a name="update-modifiers"></a>Modificadores de actualización

| Get-Help | Compatible |
|---------|---------|
| $each | Sí |
| $slice | Sí |
| $sort | Sí |
| $position | Sí |

#### <a name="bitwise-update-operator"></a>Operador de actualización bit a bit

| Get-Help | Compatible |
|---------|---------|
| $bit | Sí | 
| $bitsAllSet | No |
| $bitsAnySet | No |
| $bitsAllClear | No |
| $bitsAnyClear | No |

### <a name="geospatial-operators"></a>Operadores de geoespaciales

Operator | Compatible | 
--- | --- |
$geoWithin | Sí |
$geoIntersects | Sí | 
$near | Sí |
$nearSphere | Sí |
$geometry | Sí |
$minDistance | Sí |
$maxDistance | Sí |
$center | No |
$centerSphere | No |
$box | No |
$polygon | No |

## <a name="sort-operations"></a>Operaciones de ordenación

Cuando se usa la operación `findOneAndUpdate`, se admiten las operaciones de ordenación en un solo campo, pero no se admiten las operaciones de ordenación en varios campos.

## <a name="unique-indexes"></a>Índices únicos

Los [índices únicos](mongodb-indexing.md#unique-indexes) garantizan que un campo determinado no tiene valores duplicados en todos los documentos de una colección, de forma parecida a como se conserva la singularidad en la clave "_id" predeterminada. Puede crear índices únicos en Azure Cosmos DB mediante el comando `createIndex` con el parámetro de restricción `unique`:

```javascript
globaldb:PRIMARY> db.coll.createIndex( { "amount" : 1 }, {unique:true} )
{
    "_t" : "CreateIndexesResponse",
    "ok" : 1,
    "createdCollectionAutomatically" : false,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 4
}
```

## <a name="compound-indexes"></a>Índices compuestos

Los [índices compuestos](mongodb-indexing.md#compound-indexes-mongodb-server-version-36) proporcionan una manera de crear un índice para grupos de hasta ocho campos. Este tipo de índice difiere de los índices compuestos nativos de MongoDB. En Azure Cosmos DB, se usan índices compuestos para las operaciones de ordenación que se aplican a varios campos. Para crear un índice compuesto, debe especificar más de una propiedad como parámetro:

```javascript
globaldb:PRIMARY> db.coll.createIndex({"amount": 1, "other":1})
{
    "createdCollectionAutomatically" : false, 
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "ok" : 1
}
```

## <a name="gridfs"></a>GridFS

Azure Cosmos DB admite GridFS con cualquier controlador Mongo compatible con GridFS.

## <a name="replication"></a>Replicación

Azure Cosmos DB admite la replicación automática y nativa en las capas más inferiores. Esta lógica se amplía para lograr también una replicación global de baja latencia. Cosmos DB no es compatible con comandos de replicación manuales.

## <a name="retryable-writes"></a>Escrituras reintentables

Cosmos DB no admite aún escrituras reintentables. Los controladores cliente deben agregar el parámetro de URL "retryWrites=false" a su cadena de conexión. Se pueden agregar más parámetros de URL si se les antepone el prefijo "&". 

## <a name="sharding"></a>Particionamiento

Azure Cosmos DB admite el particionamiento de servidor automático. Administra la creación de particiones, la ubicación y el equilibrio de forma automática. Azure Cosmos DB no admite comandos de particionamiento manual, lo que significa que no tiene que invocar comandos como addShard, balancerStart, moveChunk, etc. Solo tiene que especificar la clave de partición al crear los contenedores o consultar los datos.

## <a name="sessions"></a>Sesiones

Azure Cosmos DB todavía no admite comandos de sesiones del lado servidor.

## <a name="time-to-live-ttl"></a>Período de vida (TTL)

Azure Cosmos DB admite un período de vida (TTL) basado en la marca de tiempo del documento. TTL se puede habilitar para las colecciones mediante [Azure Portal](https://portal.azure.com).

## <a name="transactions"></a>Transacciones

Las transacciones de varios documentos se admiten en una colección no particionada. Las transacciones de varios documentos no se admiten en colecciones ni en colecciones con particiones. El tiempo de espera de las transacciones es un valor fijo de 5 segundos.

## <a name="user-and-role-management"></a>Administración de usuarios y roles

Azure Cosmos DB no admite aún usuarios y roles. Sin embargo, Cosmos DB admite el control de acceso basado en rol de Azure (Azure RBAC) y claves o contraseñas de solo lectura y escritura que se pueden obtener mediante [Azure Portal](https://portal.azure.com) (página Cadena de conexión).

## <a name="write-concern"></a>Write Concern

Algunas aplicaciones se basan en [Write Concern](https://docs.mongodb.com/manual/reference/write-concern/), que especifica el número de respuestas necesarias durante una operación de escritura. Debido a la forma en la que Cosmos DB controla la replicación en un segundo plano todas las escrituras se establecen automáticamente como cuórum de forma predeterminada. Cualquier nivel de Write Concern especificado por el código de cliente se ignora. Más información en el artículo sobre el [Uso de los niveles de coherencia para maximizar la disponibilidad y el rendimiento](consistency-levels.md).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [usar Studio 3T](mongodb-mongochef.md) con la API de Azure Cosmos DB para MongoDB.
- Aprenda a [usar Robo 3T](mongodb-robomongo.md) con la API de Azure Cosmos DB para MongoDB.
- Explore [ejemplos](mongodb-samples.md) de MongoDB con la API de Azure Cosmos DB para MongoDB.
