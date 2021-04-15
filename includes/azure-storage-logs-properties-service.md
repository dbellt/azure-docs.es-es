---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: 9b18687c0a6f3e48d94431e88be8ae8137c9dcdb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96011151"
---
| Propiedad | Descripción |
|:--- |:---|
|**accountName** | El nombre de la cuenta de almacenamiento. Por ejemplo: `mystorageaccount`.  |
|**requestUrl** | Dirección URL que se solicita. |
|**userAgentHeader** | Valor del encabezado **User-Agent**, entre comillas. Por ejemplo: `WA-Storage/6.2.0 (.NET CLR 4.0.30319.42000; Win32NT 6.2.9200.0)`.|
|**referrerHeader** | Valor del encabezado **Referrer**. Por ejemplo: `http://contoso.com/about.html`.|
|**clientRequestId** | Valor de encabezado **x-ms-client-request-id** de la solicitud. Por ejemplo: `360b66a6-ad4f-4c4a-84a4-0ad7cb44f7a6`. |
|**etag** | Identificador de la ETag del objeto devuelto, entre comillas. Por ejemplo: `0x8D101F7E4B662C4`.  |
|**serverLatencyMs** | Tiempo total, expresado en milisegundos, necesario para realizar la operación solicitada. Este valor no incluye la latencia de red (el tiempo necesario para leer la solicitud entrante y enviar la respuesta al solicitante). Por ejemplo: `22`. |
|**serviceType** | Servicio asociado a esta solicitud. Por ejemplo: `blob`, `table`, `files` o `queue`. |
|**operationCount** | Número de cada operación registrada implicada en la solicitud. Este recuento comienza con un índice de `0`. Algunas solicitudes requieren más de una operación. La mayoría de las solicitudes solo realizan una operación. Por ejemplo: `1`. |
|**requestHeaderSize** | Tamaño del encabezado de la solicitud expresado en bytes. Por ejemplo: `578`. <br>Si una solicitud no se realiza correctamente, este valor puede estar vacío. |
|**requestBodySize** | Tamaño expresado en bytes de los paquetes de la solicitud leídos por el servicio de almacenamiento. <br> Por ejemplo: `0`. <br>Si una solicitud no se realiza correctamente, este valor puede estar vacío.  |
|**responseHeaderSize** | Tamaño del encabezado de la respuesta expresado en bytes. Por ejemplo: `216`. <br>Si una solicitud no se realiza correctamente, este valor puede estar vacío.  |
|**responseBodySize** | Tamaño de los paquetes de respuesta escritos por el servicio de almacenamiento, en bytes. Si una solicitud no se realiza correctamente, este valor puede estar vacío. Por ejemplo: `216`.  |
|**requestMd5** | Valor del encabezado **Content-MD5** o **x-ms-content-md5** de la solicitud. El valor del hash MD5 especificado en este campo representa el contenido de la solicitud. Por ejemplo: `788815fd0198be0d275ad329cafd1830`. <br>Este campo puede estar vacío.  |
|**serverMd5** | Valor del hash MD5 calculado por el servicio de almacenamiento. Por ejemplo: `3228b3cf1069a5489b298446321f8521`. <br>Este campo puede estar vacío.  |
|**lastModifiedTime** | Hora de la última modificación del objeto devuelto.  Por ejemplo: `Tuesday, 09-Aug-11 21:13:26 GMT`. <br>Este campo está vacío en el caso de operaciones que pueden devolver varios objetos. |
|**conditionsUsed** | Lista separada por punto y coma de pares clave-valor que representan una condición. Las condiciones pueden ser cualquiera de las siguientes: <li> If-Modified-Since <li> If-Unmodified-Since <li> If-Match <li> If-None-Match  <br> Por ejemplo: `If-Modified-Since=Friday, 05-Aug-11 19:11:54 GMT`. |
|**contentLengthHeader** | Valor del encabezado Content-Length de la solicitud enviada al servicio de almacenamiento. Si la solicitud se ha realizado correctamente, este valor es igual a requestBodySize. Si la solicitud no se ha realizado correctamente, este valor puede no ser igual a requestBodySize, o puede estar vacío. |
|**tlsVersion** | Versión de TLS usada en la conexión de la solicitud. Por ejemplo: `TLS 1.2`. |
|**smbTreeConnectID** | **treeConnectId** del Bloque de mensajes del servidor (SMB) establecido en el momento de la conexión del árbol. Por ejemplo: `0x3` |
|**smbPersistentHandleID** | Identificador de manipulador persistente de una solicitud CREATE de SMB2 que sobrevive a las reconexiones de red.  Con referencia en [MS-SMB2](/openspecs/windows_protocols/ms-smb2/f1d9b40d-e335-45fc-9d0b-199a31ede4c3) 2.2.14.1 como **SMB2_FILEID.Persistent**. Por ejemplo: `0x6003f` |
|**smbVolatileHandleID** | Identificador de manipulador volátil de una solicitud CREATE de SMB2 que se recicla en las reconexiones de red.  Con referencia en [MS-SMB2](/openspecs/windows_protocols/ms-smb2/f1d9b40d-e335-45fc-9d0b-199a31ede4c3) 2.2.14.1 como **SMB2_FILEID.Volatile**. Por ejemplo: `0xFFFFFFFF00000065` |
|**smbMessageID** | Conexión con respecto a **MessageId**. Por ejemplo: `0x3b165` |
|**smbCreditsConsumed** | Entrada o salida consumida por la solicitud, en unidades de 64 k. Por ejemplo: `0x3` |
|**smbCommandDetail** | Más información sobre esta solicitud específica en lugar del tipo general de solicitud. Por ejemplo: `0x2000 bytes at offset 0xf2000` |
|**smbFileId** | **FileId** asociado al archivo o directorio.  Más o menos análogo a un elemento FileId de NTFS. Por ejemplo: `0x9223442405598953` |
|**smbSessionID** | El elemento **SessionId** de SMB2 establecido en el momento de la configuración de la sesión. Por ejemplo: `0x8530280128000049` |
|**smbCommandMajor  uint32** | Valor de **SMB2_HEADER.Command**. Actualmente, se trata de un número entre 0 y 18, ambos incluidos. Por ejemplo: `0x6` |
|**smbCommandMinor** | Subclase de **SmbCommandMajor**, cuando sea conveniente. Por ejemplo: `DirectoryCloseAndDelete` |