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
ms.openlocfilehash: 8476c4891cef9d9055b16c7ac574e569ecf5b3f2
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107864871"
---
# <a name="get-documents-status"></a>Obtención del estado de los documentos

El método de obtención del estado de los documentos devuelve el estado de todos los documentos en una solicitud de traducción de documento por lote.

Los documentos incluidos en la respuesta se ordenan por identificador de documento en orden descendente. Si el número de documentos de la respuesta supera el límite de paginación, se usa la paginación del lado de servidor. Las respuestas paginadas indican un resultado parcial e incluyen un token de continuación en la respuesta. La ausencia de un token de continuación significa que no hay ninguna página adicional disponible.

Los parámetros de consulta $top y $skip se pueden usar para especificar un número de resultados que se van a devolver y un desplazamiento de la colección. El servidor respeta los valores especificados por el cliente. Sin embargo, los clientes deben estar preparados para controlar las respuestas que contienen un tamaño de página diferente o que contienen un token de continuación.

Cuando se incluyen $top y $skip, el servidor debe aplicar primero $skip y, a continuación, $top en la colección.

> [!NOTE]
> Si el servidor no puede aceptar $top o $skip, el servidor debe devolver un error al cliente que informa sobre él en lugar de simplemente omitir las opciones de consulta. Esto reduce el riesgo de que el cliente realice suposiciones sobre los datos devueltos.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `GET` a:
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/batches/{id}/documents
```

Aprenda a encontrar su [nombre de dominio personalizado](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Todas las solicitudes de API al servicio de traducción de documentos requieren un punto de conexión de dominio personalizado**.
> * No usará el punto de conexión que se encuentra en la página _Claves y punto de conexión_ del recurso en Azure Portal, ni el punto de conexión global del traductor (`api.cognitive.microsofttranslator.com`) para realizar solicitudes HTTP de traducción de documentos.

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

|Parámetro de consulta|Obligatorio|Descripción|
|--- |--- |--- |
|id|Verdadero|El identificador de la operación.|
|$skip|False|Omitir las entradas $skip de la colección. Cuando se proporcionan $top y $skip, se aplica primero $skip.|
|$top|False|Adoptar las entradas $top de la colección. Cuando se proporcionan $top y $skip, se aplica primero $skip.|

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
|value|DocumentStatusDetail []|El estado detallado de los documentos individuales que se enumeran a continuación|
|value.path|string|Ubicación del documento o carpeta.|
|value.createdDateTimeUtc|string|Fecha y hora de creación de la operación.|
|value.lastActionDateTimeUt|string|Fecha y hora en que se actualizó el estado de la operación.|
|value.status|status|Lista de posibles estados del trabajo o documento.<ul><li>Canceled</li><li>Cancelling</li><li>Con error</li><li>NotStarted</li><li>En ejecución</li><li>Correcto</li><li>ValidationFailed</li></ul>|
|value.to|string|A language.|
|value.progress|string|Progreso de la traducción, si está disponible.|
|value.id|string|Identificador del documento.|
|value.characterCharged|integer|Caracteres cargados por la API.|

### <a name="error-response"></a>Respuesta de error

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|código|string|Enumeraciones que contiene códigos de error de alto nivel. Valores posibles:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>RequestRateTooHigh</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>No autorizado</li></ul>|
|message|string|Obtiene un mensaje de error de alto nivel.|
|Destino|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" en el caso de un documento no válido.|
|innerError|InnerErrorV2|Nuevo formato de error interno, que cumple las directrices de la API de Cognitive Services. Contiene las propiedades requeridas ErrorCode, mensajey las propiedades opcionales de destino, detalles (par clave-valor), error interno (puede estar anidado).|
|innerError.code|string|Obtiene la cadena de error de código.|
|innerError.message|string|Obtiene un mensaje de error de alto nivel.|

## <a name="examples"></a>Ejemplos

### <a name="example-successful-response"></a>Ejemplo de respuesta correcta

Aquí se muestra un ejemplo de respuesta correcta.

```JSON
{
  "value": [
    {
      "path": "https://myblob.blob.core.windows.net/destinationContainer/fr/mydoc.txt",
      "createdDateTimeUtc": "2020-03-26T00:00:00Z",
      "lastActionDateTimeUtc": "2020-03-26T01:00:00Z",
      "status": "Running",
      "to": "fr",
      "progress": 0.1,
      "id": "273622bd-835c-4946-9798-fd8f19f6bbf2",
      "characterCharged": 0
    }
  ],
  "@nextLink": "https://westus.cognitiveservices.azure.com/translator/text/batch/v1.0.preview.1/operation/0FA2822F-4C2A-4317-9C20-658C801E0E55/documents?$top=5&$skip=15"
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
