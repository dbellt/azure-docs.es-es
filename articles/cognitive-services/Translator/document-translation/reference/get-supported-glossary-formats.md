---
title: Método de obtención de formatos de glosario admitidos
titleSuffix: Azure Cognitive Services
description: El método de obtención de formatos de glosario admitidos devuelve la lista de los formatos de glosario admitidos.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/21/2021
ms.author: v-jansk
ms.openlocfilehash: 0185a4b18ed56899de9c235bbd0438ef1dedf7c4
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111412734"
---
# <a name="get-supported-glossary-formats"></a>Obtención de formatos de glosario admitidos

El método de obtención de formatos de glosario admitidos devuelve una lista de formatos de glosario admitidos por el servicio Traducción de documentos. La lista incluye la extensión de archivo común usada.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `GET` a:
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0/glossaries/formats
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
|200|Aceptar. Devuelve la lista de formatos de archivo de glosario admitidos.|
|500|Error interno del servidor.|
|Otros códigos de estado|<ul><li>Demasiadas solicitudes</li><li>Servidor temporalmente no disponible</li></ul>|


## <a name="get-supported-glossary-formats-response"></a>Respuesta de obtención de formatos de glosario admitidos

Tipo base para la lista devuelta en la API de obtención de formatos de glosario admitidos.

### <a name="successful-get-supported-glossary-formats-response"></a>Respuesta correcta de obtención de formatos de glosario admitidos

Tipo base para la lista devuelta en la API de obtención de formatos de glosario admitidos.

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|value|FileFormat []|FileFormat[] contiene los detalles siguientes.|
|value.contentTypes|cadena []|Tipos de contenido admitidos para este formato.|
|value.defaultVersion|string|Versión predeterminada si no se especifica ninguna|
|value.fileExtensions|cadena []| Extensión de archivo admitido para este formato.|
|value.format|string|Nombre del formato.|
|value.versions|cadena []| Versión admitida.|

### <a name="error-response"></a>Respuesta de error

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|código|string|Enumeraciones que contiene códigos de error de alto nivel. Valores posibles:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>RequestRateTooHigh</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>No autorizado</li></ul>|
|message|string|Obtiene un mensaje de error de alto nivel.|
|innerError|InnerTranslationError|Nuevo formato de error interno que cumple las directrices de Cognitive Services API. Contiene las propiedades necesarias ErrorCode, message y las propiedades opcionales, target, details (par clave-valor), inner error (se puede anidar).|
|innerError.code|string|Obtiene la cadena de error de código.|
|innerError.message|string|Obtiene un mensaje de error de alto nivel.|
|innerError.target|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" en el caso de un documento no válido.|

## <a name="examples"></a>Ejemplos

### <a name="example-successful-response"></a>Ejemplo de respuesta correcta

Aquí se muestra un ejemplo de respuesta correcta.

```JSON
{
    "value": [
        {
            "format": "XLIFF",
            "fileExtensions": [
                ".xlf"
            ],
            "contentTypes": [
                "application/xliff+xml"
            ],
            "defaultVersion": "1.2",
            "versions": [
                "1.0",
                "1.1",
                "1.2"
            ]
        },
        {
            "format": "TSV",
            "fileExtensions": [
                ".tsv",
                ".tab"
            ],
            "contentTypes": [
                "text/tab-separated-values"
            ]
        },
        {
            "format": "CSV",
            "fileExtensions": [
                ".csv"
            ],
            "contentTypes": [
                "text/csv"
            ]
        }
    ]
}

```

### <a name="example-error-response"></a>Ejemplo de respuesta con error
A continuación se presenta un ejemplo de respuesta con error. El esquema de otros códigos de error es el mismo.

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
