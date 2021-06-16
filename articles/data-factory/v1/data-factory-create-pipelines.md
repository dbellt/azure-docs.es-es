---
title: Creación y programación de canalizaciones y actividades en cadena en Data Factory
description: Aprenda a crear una canalización de datos en Azure Data Factory para mover y transformar datos. Crear un flujo de trabajo controlado por datos para producir información lista para usar.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 83d7066ca003cc935526abd6b7fd4d456b4dd7d1
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665850"
---
# <a name="pipelines-and-activities-in-azure-data-factory"></a>Canalizaciones y actividades en Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-create-pipelines.md)
> * [Versión 2 (versión actual)](../concepts-pipelines-activities.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Pipelines in V2](../concepts-pipelines-activities.md) (Canalizaciones en V2).

Este artículo ayuda a conocer las canalizaciones y actividades de Azure Data Factory y a usarlas para construir flujos de trabajo controlados por datos de un extremo para los escenarios de procesamiento de datos y movimiento de datos.

> [!NOTE]
> En este artículo se considera que ha repasado antes los artículos [Introducción a Azure Data Factory](data-factory-introduction.md). Si no tiene experiencia práctica con la creación de factorías de datos, repasar los tutoriales sobre [transformación](data-factory-build-your-first-pipeline.md) o [traslado de datos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) puede ayudarlo a comprender mejor este artículo.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general
Una factoría de datos puede tener una o más canalizaciones. Una canalización es una agrupación lógica de actividades que realizan una tarea. Las actividades de una canalización definen las acciones que se van a realizar en los datos. Por ejemplo, puede usar una actividad de copia para copiar datos de una base de datos de SQL Server a una instancia de Azure Blob Storage. Después, utilice una actividad de Hive que ejecute un script de Hive en un clúster de Azure HDInsight para procesar o transformar datos de Blob Storage con el fin de generar datos de salida. Finalmente, use una segunda actividad de copia para copiar los datos de salida en Azure Synapse Analytics, en función de qué soluciones de generación de informes de inteligencia empresarial (BI) estén integradas.

Una actividad puede tomar diversos [conjuntos de datos](data-factory-create-datasets.md), o ninguno, y generar uno o varios [conjuntos de datos](data-factory-create-datasets.md). En el siguiente diagrama se muestra la relación entre la canalización, la actividad y el conjunto de datos en Data Factory:

![Relación entre la canalización, la actividad y el conjunto de datos](media/data-factory-create-pipelines/relationship-pipeline-activity-dataset.png)

Una canalización permite administrar actividades como un conjunto en lugar de hacerlo individualmente. Por ejemplo, puede implementar, programar, suspender y reanudar una canalización, en vez de trabajar con actividades de la canalización de forma independiente.

Data Factory admite dos tipos de actividades: actividades de movimiento de datos y actividades de transformación de datos. Cada actividad puede tener cero o más [conjuntos de datos](data-factory-create-datasets.md) de entrada y generar uno o más conjuntos de datos de salida.

Un conjunto de datos de entrada representa la entrada para una actividad de la canalización y un conjunto de datos de salida representa la salida de la actividad. Los conjuntos de datos identifican datos en distintos almacenes de datos, como tablas, archivos, carpetas y documentos. Después de crear un conjunto de datos, puede usarlo con las actividades de una canalización. Por ejemplo, un conjunto de datos puede ser un conjunto de datos de entrada y salida de una actividad de copia o una actividad de HDInsightHive. Para obtener más información sobre los conjuntos de datos, vea el artículo [Conjuntos de datos en Azure Data Factory](data-factory-create-datasets.md).

### <a name="data-movement-activities"></a>Actividades de movimiento de datos
Copiar actividad en Data Factory realiza una copia de los datos de un almacén de datos de origen a uno receptor. Data Factory admite los siguientes almacenes de datos. Se pueden escribir datos desde cualquier origen en todos los tipos de receptores. Haga clic en un almacén de datos para obtener información sobre cómo copiar datos a un almacén como origen o destino.

