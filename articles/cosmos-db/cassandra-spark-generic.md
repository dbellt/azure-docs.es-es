---
title: Uso de Cassandra API de Azure Cosmos DB desde Spark
description: Este artículo es la página principal para la integración de Cassandra API de Cosmos DB desde Spark.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/01/2019
ms.openlocfilehash: 8e4742e475f98198b667395aec9d55bcf2a0eaeb
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959058"
---
# <a name="connect-to-azure-cosmos-db-cassandra-api-from-spark"></a>Conexión con Cassandra API de Azure Cosmos DB desde Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Este artículo es uno de los que se encuentra entre la serie de artículos sobre la integración de Cassandra API de Azure Cosmos DB desde Spark. En los artículos se explica la conectividad, las operaciones de lenguaje de definición de datos (DDL), las operaciones básicas de lenguaje de manipulación de datos (DML) y la integración avanzada de Cassandra API de Azure Cosmos DB desde Spark. 

## <a name="prerequisites"></a>Requisitos previos
* [Aprovisionar una cuenta de Cassandra API de Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account).

* Aprovisionar la elección del entorno de Spark [[Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal) | [Spark de Azure HDInsight](../hdinsight/spark/apache-spark-jupyter-spark-sql.md) | Otros].

## <a name="dependencies-for-connectivity"></a>Dependencias de conectividad
* **Conector de Spark para Cassandra:** el conector de Spark se usa para establecer conexión con Cassandra API de Azure Cosmos DB.  Identifique y use la versión del conector que se encuentra en la [central de Maven]( https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector) que sea compatible con las versiones de Spark y Scala de su entorno de Spark. Se recomienda un entorno que admita Spark 3.0 o una versión posterior y el conector de Spark disponible en las coordenadas de Maven `com.datastax.spark:spark-cassandra-connector-assembly_2.12:3.0.0`. Si usa Spark 2.x, se recomienda un entorno con la versión de Spark 2.4.5, con el conector de Spark en las coordenadas de Maven `com.datastax.spark:spark-cassandra-connector_2.11:2.4.3`.


