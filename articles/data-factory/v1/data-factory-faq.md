---
title: 'Azure Data Factory: preguntas más frecuentes'
description: Preguntas más frecuentes acerca de Azure Data Factory.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: b8f00715eac391e9180c36d8960f54c95772c564
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753178"
---
# <a name="azure-data-factory---frequently-asked-questions"></a>Azure Data Factory: preguntas más frecuentes
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [preguntas más frecuentes acerca de Data Factory](../frequently-asked-questions.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="general-questions"></a>Preguntas generales
### <a name="what-is-azure-data-factory"></a>¿Qué es Azure Data Factory?
Data Factory es un servicio de integración de datos basado en la nube que **automatiza el movimiento y la transformación de datos**. Lo mismo que las máquinas de una fábrica toman las materias primas y las transforman en productos acabados, Data Factory organiza los servicios existentes que recopilan datos sin procesar y los transforman en información lista para usar.

Data Factory le permite crear flujos de trabajo basados en datos para mover datos entre almacenes de datos locales y en la nube, así como para procesar o transformar datos mediante servicios de proceso como Azure HDInsight y Azure Data Lake Analytics. Después de crear una canalización que realice la acción que necesita, puede programar que se ejecute periódicamente (cada hora, diariamente, semanalmente, etc.).   

Para más información, consulte [Información general y conceptos clave](data-factory-introduction.md).

### <a name="where-can-i-find-pricing-details-for-azure-data-factory"></a>¿Dónde puedo encontrar detalles de precios de Azure Data Factory?
Vea la página de [detalles de precios de Data Factory][adf-pricing-details] para obtener información al respecto.  

### <a name="how-do-i-get-started-with-azure-data-factory"></a>¿Cómo puedo comenzar con Azure Data Factory?
* Para obtener información general sobre Azure Data Factory, vea [Introducción a Azure Data Factory](data-factory-introduction.md).
* Para ver un tutorial sobre cómo **copiar o mover datos** mediante la actividad de copia, consulte [Copiar datos desde Azure Blob Storage a Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
* Para ver un tutorial sobre cómo **transformar los datos** mediante HDInsight Hive Activity, consulte [Procesamiento de datos mediante la ejecución de scripts de Hive en un clúster Hadoop](data-factory-build-your-first-pipeline.md).

### <a name="what-is-the-data-factorys-region-availability"></a>¿Cuál es la disponibilidad de regiones de Data Factory?
Data Factory está disponible en las regiones **Oeste de EE. UU.** y **Norte de Europa**. Los servicios de Compute y Storage utilizados por las factorías de datos pueden estar en otras regiones. Consulte las [regiones admitidas](data-factory-introduction.md#supported-regions).

### <a name="what-are-the-limits-on-number-of-data-factoriespipelinesactivitiesdatasets"></a>¿Cuáles son los límites en el número de factorías, canalizaciones, actividades y conjuntos de datos?
Consulte la sección **Límites de Azure Data Factory** del artículo [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits) .

### <a name="what-is-the-authoringdeveloper-experience-with-azure-data-factory-service"></a>¿Cuál es la experiencia de desarrollador/creación con el servicio de Azure Data Factory?
Puede crear factorías de datos mediante una de las siguientes herramientas y SDK:

* **Visual Studio** : puede usar Visual Studio para crear una instancia de Data Factory de Azure. Consulte [Compilación de la primera Data Factory de Azure mediante Microsoft Visual Studio](data-factory-build-your-first-pipeline-using-vs.md) .
* **Azure PowerShell** : consulte el tutorial [Creación y supervisión de Azure Data Factory mediante Azure PowerShell](data-factory-build-your-first-pipeline-using-powershell.md) para crear una factoría de datos mediante PowerShell. Consulte el contenido de [Referencia de cmdlets de Data Factory][adf-powershell-reference] en MSDN Library para obtener documentación completa de los cmdlets de Factoría de datos.
* **Biblioteca de clases .NET** Se pueden crear factorías de datos mediante programación con el SDK de .NET de Data Factory. Consulte [Creación, supervisión y administración de las factorías de datos mediante el SDK de .NET](data-factory-create-data-factories-programmatically.md) para un ver tutorial sobre la creación de una factoría de datos con el SDK de .NET. Consulte [Referencia de biblioteca de clases de Data Factory][msdn-class-library-reference] para una amplia documentación sobre el SDK de .NET de Factoría de datos.
* **API de REST** También puede usar la API de REST expuesta por el servicio Azure Data Factory para crear e implementar factorías de datos. Consulte [Referencia de la API REST de Data Factory][msdn-rest-api-reference] para ver una amplia documentación sobre la API REST de Data Factory.
* **Plantilla de Azure Resource Manager** Consulte [Tutorial: Creación de la primera factoría de datos de Azure con la plantilla de Azure Resource Manager](data-factory-build-your-first-pipeline-using-arm.md) para detalles.

### <a name="can-i-rename-a-data-factory"></a>¿Se puede cambiar el nombre de una Factoría de datos?
No. Al igual que otros recursos de Azure, el nombre de una factoría de datos de Azure no se puede cambiar.

### <a name="can-i-move-a-data-factory-from-one-azure-subscription-to-another"></a>¿Puedo mover una factoría de datos de una suscripción a Azure a otra?
Sí. Use el botón **Mover** situado en la hoja de la factoría de datos tal como se muestra en el siguiente diagrama:

![Mover factoría de datos](media/data-factory-faq/move-data-factory.png)

### <a name="what-are-the-compute-environments-supported-by-data-factory"></a>¿Cuáles son los entornos de proceso compatibles con Data Factory?
En la tabla siguiente se proporciona una lista de entornos de proceso compatibles con Data Factory y las actividades que se pueden ejecutar en ellos.

| Entorno de procesos | activities |
| --- | --- |
| [Clúster de HDInsight a petición](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) o [clúster HDInsight propio](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) |[DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [Hadoop Streaming](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](data-factory-compute-linked-services.md#azure-batch-linked-service) |[DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning Studio (clásico)](data-factory-compute-linked-services.md#azure-machine-learning-studio-classic-linked-service) |[Actividades de Studio (clásico): ejecución de lotes y recurso de actualización](data-factory-azure-ml-batch-execution-activity.md) |
| [Análisis con Azure Data Lake](data-factory-compute-linked-services.md#azure-data-lake-analytics-linked-service) |[U-SQL de análisis con Data Lake](data-factory-usql-activity.md) |
| [Azure SQL](data-factory-compute-linked-services.md#azure-sql-linked-service), [Azure Synapse Analytics](data-factory-compute-linked-services.md#azure-synapse-analytics-linked-service), [SQL Server](data-factory-compute-linked-services.md#sql-server-linked-service) |[Procedimiento almacenado](data-factory-stored-proc-activity.md) |

### <a name="how-does-azure-data-factory-compare-with-sql-server-integration-services-ssis"></a>¿Cómo se compara Azure Data Factory con SQL Server Integration Services (SSIS)? 
Consulte la presentación [Azure Data Factory frente a SSIS](https://www.sqlbits.com/Sessions/Event15/Azure_Data_Factory_vs_SSIS) de uno de nuestros MVP (profesionales más valiosos): Reza Rad. Puede que algunos de los cambios recientes de Data Factory no aparezcan en las diapositivas. Continuamente estamos agregando más funcionalidades a Azure Data Factory. Continuamente estamos agregando más funcionalidades a Azure Data Factory. Se incorporarán estas actualizaciones en la comparación de tecnologías de integración de datos de Microsoft este año más adelante.   

## <a name="activities---faq"></a>Actividades: preguntas más frecuentes
### <a name="what-are-the-different-types-of-activities-you-can-use-in-a-data-factory-pipeline"></a>¿Qué tipos de actividades diferentes se pueden usar en una canalización de Data Factory?
* [Actividades de movimiento de datos](data-factory-data-movement-activities.md) para mover los datos.
* [Actividades de transformación de datos](data-factory-data-transformation-activities.md) para procesar o transformar los datos.

### <a name="when-does-an-activity-run"></a>¿Cuándo se ejecuta una actividad?
La configuración de **disponibilidad** en la tabla de datos de salida determina cuándo se ejecuta la actividad. Si se especifican conjuntos de datos de entrada, la actividad comprueba si se cumplen todas las dependencias de datos de entrada (es decir, estado **Listo** ) antes de ejecutarse.

## <a name="copy-activity---faq"></a>Actividad de copia: preguntas más frecuentes
### <a name="is-it-better-to-have-a-pipeline-with-multiple-activities-or-a-separate-pipeline-for-each-activity"></a>¿Es mejor tener una canalización con varias actividades o una canalización independiente para cada actividad?
Se supone que las canalizaciones incluyen actividades relacionadas. Si los conjuntos de datos que las conectan no se consumen en ninguna otra actividad fuera de la canalización, puede mantener estas actividades en una canalización. De este modo, no necesitará períodos activos de canalizaciones de cadena puesto que se alinean con las demás. Además, la integridad de los datos de las tablas internas de la canalización se conservará mejor cuando se actualice la canalización. La actualización de la canalización detiene fundamentalmente todas las actividades en la canalización, las elimina y las vuelve a crear. Desde la perspectiva de la creación, puede ser más fácil ver el flujo de datos dentro de las actividades relacionadas en un archivo JSON para la canalización.

### <a name="what-are-the-supported-data-stores"></a>¿Qué almacenes de datos son compatibles?
Copiar actividad en Data Factory realiza una copia de los datos de un almacén de datos de origen a uno receptor. Data Factory admite los siguientes almacenes de datos. Se pueden escribir datos desde cualquier origen en todos los tipos de receptores. Haga clic en un almacén de datos para obtener información sobre cómo copiar datos a un almacén como origen o destino.

[!INCLUDE [data-factory-supported-data-stores](includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Los almacenes de datos con * pueden ser locales o estar en la IaaS de Azure; además, requieren que instale [Data Management Gateway](data-factory-data-management-gateway.md) en una máquina local o de la IaaS de Azure.

### <a name="what-are-the-supported-file-formats"></a>¿Qué formatos de archivo son compatibles?
Azure Data Factory admite los siguientes tipos de formato de archivo:

* [Formato de texto](data-factory-supported-file-and-compression-formats.md#text-format)
* [Formato JSON](data-factory-supported-file-and-compression-formats.md#json-format)
* [Formato Avro](data-factory-supported-file-and-compression-formats.md#avro-format)
* [Formato ORC](data-factory-supported-file-and-compression-formats.md#orc-format)
* [Formato Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format)

### <a name="where-is-the-copy-operation-performed"></a>¿Dónde se realiza la operación de copia?
Consulte la sección [Movimiento de datos disponible globalmente](data-factory-data-movement-activities.md#global) para obtener más información. En resumen, cuando en el proceso participa un almacén de datos local, la operación de copia la realiza Data Management Gateway en el entorno local. Asimismo, cuando el movimiento de datos se produce entre dos almacenes en la nube, la operación de copia se realiza en la región más cercana a la ubicación del receptor en la misma zona geográfica.

## <a name="hdinsight-activity---faq"></a>Actividad de HDInsight: preguntas más frecuentes
### <a name="what-regions-are-supported-by-hdinsight"></a>¿Qué regiones admiten HDInsight?
Vea la sección Disponibilidad geográfica en el siguiente artículo: o [Detalles de precios de HDInsight][hdinsight-supported-regions].

### <a name="what-region-is-used-by-an-on-demand-hdinsight-cluster"></a>¿Qué región se usa con un clúster de HDInsight a petición?
El clúster de HDInsight a petición se crea en la misma región que existe para el almacenamiento que se va a usar con el clúster.    

### <a name="how-to-associate-additional-storage-accounts-to-your-hdinsight-cluster"></a>¿Cómo se asocian cuentas de almacenamiento adicionales al clúster de HDInsight?
Si está usando su propio clúster de HDInsight (BYOC, Bring Your Own Cluster, traiga su propio clúster), consulte los temas siguientes:

* [Uso de un clúster de HDInsight con cuentas de almacenamiento y tiendas de metadatos alternativas][hdinsight-alternate-storage]
* [Uso de cuentas de almacenamiento adicionales con HDInsight Hive][hdinsight-alternate-storage-2]

Si usa un clúster a petición que se crea con el servicio Data Factory, especifique cuentas de almacenamiento adicionales para el servicio vinculado de HDInsight de manera que dicho servicio pueda registrarlas en su nombre. En la definición de JSON para el servicio vinculado a petición, use la propiedad **additionalLinkedServiceNames** para especificar las cuentas de almacenamiento alternativas como se muestra en el siguiente fragmento JSON:

```JSON
{
    "name": "MyHDInsightOnDemandLinkedService",
    "properties":
    {
        "type": "HDInsightOnDemandLinkedService",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "LinkedService-SampleData",
            "additionalLinkedServiceNames": [ "otherLinkedServiceName1", "otherLinkedServiceName2" ]
        }
    }
}
```
En el ejemplo anterior, otherLinkedServiceName1 y otherLinkedServiceName2 representan servicios vinculados cuyas definiciones contienen las credenciales que el clúster de HDInsight necesita para acceder a las cuentas de almacenamiento alternativas.

## <a name="slices---faq"></a>Segmentos: preguntas más frecuentes
### <a name="why-are-my-input-slices-not-in-ready-state"></a>¿Por qué mis segmentos de entrada no presentan el estado Listo?
Un error común consiste en no definir la propiedad **external** en **rue** en el conjunto de datos de entrada cuando los datos de entrada son externos a la factoría de datos (es decir, que no produce esta).

En el ejemplo siguiente, solo debe establecer **external** en true en **dataset1**.  

**DataFactory1** Pipeline 1: dataset1 -&gt; activity1 -&gt; dataset2 -&gt; activity2 -&gt; dataset3 Pipeline 2: dataset3-&gt; activity3 -&gt; dataset4

Si tiene otra factoría de datos con una canalización que toma dataset4 (producido por la canalización 2 de la factoría de datos 1), marque dataset4 como un conjunto de datos externo porque está generado por una factoría de datos distinta (DataFactory1, no DataFactory2).  

**DataFactory2**    
Pipeline 1: dataset4->activity4->dataset5

Si la propiedad externa está configurada correctamente, compruebe si los datos de entrada existen en la ubicación especificada en la definición del conjunto de datos de entrada.

### <a name="how-to-run-a-slice-at-another-time-than-midnight-when-the-slice-is-being-produced-daily"></a>¿Cómo se ejecuta un segmento en otro momento que no sea medianoche cuando este se produce diariamente?
Use la propiedad **offset** para especificar la hora en la que desea que se produzca el segmento. Consulte la sección [Disponibilidad del conjunto de datos](data-factory-create-datasets.md#dataset-availability) para obtener más información sobre esta propiedad. Este es un ejemplo rápido:

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
Los segmentos diarios comienzan a las **6:00** en lugar de a medianoche, que es el valor predeterminado.     

### <a name="how-can-i-rerun-a-slice"></a>¿Cómo puedo volver a ejecutar un segmento?
Puede volver a ejecutar un segmento de una de las siguientes maneras:

* Use la aplicación de supervisión y administración para volver a ejecutar una ventana de actividad o segmento. Consulte [Nueva ejecución de ventanas de actividad seleccionadas](data-factory-monitor-manage-app.md#perform-batch-actions) para ver instrucciones.   
* Haga clic en **Ejecutar** en la barra de comandos de la hoja **SEGMENTO DE DATOS** para el segmento en Azure Portal.
* Ejecute el cmdlet **Set-AzDataFactorySliceStatus** con el estado **En espera** para el segmento.   

    ```powershell
    Set-AzDataFactorySliceStatus -Status Waiting -ResourceGroupName $ResourceGroup -DataFactoryName $df -TableName $table -StartDateTime "02/26/2015 19:00:00" -EndDateTime "02/26/2015 20:00:00"
    ```
  Consulte [Set-AzDataFactorySliceStatus][set-azure-datafactory-slice-status] para más información sobre el cmdlet.

### <a name="how-long-did-it-take-to-process-a-slice"></a>¿Cuánto tiempo se tardó en procesar un segmento?
Utilice el explorador de ventana de actividad de la aplicación de supervisión y administración para descubrir cuánto tiempo se tardó en procesar un segmento de datos. Consulte [Explorador de ventanas de actividad](data-factory-monitor-manage-app.md#activity-window-explorer) para obtener más información.

También puede hacer lo siguiente en Azure Portal:  

1. Haga clic en la ventana **Conjuntos de datos** de la hoja **FACTORÍA DE DATOS** para su factoría de datos.
2. Haga clic en el conjunto de datos específico de la hoja **Conjuntos de datos** .
3. Seleccione el segmento en el que está interesado en la lista **Segmentos recientes** de la hoja **TABLA**.
4. Haga clic en la actividad ejecutada en la lista **Ejecuciones de actividad** de la hoja **SEGMENTO DE DATOS**.
5. Haga clic en la ventana **Propiedades** de la hoja **DETALLES DE EJECUCIÓN DE ACTIVIDAD**.
6. Debería ver el campo **DURACIÓN** con un valor. Este valor es el tiempo necesario para procesar el segmento.   

### <a name="how-to-stop-a-running-slice"></a>¿Cómo detener un segmento en ejecución?
Si debe impedir que se ejecute la canalización, puede usar el cmdlet [Suspend-AzDataFactoryPipeline](/powershell/module/az.datafactory/suspend-azdatafactorypipeline). Actualmente, la suspensión de la canalización no detiene las ejecuciones de segmentos en curso. Cuando terminan las ejecuciones en curso, no se selecciona ningún segmento adicional.

Si desea realmente detener todas las ejecuciones inmediatamente, la única manera sería eliminar la canalización y crearla de nuevo. Si decide eliminar la canalización, NO es necesario eliminar tablas y servicios vinculados usados por la canalización.

[create-factory-using-dotnet-sdk]: data-factory-create-data-factories-programmatically.md
[msdn-class-library-reference]: /dotnet/api/microsoft.azure.management.datafactories.models
[msdn-rest-api-reference]: /rest/api/datafactory/

[adf-powershell-reference]: /powershell/module/az.datafactory/
[azure-portal]: https://portal.azure.com
[set-azure-datafactory-slice-status]: /powershell/module/az.datafactory/set-Azdatafactoryslicestatus

[adf-pricing-details]: https://go.microsoft.com/fwlink/?LinkId=517777
[hdinsight-supported-regions]: https://azure.microsoft.com/pricing/details/hdinsight/
[hdinsight-alternate-storage]: https://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx
[hdinsight-alternate-storage-2]: /archive/blogs/cindygross/use-additional-storage-accounts-with-hdinsight-hive