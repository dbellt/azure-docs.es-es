---
title: 'Tutorial: Introducción al análisis de datos en cuentas de almacenamiento'
description: En este tutorial, aprenderá a analizar los datos ubicados en una cuenta de almacenamiento.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: workspace
ms.topic: tutorial
ms.date: 12/31/2020
ms.openlocfilehash: 6b88a7e6a9851018fce255fac0e39a30563b9bf4
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363842"
---
# <a name="analyze-data-in-a-storage-account"></a>Análisis de datos en una cuenta de almacenamiento

En este tutorial, aprenderá a analizar los datos ubicados en una cuenta de almacenamiento.

## <a name="overview"></a>Información general

Hasta ahora, hemos descrito escenarios en los que los datos residían en las bases de datos del área de trabajo. Ahora le mostraremos cómo trabajar con archivos que residen en cuentas de almacenamiento. En este escenario, se usará la cuenta de almacenamiento principal del área de trabajo y el contenedor que se especificó al crear el área de trabajo.

* El nombre de la cuenta de almacenamiento: **contosolake**
* El nombre del contenedor en la cuenta de almacenamiento: **users**

### <a name="create-csv-and-parquet-files-in-your-storage-account"></a>Creación de archivos CSV y Parquet en la cuenta de almacenamiento

Ejecute el siguiente código en una nueva celda de código de un cuaderno. Crea un archivo .csv y un archivo Parquet en la cuenta de almacenamiento.

```py
%%pyspark
df = spark.sql("SELECT * FROM nyctaxi.passengercountstats")
df = df.repartition(1) # This ensure we'll get a single file during write()
df.write.mode("overwrite").csv("/NYCTaxi/PassengerCountStats_csvformat")
df.write.mode("overwrite").parquet("/NYCTaxi/PassengerCountStats_parquetformat")
```

### <a name="analyze-data-in-a-storage-account"></a>Análisis de datos en una cuenta de almacenamiento

Puede analizar los datos de la cuenta de ADLS Gen2 predeterminada del área de trabajo o vincular una cuenta de ADLS Gen2 o de almacenamiento de blobs con el área de trabajo seleccionando "**Administrar**" > "**Servicios vinculados**" > "**Nuevo**" (en los pasos siguientes, se ha utilizado la cuenta de ADLS Gen2 principal).

1. En Synapse Studio, vaya al centro **Data** (Datos) y, a continuación, seleccione **Vinculado**.
1. Vaya a **Azure Data Lake Storage Gen2** > **myworkspace (Primary - contosolake)** [miáreadetrabajo (Principal: contosolake)].
1. Seleccione **users (Primary)** (usuarios [Principal]). Debería ver la carpeta **NYCTaxi**. Dentro debería ver dos carpetas llamadas **PassengerCountStats_csvformat** y **PassengerCountStats_parquetformat**.
1. Abra la carpeta **PassengerCountStats_parquetformat**. Dentro, verá un archivo Parquet con un nombre como `part-00000-2638e00c-0790-496b-a523-578da9a15019-c000.snappy.parquet`.
1. Haga clic con el botón derecho en el archivo **.parquet**, seleccione **Nuevo cuaderno** y, luego, **Load to DataFrame** (Cargar en Dataframe). Se crea un cuaderno con una celda como esta:

    ```py
    %%pyspark
    abspath = 'abfss://users@contosolake.dfs.core.windows.net/NYCTaxi/PassengerCountStats.parquet/part-00000-1f251a58-d8ac-4972-9215-8d528d490690-c000.snappy.parquet'
    df = spark.read.load(abspath, format='parquet')
    display(df.limit(10))
    ```

1. Asócielo al grupo de Spark llamado **Spark1**. Ejecute la celda.
1. Vuelva a seleccionar la carpeta **users**. De nuevo, haga clic con el botón derecho en el archivo **.parquet** y, luego, seleccione **New SQL Script** > **SELECT TOP 100 rows** (Nuevo script SQL > Seleccionar 100 primeras filas). Se crea un script SQL similar al siguiente:

    ```sql
    SELECT 
        TOP 100 *
    FROM OPENROWSET(
        BULK 'https://contosolake.dfs.core.windows.net/users/NYCTaxi/PassengerCountStats.parquet/part-00000-1f251a58-d8ac-4972-9215-8d528d490690-c000.snappy.parquet',
        FORMAT='PARQUET'
    ) AS [result]
    ```

    En la ventana de script, asegúrese de que el campo **Conectarse a** esté establecido en el grupo de SQL sin servidor **Integrado**.

1. Ejecute el script.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Organización de actividades con canalizaciones](get-started-pipelines.md)
