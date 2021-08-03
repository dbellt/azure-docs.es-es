---
title: Actualización e inserción de datos en Cassandra API de Azure Cosmos DB desde Spark
description: En este artículo se detalla cómo realizar una operación de actualización e inserción de datos en tablas de Cassandra API de Azure Cosmos DB desde Spark
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 2d25ca629953c78648bed5c33f3598b64eff658d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110464704"
---
# <a name="upsert-data-into-azure-cosmos-db-cassandra-api-from-spark"></a>Actualización e inserción de datos en Cassandra API de Azure Cosmos DB desde Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

En este artículo se describe cómo realizar una operación de actualización e inserción de datos en Cassandra API de Azure Cosmos DB desde Spark.

## <a name="cassandra-api-configuration"></a>Configuración de Cassandra API

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//if using Spark 2.x, CosmosDB library for multiple retry
//import com.microsoft.azure.cosmosdb.cassandra

//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
// if using Spark 2.x
// spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

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
> Si usa Spark 3.0 o posterior, no es necesario instalar el asistente de Cosmos DB ni el generador de conexiones. También debe usar `remoteConnectionsPerExecutor` en lugar de `connections_per_executor_max` para el conector de Spark 3 (consulte más arriba). Verá que las propiedades relacionadas con la conexión están definidas en el cuaderno anterior. Con la sintaxis siguiente, las propiedades de conexión se pueden definir de esta manera sin necesidad de definirse en el nivel de clúster (inicialización del contexto de Spark). Sin embargo, cuando se usan operaciones que requieren contexto de Spark (por ejemplo, `CassandraConnector(sc)` para `update` como se muestra a continuación), las propiedades de conexión deben definirse en el nivel de clúster.

## <a name="dataframe-api"></a>Dataframe API

### <a name="create-a-dataframe"></a>Creación de una base de datos 

```scala
// (1) Update: Changing author name to include prefix of "Sir"
// (2) Insert: adding a new book

val booksUpsertDF = Seq(
    ("b00001", "Sir Arthur Conan Doyle", "A study in scarlet", 1887),
    ("b00023", "Sir Arthur Conan Doyle", "A sign of four", 1890),
    ("b01001", "Sir Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
    ("b00501", "Sir Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
    ("b00300", "Sir Arthur Conan Doyle", "The hounds of Baskerville", 1901),
    ("b09999", "Sir Arthur Conan Doyle", "The return of Sherlock Holmes", 1905)
    ).toDF("book_id", "book_author", "book_name", "book_pub_year")
booksUpsertDF.show()
```

### <a name="upsert-data"></a>Actualización e inserción de datos

```scala
// Upsert is no different from create
booksUpsertDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .save()
```

### <a name="update-data"></a>Actualización de datos

```scala
//Cassandra connector instance
val cdbConnector = CassandraConnector(sc)

//This runs on the driver, leverage only for one off updates
cdbConnector.withSessionDo(session => session.execute("update books_ks.books set book_price=99.33 where book_id ='b00300' and book_pub_year = 1901;"))
```

## <a name="rdd-api"></a>RDD API
> [!NOTE]
> La operación de actualización e inserción desde la API de RDD es igual que la operación de creación. 

## <a name="next-steps"></a>Pasos siguientes

Continúe con los artículos siguientes para realizar otras operaciones en los datos almacenados en tablas de Cassandra API de Azure Cosmos DB:
 
* [Delete operations](cassandra-spark-delete-ops.md) (Operaciones de eliminación)
* [Aggregation operations](cassandra-spark-aggregation-ops.md) (Operaciones de agregación)
* [Table copy operations](cassandra-spark-table-copy-ops.md) (Operaciones de copia en tabla)
