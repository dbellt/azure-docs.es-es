---
title: Introducción a Azure Data Lake Storage Gen2
description: Lea esta introducción a Azure Data Lake Storage Gen2. Aprenda sobre las características clave. Revise las características admitidas de almacenamiento de blobs, las integraciones del servicio de Azure y las plataformas.
author: normesta
ms.service: storage
ms.topic: overview
ms.date: 02/25/2020
ms.author: normesta
ms.reviewer: jamesbak
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: b197d91893cc419913ace60fe30beca88484bfd2
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111590961"
---
# <a name="introduction-to-azure-data-lake-storage-gen2"></a>Introducción a Azure Data Lake Storage Gen2

‎Azure Data Lake Storage Gen2 es un conjunto de funcionalidades dedicadas al análisis de macrodatos basado en [Azure Blob Storage](storage-blobs-introduction.md). 

En Data Lake Storage Gen2 convergen las funcionalidades de [Azure Data Lake Storage Gen1](../../data-lake-store/index.yml) con Azure Blob Storage. Por ejemplo, Data Lake Storage Gen2 proporciona la semántica del sistema de archivos, la seguridad de nivel de archivo y la escala. Como estas funcionalidades se basan en Blob Storage, también disfrutará de un almacenamiento por niveles de bajo costo, con funcionalidades de alta disponibilidad y recuperación ante desastres.

## <a name="designed-for-enterprise-big-data-analytics"></a>Se ha diseñado para el análisis de macrodatos empresariales.

Data Lake Storage Gen2 convierte a Azure Storage en los cimientos para crear lagos de datos empresariales en Azure. Diseñado desde el principio para servir varios petabytes de información y mantener cientos de gigabits de rendimiento, Data Lake Storage Gen2 le ofrece una forma fácil de administrar cantidades masivas de datos.

Parte fundamental de Data Lake Storage Gen2 es la incorporación de un [espacio de nombres jerárquico](data-lake-storage-namespace.md) en Blob Storage. El espacio de nombres jerárquico organiza los objetos o archivos en una jerarquía de directorios para un acceso eficaz a los datos. Una convención de nomenclatura de almacenamiento de objetos común usa barras diagonales en el nombre para imitar una estructura jerárquica de directorios. Esta estructura se hace realidad con Data Lake Storage Gen2. Algunas operaciones, como el cambio de nombre o la eliminación de un directorio, pasan a ser opciones atómicas individuales de metadatos en el directorio. No es necesario enumerar y procesar todos los objetos que comparten el prefijo del nombre del directorio.

Data Lake Storage Gen2 se basa en Blob Storage y mejora el rendimiento, la administración y la seguridad de las siguientes maneras:

-   El **rendimiento** se optimiza porque no es necesario copiar ni transformar datos como requisito previo para el análisis. En comparación con el espacio de nombres plano de Blob Storage, el espacio de nombres jerárquico mejora considerablemente el rendimiento de las operaciones de administración de directorios, que mejora el rendimiento general del trabajo.

-   La **administración** es más fácil, ya que puede organizar y manipular archivos a través de directorios y subdirectorios.

-   La **seguridad** se aplicable porque se pueden definir permisos POSIX en los directorios o archivos individuales.

Además, Azure Data Lake Storage Gen2 es muy rentable, ya que se basa en [Azure Blob Storage](storage-blobs-introduction.md) de bajo costo. Las características adicionales reducen aún más el costo total de propiedad para la ejecución de análisis de macrodatos en Azure.

## <a name="key-features-of-data-lake-storage-gen2"></a>Características clave de Data Lake Storage Gen2

