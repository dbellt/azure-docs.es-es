---
title: Creación y uso de vistas en un grupo de SQL sin servidor
description: En esta sección, aprenderá a crear y usar vistas para encapsular consultas de un grupo de SQL sin servidor. Las vistas le permitirán volver a usar esas consultas. Las vistas también son necesarias si se desea usar herramientas, como Power BI, junto con el grupo de SQL sin servidor.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 3de7a322d90f3a6a45a0965da72a1f53d5edc3a2
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109751858"
---
# <a name="create-and-use-views-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Creación y uso de vistas mediante un grupo de SQL sin servidor en Azure Synapse Analytics

En esta sección, aprenderá a crear y usar vistas para encapsular consultas de un grupo de SQL sin servidor. Las vistas le permitirán volver a usar esas consultas. Las vistas también son necesarias si se desea usar herramientas, como Power BI, junto con el grupo de SQL sin servidor.

## <a name="prerequisites"></a>Prerrequisitos

El primer paso es crear la base de datos en que se va a crear la vista e inicializar los objetos necesarios para realizar la autenticación en Azure Storage mediante la ejecución de un [script de instalación](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) en esa base de datos. Todas las consultas de este artículo se ejecutarán en la base de datos de ejemplo.

## <a name="create-a-view"></a>Creación de una vista

Las vistas se crean de la misma manera que las vistas de SQL Server normales. La siguiente consulta crea una vista que lee el archivo *population.csv*.

> [!NOTE]
> Cambie la primera línea de la consulta, es decir, [mydbname], para usar la base de datos que ha creado.

```sql
USE [mydbname];
GO

DROP VIEW IF EXISTS populationView;
GO

CREATE VIEW populationView AS
SELECT * 
FROM OPENROWSET(
        BULK 'csv/population/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', 
        FIELDTERMINATOR =',', 
        ROWTERMINATOR = '\n'
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r];
```

La vista usa un elemento `EXTERNAL DATA SOURCE` con una dirección URL raíz del almacenamiento, como `DATA_SOURCE` y agrega una ruta de acceso relativa a los archivos.

## <a name="create-a-partitioned-view"></a>Creación de una vista con particiones

Si tiene un conjunto de archivos con particiones en la estructura jerárquica de carpetas, puede describir el patrón de partición mediante los caracteres comodín en la ruta de acceso del archivo. Use la función `FILEPATH` para exponer partes de la ruta de acceso de carpeta como columnas de partición.

```sql
CREATE VIEW TaxiView
AS SELECT *, nyc.filepath(1) AS [year], nyc.filepath(2) AS [month]
FROM
    OPENROWSET(
        BULK 'parquet/taxi/year=*/month=*/*.parquet',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT='PARQUET'
    ) AS nyc
```

Las vistas con particiones realizarán la eliminación de la partición de carpeta si se consulta esta vista con los filtros de las columnas de partición. Esta solución podría mejorar el rendimiento de las consultas.

## <a name="use-a-view"></a>Uso de una vista

Puede usar vistas en las consultas de la misma manera que las utiliza en las consultas de SQL Server.

En la consulta siguiente se muestra el uso de la vista de *population_csv* que creamos en la sección [Creación de una vista](#create-a-view). Devuelve los nombres de país/región con su población en 2019, en orden descendente.

> [!NOTE]
> Cambie la primera línea de la consulta, es decir, [mydbname], para usar la base de datos que ha creado.

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM populationView
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo consultar distintos tipos de archivo, vea los artículos [Consulta de archivos .csv](query-single-csv-file.md), [Consulta de archivos Parquet](query-parquet-files.md) y [Consulta de archivos JSON](query-json-files.md).
