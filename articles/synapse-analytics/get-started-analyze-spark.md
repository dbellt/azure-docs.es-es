---
title: 'Inicio rápido: Introducción al análisis con Spark'
description: En este tutorial aprenderá a analizar datos con Apache Spark.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: spark
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: de48f906f4dc86bf6297cfb3b76f406df49feec3
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363859"
---
# <a name="analyze-with-apache-spark"></a>Análisis con Apache Spark

En este tutorial, aprenderá los pasos básicos para cargar y analizar datos con Apache Spark para Azure Synapse.

## <a name="create-a-serverless-apache-spark-pool"></a>Crear un grupo de Apache Spark sin servidor

1. En Synapse Studio, en el lado izquierdo, seleccione **Administrar** > **Grupos de Apache Spark**.
1. Seleccione **Nuevo paso**. 
1. En **Nombre del grupo de Apache Spark** escriba **Spark1**.
1. En **Tamaño del nodo** escriba **Pequeño**.
1. En **Número de nodos**, establezca el valor mínimo en 3 y el máximo en 3.
1. Seleccione **Revisar y crear** > **Crear**. El grupo de Apache Spark estará listo en unos segundos.

## <a name="understanding-serverless-apache-spark-pools"></a>Descripción de los grupos de Apache Spark sin servidor

Un grupo de Spark sin servidor es una forma de indicar cómo un usuario desea trabajar con Spark. Al empezar a usar un grupo, se crea una sesión de Spark si es necesario. El grupo controla el número de recursos de Spark que se usarán en esa sesión y el tiempo que durará la sesión antes de que se pause automáticamente. El usuario paga por los recursos de Spark usados durante esa sesión, no por el grupo en sí. De esta manera, un grupo de Spark le permite trabajar con Spark, sin tener que preocuparse de la administración de clústeres. Esto es similar al funcionamiento de un grupo de SQL sin servidor.

## <a name="analyze-nyc-taxi-data-with-a-spark-pool"></a>Análisis de datos de taxis de Nueva York con un grupo de Spark

1. En Synapse Studio, vaya al menú central **Develop** (Desarrollo).
2. Crear un nuevo cuaderno
3. Cree una celda de código y pegue el código siguiente en ella.
    ```py
    %%pyspark
    df = spark.read.load('abfss://users@contosolake.dfs.core.windows.net/NYCTripSmall.parquet', format='parquet')
    display(df.limit(10))
    ```
1. En el cuaderno, en el menú **Attach to** (Asociar a), elija el grupo de Spark sin servidor **Spark1** que se creó anteriormente.
1. Seleccione **Run** (Ejecutar) en la celda. En caso de que sea necesario, Synapse iniciará una nueva sesión de Spark para ejecutar esta celda. Si se necesita una nueva sesión de Spark, tardará aproximadamente dos segundos en crearse. 
1. Si solo desea ver el esquema de la trama de datos, ejecute una celda con el siguiente código:

    ```py
    %%pyspark
    df.printSchema()
    ```

## <a name="load-the-nyc-taxi-data-into-the-spark-nyctaxi-database"></a>Carga de los datos de los taxis de Nueva York en la base de datos nyctaxi de Spark

Los datos están disponibles mediante la trama de datos denominada **df**. Cárguelos en una base de datos de Spark denominada **nyctaxi**.

1. Agregue una nueva celda de código al cuaderno y, a continuación, escriba el código siguiente:

    ```py
    %%pyspark
    spark.sql("CREATE DATABASE IF NOT EXISTS nyctaxi")
    df.write.mode("overwrite").saveAsTable("nyctaxi.trip")
    ```
## <a name="analyze-the-nyc-taxi-data-using-spark-and-notebooks"></a>Análisis de los datos de los taxis de Nueva York mediante Spark y cuadernos

1. Cree una celda de código y escriba el código siguiente. 

   ```py
   %%pyspark
   df = spark.sql("SELECT * FROM nyctaxi.trip") 
   display(df)
   ```

1. Ejecute la celda para que se muestren los datos de taxis de Nueva York que cargamos en la base de datos **nyctaxi** de Spark.
1. Cree una celda de código y escriba el código siguiente. Analizaremos estos datos y guardaremos los resultados en la tabla **nyctaxi.passengercountstats**.

   ```py
   %%pyspark
   df = spark.sql("""
      SELECT PassengerCount,
          SUM(TripDistanceMiles) as SumTripDistance,
          AVG(TripDistanceMiles) as AvgTripDistance
      FROM nyctaxi.trip
      WHERE TripDistanceMiles > 0 AND PassengerCount > 0
      GROUP BY PassengerCount
      ORDER BY PassengerCount
   """) 
   display(df)
   df.write.saveAsTable("nyctaxi.passengercountstats")
   ```

1. En los resultados de la celda, seleccione **Chart** (Gráfico) para ver los datos visualizados.


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis de datos con grupos de SQL dedicados](get-started-analyze-sql-pool.md)
