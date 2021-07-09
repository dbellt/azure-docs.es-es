---
title: Método de cancelación de la traducción
titleSuffix: Azure Cognitive Services
description: El método de cancelación de la traducción cancela una operación que está en cola o cuyo procesamiento está en curso.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/21/2021
ms.author: v-jansk
ms.openlocfilehash: 46348fe35f425811e2ff03208feeae6ab7a112bd
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110453586"
---
# <a name="cancel-translation"></a>Cancelación de la traducción

Puede cancelar una operación en colo o cuyo procesamiento está en curso. Sin embargo, no se podrá cancelar una operación si ya se ha completado, ha generado algún error o ya está cancelada. Se devolverá una solicitud incorrecta. Todos los documentos que hayan completado la traducción no se cancelarán y se cobrarán. Si es posible, se cancelarán todos los documentos pendientes.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `DELETE` a:

```DELETE HTTP
https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0/batches/{id}
```

Aprenda a encontrar su [nombre de dominio personalizado](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Todas las solicitudes de API al servicio de traducción de documentos requieren un punto de conexión de dominio personalizado**.
> * No usará el punto de conexión que se encuentra en la página _Claves y punto de conexión_ del recurso en Azure Portal, ni el punto de conexión global del traductor (`api.cognitive.microsofttranslator.com`) para realizar solicitudes HTTP de traducción de documentos.

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

|Parámetro de consulta|Obligatorio|Descripción|
|-----|-----|-----|
|id|Verdadero|Identificador de la operación.|

## <a name="request-headers"></a>Encabezados de solicitud

Los encabezados de solicitud son:

|encabezados|Descripción|
|-----|-----|
|Ocp-Apim-Subscription-Key|Encabezado de solicitud obligatorio|

## <a name="response-status-codes"></a>Códigos de estado de respuesta

A continuación se indican los códigos de estado HTTP posibles que devuelve una solicitud.

| Código de estado| Descripción|
|-----|-----|
|200|Aceptar. La solicitud de cancelación se ha enviado.|
|401|No autorizado. Compruebe sus credenciales.|
|404|Not found. No se encuentra el recurso. 
|500|Error interno del servidor.
|Otros códigos de estado|<ul><li>Demasiadas solicitudes</li><li>Servidor temporalmente no disponible</li></ul>|

## <a name="cancel-translation-response"></a>Respuesta de cancelación de la traducción

### <a name="successful-response"></a>Respuesta correcta

En una respuesta correcta se devuelve la información siguiente.

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|id|string|Identificador de la operación.|
|createdDateTimeUtc|string|Fecha y hora de creación de la operación.|
|lastActionDateTimeUtc|string|Fecha y hora en que se actualizó el estado de la operación.|
|status|String|Lista de posibles estados del trabajo o documento: <ul><li>Canceled</li><li>Cancelling</li><li>Con error</li><li>NotStarted</li><li>En ejecución</li><li>Correcto</li><li>ValidationFailed</li></ul>|
|Resumen|StatusSummary|Resumen que contiene los detalles que se muestran a continuación.|
|summary.total|integer|Recuento total de documentos.|
|summary.failed|integer|Recuento de documentos con errores.|
|summary.success|integer|Recuento de documentos traducidos correctamente.|
|summary.inProgress|integer|Recuento de documentos en curso.|
|summary.notYetStarted|integer|Recuento de documentos que aún no se han empezado a procesar.|
|summary.cancelled|integer|Número de cancelaciones.|
|summary.totalCharacterCharged|integer|Caracteres totales cargados por la API.|

### <a name="error-response"></a>Respuesta de error

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|código|string|Enumeraciones que contiene códigos de error de alto nivel. Valores posibles:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>RequestRateTooHigh</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>No autorizado</li></ul>|
|message|string|Obtiene un mensaje de error de alto nivel.|
|Destino|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" para un documentación no válido.|
|innerError|InnerTranslationError|Nuevo formato de error interno, que cumple las directrices de Cognitive Services API. Contiene las propiedades necesarias ErrorCode, message y las propiedades opcionales, target, details (par clave-valor), innerError (se puede anidar).|
|innerError.code|string|Obtiene la cadena de error de código.|
|innerError.message|string|Obtiene un mensaje de error de alto nivel.|
|innerError.target|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" si se trata de un documento no válido.|

## <a name="examples"></a>Ejemplos

### <a name="example-successful-response"></a>Ejemplo de respuesta correcta

El siguiente objeto JSON es un ejemplo de una respuesta correcta.

Código de estado: 200

```JSON
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
```

### <a name="example-error-response"></a>Ejemplo de respuesta con error

El siguiente objeto JSON es un ejemplo de una respuesta con error. El esquema de otros códigos de error es el mismo.

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
