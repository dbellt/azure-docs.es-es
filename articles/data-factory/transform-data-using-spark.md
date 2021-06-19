---
title: Transformación de datos mediante la actividad de Spark
description: Obtenga información sobre cómo transformar datos mediante la ejecución de programas de Spark desde una canalización de Azure Data Factory mediante la actividad de Spark.
ms.service: data-factory
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
ms.custom: seo-lt-2019
ms.date: 06/09/2021
ms.openlocfilehash: 3f9b330e4e0cadd073a60293991066f74d8521c3
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111903192"
---
# <a name="transform-data-using-spark-activity-in-azure-data-factory"></a>Transformación de datos mediante la actividad de Spark en Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-spark.md)
> * [Versión actual](transform-data-using-spark.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La actividad de Spark en una [canalización](concepts-pipelines-activities.md) de Data Factory ejecuta un programa de Spark en su clúster de HDInsight [propio](compute-linked-services.md#azure-hdinsight-linked-service) o [a petición](compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Este artículo se basa en el artículo sobre [actividades de transformación de datos](transform-data.md) , que presenta información general de la transformación de datos y las actividades de transformación admitidas. Cuando se usa un servicio vinculado a Spark a petición, Data Factory crea automáticamente un clúster Just-in-Time para procesar los datos y, luego, lo elimina una vez finalizado el procesamiento. 


## <a name="spark-activity-properties"></a>Propiedades de la actividad de Spark
Esta es la definición de JSON de ejemplo de una actividad de Spark:    

```json
{
    "name": "Spark Activity",
    "description": "Description",
    "type": "HDInsightSpark",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "sparkJobLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "rootPath": "adfspark",
        "entryFilePath": "test.py",
        "sparkConfig": {
            "ConfigItem1": "Value"
        },
        "getDebugInfo": "Failure",
        "arguments": [
            "SampleHadoopJobArgument1"
        ]
    }
}
```

En la siguiente tabla se describen las propiedades JSON que se usan en la definición de JSON:

| Propiedad              | Descripción                              | Obligatorio |
| --------------------- | ---------------------------------------- | -------- |
| name                  | Nombre de la actividad en la canalización.    | Sí      |
| description           | Texto que describe para qué se usa la actividad.  | No       |
| type                  | Para la actividad de Spark, el tipo de actividad es HDInsightSpark. | Sí      |
| linkedServiceName     | Nombre del servicio vinculado de HDInsight Spark en el que se ejecuta el programa de Spark. Para obtener más información sobre este servicio vinculado, vea el artículo [Compute linked services](compute-linked-services.md) (Servicios vinculados de procesos). | Sí      |
| SparkJobLinkedService | El servicio vinculado de Azure Storage que contiene los registros, las dependencias y los archivos de trabajos de Spark. En este caso solo se admiten servicios vinculados a **[Azure Blob Storage](./connector-azure-blob-storage.md)** y **[ADLS Gen2](./connector-azure-data-lake-storage.md)** . Si no especifica un valor para esta propiedad, se usa el almacenamiento asociado con el clúster de HDInsight. El valor de esta propiedad solo puede ser un servicio vinculado de Azure Storage. | No       |
| rootPath              | Contenedor de blobs de Azure y la carpeta que contiene el archivo de Spark. El nombre del archivo distingue mayúsculas de minúsculas. Vea la sección sobre la estructura de carpetas (sección siguiente) para obtener más información sobre la estructura de esta carpeta. | Sí      |
| entryFilePath         | Ruta de acceso relativa a la carpeta raíz del código o el paquete de Spark. El archivo de entrada debe ser un archivo de Python o un archivo .jar. | Sí      |
| className             | Clase principal de Spark o Java de la aplicación.      | No       |
| argumentos             | Lista de argumentos de línea de comandos del programa de Spark. | No       |
| proxyUser             | Cuenta de usuario de suplantación para ejecutar el programa de Spark. | No       |
| sparkConfig           | Especifique valores para las propiedades de configuración de Spark indicadas en el tema: [Spark Configuration - Application properties](https://spark.apache.org/docs/latest/configuration.html#available-properties) (Configuración de Spark: Propiedades de aplicación). | No       |
| getDebugInfo          | Especifica si se copian los archivos de registro de Spark en el almacenamiento de Azure que usa el clúster de HDInsight que especifica sparkJobLinkedService. Valores permitidos: Ninguno, Siempre o Error. Valor predeterminado: Ninguno. | No       |

## <a name="folder-structure"></a>Estructura de carpetas
Los trabajos de Spark son más extensibles que los de Pig y Hive. En los trabajos de Spark, puede proporcionar varias dependencias como paquetes JAR (ubicados en la CLASSPATH de Java), archivos de Python (ubicados en la ruta PYTHONPATH) y cualquier otro archivo.

Cree la siguiente estructura de carpetas en la instancia de Azure Blob Storage a la que hace referencia el servicio vinculado de HDInsight. Luego, cargue los archivos dependientes en las subcarpetas adecuadas de la carpeta raíz que representa **entryFilePath**. Por ejemplo, cargue los archivos de Python en la subcarpeta pyFiles y los archivos JAR en la subcarpeta jars de la carpeta raíz. En el entorno de tiempo de ejecución, el servicio Data Factory espera la siguiente estructura de carpetas en Azure Blob Storage:     

| Path                  | Descripción                              | Obligatorio | Tipo   |
| --------------------- | ---------------------------------------- | -------- | ------ |
| `.` (raíz)            | Ruta de acceso raíz del trabajo de Spark en el servicio vinculado de almacenamiento. | Sí      | Carpeta |
| &lt;Definida por el usuario&gt; | Ruta de acceso que apunta al archivo de entrada del trabajo de Spark. | Sí      | Archivo   |
| ./jars                | Todos los archivos de esta carpeta se cargan y se colocan en la ruta CLASSPATH de Java del clúster. | No       | Carpeta |
| ./pyFiles             | Todos los archivos de esta carpeta se cargan y se colocan en la ruta PYTHONPATH del clúster. | No       | Carpeta |
| ./files               | Todos los archivos de esta carpeta se cargan y se colocan en el directorio de trabajo del ejecutor. | No       | Carpeta |
| ./archives            | Todos los archivos de esta carpeta están sin comprimir. | No       | Carpeta |
| ./logs                | Carpeta que contiene los registros del clúster de Spark. | No       | Carpeta |

Este es un ejemplo de un almacenamiento que contiene dos archivos de trabajos de Spark en la instancia de Azure Blob Storage a la que hace referencia el servicio vinculado de HDInsight.

```
SparkJob1
    main.jar
    files
        input1.txt
        input2.txt
    jars
        package1.jar
        package2.jar
    logs
    
    archives
    
    pyFiles

SparkJob2
    main.py
    pyFiles
        scrip1.py
        script2.py
    logs
    
    archives
    
    jars
    
    files
    
```
## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes artículos, en los que se explica cómo transformar datos de otras maneras: 

* [Actividad de U-SQL](transform-data-using-data-lake-analytics.md)
* [Actividad de Hive](transform-data-using-hadoop-hive.md)
* [Actividad de Pig](transform-data-using-hadoop-pig.md)
* [Actividad de MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Actividad de streaming de Hadoop](transform-data-using-hadoop-streaming.md)
* [Actividad de Spark](transform-data-using-spark.md)
* [Actividad personalizada de .NET](transform-data-using-dotnet-custom-activity.md)
* [Actividad de ejecución por lotes de Azure Machine Learning Studio (clásico)](transform-data-using-machine-learning.md)
* [Actividad de procedimiento almacenado](transform-data-using-stored-procedure.md)
