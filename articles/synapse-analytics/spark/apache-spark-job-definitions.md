---
title: 'Tutorial: Creación de una definición de trabajo de Apache Spark en Synapse Studio'
description: 'Tutorial: Uso de Azure Synapse Analytics para crear definiciones de trabajos de Spark y enviarlas a un grupo de Apache Spark para Azure Synapse Analytics.'
author: Jejiang
ms.author: jejiang
ms.reviewer: jasonh
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 10/16/2020
ms.openlocfilehash: 7c60addd80f7ed027870744d12975fd76983b2f5
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109813837"
---
# <a name="tutorial-create-apache-spark-job-definition-in-synapse-studio"></a>Tutorial: Creación de una definición de trabajo de Apache Spark en Synapse Studio

En este tutorial se muestra cómo usar Synapse Studio para crear definiciones de trabajos de Apache Spark y enviarlas luego a un grupo de Apache Spark sin servidor.

En este tutorial se describen las tareas siguientes:
> [!div class="checklist"]
>
> - Creación de una definición de trabajo de Apache Spark para PySpark (Python)
> - Creación de una definición de trabajo de Apache Spark para Spark (Scala)
> - Creación de una definición de trabajo de Apache Spark para .NET Spark (C#/F#)
> - Creación de una definición de trabajo mediante la importación de un archivo JSON
> - Exportación de un archivo de definición de trabajo de Apache Spark a local
> - Envío de una definición de trabajo de Apache Spark como un trabajo por lotes
> - Adición de una definición de trabajo de Apache Spark en la canalización


## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, asegúrese de que se cumplen los requisitos siguientes:

* Un área de trabajo de Azure Synapse Analytics. Para obtener instrucciones, consulte [Creación de un área de trabajo de Azure Synapse Analytics](../../machine-learning/how-to-manage-workspace.md).
* Un grupo de Apache Spark sin servidor.
* Una cuenta de almacenamiento de ADLS Gen2. Debe tener el rol **Colaborador de datos de Storage Blob** en el sistema de archivos de ADLS Gen2 con el que quiere trabajar. Si no lo es, debe agregar el permiso manualmente.
* Si no desea usar el almacenamiento predeterminado del área de trabajo, vincule la cuenta de almacenamiento de ADLS Gen2 necesaria en Synapse Studio. 


## <a name="create-an-apache-spark-job-definition-for-pyspark-python"></a>Creación de una definición de trabajo de Apache Spark para PySpark (Python)

En esta sección, creará una definición de trabajo de Apache Spark para PySpark (Python).

1. Abra [Synapse Studio](https://web.azuresynapse.net/).

2. Puede ir a [Sample files for creating Apache Spark job definitions](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Python) (Archivos de ejemplo de creación de definiciones de trabajo en Apache Spark) para descargar **archivos de ejemplo para python.zip**; después, descomprima el paquete y extraiga los archivos **wordcount.py** y **shakespeare.txt**. 

     ![archivos de ejemplo](./media/apache-spark-job-definitions/sample-files.png)

3. Seleccione **Data** -> **Linked** -> **Azure Data Lake Storage Gen2** (Datos > Vinculados > Azure Data Lake Storage Gen2) y cargue **wordcount.py** y **shakespeare.txt** en el sistema de archivos de ADLS Gen2. 

     ![Carga del archivo de Python](./media/apache-spark-job-definitions/upload-python-file.png)

4. Seleccione el centro de conectividad **Develop** (Desarrollar), elija el icono "+" y seleccione **Spark job definition** (Definición de trabajo de Spark) para crear una definición de trabajo de Spark. 

     ![Creación de una definición para Python](./media/apache-spark-job-definitions/create-new-definition.png)

5. Seleccione **PySpark (Python)** en la lista desplegable de lenguajes de la ventana principal de definiciones de trabajos de Apache Spark.

     ![selección de python](./media/apache-spark-job-definitions/select-python.png)

6. Rellene la información de la definición de trabajo de Apache Spark. 

     |  Propiedad   | Descripción   |  
     | ----- | ----- |  
     |Job definition name (Nombre de la definición de trabajo)| Proporcione un nombre para la definición de trabajo de Apache Spark. Este nombre se puede actualizar en cualquier momento hasta que se publique. <br> Ejemplo: `job definition sample`|
     |Main definition file (Archivo de definición principal)| Archivo principal usado para el trabajo. Seleccione un archivo PY en el almacenamiento. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. <br> Ejemplo: `abfss://…/path/to/wordcount.py`|
     |Argumentos de la línea de comandos| Argumentos opcionales del trabajo. <br> Ejemplo: `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Nota: Los argumentos de la definición de trabajo de ejemplo se separan mediante un espacio.*|
     |Reference files (Archivos de referencia)| Archivos adicionales usados como referencia en el archivo de definición principal. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. |
     |Grupo de Spark| El trabajo se enviará al grupo de Apache Spark seleccionado.|
     |Versión de Spark| Versión de Apache Spark que ejecuta el grupo de Apache Spark.|
     |Ejecutores| Número de ejecutores que se van a proporcionar en el grupo de Apache Spark especificado para el trabajo.|
     |Executor size (Tamaño del ejecutor)| Número de núcleos y memoria que se van a usar para los ejecutores proporcionados en el grupo de Apache Spark especificado para el trabajo.|  
     |Driver size (Tamaño del controlador)| Número de núcleos y memoria que se van a usar para el controlador proporcionado en el grupo de Apache Spark especificado para el trabajo.|

     ![Establecimiento del valor de la definición de trabajo de Spark para Python](./media/apache-spark-job-definitions/create-py-definition.png)

7. Seleccione **Publicar** para guardar la definición de trabajo de Apache Spark.

     ![Publicación de la definición de Python](./media/apache-spark-job-definitions/publish-py-definition.png)

## <a name="create-an-apache-spark-job-definition-for-apache-sparkscala"></a>Creación de una definición de trabajo de Apache Spark para Apache Spark (Scala)

En esta sección, creará una definición de trabajo de Apache Spark para Apache Spark (Scala).

 1. Abra [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Puede ir a [Sample files for creating Apache Spark job definitions](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Scala) (Archivos de ejemplo de creación de definiciones de trabajo en Apache Spark) para descargar **archivos de ejemplo para scala.zip**; después, descomprima el paquete y extraiga los archivos **wordcount.jar** y **shakespeare.txt**. 
 
     ![archivos de ejemplo de scala](./media/apache-spark-job-definitions/sample-files-scala.png)

 3. Seleccione **Data** -> **Linked** -> **Azure Data Lake Storage Gen2** (Datos > Vinculados > Azure Data Lake Storage Gen2) y cargue **wordcount.jar** y **shakespeare.txt** en el sistema de archivos de ADLS Gen2.
 
     ![Preparación de la estructura de Scala](./media/apache-spark-job-definitions/prepare-scala-structure.png)

 4. Seleccione el centro de conectividad **Develop** (Desarrollar), elija el icono "+" y seleccione **Spark job definition** (Definición de trabajo de Spark) para crear una definición de trabajo de Spark. (La imagen de ejemplo es la misma que la del paso 4 de **Creación de una definición de trabajo de Apache Spark [Python] para PySpark**).

 5. Seleccione **Spark (Scala)** en la lista desplegable de lenguajes de la ventana principal de definiciones de trabajos de Apache Spark.

     ![selección de scala](./media/apache-spark-job-definitions/select-scala.png)

 6. Rellene la información de la definición de trabajo de Apache Spark. Puede copiar la información de ejemplo.

     |  Propiedad   | Descripción   |  
     | ----- | ----- |  
     |Job definition name (Nombre de la definición de trabajo)| Proporcione un nombre para la definición de trabajo de Apache Spark. Este nombre se puede actualizar en cualquier momento hasta que se publique. <br> Ejemplo: `scala`|
     |Main definition file (Archivo de definición principal)| Archivo principal usado para el trabajo. Seleccione un archivo JAR del almacenamiento. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. <br> Ejemplo: `abfss://…/path/to/wordcount.jar`|
     |Nombre de clase principal| Identificador completo o clase principal que se encuentra en el archivo de definición principal. <br> Ejemplo: `WordCount`|
     |Argumentos de la línea de comandos| Argumentos opcionales del trabajo. <br> Ejemplo: `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Nota: Los argumentos de la definición de trabajo de ejemplo se separan mediante un espacio.* |
     |Reference files (Archivos de referencia)| Archivos adicionales usados como referencia en el archivo de definición principal. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento.|
     |Grupo de Spark| El trabajo se enviará al grupo de Apache Spark seleccionado.|
     |Versión de Spark| Versión de Apache Spark que ejecuta el grupo de Apache Spark.|
     |Ejecutores| Número de ejecutores que se van a proporcionar en el grupo de Apache Spark especificado para el trabajo.|  
     |Executor size (Tamaño del ejecutor)| Número de núcleos y memoria que se van a usar para los ejecutores proporcionados en el grupo de Apache Spark especificado para el trabajo.|
     |Driver size (Tamaño del controlador)| Número de núcleos y memoria que se van a usar para el controlador proporcionado en el grupo de Apache Spark especificado para el trabajo.|

     ![Establecimiento del valor de la definición de trabajo de Spark para Scala](./media/apache-spark-job-definitions/create-scala-definition.png)

 7. Seleccione **Publicar** para guardar la definición de trabajo de Apache Spark.

      ![Publicación de la definición de Scala](./media/apache-spark-job-definitions/publish-scala-definition.png)

## <a name="create-an-apache-spark-job-definition-for-net-sparkcf"></a>Creación de una definición de trabajo de Apache Spark para .NET Spark (C#/F#)

En esta sección, creará una definición de trabajo de Apache Spark para .NET Spark (C#/F#).
 1. Abra [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Puede ir a [Sample files for creating Apache Spark job definitions](https://github.com/Azure-Samples/Synapse/tree/master/Spark/DotNET) (Archivos de ejemplo de creación de definiciones de trabajo en Apache Spark) para descargar **archivos de ejemplo para dotnet.zip**; después, descomprima el paquete y extraiga los archivos **wordcount.zip** y **shakespeare.txt**. 

     ![dotnet de ejemplo](./media/apache-spark-job-definitions/sample-dotnet.png)

 3. Seleccione **Data** -> **Linked** -> **Azure Data Lake Storage Gen2** (Datos > Vinculados > Azure Data Lake Storage Gen2) y cargue **wordcount.zip** y **shakespeare.txt** en el sistema de archivos de ADLS Gen2.
 
     ![Preparación de la estructura de dotnet](./media/apache-spark-job-definitions/prepare-dotnet-structure.png)

 4. Seleccione el centro de conectividad **Develop** (Desarrollar), elija el icono "+" y seleccione **Spark job definition** (Definición de trabajo de Spark) para crear una definición de trabajo de Spark. (La imagen de ejemplo es la misma que la del paso 4 de **Creación de una definición de trabajo de Apache Spark [Python] para PySpark**).

 5. Seleccione **.NET Spark(C#/F#)** en la lista desplegable de lenguajes de la ventana principal de definición de trabajos de Apache Spark.

     ![selección de dotnet](./media/apache-spark-job-definitions/select-dotnet.png)

 6. Rellene la información de la definición de trabajo de Apache Spark. Puede copiar la información de ejemplo.
    
     |  Propiedad   | Descripción   |  
     | ----- | ----- |  
     |Job definition name (Nombre de la definición de trabajo)| Proporcione un nombre para la definición de trabajo de Apache Spark. Este nombre se puede actualizar en cualquier momento hasta que se publique. <br> Ejemplo: `dotnet`|
     |Main definition file (Archivo de definición principal)| Archivo principal usado para el trabajo. Seleccione un archivo ZIP que contenga la aplicación .NET para Apache Spark (es decir, el archivo ejecutable principal, los archivos DLL que contienen las funciones definidas por el usuario y otros archivos necesarios) del almacenamiento. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. <br> Ejemplo: `abfss://…/path/to/wordcount.zip`|
     |Main executable file (Archivo ejecutable principal)| Archivo ejecutable principal del archivo ZIP de la definición principal. <br> Ejemplo: `WordCount`|
     |Argumentos de la línea de comandos| Argumentos opcionales del trabajo. <br> Ejemplo: `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result` <br> *Nota: Los argumentos de la definición de trabajo de ejemplo se separan mediante un espacio.* |
     |Reference files (Archivos de referencia)| Archivos adicionales necesarios para que los nodos de trabajo ejecuten la aplicación de .NET para Apache Spark y que no se incluyen en el archivo ZIP de la definición principal (es decir, archivos JAR dependientes, archivos DLL adicionales de funciones definidas por el usuario y otros archivos de configuración). Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento.|
     |Grupo de Spark| El trabajo se enviará al grupo de Apache Spark seleccionado.|
     |Versión de Spark| Versión de Apache Spark que ejecuta el grupo de Apache Spark.|
     |Ejecutores| Número de ejecutores que se van a proporcionar en el grupo de Apache Spark especificado para el trabajo.|  
     |Executor size (Tamaño del ejecutor)| Número de núcleos y memoria que se van a usar para los ejecutores proporcionados en el grupo de Apache Spark especificado para el trabajo.|
     |Driver size (Tamaño del controlador)| Número de núcleos y memoria que se van a usar para el controlador proporcionado en el grupo de Apache Spark especificado para el trabajo.|

     ![Establecimiento del valor de la definición de trabajo de Spark para dotnet](./media/apache-spark-job-definitions/create-dotnet-definition.png)

 7. Seleccione **Publicar** para guardar la definición de trabajo de Apache Spark.

      ![Publicación de la definición de dotnet](./media/apache-spark-job-definitions/publish-dotnet-definition.png)

## <a name="create-apache-spark-job-definition-by-importing-a-json-file"></a>Creación de una definición de trabajo de Apache Spark mediante la importación de un archivo JSON

 Puede importar un archivo JSON local existente en el área de trabajo de Azure Synapse desde el menú **Actions** (Acciones) (...) del explorador de definiciones de trabajo de Apache Spark para crear una nueva definición de trabajo de Apache Spark.

 ![creación de la definición de importación](./media/apache-spark-job-definitions/create-import-definition.png)

 
 La definición de trabajo de Spark es totalmente compatible con Livy API. Puede agregar parámetros adicionales a otras propiedades de Livy [(Documentos de Livy - AP REST [apache.org])](https://livy.incubator.apache.org/docs/latest/rest-api.html) en el archivo JSON local. También puede especificar los parámetros relacionados con la configuración de Spark en la propiedad de configuración, como se muestra a continuación. Después, puede importar el archivo JSON de nuevo para crear una nueva definición de trabajo de Apache Spark para el trabajo por lotes. Ejemplo de JSON para la importación de definiciones de Spark:
 
```Scala
   {
  "targetBigDataPool": {
    "referenceName": "socdemolarge",
    "type": "BigDataPoolReference"
  },
  "requiredSparkVersion": "2.3",
  "language": "scala",
  "jobProperties": {
    "name": "robinSparkDefinitiontest",
    "file": "adl://socdemo-c14.azuredatalakestore.net/users/robinyao/wordcount.jar",
    "className": "WordCount",
    "args": [
      "adl://socdemo-c14.azuredatalakestore.net/users/robinyao/shakespeare.txt"
    ],
    "jars": [],
    "files": [],
    "conf": {
      "spark.dynamicAllocation.enabled": "false",
      "spark.dynamicAllocation.minExecutors": "2",
      "spark.dynamicAllocation.maxExecutors": "2"
    },
    "numExecutors": 2,
    "executorCores": 8,
    "executorMemory": "24g",
    "driverCores": 8,
    "driverMemory": "24g"
  }
}

```

![otras propiedades de Livy](./media/apache-spark-job-definitions/other-livy-properties.png)

## <a name="export-an-existing-apache-spark-job-definition-file"></a>Abra una definición de trabajo de Apache Spark existente.

 Puede exportar los archivos de definición de trabajo de Apache Spark existentes al menú local desde el menú **Actions** (Acciones) (...) del Explorador de archivos. Puede seguir actualizando el archivo JSON para las propiedades adicionales de Livy, e importarlo de nuevo para crear una nueva definición de trabajo si fuera necesario.

 ![creación de definición de exportación](./media/apache-spark-job-definitions/create-export-definition.png)

 ![creación de definición de exportación 2](./media/apache-spark-job-definitions/create-export-definition-2.png)

## <a name="submit-an-apache-spark-job-definition-as-a-batch-job"></a>Envío de una definición de trabajo de Apache Spark como un trabajo por lotes

Después de crear una definición de trabajo de Apache Spark, puede enviarla a un grupo de Apache Spark. Asegúrese de tener el rol **Colaborador de datos de Storage Blob** en el sistema de archivos de ADLS Gen2 con el que quiere trabajar. Si no lo es, debe agregar el permiso manualmente.

### <a name="scenario-1-submit-apache-spark-job-definition"></a>Escenario 1: Envío de la definición de trabajo de Apache Spark
 1. Seleccione una ventana de definición de trabajo de Apache Spark para abrirla.

      ![Abrir definición de trabajo de Spark para enviar ](./media/apache-spark-job-definitions/open-spark-definition.png)

 2. Seleccione el botón **Enviar** para enviar el proyecto al grupo de Apache Spark seleccionado. Puede seleccionar la pestaña **Spark monitoring URL** (Dirección URL de supervisión de Spark) para ver la consulta de registro de la aplicación de Apache Spark.

    ![Seleccionar el botón Enviar para enviar la definición de trabajo de Spark](./media/apache-spark-job-definitions/submit-spark-definition.png)

    ![Cuadro de diálogo de envío de Spark](./media/apache-spark-job-definitions/submit-definition-result.png)

### <a name="scenario-2-view-apache-spark-job-running-progress"></a>Escenario 2: Visualización del progreso de ejecución del trabajo de Apache Spark

 1. Seleccione **Monitor** (Supervisar) y elija la opción **Apache Spark applications** (Aplicaciones de Apache Spark). Podrá encontrar la aplicación de Apache Spark enviada.

     ![Ver aplicación Spark](./media/apache-spark-job-definitions/view-spark-application.png)

 2. A continuación, haga clic en una aplicación de Apache Spark y se mostrará la ventana del trabajo **SparkJobDefinition**. Puede ver el progreso del trabajo desde aquí.
     
     ![Ver consulta del registro de la aplicación Spark](./media/apache-spark-job-definitions/view-job-log-query.png)

### <a name="scenario-3-check-output-file"></a>Escenario 3: Comprobación del archivo de salida

 1. Seleccione **Data** -> **Linked** -> **Azure Data Lake Storage Gen2** (Datos > Vinculados > Azure Data Lake Storage Gen2) (hozhaobdbj), abra la carpeta **result** que se creó anteriormente. Podrá ir a esta carpeta y comprobar si se ha generado la salida.

     ![Ver archivo de salida](./media/apache-spark-job-definitions/view-output-file.png)

## <a name="add-an-apache-spark-job-definition-into-pipeline"></a>Adición de una definición de trabajo de Apache Spark en la canalización

En esta sección, agregará una definición de trabajo de Apache Spark en la canalización.

 1. Abra una definición de trabajo de Apache Spark existente.

 2. Seleccione el icono de la parte superior derecha de la definición de trabajo de Apache Spark, elija **Existing Pipeline** (Canalización existente) o **New pipeline** (Nueva canalización). Puede consultar la página Pipeline (Canalización) para más información.

     ![incorporación a la canalización 1](./media/apache-spark-job-definitions/add-to-pipeline01.png)

     ![incorporación a la canalización 2](./media/apache-spark-job-definitions/add-to-pipeline02.png)


## <a name="next-steps"></a>Pasos siguientes

Luego, puede usar Azure Synapse Studio para crear conjuntos de datos de Power BI y administrar los datos de Power BI. Avance al artículo [Vinculación de un área de trabajo de Power BI a un área de trabajo de Synapse](../quickstart-power-bi.md) para obtener más información. 

