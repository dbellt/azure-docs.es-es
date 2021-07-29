---
title: Operaciones DDL en Cassandra API de Azure Cosmos DB de Spark
description: En este artículo se describe el espacio de claves y la tabla de operaciones DDL en Cassandra API de Azure Cosmos DB de Spark.
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/07/2020
ms.openlocfilehash: 54398620775b28f0c497a061f5ea4446a38a5ada
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110464925"
---
# <a name="ddl-operations-in-azure-cosmos-db-cassandra-api-from-spark"></a>Operaciones DDL en Cassandra API de Azure Cosmos DB de Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

En este artículo se describe el espacio de claves y la tabla de operaciones DDL en Cassandra API de Azure Cosmos DB de Spark.

## <a name="spark-context"></a>Contexto de Spark

 El conector para Cassandra API requiere que los detalles de la conexión de Cassandra se inicialicen como parte del contexto de Spark. Cuando inicia un cuaderno, el contexto de Spark ya está inicializado y no es aconsejable pararlo y reiniciarlo. Una solución es agregar la configuración de la instancia de Cassandra API en un nivel de clúster, en la configuración de clúster de Spark. Esto se hace una sola vez por cada clúster. Agregue el código siguiente a la configuración de Spark como un par clave-valor separado por espacios:
 
  ```scala
  spark.cassandra.connection.host YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com
  spark.cassandra.connection.port 10350
  spark.cassandra.connection.ssl.enabled true
  spark.cassandra.auth.username YOUR_COSMOSDB_ACCOUNT_NAME
  spark.cassandra.auth.password YOUR_COSMOSDB_KEY
  ```

## <a name="cassandra-api-related-configuration"></a>Configuración de Cassandra API 

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//if using Spark 2.x, CosmosDB library for multiple retry
//import com.microsoft.azure.cosmosdb.cassandra
//spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

//Throughput-related...adjust as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
//spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10") // Spark 2.x
spark.conf.set("spark.cassandra.connection.remoteConnectionsPerExecutor", "10") // Spark 3.x
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```

> [!NOTE]
> Si usa Spark 3.0 o posterior, no es necesario instalar el asistente de Cosmos DB ni el generador de conexiones. También debe usar `remoteConnectionsPerExecutor` en lugar de `connections_per_executor_max` para el conector de Spark 3 (consulte más arriba).

## <a name="keyspace-ddl-operations"></a>Operaciones DDL de Keyspace

### <a name="create-a-keyspace"></a>Crear un espacio de claves

```scala
//Cassandra connector instance
val cdbConnector = CassandraConnector(sc)

// Create keyspace
cdbConnector.withSessionDo(session => session.execute("CREATE KEYSPACE IF NOT EXISTS books_ks WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1 } "))
```

#### <a name="validate-in-cqlsh"></a>Validar en cqlsh

Ejecute el siguiente comando en cqlsh y verá el espacio de claves que creó anteriormente.

```bash
DESCRIBE keyspaces;
```

### <a name="drop-a-keyspace"></a>Colocar un espacio de claves

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP KEYSPACE books_ks"))
```

#### <a name="validate-in-cqlsh"></a>Validar en cqlsh

```bash
DESCRIBE keyspaces;
```
## <a name="table-ddl-operations"></a>Operaciones DDL de tabla

**Consideraciones:**  

- El rendimiento puede asignarse a nivel de tabla; para ello, use la instrucción create table.  
- Una clave de partición puede almacenar 20 GB de datos.  
- Un registro puede almacenar un máximo de 2 MB de datos.  
- Un intervalo de claves de partición puede almacenar varias claves de partición.

### <a name="create-a-table"></a>Creación de una tabla

```scala
cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT, PRIMARY KEY(book_id,book_pub_year)) WITH cosmosdb_provisioned_throughput=4000 , WITH default_time_to_live=630720000;"))
```

#### <a name="validate-in-cqlsh"></a>Validar en cqlsh

Ejecute el siguiente comando en cqlsh y verá la tabla denominada "books:" 

```bash
USE books_ks;
DESCRIBE books;
```

El rendimiento previsto y los valores TTL predeterminados no se muestran en la salida del comando anterior, pero puede obtener estos valores en el portal.

### <a name="alter-table"></a>Alter table

Puede modificar los siguientes valores mediante el comando alter table:

* rendimiento aprovisionado 
* valor del período de vida
<br>Actualmente no se admiten los cambios en columnas.

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("ALTER TABLE books_ks.books WITH cosmosdb_provisioned_throughput=8000, WITH default_time_to_live=0;"))
```

### <a name="drop-table"></a>Quitar tabla

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP TABLE IF EXISTS books_ks.books;"))
```

#### <a name="validate-in-cqlsh"></a>Validar en cqlsh

Ejecute el siguiente comando en cqlsh y verá que la tabla "books" ya no está disponible:

```bash
USE books_ks;
DESCRIBE tables;
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear el espacio de claves y la tabla, lea los siguientes artículos referentes a las operaciones CRUD y mucho más:
 
* [Create/insert operations](cassandra-spark-create-ops.md) (Operaciones de creación e inserción)  
* [Lee operaciones.](cassandra-spark-read-ops.md)  
* [Upsert operations](cassandra-spark-upsert-ops.md) (Operaciones de upsert)  
* [Delete operations](cassandra-spark-delete-ops.md) (Operaciones de eliminación)  
* [Aggregation operations](cassandra-spark-aggregation-ops.md) (Operaciones de agregación)  
* [Table copy operations](cassandra-spark-table-copy-ops.md) (Operaciones de copia en tabla)  
