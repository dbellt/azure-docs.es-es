---
title: Azure Data Factory - Referencia de scripting JSON
description: Proporciona esquemas JSON para entidades de Data Factory.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 426965d469fbe08ecc0b0b6bf295d9919e698836
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665731"
---
# <a name="azure-data-factory---json-scripting-reference"></a>Azure Data Factory - Referencia de scripting JSON
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory.


Este artículo proporciona ejemplos y esquemas JSON para definir entidades de Azure Data Factory (canalización, actividad, conjunto de datos y servicio vinculado).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="pipeline"></a>Canalización
La estructura de alto nivel de una definición de canalización es como la siguiente:

```json
{
  "name": "SamplePipeline",
  "properties": {
    "description": "Describe what pipeline does",
    "activities": [
    ],
    "start": "2016-07-12T00:00:00",
    "end": "2016-07-13T00:00:00"
  }
}
```

En la tabla siguiente se describen las propiedades dentro de las definiciones JSON de canalización:

| Propiedad | Descripción | Obligatorio
-------- | ----------- | --------
| name | Nombre de la canalización. Especifique un nombre que represente la acción que la actividad o la canalización está configurada para realizar<br/><ul><li>Número máximo de caracteres: 260</li><li>Debe empezar por una letra, un número o un carácter de subrayado (\_)</li><li>No se permiten los caracteres siguientes: “.”, “+”, “?”, “/”, “<”,”>”,”*”,”%”,”&”,”:”,”\\”</li></ul> |Sí |
| description |Texto que describe para qué se usa la actividad o la canalización | No |
| activities | Contiene una lista de actividades. | Sí |
| start |Fecha y hora de inicio para la canalización. Debe estar en [formato ISO](https://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: 2014-10-14T16:32:41. <br/><br/>Se puede especificar una hora local, por ejemplo, una hora EST. Aquí tiene un ejemplo: `2016-02-27T06:00:00**-05:00`, que es 6 a. m. EST.<br/><br/>Las propiedades start y end juntas especifican un período activo para la canalización. Los segmentos de salida solo se producen en este período activo. |No<br/><br/>Si se especifica un valor para la propiedad end, hay que especificar un valor para la propiedad start.<br/><br/>Las horas de inicio y finalización pueden estar vacías para crear una canalización. Debe especificar ambos valores para establecer un período activo para que se ejecute la canalización. Si no especifica las horas de inicio y de finalización al crear una canalización, puede establecerlas más adelante mediante el cmdlet Set-AzDataFactoryPipelineActivePeriod. |
| end |Fecha y hora de finalización para la canalización. Si se especifica, debe estar en formato ISO. Por ejemplo: 2014-10-14T17:32:41 <br/><br/>Se puede especificar una hora local, por ejemplo, una hora EST. Aquí tiene un ejemplo: `2016-02-27T06:00:00**-05:00`, que es 6 a. m. EST.<br/><br/>Para ejecutar la canalización indefinidamente, especifique 9999-09-09 como valor de la propiedad end. |No <br/><br/>Si se especifica un valor para la propiedad start, hay que especificar un valor para la propiedad end.<br/><br/>Consulte las notas para la propiedad **start** . |
| isPaused |Si se establece en true, la canalización no se ejecuta. Valor predeterminado = false. Puede utilizar esta propiedad para habilitar o deshabilitar. |No |
| pipelineMode |El método para la programación de ejecuciones para la canalización. Los valores permitidos son: scheduled (predeterminado).<br/><br/>‘Scheduled’ indica que la canalización se ejecutará en el intervalo de tiempo especificado de acuerdo con su período activo (hora de inicio y finalización). 'Onetime' indica que la canalización se ejecutará solo una vez. Las canalizaciones creadas una sola vez no se pueden modificar ni actualizar actualmente. Consulte la sección [Canalización de una vez](data-factory-create-pipelines.md#onetime-pipeline) para más información acerca de la configuración onetime. |No |
| expirationTime |Tiempo de duración después de la creación durante el que la canalización es válida y debe permanecer aprovisionada. La canalización se elimina automáticamente una vez que alcanza la hora de caducidad si no tiene ninguna ejecución activa, con error o pendiente. |No |


## <a name="activity"></a>Actividad
La estructura de alto nivel de una actividad dentro de una definición de canalización (elemento de actividades) es como la siguiente:

```json
{
    "name": "ActivityName",
    "description": "description",
    "type": "<ActivityType>",
    "inputs":  "[]",
    "outputs":  "[]",
    "linkedServiceName": "MyLinkedService",
    "typeProperties":
    {

    },
    "policy":
    {
    },
    "scheduler":
    {
    }
}
```

En la tabla siguiente se describen las propiedades dentro de la definición JSON de actividad:

| Etiqueta | Descripción | Obligatorio |
| --- | --- | --- |
| name |Nombre de la actividad. Especifique un nombre que represente la acción que la actividad está configurada para realizar<br/><ul><li>Número máximo de caracteres: 260</li><li>Debe empezar por una letra, un número o un carácter de subrayado (\_)</li><li>No se permiten los caracteres siguientes: “.”, “+”, “?”, “/”, “<”,”>”,”*”,”%”,”&”,”:”,”\\”</li></ul> |Sí |
| description |Texto que describe para qué se usa la actividad. |No |
| type |Especifica el tipo de actividad. Vea las secciones sobre [ALMACENES DE DATOS](#data-stores) y [ACTIVIDADES DE TRANSFORMACIÓN DE DATOS](#data-transformation-activities) para ver los distintos tipos de actividades. |Sí |
| inputs |Tablas de entrada utilizadas por la actividad<br/><br/>`// one input table`<br/>`"inputs":  [ { "name": "inputtable1"  } ],`<br/><br/>`// two input tables` <br/>`"inputs":  [ { "name": "inputtable1"  }, { "name": "inputtable2"  } ],` |No es para las actividades de HDInsightStreaming ni SqlServerStoredProcedure <br/> <br/> Sí para el resto |
| outputs |Tablas de salida utilizadas por la actividad.<br/><br/>`// one output table`<br/>`"outputs":  [ { "name": “outputtable1” } ],`<br/><br/>`//two output tables`<br/>`"outputs":  [ { "name": “outputtable1” }, { "name": “outputtable2” }  ],` |Sí |
| linkedServiceName |Nombre del servicio vinculado utilizado por la actividad. <br/><br/>Una actividad puede requerir que especifique el servicio vinculado que enlaza con el entorno de procesos necesario. |Sí para las actividades de HDInsight, las actividades de Azure Machine Learning Studio (clásico) y las actividades de procedimiento almacenado. <br/><br/>No para todos los demás |
| typeProperties |Propiedades en la sección typeProperties dependen del tipo de actividad. |No |
| policy |Directivas que afectan al comportamiento en tiempo de ejecución de la actividad. Si no se especifica, se usan las directivas predeterminadas. |No |
| scheduler |La propiedad "scheduler" se utiliza para definir la programación deseada para la actividad. Sus subpropiedades son las mismas que las de la [propiedad availability de un conjunto de datos](data-factory-create-datasets.md#dataset-availability). |No |

### <a name="policies"></a>Directivas
Las directivas afectan al comportamiento en tiempo de ejecución de una actividad, concretamente cuando se procesa el segmento de una tabla. En la tabla siguiente se proporciona los detalles.

| Propiedad | Valores permitidos | Valor predeterminado | Descripción |
| --- | --- | --- | --- |
| simultaneidad |Entero <br/><br/>Valor máximo: 10 |1 |Número de ejecuciones simultáneas de la actividad.<br/><br/>Determina el número de ejecuciones paralelas de la actividad que pueden tener lugar en distintos segmentos. Por ejemplo, si una actividad tiene que recorrer un gran conjunto de datos disponibles, tener un valor mayor de simultaneidad acelera el procesamiento de datos. |
| executionPriorityOrder |NewestFirst<br/><br/>OldestFirst |OldestFirst |Determina el orden de los segmentos de datos que se están procesando.<br/><br/>Por ejemplo, si tiene 2 segmentos (que tienen lugar uno a las 4 p.m. y el otro a las 5 p.m.) y ambos están pendientes de ejecución. Si establece que executionPriorityOrder sea NewestFirst, se procesará primero el segmento de las 5 p.m. De forma similar, si establece que executionPriorityORder sea OldestFIrst, se procesará el segmento de las 4 p.m. |
| retry |Entero<br/><br/>El valor máximo permitido es 10 |0 |Número de reintentos antes de que el procesamiento de datos del segmento se marque como error. La ejecución de la actividad de un segmento de datos se vuelve a intentar hasta el número de reintentos especificado. El reintento se realiza tan pronto como sea posible después del error. |
| timeout |TimeSpan |00:00:00 |Tiempo de espera para la actividad. Ejemplo: 00:10:00 (implica un tiempo de espera de 10 minutos)<br/><br/>Si un valor no se especifica o es 0, el tiempo de espera es infinito.<br/><br/>Si el tiempo de procesamiento de los datos en un segmento supera el valor de tiempo de espera, se cancela y el sistema vuelve a intentar el procesamiento. El número de reintentos depende de la propiedad retry. Si se excede el tiempo de espera, el estado será TimedOut. |
| delay |TimeSpan |00:00:00 |Especifica el retraso antes de iniciar el procesamiento de los datos del segmento.<br/><br/>La ejecución de la actividad de un segmento de datos se inicia una vez que transcurra el retraso más allá del tiempo de ejecución esperado.<br/><br/>Ejemplo: 00:10:00 (implica un retraso de 10 minutos) |
| longRetry |Entero<br/><br/>Valor máximo: 10 |1 |Número de reintentos largos antes de que la ejecución de los segmentos produzca error.<br/><br/>Los intentos de longRetry se espacian de acuerdo a longRetryInterval. Por tanto, si necesita especificar un tiempo entre reintentos, utilice longRetry. Si se especifican Retry y longRetry, cada intento de longRetry incluirá el número de intentos de Retry y el número máximo de intentos será Retry * longRetry.<br/><br/>Por ejemplo, si tenemos la siguiente configuración en la directiva de la actividad:<br/>Reintento: 3<br/>longRetry: 2<br/>longRetryInterval: 01:00:00<br/><br/>Se supone que existe un solo segmento para ejecutar (el estado es En espera) y la ejecución de la actividad no se puede realizar nunca. Inicialmente habría tres intentos consecutivos de ejecución. Después de cada intento, el estado del segmento sería Retry. Después de los 3 primeros intentos, el estado del segmento sería LongRetry.<br/><br/>Después de una hora (es decir, el valor de longRetryInteval), se produciría otro conjunto de 3 intentos consecutivos de ejecución. Después de eso, el estado del segmento sería Failed y ya no se realizarían más intentos. Por tanto, en total se realizaron 6 intentos.<br/><br/>Si una ejecución se realiza correctamente, el estado del segmento sería Ready y no se realizaría ningún otro reintento.<br/><br/>longRetry puede usarse en situaciones donde llegan datos dependientes a horas no deterministas o el entorno general en el que se produce el procesamiento de datos es poco confiable. En esos casos es posible que realizar reintentos uno tras otro no ayude, mientras que hacerlo después de un intervalo de tiempo puede generar el resultado deseado.<br/><br/>Advertencia: No establezca valores altos para longRetry o longRetryInterval. Normalmente, los valores más altos implican otros problemas sistémicos. |
| longRetryInterval |TimeSpan |00:00:00 |El retraso entre reintentos largos |

### <a name="typeproperties-section"></a>Sección typeProperties
La sección typeProperties es diferente para cada actividad. Las actividades de transformación tienen las propiedades del tipo. Vea la sección [Actividades de transformación de datos](#data-transformation-activities) de este artículo para obtener ejemplos JSON que definen las actividades de transformación en una canalización.

La **actividad de copia** tiene dos subsecciones en la sección typeProperties: **origen** y **receptor**. Vea la sección [Almacenes de datos](#data-stores) en este artículo para consultar ejemplos JSON que muestran cómo utilizar datos de un almacén como un origen o receptor.

### <a name="sample-copy-pipeline"></a>Canalización de copia de ejemplo
En la canalización de ejemplo siguiente, hay una actividad del tipo **Copy** in the **actividades** . En este ejemplo, [Copiar actividad](data-factory-data-movement-activities.md) copia datos desde un almacén de Azure Blob Storage en Azure SQL Database.

```json
{
  "name": "CopyPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "type": "Copy",
        "inputs": [
          {
            "name": "InputDataset"
          }
        ],
        "outputs": [
          {
            "name": "OutputDataset"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2016-07-12T00:00:00",
    "end": "2016-07-13T00:00:00"
  }
}
```

Tenga en cuenta los siguientes puntos:

* En la sección de actividades, solo hay una actividad con **type** establecido en **Copy**.
* La entrada de la actividad está establecida en **InputDataset**, mientras que la salida está establecida en **OutputDataset**.
* En la sección **typeProperties**, **BlobSource** se especifica como el tipo de origen y **SqlSink** como el tipo de receptor.

Vea la sección [Almacenes de datos](#data-stores) en este artículo para consultar ejemplos JSON que muestran cómo utilizar datos de un almacén como un origen o receptor.

Para obtener un tutorial completo de creación de esta canalización, consulte [Tutorial: Copia de datos de Blob Storage en SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

### <a name="sample-transformation-pipeline"></a>Canalización de transformación de ejemplo
En la canalización de ejemplo siguiente, hay una actividad del tipo **HDInsightHive** in the **actividades** . En este ejemplo, el [actividad de HDInsight Hive](data-factory-hive-activity.md) transforma los datos de Azure Blob Storage mediante la ejecución de un archivo de script de Hive en un clúster de Azure HDInsight Hadoop.

```json
{
    "name": "TransformPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [
            {
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                    "scriptLinkedService": "AzureStorageLinkedService",
                    "defines": {
                        "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                        "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                    }
                },
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "policy": {
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Month",
                    "interval": 1
                },
                "name": "RunSampleHiveActivity",
                "linkedServiceName": "HDInsightOnDemandLinkedService"
            }
        ],
        "start": "2016-04-01T00:00:00",
        "end": "2016-04-02T00:00:00",
        "isPaused": false
    }
}
```

Tenga en cuenta los siguientes puntos:

* En la sección de actividades, solo hay una actividad con **type** establecido en **HDInsightHive**.
* El archivo de script de Hive, **partitionweblogs.hql**, se almacena en la cuenta de Azure Storage (especificada por la propiedad scriptLinkedService, llamada **AzureStorageLinkedService**) en una carpeta denominada **script** del contenedor **adfgetstarted**.
* La sección **defines** se usa para especificar la configuración de tiempo de ejecución que se pasa al script de Hive como valores de configuración de Hive (por ejemplo, `${hiveconf:inputtable}` y `${hiveconf:partitionedtable}`).

Vea la sección [Actividades de transformación de datos](#data-transformation-activities) de este artículo para obtener ejemplos JSON que definen las actividades de transformación en una canalización.

Para obtener un tutorial completo de creación de esta canalización, consulte [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="linked-service"></a>Servicio vinculado
La estructura de alto nivel de una definición de servicio vinculado es como la siguiente:

```json
{
    "name": "<name of the linked service>",
    "properties": {
        "type": "<type of the linked service>",
        "typeProperties": {
        }
    }
}
```

En la tabla siguiente se describen las propiedades dentro de la definición JSON de actividad:

| Propiedad | Descripción | Obligatorio |
| -------- | ----------- | -------- |
| name | Nombre del servicio vinculado. | Sí |
| propiedades - tipo | Tipo de servicio vinculado. Por ejemplo: Azure Storage y Azure SQL Database. |
| typeProperties | La sección typeProperties tiene elementos que son diferentes para cada almacén de datos o entorno de proceso. Consulte la sección Almacenes de datos para todos los servicios vinculados de almacén de datos y [Entornos de proceso](#compute-environments) para todos los servicios vinculados de proceso. |

## <a name="dataset"></a>Dataset
Un conjunto de datos en Azure Data Factory se define como sigue:

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": <boolean flag to indicate external data. only for input datasets>,
        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        },
        "availability": {
            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
        },
        "policy":
        {
        }
    }
}
```

La tabla siguiente describe las propiedades del JSON anterior:

| Propiedad | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| name | Nombre del conjunto de datos. Consulte [Azure Data Factory: reglas de nomenclatura](data-factory-naming-rules.md) para ver este tipo de reglas. |Sí |N/D |
| type | Tipo de conjunto de datos. Especifique uno de los tipos admitidos por Azure Data Factory (por ejemplo: AzureBlob, AzureSqlTable). Vea la sección [ALMACENES DE DATOS](#data-stores) para todos los almacenes de datos y tipos de conjuntos de datos admitidos en Data Factory. |
| structure | Esquema del conjunto de datos. Contiene las columnas, sus tipos, etc. | No |N/D |
| typeProperties | Propiedades correspondientes al tipo seleccionado. Vea la sección [ALMACENES DE DATOS](#data-stores) para consultar los tipos admitidos y sus propiedades. |Sí |N/D |
| external | Marca booleana para especificar si un conjunto de datos es generado explícitamente por una canalización de la factoría de datos o no. |No |false |
| availability | Define la ventana de procesamiento o el modelo de segmentación para la producción del conjunto de datos. Consulte el artículo [Programación y ejecución](data-factory-scheduling-and-execution.md) para más información. |Sí |N/D |
| policy |Define los criterios o la condición que deben cumplir los segmentos del conjunto de datos. <br/><br/>Para más información, consulte la sección Directiva del conjunto de datos. |No |N/D |

Cada columna de la sección **Estructura** contiene las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| name |Nombre de la columna. |Sí |
| type |Tipo de datos de la columna.  |No |
| culture |Referencia cultural basada en .NET que se usará cuando se especifica el tipo y sea un tipo .NET `Datetime` o `Datetimeoffset`. El valor predeterminado es `en-us`. |No |
| format |Cadena de formato que se usará cuando se especifica el tipo y sea un tipo .NET `Datetime` o `Datetimeoffset`. |No |

En el ejemplo siguiente, el conjunto de datos tiene tres columnas, `slicetimestamp`, `projectname` y `pageviews`, que son del tipo: String, String y Decimal, respectivamente.

```json
structure:
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

La tabla siguiente describe las propiedades que puede utilizar en la sección de **disponibilidad**:

| Propiedad | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| frequency |Especifica la unidad de tiempo para la producción de segmentos del conjunto de datos.<br/><br/><b>Frecuencia admitida</b>: Minute, Hour, Day, Week, Month |Sí |N/D |
| interval |Especifica un multiplicador para frecuencia<br/><br/>”Frequency x interval” determina la frecuencia con la que se produce el segmento.<br/><br/>Si necesita segmentar el conjunto de datos cada hora, establezca <b>frequency</b> en <b>hour</b> e <b>interval</b> en <b>1</b>.<br/><br/><b>Nota</b>: Si especifica Frequency como Minute, se recomienda establecer interval en no menos de 15 |Sí |N/D |
| style |Especifica si el segmento debe producirse al principio o al final del intervalo.<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>Si frequency se establece en Month y style se establece en EndOfInterval, el segmento se produce el último día del mes. Si style se establece en StartOfInterval, el segmento se produce el primer día del mes.<br/><br/>Si frequency se establece en Day y style se establece en EndOfInterval, el segmento se produce la última hora del día.<br/><br/>Si frequency se establece en Hour y style se establece en EndOfInterval, el segmento se produce al final de la hora. Por ejemplo, para un segmento para el período de 1 p.m. – 2 p.m., el segmento se producirá a las 2 p.m. |No |EndOfInterval |
| anchorDateTime |Define la posición absoluta en el tiempo usada por el programador para calcular los límites del segmento de conjunto de datos. <br/><br/><b>Nota</b>: Si AnchorDateTime tiene partes de fecha más pormenorizadas que la frecuencia, estas se omitirán. <br/><br/>Por ejemplo, si el valor de <b>interval</b> es <b>hourly</b> (frecuencia: hora e intervalo: 1) y <b>AnchorDateTime</b> contiene <b>minutes and seconds</b>, las partes <b>minutes and seconds</b> de AnchorDateTime no se tienen en cuenta. |No |01/01/0001 |
| offset |Intervalo de tiempo en función del cual se desplazan el inicio y el final de todos los segmentos del conjunto de datos. <br/><br/><b>Nota</b>: Si se especifican anchorDateTime y offset, el resultado es el desplazamiento combinado. |No |N/D |

La sección de disponibilidad siguiente especifica que el conjunto de datos de salida se produce cada hora (o) que el conjunto de datos de entrada está disponible cada hora:

```json
"availability":
{
    "frequency": "Hour",
    "interval": 1
}
```

En la sección **policy** de la definición del conjunto de datos se definen los criterios o condiciones que deben cumplir los segmentos del conjunto de datos.

| Nombre de la directiva | Descripción | Aplicado a | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- | --- |
| minimumSizeMB |Valida que los datos de un **blob de Azure** cumplen los requisitos de tamaño mínimo (en megabytes). |Blob de Azure |No |N/D |
| minimumRows |Valida que los datos en **Azure SQL Database** o en una **tabla de Azure** contienen el número mínimo de filas. |<ul><li>Azure SQL Database</li><li>tabla de Azure</li></ul> |No |N/D |

**Ejemplo**:

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

A menos que se esté produciendo un conjunto de datos mediante Azure Data Factory, debe marcarse como **external**. Generalmente, esta configuración se aplicará a las entradas de la primera actividad de una canalización a menos que se desee usar el encadenamiento de actividades o canalizaciones.

| Nombre | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| dataDelay |Tiempo de retraso de la comprobación de la disponibilidad de los datos externos para el segmento especificado. Por ejemplo, si los datos están disponibles cada hora, la comprobación de si los datos externos están realmente disponibles y si el segmento correspondiente está preparado puede retrasarse mediante dataDelay.<br/><br/>Solo se aplica a la hora actual.  Por ejemplo, si es la 1:00 p.m. y este valor es de 10 minutos, la validación se inicia a la 1:10 p.m.<br/><br/>Esta configuración no afecta a los segmentos en el pasado; los segmentos para los que se cumple Slice End Time + dataDelay < Now se procesan sin ningún retraso.<br/><br/>Las horas posteriores a las 23:59 horas se deben especificar con el formato `day.hours:minutes:seconds`. Por ejemplo, para especificar 24 horas, no use 24:00:00; en su lugar, use 1.00:00:00. Si usa 24:00:00, se tratará como 24 días (24.00:00:00). Para 1 día y 4 horas, especifique 1:04:00:00. |No |0 |
| retryInterval |El tiempo de espera entre un error y el siguiente reintento. Si se produce un error en el intento, el siguiente se realizará después del valor especificado en retryInterval. <br/><br/>Si ahora es la 1:00 p. m., empezaremos el primer intento. Si la duración para completar la primera comprobación de validación es 1 minuto y la operación produce un error, el siguiente reintento será a la 1:00 + 1 minuto (duración) + 1 minuto (intervalo de reintento) = 1:02 p. m. <br/><br/>En el caso de los segmentos en el pasado, no habrá ningún retraso. El reintento se producirá inmediatamente. |No |00:01:00 (1 minuto) |
| retryTimeout |El tiempo de espera de cada reintento.<br/><br/>Si esta propiedad se establece en 10 minutos, la validación se debe completar en 10 minutos. Si la validación tarda más de 10 minutos en realizarse, el reintento agotará el tiempo de espera.<br/><br/>Si se agota el tiempo de espera de todos los intentos de validación, el segmento se marcará como TimedOut. |No |00:10:00 (10 minutos) |
| maximumRetry |Número de veces que se va a comprobar la disponibilidad de los datos externos. El valor máximo permitido es 10. |No |3 |


## <a name="data-stores"></a>ALMACENES DE DATOS
En la sección [Servicio vinculado](#linked-service) se proporcionan descripciones de los elementos JSON que son comunes para todos los tipos de servicios vinculados. Esta sección proporciona detalles sobre los elementos JSON que son específicos de cada almacén de datos.

En la sección [Conjunto de datos](#dataset) se proporcionan descripciones de los elementos JSON que son comunes para todos los tipos de conjuntos de datos. Esta sección proporciona detalles sobre los elementos JSON que son específicos de cada almacén de datos.

En la sección [Actividad](#activity) se proporcionan descripciones de los elementos JSON que son comunes para todos los tipos de actividades. En esta sección se proporcionan detalles sobre los elementos JSON que son específicos para cada almacén de datos cuando se utiliza como un origen/receptor en una actividad de copia.

Haga clic en el vínculo del almacén del que le interesa ver los esquemas JSON para el servicio vinculado, el conjunto de datos y el origen/receptor para la actividad de copia.

| Category | Almacén de datos
|:--- |:--- |
| **Azure** |[Almacenamiento de blobs de Azure](#azure-blob-storage) |
| &nbsp; |Azure Data Lake Store |
| &nbsp; |[Azure Cosmos DB](#azure-cosmos-db) |
| &nbsp; |[Azure SQL Database](#azure-sql-database) |
| &nbsp; |[Azure Synapse Analytics](#azure-synapse-analytics) |
| &nbsp; |[Azure Cognitive Search](#azure-cognitive-search) |
| &nbsp; |[Azure Table Storage](#azure-table-storage) |
| **Bases de datos** |[Amazon Redshift](#amazon-redshift) |
| &nbsp; |[IBM DB2](#ibm-db2) |
| &nbsp; |[MySQL](#mysql) |
| &nbsp; |[Oracle](#oracle) |
| &nbsp; |[PostgreSQL](#postgresql) |
| &nbsp; |[SAP Business Warehouse](#sap-business-warehouse) |
| &nbsp; |[SAP HANA](#sap-hana) |
| &nbsp; |[SQL Server](#sql-server) |
| &nbsp; |[Sybase](#sybase) |
| &nbsp; |[Teradata](#teradata) |
| **NoSQL** |[Cassandra](#cassandra) |
| &nbsp; |[MongoDB](#mongodb) |
| **Archivo** |[Amazon S3](#amazon-s3) |
| &nbsp; |[Sistema de archivos](#file-system) |
| &nbsp; |[FTP](#ftp) |
| &nbsp; |[HDFS](#hdfs) |
| &nbsp; |[SFTP](#sftp) |
| **Otros** |[HTTP](#http) |
| &nbsp; |[OData](#odata) |
| &nbsp; |[ODBC](#odbc) |
| &nbsp; |[Salesforce](#salesforce) |
| &nbsp; |Tabla web |

## <a name="azure-blob-storage"></a>Azure Blob Storage

### <a name="linked-service"></a>Servicio vinculado
Existen dos tipos de servicios vinculados: servicio vinculado de Azure Storage y servicio vinculado de SAS de Azure Storage.

#### <a name="azure-storage-linked-service"></a>Servicio vinculado de Azure Storage
Para vincular la cuenta de Azure Storage a una instancia de Data Factory mediante una **clave de cuenta**, cree un servicio vinculado de Azure Storage. Para definir un servicio vinculado de Azure Storage, establezca el **tipo** del servicio vinculado en **AzureStorage**. Después, puede especificar las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| connectionString |Especifique la información necesaria para conectarse a Almacenamiento de Azure para la propiedad connectionString. |Sí |

##### <a name="example"></a>Ejemplo

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

#### <a name="azure-storage-sas-linked-service"></a>Servicio vinculado de SAS de Azure Storage
El servicio vinculado de SAS de Azure Storage le permite vincular una cuenta de Azure Storage a una factoría de datos Azure con una Firma de acceso compartido (SAS). Proporciona a la instancia de Data Factory acceso restringido o limitado por el tiempo a todos los recursos o a algunos específicos (blob o contenedor) del almacenamiento. Para vincular la cuenta de Azure Storage a una instancia de Data Factory mediante una Firma de acceso compartido, cree un servicio vinculado de SAS de Azure Storage. Para definir un servicio vinculado de SAS de Azure Storage, establezca el **tipo** del servicio vinculado en **AzureStorageSas**. Después, puede especificar las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| sasUri |Especifique el URI de Firma de acceso compartido a los recursos de Azure Storage como blob, contenedor o tabla. |Sí |

##### <a name="example"></a>Ejemplo

```json
{
    "name": "StorageSasLinkedService",
    "properties": {
        "type": "AzureStorageSas",
        "typeProperties": {
            "sasUri": "<storageUri>?<sasToken>"
        }
    }
}
```

Para obtener más información sobre estos servicios vinculados, vea el artículo [Conector de Azure Blob Storage](data-factory-azure-blob-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure Blob, establezca el **tipo** del conjunto de datos en **AzureBlob**. Después, especifique las siguientes propiedades específicas de Azure Blob en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Ruta de acceso para el contenedor y la carpeta en el almacenamiento de blobs. Ejemplo: myblobcontainer\myblobfolder\ |Sí |
| fileName |Nombre del blob. La propiedad fileName es opcional y distingue entre mayúsculas y minúsculas.<br/><br/>Si especifica fileName, la actividad (incluida la copia) funciona en el blob específico.<br/><br/>Cuando no se especifica fileName, la copia incluirá todos los blobs de folderPath para el conjunto de datos de entrada.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado estaría en el siguiente formato: `Data.<Guid>.txt` (por ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |No |
| partitionedBy |partitionedBy es una propiedad opcional. Puede usarla para especificar un folderPath dinámico y un nombre de archivo para datos de series temporales. Por ejemplo, se puede parametrizar folderPath por cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
 ```


Para más información, consulte el artículo [Conector de Azure Blob](data-factory-azure-blob-connector.md#dataset-properties).

### <a name="blobsource-in-copy-activity"></a>BlobSource en la actividad de copia
Si va a copiar datos desde Azure Blob Storage, establezca el **tipo de origen** de la actividad de copia en **BlobSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True (valor predeterminado), False |No |

#### <a name="example-blobsource"></a>Ejemplo: **BlobSource**
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
### <a name="blobsink-in-copy-activity"></a>BlobSink en la actividad de copia
Si va a copiar datos a Azure Blob Storage, establezca el **tipo de receptor** de la actividad de copia en **BlobSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| copyBehavior |Define el comportamiento de copia cuando el origen es BlobSource o FileSystem. |<b>PreserveHierarchy:</b> conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen que apunta a la carpeta de origen es idéntica a la ruta de acceso relativa del archivo de destino que apunta a la carpeta de destino.<br/><br/><b>FlattenHierarchy:</b> todos los archivos de la carpeta de origen están en el primer nivel de la carpeta de destino. Los archivos de destino tienen un nombre generado automáticamente. <br/><br/><b>MergeFiles: (valor predeterminado)</b> combina todos los archivos de la carpeta de origen en un solo archivo. Si se especifica el nombre de archivo/blob, el nombre de archivo combinado sería el nombre especificado; de lo contrario, sería el nombre de archivo generado automáticamente. |No |

#### <a name="example-blobsink"></a>Ejemplo: BlobSink

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Azure Blob](data-factory-azure-blob-connector.md#copy-activity-properties).

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Azure Data Lake Store, establezca el tipo de servicio vinculado en **AzureDataLakeStore** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AzureDataLakeStore** | Sí |
| dataLakeStoreUri | Especifica información sobre la cuenta de Almacén de Azure Data Lake. Tiene el siguiente formato: `https://[accountname].azuredatalakestore.net/webhdfs/v1` o `adl://[accountname].azuredatalakestore.net/`. | Sí |
| subscriptionId | Identificador de suscripción de Azure al que pertenece Data Lake Store. | Necesario para el receptor |
| resourceGroupName | Nombre del grupo de recursos de Azure al que pertenece Data Lake Store. | Necesario para el receptor |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. | Sí (para la autenticación de entidad de servicio) |
| servicePrincipalKey | Especifique la clave de la aplicación. | Sí (para la autenticación de entidad de servicio) |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí (para la autenticación de entidad de servicio) |
| authorization | Haga clic en el botón **Autorizar** de **Data Factory Editor** y escriba sus credenciales; de esta forma, se asigna la dirección URL de autorización generada automáticamente a esta propiedad. | Sí (para la autenticación de credenciales de usuario)|
| sessionID | Id. de sesión de OAuth de la sesión de autorización de OAuth. Cada identificador de sesión es único y solo se puede usar una vez. Esta configuración se genera automáticamente al usar el Editor de Data Factory. | Sí (para la autenticación de credenciales de usuario) |

#### <a name="example-using-service-principal-authentication"></a>Ejemplo: uso de la autenticación de entidad de servicio
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info. Example: microsoft.onmicrosoft.com>"
        }
    }
}
```

#### <a name="example-using-user-credential-authentication"></a>Ejemplo: uso de la autenticación de credenciales de usuario
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Azure Data Lake Store](data-factory-azure-datalake-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure Data Lake Store, establezca el **tipo** del conjunto de datos en **AzureDataLakeStore** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| folderPath |Ruta de acceso al contenedor y a la carpeta del almacén de Azure Data Lake. |Sí |
| fileName |Nombre del archivo en el almacén de Azure Data Lake. La propiedad fileName es opcional y distingue entre mayúsculas y minúsculas. <br/><br/>Si especifica fileName, la actividad (incluida la copia) funciona en el archivo específico.<br/><br/>Cuando no se especifica fileName, la copia incluirá todos los archivos de folderPath para el conjunto de datos de entrada.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado estaría en el siguiente formato: `Data.<Guid>.txt` (por ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |No |
| partitionedBy |partitionedBy es una propiedad opcional. Puede usarla para especificar un folderPath dinámico y un nombre de archivo para datos de series temporales. Por ejemplo, se puede parametrizar folderPath por cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "AzureDataLakeStoreInput",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de Azure Data Lake Store](data-factory-azure-datalake-connector.md#dataset-properties).

### <a name="azure-data-lake-store-source-in-copy-activity"></a>Origen de Azure Data Lake Store en la actividad de copia
Si va a copiar datos desde Azure Data Lake Store, establezca el **tipo de origen** de la actividad de copia en **AzureDataLakeStoreSource** y especifique las siguientes propiedades en la sección **origen**:

**AzureDataLakeStoreSource** admite las siguientes propiedades **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True (valor predeterminado), False |No |

#### <a name="example-azuredatalakestoresource"></a>Ejemplo: AzureDataLakeStoreSource

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureDakeLaketoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureDataLakeStoreInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "AzureDataLakeStoreSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Azure Data Lake Store](data-factory-azure-datalake-connector.md#copy-activity-properties).

### <a name="azure-data-lake-store-sink-in-copy-activity"></a>Receptor de Azure Data Lake Store en la actividad de copia
Si va a copiar datos a Azure Data Lake Store, establezca el **tipo de receptor** de la actividad de copia en **AzureDataLakeStoreSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| copyBehavior |Especifica el comportamiento de copia. |<b>PreserveHierarchy:</b> conserva la jerarquía de archivos en la carpeta de destino. La ruta de acceso relativa del archivo de origen que apunta a la carpeta de origen es idéntica a la ruta de acceso relativa del archivo de destino que apunta a la carpeta de destino.<br/><br/><b>FlattenHierarchy:</b> todos los archivos de la carpeta de origen se crean en el primer nivel de la carpeta de destino. Los archivos de destino se crean con un nombre generado automáticamente.<br/><br/><b>MergeFiles:</b> combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre de archivo/blob, el nombre de archivo combinado sería el nombre especificado; de lo contrario, sería el nombre de archivo generado automáticamente. |No |

#### <a name="example-azuredatalakestoresink"></a>Ejemplo: AzureDataLakeStoreSink
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoDataLake",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureDataLakeStoreOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "AzureDataLakeStoreSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Azure Data Lake Store](data-factory-azure-datalake-connector.md#copy-activity-properties).

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Azure Cosmos DB, establezca el **tipo** del servicio vinculado en **DocumentDb** y especifique las siguientes propiedades en la sección **typeProperties**:

| **Propiedad** | **Descripción** | **Obligatorio** |
| --- | --- | --- |
| connectionString |Especifique la información necesaria para conectarse a la base de datos de Azure Cosmos DB. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CosmosDBLinkedService",
    "properties": {
        "type": "DocumentDb",
        "typeProperties": {
            "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
        }
    }
}
```
Para más información, consulte el artículo [Conector de Azure Cosmos DB](data-factory-azure-documentdb-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure Cosmos DB, establezca el **tipo** del conjunto de datos en **DocumentDbCollection** y especifique las siguientes propiedades en la sección **typeProperties**:

| **Propiedad** | **Descripción** | **Obligatorio** |
| --- | --- | --- |
| collectionName |Nombre de la colección de Azure Cosmos DB. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "PersonCosmosDBTable",
    "properties": {
        "type": "DocumentDbCollection",
        "linkedServiceName": "CosmosDBLinkedService",
        "typeProperties": {
            "collectionName": "Person"
        },
        "external": true,
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```
Para más información, consulte el artículo [Conector de Azure Cosmos DB](data-factory-azure-documentdb-connector.md#dataset-properties).

### <a name="azure-cosmos-db-collection-source-in-copy-activity"></a>Origen de la colección de Azure Cosmos DB en la actividad de copia
Si va a copiar datos desde Azure Cosmos DB, establezca el **tipo de origen** de la actividad de copia en **DocumentDbCollectionSource** y especifique las siguientes propiedades en la sección **origen**:


| **Propiedad** | **Descripción** | **Valores permitidos** | **Obligatorio** |
| --- | --- | --- | --- |
| Query |Especifique la consulta para leer los datos. |Cadena de consulta admitida por Azure Cosmos DB. <br/><br/>Ejemplo: `SELECT c.BusinessEntityID, c.PersonType, c.NameStyle, c.Title, c.Name.First AS FirstName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |No <br/><br/>Si no se especifica, la instrucción SQL que se ejecuta: `select <columns defined in structure> from mycollection` |
| nestingSeparator |Carácter especial para indicar que el documento está anidado |Cualquier carácter. <br/><br/>Azure Cosmos DB es un almacén NoSQL para documentos JSON, en el que se permiten estructuras anidadas. Azure Data Factory permite al usuario indicar la jerarquía a través de nestingSeparator que es "." en los ejemplos anteriores. Con el separador, la actividad de copia generará el objeto "Name" con tres elementos secundarios First, Middle y Last, según "Name.First", "Name.Middle" y "Name.Last" en la definición de tabla. |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "DocDbToBlobPipeline",
    "properties": {
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "DocumentDbCollectionSource",
                    "query": "SELECT Person.Id, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person",
                    "nestingSeparator": "."
                },
                "sink": {
                    "type": "BlobSink",
                    "blobWriterAddHeader": true,
                    "writeBatchSize": 1000,
                    "writeBatchTimeout": "00:00:59"
                }
            },
            "inputs": [{
                "name": "PersonCosmosDBTable"
            }],
            "outputs": [{
                "name": "PersonBlobTableOut"
            }],
            "policy": {
                "concurrency": 1
            },
            "name": "CopyFromCosmosDbToBlob"
        }],
        "start": "2016-04-01T00:00:00",
        "end": "2016-04-02T00:00:00"
    }
}
```

### <a name="azure-cosmos-db-collection-sink-in-copy-activity"></a>Receptor de la colección de Azure Cosmos DB en la actividad de copia
Si va a copiar datos a Azure Cosmos DB, establezca el **tipo de receptor** de la actividad de copia en **DocumentDbCollectionSink** y especifique las siguientes propiedades en la sección **receptor**:

| **Propiedad** | **Descripción** | **Valores permitidos** | **Obligatorio** |
| --- | --- | --- | --- |
| nestingSeparator |Un carácter especial en el nombre de columna de origen que indica que el documento anidado es necesario. <br/><br/>En el ejemplo anterior: `Name.First` en la tabla de salida produce la siguiente estructura JSON en el documento de Cosmos DB:<br/><br/>"Name": {<br/>    "First": "John"<br/>}, |Carácter que se usa para separar los niveles de anidamiento.<br/><br/>El valor predeterminado es `.` (punto). |Carácter que se usa para separar los niveles de anidamiento. <br/><br/>El valor predeterminado es `.` (punto). |
| writeBatchSize |Número de solicitudes paralelas al servicio de Azure Cosmos DB para crear documentos.<br/><br/>Puede ajustar el rendimiento cuando se copian datos en Azure Cosmos DB, y viceversa, mediante esta propiedad. Puede esperar un rendimiento mejor al aumentar writeBatchSize porque se envían más solicitudes paralelas a Azure Cosmos DB. Sin embargo, deberá evitar una limitación de solicitudes que puede generar el mensaje de error: "La tasa de solicitudes es grande".<br/><br/>La limitación de solicitudes se decide mediante una serie de factores, incluidos tamaño de los documentos, número de términos en los documentos, directiva de indexación de colección de destino, etc. Para las operaciones de copia, puede usar una colección mejor (por ejemplo, S3) para obtener el máximo rendimiento disponible (2500 unidades de solicitudes por segundo). |Entero |No (valor predeterminado: 5) |
| writeBatchTimeout |Tiempo de espera para que la operación se complete antes de que se agote el tiempo de espera. |timespan<br/><br/> Ejemplo: "00:30:00" (30 minutos). |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "BlobToDocDbPipeline",
    "properties": {
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "DocumentDbCollectionSink",
                    "nestingSeparator": ".",
                    "writeBatchSize": 2,
                    "writeBatchTimeout": "00:00:00"
                },
                "translator": {
                    "type": "TabularTranslator",
                    "ColumnMappings": "FirstName: Name.First, MiddleName: Name.Middle, LastName: Name.Last, BusinessEntityID: BusinessEntityID, PersonType: PersonType, NameStyle: NameStyle, Title: Title, Suffix: Suffix"
                }
            },
            "inputs": [{
                "name": "PersonBlobTableIn"
            }],
            "outputs": [{
                "name": "PersonCosmosDbTableOut"
            }],
            "policy": {
                "concurrency": 1
            },
            "name": "CopyFromBlobToCosmosDb"
        }],
        "start": "2016-04-14T00:00:00",
        "end": "2016-04-15T00:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de Azure Cosmos DB](data-factory-azure-documentdb-connector.md#copy-activity-properties).

## <a name="azure-sql-database"></a>Azure SQL Database

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Azure SQL Database, establezca el **tipo** de servicio vinculado en **AzureSqlDatabase** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| connectionString |Especifique la información necesaria para conectarse a la base de datos de Azure SQL para la propiedad connectionString. |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Azure SQL](data-factory-azure-sql-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure SQL Database, establezca el **tipo** de conjunto de datos en **AzureSqlTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla o vista en la base de datos de Azure SQL a la que hace referencia el servicio vinculado. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureSqlInput",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
Para más información, consulte el artículo [Conector de Azure SQL](data-factory-azure-sql-connector.md#dataset-properties).

### <a name="sql-source-in-copy-activity"></a>Origen de SQL en la actividad de copia
Si va a copiar datos desde una base de datos de Azure SQL, establezca el **tipo de origen** de la actividad de copia en **SqlSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Ejemplo: `select * from MyTable`. |No |
| sqlReaderStoredProcedureName |Nombre del procedimiento almacenado que lee datos de la tabla de origen. |Nombre del procedimiento almacenado. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado. |Pares nombre-valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
Para más información, consulte el artículo [Conector de Azure SQL](data-factory-azure-sql-connector.md#copy-activity-properties).

### <a name="sql-sink-in-copy-activity"></a>Receptor de SQL en la actividad de copia
Si va a copiar datos a Azure SQL Database, establezca el **tipo de receptor** de la actividad de copia en **SqlSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| writeBatchTimeout |Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera. |timespan<br/><br/> Ejemplo: "00:30:00" (30 minutos). |No |
| writeBatchSize |Inserta datos en la tabla SQL cuando el tamaño del búfer alcanza el valor writeBatchSize. |Entero (número de filas) |No (valor predeterminado: 10000) |
| sqlWriterCleanupScript |Especifique una consulta para que se ejecute la actividad de copia de tal forma que se limpien los datos de un segmento específico. |Una instrucción de consulta. |No |
| sliceIdentifierColumnName |Especifique el nombre de columna para que la rellene la actividad de copia con un identificador de segmentos generado automáticamente, que se usará para limpiar los datos de un segmento específico cuando se vuelva a ejecutar. |Nombre de columna de una columna con el tipo de datos binarios (32). |No |
| sqlWriterStoredProcedureName |Nombre del procedimiento almacenado que actualiza e inserta (operación de upsert) datos en la tabla de destino. |Nombre del procedimiento almacenado. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado. |Pares nombre-valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |
| sqlWriterTableType |Especifique el nombre del tipo de tabla que se usará en el procedimiento almacenado anterior. La actividad de copia dispone que los datos que se mueven estén disponibles en una tabla temporal con este tipo de tabla. El código de procedimiento almacenado puede combinar los datos copiados con datos existentes. |Un nombre de tipo de tabla. |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Azure SQL](data-factory-azure-sql-connector.md#copy-activity-properties).

## <a name="azure-synapse-analytics"></a>Azure Synapse Analytics

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Azure Synapse Analytics, establezca el **tipo** de servicio vinculado en **AzureSqlDW** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| connectionString |Especifique la información necesaria para conectarse a la instancia de Azure Synapse Analytics para la propiedad connectionString. |Sí |



#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure Synapse Analytics, establezca el **tipo** de conjunto de datos en **AzureSqlDWTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla o vista en la base de datos de Azure Synapse Analytics a la que hace referencia el servicio vinculado. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureSqlDWInput",
    "properties": {
    "type": "AzureSqlDWTable",
        "linkedServiceName": "AzureSqlDWLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#dataset-properties).

### <a name="azure-synapse-analytics-source-in-copy-activity"></a>Origen de Azure Synapse Analytics en la actividad de copia
Si va a copiar datos desde Azure Synapse Analytics, establezca el **tipo de origen** de la actividad de copia en **SqlDWSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. |No |
| sqlReaderStoredProcedureName |Nombre del procedimiento almacenado que lee datos de la tabla de origen. |Nombre del procedimiento almacenado. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado. |Pares nombre-valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLDWtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSqlDWInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlDWSource",
                    "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties).

### <a name="azure-synapse-analytics-sink-in-copy-activity"></a>Receptor de Azure Synapse Analytics en la actividad de copia
Si va a copiar datos a Azure Synapse Analytics, establezca el **tipo de receptor** de la actividad de copia en **SqlDWSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| sqlWriterCleanupScript |Especifique una consulta para que se ejecute la actividad de copia de tal forma que se limpien los datos de un segmento específico. |Una instrucción de consulta. |No |
| allowPolyBase |Indica si se usa PolyBase (si procede) en lugar del mecanismo BULKINSERT. <br/><br/> **El uso de PolyBase es el método recomendado para cargar datos en Azure Synapse Analytics.** |True <br/>False (valor predeterminado) |No |
| polyBaseSettings |Un grupo de propiedades que se pueden especificar si el valor de la propiedad **allowPolybase** está establecido en **true**. |&nbsp; |No |
| rejectValue |Especifica el número o porcentaje de filas que se pueden rechazar antes de que se produzca un error en la consulta. <br/><br/>Más información sobre las opciones de rechazo de PolyBase en la sección **Argumentos** del tema [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql) [CREAR UNA TABLA EXTERNA (Transact-SQL)]. |0 (predeterminado), 1, 2, … |No |
| rejectType |Especifica si se indica la opción rejectValue como un valor literal o un porcentaje. |Valor (predeterminado), Porcentaje |No |
| rejectSampleValue |Determina el número de filas que se van a recuperar antes de que PolyBase vuelva a calcular el porcentaje de filas rechazadas. |1, 2, … |Sí, si el valor de **rejectType** es **percentage** |
| useTypeDefault |Especifica cómo administrar valores que faltan en archivos de texto delimitado cuando PolyBase recupera datos del archivo de texto.<br/><br/>Más información sobre esta propiedad en la sección de argumentos de [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql). |True, False (predeterminada) |No |
| writeBatchSize |Inserta datos en la tabla SQL cuando el tamaño del búfer alcance writeBatchSize |Entero (número de filas) |No (valor predeterminado: 10000) |
| writeBatchTimeout |Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera. |timespan<br/><br/> Ejemplo: "00:30:00" (30 minutos). |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQLDW",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlDWOutput"
            }],
            "typeProperties": {
                "source": {
                "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlDWSink",
                    "allowPolyBase": true
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties).

## <a name="azure-cognitive-search"></a>Azure Cognitive Search

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Azure Cognitive Search, establezca el **tipo** de servicio vinculado en **AzureSearch** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| -------- | ----------- | -------- |
| url | URL del servicio de búsqueda. | Sí |
| key | Clave de administración del servicio de búsqueda. | Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureSearchLinkedService",
    "properties": {
        "type": "AzureSearch",
        "typeProperties": {
            "url": "https://<service>.search.windows.net",
            "key": "<AdminKey>"
        }
    }
}
```

Para obtener más información, consulte el artículo [Conector de Azure Cognitive Search](data-factory-azure-search-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure Cognitive Search, establezca el **tipo** de conjunto de datos en **AzureSearchIndex** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| -------- | ----------- | -------- |
| type | La propiedad type debe establecerse en **AzureSearchIndex**.| Sí |
| indexName | Nombre del índice de búsqueda. Data Factory no crea el índice. El índice debe existir en Azure Cognitive Search. | Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureSearchIndexDataset",
    "properties": {
        "type": "AzureSearchIndex",
        "linkedServiceName": "AzureSearchLinkedService",
        "typeProperties": {
            "indexName": "products"
        },
        "availability": {
            "frequency": "Minute",
            "interval": 15
        }
    }
}
```

Para obtener más información, consulte el artículo [Conector de Azure Cognitive Search](data-factory-azure-search-connector.md#dataset-properties).

### <a name="azure-cognitive-search-index-sink-in-copy-activity"></a>Receptor del índice de Azure Cognitive Search en la actividad de copia
Si va a copiar datos en un índice de búsqueda, establezca el **tipo de receptor** de la actividad de copia en **AzureSearchIndexSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| -------- | ----------- | -------------- | -------- |
| WriteBehavior | Especifica si, cuando ya haya un documento en el índice, se realizará una operación de combinación o de reemplazo. | Combinar (predeterminado)<br/>Cargar| No |
| WriteBatchSize | Carga datos en el índice de búsqueda cuando el tamaño del búfer alcanza el valor de writeBatchSize. | De 1 a 1000. El valor predeterminado es 1000. | No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "SqlServertoAzureSearchIndex",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " SqlServerInput"
            }],
            "outputs": [{
                "name": "AzureSearchIndexDataset"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "AzureSearchIndexSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para obtener más información, consulte el artículo [Conector de Azure Cognitive Search](data-factory-azure-search-connector.md#copy-activity-properties).

## <a name="azure-table-storage"></a>Azure Table Storage

### <a name="linked-service"></a>Servicio vinculado
Existen dos tipos de servicios vinculados: servicio vinculado de Azure Storage y servicio vinculado de SAS de Azure Storage.

#### <a name="azure-storage-linked-service"></a>Servicio vinculado de Azure Storage
Para vincular la cuenta de Azure Storage a una instancia de Data Factory mediante una **clave de cuenta**, cree un servicio vinculado de Azure Storage. Para definir un servicio vinculado de Azure Storage, establezca el **tipo** del servicio vinculado en **AzureStorage**. Después, puede especificar las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |La propiedad type debe establecerse en: **AzureStorage** |Sí |
| connectionString |Especifique la información necesaria para conectarse a Almacenamiento de Azure para la propiedad connectionString. |Sí |

**Ejemplo**:

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

#### <a name="azure-storage-sas-linked-service"></a>Servicio vinculado de SAS de Azure Storage
El servicio vinculado de SAS de Azure Storage le permite vincular una cuenta de Azure Storage a una factoría de datos Azure con una Firma de acceso compartido (SAS). Proporciona a la instancia de Data Factory acceso restringido o limitado por el tiempo a todos los recursos o a algunos específicos (blob o contenedor) del almacenamiento. Para vincular la cuenta de Azure Storage a una instancia de Data Factory mediante una Firma de acceso compartido, cree un servicio vinculado de SAS de Azure Storage. Para definir un servicio vinculado de SAS de Azure Storage, establezca el **tipo** del servicio vinculado en **AzureStorageSas**. Después, puede especificar las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |La propiedad type debe establecerse en: **AzureStorageSas** |Sí |
| sasUri |Especifique el URI de Firma de acceso compartido a los recursos de Azure Storage como blob, contenedor o tabla. |Sí |

**Ejemplo**:

```json
{
    "name": "StorageSasLinkedService",
    "properties": {
        "type": "AzureStorageSas",
        "typeProperties": {
            "sasUri": "<storageUri>?<sasToken>"
        }
    }
}
```

Para más información sobre estos servicios vinculados, vea el artículo [Conector de Azure Table Storage](data-factory-azure-table-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Azure Table, establezca el **tipo** de conjunto de datos en **AzureTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de Base de datos de tablas de Azure a la que hace referencia el servicio vinculado. |Sí. Cuando se especifica un elemento tableName sin azureTableSourceQuery, se copian todos los registros de la tabla en el destino. Si también se especifica azureTableSourceQuery, los registros de la tabla que satisfacen los requisitos de la consulta se copian en el destino. |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AzureTableInput",
    "properties": {
        "type": "AzureTable",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información sobre estos servicios vinculados, vea el artículo [Conector de Azure Table Storage](data-factory-azure-table-connector.md#dataset-properties).

### <a name="azure-table-source-in-copy-activity"></a>Origen de Azure Table en la actividad de copia
Si va a copiar datos desde Azure Table Storage, establezca el **tipo de origen** de la actividad de copia en **AzureTableSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| AzureTableSourceQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta de tabla de Azure. Consulte los ejemplos en la sección siguiente. |No. Cuando se especifica un elemento tableName sin azureTableSourceQuery, se copian todos los registros de la tabla en el destino. Si también se especifica azureTableSourceQuery, los registros de la tabla que satisfacen los requisitos de la consulta se copian en el destino. |
| azureTableSourceIgnoreTableNotFound |Indica si se omite la excepción de la tabla inexistente. |TRUE<br/>FALSE |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureTabletoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureTableInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "AzureTableSource",
                    "AzureTableSourceQuery": "PartitionKey eq 'DefaultPartitionKey'"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información sobre estos servicios vinculados, vea el artículo [Conector de Azure Table Storage](data-factory-azure-table-connector.md#copy-activity-properties).

### <a name="azure-table-sink-in-copy-activity"></a>Receptor de Azure Table en la actividad de copia
Si va a copiar datos a Azure Table Storage, establezca el **tipo de receptor** de la actividad de copia en **AzureTableSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| azureTableDefaultPartitionKeyValue |Valor predeterminado de la clave de la partición que puede usar el receptor. |Valor de cadena. |No |
| azureTablePartitionKeyName |Especifique el nombre de la columna cuyos valores se usan como claves de partición. Si no se especifica, se utiliza AzureTableDefaultPartitionKeyValue como clave de la partición. |Un nombre de columna. |No |
| azureTableRowKeyName |Especifique el nombre de la columna cuyos valores se usan como claves de fila. Si no se especifica, use un GUID para cada fila. |Un nombre de columna. |No |
| azureTableInsertType |Modo de insertar datos en la tabla de Azure.<br/><br/>Esta propiedad controla si los valores de las filas existentes en la tabla de salida con claves de partición y de fila coincidentes se van a reemplazar o a combinar. <br/><br/>Consulte los temas [Insert or Merge Entity](/rest/api/storageservices/Insert-Or-Merge-Entity) (Insertar o combinar entidad) e [Insert or Replace Entity](/rest/api/storageservices/Insert-Or-Replace-Entity) (Insertar o remplazar entidad) para más información sobre cómo funcionan estas opciones (combinación y reemplazo). <br/><br> Esta configuración se aplica en el nivel de fila, no en el nivel de tabla, y ninguna opción elimina filas de la tabla de salida que no existan en la entrada. |merge (predeterminado)<br/>replace |No |
| writeBatchSize |Inserta datos en la tabla de Azure cuando se alcanza el valor de writeBatchSize o writeBatchTimeout. |Entero (número de filas) |No (valor predeterminado: 10000) |
| writeBatchTimeout |Inserta datos en la tabla de Azure cuando se alcanza el valor de writeBatchSize o writeBatchTimeout. |timespan<br/><br/>Ejemplo: "00:20:00" (20 minutos) |No (el valor predeterminado de intervalo de tiempo del cliente de almacenamiento es 90 segundos) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoTable",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureTableOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "AzureTableSink",
                    "writeBatchSize": 100,
                    "writeBatchTimeout": "01:00:00"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
Para más información sobre estos servicios vinculados, vea el artículo [Conector de Azure Table Storage](data-factory-azure-table-connector.md#copy-activity-properties).

## <a name="amazon-redshift"></a>Amazon RedShift

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Amazon Redshift, establezca el **tipo** de servicio vinculado en **AmazonRedshift** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Dirección IP o nombre de host del servidor de Amazon Redshift. |Sí |
| port |El número del puerto TCP que el servidor de Amazon Redshift utiliza para escuchar las conexiones del cliente. |No, el valor predeterminado es: 5439 |
| database |Nombre de la base de datos de Amazon Redshift. |Sí |
| username |Nombre del usuario que tiene acceso a la base de datos. |Sí |
| password |Contraseña para la cuenta de usuario. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties": {
        "type": "AmazonRedshift",
        "typeProperties": {
            "server": "<Amazon Redshift host name or IP address>",
            "port": 5439,
            "database": "<database name>",
            "username": "user",
            "password": "password"
        }
    }
}
```

Para más información, consulte el artículo Conector de Amazon Redshift.

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Amazon Redshift, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la base de datos de Amazon Redshift a la que hace referencia el servicio vinculado. |No (si se especifica **query** de **RelationalSource**) |


#### <a name="example"></a>Ejemplo

```json
{
    "name": "AmazonRedshiftInputDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "AmazonRedshiftLinkedService",
        "typeProperties": {
            "tableName": "<Table name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
Para más información, consulte el artículo Conector de Amazon Redshift.

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde Amazon Redshift, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. |No (si se especifica **tableName** de **dataset**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopyAmazonRedshiftToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "AmazonRedshiftInputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "AmazonRedshiftToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```
Para más información, consulte el artículo Conector de Amazon Redshift.

## <a name="ibm-db2"></a>IBM DB2

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de IBM DB2, establezca el **tipo** de servicio vinculado en **OnPremisesDB2** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Nombre del servidor DB2. |Sí |
| database |Nombre de la base de datos DB2. |Sí |
| esquema |Nombre del esquema de la base de datos. El nombre del esquema distingue mayúsculas de minúsculas. |No |
| authenticationType |Tipo de autenticación usado para conectarse a la base de datos DB2. Los valores posibles son: Anonymous, Basic y Windows. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación Basic o Windows. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Factoría de datos para conectarse a la base de datos DB2 local. |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "OnPremDb2LinkedService",
    "properties": {
        "type": "OnPremisesDb2",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```
Para más información, consulte el artículo Conector de IBM DB2.

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de DB2, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de base de datos DB2 a la que hace referencia el servicio vinculado. tableName distingue mayúsculas de minúsculas. |No (si se especifica **query** de **RelationalSource**)

#### <a name="example"></a>Ejemplo
```json
{
    "name": "Db2DataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremDb2LinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo Conector de IBM DB2.

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde IBM DB2, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `"query": "select * from "MySchema"."MyTable""`. |No (si se especifica **tableName** de **dataset**) |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "CopyDb2ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from \"Orders\""
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "Db2DataSet"
            }],
            "outputs": [{
                "name": "AzureBlobDb2DataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "Db2ToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```
Para más información, consulte el artículo Conector de IBM DB2.

## <a name="mysql"></a>MySQL

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de MySQL, establezca el **tipo** de servicio vinculado en **OnPremisesMySql** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Nombre del servidor MySQL. |Sí |
| database |Nombre de la base de datos MySQL. |Sí |
| esquema |Nombre del esquema de la base de datos. |No |
| authenticationType |Tipo de autenticación usado para conectarse a la Base de datos MySQL. Los valores posibles son: `Basic`. |Sí |
| userName |Especifique el nombre de usuario para conectarse a la base de datos de MySQL. |Sí |
| password |Especifique la contraseña de la cuenta de usuario especificada. |Sí |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Factoría de datos para conectarse a la Base de datos MySQL local. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "OnPremMySqlLinkedService",
    "properties": {
        "type": "OnPremisesMySql",
        "typeProperties": {
            "server": "<server name>",
            "database": "<database name>",
            "schema": "<schema name>",
            "authenticationType": "<authentication type>",
            "userName": "<user name>",
            "password": "<password>",
            "gatewayName": "<gateway>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de MySQL](data-factory-onprem-mysql-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de MySQL, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de la Base de datos MySQL a la que hace referencia el servicio vinculado. |No (si se especifica **query** de **RelationalSource**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "MySqlDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremMySqlLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
Para más información, consulte el artículo [Conector de MySQL](data-factory-onprem-mysql-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde una base de datos MySQL, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. |No (si se especifica **tableName** de **dataset**) |


#### <a name="example"></a>Ejemplo
```json
{
    "name": "CopyMySqlToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "MySqlDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobMySqlDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "MySqlToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de MySQL](data-factory-onprem-mysql-connector.md#copy-activity-properties).

## <a name="oracle"></a>Oracle

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Oracle, establezca el **tipo** de servicio vinculado en **OnPremisesOracle** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| driverType | Especifique qué controlador usar para copiar datos en bases de datos de Oracle o desde ellas. Los valores permitidos son **Microsoft** u **ODP** (valor predeterminado). Consulte la sección Versiones compatibles e instalación para obtener información detallada sobre los controladores. | No |
| connectionString | Especifique la información necesaria para conectarse a la instancia de Base de datos de Oracle para la propiedad connectionString. | Sí |
| gatewayName | Nombre de la puerta de enlace que se usa para conectarse al servidor local de Oracle |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Oracle](data-factory-onprem-oracle-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Oracle, establezca el **tipo** de conjunto de datos en **OracleTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de Base de datos de Oracle a la que hace referencia el servicio vinculado. |No (si se especifica **oracleReaderQuery** de **OracleSource**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2016-02-27T12:00:00",
            "frequency": "Hour"
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
Para más información, consulte el artículo [Conector de Oracle](data-factory-onprem-oracle-connector.md#dataset-properties).

### <a name="oracle-source-in-copy-activity"></a>Origen de Oracle en la actividad de copia
Si va a copiar datos desde una base de datos de Oracle, establezca el **tipo de origen** de la actividad de copia en **OracleSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| oracleReaderQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable` <br/><br/>Si no se especifica, la instrucción SQL que se ejecuta: `select * from MyTable` |No (si se especifica **tableName** de **dataset**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "OracletoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " OracleInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "OracleSource",
                    "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
            "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Oracle](data-factory-onprem-oracle-connector.md#copy-activity-properties).

### <a name="oracle-sink-in-copy-activity"></a>Receptor de Oracle en la actividad de copia
Si va a copiar datos a una base de datos de Oracle, establezca el **tipo de receptor** de la actividad de copia en **OracleSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| writeBatchTimeout |Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera. |timespan<br/><br/> Ejemplo: 00:30:00 (30 minutos). |No |
| writeBatchSize |Inserta datos en la tabla SQL cuando el tamaño del búfer alcanza el valor writeBatchSize. |Entero (número de filas) |No (valor predeterminado: 100) |
| sqlWriterCleanupScript |Especifique una consulta para que se ejecute la actividad de copia de tal forma que se limpien los datos de un segmento específico. |Una instrucción de consulta. |No |
| sliceIdentifierColumnName |Especifique el nombre de columna para que la rellene la actividad de copia con un identificador de segmentos generado automáticamente, que se usará para limpiar los datos de un segmento específico cuando se vuelva a ejecutar. |Nombre de columna de una columna con el tipo de datos binarios (32). |No |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-05T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoOracle",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "OracleOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "OracleSink"
                }
            },
            "scheduler": {
                "frequency": "Day",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
Para más información, consulte el artículo [Conector de Oracle](data-factory-onprem-oracle-connector.md#copy-activity-properties).

## <a name="postgresql"></a>PostgreSQL

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de PostgreSQL, establezca el **tipo** de servicio vinculado en **OnPremisesPostgreSql** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Nombre del servidor de PostgreSQL. |Sí |
| database |Nombre de la base de datos de PostgreSQL. |Sí |
| esquema |Nombre del esquema de la base de datos. El nombre del esquema distingue mayúsculas de minúsculas. |No |
| authenticationType |Tipo de autenticación usado para conectarse a la base de datos de PostgreSQL. Los valores posibles son: Anonymous, Basic y Windows. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación Basic o Windows. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Factoría de datos para conectarse a la base de datos de PostgreSQL local. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "OnPremPostgreSqlLinkedService",
    "properties": {
        "type": "OnPremisesPostgreSql",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```
Para más información, consulte el artículo [Conector de PostgreSQL](data-factory-onprem-postgresql-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de PostgreSQL, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de Base de datos de PostgreSQL a la que hace referencia el servicio vinculado. tableName distingue mayúsculas de minúsculas. |No (si se especifica **query** de **RelationalSource**) |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "PostgreSqlDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremPostgreSqlLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
Para más información, consulte el artículo [Conector de PostgreSQL](data-factory-onprem-postgresql-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde una base de datos PostgreSQL, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: "query": "select * from \"MySchema\".\"MyTable\"". |No (si se especifica **tableName** de **dataset**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopyPostgreSqlToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from \"public\".\"usstates\""
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "PostgreSqlDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobPostgreSqlDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "PostgreSqlToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de PostgreSQL](data-factory-onprem-postgresql-connector.md#copy-activity-properties).

## <a name="sap-business-warehouse"></a>SAP Business Warehouse


### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de SAP Business Warehouse (BW), establezca el **tipo** de servicio vinculado en **SapBw** y especifique las siguientes propiedades en la sección **typeProperties**:

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
server | Nombre del servidor en el que reside la instancia de SAP BW. | string | Sí
systemNumber | Número del sistema de SAP BW. | Número decimal de dos dígitos que se representa en forma de cadena. | Sí
clientId | Identificador del cliente en el sistema SAP W. | Número decimal de tres dígitos que se representa en forma de cadena. | Sí
username | Nombre del usuario que tiene acceso al servidor SAP | string | Sí
password | Contraseña del usuario. | string | Sí
gatewayName | Nombre de la puerta de enlace que debe usar el servicio Data Factory para conectarse a la instancia de SAP BW local. | string | Sí
encryptedCredential | La cadena de credenciales cifrada. | string | No

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SapBwLinkedService",
    "properties": {
        "type": "SapBw",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de SAP Business Warehouse](data-factory-sap-business-warehouse-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de SAP BW, establezca el **tipo** de conjunto de datos en **RelationalTable**. No hay ninguna propiedad específica del tipo compatible con el conjunto de datos de SAP BW de tipo **RelationalTable**.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SapBwDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapBwLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
Para más información, consulte el artículo [Conector de SAP Business Warehouse](data-factory-sap-business-warehouse-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde SAP Business Warehouse, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query | Especifica la consulta MDX para leer datos de la instancia de SAP BW. | Consulta MDX. | Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopySapBwToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "<MDX query for SAP BW>"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "SapBwDataset"
            }],
            "outputs": [{
                "name": "AzureBlobDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SapBwToBlob"
        }],
        "start": "2017-03-01T18:00:00",
        "end": "2017-03-01T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de SAP Business Warehouse](data-factory-sap-business-warehouse-connector.md#copy-activity-properties).

## <a name="sap-hana"></a>SAP HANA

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de SAP HANA, establezca el **tipo** de servicio vinculado en **SapHana** y especifique las siguientes propiedades en la sección **typeProperties**:

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
server | Nombre del servidor en el que reside la instancia de SAP HANA. Si el servidor usa un puerto personalizado, especifique `server:port`. | string | Sí
authenticationType | Tipo de autenticación. | cadena. "Basic" o "Windows" | Sí
username | Nombre del usuario que tiene acceso al servidor SAP | string | Sí
password | Contraseña del usuario. | string | Sí
gatewayName | Nombre de la puerta de enlace que debe usar el servicio Data Factory para conectarse a la instancia de SAP HANA local. | string | Sí
encryptedCredential | La cadena de credenciales cifrada. | string | No

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "server": "<server name>",
            "authenticationType": "<Basic, or Windows>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}

```
Para más información, consulte el artículo [Conector de SAP HANA](data-factory-sap-hana-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de SAP HANA, establezca el **tipo** de conjunto de datos en **RelationalTable**. No hay ninguna propiedad específica del tipo compatible con el conjunto de datos de SAP HANA de tipo **RelationalTable**.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SapHanaDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapHanaLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
Para más información, consulte el artículo [Conector de SAP HANA](data-factory-sap-hana-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde un almacén de datos de SAP HANA, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query | Especifica la consulta SQL para leer datos de la instancia de SAP HANA. | Consulta SQL. | Sí |


#### <a name="example"></a>Ejemplo


```json
{
    "name": "CopySapHanaToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "<SQL Query for HANA>"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "SapHanaDataset"
            }],
            "outputs": [{
                "name": "AzureBlobDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SapHanaToBlob"
        }],
        "start": "2017-03-01T18:00:00",
        "end": "2017-03-01T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de SAP HANA](data-factory-sap-hana-connector.md#copy-activity-properties).


## <a name="sql-server"></a>SQL Server

### <a name="linked-service"></a>Servicio vinculado
Se creará un servicio vinculado de tipo **OnPremisesSqlServer** para vincular una base de datos SQL Server local a una factoría de datos. En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio SQL Server vinculado.

En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio SQL Server vinculado.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type se debe establecer en: **OnPremisesSqlServer**. |Sí |
| connectionString |Especifique la información de connectionString necesaria para conectarse a la base de datos SQL Server mediante autenticación de SQL o autenticación de Windows. |Sí |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Data Factory para conectarse a la base de datos de SQL Server. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación de Windows. Ejemplo: **nombreDeDominio\\username**. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |

Puede cifrar las credenciales con el cmdlet **New-AzDataFactoryEncryptValue** y usarlas en la cadena de conexión, como se muestra en el ejemplo siguiente (propiedad **EncryptedCredential**):

```json
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```


#### <a name="example-json-for-using-sql-authentication"></a>Ejemplo: JSON para usar autenticación de SQL

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
#### <a name="example-json-for-using-windows-authentication"></a>Ejemplo: JSON para usar autenticación de Windows

Si se especifican el nombre de usuario y la contraseña, la puerta de enlace los usa para suplantar la cuenta de usuario especificada para conectarse a la base de datos de SQL Server. En caso contrario, la puerta de enlace se conectará directamente a SQL Server con el contexto de seguridad de puerta de enlace (su cuenta de inicio).

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de SQL Server](data-factory-sqlserver-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de SQL Server, establezca el **tipo** de conjunto de datos en **SqlServerTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de base de datos de SQL Server a la que hace referencia el servicio vinculado. |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "SqlServerInput",
    "properties": {
        "type": "SqlServerTable",
        "linkedServiceName": "SqlServerLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de SQL Server](data-factory-sqlserver-connector.md#dataset-properties).

### <a name="sql-source-in-copy-activity"></a>Origen de SQL en la actividad de copia
Si va a copiar datos de una base de datos SQL Server, establezca el **tipo de origen** de la actividad de copia en **SqlSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. Es posible hacer referencia a varias tablas de la base de datos a la que hace referencia el conjunto de datos de entrada. Si no se especifica, la instrucción SQL que se ejecuta: select from MyTable. |No |
| sqlReaderStoredProcedureName |Nombre del procedimiento almacenado que lee datos de la tabla de origen. |Nombre del procedimiento almacenado. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado. |Pares nombre-valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |

Si se especifica **sqlReaderQuery** para SqlSource, la actividad de copia ejecuta la consulta en el origen de Base de datos SQL de Azure para obtener los datos.

Como alternativa, puede indicar un procedimiento almacenado mediante la especificación de **sqlReaderStoredProcedureName** y **storedProcedureParameters** (si el procedimiento almacenado toma parámetros).

Si no especifica sqlReaderQuery ni sqlReaderStoredProcedureName, las columnas definidas en la sección sobre la estructura se usan para crear una consulta Select para ejecutarla en Base de datos de SQL Server. Si la definición del conjunto de datos no tiene la estructura, se seleccionan todas las columnas de la tabla.

> [!NOTE]
> Cuando use **sqlReaderStoredProcedureName**, necesitará especificar un valor para la propiedad **tableName** del conjunto de datos JSON. Pero no se ha realizado ninguna validación en esta tabla.


#### <a name="example"></a>Ejemplo
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "SqlServertoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " SqlServerInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

En este ejemplo, **sqlReaderQuery** se especifica para SqlSource. La actividad de copia ejecuta esta consulta en el origen de Base de datos de SQL Server para obtener los datos. Como alternativa, puede indicar un procedimiento almacenado mediante la especificación de **sqlReaderStoredProcedureName** y **storedProcedureParameters** (si el procedimiento almacenado toma parámetros). sqlReaderQuery puede hacer referencia a varias tablas de la base de datos a la que hace referencia el conjunto de datos de entrada. No se limita exclusivamente a la tabla establecida como typeProperty tableName del conjunto de datos.

Si no especifica sqlReaderQuery ni sqlReaderStoredProcedureName, las columnas definidas en la sección sobre la estructura se usan para crear una consulta Select para ejecutarla en Base de datos de SQL Server. Si la definición del conjunto de datos no tiene la estructura, se seleccionan todas las columnas de la tabla.

Para más información, consulte el artículo [Conector de SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties).

### <a name="sql-sink-in-copy-activity"></a>Receptor de SQL en la actividad de copia
Si va a copiar datos a una base de datos SQL Server, establezca el **tipo de receptor** de la actividad de copia en **SqlSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| writeBatchTimeout |Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera. |timespan<br/><br/> Ejemplo: "00:30:00" (30 minutos). |No |
| writeBatchSize |Inserta datos en la tabla SQL cuando el tamaño del búfer alcanza el valor writeBatchSize. |Entero (número de filas) |No (valor predeterminado: 10000) |
| sqlWriterCleanupScript |Especifique la consulta para que la actividad de copia se ejecute de tal forma que se limpien los datos de un segmento específico. Consulte la sección sobre repetibilidad para más información. |Una instrucción de consulta. |No |
| sliceIdentifierColumnName |Especifique el nombre de columna para que la rellene la actividad de copia con un identificador de segmentos generado automáticamente, que se usará para limpiar los datos de un segmento específico cuando se vuelva a ejecutar. Consulte la sección sobre repetibilidad para más información. |Nombre de columna de una columna con el tipo de datos binarios (32). |No |
| sqlWriterStoredProcedureName |Nombre del procedimiento almacenado que actualiza e inserta (operación de upsert) datos en la tabla de destino. |Nombre del procedimiento almacenado. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado. |Pares nombre-valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |
| sqlWriterTableType |Especifique el nombre del tipo de tabla que se usará en el procedimiento almacenado anterior. La actividad de copia dispone que los datos que se mueven estén disponibles en una tabla temporal con este tipo de tabla. El código de procedimiento almacenado puede combinar los datos copiados con datos existentes. |Un nombre de tipo de tabla. |No |

#### <a name="example"></a>Ejemplo
La canalización contiene una actividad de copia que está configurada para usar estos conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **BlobSource** y el tipo **sink**, en **SqlSink**.

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": " SqlServerOutput "
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties).

## <a name="sybase"></a>Sybase

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Sybase, establezca el **tipo** de servicio vinculado en **OnPremisesSybase** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Nombre del servidor de Sybase. |Sí |
| database |Nombre de la base de datos Sybase. |Sí |
| esquema |Nombre del esquema de la base de datos. |No |
| authenticationType |Tipo de autenticación usado para conectarse a la base de datos Sybase. Los valores posibles son: Anonymous, Basic y Windows. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación Basic o Windows. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Factoría de datos para conectarse a la base de datos de Sybase local. |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "OnPremSybaseLinkedService",
    "properties": {
        "type": "OnPremisesSybase",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Sybase](data-factory-onprem-sybase-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Sybase, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en la instancia de Base de datos Sybase a la que hace referencia el servicio vinculado. |No (si se especifica **query** de **RelationalSource**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SybaseDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremSybaseLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de Sybase](data-factory-onprem-sybase-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde una base de datos de Sybase, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. |No (si se especifica **tableName** de **dataset**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopySybaseToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from DBA.Orders"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "SybaseDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobSybaseDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SybaseToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de Sybase](data-factory-onprem-sybase-connector.md#copy-activity-properties).

## <a name="teradata"></a>Teradata

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Teradata, establezca el **tipo** de servicio vinculado en **OnPremisesTeradata** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Nombre del servidor de Teradata. |Sí |
| authenticationType |Tipo de autenticación usado para conectarse a la base de datos Teradata. Los valores posibles son: Anonymous, Basic y Windows. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación Basic o Windows. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Factoría de datos para conectarse a la base de datos Teradata local. |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "OnPremTeradataLinkedService",
    "properties": {
        "type": "OnPremisesTeradata",
        "typeProperties": {
            "server": "<server>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Teradata](data-factory-onprem-teradata-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de blob de Teradata, establezca el **tipo** de conjunto de datos en **RelationalTable**. Actualmente, no hay ninguna propiedad de tipo compatible con el conjunto de datos de Teradata.

#### <a name="example"></a>Ejemplo
```json
{
    "name": "TeradataDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremTeradataLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de Teradata](data-factory-onprem-teradata-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde una base de datos de Teradata, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopyTeradataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', SliceStart, SliceEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "TeradataDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobTeradataDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "TeradataToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "isPaused": false
    }
}
```

Para más información, consulte el artículo [Conector de Teradata](data-factory-onprem-teradata-connector.md#copy-activity-properties).

## <a name="cassandra"></a>Cassandra


### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Cassandra, establezca el **tipo** de servicio vinculado en **OnPremisesCassandra** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| host |Una o varias direcciones IP o nombres de host de los servidores de Cassandra.<br/><br/>Especifica una lista de direcciones IP o nombres de host separada por comas para conectar con todos los servidores a la vez. |Sí |
| port |Puerto TCP que el servidor de Cassandra utiliza para escuchar las conexiones del cliente. |No, el valor predeterminado es: 9042 |
| authenticationType |Básica o anónima |Sí |
| username |Especifique el nombre de usuario de la cuenta de usuario. |Sí, si el valor de authenticationType es Basic. |
| password |Especifique la contraseña para la cuenta de usuario. |Sí, si el valor de authenticationType es Basic. |
| gatewayName |Nombre de la puerta de enlace que se va a utilizar en la conexión con la base de datos de Cassandra local. |Sí |
| encryptedCredential |Credencial cifrada por la puerta de enlace. |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CassandraLinkedService",
    "properties": {
        "type": "OnPremisesCassandra",
        "typeProperties": {
            "authenticationType": "Basic",
            "host": "<cassandra server name or IP address>",
            "port": 9042,
            "username": "user",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Cassandra](data-factory-onprem-cassandra-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Cassandra, establezca el **tipo** de conjunto de datos en **CassandraTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| keyspace |Nombre del espacio de claves o esquema de la base de datos de Cassandra. |Sí (si no hay definida ninguna **consulta** para **CassandraSource**). |
| tableName |Nombre de la tabla de la base de datos de Cassandra. |Sí (si no hay definida ninguna **consulta** para **CassandraSource**). |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CassandraInput",
    "properties": {
        "linkedServiceName": "CassandraLinkedService",
        "type": "CassandraTable",
        "typeProperties": {
            "tableName": "mytable",
            "keySpace": "<key space>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de Cassandra](data-factory-onprem-cassandra-connector.md#dataset-properties).

### <a name="cassandra-source-in-copy-activity"></a>Origen de Cassandra en la actividad de copia
Si va a copiar datos desde Cassandra, establezca el **tipo de origen** de la actividad de copia en **CassandraSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Consulta SQL-92 o consulta CQL. Vea la [CQL reference](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html)(referencia de CQL). <br/><br/>Cuando utilice una consulta SQL, especifique **nombre de espacio de claves.nombre de tabla** para representar la tabla que quiere consultar. |No (si tableName y el espacio de claves del conjunto de datos están definidos). |
| consistencyLevel |El nivel de coherencia establece el número de réplicas que deben responder a una solicitud de lectura antes de que se devuelvan datos a la aplicación cliente. Cassandra comprueba el número de réplicas especificado para que los datos satisfagan la solicitud de lectura. |ONE, TWO, THREE, QUORUM, ALL, LOCAL_QUORUM, EACH_QUORUM, LOCAL_ONE. Para más información, consulte [Configuring data consistency](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) (Configuración de la coherencia de datos). |No. El valor predeterminado es ONE. |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "CassandraToAzureBlob",
            "description": "Copy from Cassandra to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "CassandraInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "CassandraSource",
                    "query": "select id, firstname, lastname from mykeyspace.mytable"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Cassandra](data-factory-onprem-cassandra-connector.md#copy-activity-properties).

## <a name="mongodb"></a>MongoDB

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de MongoDB, establezca el **tipo** de servicio vinculado en **OnPremisesMongoDB** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| server |Dirección IP o nombre de host del servidor de MongoDB. |Sí |
| port |Puerto TCP que el servidor de MongoDB utiliza para escuchar las conexiones del cliente. |Valor predeterminado opcional: 27017 |
| authenticationType |Básica o anónima. |Sí |
| username |Cuenta de usuario para tener acceso a MongoDB. |Sí (si se usa la autenticación básica). |
| password |Contraseña del usuario. |Sí (si se usa la autenticación básica). |
| authSource |Nombre de la base de datos de MongoDB que desea usar para comprobar las credenciales de autenticación. |Opcional (si se usa la autenticación básica). Valor predeterminado: se utiliza la cuenta de administrador y la base de datos especificada mediante la propiedad databaseName. |
| databaseName |Nombre de la base de datos de MongoDB a la que desea acceder. |Sí |
| gatewayName |Nombre de la puerta de enlace que accede al almacén de datos. |Sí |
| encryptedCredential |Credencial cifrada por la puerta de enlace. |Opcional |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "OnPremisesMongoDbLinkedService",
    "properties": {
        "type": "OnPremisesMongoDb",
        "typeProperties": {
            "authenticationType": "<Basic or Anonymous>",
            "server": "< The IP address or host name of the MongoDB server >",
            "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>",
            "username": "<username>",
            "password": "<password>",
            "authSource": "< The database that you want to use to check your credentials for authentication. >",
            "databaseName": "<database name>",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Para obtener más información, vea el artículo [Conector de MongoDB](data-factory-on-premises-mongodb-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de MongoDB, establezca el **tipo** del conjunto de datos en **MongoDbCollection** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| collectionName |Nombre de la colección en la base de datos de MongoDB. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "MongoDbInputDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": "OnPremisesMongoDbLinkedService",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

Para obtener más información, vea el artículo [Conector de MongoDB](data-factory-on-premises-mongodb-connector.md#dataset-properties).

#### <a name="mongodb-source-in-copy-activity"></a>Origen de MongoDB en la actividad de copia
Si va a copiar datos desde MongoDB, establezca el **tipo de origen** de la actividad de copia en **MongoDbSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL-92. Por ejemplo: `select * from MyTable`. |No (si se especifica **collectionName** de **dataset**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopyMongoDBToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "MongoDbSource",
                    "query": "select * from MyTable"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "MongoDbInputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "MongoDBToAzureBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Para obtener más información, vea el artículo [Conector de MongoDB](data-factory-on-premises-mongodb-connector.md#copy-activity-properties).

## <a name="amazon-s3"></a>Amazon S3


### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Amazon S3, establezca el **tipo** de servicio vinculado en **AwsAccessKey** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| accessKeyID |Id. de la clave de acceso secreta. |string |Sí |
| secretAccessKey |La propia clave de acceso secreta. |Cadena secreta cifrada |Sí |

#### <a name="example"></a>Ejemplo
```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Amazon S3](data-factory-amazon-simple-storage-service-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Amazon S3, establezca el **tipo** de conjunto de datos en **AmazonS3** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| bucketName |Nombre del depósito de S3. |String |Sí |
| key |La clave del objeto S3. |String |No |
| prefix |Prefijo de la clave del objeto S3. Se seleccionan objetos cuyas claves comienzan por este prefijo. Se aplica solo cuando la clave está vacía. |String |No |
| version |La versión del objeto S3 si está habilitado el control de versiones de S3. |String |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No | |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No | |


> [!NOTE]
> bucketName + key especifica la ubicación del objeto S3, donde bucket es el contenedor raíz de los objetos S3 y key es la ruta de acceso completa al objeto S3.

#### <a name="example-sample-dataset-with-prefix"></a>Ejemplo: Conjunto de datos de ejemplo con prefijo

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "prefix": "testFolder/test",
            "bucketName": "<S3 bucket name>",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
#### <a name="example-sample-data-set-with-version"></a>Ejemplo: Conjunto de datos de ejemplo (con versión)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "key": "testFolder/test.orc",
            "bucketName": "<S3 bucket name>",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

#### <a name="example-dynamic-paths-for-s3"></a>Ejemplo: Rutas de acceso dinámicas para S3
En el ejemplo, usamos valores fijos para las propiedades key y bucketName en el conjunto de datos de Amazon S3.

```json
"key": "testFolder/test.orc",
"bucketName": "<S3 bucket name>",
```

Puede hacer que Data Factory calcule estas propiedades dinámicamente en tiempo de ejecución mediante variables del sistema como SliceStart.

```json
"key": "$$Text.Format('{0:MM}/{0:dd}/test.orc', SliceStart)"
"bucketName": "$$Text.Format('{0:yyyy}', SliceStart)"
```

Y lo mismo puede hacer para la propiedad prefix de un conjunto de datos de Amazon S3. Consulte [Azure Data Factory: funciones y variables del sistema](data-factory-functions-variables.md) para ver una lista de funciones y variables admitidas.

Para más información, consulte el artículo [Conector de Amazon S3](data-factory-amazon-simple-storage-service-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Origen de Sistema de archivos en la actividad de copia
Si va a copiar datos desde Amazon S3, establezca el **tipo de origen** de la actividad de copia en **FileSystemSource** y especifique las siguientes propiedades en la sección **origen**:


| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Especifica si se mostrarán objetos S3 de manera recursiva en el directorio. |true/false |No |


#### <a name="example"></a>Ejemplo


```json
{
    "name": "CopyAmazonS3ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource",
                    "recursive": true
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "AmazonS3InputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "AmazonS3ToBlob"
        }],
        "start": "2016-08-08T18:00:00",
        "end": "2016-08-08T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de Amazon S3](data-factory-amazon-simple-storage-service-connector.md#copy-activity-properties).

## <a name="file-system"></a>Sistema de archivos


### <a name="linked-service"></a>Servicio vinculado
Un sistema de archivos local se puede vincular a una factoría de datos de Azure con el servicio vinculado del **servidor de archivos local**. En la tabla siguiente se proporciona la descripción de los elementos JSON específicos para el servicio vinculado del servidor de archivos local.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |Asegúrese de que la propiedad type esté establecida en **OnPremisesFileServer**. |Sí |
| host |Especifica la ruta de acceso raíz de la carpeta que quiere copiar. Use el carácter de escape "\" para los caracteres especiales de la cadena. Consulte los casos que se exponen en Ejemplos de definiciones de servicio vinculado y conjunto de datos. |Sí |
| userid |Especifique el identificador del usuario que tiene acceso al servidor. |No (si elige encryptedCredential) |
| password |Especifique la contraseña del usuario (identificador de usuario). |No (si elige encryptedCredential) |
| encryptedCredential |Especifique las credenciales cifradas que puede obtener mediante la ejecución del cmdlet New-AzDataFactoryEncryptValue. |No (si opta por especificar el identificador de usuario y la contraseña en texto sin formato) |
| gatewayName |Especifica el nombre de la puerta de enlace que debe usar Data Factory para conectarse al servidor de archivos local. |Sí |

#### <a name="sample-folder-path-definitions"></a>Definiciones de ruta de acceso a la carpeta de ejemplo

| Escenario | Host en definición de servicio vinculado | folderPath en definición de conjunto de datos |
| --- | --- | --- |
| Carpeta local en la máquina de Data Management Gateway:: <br/><br/>Ejemplos: D:\\\* o D:\folder\subfolder\\\* |D:\\\\ (para Data Management Gateway 2.0 y versiones posteriores) <br/><br/> localhost (para versiones anteriores a Data Management Gateway 2.0) |\\\\ o la carpeta\\\\subcarpeta (Data Management Gateway 2.0 y versiones posteriores) <br/><br/>D:\\\\ o D:\\\\carpeta\\\\subcarpeta (para versiones de la puerta de enlace interiores a 2.0) |
| Carpeta compartida remota: <br/><br/>Ejemplos: \\\\myserver\\share\\\* o \\\\myserver\\share\\folder\\subfolder\\\* |\\\\\\\\myserver\\\\ |.\\\\ o carpeta\\\\subcarpeta |


#### <a name="example-using-username-and-password-in-plain-text"></a>Ejemplo: Uso de nombre de usuario y contraseña en texto sin formato

```json
{
    "Name": "OnPremisesFileServerLinkedService",
    "properties": {
        "type": "OnPremisesFileServer",
        "typeProperties": {
            "host": "\\\\Contosogame-Asia",
            "userid": "Admin",
            "password": "123456",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-encryptedcredential"></a>Ejemplo: Uso de encryptedcredential

```json
{
    "Name": " OnPremisesFileServerLinkedService ",
    "properties": {
        "type": "OnPremisesFileServer",
        "typeProperties": {
            "host": "D:\\",
            "encryptedCredential": "WFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5xxxxxxxxxxxxxxxxx",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Para más información, consulte el artículo [Conector del sistema de archivos](data-factory-onprem-file-system-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos del sistema de archivos, establezca el **tipo** de conjunto de datos en **FileShare** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Especifica la subruta de acceso a la carpeta. Use el carácter de escape "\" para los caracteres especiales de la cadena. Consulte los casos que se exponen en Ejemplos de definiciones de servicio vinculado y conjunto de datos.<br/><br/>Puede combinar esta propiedad con **partitionBy** para que las rutas de acceso de carpeta se basen en las fechas y horas de inicio y finalización del segmento. |Sí |
| fileName |Especifique el nombre del archivo en **folderPath** si quiere que la tabla haga referencia a un archivo específico de la carpeta. Si no especifica ningún valor para esta propiedad, la tabla apunta a todos los archivos de la carpeta.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado está en el siguiente formato: <br/><br/>`Data.<Guid>.txt` (Ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |No |
| fileFilter |Especifique el filtro que se va a usar para seleccionar un subconjunto de archivos de folderPath, en lugar de todos los archivos. <br/><br/>Valores permitidos son: `*` (varios caracteres) y `?` (un único individual).<br/><br/>Ejemplo 1: "fileFilter": "*.log"<br/>Ejemplo 2: "fileFilter": 2016-1-?.txt"<br/><br/>Tenga en cuenta que fileFilter es aplicable a un conjunto de datos de FileShare de entrada. |No |
| partitionedBy |Puede usar partitionedBy para especificar un valor dinámico de folderPath/fileName para los datos de series temporales. Por ejemplo, folderPath se parametriza por cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**; y los niveles admitidos: **Optimal** y **Fastest**. consulte [Formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

> [!NOTE]
> No se puede usar fileName y fileFilter a la vez.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "OnpremisesFileSystemInput",
    "properties": {
        "type": " FileShare",
        "linkedServiceName": " OnPremisesFileServerLinkedService ",
        "typeProperties": {
            "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
            "fileName": "{Hour}.csv",
            "partitionedBy": [{
                "name": "Year",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                        "format": "yyyy"
                }
            }, {
                "name": "Month",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "MM"
                }
            }, {
                "name": "Day",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "dd"
                }
            }, {
                "name": "Hour",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "HH"
                }
            }]
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector del sistema de archivos](data-factory-onprem-file-system-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Origen de Sistema de archivos en la actividad de copia
Si va a copiar datos desde el sistema de archivos, establezca el **tipo de origen** de la actividad de copia en **FileSystemSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True, False (predeterminada) |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2015-06-01T18:00:00",
        "end": "2015-06-01T19:00:00",
        "description": "Pipeline for copy activity",
        "activities": [{
            "name": "OnpremisesFileSystemtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "OnpremisesFileSystemInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
            "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
Para más información, consulte el artículo [Conector del sistema de archivos](data-factory-onprem-file-system-connector.md#copy-activity-properties).

### <a name="file-system-sink-in-copy-activity"></a>Receptor de Sistema de archivos en la actividad de copia
Si va a copiar datos al sistema de archivos, establezca el **tipo de receptor** de la actividad de copia en **FileSystemSink** y especifique las siguientes propiedades en la sección **receptor**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| copyBehavior |Define el comportamiento de copia cuando el origen es BlobSource o FileSystem. |**PreserveHierarchy:** conserva la jerarquía de archivos en la carpeta de destino. Es decir, la ruta de acceso relativa del archivo de origen a la carpeta de origen es la misma que la ruta de acceso relativa del archivo de destino a la carpeta de destino.<br/><br/>**FlattenHierarchy:** todos los archivos de la carpeta de origen se crean en el primer nivel de la carpeta de destino. Los archivos de destino se crean con un nombre generado automáticamente.<br/><br/>**MergeFiles:** combina todos los archivos de la carpeta de origen en un archivo. Si se especifica el nombre o el nombre del blob, el nombre de archivo combinado es el nombre especificado. De lo contrario, es un nombre de archivo generado automáticamente. |No |

auto-

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2015-06-01T18:00:00",
        "end": "2015-06-01T20:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoOnPremisesFile",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "OnpremisesFileSystemOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "FileSystemSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 3,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector del sistema de archivos](data-factory-onprem-file-system-connector.md#copy-activity-properties).

## <a name="ftp"></a>FTP

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de FTP, establezca el **tipo** de servicio vinculado en **FtpServer** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio | Valor predeterminado |
| --- | --- | --- | --- |
| host |Nombre o dirección IP del servidor FTP |Sí |&nbsp; |
| authenticationType |Especificar el tipo de autenticación |Sí |Basic, Anonymous |
| username |Usuario que tiene acceso al servidor FTP |No |&nbsp; |
| password |Contraseña para el usuario (nombre de usuario) |No |&nbsp; |
| encryptedCredential |Credenciales cifradas para acceder al servidor FTP |No |&nbsp; |
| gatewayName |Nombre de la puerta de enlace de administración de datos para conectarse a un servidor FTP local |No |&nbsp; |
| port |Puerto en el que escucha el servidor FTP |No |21 |
| enableSsl |Especificar si desea usar FTP a través del canal SSL/TLS |No |true |
| enableServerCertificateValidation |Especificar si quiere habilitar la validación de certificados de servidor TLS/SSL al usar FTP sobre el canal SSL/TLS |No |true |

#### <a name="example-using-anonymous-authentication"></a>Ejemplo: Uso de autenticación anónima

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
            "typeProperties": {
            "authenticationType": "Anonymous",
            "host": "myftpserver.com"
        }
    }
}
```

#### <a name="example-using-username-and-password-in-plain-text-for-basic-authentication"></a>Ejemplo: Uso de nombre de usuario y contraseña en texto sin formato para la autenticación básica

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```

#### <a name="example-using-port-enablessl-enableservercertificatevalidation"></a>Ejemplo: Uso de puerto, enableSsl, enableServerCertificateValidation

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456",
            "port": "21",
            "enableSsl": true,
            "enableServerCertificateValidation": true
        }
    }
}
```

#### <a name="example-using-encryptedcredential-for-authentication-and-gateway"></a>Ejemplo: Uso de encryptedCredential para la autenticación y la puerta de enlace

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "gatewayName": "<onpremgateway>"
        }
      }
}
```

Para más información, consulte el artículo [Conector de FTP](data-factory-ftp-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de FTP, establezca el **tipo** de conjunto de datos en **FileShare** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Subruta de acceso a la carpeta. Use el carácter de escape "\" para los caracteres especiales de la cadena. Consulte los casos que se exponen en Ejemplos de definiciones de servicio vinculado y conjunto de datos.<br/><br/>Puede combinar esta propiedad con **partitionBy** para que las rutas de acceso de carpeta se basen en las fechas y horas de inicio y finalización del segmento. |Sí
| fileName |Especifique el nombre del archivo en **folderPath** si quiere que la tabla haga referencia a un archivo específico de la carpeta. Si no especifica ningún valor para esta propiedad, la tabla apunta a todos los archivos de la carpeta.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado estaría en el siguiente formato: <br/><br/>`Data.<Guid>.txt` (Ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |No |
| fileFilter |Especifique el filtro que se va a usar para seleccionar un subconjunto de archivos de folderPath, en lugar de todos los archivos.<br/><br/>Valores permitidos son: `*` (varios caracteres) y `?` (un único individual).<br/><br/>Ejemplos 1: `"fileFilter": "*.log"`<br/>Ejemplo 2: `"fileFilter": 2016-1-?.txt"`<br/><br/> fileFilter es aplicable a un conjunto de datos FileShare de entrada. Esta propiedad no es compatible con HDFS. |No |
| partitionedBy |partitionedBy se puede usar para especificar un valor de folderPath dinámico, un nombre de archivo para datos de series temporales. Por ejemplo, folderPath se parametriza por cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**; y los niveles admitidos: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |
| useBinaryTransfer |Especifica si se usará el modo de transferencia binario. Su valor es true para el modo binario y false para ASCII. Valor predeterminado: verdadero. Esta propiedad solo puede usarse cuando el tipo de servicio vinculado asociado es: FtpServer. |No |

> [!NOTE]
> filename y fileFilter no pueden usarse simultáneamente.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "FTPFileInput",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "FTPLinkedService",
        "typeProperties": {
            "folderPath": "<path to shared folder>",
            "fileName": "test.csv",
            "useBinaryTransfer": true
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Para más información, consulte el artículo [Conector de FTP](data-factory-ftp-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Origen de Sistema de archivos en la actividad de copia
Si va a copiar datos desde un servidor FTP, establezca el **tipo de origen** de la actividad de copia en **FileSystemSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True, False (predeterminada) |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "FTPToBlobCopy",
            "inputs": [{
                "name": "FtpFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-08-24T18:00:00",
        "end": "2016-08-24T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de FTP](data-factory-ftp-connector.md#copy-activity-properties).


## <a name="hdfs"></a>HDFS

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de HDFS, establezca el **tipo** de servicio vinculado en **Hdfs** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type debe establecerse en: **Hdfs** |Sí |
| Url |Dirección URL a HDFS |Sí |
| authenticationType |Anónima o Windows. <br><br> Para usar la **autenticación Kerberos** para el conector HDFS, consulte esta sección a fin de configurar el entorno local en consecuencia. |Sí |
| userName |Nombre de usuario para la autenticación de Windows |Sí (para la autenticación de Windows) |
| password |Contraseña para la autenticación de Windows |Sí (para la autenticación de Windows) |
| gatewayName |Nombre de la puerta de enlace que el servicio Factoría de datos debe usar para conectarse a HDFS. |Sí |
| encryptedCredential |Salida de [New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue) de la credencial de acceso. |No |

#### <a name="example-using-anonymous-authentication"></a>Ejemplo: Uso de autenticación anónima

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "userName": "hadoop",
            "url": "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-windows-authentication"></a>Ejemplo: Con la autenticación de Windows

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "authenticationType": "Windows",
            "userName": "Administrator",
            "password": "password",
            "url": "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Para más información, consulte el artículo Conector de HDFS.

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de HDFS, establezca el **tipo** de conjunto de datos en **FileShare** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Ruta de acceso a la carpeta. Ejemplo: `myfolder`<br/><br/>Use el carácter de escape "\" para los caracteres especiales de la cadena. Por ejemplo: para folder\subfolder, especifique la carpeta\\\\subcarpeta y para d:\samplefolder, especifique d:\\\\samplefolder.<br/><br/>Puede combinar esta propiedad con **partitionBy** para que las rutas de acceso de carpeta se basen en las fechas y horas de inicio y finalización del segmento. |Sí |
| fileName |Especifique el nombre del archivo en **folderPath** si quiere que la tabla haga referencia a un archivo específico de la carpeta. Si no especifica ningún valor para esta propiedad, la tabla apunta a todos los archivos de la carpeta.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado estaría en el siguiente formato: <br/><br/>`Data.<Guid>.txt` (por ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |No |
| partitionedBy |partitionedBy se puede usar para especificar un valor de folderPath dinámico, un nombre de archivo para datos de series temporales. Por ejemplo, folderPath se parametriza para cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

> [!NOTE]
> filename y fileFilter no pueden usarse simultáneamente.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "InputDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "HDFSLinkedService",
        "typeProperties": {
            "folderPath": "DataTransfer/UnitTest/"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Para más información, consulte el artículo Conector de HDFS.

### <a name="file-system-source-in-copy-activity"></a>Origen de Sistema de archivos en la actividad de copia
Si va a copiar datos desde HDFS, establezca el **tipo de origen** de la actividad de copia en **FileSystemSource** y especifique las siguientes propiedades en la sección **origen**:

**FileSystemSource** admite las siguientes propiedades:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True, False (predeterminada) |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "HdfsToBlobCopy",
            "inputs": [{
                "name": "InputDataset"
            }],
            "outputs": [{
                "name": "OutputDataset"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Para más información, consulte el artículo Conector de HDFS.

## <a name="sftp"></a>SFTP


### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de SFTP, establezca el **tipo** de servicio vinculado en **Sftp** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| host | Nombre o dirección IP del servidor SFTP. |Sí |
| port |Puerto en el que escucha el servidor SFTP. El valor predeterminado es: 21 |No |
| authenticationType |Especifique el tipo de autenticación. Valores permitidos: **Basic**, **SshPublicKey**. <br><br> Consulte las secciones Uso de autenticación básica y [Uso de autenticación de clave pública SSH](#using-ssh-public-key-authentication) sobre más propiedades y ejemplos JSON respectivamente. |Sí |
| skipHostKeyValidation | Especifique si desea omitir la validación de claves de host. | No. El valor predeterminado es false. |
| hostKeyFingerprint | Especifique la huella dactilar de la clave de host. | Sí, si `skipHostKeyValidation` está establecido en false.  |
| gatewayName |Nombre de la puerta de enlace de administración de datos para conectarse a un servidor SFTP local. | Sí, si va a copiar datos desde un servidor SFTP local. |
| encryptedCredential | Credenciales cifradas para acceder al servidor SFTP. Se generan automáticamente cuando se especifica la autenticación básica (nombre de usuario + contraseña) o la autenticación SshPublicKey (nombre de usuario + ruta de acceso o contenido de la clave privada) en el Asistente para copiar o el cuadro de diálogo emergente ClickOnce. | No. Se aplica solo cuando se copian datos desde un servidor SFTP local. |

#### <a name="example-using-basic-authentication"></a>Ejemplo: Uso de autenticación básica

Para usar la autenticación básica, establezca `authenticationType` como `Basic`, y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| username | Usuario que tiene acceso al servidor SFTP. |Sí |
| password | Contraseña para el usuario (nombre de usuario). | Sí |

```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<SFTP server name or IP address>",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "password": "xxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-basic-authentication-with-encrypted-credential"></a>Ejemplo: **Autenticación básica con credenciales cifradas**

```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<FTP server name or IP address>",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="using-ssh-public-key-authentication"></a>**Uso de la autenticación de clave pública SSH:**

Para usar la autenticación básica, establezca `authenticationType` como `SshPublicKey`, y especifique las siguientes propiedades además de las genéricas del conector SFTP descritas en la última sección:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| username |Usuario que tiene acceso al servidor SFTP. |Sí |
| privateKeyPath | Especifique una ruta de acceso absoluta al archivo de clave privada al que pueda acceder la puerta de enlace. | Especifique `privateKeyPath` o `privateKeyContent`. <br><br> Se aplica solo cuando se copian datos desde un servidor SFTP local. |
| privateKeyContent | Una cadena serializada del contenido de la clave privada. El Asistente para copiar puede leer el archivo de clave privada y extraer automáticamente el contenido de la clave privada. Si va a usar cualquier otra herramienta o SDK, utilice en su lugar la propiedad privateKeyPath. | Especifique `privateKeyPath` o `privateKeyContent`. |
| passPhrase | Especifique la contraseña o la frase de contraseña para descifrar la clave privada si el archivo de clave está protegido por una frase de contraseña. | Sí, si el archivo de clave privada está protegido por una frase de contraseña. |

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyPath",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<FTP server name or IP address>",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true,
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-sshpublickey-authentication-using-private-key-content"></a>Ejemplo: **autenticación de SshPublicKey mediante contenido de clave privada**

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyContent",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver.westus.cloudapp.azure.com",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyContent": "<base64 string of the private key content>",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true
        }
    }
}
```

Para más información, consulte el artículo [Conector de SFTP](data-factory-sftp-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de SFTP, establezca el **tipo** de conjunto de datos en **FileShare** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| folderPath |Subruta de acceso a la carpeta. Use el carácter de escape "\" para los caracteres especiales de la cadena. Consulte los casos que se exponen en Ejemplos de definiciones de servicio vinculado y conjunto de datos.<br/><br/>Puede combinar esta propiedad con **partitionBy** para que las rutas de acceso de carpeta se basen en las fechas y horas de inicio y finalización del segmento. |Sí |
| fileName |Especifique el nombre del archivo en **folderPath** si quiere que la tabla haga referencia a un archivo específico de la carpeta. Si no especifica ningún valor para esta propiedad, la tabla apunta a todos los archivos de la carpeta.<br/><br/>Si no se especifica fileName para un conjunto de datos de salida, el nombre del archivo generado estaría en el siguiente formato: <br/><br/>`Data.<Guid>.txt` (Ejemplo: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |No |
| fileFilter |Especifique el filtro que se va a usar para seleccionar un subconjunto de archivos de folderPath, en lugar de todos los archivos.<br/><br/>Valores permitidos son: `*` (varios caracteres) y `?` (un único individual).<br/><br/>Ejemplos 1: `"fileFilter": "*.log"`<br/>Ejemplo 2: `"fileFilter": 2016-1-?.txt"`<br/><br/> fileFilter es aplicable a un conjunto de datos FileShare de entrada. Esta propiedad no es compatible con HDFS. |No |
| partitionedBy |partitionedBy se puede usar para especificar un valor de folderPath dinámico, un nombre de archivo para datos de series temporales. Por ejemplo, folderPath se parametriza por cada hora de datos. |No |
| format | Se admiten los tipos de formato siguientes: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Establezca la propiedad **type** de formato en uno de los siguientes valores. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si desea **copiar los archivos tal cual** entre los almacenes basados en archivos (copia binaria), omita la sección de formato en las definiciones de los conjuntos de datos de entrada y salida. |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |
| useBinaryTransfer |Especifica si se usará el modo de transferencia binario. Su valor es true para el modo binario y false para ASCII. Valor predeterminado: verdadero. Esta propiedad solo puede usarse cuando el tipo de servicio vinculado asociado es: FtpServer. |No |

> [!NOTE]
> filename y fileFilter no pueden usarse simultáneamente.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SFTPFileInput",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "SftpLinkedService",
        "typeProperties": {
            "folderPath": "<path to shared folder>",
            "fileName": "test.csv"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Para más información, consulte el artículo [Conector de SFTP](data-factory-sftp-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Origen de Sistema de archivos en la actividad de copia
Si va a copiar datos desde un origen SFTP, establezca el **tipo de origen** de la actividad de copia en **FileSystemSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| recursive |Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. |True, False (predeterminada) |No |



#### <a name="example"></a>Ejemplo

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "SFTPToBlobCopy",
            "inputs": [{
                "name": "SFTPFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2017-02-20T18:00:00",
        "end": "2017-02-20T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de SFTP](data-factory-sftp-connector.md#copy-activity-properties).


## <a name="http"></a>HTTP

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de HTTP, establezca el **tipo** de servicio vinculado en **Http** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| url | Dirección URL base para el servidor web | Sí |
| authenticationType | Especifica el tipo de autenticación. Los valores permitidos son: **Anónima**, **Básica**, **Implícita**, **Windows** o **ClientCertificate**. <br><br> Consulte en las secciones después de esta tabla más propiedades y ejemplos de JSON para esos tipos de autenticación respectivamente. | Sí |
| enableServerCertificateValidation | Especificar si quiere habilitar la validación de certificados TLS/SSL de servidor si el origen es un servidor web HTTPS | No, el valor predeterminado es True. |
| gatewayName | Nombre de la instancia de Data Management Gateway para conectarse a un origen HTTP local. | Sí si va a copiar datos desde un origen HTTP local. |
| encryptedCredential | Credenciales cifradas para acceder al punto de conexión HTTP. Generadas automáticamente cuando se configura la información de autenticación en el Asistente para copia o en el cuadro de diálogo emergente de ClickOnce. | No. Se aplica solo cuando se copian datos desde un servidor HTTP local. |

#### <a name="example-using-basic-digest-or-windows-authentication"></a>Ejemplo: Uso de la autenticación Basic, Digest o Windows
Establezca `authenticationType` como `Basic`, `Digest` o `Windows`, y especifique las siguientes propiedades además de las genéricas del conector HTTP descritas antes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| username | Nombre de usuario para acceder al punto de conexión HTTP. | Sí |
| password | Contraseña para el usuario (nombre de usuario). | Sí |

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "basic",
            "url": "https://en.wikipedia.org/wiki/",
            "userName": "user name",
            "password": "password"
        }
    }
}
```

#### <a name="example-using-clientcertificate-authentication"></a>Ejemplo: Uso de la autenticación ClientCertificate

Para usar la autenticación básica, establezca `authenticationType` como `ClientCertificate`, y especifique las siguientes propiedades además de las genéricas del conector HTTP descritas antes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| embeddedCertData | El contenido con codificación Base64 de datos binarios del archivo de intercambio de información personal (PFX). | Especifique `embeddedCertData` o `certThumbprint`. |
| certThumbprint | La huella digital del certificado que se instaló en el almacén de certificados de la máquina de la puerta de enlace. Se aplica solo cuando se copian datos desde un origen HTTP local. | Especifique `embeddedCertData` o `certThumbprint`. |
| password | Contraseña asociada con el certificado. | No |

Si utiliza `certThumbprint` para la autenticación y el certificado está instalado en el almacén personal del equipo local, necesita conceder el permiso de lectura al servicio de puerta de enlace:

1. Inicie Microsoft Management Console (MMC). Agregue el complemento **Certificados** que tenga como destino **Equipo local**.
2. Expanda **Certificados**, **Personal** y haga clic en **Certificados**.
3. Haga clic con el botón derecho en el certificado del almacén personal y seleccione **Todas las tareas**->**Administrar claves privadas**.
3. En la pestaña **Seguridad**, agregue la cuenta de usuario en que se ejecuta el servicio de host de Data Management Gateway con acceso de lectura al certificado.

**Ejemplo: uso del certificado de cliente:** Este servicio vinculado vincula su factoría de datos a un servidor de web HTTP local. Utiliza un certificado de cliente que está instalado en la máquina con Data Management Gateway instalado.

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "certThumbprint": "thumbprint of certificate",
            "gatewayName": "gateway name"
        }
    }
}
```

#### <a name="example-using-client-certificate-in-a-file"></a>Ejemplo: Uso del certificado de cliente en un archivo
Este servicio vinculado vincula su factoría de datos a un servidor de web HTTP local. Utiliza un archivo de certificado de cliente en la máquina con Data Management Gateway instalado.

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "embeddedCertData": "base64 encoded cert data",
            "password": "password of cert"
        }
    }
}
```

Para más información, consulte el artículo [Conector de HTTP](data-factory-http-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de HTTP, establezca el **tipo** de conjunto de datos en **Http** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| relativeUrl | Dirección URL relativa al recurso que contiene los datos. Cuando no se especifica la ruta de acceso, se solo se usa la dirección URL especificada en la definición de servicio vinculado. <br><br> Para construir la dirección URL dinámica, puede usar [funciones de Data Factory y variables del sistema](data-factory-functions-variables.md); por ejemplo: `"relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)"`. | No |
| requestMethod | Método HTTP. Los valores permitidos son **GET** o **POST**. | No. El valor predeterminado es `GET`. |
| additionalHeaders | Encabezados de solicitud HTTP adicionales. | No |
| requestBody | Cuerpo de la solicitud HTTP. | No |
| format | Si desea simplemente **recuperar los datos del punto de conexión HTTP tal cual** sin analizarlos, omita esta configuración de formato. <br><br> Si desea analizar el contenido de la respuesta HTTP durante la copia, se admiten los siguientes tipos de formato: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Para más información, consulte las secciones [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format), [Formato Json](data-factory-supported-file-and-compression-formats.md#json-format), [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Formato Orc](data-factory-supported-file-and-compression-formats.md#orc-format) y [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). |No |
| compression | Especifique el tipo y el nivel de compresión de los datos. Estos son los tipos que se admiten: **GZip**, **Deflate**, **BZip2** y **ZipDeflate**. Estos son los niveles que se admiten: **Optimal** y **Fastest**. Para más información, consulte el artículo sobre [formatos de compresión de archivos en Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |No |

#### <a name="example-using-the-get-default-method"></a>Ejemplo: Uso del método GET (predeterminado)

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "XXX/test.xml",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

#### <a name="example-using-the-post-method"></a>Ejemplo: Uso del método POST

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "/XXX/test.xml",
            "requestMethod": "Post",
            "requestBody": "body for POST HTTP request"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```
Para más información, consulte el artículo [Conector de HTTP](data-factory-http-connector.md#dataset-properties).

### <a name="http-source-in-copy-activity"></a>Origen de HTTP en la actividad de copia
Si va a copiar datos desde un origen HTTP, establezca el **tipo de origen** de la actividad de copia en **HttpSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Obligatorio |
| -------- | ----------- | -------- |
| httpRequestTimeout | El tiempo que la solicitud HTTP espera (TimeSpan) a obtener una respuesta. Es el tiempo de espera para obtener una respuesta, no para leer los datos de la respuesta. | No. Valor predeterminado: 00:01:40 |


#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "HttpSourceToAzureBlob",
            "description": "Copy from an HTTP source to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "HttpSourceDataInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "HttpSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de HTTP](data-factory-http-connector.md#copy-activity-properties).

## <a name="odata"></a>OData

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de OData, establezca el **tipo** de servicio vinculado en **OData** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| url |Dirección URL del servicio de OData. |Sí |
| authenticationType |Tipo de autenticación que se usa para conectarse al origen de OData. <br/><br/> Para OData en la nube, los valores posibles son Anonymous, Basic y OAuth (tenga en cuenta que Azure Data Factory en estos momentos solo admite la autenticación OAuth basada en Azure Active Directory). <br/><br/> Para OData local, los valores posibles son: Anonymous, Basic y Windows. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación básica. |Sí (solo si usa la autenticación básica) |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |Sí (solo si usa la autenticación básica) |
| authorizedCredential |Si está usando OAuth, haga clic en el botón **Autorizar** del Editor o el Asistente para copiar de Data Factory y escriba su credencial. Después, el valor de esta propiedad se generará automáticamente. |Sí (solo si usa la autenticación OAuth) |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Factoría de datos para conectarse al servicio de OData local. Especifique solo si va a copiar datos del origen de OData local. |No |

#### <a name="example---using-basic-authentication"></a>Ejemplo: Uso de la autenticación básica
```json
{
    "name": "inputLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Basic",
            "username": "username",
            "password": "password"
        }
    }
}
```

#### <a name="example---using-anonymous-authentication"></a>Ejemplo: Uso de autenticación anónima

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        }
    }
}
```

#### <a name="example---using-windows-authentication-accessing-on-premises-odata-source"></a>Ejemplo: Uso de la autenticación de Windows para acceder al origen de OData local

```json
{
    "name": "inputLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of on-premises OData source, for example, Dynamics CRM>",
            "authenticationType": "Windows",
            "username": "domain\\user",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example---using-oauth-authentication-accessing-cloud-odata-source"></a>Ejemplo: Uso de la autenticación OAuth para tener acceso al origen de OData en la nube
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "<endpoint of cloud OData source, for example, https://<tenant>.crm.dynamics.com/XRMServices/2011/OrganizationData.svc>",
            "authenticationType": "OAuth",
            "authorizedCredential": "<auto generated by clicking the Authorize button on UI>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de OData](data-factory-odata-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de OData , establezca el **tipo** de conjunto de datos en **ODataResource** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| path |Ruta de acceso al recurso de OData |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "ODataDataset",
    "properties": {
        "type": "ODataResource",
        "typeProperties": {
            "path": "Products"
        },
        "linkedServiceName": "ODataLinkedService",
        "structure": [],
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
        }
    }
}
```

Para más información, consulte el artículo [Conector de OData](data-factory-odata-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde un origen de OData, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Ejemplo | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |"?$select=Name, Description&$top=5" |No |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopyODataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "?$select=Name, Description&$top=5"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "ODataDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobODataDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "ODataToBlob"
        }],
        "start": "2017-02-01T18:00:00",
        "end": "2017-02-03T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de OData](data-factory-odata-connector.md#copy-activity-properties).


## <a name="odbc"></a>ODBC


### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de ODBC, establezca el **tipo** de servicio vinculado en **OnPremisesOdbc** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| connectionString |La parte de la credencial de no acceso de la cadena de conexión, así como una credencial cifrada opcional. Vea ejemplos en las secciones siguientes. |Sí |
| credencial |La parte de la credencial de acceso de la cadena de conexión especificada en formato de valor de propiedad específico del controlador. Ejemplo: `“Uid=<user ID>;Pwd=<password>;RefreshToken=<secret refresh token>;”.` |No |
| authenticationType |Tipo de autenticación que se usa para conectarse al almacén de datos ODBC. Los valores posibles son: Anónima y básica. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación básica. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |
| gatewayName |Nombre de la puerta de enlace que el servicio Factoría de datos debe usar para conectarse al almacén de datos ODBC. |Sí |

#### <a name="example---using-basic-authentication"></a>Ejemplo: Uso de la autenticación básica

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```
#### <a name="example---using-basic-authentication-with-encrypted-credentials"></a>Ejemplo: Uso de la autenticación básica con credenciales cifradas
Puede cifrar las credenciales mediante el cmdlet [New AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue).

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=myserver.database.windows.net; Database=TestDatabase;;EncryptedCredential=eyJDb25uZWN0...........................",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-anonymous-authentication"></a>Ejemplo: Uso de autenticación anónima

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "connectionString": "Driver={SQL Server};Server={servername}.database.windows.net; Database=TestDatabase;",
            "credential": "UID={uid};PWD={pwd}",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de ODBC](data-factory-odbc-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de ODBC, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla en el almacén de datos ODBC. |Sí |


#### <a name="example"></a>Ejemplo

```json
{
    "name": "ODBCDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "ODBCLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de ODBC](data-factory-odbc-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde un almacén de datos de ODBC, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: `select * from MyTable`. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "CopyODBCToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "OdbcDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobOdbcDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "OdbcToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Para más información, consulte el artículo [Conector de ODBC](data-factory-odbc-connector.md#copy-activity-properties).

## <a name="salesforce"></a>Salesforce


### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Salesforce, establezca el **tipo** de servicio vinculado en **Salesforce** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| environmentUrl | Especifique la URL de la instancia de Salesforce. <br><br> - La dirección predeterminada es "https:\//login.salesforce.com". <br> - Para copiar datos desde el espacio aislado, especifique "https://test.salesforce.com". <br> - Para copiar datos del dominio personalizado, especifique, por ejemplo, "https://[dominio].my.salesforce.com". |No |
| username |Especifique el nombre de usuario de la cuenta de usuario. |Sí |
| password |Especifique la contraseña para la cuenta de usuario. |Sí |
| securityToken |Especifique el token de seguridad para la cuenta de usuario. Consulte [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Obtención de un token de seguridad) para ver instrucciones sobre cómo restablecer u obtener un token de seguridad. Para más información acerca de los tokens de seguridad en general, consulte [Security and the API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)(Seguridad y la API). |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SalesforceLinkedService",
    "properties": {
        "type": "Salesforce",
        "typeProperties": {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Salesforce](data-factory-salesforce-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Salesforce, establezca el **tipo** de conjunto de datos en **RelationalTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla de Salesforce. |No (si se especifica una **consulta** de **RelationalSource**) |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Para más información, consulte el artículo [Conector de Salesforce](data-factory-salesforce-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Origen relacional en la actividad de copia
Si va a copiar datos desde Salesforce, establezca el **tipo de origen** de la actividad de copia en **RelationalSource** y especifique las siguientes propiedades en la sección **origen**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| Query |Utilice la consulta personalizada para leer los datos. |Consulta de SQL-92 o de [Salesforce Object Query Language (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm). Por ejemplo: `select * from MyTable__c`. |No (si se especifica **tableName** de **dataset**) |

#### <a name="example"></a>Ejemplo



```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "SalesforceInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

> [!IMPORTANT]
> La parte "__c" del nombre de la API es necesaria para cualquier objeto personalizado.

Para más información, consulte el artículo [Conector de Salesforce](data-factory-salesforce-connector.md#copy-activity-properties).

## <a name="web-data"></a>Datos web

### <a name="linked-service"></a>Servicio vinculado
Para definir un servicio vinculado de Web, establezca el **tipo** de servicio vinculado en **Web** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| Url |Dirección URL para el origen de Web |Sí |
| authenticationType |Anonymous. |Sí |


#### <a name="example"></a>Ejemplo


```json
{
    "name": "web",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "url": "https://en.wikipedia.org/wiki/"
        }
    }
}
```

Para más información, consulte el artículo [Conector de Tabla web](data-factory-web-table-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Para definir un conjunto de datos de Web, establezca el **tipo** de conjunto de datos en **WebTable** y especifique las siguientes propiedades en la sección **typeProperties**:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |Tipo del conjunto de datos. Debe establecerse en **WebTable** |Sí |
| path |Dirección URL relativa al recurso que contiene la tabla. |No. Cuando no se especifica la ruta de acceso, se solo se usa la dirección URL especificada en la definición de servicio vinculado. |
| índice |Índice de la tabla en el recurso. Consulte la sección Obtención de índice de una tabla en una página HTML para saber los pasos necesarios para obtener el índice de una tabla en una página HTML. |Sí |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Para más información, consulte el artículo [Conector de Tabla web](data-factory-web-table-connector.md#dataset-properties).

### <a name="web-source-in-copy-activity"></a>Origen de Web en la actividad de copia
Si va a copiar datos de una tabla web, establezca el **tipo de origen** de la actividad de copia en **WebSource**. En este momento, cuando el origen de la actividad de copia es de tipo **WebSource**, no se admite ninguna propiedad adicional.

#### <a name="example"></a>Ejemplo

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "WebTableToAzureBlob",
            "description": "Copy from a Web table to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "WebTableInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "WebSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Para más información, consulte el artículo [Conector de Tabla web](data-factory-web-table-connector.md#copy-activity-properties).

## <a name="compute-environments"></a>ENTORNOS DE PROCESO
En la tabla siguiente se enumeran entornos de proceso compatibles con Data Factory y las actividades de transformación que se pueden ejecutar en ellos. Haga clic en el vínculo del proceso que le interesa para ver los esquemas JSON del servicio vinculado para vincularlo a una instancia de Data Factory.

| Entorno de procesos | Actividades |
| --- | --- |
| [Clúster de HDInsight a petición](#on-demand-azure-hdinsight-cluster) o [clúster HDInsight propio](#existing-azure-hdinsight-cluster) |[Actividad personalizada de .NET](#net-custom-activity), [actividad de Hive](#hdinsight-hive-activity), [actividad de Pig](#hdinsight-pig-activity), [actividad de MapReduce](#hdinsight-mapreduce-activity), actividad de streaming de Hadoop y [actividad de Spark](#hdinsight-spark-activity) |
| [Azure Batch](#azure-batch) |[Actividad personalizada de .NET](#net-custom-activity) |
| [Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic) | [Actividad de ejecución de lotes de Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic-batch-execution-activity), [actividad de actualización de recurso de Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic-update-resource-activity). |
| [Análisis con Azure Data Lake](#azure-data-lake-analytics) |[U-SQL de análisis con Data Lake](#data-lake-analytics-u-sql-activity) |
| [Azure SQL Database](#azure-sql-database), [Azure Synapse Analytics](#azure-synapse-analytics), [SQL Server](#sql-server-stored-procedure) |[Procedimiento almacenado](#stored-procedure-activity) |

## <a name="on-demand-azure-hdinsight-cluster"></a>Clúster de Azure HDInsight a petición
El servicio Azure Data Factory crea automáticamente un clúster de HDInsight basado en Windows/Linux a petición para procesar los datos. El clúster se crea en la misma región que la cuenta de almacenamiento (propiedad linkedServiceName en JSON) asociada al clúster. Puede ejecutar las siguientes actividades de transformación en este servicio vinculado: [actividad personalizada de .NET](#net-custom-activity), [actividad de Hive](#hdinsight-hive-activity), [actividad de Pig](#hdinsight-pig-activity), [actividad de MapReduce](#hdinsight-mapreduce-activity), actividad de streaming de Hadoop y [actividad de Spark](#hdinsight-spark-activity).

### <a name="linked-service"></a>Servicio vinculado
En la siguiente tabla se ofrecen descripciones de las propiedades que se usan en la definición de JSON en Azure de un servicio vinculado de HDInsight a petición.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type se debe establecer en **HDInsightOnDemand**. |Sí |
| clusterSize |Número de nodos de datos o trabajo del clúster El clúster de HDInsight se crea con dos nodos principales junto con el número de nodos de trabajo que haya especificado para esta propiedad. Los nodos son de tamaño Standard_D3 con 4 núcleos, por lo que un clúster de nodos de 4 trabajos necesitará 24 núcleos (4\*4 = 16 para nodos de trabajo, más 2\*4 = 8 para nodos principales). Consulte [Creación de clústeres de Hadoop basados en Linux en HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md) para más información acerca del nivel Standard_D3. |Sí |
| timeToLive |El tiempo de inactividad permitido para el clúster de HDInsight a petición. Especifica cuánto tiempo permanece activo el clúster de HDInsight a petición después de la finalización de una ejecución de actividad si no hay ningún otro trabajo activo en el clúster.<br/><br/>Por ejemplo, si una ejecución de actividad tarda 6 minutos y timetolive está establecido en 5 minutos, el clúster permanece activo durante 5 minutos después de los 6 minutos de procesamiento de la ejecución de actividad. Si se realiza otra ejecución de actividad con un margen de 6 minutos, la procesa el mismo clúster.<br/><br/>Crear un clúster de HDInsight a petición es una operación costosa (podría tardar un poco); use esta configuración si es necesario para mejorar el rendimiento de una factoría de datos mediante la reutilización de un clúster de HDInsight a petición.<br/><br/>Si establece el valor de timetolive en 0, el clúster se elimina en cuanto se procesa la ejecución de actividad. Por otra parte, si establece un valor alto, el clúster puede permanecer inactivo innecesariamente, lo que conlleva unos costos elevados. Por lo tanto, es importante que establezca el valor adecuado en función de sus necesidades.<br/><br/>Varias canalizaciones pueden compartir la misma instancia del clúster de HDInsight a petición si el valor de la propiedad timetolive está correctamente configurado. |Sí |
| version |Versión del clúster de HDInsight. Para obtener más información, consulte [Versiones admitidas de HDInsight en Azure Data Factory](data-factory-compute-linked-services.md#supported-hdinsight-versions-in-azure-data-factory). |No |
| linkedServiceName |El servicio vinculado de Azure Storage que usará el clúster a petición para almacenar y procesar datos. <p>Actualmente, no se puede crear un clúster de HDInsight a petición que utilice una instancia de Azure Data Lake Store como almacenamiento. Si desea almacenar los datos de resultados del procesamiento de HDInsight en una instancia de Azure Data Lake Store, utilice una actividad de copia para copiar los datos desde Azure Blob Storage a Azure Data Lake Store.</p>  | Sí |
| additionalLinkedServiceNames |Especifica cuentas de almacenamiento adicionales para el servicio vinculado de HDInsight, de forma que el servicio Factoría de datos pueda registrarlas en su nombre. |No |
| osType |Tipo de sistema operativo. Los valores permitidos son: Windows (valor predeterminado) y Linux |No |
| hcatalogLinkedServiceName |Nombre del servicio vinculado de SQL de Azure que apunta a la base de datos de HCatalog. El clúster de HDInsight a petición se crea usando Azure SQL Database como el almacén de metadatos. |No |

### <a name="json-example"></a>Ejemplo JSON
En el siguiente JSON se define un servicio vinculado de HDInsight a petición basado en Linux. El servicio Data Factory crea automáticamente un clúster de HDInsight **basado en Linux** al procesar un segmento de datos.

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "StorageLinkedService"
        }
    }
}
```

Para más información, vea [Servicios vinculados de procesos](data-factory-compute-linked-services.md).

## <a name="existing-azure-hdinsight-cluster"></a>Clúster de Azure HDInsight existente
Puede crear un servicio vinculado de HDInsight de Azure para registrar su propio clúster de HDInsight con la Factoría de datos. Puede ejecutar las siguientes actividades de transformación de datos en este servicio vinculado: [actividad personalizada de .NET](#net-custom-activity), [actividad de Hive](#hdinsight-hive-activity), [actividad de Pig](#hdinsight-pig-activity), [actividad de MapReduce](#hdinsight-mapreduce-activity), actividad de streaming de Hadoop y [actividad de Spark](#hdinsight-spark-activity).

### <a name="linked-service"></a>Servicio vinculado
En la siguiente tabla se ofrecen descripciones de las propiedades que se usan en la definición de JSON en Azure de un servicio vinculado de Azure HDInsight.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type se debe establecer en **HDInsight**. |Sí |
| clusterUri |El URI del clúster de HDInsight. |Sí |
| username |Especifique el nombre de usuario que se usará para conectarse a un clúster de HDInsight existente. |Sí |
| password |Especifique la contraseña para la cuenta de usuario. |Sí |
| linkedServiceName | Nombre del servicio vinculado para Azure Storage que hace referencia al almacenamiento Azure Blob Storage que usa el clúster de HDInsight. <p>Actualmente, no se puede especificar un servicio vinculado de Azure Data Lake Store para esta propiedad. Puede acceder a los datos de Azure Data Lake Store desde scripts de Pig/Hive si el clúster de HDInsight tiene acceso a Data Lake Store. </p>  |Sí |

Para conocer las versiones de los clústeres de HDInsight admitidas, consulte [Versiones compatibles con HDInsight](data-factory-compute-linked-services.md#supported-hdinsight-versions-in-azure-data-factory).

#### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "HDInsightLinkedService",
    "properties": {
        "type": "HDInsight",
        "typeProperties": {
            "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
            "userName": "admin",
            "password": "<password>",
            "linkedServiceName": "MyHDInsightStoragelinkedService"
        }
    }
}
```

## <a name="azure-batch"></a>Azure Batch
Puede crear un servicio vinculado de Azure Batch para registrar un grupo de Batch de máquinas virtuales (VM) en una instancia de Data Factory. Puede ejecutar actividades personalizadas .NET con Azure Batch o Azure HDInsight. Puede ejecutar una [actividad personalizada de .NET](#net-custom-activity) en este servicio vinculado.

### <a name="linked-service"></a>Servicio vinculado
En la siguiente tabla se ofrecen descripciones de las propiedades que se usan en la definición de JSON en Azure de un servicio vinculado de Azure Batch.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type se debe establecer en **AzureBatch**. |Sí |
| accountName |Nombre de la cuenta de Azure Batch. |Sí |
| accessKey |Clave de acceso de la cuenta de Azure Batch. |Sí |
| poolName |Nombre del grupo de máquinas virtuales. |Sí |
| linkedServiceName |Nombre del servicio vinculado de Azure Storage asociado a este servicio vinculado de Azure Batch. Este servicio vinculado se usa para los archivos de almacenamiento provisional necesarios para ejecutar la actividad y almacenar los registros de ejecución de la actividad. |Sí |


#### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "AzureBatchLinkedService",
    "properties": {
        "type": "AzureBatch",
        "typeProperties": {
            "accountName": "<Azure Batch account name>",
            "accessKey": "<Azure Batch account key>",
            "poolName": "<Azure Batch pool name>",
            "linkedServiceName": "<Specify associated storage linked service reference here>"
        }
    }
}
```

## <a name="azure-machine-learning-studio-classic"></a>Azure Machine Learning Studio (clásico)
Un servicio vinculado de Azure Machine Learning Studio (clásico) se crea para registrar un punto de conexión de puntuación por lotes de Studio (clásico) con una factoría de datos. Dos actividades de transformación de datos que se pueden ejecutar en este servicio vinculado son: [Actividad de ejecución de lotes de Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic-batch-execution-activity), [actividad de actualización de recurso de Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic-update-resource-activity).

### <a name="linked-service"></a>Servicio vinculado
En la siguiente tabla se ofrecen descripciones de las propiedades que se usan en la definición de JSON en Azure de un servicio vinculado de Studio (clásico).

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| Tipo |La propiedad type se debe establecer en: **AzureML**. |Sí |
| mlEndpoint |La dirección URL de puntuación por lotes. |Sí |
| apiKey |La API del modelo de área de trabajo publicado. |Sí |

#### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://[batch scoring endpoint]/jobs",
            "apiKey": "<apikey>"
        }
    }
}
```

## <a name="azure-data-lake-analytics"></a>Análisis con Azure Data Lake
Cree un servicio vinculado con **Análisis con Azure Data Lake** para vincular un servicio de proceso de Análisis con Azure Data Lake a un generador de datos de Azure antes de usar la [actividad U-SQL de Análisis con Data Lake](data-factory-usql-activity.md) en una canalización.

### <a name="linked-service"></a>Servicio vinculado

En la siguiente tabla se ofrecen descripciones de las propiedades que se usan en la definición de JSON de un servicio vinculado de Azure Data Lake Analytics.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| Tipo |La propiedad type se debe establecer en: **AzureDataLakeAnalytics**. |Sí |
| accountName |Nombre de la cuenta de Análisis de Azure Data Lake |Sí |
| dataLakeAnalyticsUri |Identificador URI de Análisis de Azure Data Lake. |No |
| authorization |El código de autorización se recupera automáticamente después de hacer clic en el botón **Autorizar** situado en el Editor de Factoría de datos y de completar el inicio de sesión de OAuth. |Sí |
| subscriptionId |Identificador de suscripción de Azure |No (si no se especifica, se usa la suscripción de Data Factory). |
| resourceGroupName |Nombre del grupo de recursos de Azure |No (si no se especifica, se usa el grupo de recursos de la factoría de datos). |
| sessionID |Identificador de sesión de la sesión de autorización de OAuth. Cada identificador de sesión es único y solo se puede usar una vez. Si usa Data Factory Editor, este identificador se genera automáticamente. |Sí |


#### <a name="json-example"></a>Ejemplo JSON
En el ejemplo siguiente se proporciona la definición de JSON de un servicio vinculado de Análisis con Azure Data Lake.

```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>",
            "subscriptionId": "<subscription id>",
            "resourceGroupName": "<resource group name>"
        }
    }
}
```

## <a name="sql-server-stored-procedure"></a>Procedimiento almacenado de SQL Server

Cree un servicio vinculado de SQL Server y úselo con la [actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para invocar un procedimiento almacenado desde una canalización de Data Factory.

### <a name="linked-service"></a>Servicio vinculado
Se creará un servicio vinculado de tipo **OnPremisesSqlServer** para vincular una base de datos SQL Server local a una factoría de datos. En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio SQL Server vinculado.

En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio SQL Server vinculado.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type se debe establecer en: **OnPremisesSqlServer**. |Sí |
| connectionString |Especifique la información de connectionString necesaria para conectarse a la base de datos SQL Server mediante autenticación de SQL o autenticación de Windows. |Sí |
| gatewayName |Nombre de la puerta de enlace que debe usar el servicio Data Factory para conectarse a la base de datos de SQL Server. |Sí |
| username |Especifique el nombre de usuario si usa la autenticación de Windows. Ejemplo: **nombreDeDominio\\username**. |No |
| password |Especifique la contraseña de la cuenta de usuario especificada para el nombre de usuario. |No |

Puede cifrar las credenciales con el cmdlet **New-AzDataFactoryEncryptValue** y usarlas en la cadena de conexión, como se muestra en el ejemplo siguiente (propiedad **EncryptedCredential**):

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```


#### <a name="example-json-for-using-sql-authentication"></a>Ejemplo: JSON para usar autenticación de SQL

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
#### <a name="example-json-for-using-windows-authentication"></a>Ejemplo: JSON para usar autenticación de Windows

Si se especifican el nombre de usuario y la contraseña, la puerta de enlace los usa para suplantar la cuenta de usuario especificada para conectarse a la base de datos de SQL Server. En caso contrario, la puerta de enlace se conectará directamente a SQL Server con el contexto de seguridad de puerta de enlace (su cuenta de inicio).

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Para más información, consulte el artículo [Conector de SQL Server](data-factory-sqlserver-connector.md#linked-service-properties).

## <a name="data-transformation-activities"></a>ACTIVIDADES DE TRANSFORMACIÓN DE DATOS

Actividad | Descripción
-------- | -----------
[Actividad de Hive de HDInsight](#hdinsight-hive-activity) | La actividad de Hive de HDInsight en una canalización de Data Factory ejecuta consultas de Hive en su propio clúster de HDInsight o en uno a petición basado en Windows/Linux.
[Actividad de Pig de HDInsight](#hdinsight-pig-activity) | La actividad de Pig de HDInsight en una canalización de Data Factory ejecuta consultas de Pig en su propio clúster de HDInsight o en uno a petición basado en Windows/Linux.
[Actividad de MapReduce de HDInsight](#hdinsight-mapreduce-activity) | La actividad MapReduce de HDInsight en una canalización de Data Factory ejecuta programas de MapReduce en su propio clúster de HDInsight o en uno basado en Windows/Linux a petición.
[Actividad de streaming de HDInsight](#hdinsight-streaming-activity) | La actividad de streaming de HDInsight en una canalización de Data Factory ejecuta programas de streaming de Hadoop en su propio clúster de HDInsight o en uno basado en Windows/Linux a petición.
[Actividad de HDInsight Spark](#hdinsight-spark-activity) | La actividad de Spark de HDInsight en una canalización de Data Factory ejecuta consultas de Spark en su propio clúster de HDInsight.
[Actividad de ejecución de lotes de Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic-batch-execution-activity) | Azure Data Factory permite crear fácilmente canalizaciones que usan un servicio web de Studio (clásico) publicado para realizar análisis predictivos. Mediante la actividad de ejecución de lotes en una canalización de Azure Data Factory, puede invocar un servicio web de Studio (clásico) para realizar predicciones sobre los datos del lote.
[Actividad de actualización de recurso de Azure Machine Learning Studio (clásico)](#azure-machine-learning-studio-classic-update-resource-activity) | Pasado algún tiempo, los modelos predictivos en los experimentos de puntuación de Azure Machine Learning Studio (clásico) tienen que volver a entrenarse con nuevos conjuntos de datos de entrada. Después de terminar con el nuevo entrenamiento, tendrá que actualizar el servicio web de puntuación con el modelo de Machine Learning que volvió a entrenar. Puede usar la Actividad de recursos de actualización para actualizar el servicio web con el modelo recién entrenado.
[Actividad de procedimiento almacenado](#stored-procedure-activity) | Puede usar la actividad de procedimiento almacenado en una canalización de Data Factory para invocar un procedimiento almacenado en uno de los siguientes almacenes de datos: Azure SQL Database, Azure Synapse Analytics y base de datos de SQL Server en una empresa o una máquina virtual de Azure.
[Actividad U-SQL de Data Lake Analytics](#data-lake-analytics-u-sql-activity) | La actividad de U-SQL de Data Lake Analytics ejecuta un script de U-SQL en un clúster de Azure Data Lake Analytics.
[Actividad personalizada de .NET](#net-custom-activity) | Si necesita transformar datos de algún modo no compatible con Data Factory, puede crear una actividad personalizada con su propia lógica de procesamiento de datos y usarla en la canalización. Puede configurar una actividad de .NET personalizada para ejecutarse mediante un servicio Azure Batch o un clúster de Azure HDInsight.


## <a name="hdinsight-hive-activity"></a>Actividad de Hive de HDInsight
Puede especificar las siguientes propiedades en una definición JSON de actividad de Hive. La propiedad type de la actividad debe ser: **HDInsightHive**. Debe crear primero un servicio vinculado de HDInsight y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en HDInsightHive:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| script |Especifica el script de Hive en línea |No |
| scriptPath |Almacena el script de Hive en un almacenamiento de blobs de Azure y proporciona la ruta de acceso al archivo. Use la propiedad 'script' o 'scriptPath'. No se pueden usar las dos juntas. El nombre del archivo distingue mayúsculas de minúsculas. |No |
| defines |Especifique parámetros como pares de clave y valor para referencia en el script de Hive con 'hiveconf' |No |

Estas propiedades de tipo son específicas de la actividad de Hive. Otras propiedades (fuera de la sección typeProperties) son compatibles con todas las actividades.

### <a name="json-example"></a>Ejemplo JSON
El siguiente fragmento JSON define una actividad de Hive de HDInsight en una canalización.

```json
{
    "name": "Hive Activity",
    "description": "description",
    "type": "HDInsightHive",
    "inputs": [
      {
        "name": "input tables"
      }
    ],
    "outputs": [
      {
        "name": "output tables"
      }
    ],
    "linkedServiceName": "MyHDInsightLinkedService",
    "typeProperties": {
      "script": "Hive script",
      "scriptPath": "<pathtotheHivescriptfileinAzureblobstorage>",
      "defines": {
        "param1": "param1Value"
      }
    },
   "scheduler": {
      "frequency": "Day",
      "interval": 1
    }
}
```

Para más información, consulte el artículo [Actividad de Hive](data-factory-hive-activity.md).

## <a name="hdinsight-pig-activity"></a>Actividad de Pig de HDInsight
Puede especificar las siguientes propiedades en una definición JSON de actividad de Pig. La propiedad type de la actividad debe ser: **HDInsightPig**. Debe crear primero un servicio vinculado de HDInsight y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en HDInsightPig:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| script |Especifica el script de Pig en línea |No |
| scriptPath |Almacena el script de Pig en un almacenamiento de blobs de Azure y proporciona la ruta de acceso al archivo. Use la propiedad 'script' o 'scriptPath'. No se pueden usar las dos juntas. El nombre del archivo distingue mayúsculas de minúsculas. |No |
| defines |Especifique parámetros como pares de clave y valor para referencia en el script de Pig |No |

Estas propiedades de tipo son específicas de la actividad de Pig. Otras propiedades (fuera de la sección typeProperties) son compatibles con todas las actividades.

### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "HiveActivitySamplePipeline",
      "properties": {
    "activities": [
        {
            "name": "Pig Activity",
            "description": "description",
            "type": "HDInsightPig",
            "inputs": [
                  {
                    "name": "input tables"
                  }
            ],
            "outputs": [
                  {
                    "name": "output tables"
                  }
            ],
            "linkedServiceName": "MyHDInsightLinkedService",
            "typeProperties": {
                  "script": "Pig script",
                  "scriptPath": "<pathtothePigscriptfileinAzureblobstorage>",
                  "defines": {
                    "param1": "param1Value"
                  }
            },
               "scheduler": {
                  "frequency": "Day",
                  "interval": 1
            }
          }
    ]
  }
}
```

Para más información, consulte el artículo Actividad de Pig.

## <a name="hdinsight-mapreduce-activity"></a>Actividad de MapReduce de HDInsight
Puede especificar las siguientes propiedades en una definición JSON de actividad de MapReduce. La propiedad type de la actividad debe ser: **HDInsightMapReduce**. Debe crear primero un servicio vinculado de HDInsight y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en HDInsightMapReduce:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| jarLinkedService | Nombre del servicio vinculado de Azure Storage que contiene el archivo JAR. | Sí |
| jarFilePath | Ruta de acceso al archivo JAR en Azure Storage. | Sí |
| className | Nombre de la clase principal en el archivo JAR. | Sí |
| argumentos | Una lista de argumentos separados por comas para el programa de MapReduce. En tiempo de ejecución, verá unos argumentos adicionales (por ejemplo, mapreduce.job.tags) desde el marco de trabajo MapReduce. Para diferenciar sus argumentos con los argumentos de MapReduce, considere el uso tanto de opción como de valor como argumentos, tal como se muestra en el siguiente ejemplo (-s, --input, --output, etc., son opciones seguidas inmediatamente por sus valores). | No |

### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "MahoutMapReduceSamplePipeline",
    "properties": {
        "description": "Sample Pipeline to Run a Mahout Custom Map Reduce Jar. This job calculates an Item Similarity Matrix to determine the similarity between two items",
        "activities": [
            {
                "type": "HDInsightMapReduce",
                "typeProperties": {
                    "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
                    "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
                    "jarLinkedService": "StorageLinkedService",
                    "arguments": ["-s", "SIMILARITY_LOGLIKELIHOOD", "--input", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/input", "--output", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/output/", "--maxSimilaritiesPerItem", "500", "--tempDir", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/temp/mahout"]
                },
                "inputs": [
                    {
                        "name": "MahoutInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "MahoutOutput"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "MahoutActivity",
                "description": "Custom Map Reduce to generate Mahout result",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-01-03T00:00:00",
        "end": "2017-01-04T00:00:00"
    }
}
```

Para más información, consulte el artículo [Actividad de MapReduce](data-factory-map-reduce.md).

## <a name="hdinsight-streaming-activity"></a>Actividad de streaming de HDInsight
Puede especificar las siguientes propiedades en una definición JSON de actividad de streaming de Hadoop. La propiedad type de la actividad debe ser: **HDInsightStreaming**. Debe crear primero un servicio vinculado de HDInsight y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en HDInsightStreaming:

| Propiedad | Descripción |
| --- | --- |
| mapper | Nombre del archivo ejecutable del asignador. En el ejemplo, cat.exe es el ejecutable del asignador.|
| reducer | Nombre del archivo ejecutable del reductor. En el ejemplo, wc.exe es el ejecutable del reductor. |
| input | Archivo de entrada (incluida la ubicación) del asignador. En el ejemplo: `"wasb://adfsample@<account name>.blob.core.windows.net/example/data/gutenberg/davinci.txt"`: adfsample es el contenedor de blobs, example/data/Gutenberg es la carpeta y davinci.txt es el blob. |
| output | Archivo de salida (incluida la ubicación) del reductor. La salida del trabajo de streaming de Hadoop se escribirá en la ubicación especificada para esta propiedad. |
| filePaths | Rutas de acceso a los archivos ejecutables del asignador y del reductor. En el ejemplo: "adfsample/example/apps/wc.exe", adfsample es el contenedor de blobs, example/apps es la carpeta y wc.exe es el ejecutable. |
| fileLinkedService | Servicio vinculado de Azure Storage que representa el almacenamiento de Azure que contiene los archivos especificados en la sección filePaths. |
| argumentos | Una lista de argumentos separados por comas para el programa de MapReduce. En tiempo de ejecución, verá unos argumentos adicionales (por ejemplo, mapreduce.job.tags) desde el marco de trabajo MapReduce. Para diferenciar sus argumentos con los argumentos de MapReduce, considere el uso tanto de opción como de valor como argumentos, tal como se muestra en el siguiente ejemplo (-s, --input, --output, etc., son opciones seguidas inmediatamente por sus valores). |
| getDebugInfo | Un elemento opcional. Cuando se establece en Failure, los registros solo se descargan en caso de error. Cuando se establece en All, los registros se descargan siempre, sea cual sea el estado de ejecución. |

> [!NOTE]
> Debe especificar un conjunto de datos de salida para la actividad de streaming de Hadoop en la propiedad **outputs**. Este conjunto de datos puede ser simplemente un conjunto de datos ficticio que es necesario para la programación de la canalización (por hora, diaria, etc.). Si la actividad no adopta una entrada, puede omitir la especificación de un conjunto de datos de entrada para la actividad en la propiedad **entradas**.

## <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "HadoopStreamingPipeline",
    "properties": {
        "description": "Hadoop Streaming Demo",
        "activities": [
            {
                "type": "HDInsightStreaming",
                "typeProperties": {
                    "mapper": "cat.exe",
                    "reducer": "wc.exe",
                    "input": "wasb://<nameofthecluster>@spestore.blob.core.windows.net/example/data/gutenberg/davinci.txt",
                    "output": "wasb://<nameofthecluster>@spestore.blob.core.windows.net/example/data/StreamingOutput/wc.txt",
                    "filePaths": ["<nameofthecluster>/example/apps/wc.exe","<nameofthecluster>/example/apps/cat.exe"],
                    "fileLinkedService": "StorageLinkedService",
                    "getDebugInfo": "Failure"
                },
                "outputs": [
                    {
                        "name": "StreamingOutputDataset"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "RunHadoopStreamingJob",
                "description": "Run a Hadoop streaming job",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2014-01-04T00:00:00",
        "end": "2014-01-05T00:00:00"
    }
}
```

Para más información, consulte el artículo [Actividad de streaming de Hadoop](data-factory-hadoop-streaming-activity.md).

## <a name="hdinsight-spark-activity"></a>Actividad de HDInsight Spark
Puede especificar las siguientes propiedades en una definición JSON de actividad de Spark. La propiedad type de la actividad debe ser: **HDInsightSpark**. Debe crear primero un servicio vinculado de HDInsight y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en HDInsightSpark:

| Propiedad | Descripción | Obligatorio |
| -------- | ----------- | -------- |
| rootPath | Contenedor de blobs de Azure y la carpeta que contiene el archivo de Spark. El nombre del archivo distingue mayúsculas de minúsculas. | Sí |
| entryFilePath | Ruta de acceso relativa a la carpeta raíz del código o el paquete de Spark. | Sí |
| className | Clase principal de Spark o Java de la aplicación. | No |
| argumentos | Lista de argumentos de línea de comandos del programa de Spark. | No |
| proxyUser | Cuenta de usuario de suplantación para ejecutar el programa de Spark. | No |
| sparkConfig | Propiedades de configuración de Spark. | No |
| getDebugInfo | Especifica si se copian los archivos de registro de Spark en el almacenamiento de Azure que usa el clúster de HDInsight que especifica sparkJobLinkedService. Valores permitidos: Ninguno, Siempre o Error. Valor predeterminado: Ninguno. | No |
| sparkJobLinkedService | El servicio vinculado de Azure Storage que contiene los registros, las dependencias y los archivos de trabajos de Spark.  Si no especifica un valor para esta propiedad, se usa el almacenamiento asociado con el clúster de HDInsight. | No |

### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "SparkPipeline",
    "properties": {
        "activities": [
            {
                "type": "HDInsightSpark",
                "typeProperties": {
                    "rootPath": "adfspark\\pyFiles",
                    "entryFilePath": "test.py",
                    "getDebugInfo": "Always"
                },
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "name": "MySparkActivity",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-02-05T00:00:00",
        "end": "2017-02-06T00:00:00"
    }
}
```
Tenga en cuenta los siguientes puntos:

- La propiedad de **tipo** se establece en **HDInsightSpark**.
- El valor de **rootPath** se establece en **adfspark\\pyFiles**, donde adfspark es el contenedor de Azure Blob y pyFiles es la carpeta adecuada en ese contenedor. En este ejemplo, la instancia de Azure Blob Storage es la que está asociada con el clúster de Spark. Puede cargar el archivo en una instancia de Azure Storage diferente. Si lo hace, cree un servicio vinculado de Azure Storage para vincular esa cuenta de almacenamiento con la factoría de datos. A continuación, especifique el nombre del servicio vinculado como un valor de la propiedad **sparkJobLinkedService**. Consulte las propiedades de la actividad de Spark para más detalles sobre esta y otras propiedades que admite la actividad de Spark.
- El valor de **entryFilePath** se establece en **test.py**, que es el archivo de Python.
- La propiedad **getDebugInfo** está establecida en **Siempre**, lo que significa que siempre se generan archivos de registro (acierto o error).

    > [!IMPORTANT]
    > Se recomienda que no establezca esta propiedad como Siempre en un entorno de producción, a menos que esté solucionando un problema.
- La sección de **salida** tiene un conjunto de datos de salida. Debe especificar un conjunto de datos de salida, incluso si el programa de Spark no genera ninguna salida. El conjunto de datos de salida impulsa la programación de la canalización (cada hora, cada día, etc.).

Para más información sobre la actividad, consulte el artículo [Actividad de Spark](data-factory-spark.md).

## <a name="azure-machine-learning-studio-classic-batch-execution-activity"></a>Actividad de ejecución de lotes de Azure Machine Learning Studio (clásico)
Puede especificar las siguientes propiedades en una definición JSON de actividad de ejecución de lotes de Azure Machine Learning Studio (clásico). La propiedad type de la actividad debe ser: **AzureMLBatchExecution**. Debe crear primero un servicio vinculado de Studio (clásico) y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en AzureMLBatchExecution:

Propiedad | Descripción | Obligatorio
-------- | ----------- | --------
webServiceInput | Conjunto de datos que se pasará como entrada para el servicio web de Studio (clásico). Este conjunto de datos también debe incluirse en las entradas de la actividad. |Use webServiceInput o webServiceInputs. |
webServiceInputs | Especifique los conjuntos de datos que se van a pasar como entradas para el servicio web de Studio (clásico). Si el servicio web toma varias entradas, use la propiedad webServiceInputs en lugar de usar webServiceInput. Los conjuntos de datos a los que hace referencia **webServiceInputs** también deben incluirse en las **entradas** de la actividad. | Use webServiceInput o webServiceInputs. |
webServiceOutputs | Conjuntos de datos que se asignan como salidas para el servicio web de Studio (clásico). El servicio web devuelve los datos de salida en este conjunto de datos. | Sí |
globalParameters | Especifique valores para los parámetros del servicio web en esta sección. | No |

### <a name="json-example"></a>Ejemplo JSON
En este ejemplo, la actividad tiene el conjunto de datos **MLSqlInput** como entrada y **MLSqlOutput** como salida. **MLSqlInput** se pasa como entrada al servicio web mediante la propiedad JSON **webServiceInput**. **MLSqlOutput** se pasa como salida al servicio web mediante la propiedad JSON **webServiceOutputs**.

```json
{
   "name": "MLWithSqlReaderSqlWriter",
   "properties": {
      "description": "Azure ML model with sql azure reader/writer",
      "activities": [{
         "name": "MLSqlReaderSqlWriterActivity",
         "type": "AzureMLBatchExecution",
         "description": "test",
         "inputs": [ { "name": "MLSqlInput" }],
         "outputs": [ { "name": "MLSqlOutput" } ],
         "linkedServiceName": "MLSqlReaderSqlWriterDecisionTreeModel",
         "typeProperties":
         {
            "webServiceInput": "MLSqlInput",
            "webServiceOutputs": {
               "output1": "MLSqlOutput"
            },
            "globalParameters": {
               "Database server name": "<myserver>.database.windows.net",
               "Database name": "<database>",
               "Server user account name": "<user name>",
               "Server user account password": "<password>"
            }
         },
         "policy": {
            "concurrency": 1,
            "executionPriorityOrder": "NewestFirst",
            "retry": 1,
            "timeout": "02:00:00"
         }
      }],
      "start": "2016-02-13T00:00:00",
       "end": "2016-02-14T00:00:00"
   }
}
```

En el ejemplo JSON, el servicio web de Studio (clásico) implementado usa un módulo lector y otro escritor para leer y escribir datos desde y hacia una base de datos de Azure SQL Database. Este servicio web expone los cuatro parámetros siguientes:  nombre del servidor de bases de datos, nombre de base de datos, nombre de la cuenta del usuario del servidor y contraseña de la cuenta de usuario del servidor.

> [!NOTE]
> Solo las entradas y salidas de la actividad AzureMLBatchExecution pueden pasarse como parámetros al servicio web. Por ejemplo, en el fragmento JSON anterior, MLSqlInput es una entrada a la actividad AzureMLBatchExecution, que se pasa como entrada al servicio web mediante el parámetro webServiceInput.

## <a name="azure-machine-learning-studio-classic-update-resource-activity"></a>Actividad de actualización de recurso de Azure Machine Learning Studio (clásico)
Puede especificar las siguientes propiedades en una definición JSON de actividad de actualización de recurso de Azure Machine Learning Studio (clásico). La propiedad type de la actividad debe ser: **AzureMLUpdateResource**. Debe crear primero un servicio vinculado de Studio (clásico) y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en AzureMLUpdateResource:

Propiedad | Descripción | Obligatorio
-------- | ----------- | --------
trainedModelName | Nombre del modelo reentrenado | Sí |
trainedModelDatasetName | Conjunto de datos que apunta al archivo iLearner devuelto por la operación de reentrenamiento | Sí |

### <a name="json-example"></a>Ejemplo JSON
La canalización tiene dos actividades: **AzureMLBatchExecution** y **AzureMLUpdateResource**. La actividad de ejecución de lotes de Studio (clásico) toma los datos de entrenamiento como entrada y genera como salida un archivo iLearner. La actividad invoca el servicio web de entrenamiento (el experimento de entrenamiento expuesto como servicio web) con los datos de entrenamiento de entrada y recibe el archivo ilearner desde el servicio web. El placeholderBlob es simplemente un conjunto de datos de salida ficticio que el servicio de Azure Data Factory necesita para ejecutar la canalización.


```json
{
    "name": "pipeline",
    "properties": {
        "activities": [
            {
                "name": "retraining",
                "type": "AzureMLBatchExecution",
                "inputs": [
                    {
                        "name": "trainingData"
                    }
                ],
                "outputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "typeProperties": {
                    "webServiceInput": "trainingData",
                    "webServiceOutputs": {
                        "output1": "trainedModelBlob"
                    }
                 },
                "linkedServiceName": "trainingEndpoint",
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            },
            {
                "type": "AzureMLUpdateResource",
                "typeProperties": {
                    "trainedModelName": "trained model",
                    "trainedModelDatasetName" :  "trainedModelBlob"
                },
                "inputs": [{ "name": "trainedModelBlob" }],
                "outputs": [{ "name": "placeholderBlob" }],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "AzureML Update Resource",
                "linkedServiceName": "updatableScoringEndpoint2"
            }
        ],
        "start": "2016-02-13T00:00:00",
        "end": "2016-02-14T00:00:00"
    }
}
```

## <a name="data-lake-analytics-u-sql-activity"></a>Actividad U-SQL de Análisis de Data Lake
Puede especificar las siguientes propiedades en una definición JSON de actividad de U-SQL. La propiedad type de la actividad debe ser: **DataLakeAnalyticsU-SQL**. Debe crear un servicio vinculado de Azure Data Lake Analytics y especificar el nombre de este como un valor para la propiedad **linkedServiceName**. Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en DataLakeAnalyticsU-SQL:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| scriptPath |Ruta de acceso a la carpeta que contiene el script U-SQL. El nombre del archivo distingue mayúsculas de minúsculas. |No (si se utiliza el script) |
| scriptLinkedService |Servicio vinculado que se vincula al almacenamiento que contiene el script para la factoría de datos |No (si se utiliza el script) |
| script |Especifique el script en línea en lugar de scriptPath y scriptLinkedService. Por ejemplo, "script": "CREATE DATABASE test". |No (si usa scriptPath y scriptLinkedService) |
| degreeOfParallelism |Número máximo de nodos que se usará de forma simultánea para ejecutar el trabajo. |No |
| priority |Determina qué trabajos de todos los están en cola deben seleccionarse para ejecutarse primero. Cuanto menor sea el número, mayor será la prioridad. |No |
| parámetros |Parámetros del script SQL U |No |

### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This pipeline computes events for en-gb locale and date less than Feb 19, 2012.",
        "activities":
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs":
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00",
        "end": "2015-08-08T01:00:00",
        "isPaused": false
    }
}
```

Para más información, consulte [Actividad de U-SQL de Data Lake Analytics](data-factory-usql-activity.md).

## <a name="stored-procedure-activity"></a>Actividad de procedimiento almacenado
Puede especificar las siguientes propiedades en una definición JSON de procedimiento almacenado. La propiedad type de la actividad debe ser: **SqlServerStoredProcedure**. Debe crear uno de los siguientes servicios vinculados y especificar el nombre de este como un valor para la propiedad **linkedServiceName**:

- SQL Server
- Azure SQL Database
- Azure Synapse Analytics

Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en SqlServerStoredProcedure:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| storedProcedureName |Especifique el nombre del procedimiento almacenado en Azure SQL Database o en Azure Synapse Analytics que se representa por medio del servicio vinculado que usa la tabla de salida. |Sí |
| storedProcedureParameters |Especifique valores para los parámetros del procedimiento almacenado. Si necesita pasar null para un parámetro, use la sintaxis: "param1": null (todo en minúsculas). Vea el ejemplo siguiente para aprender el uso de esta propiedad. |No |

Si especifica un conjunto de datos de entrada, debe estar disponible (en estado "Listo") para que se ejecute la actividad de procedimiento almacenado. El conjunto de datos de entrada no se puede usar en el procedimiento almacenado como parámetro. Solo se utiliza para comprobar la dependencia antes de iniciar la actividad de procedimiento almacenado. Debe especificar un conjunto de datos para una actividad de procedimiento almacenado.

El conjunto de datos de salida especifica la **programación** para la actividad de procedimiento almacenada (por hora, semanal, mensual, etc.). El conjunto de datos de salida debe usar un **servicio vinculado** que haga referencia a una instancia de Azure SQL Database, una instancia de Azure Synapse Analytics o una base de datos de SQL Server en la que quiera que el procedimiento almacenado se ejecute. El conjunto de datos de salida puede usarse como una forma de pasar el resultado del procedimiento almacenado para su posterior procesamiento por otra actividad ([encadenamiento de actividades](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)) en la canalización. Sin embargo, Data Factory no escribe automáticamente la salida de un procedimiento almacenado en este conjunto de datos. Es el procedimiento almacenado el que escribe en una tabla SQL a la que apunta el conjunto de datos de salida. En algunos casos, el conjunto de datos de salida puede ser un **conjunto de datos ficticio**, que solo se utilice para especificar la programación para ejecutar la actividad de procedimiento almacenado.

### <a name="json-example"></a>Ejemplo JSON

```json
{
    "name": "SprocActivitySamplePipeline",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "usp_sample",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)"
                    }
                },
                "outputs": [{ "name": "sprocsampleout" }],
                "name": "SprocActivitySample"
            }
        ],
         "start": "2016-08-02T00:00:00",
         "end": "2016-08-02T05:00:00",
        "isPaused": false
    }
}
```

Para más información, consulte el artículo [Actividad de procedimiento almacenado](data-factory-stored-proc-activity.md).

## <a name="net-custom-activity"></a>Actividad personalizada de .NET
Puede especificar las siguientes propiedades en una definición JSON de actividad personalizada de .NET. La propiedad type de la actividad debe ser: **DotNetActivity**. Debe crear un servicio vinculado de Azure HDInsight o un servicio vinculado de Azure Batch y especificar el nombre de dicho servicio vinculado como un valor para la propiedad **linkedServiceName**: Se admiten las siguientes propiedades en la sección **typeProperties** cuando se establece el tipo de actividad en DotNetActivity:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| AssemblyName | Nombre del ensamblado. En el ejemplo, es: **MyDotnetActivity.dll**. | Sí |
| EntryPoint |Nombre de la clase que implementa la interfaz IDotNetActivity. En el ejemplo, es: **MyDotNetActivityNS.MyDotNetActivity**, donde MyDotNetActivityNS es el espacio de nombres y MyDotNetActivity es la clase.  | Sí |
| PackageLinkedService | Nombre del servicio vinculado de Azure Storage que apunta a una instancia de Blob Storage que contiene el archivo ZIP de la actividad personalizada. En el ejemplo, es: **AzureStorageLinkedService**.| Sí |
| PackageFile | Nombre del archivo ZIP. En el ejemplo, es: **customactivitycontainer/MyDotNetActivity.zip**. | Sí |
| extendedProperties | Propiedades extendidas que puede definir y pasar a código .NET. En este ejemplo, la variable **SliceStart** se establece en un valor basado en la variable del sistema SliceStart. | No |

### <a name="json-example"></a>Ejemplo JSON

```json
{
  "name": "ADFTutorialPipelineCustom",
  "properties": {
    "description": "Use custom activity",
    "activities": [
      {
        "Name": "MyDotNetActivity",
        "Type": "DotNetActivity",
        "Inputs": [
          {
            "Name": "InputDataset"
          }
        ],
        "Outputs": [
          {
            "Name": "OutputDataset"
          }
        ],
        "LinkedServiceName": "AzureBatchLinkedService",
        "typeProperties": {
          "AssemblyName": "MyDotNetActivity.dll",
          "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
          "PackageLinkedService": "AzureStorageLinkedService",
          "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
          "extendedProperties": {
            "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"
          }
        },
        "Policy": {
          "Concurrency": 2,
          "ExecutionPriorityOrder": "OldestFirst",
          "Retry": 3,
          "Timeout": "00:30:00",
          "Delay": "00:00:00"
        }
      }
    ],
    "start": "2016-11-16T00:00:00",
    "end": "2016-11-16T05:00:00",
    "isPaused": false
  }
}
```

Para obtener información detallada, vea el artículo [Uso de actividades personalizadas en Data Factory](data-factory-use-custom-activities.md).

## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes tutoriales:

- [Tutorial: crear una canalización con la actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
- [Tutorial: crear una canalización con la actividad de hive](data-factory-build-your-first-pipeline.md)
