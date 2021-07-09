---
title: Obtención del estado de los documentos
titleSuffix: Azure Cognitive Services
description: El método de obtención del estado de los documentos devuelve el estado de todos los documentos en una solicitud de traducción de documento por lote.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/21/2021
ms.author: v-jansk
ms.openlocfilehash: 355e692d6091cee443608c2239173c873bb8e1a5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110453551"
---
# <a name="get-documents-status"></a>Obtención del estado de los documentos

Si el número de documentos de la respuesta supera el límite de paginación, se usa la paginación del lado de servidor. Las respuestas paginadas indican un resultado parcial e incluyen un token de continuación en la respuesta. La ausencia de un token de continuación significa que no hay otras páginas disponibles.

Los parámetros de consulta $top, $skip y $maxpagesize se pueden usar para especificar el número de resultados que se van a devolver y el desplazamiento en la colección.

$top indica el número total de registros que el usuario desea que se devuelvan entre todas las páginas. $skip indica el número de registros que se omitirán de la lista de estado de documentos mantenida por el servidor según el método de ordenación especificado. De manera predeterminada, se ordena por hora de inicio descendente. $maxpagesize es el número máximo de elementos que se devuelven en una página. Si se solicitan más elementos mediante $top (o no se especifica $top y hay más elementos que devolver), @nextLink contendrá el vínculo a la página siguiente.

El parámetro de consulta $orderBy se puede usar para ordenar la lista devuelta (por ejemplo, "$orderBy=createdDateTimeUtc asc" o "$orderBy=createdDateTimeUtc desc"). La ordenación predeterminada es por createdDateTimeUtc descendente. Se pueden usar algunos parámetros de consulta para filtrar la lista devuelta (por ejemplo, "status=Succeeded,Cancelled") solo devolverá documentos correctos y cancelados. createdDateTimeUtcStart y createdDateTimeUtcEnd se pueden usar combinados o por separado para especificar un intervalo de fecha y hora por el que filtrar la lista devuelta. Los parámetros de consulta de filtrado admitidos son (status, ID, createdDateTimeUtcStart y createdDateTimeUtcEnd).

Cuando se incluyen $top y $skip, el servidor debe aplicar primero $skip y, a continuación, $top en la colección. 