-   **Acceso compatible con Hadoop**: Data Lake Storage Gen2 le permite administrar y acceder a los datos igual que lo haría con un [sistema de archivos distribuido de Hadoop (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html). El nuevo [controlador ABFS](data-lake-storage-abfs-driver.md) (que se usa para acceder a los datos) está disponible en todos los entornos de Apache Hadoop. Estos entornos incluyen [Azure HDInsight](../../hdinsight/index.yml) *,* [Azure Databricks](/azure/databricks/) y [Azure Synapse Analytics](../../synapse-analytics/index.yml).

-   **Superconjunto de permisos POSIX**: el modelo de seguridad de Data Lake Gen2 es compatible con los permisos de ACL y POSIX junto con granularidad adicional específica de Data Lake Storage Gen2. La configuración se puede realizar mediante el Explorador de Storage o a través de marcos como Hive y Spark.

-   **Rentabilidad**: Data Lake Storage Gen2 ofrece transacciones y capacidad de almacenamiento de bajo costo. Algunas características como el [ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md) optimizan los costos cuando los datos pasan por su ciclo de vida.

-   **Controlador optimizado**: El controlador ABFS está [optimizado específicamente](data-lake-storage-abfs-driver.md) para el análisis de macrodatos. Las API de REST correspondientes se exponen a través del punto de conexión `dfs.core.windows.net`.

### <a name="scalability"></a>Escalabilidad

Azure Storage es escalable de forma natural si obtiene acceso a través de las interfaces de Data Lake Storage Gen2 o Blob Storage. Es capaz de almacenar y atender *muchos exabytes de datos*. Esta cantidad de almacenamiento está disponible con rendimiento medido en gigabits por segundo (Gbps) en niveles altos de operaciones de entrada/salida por segundo (IOPS). El procesamiento se ejecuta en las latencias por solicitud casi constantes que se miden en los niveles de servicio, cuenta y archivo.

### <a name="cost-effectiveness"></a>Rentabilidad

Dado que Data Lake Storage Gen2 se basa en Azure Blob Storage, tanto la capacidad de almacenamiento como los costos de transacción son menores. A diferencia de otros servicios de almacenamiento en la nube, no es preciso mover o transformar los datos para poder analizarlos. Para más información sobre los precios, consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage).

Además, características como el [espacio de nombres jerárquico](data-lake-storage-namespace.md) mejoran significativamente el rendimiento general de muchos trabajos de análisis. Esta mejora del rendimiento significa que se requerirá menos eficacia de proceso para procesar la misma cantidad de datos, lo que genera un menor costo total de propiedad (TCO) para el trabajo de análisis de un extremo a otro.

### <a name="one-service-multiple-concepts"></a>Un servicio, varios conceptos

Como Data Lake Storage Gen2 se basa en Azure Blob Storage, varios conceptos pueden describir los mismos elementos compartidos.

A continuación, se muestran las entidades equivalentes, tal como se describen en distintos conceptos. A menos que se especifique lo contrario, estas entidades son sinónimos directos:

| Concepto                                | Organización de nivel superior | Organización de nivel inferior                                            | Contenedor de datos |
|----------------------------------------|------------------------|---------------------------------------------------------------------|----------------|
| Blobs: almacenamiento de objetos de uso general | Contenedor              | Directorio virtual (solo SDK, no proporciona manipulación atómica) | Blob           |
| Azure Data Lake Storage Gen2 (Analytics Storage)          | Contenedor            | Directorio                                                           | Archivo           |

## <a name="supported-blob-storage-features"></a>Características de Blob Storage compatibles

Las características de Blob Storage, como el [registro de diagnóstico](../common/storage-analytics-logging.md), los [niveles de acceso](storage-blob-storage-tiers.md) y las [directivas de administración del ciclo de vida de Blob Storage](storage-lifecycle-management-concepts.md) están disponibles en su cuenta. 

Para ver una lista de las funciones de Blob Storage admitidas, consulte el artículo acerca de las [características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md).

## <a name="supported-azure-service-integrations"></a>Integraciones de servicios de Azure admitidos

Data Lake Storage Gen2 admite varios servicios de Azure, que se pueden usar para ingerir datos, realizar análisis y crear representaciones visuales. Para ver una lista de los servicios de Azure admitidos, consulte el documento en que se indica cuáles son los [servicios de Azure que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

## <a name="supported-open-source-platforms"></a>Plataformas de código abierto compatibles

Varias plataformas de código abierto son compatibles con Data Lake Storage Gen2. Para ver una lista completa, consulte [Plataformas de código abierto que admiten Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

## <a name="see-also"></a>Consulte también

- [Problemas conocidos con Azure Data Lake Storage Gen2](data-lake-storage-known-issues.md)
- [Acceso multiprotocolo en Azure Data Lake Storage](data-lake-storage-multi-protocol-access.md)
