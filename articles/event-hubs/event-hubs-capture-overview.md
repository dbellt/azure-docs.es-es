---
title: 'Captura de eventos de streaming: Azure Event Hubs | Microsoft Docs'
description: En este artículo se proporciona información general sobre la característica Capture que permite capturar eventos de streaming a través de Azure Event Hubs.
ms.topic: article
ms.date: 02/16/2021
ms.openlocfilehash: d3153c4d82cfbdf232d1834db8f26462f893961b
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375100"
---
# <a name="capture-events-through-azure-event-hubs-in-azure-blob-storage-or-azure-data-lake-storage"></a>Captura de eventos a través de Azure Event Hubs en Azure Blob Storage o Azure Data Lake Storage
Azure Event Hubs permite capturar automáticamente los datos de streaming de Event Hubs de la cuenta de [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/) o de [Azure Data Lake Storage Gen1 o Gen2](https://azure.microsoft.com/services/data-lake-store/) que prefiera, con la flexibilidad adicional de especificar un intervalo de tiempo o de tamaño. La configuración de Capture es rápida, su ejecución no conlleva ningún gasto administrativo y se escala automáticamente con [unidades de rendimiento](event-hubs-scalability.md#throughput-units) de Event Hubs en el nivel estándar o [unidades de procesamiento](event-hubs-scalability.md#processing-units) en el nivel Premium. El uso de Event Hubs Capture constituye la forma más sencilla de cargar datos de streaming en Azure y permite centrarse en el procesamiento de datos, en lugar de en su captura.

> [!NOTE]
> La configuración de Event Hubs Capture para usar Azure Data Lake Storage **Gen2** es la misma que la configuración para utilizar una instancia de Azure Blob Storage. Para obtener información, consulte [Configuración de Event Hubs Capture](event-hubs-capture-enable-through-portal.md). 

Event Hubs Capture permite procesar las canalizaciones en tiempo real y las basadas en lotes en la misma transmisión, lo que permite crear soluciones que crecen a la par que sus necesidades. Si ya está creando sistemas basados en lotes pensando en un futuro procesamiento en tiempo real o desea agregar una ruta de acceso inactiva eficaz a una solución en tiempo real existente, Event Hubs Capture facilita el trabajo con datos de streaming.

> [!IMPORTANT]
> La cuenta de almacenamiento de destino (Azure Storage o Azure Data Lake Storage) debe estar en la misma suscripción que el centro de eventos. 

## <a name="how-event-hubs-capture-works"></a>Cómo funciona Event Hubs Capture

Event Hubs es un búfer duradero de retención temporal para la entrada de datos de telemetría, es similar a un registro distribuido. La clave para reducir horizontalmente en Event Hubs es el [modelo de consumidor con particiones](event-hubs-scalability.md#partitions). Cada partición es un segmento de datos independiente y se consume de forma independiente. Con el tiempo estos datos envejecen basándose en el período de retención configurable. Como consecuencia, un centro de eventos determinado nunca llega a estar "demasiado lleno".

Event Hubs Capture permite especificar una cuenta y un contenedor propios de Azure Blob Storage, o una cuenta de Azure Data Lake Storage, que se usan para almacenar los datos capturados. Estas cuentas pueden estar en la misma región que el centro de eventos o en otra, lo que se suma a la flexibilidad de la característica Event Hubs Capture.

Los datos capturados se escriben en el formato de [Apache Avro][Apache Avro], que es un formato compacto, rápido y binario que proporciona estructuras de datos enriquecidos con un esquema insertado. Este formato se usa ampliamente en el ecosistema de Hadoop, en Stream Analytics y en Azure Data Factory. En este mismo artículo encontrará más información acerca de cómo trabajar con Avro.

### <a name="capture-windowing"></a>Ventanas de captura

Event Hubs Capture permite configurar una ventana para controlar la captura. Esta ventana es una configuración mínima de tamaño y tiempo con una "directiva de que el primero gana", lo que significa que el primer desencadenador que se encuentre provocará una operación de captura. Si tiene una ventana de captura de 100 MB cada quince minutos y envía 1 MB/s, la ventana de tamaño se desencadenará antes que la ventana de tiempo. Cada partición se captura de forma independiente y escribe un blob en bloques completado en el momento de la captura, denominado como la hora en que se encontró el intervalo de captura. Esta es la convención de nomenclatura de almacenamiento:

```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

Tenga en cuenta que los valores de fecha se rellenan con ceros. Un ejemplo de nombre de archivo podría ser el siguiente:

```
https://mystorageaccount.blob.core.windows.net/mycontainer/mynamespace/myeventhub/0/2017/12/08/03/03/17.avro
```

En caso de que Azure Storage Blob está temporalmente no disponible, Event Hubs Capture retendrá los datos durante el período de retención de datos configurado en el centro de eventos y volverá a rellenar los datos una vez que la cuenta de almacenamiento esté disponible nuevamente.

### <a name="scaling-throughput-units-or-processing-units"></a>Escalado de unidades de rendimiento o unidades de procesamiento

En el nivel estándar de Event Hubs, el tráfico se controla mediante [unidades de rendimiento](event-hubs-scalability.md#throughput-units) y, en el nivel Premium de Event Hubs, se controla mediante [unidades de procesamiento](event-hubs-scalability.md#processing-units). Event Hubs Capture copia los datos directamente del almacenamiento interno de Event Hubs, omite las cuotas de salida de unidades de rendimiento o procesamiento y guarda la salida para otros lectores de procesamiento, como Stream Analytics o Spark.

Una vez configurado, Event Hubs Capture se ejecuta automáticamente cuando se envía el primer evento y continúa ejecutándose. Para facilitar que el procesamiento de bajada sepa que el proceso funciona, Event Hubs escribe archivos vacíos cuando no hay datos. Este proceso proporciona un marcador y una cadencia predecibles que puede alimentar sus procesadores de lotes.

## <a name="setting-up-event-hubs-capture"></a>Configuración de Event Hubs Capture

Puede configurar la funcionalidad de captura en el momento de creación del centro de eventos mediante [Azure Portal](https://portal.azure.com) o con las plantillas de Azure Resource Manager. Para más información, consulte los siguientes artículos.

- [Habilitación de Event Hubs Capture mediante Azure Portal](event-hubs-capture-enable-through-portal.md)
- [Creación de un espacio de nombres de Event Hubs con un centro de eventos y habilitación de la característica Capture mediante una plantilla de Azure Resource Manager](event-hubs-resource-manager-namespace-event-hub-enable-capture.md)

> [!NOTE]
> Si habilita la característica de captura en un centro de eventos existente, la característica capturará los eventos que lleguen al centro de eventos **después** de activar la característica en cuestión. No capturará aquellos eventos que ya existían en el centro de eventos antes de que se activara. 

## <a name="exploring-the-captured-files-and-working-with-avro"></a>Exploración de los archivos capturados y trabajo con Avro

Event Hubs Capture crea archivos en formato Avro, como se especifica en la ventana de tiempo configurada. Dichos archivos se pueden ver en cualquier herramienta, como el [Explorador de Azure Storage][Azure Storage Explorer]. Los archivos se pueden descargar de forma local para trabajar con ellos.

Los archivos que genera Event Hubs Capture tienen el siguiente esquema de Avro:

![Esquema de Avro][3]

Para explorar los archivos de Avro fácilmente, utilice el archivo jar [Avro Tools][Avro Tools] desde Apache. También puede usar [Apache Drill][Apache Drill] para una experiencia sencilla con SQL o [Apache Spark][Apache Spark] para realizar un procesamiento distribuido complejo en los datos ingeridos. 

### <a name="use-apache-drill"></a>Uso de Apache Drill

[Apache Drill][Apache Drill] es un "motor de consulta SQL de código abierto para la exploración de macrodatos" que puede consultar datos estructurados y semiestructurados. El motor se puede ejecutar como un nodo independiente o como un clúster grande para lograr un rendimiento óptimo.

Hay disponible una compatibilidad nativa con Azure Blob Storage, lo cual facilita la consulta de datos en un archivo de Avro, como se describe en esta documentación:

[Apache Drill: Complemento de Azure Blob Storage][Apache Drill: Azure Blob Storage Plugin]

Para consultar fácilmente archivos capturados, puede crear y ejecutar una máquina virtual con Apache Drill habilitado mediante un contenedor para acceder a Azure Blob Storage. Vea el ejemplo siguiente: [Streaming a escala con Event Hubs Capture](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubs-capture).

### <a name="use-apache-spark"></a>Uso de Apache Spark

[Apache Spark][Apache Spark] es un "motor de análisis unificado para el procesamiento de datos a gran escala". Admite diferentes idiomas, incluido SQL, y puede acceder fácilmente a Azure Blob Storage. Hay varias opciones para ejecutar Apache Spark en Azure, y todas proporcionan un acceso fácil a Azure Blob Storage:

- [HDInsight: Archivos adicionales en Azure Storage][HDInsight: Address files in Azure storage]
- [Azure Databricks: Azure Blob Storage][Azure Databricks: Azure Blob Storage]
- [Azure Kubernetes Service](../aks/spark-job.md) 

### <a name="use-avro-tools"></a>Uso de Avro Tools

Las herramientas de [Avro Tools][Avro Tools] están disponibles como un paquete jar. Tras descargar este archivo, para ver el esquema de un archivo específico de Avro, ejecute el comando siguiente:

```shell
java -jar avro-tools-1.9.1.jar getschema <name of capture file>
```

Este comando devuelve

```json
{

    "type":"record",
    "name":"EventData",
    "namespace":"Microsoft.ServiceBus.Messaging",
    "fields":[
                 {"name":"SequenceNumber","type":"long"},
                 {"name":"Offset","type":"string"},
                 {"name":"EnqueuedTimeUtc","type":"string"},
                 {"name":"SystemProperties","type":{"type":"map","values":["long","double","string","bytes"]}},
                 {"name":"Properties","type":{"type":"map","values":["long","double","string","bytes"]}},
                 {"name":"Body","type":["null","bytes"]}
             ]
}
```

Avro Tools también puede utilizarse para convertir el archivo al formato JSON y realizar otro procesamiento.

Para realizar un procesamiento más avanzado, descargue e instale Avro para la plataforma que desee. En el momento de redactar este artículo, existen implementaciones para C, C++, C\#, Java, NodeJS, Perl, PHP, Python y Ruby.

Apache Avro tiene guías de introducción para [Java][Java] y [Python][Python] muy completas. También puede leer el artículo [Getting started with Event Hubs Capture (Introducción a Event Hubs Capture)](event-hubs-capture-python.md).

## <a name="how-event-hubs-capture-is-charged"></a>Cómo se factura Event Hubs Capture

Event Hubs Capture se mide de forma similar a las [unidades de rendimiento](event-hubs-scalability.md#throughput-units) (nivel estándar) o las [unidades de procesamiento](event-hubs-scalability.md#processing-units) (nivel Premium): como un cargo por hora. El cargo es directamente proporcional al número de unidades de rendimiento o procesamiento adquiridas para el espacio de nombres. A medida que las unidades de rendimiento o procesamiento se incrementan y reducen, las mediciones de Event Hubs Capture aumentan y disminuyen con el fin de proporcionar un rendimiento coincidente. Los medidores se ejecutan de manera simultánea. Para conocer los precios detallados, consulte [Precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/). 

Capture no consume la cuota de salida, porque se factura por separado. 

## <a name="integration-with-event-grid"></a>Integración con Event Grid 

Puede crear una suscripción de Azure Event Grid con un espacio de nombres de Event Hubs como origen. En el tutorial siguiente se muestra cómo crear una suscripción de Event Grid con un centro de eventos como origen y una aplicación de Azure Functions como receptor: [Procesamiento y migración de datos capturados de Event Hubs a Azure Synapse Analytics mediante Event Grid y Azure Functions](store-captured-data-data-warehouse.md).

## <a name="next-steps"></a>Pasos siguientes
Event Hubs Capture es el modo más sencillo de obtener datos en Azure. Con Azure Data Lake, Azure Data Factory y Azure HDInsight, se puede realizar el procesamiento por lotes y cualquier otro análisis mediante las plataformas y herramientas conocidas a la escala que necesite.

Aprenda a habilitar esta característica mediante Azure Portal y una plantilla de Azure Resource Manager:

- [Uso de Azure Portal para habilitar la captura de Event Hubs](event-hubs-capture-enable-through-portal.md)
- [Uso de una plantilla de Azure Resource Manager para habilitar Event Hubs Capture](event-hubs-resource-manager-namespace-event-hub-enable-capture.md)


[Apache Avro]: https://avro.apache.org/
[Apache Drill]: https://drill.apache.org/
[Apache Spark]: https://spark.apache.org/
[support request]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[Azure Storage Explorer]: https://github.com/microsoft/AzureStorageExplorer/releases
[3]: ./media/event-hubs-capture-overview/event-hubs-capture3.png
[Avro Tools]: https://downloads.apache.org/avro/stable/java/
[Java]: https://avro.apache.org/docs/current/gettingstartedjava.html
[Python]: https://avro.apache.org/docs/current/gettingstartedpython.html
[Event Hubs overview]: ./event-hubs-about.md
[HDInsight: Address files in Azure storage]: ../hdinsight/hdinsight-hadoop-use-blob-storage.md
[Azure Databricks: Azure Blob Storage]:https://docs.databricks.com/spark/latest/data-sources/azure/azure-storage.html
[Apache Drill: Azure Blob Storage Plugin]:https://drill.apache.org/docs/azure-blob-storage-plugin/
[Streaming at Scale: Event Hubs Capture]:https://github.com/yorek/streaming-at-scale/tree/master/event-hubs-capture
