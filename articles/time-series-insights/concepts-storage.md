---
title: Introducción a Storage - Azure Time Series Insights Gen2 | Microsoft Docs
description: Información acerca del almacenamiento de datos en Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 01/21/2021
ms.custom: seodec18
ms.openlocfilehash: 70a0ecb6e9ff2707401517e185964edf512a94c9
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204268"
---
# <a name="data-storage"></a>Almacenamiento de datos

En este artículo se describe el almacenamiento de datos en Azure Time Series Insights Gen2. Se explican el almacenamiento intermedio y en reposo, la disponibilidad de los datos y los procedimientos recomendados.

## <a name="provisioning"></a>Aprovisionamiento

Al crear un entorno de Azure Time Series Insights Gen2, tiene las siguientes opciones:

* Almacenamiento de datos en reposo:
  * Cree un nuevo recurso de Azure Storage en la suscripción y la región que ha elegido para el entorno.
  * Conecte una cuenta de Azure Storage existente. Esta opción solo está disponible si se implementa desde una [plantilla](/azure/templates/microsoft.timeseriesinsights/allversions) de Azure Resource Manager y no está visible en Azure Portal.
* Almacenamiento intermedio de datos:
  * El almacenamiento intermedio de datos es opcional y se puede habilitar o deshabilitar durante o después del momento del aprovisionamiento. Si decide habilitar el almacenamiento intermedio en un momento posterior y ya hay datos en el almacenamiento en reposo, consulte [esta](concepts-storage.md#warm-store-behavior) sección a continuación para comprender el comportamiento esperado. El tiempo de retención de datos del almacenamiento intermedio se puede configurar entre 7 y 31 días, y esto también se puede ajustar según sea necesario.

Cuando se ingiere un evento, se indexa en el almacenamiento intermedio (si está habilitado) y en el almacenamiento en reposo.

[![Información general sobre almacenamiento](media/concepts-storage/pipeline-to-storage.png)](media/concepts-storage/pipeline-to-storage.png#lightbox)

> [!WARNING]
> Como propietario de la cuenta de Azure Blob Storage donde residen los datos del almacenamiento intermedio, tiene acceso total a todos los datos de la cuenta. Este acceso incluye permisos de escritura y eliminación. No modifique ni elimine los datos que escribe Azure Time Series Insights Gen2, ya que estas operaciones pueden provocar una pérdida de datos.

## <a name="data-availability"></a>Disponibilidad de los datos

Azure Time Series Insights Gen2 crea particiones de los datos y los indexa para lograr un rendimiento óptimo de las consultas. Los datos están disponibles para realizar consultas tanto en almacenamiento parcialmente activo (si está habilitado) como en almacenamiento en reposo. La cantidad de datos que se ingieren y la tasa de rendimiento por partición pueden afectar a la disponibilidad. Consulte las [limitaciones de rendimiento](./concepts-streaming-ingress-throughput-limits.md) y los [procedimientos recomendados](./concepts-streaming-ingestion-event-sources.md#streaming-ingestion-best-practices) del origen de eventos para obtener el mejor rendimiento. También puede configurar una [alerta](./time-series-insights-environment-mitigate-latency.md#monitor-latency-and-throttling-with-alerts) de retraso para recibir una notificación si el entorno experimenta problemas al procesar los datos.

> [!IMPORTANT]
> Es posible que experimente un período de hasta 60 segundos hasta que los datos estén disponibles. Si experimenta una latencia considerable de más de 60 segundos, envíe una incidencia de soporte técnico a través de Azure Portal.

## <a name="warm-store"></a>Almacenamiento intermedio

Los datos del almacenamiento intermedio solo están disponibles mediante las [API de consulta de Time Series](./concepts-query-overview.md), el [explorador de Azure Time Series Insights TSI](./concepts-ux-panels.md) o el [conector de Power BI](./how-to-connect-power-bi.md). Las consultas al almacenamiento intermedio son gratuitas y no hay ninguna cuota, pero hay un [límite de 30](/rest/api/time-series-insights/reference-api-limits#query-apis---limits) solicitudes simultáneas.

### <a name="warm-store-behavior"></a>Comportamiento del almacenamiento intermedio

* Cuando está habilitado, todos los datos transmitidos al entorno se enrutarán al almacenamiento intermedio, independientemente de la marca de tiempo del evento. Tenga en cuenta que la canalización de ingesta de streaming está diseñada para el streaming casi en tiempo real y [no se admite](./concepts-streaming-ingestion-event-sources.md#historical-data-ingestion) la ingesta de eventos históricos.
* El período de retención se calcula en función del momento en el que el evento se indexa en el almacenamiento intermedio, no según la marca de tiempo del evento. Esto significa que los datos ya no están disponibles en el almacenamiento intermedio una vez transcurrido el período de retención, incluso si la marca de tiempo del evento es para el futuro.
  * Ejemplo: un evento con pronósticos meteorológicos de 10 días se ingiere y se indexa en un contenedor de almacenamiento intermedio configurado con un período de retención de 7 días. Después de siete días, el pronóstico no estará accesible en el almacenamiento intermedio, pero se puede consultar desde el almacenamiento en reposo.
* Si habilita el almacenamiento intermedio en un entorno existente que ya tiene datos recientes indexados en el almacenamiento en reposo, tenga en cuenta que el almacenamiento intermedio no se rellenará con estos datos.
* Si acaba de habilitar el almacenamiento intermedio y experimenta problemas al ver los datos recientes en el explorador, puede desactivar temporalmente las consultas del almacenamiento intermedio:

   [![Deshabilitar las consultas del almacenamiento intermedio](media/concepts-storage/toggle-warm.png)](media/concepts-storage/toggle-warm.png#lightbox)

## <a name="cold-store"></a>Almacenamiento en reposo

En esta sección se describen detalles de Azure Storage relativos a Azure Time Series Insights Gen2.

Para obtener una descripción detallada de Azure Blob Storage, lea [Introducción a Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md).

### <a name="your-cold-storage-account"></a>Cuenta de almacenamiento en reposo

Azure Time Series Insights Gen2 conserva un máximo de dos copias de cada evento en su cuenta de Azure Storage. Una copia almacena los eventos ordenados por hora de ingesta, permitiendo siempre el acceso a los eventos en una secuencia ordenada por tiempo. Con el tiempo, Azure Time Series Insights Gen2 también crea una copia con una nueva partición de los datos que se van a optimizar para las consultas eficaces.

Todos los datos se almacenan de forma indefinida en su cuenta de Azure Storage.

> [!WARNING]
> No restrinja el acceso a través de una red de Internet pública a la cuenta de almacenamiento usada por Time Series Insights o, de lo contrario, se interrumpirá la conexión necesaria.

#### <a name="writing-and-editing-blobs"></a>Escritura y edición de blobs

Para garantizar el rendimiento de las consultas y la disponibilidad de los datos, no edite ni elimine los blobs que crea Azure Time Series Insights Gen2.

#### <a name="accessing-cold-store-data"></a>Acceso a datos de almacenamiento en reposo

Además de acceder a los datos desde el [explorador de Azure Time Series Insights](./concepts-ux-panels.md) y las [API Time Series Query](./concepts-query-overview.md), puede que también desee acceder a los datos directamente desde los archivos Parquet almacenados en el almacenamiento en reposo. Por ejemplo, puede leer, transformar y limpiar los datos en un cuaderno de Jupyter y luego usarlo para entrenar el modelo de Azure Machine Learning en el mismo flujo de trabajo de Spark.

Para acceder a los datos directamente desde su cuenta de Azure Storage, necesita acceso de lectura a la cuenta usada para almacenar los datos de Azure Time Series Insights Gen2. A continuación, puede leer los datos seleccionados en función de la hora de creación del archivo de Parquet que se encuentra en la carpeta `PT=Time` que se describe a continuación en la sección [Formato de archivo de Parquet](#parquet-file-format-and-folder-structure).  Para más información sobre cómo habilitar el acceso de lectura a su cuenta de almacenamiento, consulte [Administración del acceso a los recursos de la cuenta de almacenamiento](../storage/blobs/anonymous-read-access-configure.md).

#### <a name="data-deletion"></a>Eliminación de datos

No elimine los archivos de Azure Time Series Insights Gen2. Administre los datos relacionados solo desde Azure Time Series Insights Gen2.

### <a name="parquet-file-format-and-folder-structure"></a>Formato de archivo de Parquet y estructura de carpetas

Parquet es un formato de archivo en columnas de código abierto diseñado para lograr un almacenamiento y un rendimiento eficaces. Azure Time Series Insights Gen2 usa Parquet para habilitar el rendimiento de las consultas basadas en el identificador de la serie temporal a escala.

Para más información sobre el tipo de archivo de Parquet, lea la [documentación de Parquet](https://parquet.apache.org/documentation/latest/).

Azure Time Series Insights Gen2 almacena copias de los datos de la siguiente manera:

* La primera copia inicial se particiona por hora de ingesta y almacena los datos más o menos en orden de llegada. Los datos residen en la carpeta `PT=Time`:

  `V=1/PT=Time/Y=<YYYY>/M=<MM>/<YYYYMMDDHHMMSSfff>_<TSI_INTERNAL_SUFFIX>.parquet`

* La segunda copia con particiones se agrupa por los identificadores de Time Series y reside en la carpeta `PT=TsId`:

  `V=1/PT=TsId/<TSI_INTERNAL_NAME>.parquet`

La marca de tiempo de los nombres de blob de la carpeta `PT=Time` corresponde a la hora de llegada de los datos a Azure Time Series Insights Gen2, no a la marca de tiempo de los eventos.

Los datos de la carpeta `PT=TsId` se optimizan para la consulta a lo largo del tiempo y no están estáticos. Durante la creación de particiones, es posible que algunos eventos estén presentes en varios blobs. No se garantiza que la nomenclatura de los blobs de esta carpeta siga siendo la misma.

En general, si necesita acceder a los datos directamente a través de los archivos de Parquet, use la carpeta `PT=Time`.  La futura funcionalidad permitirá el acceso eficaz a la carpeta `PT=TsId`.

> [!NOTE]
>
> * `<YYYY>` se asigna a una representación de año de cuatro dígitos.
> * `<MM>` se asigna a una representación de mes de dos dígitos.
> * `<YYYYMMDDHHMMSSfff>` se asigna a una representación de marca de tiempo con un año de cuatro dígitos (`YYYY`), un mes de dos dígitos (`MM`), un día de dos dígitos (`DD`), una hora de dos dígitos (`HH`), un minuto de dos dígitos (`MM`), un segundo de dos dígitos (`SS`) y un milisegundo de tres dígitos (`fff`).

Los eventos de Azure Time Series Insights Gen2 se asignan al contenido de los archivos de Parquet como se indica a continuación:

* Cada evento se asigna a una sola fila.
* Cada fila incluye la columna **timestamp** con una marca de tiempo del evento. La propiedad time-stamp nunca es null. Su valor predeterminado es **event enqueued time** si no se especifica en el origen del evento. La marca de tiempo almacenada siempre está en UTC.
* Cada fila incluye las columnas de identificadores de la serie temporal (TSID) como se define durante la creación del entorno de Azure Time Series Insights Gen2. El nombre de la propiedad del identificador de Time Series incluye el sufijo `_string`.
* Las demás propiedades enviadas como datos de telemetría se asignan a nombres de columna que terminan en `_bool` (booleano), `_datetime` (marca de tiempo), `_long` (largo) o `_double` (doble),`_string` (cadena) o `dynamic` (dinámico) en función del tipo de propiedad.  Para obtener más información, lea [Tipos de datos compatibles](./concepts-supported-data-types.md).
* Este esquema de asignación se aplica a la primera versión del formato de archivo, a la que se hace referencia como **V=1** y se almacena en la carpeta base del mismo nombre. A medida que esta característica evolucione, este esquema de asignación podría cambiar y se incrementaría el nombre de referencia.

## <a name="next-steps"></a>Pasos siguientes

* Lea sobre el [modelado de datos](./concepts-model-overview.md).

* Planeamiento del [entorno de Azure Time Series Insights Gen2](./how-to-plan-your-environment.md).
