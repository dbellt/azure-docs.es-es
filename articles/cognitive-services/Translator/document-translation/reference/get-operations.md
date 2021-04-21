---
title: Obtener operaciones en la traducción de documentos
titleSuffix: Azure Cognitive Services
description: El método Obtener operaciones devuelve una lista de solicitudes por lotes enviadas y el estado de cada solicitud.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/25/2021
ms.author: v-jansk
ms.openlocfilehash: fd7cee564aa3a00e21d1e707d08a18115d519925
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484683"
---
# <a name="document-translation-get-operations"></a>Traducción de documentos: Obtener operaciones

El método Obtener operaciones devuelve una lista de solicitudes por lotes enviadas y el estado de cada solicitud. Esta lista solo contiene las solicitudes por lotes enviadas por el usuario (según la suscripción). El estado de cada solicitud se ordena por el identificador.

Si el número de solicitudes supera el límite de paginación, se usa la paginación en el servidor. Las respuestas paginadas indican un resultado parcial e incluyen un token de continuación en la respuesta. La ausencia de un token de continuación significa que no hay ninguna página adicional disponible.

Los parámetros de consulta $top y $skip se pueden usar para especificar un número de resultados que se van a devolver y un desplazamiento de la colección.

El servidor respeta los valores especificados por el cliente. Sin embargo, los clientes deben estar preparados para controlar las respuestas que contienen un tamaño de página diferente o que contienen un token de continuación.

Cuando se incluyen $top y $skip, el servidor debe aplicar primero $skip y, a continuación, $top en la colección. 

> [!NOTE]
> Si el servidor no puede aceptar $top o $skip, el servidor debe devolver un error al cliente que informa sobre él en lugar de simplemente omitir las opciones de consulta. Esto reduce el riesgo de que el cliente realice suposiciones sobre los datos devueltos.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `GET` a:
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/batches
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
|200|Aceptar. Solicitud correcta y devuelve el estado de todas las operaciones. HeadersRetry-After: integerETag: string|
|400|Solicitud incorrecta. Solicitud no válida. Compruebe los parámetros de entrada.|
|401|No autorizado. Compruebe sus credenciales.|
|500|Error interno del servidor.|
|Otros códigos de estado|<ul><li>Demasiadas solicitudes</li><li>Servidor temporalmente no disponible</li></ul>|

## <a name="get-operations-response"></a>Respuesta a Obtener operaciones

### <a name="successful-get-operations-response"></a>Respuesta correcta a Obtener operaciones

En una respuesta correcta se devuelve la información siguiente.

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|id|string|Identificador de la operación.|
|createdDateTimeUtc|string|Fecha y hora de creación de la operación.|
|lastActionDateTimeUtc|string|Fecha y hora en que se actualizó el estado de la operación.|
|status|String|Lista de posibles estados del trabajo o documento: <ul><li>Canceled</li><li>Cancelling</li><li>Con error</li><li>NotStarted</li><li>En ejecución</li><li>Correcto</li><li>ValidationFailed</li></ul>|
|Resumen|StatusSummary[]|Resumen que contiene los detalles que se muestran a continuación.|
|summary.total|integer|Recuento total de documentos.|
|summary.failed|integer|Recuento de documentos con errores.|
|summary.success|integer|Recuento de documentos traducidos correctamente.|
|summary.inProgress|integer|Recuento de documentos en curso.|
|summary.notYetStarted|integer|Recuento de documentos que aún no se han empezado a procesar.|
|summary.cancelled|integer|Recuento de documentos cancelados.|
|summary.totalCharacterCharged|integer|Recuento total de caracteres cargados.|

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
      "id": "727bf148-f327-47a0-9481-abae6362f11e",
      "createdDateTimeUtc": "2020-03-26T00:00:00Z",
      "lastActionDateTimeUtc": "2020-03-26T01:00:00Z",
      "status": "Succeeded",
      "summary": {
        "total": 10,
        "failed": 1,
        "success": 9,
        "inProgress": 0,
        "notYetStarted": 0,
        "cancelled": 0,
        "totalCharacterCharged": 0
      }
    }
  ]
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
