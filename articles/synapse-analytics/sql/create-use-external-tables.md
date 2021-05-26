---
title: Creación y uso de tablas externas en un grupo de Synapse SQL
description: En esta sección, aprenderá a crear y usar tablas externas en un grupo de Synapse SQL.
services: synapse-analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.openlocfilehash: 69b900d8695b9fa9f237411d75f0c77856f6d0eb
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378067"
---
# <a name="create-and-use-native-external-tables-using-sql-pools-in-azure-synapse-analytics"></a>Creación y uso de tablas externas nativas mediante grupos de SQL en Azure Synapse Analytics

En esta sección, aprenderá a crear y usar [tablas externas nativas](develop-tables-external-tables.md) en grupos de Synapse SQL. Las tablas externas nativas tienen un mejor rendimiento en comparación con las tablas externas con `TYPE=HADOOP` en su definición de origen de datos externo. Esto se debe a que las tablas externas nativas usan código nativo para acceder a los datos externos. 

Las tablas externas son útiles cuando se quiere controlar el acceso a los datos externos en grupos de Synapse SQL. También lo son si quiere usar herramientas, como Power BI, junto con grupos de Synapse SQL. Las tablas externas pueden acceder a dos tipos de almacenamiento:
- Almacenamiento público, en el que los usuarios acceden a archivos de almacenamiento público.
- Almacenamiento protegido, en el que los usuarios acceden a los archivos de almacenamiento mediante una credencial de SAS, una identidad de Azure AD o una identidad administrada del área de trabajo de Synapse.

> [!NOTE]
>  En grupos de SQL dedicados solo puede usar tablas externas nativas de Parquet. Las tablas externas nativas de Parquet están en versión preliminar pública limitada en los grupos de SQL dedicados porque esta característica todavía no está disponible en todas las regiones. Si quiere unirse a la versión preliminar pública para comprobar si puede usar la tabla externa nativa de Parquet en los grupos dedicados, póngase en contacto con el administrador técnico de cuentas o el arquitecto de soluciones en la nube de Microsoft. Si quiere usar la funcionalidad disponible con carácter general en grupos de SQL dedicados, o si necesita acceder a archivos CSV o ORC, use tablas externas. Las tablas externas nativas están disponibles con carácter general en grupos de SQL sin servidor.
> Más información sobre las diferencias entre las tablas externas nativas y Hadoop en [Uso de tablas externas con Synapse SQL](develop-tables-external-tables.md).

## <a name="prerequisites"></a>Prerrequisitos

El primer paso es crear la base de datos en que se crearán las tablas. Luego, se crean los siguientes objetos que se usan en este ejemplo:
- DATABASE SCOPED CREDENTIAL `sqlondemand` que permite el acceso a la cuenta de Azure Storage `https://sqlondemandstorage.blob.core.windows.net` protegida mediante SAS.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL [sqlondemand]
    WITH IDENTITY='SHARED ACCESS SIGNATURE',  
    SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
    ```

- EXTERNAL DATA SOURCE `sqlondemanddemo` que hace referencia a una cuenta de almacenamiento de demostración protegida con clave SAS y EXTERNAL DATA SOURCE `nyctlc` que hace referencia a una cuenta de Azure Storage disponible públicamente en la ubicación `https://azureopendatastorage.blob.core.windows.net/nyctlc/`.

    ```sql
    CREATE EXTERNAL DATA SOURCE SqlOnDemandDemo WITH (
        LOCATION = 'https://sqlondemandstorage.blob.core.windows.net',
        CREDENTIAL = sqlondemand
    );
    GO
    CREATE EXTERNAL DATA SOURCE nyctlc
    WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/nyctlc/')
    GO
    CREATE EXTERNAL DATA SOURCE DeltaLakeStorage
    WITH ( location = 'https://sqlondemandstorage.blob.core.windows.net/delta-lake/' );
    ```

- Los formatos de archivo `QuotedCSVWithHeaderFormat` y `ParquetFormat` que describen los tipos de archivo .csv y Parquet.

    ```sql
    CREATE EXTERNAL FILE FORMAT QuotedCsvWithHeaderFormat
    WITH (  
        FORMAT_TYPE = DELIMITEDTEXT,
        FORMAT_OPTIONS ( FIELD_TERMINATOR = ',', STRING_DELIMITER = '"', FIRST_ROW = 2   )
    );
    GO
    CREATE EXTERNAL FILE FORMAT ParquetFormat WITH (  FORMAT_TYPE = PARQUET );
    GO
    CREATE EXTERNAL FILE FORMAT DeltaLakeFormat WITH (  FORMAT_TYPE = DELTA );
    GO
    ```

Las consultas de este artículo se ejecutarán en la base de datos de ejemplo y usarán estos objetos. 

