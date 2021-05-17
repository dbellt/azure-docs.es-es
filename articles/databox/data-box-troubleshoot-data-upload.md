---
title: Revisión de los errores de copia en las cargas desde dispositivos Azure Data Box, Azure Data Box Heavy
description: Describe la revisión y el seguimiento de los errores que no se pueden reintentar y que impiden que los archivos se carguen desde un dispositivo Azure Data Box o Azure Data Box Heavy.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: pod
ms.topic: troubleshooting
ms.date: 05/10/2021
ms.author: alkohli
ms.openlocfilehash: 7ede9b952a27c8ed0df2c24598fc65187036e5dc
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790494"
---
# <a name="review-copy-errors-in-uploads-from-azure-data-box-and-azure-data-box-heavy-devices"></a>Revisión de los errores de copia en las cargas desde dispositivos Azure Data Box y Azure Data Box Heavy

En este artículo se describe la revisión y el seguimiento de los errores que no se pueden reintentar y que ocasionalmente impiden que los archivos se carguen en la nube desde un dispositivo Azure Data Box o Azure Data Box Heavy.

> [!NOTE]
> La información de este artículo se aplica solo a los pedidos de importación.

## <a name="upload-errors-notification"></a>Notificación de errores de carga

Cuando los datos se cargan en Azure desde el dispositivo, algunas cargas de archivos pueden producir ocasionalmente errores debido a errores de configuración que no se pueden resolver mediante un reintento. En ese caso, recibirá una notificación que le dará la oportunidad de revisar y corregir los errores para una carga posterior.

