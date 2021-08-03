---
title: Consulta de archivos con el formato Delta Lake mediante un grupo de SQL sin servidor (versión preliminar)
description: En este artículo, aprenderá a consultar archivos almacenados en formato Apache Delta Lake mediante un grupo de SQL sin servidor.
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 04/27/2021
ms.author: jovanpop
ms.reviewer: jrasnick
ms.openlocfilehash: f3f6557397985f4d329fc496e5f81046eeed195b
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111440869"
---
# <a name="query-delta-lake-files-preview-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Consulta de archivos de Delta Lake (versión preliminar) mediante un grupo de SQL sin servidor en Azure Synapse Analytics

En este artículo, aprenderá a escribir una consulta mediante un grupo de Synapse SQL sin servidor para leer archivos de Apache Delta Lake.
Delta Lake es una capa de almacenamiento de código abierto que ofrece transacciones ACID (atomicidad, coherencia, aislamiento y durabilidad) para cargas de trabajo de macrodatos de Apache Spark.

El grupo de SQL sin servidor del área de trabajo de Synapse permite leer los datos almacenados en formato Delta Lake y suministrarlos a las herramientas de generación de informes. Un grupo de SQL sin servidor puede leer archivos de Delta Lake creados mediante Apache Spark, Azure Databricks o cualquier otro productor del formato Delta Lake.

Los grupos de Apache Spark de Azure Synapse permiten a los ingenieros de datos modificar los archivos de Delta Lake mediante Scala, PySpark y .NET. Los grupos de SQL sin servidor ayudan a los analistas de datos a crear informes sobre archivos de Delta Lake creados por ingenieros de datos.

[!INCLUDE [synapse-analytics-preview-features](../../../includes/synapse-analytics-preview-features.md)]

## <a name="quickstart-example"></a>Ejemplo de inicio rápido

La función [OPENROWSET](develop-openrowset.md) permite leer el contenido de los archivos de Delta Lake al proporcionar la dirección URL a la carpeta raíz.

### <a name="read-delta-lake-folder"></a>Lectura de la carpeta de Delta Lake

La forma más fácil de ver el contenido del archivo `DELTA` es proporcionar la dirección URL del archivo a la función [OPENROWSET](develop-openrowset.md) y especificar el formato `DELTA`. Si el archivo está disponible públicamente o si la identidad de Azure AD puede tener acceso a este archivo, debería poder ver su contenido mediante una consulta como la que se muestra en el ejemplo siguiente:

```sql
select top 10 *
from openrowset(
    bulk 'https://sqlondemandstorage.blob.core.windows.net/delta-lake/covid/',
    format = 'delta') as rows
```

Los nombres de columna y los tipos de datos se leen automáticamente de los archivos de Delta Lake. La función `OPENROWSET` usa los mejores tipos de suposición, como VARCHAR(1000) para las columnas de cadena.

El URI de la función `OPENROWSET` debe hacer referencia a la carpeta raíz de Delta Lake que contiene una subcarpeta denominada `_delta_log`.

> [!div class="mx-imgBorder"]
>![Carpeta ECDC COVID-19 de Delta Lake](./media/shared/covid-delta-lake-studio.png)

Si no tiene esta subcarpeta, no usa el formato Delta Lake. Puede convertir los archivos sin formato de Parquet de la carpeta al formato Delta Lake mediante el siguiente script de Python de Apache Spark:

```python
%%pyspark
from delta.tables import *
deltaTable = DeltaTable.convertToDelta(spark, "parquet.`abfss://delta-lake@sqlondemandstorage.dfs.core.windows.net/covid`")
```

Para mejorar el rendimiento de las consultas, considere la posibilidad de especificar tipos explícitos en [la ](#explicitly-specify-schema)cláusula `WITH`.

> [!NOTE]
> El grupo de Synapse SQL sin servidor usa la inferencia de esquema para determinar automáticamente las columnas y sus tipos. Las reglas de inferencia de esquema son las mismas que se usan para los archivos de Parquet.
> Para información sobre la asignación de tipos de Delta Lake al tipo nativo de SQL, consulte [Asignación de tipos para Parquet](develop-openrowset.md#type-mapping-for-parquet). 

Asegúrese de que puede acceder al archivo. Si el archivo está protegido con una clave SAS o una identidad personalizada de Azure, deberá configurar una [credencial de nivel de servidor para el inicio de sesión de SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

> [!IMPORTANT]
> Asegúrese de usar una intercalación de base de datos UTF-8 (por ejemplo, `Latin1_General_100_BIN2_UTF8`), ya que los valores de cadena de los archivos de Delta Lake se codifican con UTF-8.
> Si la codificación de texto del archivo de Delta Lake y la intercalación no coinciden, se pueden producir errores de conversión inesperados.
> Puede cambiar fácilmente la intercalación predeterminada de la base de datos actual mediante la siguiente instrucción T-SQL: `alter database current collate Latin1_General_100_BIN2_UTF8`.

### <a name="data-source-usage"></a>Uso del origen de datos

En los ejemplos anteriores se usaba la ruta de acceso completa al archivo. Como alternativa, puede crear un origen de datos externo con la ubicación que apunte a la carpeta raíz del almacenamiento. Una vez creado el origen de datos externo, use el origen de datos y la ruta de acceso relativa al archivo en la función `OPENROWSET`. De este modo, no es necesario usar el URI absoluto completo para los archivos. También puede definir credenciales personalizadas para acceder a la ubicación de almacenamiento.

> [!IMPORTANT]
> Los orígenes de datos solo se pueden crear en bases de datos personalizadas (no en la base de datos maestra o en las bases de datos replicadas desde grupos de Apache Spark). 

Para usar los ejemplos siguientes, deberá completar este paso:
1. **Cree una base de datos** con un origen de datos que haga referencia a la cuenta de almacenamiento [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). 
1. Inicialice los objetos ejecutando el [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en la base de datos que creó en el paso 1. Este script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan en estos ejemplos.

Si creó la base de datos y usó esta como contexto (mediante la instrucción `USE database_name` o la selección de la base de datos en la lista desplegable en algún editor de consultas), puede crear el origen de datos externo que contiene el URI raíz del conjunto de datos y usarlo para consultar archivos de Delta Lake:

```sql
create external data source DeltaLakeStorage
with ( location = 'https://sqlondemandstorage.blob.core.windows.net/delta-lake/' );
go

select top 10 *
from openrowset(
        bulk 'covid',
        data_source = 'DeltaLakeStorage',
        format = 'delta'
    ) as rows
```

Si un origen de datos está protegido con una clave SAS o una identidad personalizada, puede configurar el [origen de datos con una credencial de ámbito de base de datos](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Especificación explícita del esquema

`OPENROWSET` permite especificar explícitamente qué columnas desea leer del archivo con la cláusula `WITH`:

```sql
select top 10 *
from openrowset(
        bulk 'covid',
        data_source = 'DeltaLakeStorage',
        format = 'delta'
    )
    with ( date_rep date,
           cases int,
           geo_id varchar(6)
           ) as rows
