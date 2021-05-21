---
title: 'Tutorial: Introducción al análisis de datos con un grupo de SQL sin servidor'
description: En este tutorial, aprenderá a analizar datos con un grupo de SQL sin servidor mediante los datos ubicados en bases de datos de Spark.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: tutorial
ms.date: 04/15/2021
ms.openlocfilehash: 8a8e8fae151b0d9be318d4dfad832ead34ef04da
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738093"
---
# <a name="analyze-data-with-a-serverless-sql-pool"></a>Análisis de datos con un grupo de SQL sin servidor

En este tutorial, aprenderá a analizar los datos con un grupo de SQL sin servidor. 

## <a name="the-built-in-serverless-sql-pool"></a>El grupo de SQL sin servidor integrado

Los grupos de SQL sin servidor permiten usar SQL sin necesidad de reservar capacidad. La facturación de un grupo de SQL sin servidor se basa en la cantidad de datos procesados para ejecutar la consulta y no en el número de recursos usados para ello.

Todas las áreas de trabajo incluyen un grupo de SQL sin servidor preconfigurado llamado **Built-in**. 

## <a name="analyze-nyc-taxi-data-with-a-serverless-sql-pool"></a>Análisis de datos de taxis de Nueva York con un grupo de SQL sin servidor
 
> [!NOTE]
> Asegúrese de que ha [colocado los datos de ejemplo en la cuenta de almacenamiento principal](get-started-create-workspace.md#place-sample-data-into-the-primary-storage-account).

1. En Synapse Studio, vaya al menú central **Develop** (Desarrollo).
1. Se creará un nuevo script de SQL.
1. Pegue el código siguiente en el script.

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://contosolake.dfs.core.windows.net/users/NYCTripSmall.parquet',
            FORMAT='PARQUET'
        ) AS [result]
    ```
1. Haga clic en **Ejecutar**. 

La exploración de datos es simplemente un escenario simplificado en el que puede comprender las características básicas de los datos. Aprenda más sobre la exploración y el análisis de datos en este [tutorial](sql/tutorial-data-analyst.md).

## <a name="create-data-exploration-database"></a>Creación de una base de datos de exploración de datos

Puede examinar el contenido de los archivos directamente mediante la base de datos `master`. En algunos escenarios sencillos de exploración de datos, no es necesario crear una base de datos independiente.
Sin embargo, a medida que continúa la exploración de datos, puede ser conveniente crear algunos objetos de utilidad, como por ejemplo:
- Orígenes de datos externos que representan las referencias con nombre de las cuentas de almacenamiento.
- Credenciales con ámbito de base de datos que permiten especificar cómo autenticarse en un origen de datos externo.
- Usuarios de base de datos con permisos para acceder a algunos orígenes de datos u objetos de base de datos.
- Vistas, procedimientos y funciones de la utilidad que puede usar en las consultas.

1. Cree una base de datos independiente para usar estos objetos. No se pueden crear objetos de base de datos personalizados en la base de datos `master`.

    ```sql
    CREATE DATABASE DataExplorationDB 
                    COLLATE Latin1_General_100_BIN2_UTF8
    ```

   > [!IMPORTANT]
   > Use una intercalación con sufijo `_UTF8` para que el texto UTF-8 se convierta correctamente en columnas `VARCHAR`. `Latin1_General_100_BIN2_UTF8` proporciona el mejor rendimiento en las consultas que leen datos de archivos Parquet y contenedores de Cosmos DB.

2. Cambie a `DataExplorationDB`, donde puede crear objetos de utilidad, como credenciales y orígenes de datos.

    ```sql
    CREATE EXTERNAL DATA SOURCE ContosoLake
    WITH ( LOCATION = 'https://contosolake.dfs.core.windows.net')
    ```

   > [!NOTE]
   > Se puede crear un origen de datos externo sin credencial. En ese caso, la identidad del autor de la llamada se usará para acceder al origen de datos externo.

3. También puede crear un inicio de sesión para un usuario en `DataExplorationDB` para acceder a datos externos:

    ```sql
    CREATE LOGIN data_explorer WITH PASSWORD = 'My Very Strong Password 1234!';
    ```

    Cree un usuario de base de datos en `DataExplorationDB` para el inicio de sesión y conceda el permiso `ADMINISTER DATABASE BULK OPERATIONS`.
    ```sql
    CREATE USER data_explorer FOR LOGIN data_explorer;
    GO
    GRANT ADMINISTER DATABASE BULK OPERATIONS TO data_explorer;
    GO
    ```

4. Explore el contenido del archivo mediante la ruta de acceso relativa y el origen de datos:

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
                BULK '/users/NYCTripSmall.parquet',
                DATA_SOURCE = 'ContosoLake',
                FORMAT='PARQUET'
        ) AS [result]
    ```

La base de datos de exploración de datos es solamente un simple marcador de posición donde puede almacenar los objetos de utilidad. El grupo de Synapse SQL le permite hacer mucho más y crear una instancia lógica de Data Warehouse: una capa relacional basada en orígenes de datos de Azure. Aprenda más sobre la creación de instancias lógicas de Data Warehouse en este [tutorial](sql/tutorial-data-analyst.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis de datos con un grupo de Spark sin servidor](get-started-analyze-spark.md)
