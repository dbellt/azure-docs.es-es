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
ms.openlocfilehash: c6f2dfe0d4846227400ac9b3c7ac3e6ead8f0b57
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567560"
---
# <a name="analyze-data-with-a-serverless-sql-pool"></a>Análisis de datos con un grupo de SQL sin servidor

En este tutorial, aprenderá a analizar los datos con un grupo de SQL sin servidor. 

## <a name="the-built-in-serverless-sql-pool"></a>El grupo de SQL sin servidor integrado

Los grupos de SQL sin servidor permiten usar SQL sin necesidad de reservar capacidad. La facturación de un grupo de SQL sin servidor se basa en la cantidad de datos procesados para ejecutar la consulta y no en el número de recursos usados para ello.

Todas las áreas de trabajo incluyen un grupo de SQL sin servidor preconfigurado llamado **Built-in**. 

## <a name="analyze-nyc-taxi-data-with-a-serverless-sql-pool"></a>Análisis de datos de taxis de Nueva York con un grupo de SQL sin servidor

1. En Synapse Studio, vaya al menú central **Develop** (Desarrollo).
1. Se creará un nuevo script de SQL.
1. Pegue el código siguiente en el script.

    ```
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://contosolake.dfs.core.windows.net/users/NYCTripSmall.parquet',
            FORMAT='PARQUET'
        ) AS [result]
    ```
1. Haga clic en **Ejecutar**. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis de datos con un grupo de Spark sin servidor](get-started-analyze-spark.md)
