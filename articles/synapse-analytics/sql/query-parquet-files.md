---
title: Consulta de archivos de Parquet mediante un grupo de SQL sin servidor
description: En este artículo, aprenderá a consultar archivos de Parquet mediante un grupo de SQL sin servidor.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 72bf8e76217e8a3bcecd381d8d61815c539dd196
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107209594"
---
# <a name="query-parquet-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Consulta de archivos de Parquet mediante un grupo de SQL sin servidor en Azure Synapse Analytics

En este artículo, aprenderá a escribir una consulta mediante un grupo de SQL sin servidor que leerá archivos de Parquet.

## <a name="quickstart-example"></a>Ejemplo de inicio rápido

La función `OPENROWSET` permite leer el contenido del archivo Parquet al proporcionar la dirección URL al archivo.

### <a name="read-parquet-file"></a>Lectura del archivo de Parquet

La forma más fácil de ver el contenido del archivo `PARQUET` es proporcionar la dirección URL del archivo a la función `OPENROWSET` y especificar el `FORMAT` de Parquet. Si el archivo está disponible públicamente o si la identidad de Azure AD puede tener acceso a este archivo, debería poder ver el contenido del archivo mediante la consulta como la que se muestra en el ejemplo siguiente:

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
    format = 'parquet') as rows
```

Asegúrese de que puede tener acceso a este archivo. Si el archivo está protegido con una clave SAS o una identidad personalizada de Azure, necesitaría configurar una [credencial de nivel de servidor para el inicio de sesión de SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

> [!IMPORTANT]
> Asegúrese de usar alguna intercalación de base de datos UTF-8 (por ejemplo, `Latin1_General_100_BIN2_UTF8`) porque los valores de cadena de los archivos de Parquet se codifican como UTF-8.
> Una falta de coincidencia entre la codificación de texto del archivo de Parquet y la intercalación puede producir errores de conversión inesperados.
> Puede cambiar fácilmente la intercalación predeterminada de la base de datos actual mediante la siguiente instrucción T-SQL: `alter database current collate Latin1_General_100_BIN2_UTF8`.

### <a name="data-source-usage"></a>Uso del origen de datos

En el ejemplo anterior se usa la ruta de acceso completa al archivo. Como alternativa, puede crear un origen de datos externo con la ubicación que apunta a la carpeta raíz del almacenamiento y usar ese origen de datos y la ruta de acceso relativa al archivo en la función `OPENROWSET`:

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) as rows
```

Si un origen de datos está protegido con una clave SAS o una identidad personalizada, puede configurar el [origen de datos con una credencial de ámbito de base de datos](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Especificación explícita del esquema

`OPENROWSET` permite especificar explícitamente qué columnas desea leer del archivo con la cláusula `WITH`:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) with ( date_rep date, cases int, geo_id varchar(6) ) as rows
```

> [!IMPORTANT]
> Asegúrese de especificar explícitamente alguna intercalación UTF-8 (por ejemplo, `Latin1_General_100_BIN2_UTF8`) para todas las columnas de cadena de la cláusula `WITH` o establezca alguna intercalación UTF-8 en el nivel de base de datos.
> La falta de coincidencia entre la codificación de texto del archivo y la intercalación de las columnas de cadena podría producir errores de conversión inesperados.
> Puede cambiar fácilmente la intercalación predeterminada de la base de datos actual mediante la siguiente instrucción T-SQL: `alter database current collate Latin1_General_100_BIN2_UTF8`.
> Puede establecer fácilmente la intercalación en los tipos de columnas mediante la siguiente definición: `geo_id varchar(6) collate Latin1_General_100_BIN2_UTF8`.

En las secciones siguientes, puede ver cómo consultar varios tipos de archivos PARQUET.

## <a name="prerequisites"></a>Prerrequisitos

El primer paso consiste en **crear una base de datos** con un origen de datos que haga referencia a la cuenta de almacenamiento de [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Luego, se inicializan los objetos, para lo que hay que ejecutar un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. Este script de instalación creará los orígenes de datos, las credenciales con ámbito de base de datos y los formatos de archivo externos que se usan en estos ejemplos.

## <a name="dataset"></a>Dataset

En este ejemplo se usa el conjunto de datos [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Puede consultar los archivos Parquet de la misma manera en que [lee archivos CSV](query-parquet-files.md). La única diferencia es que el parámetro `FILEFORMAT` debe establecerse en `PARQUET`. En los ejemplos de este artículo se muestran los detalles de la lectura de archivos de Parquet.

## <a name="query-set-of-parquet-files"></a>Consulta de conjunto de archivos de Parquet

Solo puede especificar las columnas de interés al consultar los archivos de Parquet.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) WITH (
        tpepPickupDateTime DATETIME2,
        passengerCount INT
    ) AS nyc
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="automatic-schema-inference"></a>Inferencia automática del esquema

No es necesario usar la cláusula OPENROWSET WITH al leer los archivos de Parquet. Los nombres de columna y los tipos de datos se leen automáticamente desde los archivos de Parquet.

En el ejemplo siguiente se muestran las funcionalidades de inferencia automática del esquema para los archivos de Parquet. Devuelve el número de filas de septiembre de 2018 sin especificar un esquema.

> [!NOTE]
> No es necesario especificar columnas en la cláusula OPENROWSET WITH al leer los archivos de Parquet. En ese caso, el servicio de consulta del grupo de SQL sin servidor utilizará los metadatos del archivo de Parquet y enlazará las columnas por el nombre.

```sql
SELECT TOP 10 *
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=9/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) AS nyc
```

### <a name="query-partitioned-data"></a>Consulta de datos con particiones

El conjunto de datos que se proporciona en este ejemplo se divide (particiona) en subcarpetas independientes. Puede dirigirse a particiones específicas mediante la función filepath. Este ejemplo muestra los importes de las tarifas por year, month y payment_type durante los tres primeros meses de 2017.

> [!NOTE]
> La consulta del grupo de SQL sin servidor es compatible con el esquema de partición de Hive y Hadoop.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=*/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) nyc
WHERE
    nyc.filepath(1) = 2017
    AND nyc.filepath(2) IN (1, 2, 3)
    AND tpepPickupDateTime BETWEEN CAST('1/1/2017' AS datetime) AND CAST('3/31/2017' AS datetime)
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="type-mapping"></a>Asignación de tipos

Para la asignación de tipos de Parquet al tipo nativo de SQL, consulte la [asignación de tipos para Parquet](develop-openrowset.md#type-mapping-for-parquet).

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para obtener más información sobre cómo [consultar los tipos anidados de Parquet](query-parquet-nested-types.md).
