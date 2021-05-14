---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: e38d7e07582fcbaa5b81e2dd694ddc39d6906be1
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753249"
---
Azure Data Factory admite las siguientes actividades de transformación que se pueden agregar a las canalizaciones tanto individualmente como encadenadas a otra actividad.

| Actividad de transformación de datos | Entorno de procesos |
|:--- |:--- |
| [Hive](../data-factory-hive-activity.md) |HDInsight [Hadoop] |
| [Pig](../data-factory-pig-activity.md) |HDInsight [Hadoop] |
| [de Hadoop](../data-factory-map-reduce.md) |HDInsight [Hadoop] |
| [Hadoop Streaming](../data-factory-hadoop-streaming-activity.md) |HDInsight [Hadoop] |
| [Spark](../data-factory-spark.md) | HDInsight [Hadoop] |
| [Actividades de Estudio de Azure Machine Learning (clásico): ejecución de lotes y recurso de actualización](../data-factory-azure-ml-batch-execution-activity.md) |Azure VM |
| [Procedimiento almacenado](../data-factory-stored-proc-activity.md) |Azure SQL, Azure Synapse Analytics o SQL Server |
| [U-SQL de análisis con Data Lake](../data-factory-usql-activity.md) |Análisis con Azure Data Lake |
| [DotNet](../data-factory-use-custom-activities.md) |HDInsight [Hadoop] o Azure Batch |

> [!NOTE]
> Puede usar la actividad MapReduce para ejecutar programas Spark en su clúster de HDInsight Spark. Consulte [Invocar programas Spark desde Azure Data Factory](../data-factory-spark.md) para obtener más información.
> Puede crear una actividad personalizada para ejecutar scripts de R en su clúster de HDInsight con R instalado. Consulte [Run R Script using Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample)(Ejecutar script de R con Data Factory de Azure).
> 
> 

