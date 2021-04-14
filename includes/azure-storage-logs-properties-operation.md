---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: 2474b8920c5387c7896b413f229c2f5b06cdafb1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96011190"
---
| Propiedad | Descripción |
|:--- |:---|
|**time** | Hora universal coordinada (UTC) a la que el almacenamiento ha recibido la solicitud. Por ejemplo: `2018/11/08 21:09:36.6900118`.|
|**resourceId** | Identificador de recurso de la cuenta de almacenamiento. Por ejemplo: `/subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/`<br>`myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/storageAccounts/blobServices/default`|
|**category** | Categoría de la operación solicitada. Por ejemplo: `StorageRead`, `StorageWrite` o `StorageDelete`.|
|**operationName** | Tipo de operación REST realizada. <br> Puede encontrar una lista completa de operaciones en el tema [Operaciones y mensajes de estado registrados de Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). |
|**operationVersion** | Versión del servicio de almacenamiento especificada al realizar la solicitud. Es equivalente al valor del encabezado **x-ms-version**. Por ejemplo: `2017-04-17`.|
|**schemaVersion** | Versión del esquema del registro. Por ejemplo: `1.0`.|
|**statusCode** | Código de estado HTTP de la solicitud. Si se interrumpe la solicitud, este valor se podría establecer en `Unknown`. <br> Por ejemplo: `206` |
|**statusText** | Estado de la operación solicitada.  Puede encontrar una lista completa de mensajes de estado en [Operaciones y mensajes de estado registrados por Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages). En la versión 2017-04-17 y posteriores, el mensaje de estado `ClientOtherError` no se usa. En su lugar, este campo contiene un código de error. Por ejemplo: `SASSuccess`  |
|**durationMs** | Tiempo total, expresado en milisegundos, necesario para realizar la operación solicitada. Incluye el tiempo necesario para leer la solicitud entrante y para enviar la respuesta al solicitante. Por ejemplo: `12`.|
|**callerIpAddress** | Dirección IP del solicitante, incluido el número de puerto. Por ejemplo: `192.100.0.102:4362`. |
|**correlationId** | Identificador que se usa para poner en correlación registros entre recursos. Por ejemplo: `b99ba45e-a01e-0042-4ea6-772bbb000000`. |
|**ubicación** | Ubicación de la cuenta de almacenamiento. Por ejemplo: `North Europe`. |
|**protocolo**|Protocolo que se usa en la operación. Por ejemplo: `HTTP`, `HTTPS`, `SMB` o `NFS`|
| **uri** | Identificador uniforme de recursos que se solicita. |