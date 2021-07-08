---
title: Migración a Azure Managed Instance for Apache Cassandra mediante Apache Spark
description: Aprenda cómo migrar a Azure Managed Instance for Apache Cassandra mediante Apache Spark.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: overview
ms.date: 06/02/2021
ms.openlocfilehash: b49e1c42b3b81a548a1860a4e4e4ee1ea97a7f4c
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955994"
---
# <a name="migrate-to-azure-managed-instance-for-apache-cassandra-using-apache-spark"></a>Migración a Azure Managed Instance for Apache Cassandra mediante Apache Spark

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Siempre que sea posible, se recomienda usar la replicación nativa de Apache Cassandra para migrar datos del clúster existente a Azure Managed Instance for Apache Cassandra mediante la configuración de un [clúster híbrido](configure-hybrid-cluster.md). Este enfoque usará el protocolo de Gossip de Apache Cassandra para replicar datos del centro de datos de origen en el nuevo centro de datos de la instancia administrada. Sin embargo, puede haber algunos escenarios en los que la versión de la base de datos de origen no sea compatible, o una configuración de clúster híbrido no sea factible. 

En este artículo se describe cómo migrar datos a Azure Managed Instance for Apache Cassandra sin conexión mediante el conector de Spark para Cassandra y Azure Databricks para Apache Spark.

## <a name="prerequisites"></a>Prerrequisitos

* Aprovisione un clúster de Azure Managed Instance for Apache Cassandra mediante [Azure Portal](create-cluster-portal.md) o la [CLI de Azure](create-cluster-cli.md), y asegúrese de que puede [conectarse al clúster con CQLSH](./create-cluster-portal.md#connecting-to-your-cluster).

* [Aprovisione una cuenta de Azure Databricks dentro de la red virtual de Cassandra administrada](deploy-cluster-databricks.md). Asegúrese de que también tiene acceso de red al clúster de Cassandra de origen.

* Asegúrese de ya haber migrado el esquema del espacio de claves o tablas de la base de datos de Cassandra de origen a la base de datos de Managed Instance for Cassandra de destino.


## <a name="provision-an-azure-databricks-cluster"></a>Aprovisionar un clúster de Azure Databricks

Se recomienda seleccionar el entorno de ejecución de Databricks Runtime versión 7.5, que admite Spark 3.0:

:::image type="content" source="../cosmos-db/media/cassandra-migrate-cosmos-db-databricks/databricks-runtime.png" alt-text="Captura de pantalla que muestra la búsqueda de la versión de tiempo de ejecución de Databricks":::

## <a name="add-dependencies"></a>Adición de dependencias

Agregue la biblioteca de conectores de Cassandra de Apache Spark a su clúster para conectarse a los puntos de conexión nativos y de Cassandra de Azure Cosmos DB. En el clúster, seleccione **Libraries** > **Install New** > **Maven** (Bibliotecas > Instalar nueva > Maven) y, después, agregue `com.datastax.spark:spark-cassandra-connector-assembly_2.12:3.0.0` en las coordenadas de Maven.

:::image type="content" source="../cosmos-db/media/cassandra-migrate-cosmos-db-databricks/databricks-search-packages.png" alt-text="Captura de pantalla que muestra la búsqueda de paquetes Maven en Databricks":::

Seleccione **Install** (Instalar) y asegúrese de reiniciar el clúster cuando se complete la instalación.

> [!NOTE]
> Asegúrese de reiniciar el clúster de Databricks después de que se haya instalado la biblioteca de conectores de Cassandra.

## <a name="create-scala-notebook-for-migration"></a>Creación de un cuaderno de Scala para la migración

Cree un cuaderno de Scala en Databricks. Reemplace las configuraciones de Cassandra de origen y de destino con las credenciales correspondientes, así como los espacios de claves y las tablas de origen y destino. Después, ejecute el código siguiente:

```scala
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql._
import org.apache.spark.SparkContext

// source cassandra configs
val sourceCassandra = Map( 
    "spark.cassandra.connection.host" -> "<Source Cassandra Host>",
    "spark.cassandra.connection.port" -> "9042",
    "spark.cassandra.auth.username" -> "<USERNAME>",
    "spark.cassandra.auth.password" -> "<PASSWORD>",
    "spark.cassandra.connection.ssl.enabled" -> "false",
    "keyspace" -> "<KEYSPACE>",
    "table" -> "<TABLE>"
)

//target cassandra configs
val targetCassandra = Map( 
    "spark.cassandra.connection.host" -> "<Source Cassandra Host>",
    "spark.cassandra.connection.port" -> "9042",
    "spark.cassandra.auth.username" -> "<USERNAME>",
    "spark.cassandra.auth.password" -> "<PASSWORD>",
    "spark.cassandra.connection.ssl.enabled" -> "true",
    "keyspace" -> "<KEYSPACE>",
    "table" -> "<TABLE>",
    //throughput related settings below - tweak these depending on data volumes. 
    "spark.cassandra.output.batch.size.rows"-> "1",
    "spark.cassandra.output.concurrent.writes" -> "1000",
    "spark.cassandra.connection.remoteConnectionsPerExecutor" -> "10",
    "spark.cassandra.concurrent.reads" -> "512",
    "spark.cassandra.output.batch.grouping.buffer.size" -> "1000",
    "spark.cassandra.connection.keep_alive_ms" -> "600000000"
)

//Read from source Cassandra
val DFfromSourceCassandra = sqlContext
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(sourceCassandra)
  .load
  
//Write to target Cassandra
DFfromSourceCassandra
  .write
  .format("org.apache.spark.sql.cassandra")
  .options(targetCassandra)
  .mode(SaveMode.Append) // only required for Spark 3.x
  .save
```

> [!NOTE]
> Si tiene la necesidad de conservar o retrasar `writetime` de cada fila, consulte el artículo sobre la [migración en vivo](dual-write-proxy-migration.md). 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure](manage-resources-cli.md)