## <a name="external-table-on-a-file"></a>Tabla externa en un archivo

Puede crear tablas externas que accedan a los datos de una cuenta de Azure Storage que permita el acceso a los usuarios con una identidad de Azure AD o clave SAS. Puede crear tablas externas de la misma manera que crea tablas externas de SQL Server normales. 

La siguiente consulta crea una tabla externa que lee el archivo *population.csv* de la cuenta de Azure Storage de demostración de SynapseSQL a la que se hace referencia mediante un `sqlondemanddemo` origen de datos y que está protegida con una credencial cuyo ámbito es la base de datos llamada `sqlondemand`. 

El origen de datos y la credencial cuyo ámbito es la base de datos se crean en el [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql).

> [!NOTE]
> Cambie la primera línea de la consulta, es decir, [mydbname], para usar la base de datos que ha creado. 

```sql
USE [mydbname];
GO
CREATE EXTERNAL TABLE populationExternalTable
(
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
)
WITH (
    LOCATION = 'csv/population/population.csv',
    DATA_SOURCE = sqlondemanddemo,
    FILE_FORMAT = QuotedCSVWithHeaderFormat
);
```

Las tablas CSV nativas solo están disponibles actualmente en los grupos de SQL sin servidor.

## <a name="external-table-on-a-set-of-files"></a>Tabla externa en un conjunto de archivos

Puede crear tablas externas que lean datos de un conjunto de archivos que se encuentran en Azure Storage:

```sql
CREATE EXTERNAL TABLE Taxi (
     vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
     pickup_datetime DATETIME2, 
     dropoff_datetime DATETIME2,
     passenger_count INT,
     trip_distance FLOAT,
     fare_amount FLOAT,
     tip_amount FLOAT,
     tolls_amount FLOAT,
     total_amount FLOAT
) WITH (
         LOCATION = 'yellow/puYear=*/puMonth=*/*.parquet',
         DATA_SOURCE = nyctlc,
         FILE_FORMAT = ParquetFormat
);
```

Puede especificar el patrón que deben seguir los archivos para que la tabla externa haga referencia a estos. El patrón solo es necesario para las tablas de CSV y Parquet. Si usa el formato Delta Lake, solo es necesario especificar una carpeta raíz y la tabla externa encontrará automáticamente el patrón.

> [!NOTE]
> La tabla se crea en una estructura de carpetas con particiones, pero no se pueden eliminar algunas de estas particiones. Si desea mejorar el rendimiento omitiendo los archivos que no cumplen algunos criterios (como un año o un mes específicos), utilice las [vistas de datos externos](create-use-views.md#partitioned-views).

## <a name="delta-lake-external-table"></a>Tabla externa de Delta Lake

Las tablas externas se pueden crear encima de una carpeta de Delta Lake. La única diferencia entre las tablas externas creadas en un [único archivo](#external-table-on-a-file) o un [conjunto de archivos](#external-table-on-a-set-of-files) y las tablas externas creadas en formato Delta Lake es que en la tabla externa de Delta Lake se debe hacer referencia a una carpeta que contenga la estructura de Delta Lake.

> [!div class="mx-imgBorder"]
>![Carpeta ECDC COVID-19 de Delta Lake](./media/shared/covid-delta-lake-studio.png)

Un ejemplo de una definición de tabla creada en una carpeta de Delta Lake es:

```sql
CREATE EXTERNAL TABLE Covid (
     date_rep date,
     cases int,
     geo_id varchar(6)
) WITH (
        LOCATION = 'covid', --> the root folder containing the Delta Lake files
        data_source = DeltaLakeStorage,
        FILE_FORMAT = DeltaLakeFormat
);
```

## <a name="use-an-external-table"></a>Uso de una tabla externa

Puede usar [tablas externas](develop-tables-external-tables.md) en las consultas de la misma manera que las utiliza en las consultas de SQL Server.

En la consulta siguiente se muestra, para lo que se usa la tabla externa *population* que creamos en la sección anterior. Devuelve los nombres de país/región con su población en 2019, en orden descendente.

> [!NOTE]
> Cambie la primera línea de la consulta, es decir, [mydbname], para usar la base de datos que ha creado.

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM populationExternalTable
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

El rendimiento de esta consulta puede variar en función de la región. Es posible que el área de trabajo no se coloque en la misma región que las cuentas de almacenamiento de Azure que se usan en estos ejemplos. En el caso de las cargas de trabajo de producción, coloque el área de trabajo de Synapse y el almacenamiento de Azure en la misma región.

## <a name="next-steps"></a>Pasos siguientes

Para información sobre cómo se almacenan los resultados de una consulta en Storage, consulte el artículo [Almacenamiento de los resultados de las consultas en Storage](../sql/create-external-table-as-select.md).
