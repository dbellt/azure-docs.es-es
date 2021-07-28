---
title: Creación o inserción de datos en Cassandra API de Azure Cosmos DB desde Spark
description: En este artículo se detalla cómo insertar datos de ejemplo en tablas de Cassandra API de Azure Cosmos DB
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 585c2985850d47b8df80df6d748bff3a8fafcb1b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110464942"
---
# <a name="createinsert-data-into-azure-cosmos-db-cassandra-api-from-spark"></a>Creación e inserción de datos en Cassandra API de Azure Cosmos DB desde Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]
 
En este artículo se describe cómo insertar datos de ejemplo en una tabla de Cassandra API de Azure Cosmos DB desde Spark.

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
> Si usa Spark 3.0 o posterior, no es necesario instalar el asistente de Cosmos DB ni el generador de conexiones. También debe usar `remoteConnectionsPerExecutor` en lugar de `connections_per_executor_max` para el conector de Spark 3 (consulte más arriba). Verá que las propiedades relacionadas con la conexión están definidas en el cuaderno anterior. Con la sintaxis siguiente, las propiedades de conexión se pueden definir de esta manera sin necesidad de definirse en el nivel de clúster (inicialización del contexto de Spark).

## <a name="dataframe-api"></a>Dataframe API

### <a name="create-a-dataframe-with-sample-data"></a>Creación de una trama con datos de ejemplo

```scala
// Generate a dataframe containing five records
val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
).toDF("book_id", "book_author", "book_name", "book_pub_year")

//Review schema
booksDF.printSchema

//Print
booksDF.show
```

> [!NOTE]
> Aún no se admite la funcionalidad "Crear si no existe" a nivel de registro.

### <a name="persist-to-azure-cosmos-db-cassandra-api"></a>Persistencia en Cassandra API de Azure Cosmos DB

Al guardar los datos, también puede establecer el período de vida y la configuración de la directiva de coherencia como se muestra en el siguiente ejemplo:

```scala
//Persist
booksDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000"))
  .save()
```

> [!NOTE]
> No se admite todavía el período de vida a nivel de columna.

#### <a name="validate-in-cqlsh"></a>Validar en cqlsh

```sql
use books_ks;
select * from books;
```

## <a name="resilient-distributed-database-rdd-api"></a>Resilient Distributed Database (RDD) API

### <a name="create-a-rdd-with-sample-data"></a>Creación de una RDD con datos de ejemplo
```scala
//Drop and re-create table to delete records created in the previous section 
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP TABLE IF EXISTS books_ks.books;"))

cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT, PRIMARY KEY(book_id,book_pub_year)) WITH cosmosdb_provisioned_throughput=4000 , WITH default_time_to_live=630720000;"))

//Create RDD
val booksRDD = sc.parallelize(Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
))

//Review
booksRDD.take(2).foreach(println)
```

> [!NOTE]
> Aún no se admite la funcionalidad "Crear si no existe".

### <a name="persist-to-azure-cosmos-db-cassandra-api"></a>Persistencia en Cassandra API de Azure Cosmos DB

Al guardar los datos en Cassandra API, también puede establecer el período de vida y la configuración de la directiva de coherencia como se muestra en el siguiente ejemplo:

```scala
import com.datastax.spark.connector.writer._

//Persist
booksRDD.saveToCassandra("books_ks", "books", SomeColumns("book_id", "book_author", "book_name", "book_pub_year"),writeConf = WriteConf(ttl = TTLOption.constant(900000),consistencyLevel = ConsistencyLevel.ALL))
```

#### <a name="validate-in-cqlsh"></a>Validar en cqlsh

```sql
use books_ks;
select * from books;
```

## <a name="next-steps"></a>Pasos siguientes

Tras insertar los datos en la tabla de Cassandra API de Azure Cosmos DB, continúe con los siguientes artículos para realizar otras operaciones en los datos almacenados en Cassandra API de Cosmos DB:
 
* [Lee operaciones.](cassandra-spark-read-ops.md)
* [Upsert operations](cassandra-spark-upsert-ops.md) (Operaciones de upsert)
* [Delete operations](cassandra-spark-delete-ops.md) (Operaciones de eliminación)
* [Aggregation operations](cassandra-spark-aggregation-ops.md) (Operaciones de agregación)
* [Table copy operations](cassandra-spark-table-copy-ops.md) (Operaciones de copia en tabla)