```

Con la especificación explícita del esquema del conjunto de resultados, puede minimizar los tamaños de los tipos y usar los tipos más precisos VARCHAR(6) en las columnas de cadena en lugar de VARCHAR(1000) pesimista. La minimización de tipos podría mejorar considerablemente el rendimiento de las consultas.

> [!IMPORTANT]
> Asegúrese de especificar explícitamente alguna intercalación UTF-8 (por ejemplo, `Latin1_General_100_BIN2_UTF8`) para todas las columnas de cadena de la cláusula `WITH` o establezca alguna intercalación UTF-8 en el nivel de base de datos.
> La falta de coincidencia entre la codificación de texto del archivo y la intercalación de las columnas de cadena podría producir errores de conversión inesperados.
> Puede cambiar fácilmente la intercalación predeterminada de la base de datos actual mediante la siguiente instrucción T-SQL: `alter database current collate Latin1_General_100_BIN2_UTF8`.
> Puede establecer fácilmente la intercalación en los tipos de columnas mediante la siguiente definición: `geo_id varchar(6) collate Latin1_General_100_BIN2_UTF8`.

## <a name="dataset"></a>Dataset

En este ejemplo se usa el conjunto de datos [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Puede consultar los archivos Parquet de la misma manera en que [lee archivos CSV](query-parquet-files.md). La única diferencia es que el parámetro `FILEFORMAT` debe establecerse en `PARQUET`. En los ejemplos de este artículo se muestran los detalles de la lectura de archivos de Parquet.


### <a name="query-partitioned-data"></a>Consulta de datos con particiones
El conjunto de datos que se proporciona en este ejemplo se divide (particiona) en subcarpetas independientes.
A diferencia de [Parquet](query-parquet-files.md), no es necesario dirigirse a particiones específicas mediante la función `FILEPATH`. `OPENROWSET` identificará las columnas de partición en la estructura de carpetas de Delta Lake y le permitirá consultar directamente los datos mediante estas columnas. Este ejemplo muestra los importes de las tarifas por year, month y payment_type durante los tres primeros meses de 2017.

```sql
SELECT
        YEAR(pickup_datetime) AS year,
        passenger_count,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'yellow',
        DATA_SOURCE = 'DeltaLakeStorage',
        FORMAT='DELTA'
    ) nyc
WHERE
    nyc.year = 2017
    AND nyc.month IN (1, 2, 3)
    AND pickup_datetime BETWEEN CAST('1/1/2017' AS datetime) AND CAST('3/31/2017' AS datetime)
GROUP BY
    passenger_count,
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime),
    passenger_count;
```

La función `OPENROWSET` eliminará las particiones que no coincidan con los valores de `year` y `month` en la cláusula where. Esta técnica de recorte de archivos o particiones disminuirá de forma considerable el conjunto de datos, mejorará el rendimiento y reducirá el costo de la consulta.

El nombre de la carpeta en la función `OPENROWSET` (`yellow` en este ejemplo) se concatena con `LOCATION` en el origen de datos `DeltaLakeStorage` y debe hacer referencia a la carpeta raíz Delta Lake que contiene una subcarpeta denominada `_delta_log`.

> [!div class="mx-imgBorder"]
>![Carpeta Yellow Taxi de Delta Lake](./media/shared/yellow-taxi-delta-lake.png)

Si no tiene esta subcarpeta, no usa el formato Delta Lake. Puede convertir los archivos sin formato de Parquet de la carpeta al formato Delta Lake mediante el siguiente script de Python de Apache Spark:

```python
%%pyspark
from delta.tables import *
deltaTable = DeltaTable.convertToDelta(spark, "parquet.`abfss://delta-lake@sqlondemandstorage.dfs.core.windows.net/yellow`&quot;, &quot;year INT, month INT")
```

El segundo argumento de la función `DeltaTable.convertToDeltaLake` representa las columnas de partición (año y mes) que forman parte del patrón de carpeta (`year=*/month=*` en este ejemplo) y sus tipos.

## <a name="limitations"></a>Limitaciones

- La inferencia de esquemas no funciona si tiene tipos de datos complejos. En este caso, use el esquema `WITH` explícito y especifique el tipo `VARCHAR(MAX)`. 
- La función `OPENROWSET` no admite la actualización de archivos ni viajes en el tiempo de Delta Lake. Para realizar estas acciones, use el motor de Apache Spark.

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para obtener más información sobre cómo [consultar los tipos anidados de Parquet](query-parquet-nested-types.md).
Si quiere seguir creando la solución Delta Lake, aprenda a crear [vistas](create-use-views.md#delta-lake-views) o [tablas externas](create-use-external-tables.md#delta-lake-external-table) en la carpeta de Delta Lake.

## <a name="see-also"></a>Consulte también

- [¿Qué es Delta Lake?](../spark/apache-spark-what-is-delta-lake.md)
- [Aprenda a usar Delta Lake en grupos de Apache Spark para Azure Synapse Analytics](../spark/apache-spark-delta-lake-overview.md)
- [Procedimientos recomendados de Delta Lake para Azure Databricks](/azure/databricks/best-practices-index)
- [Página de documentación de Delta Lake](https://docs.delta.io/latest/delta-intro.html)
