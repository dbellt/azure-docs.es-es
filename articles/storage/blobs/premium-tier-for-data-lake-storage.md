---
title: Nivel Premium para Azure Data Lake Storage | Microsoft Docs
description: Use el nivel de rendimiento Premium con Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.custom: references_regions
ms.date: 10/30/2020
ms.author: normesta
ms.openlocfilehash: a9ed305dbd081486f0b892fcc38ada83e9666bd9
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143348"
---
# <a name="premium-tier-for-azure-data-lake-storage"></a>Nivel Premium para Azure Data Lake Storage

Azure Data Lake Storage Gen2 ahora admite el [nivel de rendimiento Premium](storage-blob-performance-tiers.md#premium-performance). El nivel de rendimiento Premium es ideal para aplicaciones de análisis de macrodatos y cargas de trabajo que requieren una latencia baja uniforme y tienen un gran número de transacciones.

## <a name="workloads-that-can-benefit-from-the-premium-performance-tier"></a>Cargas de trabajo que pueden beneficiarse del nivel de rendimiento Premium

Entre las cargas de trabajo de ejemplo se incluyen cargas de trabajo interactivas, IoT, análisis de streaming, inteligencia artificial y aprendizaje automático. 

**Cargas de trabajo interactivas** 

Estas cargas de trabajo requieren actualizaciones instantáneas y comentarios de los usuarios, como es el caso de las aplicaciones de comercio electrónico y de mapas, aplicaciones de vídeo interactivas, etc. Por ejemplo, en una aplicación de comercio electrónico, es probable que los elementos que se ven con menos frecuencia no estén almacenados en caché. Sin embargo, deben mostrarse al cliente a petición de forma instantánea. En otro ejemplo, los científicos de datos, analistas y desarrolladores pueden obtener información sujeta a limitaciones temporales incluso más rápido mediante la ejecución de consultas en los datos almacenados en una cuenta que use el nivel de rendimiento Premium. 

**IoT/análisis de streaming** 

En un escenario de IoT, se podría insertar un gran número de operaciones más pequeñas en la nube cada segundo. Se podrían ingerir grandes cantidades de datos, agregarlos con fines de análisis y, finalmente, eliminarlos de forma casi inmediata. Las funcionalidades de alto nivel de ingesta del nivel de rendimiento Premium lo hacen especialmente eficaz para este tipo de carga de trabajo. 

**Inteligencia artificial y aprendizaje automático (IA y ML)** 

Este tema trata sobre el uso y procesamiento de diferentes tipos de datos como objetos visuales, voz y texto. Este tipo de informática de alto rendimiento de la carga de trabajo se ocupa de grandes cantidades de datos que requieren una respuesta rápida y tiempos de ingesta eficientes para el análisis de datos. 

## <a name="cost-effectiveness"></a>Rentabilidad

El nivel de rendimiento Premium tiene un costo de almacenamiento mayor, pero un costo de transacción menor en comparación con el nivel de rendimiento Estándar. Si las aplicaciones y las cargas de trabajo ejecutan un gran número de transacciones, el nivel de rendimiento Premium puede ser rentable.

En la tabla siguiente se muestra la rentabilidad del nivel Premium para Azure Data Lake Storage. Cada columna representa el número de transacciones en un mes.  Cada fila representa el porcentaje de transacciones que son de lectura. Cada celda de la tabla muestra el porcentaje de reducción de costos asociado a un porcentaje de transacciones de lectura y al número de transacciones ejecutadas. 

Por ejemplo, suponiendo que su cuenta se encuentra en la región Este de EE. UU. 2, el número de transacciones con su cuenta supera los 90 millones, y el 70 % de esas transacciones son de lectura, el nivel de rendimiento Premium es más rentable.

> [!div class="mx-imgBorder"]
> ![la imagen va aquí](./media/premium-tier-for-data-lake-storage/premium-performance-data-lake-storage-cost-analysis-table.png)

> [!NOTE] 
> Si prefiere evaluar la rentabilidad en función del número de transacciones por segundo para cada TB de datos, puede usar los encabezados de las columnas que aparecen en la parte inferior de la tabla.

Para obtener más información sobre los precios, consulte la página [Precios de Azure Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/).

## <a name="feature-availability"></a>Disponibilidad de características 

Es posible que algunas características de Blob Storage no estén disponibles o que solo tengan compatibilidad parcial con el nivel de rendimiento Premium. Para obtener una lista completa, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md). Luego, revise una lista de [problemas conocidos](data-lake-storage-known-issues.md) para evaluar cualquier brecha que se produzca en la funcionalidad.

