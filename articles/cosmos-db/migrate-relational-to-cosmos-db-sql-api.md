---
title: Migración de datos relacionales de uno a varios a la API de SQL de Azure Cosmos DB
description: Obtenga información sobre cómo administrar la migración de datos complejos para relaciones de uno a varios en la API de SQL
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 12/12/2019
ms.author: thvankra
ms.openlocfilehash: 7dbb162749e0a2f84140b8e9394934198d096eac
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589625"
---
# <a name="migrate-one-to-few-relational-data-into-azure-cosmos-db-sql-api-account"></a>Migración de datos relacionales de uno a varios a una cuenta de la API de SQL de Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Para migrar de una base de datos relacional a la API de SQL de Azure Cosmos DB, puede que necesite modificar el modelo de datos para optimizarlo.

Una transformación común es la desnormalización de los datos al insertar subelementos relacionados en un solo documento JSON. A continuación se describen algunas opciones para realizar el proceso con Azure Data Factory o Azure Databricks. Para obtener instrucciones generales sobre el modelado de datos para Cosmos DB, consulte [Modelado de datos en Azure Cosmos DB](modeling-data.md).  

## <a name="example-scenario"></a>Escenario de ejemplo

Supongamos que existen las dos tablas siguientes en una base de datos SQL: Orders y OrderDetails.


:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/orders.png" alt-text="Captura de pantalla que muestra las tablas Orders y OrderDetails en la base de datos SQL." border="false" :::

Se quiere combinar esta relación de uno a varios en un documento JSON durante la migración. Para ello, se puede crear una consulta T-SQL con "FOR JSON" como se indica a continuación:

```sql
SELECT
  o.OrderID,
  o.OrderDate,
  o.FirstName,
  o.LastName,
  o.Address,
  o.City,
  o.State,
  o.PostalCode,
  o.Country,
  o.Phone,
  o.Total,
  (select OrderDetailId, ProductId, UnitPrice, Quantity from OrderDetails od where od.OrderId = o.OrderId for json auto) as OrderDetails
FROM Orders o;
```

Los resultados de esta consulta tendrán el siguiente aspecto: 

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/for-json-query-result.png" alt-text="Detalles de pedido" lightbox="./media/migrate-relational-to-cosmos-sql-api/for-json-query-result.png":::

Lo ideal sería usar una única actividad de copia de Azure Data Factory (ADF) para consultar datos SQL como origen y escribir la salida directamente en el receptor de Azure Cosmos DB como objetos JSON adecuados. Actualmente, no es posible realizar la transformación de JSON necesaria en una actividad de copia. Si se intenta copiar los resultados de la consulta anterior en un contenedor de la API de SQL de Azure Cosmos DB, el campo OrderDetails se mostrará como una propiedad de cadena de nuestro documento, en lugar de ser la matriz JSON esperada.

Esta limitación actual se puede solucionar de una de las siguientes maneras:

* **Usar Azure Data Factory con dos actividades de copia**: 
  1. Almacene los datos con formato JSON de SQL en un archivo de texto en una ubicación de almacenamiento de blobs intermedia, y 
  2. Cargue los datos del archivo de texto JSON en un contenedor en Azure Cosmos DB.

* **Usar Azure Databricks para leer de SQL y escribir en Azure Cosmos DB**: en este caso, se mostrarán dos opciones.


Echemos un vistazo a estos métodos con más detalle:

## <a name="azure-data-factory"></a>Azure Data Factory

Aunque no se puede insertar OrderDetails como una matriz JSON en el documento de Cosmos DB de destino, se puede solucionar el problema al usar dos actividades de copia independientes.

### <a name="copy-activity-1-sqljsontoblobtext"></a>Actividad de copia n.º 1: SqlJsonToBlobText

En el caso de los datos de origen, se usa una consulta SQL para obtener el conjunto de resultados como una sola columna con un objeto JSON por fila (que representa al pedido) mediante las funciones OPENJSON y FOR JSON PATH de SQL Server:

```sql
SELECT [value] FROM OPENJSON(
  (SELECT
    id = o.OrderID,
    o.OrderDate,
    o.FirstName,
    o.LastName,
    o.Address,
    o.City,
    o.State,
    o.PostalCode,
    o.Country,
    o.Phone,
    o.Total,
    (select OrderDetailId, ProductId, UnitPrice, Quantity from OrderDetails od where od.OrderId = o.OrderId for json auto) as OrderDetails
   FROM Orders o FOR JSON PATH)
)
```

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf1.png" alt-text="Copia de ADF":::


