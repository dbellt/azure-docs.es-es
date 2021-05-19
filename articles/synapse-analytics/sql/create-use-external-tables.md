---
title: Creación y uso de tablas externas en un grupo de SQL sin servidor
description: En esta sección, aprenderá a crear y usar tablas externas en un grupo de SQL sin servidor.
services: synapse-analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.openlocfilehash: ea98666b38628dcf0284b4fabbf7aa239bf270df
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109751840"
---
# <a name="create-and-use-external-tables-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Creación y uso de tablas externas mediante un grupo de SQL sin servidor en Azure Synapse Analytics

En esta sección, aprenderá a crear y usar [tablas externas](develop-tables-external-tables.md) en un grupo de SQL sin servidor. Las tablas externas son útiles si se desea controlar el acceso a los datos externos en el grupo de SQL sin servidor y si se desea usar herramientas, como Power BI, junto con el grupo de SQL sin servidor. Las tablas externas pueden acceder a dos tipos de almacenamiento:
- Almacenamiento público, en el que los usuarios acceden a archivos de almacenamiento público.
- Almacenamiento protegido, en el que los usuarios acceden a los archivos de almacenamiento mediante una credencial de SAS, una identidad de Azure AD o una identidad administrada del área de trabajo de Synapse.

## <a name="prerequisites"></a>Prerrequisitos

El primer paso es crear la base de datos en que se crearán las tablas. Luego, se inicializan los objetos, para lo que haya que ejecutar un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. Este script creará los siguientes objetos que se usan en este ejemplo:
- DATABASE SCOPED CREDENTIAL `sqlondemand` que permite el acceso a la cuenta de Azure Storage `https://sqlondemandstorage.blob.core.windows.net` protegida mediante SAS.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL [sqlondemand]
    WITH IDENTITY='SHARED ACCESS SIGNATURE',  
    SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
    ```

- EXTERNAL DATA SOURCE `sqlondemanddemo` que hace referencia a una cuenta de almacenamiento de demostración protegida con clave SAS y EXTERNAL DATA SOURCE `YellowTaxi` que hace referencia a una cuenta de Azure Storage disponible públicamente en la ubicación `https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/`.

    ```sql
    CREATE EXTERNAL DATA SOURCE SqlOnDemandDemo WITH (
        LOCATION = 'https://sqlondemandstorage.blob.core.windows.net',
        CREDENTIAL = sqlondemand
    );
    GO
    CREATE EXTERNAL DATA SOURCE YellowTaxi
    WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/')
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
    ```

Las consultas de este artículo se ejecutarán en la base de datos de ejemplo y usarán estos objetos. 

## <a name="create-an-external-table-on-protected-data"></a>Creación de una tabla externa en datos protegidos

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

## <a name="create-an-external-table-on-public-data"></a>Creación de una tabla externa en datos públicos

Puede crear tablas externas que lean los datos de los archivos que se encuentran en Azure Storage disponible públicamente. Este [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) creará un origen de datos externo público y una definición de formato de archivo Parquet que se usa en la siguiente consulta:

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
         LOCATION = 'puYear=*/puMonth=*/*.parquet',
         DATA_SOURCE = YellowTaxi,
         FILE_FORMAT = ParquetFormat
);
```

> [!NOTE]
> La tabla se crea en una estructura de carpetas con particiones, pero no se pueden eliminar algunas de estas particiones. Si desea mejorar el rendimiento omitiendo los archivos que no cumplen algunos criterios (como un año o un mes específicos), utilice las [vistas de datos externos](create-use-views.md).

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

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo se almacenan los resultados de una consulta en el almacenamiento, consulte el artículo [Almacenamiento de los resultados de una consulta en Storage](../sql/create-external-table-as-select.md).
