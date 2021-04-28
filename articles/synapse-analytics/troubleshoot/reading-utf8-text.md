---
title: 'Solución de problemas: Lectura de texto UTF-8 desde archivos .csv o archivos Parquet con un grupo de SQL sin servidor'
description: Lectura de texto UTF-8 desde archivos .csv o archivos Parquet con un grupo de SQL sin servidor en Azure Synapse Analytics
author: julieMSFT
ms.author: jrasnick
ms.topic: troubleshooting
ms.service: synapse-analytics
ms.subservice: troubleshooting
ms.date: 12/03/2020
ms.openlocfilehash: 956621a1a0ea909cbedbfc05bac24c21f6ccf6e8
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108141350"
---
# <a name="troubleshoot-reading-utf-8-text-from-csv-or-parquet-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Solución de problemas de lectura de texto UTF-8 desde archivos .csv o archivos Parquet con un grupo de SQL sin servidor en Azure Synapse Analytics

En este artículo se proporcionan pasos para la solución de problemas de lectura de texto UTF-8 desde archivos .csv o archivos Parquet con un grupo de SQL sin servidor en Azure Synapse Analytics.

Cuando se lee texto UTF-8 desde un archivo .csv o Parquet con el grupo de SQL sin servidor, algunos caracteres especiales, como ü y ö, se convierten incorrectamente si la consulta devuelve columnas varchar con intercalaciones que no son de UTF-8. Se trata de un problema conocido en SQL Server y Azure SQL. La intercalación que no es de UTF-8 es el valor predeterminado en Synapse SQL, por lo que las consultas de clientes se verán afectadas. Es posible que los clientes que usan caracteres estándar en inglés y algún subconjunto de caracteres latinos extendidos no perciban los errores de conversión. La conversión incorrecta se explica con más detalle en [Usar siempre intercalaciones de UTF-8 para leer texto UTF-8 en un grupo de SQL sin servidor](https://techcommunity.microsoft.com/t5/azure-synapse-analytics/always-use-utf-8-collations-to-read-utf-8-text-in-serverless-sql/ba-p/1883633).

## <a name="workaround"></a>Solución alternativa

La solución a este problema es usar siempre la intercalación UTF-8 al leer texto UTF-8 desde archivos CSV o Parquet.

- En muchos casos, solo tiene que establecer la intercalación de UTF-8 en la base de datos (operación de metadatos).

   ```sql
   alter database MyDB
         COLLATE Latin1_General_100_BIN2_UTF8;
   ```

- Puede definir explícitamente la intercalación en la columna VARCHAR de OPENROWSET o en una tabla externa:

   ```sql
   select geo_id, cases = sum(cases)
   from openrowset(
           bulk 'latest/ecdc_cases.parquet', data_source = 'covid', format = 'parquet'
       ) with ( cases int,
                geo_id VARCHAR(6) COLLATE Latin1_General_100_BIN2_UTF8 ) as rows
   group by geo_id
   ```
 
- Si no especificó la intercalación de UTF-8 en las tablas externas que leen datos de UTF-8, debe volver a crear las tablas externas afectadas y establecer la intercalación de UTF-8 en las columnas varchar (operación de metadatos).


## <a name="next-steps"></a>Pasos siguientes

* [Consulta de archivos Parquet con Synapse SQL](../sql/query-parquet-files.md)
* [Consulta de archivos CSV con Synapse SQL](../sql/query-single-csv-file.md)
* [CETAS con Synapse SQL](../sql/develop-tables-cetas.md)
* [Inicio rápido: Uso de grupos de SQL sin servidor](../quickstart-sql-on-demand.md)