> [!NOTE]
> Si el servidor no puede aceptar $top o $skip, el servidor debe devolver un error al cliente que informa sobre él en lugar de simplemente omitir las opciones de consulta. Esto reduce el riesgo de que el cliente realice suposiciones sobre los datos devueltos.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `GET` a:
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0/batches/{id}/documents
```

Aprenda a encontrar su [nombre de dominio personalizado](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Todas las solicitudes de API al servicio de traducción de documentos requieren un punto de conexión de dominio personalizado**.
> * No usará el punto de conexión que se encuentra en la página _Claves y punto de conexión_ del recurso en Azure Portal, ni el punto de conexión global del traductor (`api.cognitive.microsofttranslator.com`) para realizar solicitudes HTTP de traducción de documentos.

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

|Parámetro de consulta|En|Obligatorio|Tipo|Descripción|
|--- |--- |--- |--- |--- |
|id|path|True|string|El identificador de la operación.|
|$maxpagesize|Query|Falso|integer int32|$maxpagesize es el número máximo de elementos que se devuelven en una página. Si se solicitan más elementos mediante $top (o no se especifica $top y hay más elementos que devolver), @nextLink contendrá el vínculo a la página siguiente. Los clientes PUEDEN solicitar la paginación controlada por el servidor con un tamaño de página específico especificando su preferencia para $maxpagesize. El servidor DEBE respetar esta preferencia si el tamaño de página especificado es menor que el tamaño de página predeterminado del servidor.|
|$orderBy|Query|Falso|array|Consulta de ordenación de la colección (por ejemplo, "CreatedDateTimeUtc asc", "CreatedDateTimeUtc desc").|
|$skip|Query|Falso|integer int32|$skip indica el número de registros que se omitirán de la lista de registros mantenida por el servidor según el método de ordenación especificado. De manera predeterminada, se ordena por hora de inicio descendente. Los clientes pueden usar los parámetros de consulta $top y $skip para especificar el número de resultados que se van a devolver y el desplazamiento en la colección. Cuando el cliente especifica $top y $skip, el servidor DEBE aplicar primero $skip y, a continuación, $top en la colección. Nota: Si el servidor no puede respetar $top o $skip, el servidor DEBE devolver un error al cliente que informe al respecto en lugar de simplemente omitir las opciones de consulta.|
|$top|Query|Falso|integer int32|$top indica el número total de registros que el usuario desea que se devuelvan entre todas las páginas. Los clientes pueden usar los parámetros de consulta $top y $skip para especificar el número de resultados que se van a devolver y el desplazamiento en la colección. Cuando el cliente especifica $top y $skip, el servidor DEBE aplicar primero $skip y, a continuación, $top en la colección. Nota: Si el servidor no puede respetar $top o $skip, el servidor DEBE devolver un error al cliente que informe al respecto en lugar de simplemente omitir las opciones de consulta.|
|createdDateTimeUtcEnd|Query|Falso|string date-time|Fecha y hora de finalización para obtener los elementos anteriores.|
|createdDateTimeUtcStart|Query|Falso|string date-time|Fecha y hora de inicio después de la cual se obtienen los elementos.|
|ids|Query|Falso|array|Los ID que se usarán en el filtrado.|
|statuses|Query|Falso|array|Estados que se usarán en el filtrado.|

## <a name="request-headers"></a>Encabezados de solicitud

Los encabezados de solicitud son:

|encabezados|Descripción|
|--- |--- |
|Ocp-Apim-Subscription-Key|Encabezado de solicitud obligatorio|

## <a name="response-status-codes"></a>Códigos de estado de respuesta

A continuación se indican los códigos de estado HTTP posibles que devuelve una solicitud.

|Código de estado|Descripción|
|--- |--- |
|200|Aceptar. Solicitud correcta y devuelve el estado de los documentos. HeadersRetry-After: integerETag: string|
|400|Solicitud no válida. Compruebe los parámetros de entrada.|
|401|No autorizado. Compruebe sus credenciales.|
|404|No se encuentra el recurso.|
|500|Error interno del servidor.|
|Otros códigos de estado|<ul><li>Demasiadas solicitudes</li><li>Servidor temporalmente no disponible</li></ul>|


## <a name="get-documents-status-response"></a>Respuesta de obtención del estado de los documentos

### <a name="successful-get-documents-status-response"></a>Respuesta correcta del estado de obtención de los documentos

En una respuesta correcta se devuelve la información siguiente.

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|@nextLink|string|Dirección URL de la página siguiente Valor NULL si no hay más páginas disponibles|
|value|DocumentStatus []|El estado detallado de los documentos individuales que se enumeran a continuación|
|value.path|string|Ubicación del documento o carpeta.|
|value.sourcePath|string|Ubicación del documento de origen.|
|value.createdDateTimeUtc|string|Fecha y hora de creación de la operación.|
|value.lastActionDateTimeUtc|string|Fecha y hora en que se actualizó el estado de la operación.|
|value.status|status|Lista de posibles estados del trabajo o documento.<ul><li>Canceled</li><li>Cancelling</li><li>Con error</li><li>NotStarted</li><li>En ejecución</li><li>Correcto</li><li>ValidationFailed</li></ul>|
|value.to|string|A language.|
|value.progress|number|Progreso de la traducción, si está disponible.|
|value.id|string|Identificador del documento.|
|value.characterCharged|integer|Caracteres cargados por la API.|

### <a name="error-response"></a>Respuesta de error

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|código|string|Enumeraciones que contiene códigos de error de alto nivel. Valores posibles:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>RequestRateTooHigh</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>No autorizado</li></ul>|
|message|string|Obtiene un mensaje de error de alto nivel.|
|Destino|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" en el caso de un documento no válido.|
|innerError|InnerTranslationError|Nuevo formato de error interno, que cumple las directrices de Cognitive Services API. Contiene las propiedades necesarias ErrorCode, message y las propiedades opcionales, target, details (par clave-valor), innerError (se puede anidar).|
|innerError.code|string|Obtiene la cadena de error de código.|
|innerError.message|string|Obtiene un mensaje de error de alto nivel.|
|innerError.target|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" si se trata de un documento no válido.|

## <a name="examples"></a>Ejemplos

### <a name="example-successful-response"></a>Ejemplo de respuesta correcta

Aquí se muestra un ejemplo de respuesta correcta.

```JSON
{
  "value": [
    {
      "path": "https://myblob.blob.core.windows.net/destinationContainer/fr/mydoc.txt",
      "sourcePath": "https://myblob.blob.core.windows.net/sourceContainer/fr/mydoc.txt",
      "createdDateTimeUtc": "2020-03-26T00:00:00Z",
      "lastActionDateTimeUtc": "2020-03-26T01:00:00Z",
      "status": "Running",
      "to": "fr",
      "progress": 0.1,
      "id": "273622bd-835c-4946-9798-fd8f19f6bbf2",
      "characterCharged": 0
    }
  ],
  "@nextLink": "https://westus.cognitiveservices.azure.com/translator/text/batch/v1.0/operation/0FA2822F-4C2A-4317-9C20-658C801E0E55/documents?$top=5&$skip=15"
}
```

### <a name="example-error-response"></a>Ejemplo de respuesta con error

A continuación se presenta un ejemplo de una respuesta con error. El esquema de otros códigos de error es el mismo.

Código de estado: 500

```JSON
{
  "error": {
    "code": "InternalServerError",
    "message": "Internal Server Error",
    "target": "Operation",
    "innerError": {
      "code": "InternalServerError",
      "message": "Unexpected internal server error has occurred"
    }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Siga nuestro inicio rápido para obtener más información sobre el uso de Traducción de documentos y la biblioteca cliente.

> [!div class="nextstepaction"]
> [Introducción a la traducción de documentos](../get-started-with-document-translation.md)