* **Biblioteca auxiliar de Azure Cosmos DB para Cassandra API:** si usa una versión de Spark 2.x, además del conector de Spark, necesita otra biblioteca llamada [azure-cosmos-cassandra-spark-helper]( https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.2.0/jar) con las coordenadas de Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.2.0` de Azure Cosmos DB para administrar el [límite de velocidad](./manage-scale-cassandra.md#handling-rate-limiting-429-errors). Esta biblioteca contiene un generador de conexión y clases de directivas de reintentos personalizados.

  La directiva de reintentos de Azure Cosmos DB está configurada para controlar las excepciones del código de estado HTTP 429 ("Tasa de solicitud grande"). Cassandra API de Azure Cosmos DB traslada estas excepciones como errores sobrecargados en el protocolo nativo de Cassandra y el usuario puede volver a intentarlo con interrupciones. Dado que Azure Cosmos DB usa el modelo de rendimiento aprovisionado, se producen excepciones de limitación de tasas de solicitud al aumentar las tasas de entrada/salida. La directiva de reintentos protege los trabajos de Spark frente a picos de datos que superan momentáneamente el rendimiento asignado al contenedor. Si usa el conector de Spark 3.x, no es necesario implementar esta biblioteca. 

  > [!NOTE] 
  > La directiva de reintentos solo puede proteger los trabajos de Spark frente a picos momentáneos. Si no ha configurado suficientes RU necesarias para ejecutar la carga de trabajo, la directiva de reintentos no es aplicable y la clase de directiva de reintentos vuelve a producir la excepción.

* **Detalles de conexión de la cuenta de Azure Cosmos DB:** nombre de la cuenta, punto de conexión de la cuenta y clave de Cassandra API de Azure.

## <a name="optimizing-spark-connector-throughput-configuration"></a>Optimización de la configuración de rendimiento del conector de Spark 

En la sección siguiente se muestran todos los parámetros pertinentes para controlar el rendimiento mediante el conector de Spark para Cassandra. Para optimizar los parámetros a fin de maximizar el rendimiento de los trabajos de Spark, las configuraciones de `spark.cassandra.output.concurrent.writes`, `spark.cassandra.concurrent.reads` y `spark.cassandra.input.reads_per_sec` deben ser las correctas para así evitar demasiados límites e interrupciones (lo que, a su vez, puede dar lugar a un menor rendimiento).

El valor óptimo de estas configuraciones depende de cuatro factores:

-   La cantidad de rendimiento (unidades de solicitud) configurada para la tabla en la que se ingieren los datos.
- El número de trabajos en el clúster de Spark.
-   El número de ejecutores configurados para el trabajo de Spark (que se puede controlar mediante `spark.cassandra.connection.connections_per_executor_max` o `spark.cassandra.connection.remoteConnectionsPerExecutor`, según la versión de Spark).
-   La latencia media de cada solicitud a Cosmos DB, si se le coloca en el mismo centro de datos. Suponga que este valor es de 10 ms para escrituras y 3 ms para lecturas.

Por ejemplo, si tenemos 5 trabajos y un valor de `spark.cassandra.output.concurrent.writes`= 1 y un valor de `spark.cassandra.connection.remoteConnectionsPerExecutor`= 1, tenemos 5 trabajos que escriben simultáneamente en la tabla, cada uno con 1 subproceso. Si se tardan 10 ms en realizar una sola escritura, podemos enviar 100 solicitudes (1000 milisegundos divididos por 10) por segundo, por subproceso. Con 5 trabajos, serían 500 escrituras por segundo. Con un costo medio de 5 unidades de solicitud (RU) por escritura, la tabla de destino necesitaría 2500 unidades de solicitud aprovisionadas como mínimo (5 RU x 500 escrituras por segundo).

Al aumentar el número de ejecutores, puede aumentar el número de subprocesos de un trabajo determinado, lo que a su vez puede aumentar el rendimiento. Sin embargo, el efecto exacto de esta acción puede ser variable en función del trabajo, mientras que el control del rendimiento con el número de trabajos es más determinista. También puede determinar el costo exacto de una solicitud determinada mediante la generación de perfiles para obtener el cargo de unidad de solicitud (RU). Esta opción le ayudará a ser más preciso al aprovisionar el rendimiento de la tabla o el espacio de claves. Consulte nuestro artículo [aquí](./find-request-unit-charge-cassandra.md) para comprender cómo obtener los cargos por unidad de solicitud en un nivel de solicitud. 

> [!NOTE]
> En la guía anterior se supone una distribución de datos razonablemente uniforme. Si tiene un sesgo significativo en los datos (es decir, un número desmesuradamente grande de lecturas y escrituras para el mismo valor de clave de partición), es posible que siga experimentando cuellos de botella, incluso si tiene un gran número de [unidades de solicitud](./request-units.md) aprovisionadas en la tabla. Las unidades de solicitud se dividen equitativamente entre las particiones físicas y, una asimetría de datos intensiva puede causar un cuello de botella de las solicitudes en una única partición.
    
## <a name="spark-connector-throughput-configuration-parameters"></a>Parámetros de configuración de rendimiento del conector de Spark

En la tabla siguiente se enumeran los parámetros de configuración de rendimiento específicos de Cassandra API de Azure Cosmos DB proporcionados por el conector. Para obtener una lista detallada de todos los parámetros de configuración, consulte la página de [referencia de configuración](https://github.com/datastax/spark-cassandra-connector/blob/master/doc/reference.md) del repositorio de GitHub del conector de Cassandra de Spark.

| **Nombre de la propiedad** | **Valor predeterminado** | **Descripción** |
|---------|---------|---------|
| spark.cassandra.output.batch.size.rows |  1 |Número de filas por lote único. Establezca este parámetro en 1. Este parámetro se utiliza para lograr un mayor rendimiento para las cargas de trabajo altas. |
| spark.cassandra.connection.connections_per_executor_max (Spark 2.x) spark.cassandra.connection.remoteConnectionsPerExecutor (Spark 3.x)  | None | Número máximo de conexiones por nodo y ejecutor. 10*n equivale a diez conexiones por nodo en un clúster de Cassandra con n nodos. Por lo tanto, si necesita cinco conexiones por nodo y ejecutor para un clúster de Cassandra de cinco nodos, debe establecer esta configuración en 25. Modifique este valor según el grado de paralelismo o el número de ejecutores configurados para los trabajos de Spark.   |
| spark.cassandra.output.concurrent.writes  |  100 | Define el número de escrituras paralelas que pueden producirse por ejecutor. Dado que ha establecido "batch.size.rows" en 1, asegúrese de escalar verticalmente este valor en consecuencia. Modifique este valor según el grado de paralelismo o el rendimiento que desea lograr para la carga de trabajo. |
| spark.cassandra.concurrent.reads |  512 | Define el número de lecturas en paralelo que pueden producirse por ejecutor. Modifique este valor según el grado de paralelismo o el rendimiento que desea lograr para la carga de trabajo.  |
| spark.cassandra.output.throughput_mb_per_sec  | None | Define el rendimiento de escritura total por ejecutor. Este parámetro puede usarse como límite superior para el rendimiento de trabajo de Spark y basarse en el rendimiento aprovisionado del contenedor de Cosmos.   |
| spark.cassandra.input.reads_per_sec| None   | Define el rendimiento de lectura total por ejecutor. Este parámetro puede usarse como límite superior para el rendimiento de trabajo de Spark y basarse en el rendimiento aprovisionado del contenedor de Cosmos.  |
| spark.cassandra.output.batch.grouping.buffer.size |  1000  | Define el número de lotes por cada tarea única de Spark que se pueden almacenar en la memoria antes de enviarlos a Cassandra API. |
| spark.cassandra.connection.keep_alive_ms | 60000 | Define el período de tiempo hasta el que están disponibles las conexiones no utilizadas. | 

Ajuste el rendimiento y el grado de paralelismo de estos parámetros en función de la carga de trabajo que espera para los trabajos de Spark y el rendimiento que se ha aprovisionado para la cuenta de Cosmos DB.


## <a name="connecting-to-azure-cosmos-db-cassandra-api-from-spark"></a>Conexión con Cassandra API de Azure Cosmos DB desde Spark

### <a name="cqlsh"></a>cqlsh
Los comandos siguientes proporcionan información detallada acerca de cómo conectarse a Cassandra API de Azure CosmosDB desde cqlsh.  Esto es útil para la validación mientras se ejecuta a través de los ejemplos de Spark.<br>
**Desde Linux/Unix/Mac:**

```bash
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false
cqlsh.py YOUR-COSMOSDB-ACCOUNT-NAME.cassandra.cosmosdb.azure.com 10350 -u YOUR-COSMOSDB-ACCOUNT-NAME -p YOUR-COSMOSDB-ACCOUNT-KEY --ssl
```

### <a name="1--azure-databricks"></a>1.  Azure Databricks
En el siguiente artículo se contempla el aprovisionamiento del clúster de Azure Databricks, la configuración del clúster para conectarse a Cassandra API de Azure Cosmos DB y varios cuadernos de ejemplo que abarcan operaciones DDL, operaciones DML y mucho más.<BR>
[Uso de Cassandra API de Azure Cosmos DB desde Azure Databricks](cassandra-spark-databricks.md)<BR>
  
### <a name="2--azure-hdinsight-spark"></a>2.  Spark de Azure HDInsight
En el siguiente artículo se contempla el servicio de Spark de Azure HDInsight, el aprovisionamiento del clúster de Azure Databricks, la configuración del clúster para conectarse a Cassandra API de Azure Cosmos DB y varios cuadernos de ejemplo que abarcan operaciones DDL, operaciones DML y mucho más.<BR>
[Uso de Cassandra API de Azure Cosmos DB desde Spark de Azure HDInsight](cassandra-spark-hdinsight.md)
 
### <a name="3--spark-environment-in-general"></a>3.  Entorno de Spark en general
Mientras que las secciones anteriores eran específicas de servicios de PaaS basados en Spark de Azure, esta sección trata cualquier entorno general de Spark.  A continuación se proporciona información detallada acerca de las dependencias del conector, las importaciones y la configuración de sesión de Spark. En la sección "Siguientes pasos" se muestran ejemplos de código para operaciones DDL, operaciones DML y mucho más.  

#### <a name="connector-dependencies"></a>Dependencias del conector:

1. Agregue las coordenadas de Maven para obtener el [conector de Cassandra para Spark](cassandra-spark-generic.md#dependencies-for-connectivity).
2. Agregue las coordenadas de Maven para la [biblioteca auxiliar de Azure Cosmos DB](cassandra-spark-generic.md#dependencies-for-connectivity) para Cassandra API.

#### <a name="imports"></a>Importaciones:

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra
```

#### <a name="spark-session-configuration"></a>Configuración de la sesión de Spark:

```scala
//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

//Throughput-related. You can adjust the values as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
//spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10") // Spark 2.x
spark.conf.set("spark.cassandra.connection.remoteConnectionsPerExecutor", "10") // Spark 3.x
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```

## <a name="next-steps"></a>Pasos siguientes

En los artículos siguientes se muestra la integración de Spark con Cassandra API de Azure Cosmos DB. 
 
* [operaciones DDL](cassandra-spark-ddl-ops.md)
* [Create/insert operations](cassandra-spark-create-ops.md) (Operaciones de creación e inserción)
* [Lee operaciones.](cassandra-spark-read-ops.md)
* [Upsert operations](cassandra-spark-upsert-ops.md) (Operaciones de upsert)
* [Delete operations](cassandra-spark-delete-ops.md) (Operaciones de eliminación)
* [Aggregation operations](cassandra-spark-aggregation-ops.md) (Operaciones de agregación)
* [Table copy operations](cassandra-spark-table-copy-ops.md) (Operaciones de copia en tabla)