[!INCLUDE [data-factory-supported-data-stores](includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Los almacenes de datos con * pueden ser locales o estar en la IaaS de Azure; además, requieren que instale [Data Management Gateway](data-factory-data-management-gateway.md) en una máquina local o de la IaaS de Azure.

Para más información, consulte el artículo sobre [actividades de movimiento de datos](data-factory-data-movement-activities.md).

### <a name="data-transformation-activities"></a>Actividades de transformación de datos
[!INCLUDE [data-factory-transformation-activities](includes/data-factory-transformation-activities.md)]

Para más información, consulte el artículo sobre [actividades de transformación de datos](data-factory-data-transformation-activities.md).

### <a name="custom-net-activities"></a>Actividades personalizadas de .NET
Si necesita mover datos con un almacén de datos como origen o destino que no sea compatible con la actividad de copia, o bien transformar datos con su propia lógica, cree una **actividad personalizada de .NET**. Consulte el artículo [Uso de actividades personalizadas en una canalización de Azure Data Factory](data-factory-use-custom-activities.md)para obtener más información sobre la creación y el uso de una actividad personalizada.

## <a name="schedule-pipelines"></a>Programar canalizaciones
Una canalización solo está activa entre su hora de **inicio** y de **finalización**. No se ejecuta antes de la hora de inicio ni después de la hora de finalización. Si la canalización está en pausa, no se ejecutará, independientemente de su hora de inicio y de finalización. Para que se ejecute una canalización, no debe estar en pausa. Consulte [Programación y ejecución](data-factory-scheduling-and-execution.md) para comprender cómo funciona la programación y la ejecución en Azure Data Factory.

## <a name="pipeline-json"></a>JSON de canalización
Vamos a fijarnos un poco más en cómo se define una canalización en formato JSON. La estructura genérica de una canalización tiene el aspecto siguiente:

```json
{
    "name": "PipelineName",
    "properties":
    {
        "description" : "pipeline description",
        "activities":
        [

        ],
        "start": "<start date-time>",
        "end": "<end date-time>",
        "isPaused": true/false,
        "pipelineMode": "scheduled/onetime",
        "expirationTime": "15.00:00:00",
        "datasets":
        [
        ]
    }
}
```

| Etiqueta | Descripción | Obligatorio |
| --- | --- | --- |
| name |Nombre de la canalización. Especifique un nombre que represente la acción que realizará la canalización. <br/><ul><li>Número máximo de caracteres: 260</li><li>Debe empezar por una letra, un número o un carácter de subrayado (\_)</li><li>No se permiten los caracteres siguientes: “.”, “+”, “?”, “/”, “<”,”>”,”\*”,”%”,”&”,”:”,”\\”</li></ul> |Sí |
| description | Especifique el texto que describe para qué se usa la canalización. |Sí |
| activities | La sección **activities** puede contener una o más actividades definidas. Vea la sección siguiente para obtener más información sobre el elemento JSON de actividades. | Sí |
| start | Fecha y hora de inicio para la canalización. Debe estar en [formato ISO](https://en.wikipedia.org/wiki/ISO_8601). Por ejemplo: `2016-10-14T16:32:41Z`. <br/><br/>Se puede especificar una hora local, por ejemplo, una hora EST. Aquí tiene un ejemplo: `2016-02-27T06:00:00-05:00`, que es 06:00 EST.<br/><br/>Las propiedades start y end juntas especifican un período activo para la canalización. Los segmentos de salida solo se producen en este período activo. |No<br/><br/>Si se especifica un valor para la propiedad end, hay que especificar un valor para la propiedad start.<br/><br/>Las horas de inicio y finalización pueden estar vacías para crear una canalización. Debe especificar ambos valores para establecer un período activo para que se ejecute la canalización. Si no especifica las horas de inicio y de finalización al crear una canalización, puede establecerlas más adelante mediante el cmdlet Set-AzDataFactoryPipelineActivePeriod. |
| end | Fecha y hora de finalización para la canalización. Si se especifica, debe estar en formato ISO. Por ejemplo: `2016-10-14T17:32:41Z` <br/><br/>Se puede especificar una hora local, por ejemplo, una hora EST. Aquí tiene un ejemplo: `2016-02-27T06:00:00-05:00`, que es 6 a. m. EST.<br/><br/>Para ejecutar la canalización indefinidamente, especifique 9999-09-09 como valor de la propiedad end. <br/><br/> Una canalización solo está activa entre su hora de inicio y de finalización. No se ejecuta antes de la hora de inicio ni después de la hora de finalización. Si la canalización está en pausa, no se ejecutará, independientemente de su hora de inicio y de finalización. Para que se ejecute una canalización, no debe estar en pausa. Consulte [Programación y ejecución](data-factory-scheduling-and-execution.md) para comprender cómo funciona la programación y la ejecución en Azure Data Factory. |No <br/><br/>Si se especifica un valor para la propiedad start, hay que especificar un valor para la propiedad end.<br/><br/>Consulte las notas para la propiedad **start** . |
| isPaused | Si se establece en True, la canalización no se ejecuta. Está en estado de pausa. Valor predeterminado = false. Puede utilizar esta propiedad para habilitar o deshabilitar una canalización. |No |
| pipelineMode | El método para la programación de ejecuciones para la canalización. Los valores permitidos son: scheduled (predeterminado).<br/><br/>‘Scheduled’ indica que la canalización se ejecutará en el intervalo de tiempo especificado de acuerdo con su período activo (hora de inicio y finalización). 'Onetime' indica que la canalización se ejecutará solo una vez. Las canalizaciones creadas una sola vez no se pueden modificar ni actualizar actualmente. Consulte la sección [Canalización de una vez](#onetime-pipeline) para más información acerca de la configuración onetime. |No |
| expirationTime | Tiempo de duración después de la creación durante el que la [canalización de un solo uso](#onetime-pipeline) es válida y debe permanecer aprovisionada. La canalización se elimina automáticamente una vez que alcanza la hora de caducidad si no tiene ninguna ejecución activa, con error o pendiente. Valor predeterminado: `"expirationTime": "3.00:00:00"`|No |
| conjuntos de datos |Lista de conjuntos de datos que usarán las actividades definidas en la canalización. Esta propiedad puede utilizarse para definir conjuntos de datos que son específicos de esta canalización y que no se definen dentro de la factoría de datos. Los conjuntos de datos definidos dentro de esta canalización solo pueden utilizarse en ella y no se pueden compartir. Consulte la sección [Conjuntos de datos limitados](data-factory-create-datasets.md#scoped-datasets) para más información. |No |

## <a name="activity-json"></a>Actividad de JSON
La sección **activities** puede contener una o más actividades definidas. Cada actividad tiene la siguiente estructura de nivel superior:

```json
{
    "name": "ActivityName",
    "description": "description",
    "type": "<ActivityType>",
    "inputs": "[]",
    "outputs": "[]",
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

En la tabla siguiente se describen las propiedades en la definición JSON de la actividad:

| Etiqueta | Descripción | Obligatorio |
| --- | --- | --- |
| name | Nombre de la actividad. Especifique un nombre que represente la acción que realizará la actividad. <br/><ul><li>Número máximo de caracteres: 260</li><li>Debe empezar por una letra, un número o un carácter de subrayado (\_)</li><li>No se permiten los caracteres siguientes: “.”, “+”, “?”, “/”, “<”,”>”,”*”,”%”,”&”,”:”,”\\”</li></ul> |Sí |
| description | Texto que describe para qué se usa la actividad. |Sí |
| type | Tipo de la actividad. Consulte los artículos [Actividades de movimiento de datos](#data-movement-activities) y [Actividades de transformación de datos](#data-transformation-activities) para ver los diferentes tipos de actividades. |Sí |
| inputs |Tablas de entrada utilizadas por la actividad<br/><br/>`// one input table`<br/>`"inputs":  [ { "name": "inputtable1"  } ],`<br/><br/>`// two input tables` <br/>`"inputs":  [ { "name": "inputtable1"  }, { "name": "inputtable2"  } ],` |Sí |
| outputs |Tablas de salida utilizadas por la actividad.<br/><br/>`// one output table`<br/>`"outputs":  [ { "name": "outputtable1" } ],`<br/><br/>`//two output tables`<br/>`"outputs":  [ { "name": "outputtable1" }, { "name": "outputtable2" }  ],` |Sí |
| linkedServiceName |Nombre del servicio vinculado utilizado por la actividad. <br/><br/>Una actividad puede requerir que especifique el servicio vinculado que enlaza con el entorno de procesos necesario. |Sí para actividad de HDInsight y actividad de puntuación por lotes de Azure Machine Learning Studio (clásico) <br/><br/>No para todos los demás |
| typeProperties |Las propiedades de la sección **typeProperties** dependen del tipo de actividad. Para ver las propiedades de tipo de una actividad, haga clic en vínculos a la actividad de la sección anterior. | No |
| policy |Directivas que afectan al comportamiento en tiempo de ejecución de la actividad. Si no se especifica, se usan las directivas predeterminadas. |No |
| scheduler | La propiedad "scheduler" se utiliza para definir la programación deseada para la actividad. Sus subpropiedades son las mismas que las de la [propiedad availability de un conjunto de datos](data-factory-create-datasets.md#dataset-availability). |No |

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

## <a name="sample-copy-pipeline"></a>Canalización de copia de ejemplo
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
    "start": "2016-07-12T00:00:00Z",
    "end": "2016-07-13T00:00:00Z"
  }
}
```

Tenga en cuenta los siguientes puntos:

* En la sección de actividades, solo hay una actividad con **type** establecido en **Copy**.
* La entrada de la actividad está establecida en **InputDataset**, mientras que la salida está establecida en **OutputDataset**. Vea el artículo [Conjuntos de datos](data-factory-create-datasets.md) para definir conjuntos de datos en JSON.
* En la sección **typeProperties**, **BlobSource** se especifica como el tipo de origen y **SqlSink** como el tipo de receptor. En la sección [Actividades de movimiento de datos](#data-movement-activities), haga clic en el almacén de datos que quiere usar como origen o un receptor para obtener más información sobre cómo mover datos con ese almacén de datos como origen o destino.

Para obtener un tutorial completo de creación de esta canalización, consulte [Tutorial: Copia de datos de Blob Storage en SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="sample-transformation-pipeline"></a>Canalización de transformación de ejemplo
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
        "start": "2016-04-01T00:00:00Z",
        "end": "2016-04-02T00:00:00Z",
        "isPaused": false
    }
}
```

Tenga en cuenta los siguientes puntos:

* En la sección de actividades, solo hay una actividad con **type** establecido en **HDInsightHive**.
* El archivo de script de Hive, **partitionweblogs.hql**, se almacena en la cuenta de Azure Storage (especificada por la propiedad scriptLinkedService, llamada **AzureStorageLinkedService**) en una carpeta denominada **script** del contenedor **adfgetstarted**.
* La sección `defines` se usa para especificar la configuración de tiempo de ejecución que se pasa al script de Hive como valores de configuración de Hive (por ejemplo, `${hiveconf:inputtable}`, `${hiveconf:partitionedtable}`).

La sección **typeProperties** es diferente para cada actividad de transformación. Para obtener información sobre las propiedades de tipo compatibles con una actividad de transformación, haga clic en la actividad de transformación de la tabla [Actividades de transformación de datos](#data-transformation-activities).

Para obtener un tutorial completo de creación de esta canalización, consulte [Tutorial: Compilación de la primera canalización para procesar datos mediante el clúster de Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="multiple-activities-in-a-pipeline"></a>Varias actividades en una canalización
Las dos canalizaciones de ejemplo anteriores solo tienen una actividad, pero se pueden tener más de una actividad en una canalización.

Si tiene varias actividades en una canalización y la salida de una actividad no es la entrada de otra actividad, las actividades se pueden ejecutar en paralelo si los segmentos de datos de entrada para las actividades están listos.

Puede encadenar dos actividades haciendo que el conjunto de datos de salida de una actividad sea el conjunto de datos de entrada de la otra actividad. La segunda actividad se ejecuta solo cuando la primera de ellas se completa correctamente.

![Encadenamiento de las actividades de la misma canalización](./media/data-factory-create-pipelines/chaining-one-pipeline.png)

En este ejemplo, la canalización tiene dos actividades: Activity1 y Activity2. Activity1 toma Dataset1 como entrada y genera una salida Dataset2. La actividad toma Dataset2 como entrada y genera una salida Dataset3. Como la salida de Activity1 (Dataset2) es la entrada de Activity2, Activity2 solo se ejecuta después de que la actividad se complete correctamente y genere el segmento de Dataset2. Si se produce un error en Activity1 por algún motivo y no genera el segmento de Dataset2, Activity2 no se ejecuta en dicho segmento (por ejemplo: de 09:00 a 10:00).

También puede encadenar actividades que se encuentran en diferentes canalizaciones.

![Encadenamiento de las actividades de dos canalizaciones](./media/data-factory-create-pipelines/chaining-two-pipelines.png)

En este ejemplo, Pipeline1 solo tiene una actividad que toma Dataset1 como entrada y genera Dataset2 como salida. Pipeline2 también tiene únicamente una actividad que toma Dataset2 como entrada y Dataset3 como salida.

Para obtener más información, vea [Programación y ejecución](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).
## <a name="create-and-monitor-pipelines"></a>Creación y supervisión de canalizaciones
Puede crear canalizaciones utilizando una de estas herramientas o SDK.

- Asistente para copia
- Visual Studio
- Azure PowerShell
- Plantilla del Administrador de recursos de Azure
- API DE REST
- API de .NET

Vea los siguientes tutoriales para obtener instrucciones paso a paso sobre cómo crear canalizaciones con una de estas herramientas o SDK.

- [Creación de una canalización con una actividad de transformación de datos](data-factory-build-your-first-pipeline.md)
- [Creación de una canalización con una actividad de movimiento de datos](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

Una vez que se haya creado o implementado la canalización, puede administrar y supervisar sus canalizaciones utilizando las hojas de Azure Portal o la aplicación de supervisión y administración. Vaya a los siguientes temas para obtener instrucciones paso a paso.

- [Supervisión y administración de canalizaciones con las hojas de Azure Portal](data-factory-monitor-manage-pipelines.md)
- [Supervisión y administración de canalizaciones con la aplicación de supervisión y administración](data-factory-monitor-manage-app.md)

## <a name="onetime-pipeline"></a>Canalización de una vez
Puede crear y programar una canalización que se ejecute periódicamente (por ejemplo, cada hora y diariamente) entre las horas de inicio y finalización que especifique en la definición de la canalización. Para más información, consulte Programación de actividades. También puede crear una canalización que se ejecute una sola vez. Para ello, establezca la propiedad **pipelineMode** en **onetime** en la definición de la canalización,como se muestra en el siguiente ejemplo de JSON. El valor predeterminado de esta propiedad es **scheduled**.

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
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
                "name": "CopyActivity-0"
            }
        ],
        "pipelineMode": "OneTime"
    }
}
```

Tenga en cuenta lo siguiente:

* No se especifican las horas de **inicio** y **finalización** de la canalización.
* La disponibilidad (**availability**) de los conjuntos de datos de entrada y salida se especifica (**frequency** e **interval**), incluso aunque Data Factory no use los valores.
* La vista Diagrama no muestra las canalizaciones de una vez. Este comportamiento es así por diseño.
* Las canalizaciones de una vez no se pueden actualizar. Puede clonar una canalización de una vez, cambiarle el nombre, actualizar las propiedades e implementarla para crear otra.

## <a name="next-steps"></a>Pasos siguientes
- Para obtener más información sobre los conjuntos de datos, vea el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md).
- Si quiere conocer más detalles sobre cómo programar y ejecutar canalizaciones en Azure Data Factory, consulte [este artículo](data-factory-scheduling-and-execution.md).
