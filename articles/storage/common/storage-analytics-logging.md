---
title: Registro de Azure Storage Analytics
description: Use Storage Analytics para registrar los detalles acerca de las solicitudes de Azure Storage. Vea qué solicitudes se registran, cómo se almacenan los registros, cómo habilitar el registro de almacenamiento, etc.
author: normesta
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.date: 01/29/2021
ms.author: normesta
ms.reviewer: fryu
ms.custom: monitoring, devx-track-csharp
ms.openlocfilehash: d48ab6223485807400b6749bcf72691261405495
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108134562"
---
# <a name="azure-storage-analytics-logging"></a>Registro de Azure Storage Analytics

Storage Analytics registra información detallada sobre las solicitudes correctas y erróneas realizadas a un servicio de almacenamiento. Esta información se puede utilizar para supervisar solicitudes concretas y para diagnosticar problemas con un servicio de almacenamiento. Las solicitudes se registran en función de la mejor opción. Esto significa que la mayoría de las solicitudes darán lugar a una entrada de registro, pero no se garantiza la integridad y la escala de tiempo de los registros de Storage Analytics. 

> [!NOTE]
> Se recomienda usar los registros de Azure Storage en Azure Monitor en lugar de los registros de Storage Analytics. Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (que incluye Azure Data Lake Storage Gen2), archivos, colas y tablas. Para más información, consulte cualquiera de los siguientes artículos:
>
> - [Supervisión de Azure Blob Storage](../blobs/monitor-blob-storage.md)
> - [Supervisión de Azure Files](../files/storage-files-monitoring.md)
> - [Supervisión de Azure Queue Storage](../queues/monitor-queue-storage.md)
> - [Supervisión de Azure Table Storage](../tables/monitor-table-storage.md)

 El registro de Storage Analytics no está habilitado de forma predeterminada en la cuenta de almacenamiento. Puede habilitarlo en [Azure Portal](https://portal.azure.com/) o mediante PowerShell o la CLI de Azure. Para una guía paso a paso, consulte [Habilitación y administración de los registros de Azure Storage Analytics (clásico)](manage-storage-analytics-logs.md). 

También puede habilitar los registros de Storage Analytics mediante programación a través de la API de REST o la biblioteca cliente. Use las operaciones [Obtener las propiedades de Blob service](/rest/api/storageservices/Blob-Service-REST-API), [Obtener las propiedades de Queue service](/rest/api/storageservices/Get-Queue-Service-Properties) y [Obtener las propiedades de Table service](/rest/api/storageservices/Get-Table-Service-Properties) para habilitar Storage Analytics en cada servicio. Para ver un ejemplo que habilita los registros de Storage Analytics mediante .NET, consulte [Habilitación de registros](manage-storage-analytics-logs.md).

 Las entradas del registro se crean solo si se presentan solicitudes al punto de conexión de servicio. Por ejemplo, si una cuenta de almacenamiento tiene actividad en el punto de conexión de Blob service, pero no en los puntos de conexión de Table service o Queue service, solo se crearán los registros correspondientes a Blob service.

> [!NOTE]
>  Actualmente, el registro de Storage Analytics está disponible solo para los servicios Blob, Queue y Table service. El registro de Storage Analytics también está disponible para las cuentas [BlockBlobStorage](./storage-account-create.md) de rendimiento Premium. Sin embargo, no está disponible para las cuentas de uso general v2 con rendimiento Premium.

## <a name="requests-logged-in-logging"></a>Solicitudes registradas en el registro
### <a name="logging-authenticated-requests"></a>Registrar solicitudes de autenticación

 Se registran los siguientes tipos de solicitudes autenticadas:

- Solicitudes correctas
- Solicitudes erróneas, incluidos errores de tiempo de espera, de limitación, de red, de autorización y de otro tipo
- Solicitudes que usan una firma de acceso compartido (SAS) u OAuth, tanto las correctas como las erróneas
- Solicitudes de datos de análisis

  Las solicitudes realizadas por el propio Storage Analytics, como la creación o eliminación del registro, no se registran. Puede encontrar una lista completa de los datos registrados en los temas [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).

### <a name="logging-anonymous-requests"></a>Registrar solicitudes anónimas

 Se registran los siguientes tipos de solicitudes anónimas:

- Solicitudes correctas
- Errores del servidor
- Errores de tiempo de espera de cliente y servidor
- Solicitudes GET erróneas con el código de error 304 (No modificado)

  El resto de solicitudes anónimas erróneas no se registran. Puede encontrar una lista completa de los datos registrados en los temas [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).

## <a name="how-logs-are-stored"></a>Cómo se almacenan los registros

Todos los registros se almacenan en blobs en bloques, en un contenedor denominado `$logs`, que se crea automáticamente cuando Storage Analytics se habilita en una cuenta de almacenamiento. El contenedor `$logs` se encuentra en el espacio de nombres del blob de la cuenta de almacenamiento, por ejemplo: `http://<accountname>.blob.core.windows.net/$logs`. Este contenedor no se puede eliminar una vez habilitado Storage Analytics, aunque su contenido sí se puede eliminar. Si usa la herramienta de exploración del almacenamiento para ir directamente al contenedor, verá todos los blobs que contienen los datos de registro.

> [!NOTE]
>  El contenedor `$logs` no se muestra cuando se realiza una operación para enumerar los contenedores, como la operación Lista de contenedores. Se debe tener acceso a él directamente. Por ejemplo, puede utilizar la operación Mostrar blobs para acceder a los blobs en el contenedor `$logs`.

A medida que se registran las solicitudes, Storage Analytics cargará los resultados intermedios como bloques. Periódicamente, Storage Analytics confirmará estos bloques para que estén disponibles como blobs. Hasta que los datos de registro aparezcan en los blobs del contenedor **$logs**, puede transcurrir hasta una hora, dada la frecuencia con la que el servicio de almacenamiento vacía la escritura de registros. Es posible que haya entradas duplicadas en los registros creados durante la misma hora. Puede determinar si un registro es un duplicado comprobando el número **RequestId** y **Operation**.

Si tiene un gran volumen de datos de registro con varios archivos correspondientes a cada hora, puede usar los metadatos del blob para averiguar qué datos contiene el registro (examinando para ello los campos de metadatos del blob). Esto resulta útil también porque, a veces, puede haber un retraso mientras los datos se escriben en los archivos de registro, y es que los metadatos del blob indican el contenido del blob de forma más precisa que el nombre del blob.

La mayoría de las herramientas de exploración del almacenamiento permiten ver los metadatos de los blobs, si bien esta información se puede consultar también con PowerShell o mediante programación. El siguiente fragmento de código de PowerShell es un ejemplo de filtrado de la lista de blobs de registro por nombre para especificar una hora, y por metadatos para identificar únicamente aquellos registros que contengan operaciones de escritura (**write**).  

 ```powershell
 Get-AzStorageBlob -Container '$logs' |  
 Where-Object {  
     $_.Name -match 'blob/2014/05/21/05' -and   
     $_.ICloudBlob.Metadata.LogType -match 'write'  
 } |  
 ForEach-Object {  
     "{0}  {1}  {2}  {3}" –f $_.Name,   
     $_.ICloudBlob.Metadata.StartTime,   
     $_.ICloudBlob.Metadata.EndTime,   
     $_.ICloudBlob.Metadata.LogType  
 }  
 ```  

Para obtener información sobre cómo mostrar los blobs mediante programación, vea [Enumerar recursos de blob](/rest/api/storageservices/Enumerating-Blob-Resources) y [Configurar y recuperar propiedades y metadatos de recursos de blob](/rest/api/storageservices/Setting-and-Retrieving-Properties-and-Metadata-for-Blob-Resources).  

### <a name="log-naming-conventions"></a>Convenciones de nomenclatura de los registros

 Cada registro se escribirá con el siguiente formato:

 `<service-name>/YYYY/MM/DD/hhmm/<counter>.log`

 En la siguiente tabla se describe cada atributo del nombre del registro:

|Atributo|Descripción|
|---------------|-----------------|
|`<service-name>`|El nombre del servicio de almacenamiento. Por ejemplo: `blob`, `table` o `queue`|
|`YYYY`|El año de cuatro dígitos del registro. Por ejemplo: `2011`|
|`MM`|El mes de dos dígitos del registro. Por ejemplo: `07`|
|`DD`|El día de dos dígitos del registro. Por ejemplo: `31`|
|`hh`|La hora de dos dígitos que indica la hora inicial de los registros, en formato UTC de 24 horas. Por ejemplo: `18`|
|`mm`|El número de dos dígitos que indica el minuto inicial de los registros. **Nota:**  Este valor no se admite en la versión actual de Storage Analytics, y su valor será siempre `00`.|
|`<counter>`|Un contador basado en cero con seis dígitos que indica el número de blobs del registro generados para el servicio de almacenamiento en un período de tiempo de una hora. Este contador se inicia en `000000`. Por ejemplo: `000001`|

 El siguiente es un nombre completo de registro de ejemplo que combina los ejemplos anteriores:

 `blob/2011/07/31/1800/000001.log`

 El siguiente es un URI de ejemplo que se puede utilizar para obtener acceso al registro anterior:

 `https://<accountname>.blob.core.windows.net/$logs/blob/2011/07/31/1800/000001.log`

 Cuando se registra una solicitud de almacenamiento, el nombre del registro resultante depende de la hora en la que se completó la operación solicitada. Por ejemplo, si una solicitud para obtener un blob finalizó a las 6:30 P.M. del 31/7/2011, el registro se escribirá con el siguiente prefijo: `blob/2011/07/31/1800/`

### <a name="log-metadata"></a>Metadatos del registro

 Todos los blobs del registro se almacenan con metadatos que se pueden utilizar para identificar los datos de registro que contiene el blob. En la siguiente tabla se describe cada atributo de metadatos:

|Atributo|Descripción|
|---------------|-----------------|
|`LogType`|Describe si el registro contiene información relacionada con las operaciones de lectura, escritura o eliminación. Este valor puede incluir un tipo o una combinación de los tres, separados por comas.<br /><br /> Ejemplo 1: `write`<br /><br /> Ejemplo 2: `read,write`<br /><br /> Ejemplo 3: `read,write,delete`|
|`StartTime`|La primera hora de una entrada en el registro, en formato `YYYY-MM-DDThh:mm:ssZ`. Por ejemplo: `2011-07-31T18:21:46Z`|
|`EndTime`|La última hora de una entrada en el registro, en formato `YYYY-MM-DDThh:mm:ssZ`. Por ejemplo: `2011-07-31T18:22:09Z`|
|`LogVersion`|La versión del formato del registro.|

 En la siguiente lista se muestran los metadatos de ejemplo completos utilizando los ejemplos anteriores:

-   `LogType=write`
-   `StartTime=2011-07-31T18:21:46Z`
-   `EndTime=2011-07-31T18:22:09Z`
-   `LogVersion=1.0`


## <a name="next-steps"></a>Pasos siguientes

* [Habilitación y administración de los registros de Azure Storage Analytics (clásico)](manage-storage-analytics-logs.md)
* [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format)
* [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)
* [Métricas de Storage Analytics (clásica)](storage-analytics-metrics.md)