En el caso del receptor de la actividad de copia SqlJsonToBlobText, se selecciona "Texto delimitado" y se apunta a una carpeta específica de Azure Blob Storage con un nombre de archivo único generado de forma dinámica (por ejemplo, "@concat(pipeline().RunId,".json").
Dado que el archivo de texto no está realmente "delimitado" y no queremos que se analice en columnas independientes mediante comas y, además, queremos conservar las comillas dobles ("), el "Delimitador de columna" se establece en una tabulación ("\t") u otro carácter que no aparece en los datos y el carácter de comillas se establece en "Sin carácter de comillas".

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf2.png" alt-text="Captura de pantalla que resalta las configuraciones Delimitador de columna y Carácter de comillas.":::

### <a name="copy-activity-2-blobjsontocosmos"></a>Actividad de copia n.º 2: BlobJsonToCosmos

A continuación, se modifica la canalización de ADF al agregar la segunda actividad de copia que busca en Azure Blob Storage al archivo de texto que creó la primera actividad. Este se procesa como un origen "JSON" a finde insertarse en el receptor de Cosmos DB como un documento por cada fila JSON encontrada en el archivo de texto.

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf3.png" alt-text="Captura de pantalla que resalta el archivo de origen JSON y los campos Ruta de acceso de archivo.":::

También se puede agregar una actividad "Eliminar" a la canalización para que elimine todos los archivos anteriores que queden en la carpeta /Orders/ antes de cada ejecución. Nuestra canalización de ADF ahora tiene un aspecto similar al siguiente:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf4.png" alt-text="Captura de pantalla en la que se resalta la actividad de eliminación.":::

Después de desencadenar la canalización anterior, se crea un archivo en la ubicación intermedia de Azure Blob Storage que contiene un objeto JSON por cada fila:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf5.png" alt-text="Captura de pantalla que muestra el archivo creado que contiene los objetos JSON.":::

También se muestran los documentos de Orders con OrderDetails insertados correctamente en nuestra colección de Cosmos DB:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf6.png" alt-text="Captura de pantalla que muestra los detalles del pedido como parte del documento de Cosmos DB":::


## <a name="azure-databricks"></a>Azure Databricks

También se puede usar Spark en [Azure Databricks](https://azure.microsoft.com/services/databricks/) para copiar los datos del origen de SQL Database al destino de Azure Cosmos DB sin crear los archivos de texto o JSON intermedios en Azure Blob Storage. 

> [!NOTE]
> Para mayor claridad y simplicidad, los fragmentos de código siguientes incluyen contraseñas de base de datos ficticias insertadas explícitamente. Sin embargo, siempre debe usar secretos de Azure Databricks.
>

En primer lugar, se crean y se conectan las bibliotecas del [conector de SQL](/connectors/sql/) y del [conector de Azure Cosmos DB](https://docs.databricks.com/data/data-sources/azure/cosmosdb-connector.html) necesarias en nuestro clúster de Azure Databricks. Reinicie el clúster para asegurarse de las bibliotecas se carguen.

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks1.png" alt-text="Captura de pantalla que muestra dónde crear y conectar las bibliotecas de conectores de SQL y conectores de Azure Cosmos DB necesarias en nuestro clúster de Azure Databricks.":::

A continuación, se incluyen dos ejemplos; uno para Scala y otro para Python. 

### <a name="scala"></a>Scala
En este caso, se obtienen los resultados de la consulta SQL con la salida "FOR JSON" en un objeto DataFrame:

```scala
// Connect to Azure SQL /connectors/sql/
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._
val configSql = Config(Map(
  "url"          -> "xxxx.database.windows.net",
  "databaseName" -> "xxxx",
  "queryCustom"  -> "SELECT o.OrderID, o.OrderDate, o.FirstName, o.LastName, o.Address, o.City, o.State, o.PostalCode, o.Country, o.Phone, o.Total, (SELECT OrderDetailId, ProductId, UnitPrice, Quantity FROM OrderDetails od WHERE od.OrderId = o.OrderId FOR JSON AUTO) as OrderDetails FROM Orders o",
  "user"         -> "xxxx", 
  "password"     -> "xxxx" // NOTE: For clarity and simplicity, this example includes secrets explicitely as a string, but you should always use Databricks secrets
))
// Create DataFrame from Azure SQL query
val orders = sqlContext.read.sqlDB(configSql)
display(orders)
```

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks2.png" alt-text="Captura de pantalla que muestra el resultado de la consulta SQL en un DataFrame.":::

A continuación, se establece una conexión con la base de datos y la colección de Cosmos DB:

```scala
// Connect to Cosmos DB https://docs.databricks.com/data/data-sources/azure/cosmosdb-connector.html
import org.joda.time._
import org.joda.time.format._
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark.CosmosDBSpark
import com.microsoft.azure.cosmosdb.spark.config.Config
import org.apache.spark.sql.functions._
import org.joda.time._
import org.joda.time.format._
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark.CosmosDBSpark
import com.microsoft.azure.cosmosdb.spark.config.Config
import org.apache.spark.sql.functions._
val configMap = Map(
  "Endpoint" -> "https://xxxx.documents.azure.com:443/",
  // NOTE: For clarity and simplicity, this example includes secrets explicitely as a string, but you should always use Databricks secrets
  "Masterkey" -> "xxxx",
  "Database" -> "StoreDatabase",
  "Collection" -> "Orders")
val configCosmos = Config(configMap)
```

Por último, se define el esquema y se usa from_json para aplicar el objeto DataFrame antes de guardarlo en la colección de CosmosDB.

```scala
// Convert DataFrame to proper nested schema
import org.apache.spark.sql.types._
val orderDetailsSchema = ArrayType(StructType(Array(
    StructField("OrderDetailId",IntegerType,true),
    StructField("ProductId",IntegerType,true),
    StructField("UnitPrice",DoubleType,true),
    StructField("Quantity",IntegerType,true)
  )))
val ordersWithSchema = orders.select(
  col("OrderId").cast("string").as("id"),
  col("OrderDate").cast("string"),
  col("FirstName").cast("string"),
  col("LastName").cast("string"),
  col("Address").cast("string"),
  col("City").cast("string"),
  col("State").cast("string"),
  col("PostalCode").cast("string"),
  col("Country").cast("string"),
  col("Phone").cast("string"),
  col("Total").cast("double"),
  from_json(col("OrderDetails"), orderDetailsSchema).as("OrderDetails")
)
display(ordersWithSchema)
// Save nested data to Cosmos DB
CosmosDBSpark.save(ordersWithSchema, configCosmos)
```

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks3.png" alt-text="Captura de pantalla que resalta la matriz adecuada para guardar en una colección de Cosmos DB.":::


### <a name="python"></a>Python

Como método alternativo, es posible que deba ejecutar transformaciones JSON en Spark (si la base de datos de origen no admite "FOR JSON" o una operación similar), o que quiera usar operaciones en paralelo para un conjunto de datos muy grande. El siguiente es un ejemplo de PySpark. Empiece por configurar las conexiones de base de datos de origen y de destino en la primera celda:

```python
import uuid
import pyspark.sql.functions as F
from pyspark.sql.functions import col
from pyspark.sql.types import StringType,DateType,LongType,IntegerType,TimestampType
 
#JDBC connect details for SQL Server database
jdbcHostname = "jdbcHostname"
jdbcDatabase = "OrdersDB"
jdbcUsername = "jdbcUsername"
jdbcPassword = "jdbcPassword"
jdbcPort = "1433"
 
connectionProperties = {
  "user" : jdbcUsername,
  "password" : jdbcPassword,
  "driver" : "com.microsoft.sqlserver.jdbc.SQLServerDriver"
}
jdbcUrl = "jdbc:sqlserver://{0}:{1};database={2};user={3};password={4}".format(jdbcHostname, jdbcPort, jdbcDatabase, jdbcUsername, jdbcPassword)
 
#Connect details for Target Azure Cosmos DB SQL API account
writeConfig = {
    "Endpoint": "Endpoint",
    "Masterkey": "Masterkey",
    "Database": "OrdersDB",
    "Collection": "Orders",
    "Upsert": "true"
}
```

A continuación, se consultará la base de datos de origen (en este caso, SQL Server) para recuperar los registros de Order y OrderDetails y los resultados se colocarán en objetos DataFrame de Spark. También se creará una lista que contenga todos los Id. de pedido y un grupo de subprocesos para las operaciones en paralelo:

```python
import json
import ast
import pyspark.sql.functions as F
import uuid
import numpy as np
import pandas as pd
from functools import reduce
from pyspark.sql import SQLContext
from pyspark.sql.types import *
from pyspark.sql import *
from pyspark.sql.functions import exp
from pyspark.sql.functions import col
from pyspark.sql.functions import lit
from pyspark.sql.functions import array
from pyspark.sql.types import *
from multiprocessing.pool import ThreadPool
 
#get all orders
orders = sqlContext.read.jdbc(url=jdbcUrl, table="orders", properties=connectionProperties)
 
#get all order details
orderdetails = sqlContext.read.jdbc(url=jdbcUrl, table="orderdetails", properties=connectionProperties)
 
#get all OrderId values to pass to map function 
orderids = orders.select('OrderId').collect()
 
#create thread pool big enough to process merge of details to orders in parallel
pool = ThreadPool(10)
```

A continuación, cree una función para escribir los elementos de Order en la colección de la API de SQL de destino. Esta función filtrará todos los detalles del pedido para un Id. de pedido específico, los convertirá en una matriz JSON e insertará la matriz en un documento JSON que se escribirá en la colección de la API de SQL de destino de ese pedido:

```python
def writeOrder(orderid):
  #filter the order on current value passed from map function
  order = orders.filter(orders['OrderId'] == orderid[0])
  
  #set id to be a uuid
  order = order.withColumn("id", lit(str(uuid.uuid1())))
  
  #add details field to order dataframe
  order = order.withColumn("details", lit(''))
  
  #filter order details dataframe to get details we want to merge into the order document
  orderdetailsgroup = orderdetails.filter(orderdetails['OrderId'] == orderid[0])
  
  #convert dataframe to pandas
  orderpandas = order.toPandas()
  
  #convert the order dataframe to json and remove enclosing brackets
  orderjson = orderpandas.to_json(orient='records', force_ascii=False)
  orderjson = orderjson[1:-1] 
  
  #convert orderjson to a dictionaory so we can set the details element with order details later
  orderjsondata = json.loads(orderjson)
  
  
  #convert orderdetailsgroup dataframe to json, but only if details were returned from the earlier filter
  if (orderdetailsgroup.count() !=0):
    #convert orderdetailsgroup to pandas dataframe to work better with json
    orderdetailsgroup = orderdetailsgroup.toPandas()
    
    #convert orderdetailsgroup to json string
    jsonstring = orderdetailsgroup.to_json(orient='records', force_ascii=False)
    
    #convert jsonstring to dictionary to ensure correct encoding and no corrupt records
    jsonstring = json.loads(jsonstring)
    
    #set details json element in orderjsondata to jsonstring which contains orderdetailsgroup - this merges order details into the order 
    orderjsondata['details'] = jsonstring
 
  #convert dictionary to json
  orderjsondata = json.dumps(orderjsondata)
 
  #read the json into spark dataframe
  df = spark.read.json(sc.parallelize([orderjsondata]))
  
  #write the dataframe (this will be a single order record with merged many-to-one order details) to cosmos db using spark the connector
  #https://docs.microsoft.com/azure/cosmos-db/spark-connector
  df.write.format("com.microsoft.azure.cosmosdb.spark").mode("append").options(**writeConfig).save()
```

Por último, se llamará a la función anterior mediante una función de asignación en el grupo de subprocesos para que se ejecute en paralelo, pasando la lista de Id. de pedido que se ha creado anteriormente:

```python
#map order details to orders in parallel using the above function
pool.map(writeOrder, orderids)
```
En cualquiera de estos métodos, al final, debe haber objetos OrderDetails insertados y guardados correctamente en cada documento Order de la colección de Cosmos DB:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks4.png" alt-text="Databricks":::

## <a name="next-steps"></a>Pasos siguientes
* Más información sobre el [modelado de datos en Azure Cosmos DB](./modeling-data.md)
* Más información sobre [cómo modelar y crear particiones de datos en Azure Cosmos DB](./how-to-model-partition-example.md)