Verá la siguiente notificación en Azure Portal. Los errores aparecen en el registro de copia de datos, que se puede abrir mediante **DATA COPY PATH**. Para obtener instrucciones sobre cómo resolver los errores, vea [Resumen de errores de carga que no se pueden reintentar](#summary-of-non-retryable-upload-errors).

![Notificación de errores durante la carga](media/data-box-troubleshoot-data-upload/copy-completed-with-errors-notification-01.png)

No puede corregir estos errores. La carga se ha completado con errores. La notificación le permite conocer los problemas de configuración que debe corregir antes de probar otra carga mediante transferencia de red o un nuevo pedido de importación.

Después de revisar los errores y confirmar que está listo para continuar, los datos se borrarán del dispositivo de forma segura. Si no responde a la notificación, el pedido se completa automáticamente después de 14 días. Para obtener instrucciones paso a paso, vea [Revisar errores y continuar](#review-errors-and-proceed).


## <a name="review-errors-and-proceed"></a>Revisar errores y continuar

El pedido se completará automáticamente después de 14 días. Si actúa al recibir la notificación, puede avanzar más rápidamente.

[!INCLUDE [data-box-review-nonretryable-errors](../../includes/data-box-review-nonretryable-errors.md)]


## <a name="summary-of-non-retryable-upload-errors"></a>Resumen de errores de carga que no se pueden reintentar

Los siguientes errores que no se pueden reintentar generan una notificación:

|Categoría del error                    |Código de error |Mensaje de error                                                                             |
|----------------------------------|-----------|------------------------------------------------------------------------------------------|
|UploadErrorCloudHttp              |400        |Solicitud incorrecta (nombre de archivo no válido). [Más información.](#bad-request-file-name-not-valid)|
|UploadErrorCloudHttp              |400        |El valor de uno de los encabezados HTTP no tiene el formato correcto. [Más información](#the-value-for-one-of-the-http-headers-is-not-in-the-correct-format).|
|UploadErrorCloudHttp              |409        |Esta operación no se permite porque el blob es inmutable debido a una directiva. [Más información](#this-operation-is-not-permitted-as-the-blob-is-immutable-due-to-policy).|
|UploadErrorCloudHttp              |409        |La capacidad total aprovisionada de los recursos compartidos no puede superar el límite de tamaño máximo de la cuenta. [Más información](#the-total-provisioned-capacity-of-the-shares-cannot-exceed-the-account-maximum-size-limit).|
|UploadErrorCloudHttp              |409        |El tipo de blob no es válido para esta operación. [Más información](#the-blob-type-is-invalid-for-this-operation).|
|UploadErrorCloudHttp              |409        |Actualmente hay una concesión en el blob y no se especificó ningún identificador de concesión en la solicitud. [Más información](#there-is-currently-a-lease-on-the-blob-and-no-lease-id-was-specified-in-the-request).|
|UploadErrorManagedConversionError |409        |El tamaño del blob que se quiere importar no es válido. El tamaño del blob es de `<blob-size>` bytes. Los tamaños admitidos se encuentran entre 20971520 bytes y 8192 GiB. [Más información](#the-size-of-the-blob-being-imported-is-invalid-the-blob-size-is-blob-size-bytes-supported-sizes-are-between-20971520-bytes-and-8192-gib)|
<!--Temporarily removed from table: Bad Request (file property failure for Azure Files)-->

Para obtener más información sobre el contenido del registro de la copia de datos, vea [Seguimiento y registro de eventos para un pedido de importación de Azure Data Box y Azure Data Box Heavy](data-box-logs.md).

Pueden producirse otros errores de API de REST durante las cargas de datos. Para más información, consulte [Códigos de error comunes de la API de REST](/rest/api/storageservices/common-rest-api-error-codes).

> [!NOTE]
> En las secciones **Seguimiento** de las descripciones de errores se describe cómo actualizar la configuración de datos antes de realizar un nuevo pedido de importación o realizar una transferencia de red. No se pueden corregir estos errores en la carga actual.


### <a name="bad-request-file-name-not-valid"></a>Solicitud incorrecta (nombre de archivo no válido)

**Categoría del error:** UploadErrorCloudHttp 

**Código de error:** 400

**Descripción del error:** la mayoría de los problemas de nomenclatura de archivos se detectan durante la fase **Preparar para enviar** o se corrigen automáticamente durante la carga (lo que da lugar a un estado **Copia con advertencia**). Cuando no se detecta un nombre de archivo no válido, el archivo no se carga en Azure.

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido, cambie el nombre de los archivos indicados para que cumplan con los requisitos de nomenclatura de Azure Files. Para los requisitos de nomenclatura, vea [Nombres de directorio y archivo](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).


<!--TEMPORARILY REMOVED. Product team may restore later. ### Bad Request (File property failure for Azure Files)

**Error category:** UploadErrorCloudHttp 

**Error code:** 400

**Error description:** Data import will fail if the upload of file properties fails for Azure Files.  

**Follow-up:** You can't fix this error in the current upload. The upload will complete with errors. Before you do a network transfer or start a new import order, *GET TROUBLESHOOTING*.-->


### <a name="the-value-for-one-of-the-http-headers-is-not-in-the-correct-format"></a>El valor de uno de los encabezados HTTP no tiene el formato correcto

**Categoría del error:** UploadErrorCloudHttp 

**Código de error:** 400

**Descripción del error:** no se pudieron cargar los blobs indicados porque no cumplen los requisitos de formato o tamaño de los blobs de Azure Storage.

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido de importación, asegúrese de que:

- Los blobs en páginas indicados se alinean con los límites de página de 512 bytes.

- Los blobs en bloques indicados no superan el tamaño máximo de 4,75 TiB.


### <a name="this-operation-is-not-permitted-as-the-blob-is-immutable-due-to-policy"></a>Esta operación no se permite porque el blob es inmutable debido a una directiva

**Categoría del error:** UploadErrorCloudHttp 

**Código de error:** 409

**Descripción del error:** si un contenedor de Blob Storage está configurado como Escribir una vez, leer muchas (WORM), se producirá un error en la carga de los blobs que ya están almacenados en el contenedor.

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido de importación, asegúrese de que los blobs indicados no forman parte de un contenedor de almacenamiento inmutable. Para más información, consulte [Almacenamiento de datos de blobs críticos para la empresa con almacenamiento inmutable](/azure/storage/blobs/storage-blob-immutable-storage).


### <a name="the-total-provisioned-capacity-of-the-shares-cannot-exceed-the-account-maximum-size-limit"></a>La capacidad total aprovisionada de los recursos compartidos no puede superar el límite de tamaño máximo de la cuenta.

**Categoría del error:** UploadErrorCloudHttp 

**Código de error:** 409

**Descripción del error:** se ha producido un error en la carga porque el tamaño total de los datos supera el límite de tamaño de la cuenta de almacenamiento. Por ejemplo, la capacidad máxima de una cuenta FileStorage es de 100 TiB. Si el tamaño total de los datos supera los 100 TiB, se producirá un error en la carga.  

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido de importación, asegúrese de que la capacidad total de todos los recursos compartidos de la cuenta de almacenamiento no supere el límite de tamaño de la cuenta de almacenamiento. Para más información, vea [Límites de tamaño de cuenta de almacenamiento de Azure](data-box-limits.md#azure-storage-account-size-limits).


### <a name="the-blob-type-is-invalid-for-this-operation"></a>El tipo de blob no es válido para esta operación

**Categoría del error:** UploadErrorCloudHttp 

**Código de error:** 409

**Descripción del error:** se producirá un error en la importación de datos a un blob en la nube si se modifican los datos o las propiedades del blob de destino.

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido de importación, asegúrese de que no haya ninguna modificación simultánea de los blobs indicados ni de sus propiedades durante la carga.

### <a name="there-is-currently-a-lease-on-the-blob-and-no-lease-id-was-specified-in-the-request"></a>Actualmente hay una concesión del blob y no se especificó ningún id. de concesión en la solicitud

**Categoría del error:** UploadErrorCloudHttp 

**Código de error:** 409

**Descripción del error:** la importación de datos a un blob en la nube producirá un error si el blob de destino tiene una concesión activa.

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido de importación, asegúrese de que los blobs enumerados no tienen una concesión activa. Para más información, vea [Simultaneidad pesimista para blobs](/azure/storage/blobs/concurrency-manage?tabs=dotnet#pessimistic-concurrency-for-blobs).


### <a name="the-size-of-the-blob-being-imported-is-invalid-the-blob-size-is-blob-size-bytes-supported-sizes-are-between-20971520-bytes-and-8192-gib"></a>El tamaño del blob que se quiere importar no es válido. El tamaño del blob es de `<blob-size>` bytes. Los tamaños admitidos se encuentran entre 20971520 bytes y 8192 GiB.

**Categoría del error:** UploadErrorManagedConversionError

**Código de error:** 409

**Descripción del error:** los blobs en páginas indicados no se pudieron cargar porque no tienen un tamaño que se pueda convertir en un disco administrado. Para convertirse en un disco administrado, un blob en páginas debe tener un tamaño de 20 MB (20 971 520 bytes) a 8192 GiB.

**Seguimiento:** no se puede corregir este error en la carga actual. La carga se ha completado con errores. Antes de realizar una transferencia de red o iniciar un nuevo pedido de importación, asegúrese de que cada blob indicado tiene un tamaño de 20 MB a 8192 GiB.


## <a name="next-steps"></a>Pasos siguientes

- [Revisión de los errores comunes de la API de REST](/rest/api/storageservices/common-rest-api-error-codes)
- [Comprobación de la carga de datos en Azure](data-box-deploy-picked-up.md?tabs=in-us-canada-europe#verify-data-upload-to-azure-8)
