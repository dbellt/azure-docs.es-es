---
title: Ejecución de canalizaciones de Azure Machine Learning
description: Aprenda a ejecutar canalizaciones de Azure Machine Learning en las canalizaciones de Azure Data Factory.
ms.service: data-factory
ms.topic: conceptual
ms.author: weetok
author: dcstwh
ms.date: 07/16/2020
ms.openlocfilehash: d0b941325d6bbabcb083b5f71a40c145d3ef48d0
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103477"
---
# <a name="execute-azure-machine-learning-pipelines-in-azure-data-factory"></a>Ejecución de canalizaciones de Azure Machine Learning en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Ejecute canalizaciones de Azure Machine Learning en las canalizaciones de Azure Data Factory como un paso más. La actividad de ejecución de canalización de Machine Learning posibilita escenarios de predicción por lotes, como la identificación de posibles impagos de préstamos, la determinación de opiniones y el análisis de los patrones de comportamiento de los clientes.

En el vídeo siguiente se muestra una introducción de seis minutos y una demostración de esta característica.

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/How-to-execute-Azure-Machine-Learning-service-pipelines-in-Azure-Data-Factory/player]

## <a name="syntax"></a>Sintaxis

```json
{
    "name": "Machine Learning Execute Pipeline",
    "type": "AzureMLExecutePipeline",
    "linkedServiceName": {
        "referenceName": "AzureMLService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "mlPipelineId": "machine learning pipeline ID",
        "experimentName": "experimentName",
        "mlPipelineParameters": {
            "mlParameterName": "mlParameterValue"
        }
    }
}

```

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
name | Nombre de la actividad en la canalización | String | Sí
type | El tipo de actividad es "AzureMLExecutePipeline" | String | Sí
linkedServiceName | Servicio vinculado a Azure Machine Learning | Referencia al servicio vinculado | Sí
mlPipelineId | Identificador de la canalización de Azure Machine Learning publicada | Cadena (o expresión con un valor resultType de cadena) | Sí
experimentName | Nombre de experimento del historial de ejecuciones de la canalización de Machine Learning | Cadena (o expresión con un valor resultType de cadena) | No
mlPipelineParameters | Pares clave-valor que se pasan al punto de conexión de la canalización de Azure Machine Learning publicada. Las claves deben coincidir con los nombres de los parámetros de la canalización definidos en la canalización de Machine Learning publicada. | Objeto con pares clave-valor (o expresión con objeto resultType) | No
mlParentRunId | El identificador principal de la ejecución de la canalización de Azure Machine Learning | Cadena (o expresión con un valor resultType de cadena) | No
dataPathAssignments | Diccionario que se usa para cambiar las rutas de acceso a los datos en Azure Machine Learning. Habilita el cambio de rutas de acceso a datos. | Objeto con pares clave-valor | No
continueOnStepFailure | Puede decidir si se continúa la ejecución del resto de pasos de la ejecución de la canalización de Machine Learning si se produce un error en un paso. | boolean | No

> [!NOTE]
> Para rellenar los elementos desplegables del nombre y el identificador de la canalización de Machine Learning, el usuario debe tener permiso para enumerar las canalizaciones de ML. La experiencia de usuario de ADF llama a las API de AzureMLService directamente con las credenciales del usuario que ha iniciado sesión.  

## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes artículos, en los que se explica cómo transformar datos de otras maneras:

* [Actividad de ejecución de Data Flow](control-flow-execute-data-flow-activity.md)
* [Actividad de U-SQL](transform-data-using-data-lake-analytics.md)
* [Actividad de Hive](transform-data-using-hadoop-hive.md)
* [Actividad de Pig](transform-data-using-hadoop-pig.md)
* [Actividad de MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Actividad de streaming de Hadoop](transform-data-using-hadoop-streaming.md)
* [Actividad de Spark](transform-data-using-spark.md)
* [Actividad personalizada de .NET](transform-data-using-dotnet-custom-activity.md)
* [Actividad de procedimiento almacenado](transform-data-using-stored-procedure.md)