## <a name="enabling-the-premium-performance-tier"></a>Habilitación del nivel de rendimiento Premium 

Para usar el nivel Premium para Azure Data Lake Storage, puede crear una cuenta BlockBlobStorage con la opción **Espacio de nombres jerárquico** **habilitada**. Para obtener una guía completa, consulte [Creación de una cuenta BlockBlobStorage](../common/storage-account-create.md).

Al crear la cuenta, asegúrese de elegir la opción de rendimiento **Premium** y el tipo de cuenta **BlockBlobStorage**.

> [!div class="mx-imgBorder"]
> ![Creación de cuenta blockblobstorage](./media/premium-tier-for-data-lake-storage/create-block-blob-storage-account.png)

Habilite la opción **Espacio de nombres jerárquico** en la pestaña **Opciones avanzadas** de la página **Crear cuenta de almacenamiento**. Debe habilitar esta configuración al crear la cuenta. No se puede habilitar después.

En la siguiente imagen se muestra esta configuración en la página **Crear cuenta de almacenamiento**.

> [!div class="mx-imgBorder"]
> ![Opción Servicio de espacio de nombres jerárquico](./media/create-data-lake-storage-account/hierarchical-namespace-feature.png)

## <a name="regional-availability"></a>Disponibilidad regional

El nivel Premium para Azure Data Lake Storage está disponible en las siguientes regiones.

|Region|Redundancia|
|--|--|
|Este de EE. UU.|LRS, ZRS|
|Este de EE. UU. 2|LRS, ZRS|
|Centro de EE. UU.|LRS|
|Oeste de EE. UU.|LRS|
|Oeste de EE. UU. 2|LRS, ZRS|
|Centro-Oeste de EE. UU.|LRS|
|Centro-sur de EE. UU.|LRS|
|Centro de Canadá|LRS|
|Este de Canadá|LRS|
|Norte de Europa|LRS, ZRS|
|Oeste de Europa|LRS, ZRS|
|Sur de Reino Unido|LRS|
|Oeste de Reino Unido|LRS|
|Centro de Francia|LRS|
|Este de Asia|LRS|
|Centro de Corea del Sur|LRS|
|Corea del Sur|LRS|
|Centro de la India|LRS|
|Oeste de la India|LRS|
|Norte de Emiratos Árabes Unidos|LRS|
|Japón Oriental|LRS|
|Japón Occidental|LRS|
|Sudeste de Asia|LRS, ZRS|
|Este de Australia|LRS, ZRS|
|Sudeste de Australia|LRS|
|Sur de Brasil|LRS|

## <a name="next-steps"></a>Pasos siguientes

Use el nivel Premium para Azure Data Lake Storage con su servicio de análisis favorito, como Azure Databricks, Azure HDInsight y Azure Synapse Analytics. 

- [Tutorial: Azure Data Lake Storage Gen2, Azure Databricks y Spark](data-lake-storage-use-databricks-spark.md) 
- [Uso de Azure Data Lake Storage Gen2 con clústeres de Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md). Actualmente, HDInsight admite una cuenta que use el nivel de rendimiento Premium junto con un clúster de HBase que tenga habilitadas las escrituras aceleradas.
- [Inicio rápido: Creación de un área de trabajo de Synapse](../../synapse-analytics/quickstart-create-workspace.md)