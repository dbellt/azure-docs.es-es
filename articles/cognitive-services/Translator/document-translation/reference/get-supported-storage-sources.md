---
title: Método de obtención de orígenes de almacenamiento admitidos
titleSuffix: Azure Cognitive Services
description: El método de obtención de orígenes de almacenamiento admitidos devuelve una lista de los orígenes de almacenamiento admitidos.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/21/2021
ms.author: v-jansk
ms.openlocfilehash: b0cd1c179de36147169239ab50785a9d93dfa5bb
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110458765"
---
# <a name="get-supported-storage-sources"></a>Obtención de orígenes de almacenamiento admitidos

El método de obtención de orígenes de almacenamiento admitidos devuelve una lista de las opciones y los orígenes de almacenamiento admitidos por el servicio Traducción de documentos.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `GET` a:
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0/storagesources
```

Aprenda a encontrar su [nombre de dominio personalizado](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Todas las solicitudes de API al servicio de traducción de documentos requieren un punto de conexión de dominio personalizado**.
> * No usará el punto de conexión que se encuentra en la página _Claves y punto de conexión_ del recurso en Azure Portal, ni el punto de conexión global del traductor (`api.cognitive.microsofttranslator.com`) para realizar solicitudes HTTP de traducción de documentos.

## <a name="request-headers"></a>Encabezados de solicitud

Los encabezados de solicitud son:

|encabezados|Descripción|
|--- |--- |
|Ocp-Apim-Subscription-Key|Encabezado de solicitud obligatorio|

## <a name="response-status-codes"></a>Códigos de estado de respuesta

A continuación se indican los códigos de estado HTTP posibles que devuelve una solicitud.

|Código de estado|Descripción|
|--- |--- |
|200|Aceptar. Solicitud correcta y devuelve la lista de orígenes de almacenamiento.|
|500|Error interno del servidor.|
|Otros códigos de estado|<ul><li>Demasiadas solicitudes</li><li>Servidor temporalmente no disponible</li></ul>|

## <a name="get-supported-storage-sources-response"></a>Respuesta de obtención de orígenes de almacenamiento admitidos

### <a name="successful-get-supported-storage-sources-response"></a>Respuesta correcta de obtención de orígenes de almacenamiento admitidos
Tipo base para la lista devuelta en la API de obtención de orígenes de almacenamiento admitidos.

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|value|cadena []|Lista de objetos.|


### <a name="error-response"></a>Respuesta de error

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|código|string|Enumeraciones que contiene códigos de error de alto nivel. Valores posibles:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>RequestRateTooHigh</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>No autorizado</li></ul>|
|message|string|Obtiene un mensaje de error de alto nivel.|
|innerError|InnerTranslationError|Nuevo formato de error interno que cumple las directrices de Cognitive Services API. Contiene las propiedades necesarias ErrorCode, message y las propiedades opcionales, target, details (par clave-valor), innerError (se puede anidar).|
|innerError.code|string|Obtiene la cadena de error de código.|
|innerError.message|string|Obtiene un mensaje de error de alto nivel.|
|innerError.target|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" en el caso de un documento no válido.|

## <a name="examples"></a>Ejemplos

### <a name="example-successful-response"></a>Ejemplo de respuesta correcta

Aquí se muestra un ejemplo de respuesta correcta.

```JSON
{
  "value": [
    "AzureBlob"
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
