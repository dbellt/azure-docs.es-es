---
title: Inicio de traducción
titleSuffix: Azure Cognitive Services
description: Inicie una solicitud de traducción de documentos al servicio de traducción de documentos.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/21/2021
ms.author: v-jansk
ms.openlocfilehash: 2896c5c78acb98c798f85684ef6f800f82549b06
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111412716"
---
# <a name="start-translation"></a>Inicio de traducción

Use esta API para iniciar una solicitud de traducción masiva (por lotes) al servicio de traducción de documentos. Cada solicitud puede contener varios documentos y debe contener un contenedor de origen y otro de destino para cada documento.

El filtro de prefijo y sufijo (si se proporcionan) se usan para filtrar carpetas. El prefijo se aplica a la subruta después del nombre del contenedor.

Los glosarios/memorias de traducción pueden incluirse en la solicitud y el servicio los aplica cuando se traduce el documento.

Si el glosario no es válido o no se puede acceder a él durante la traducción, se indica un error en el estado del documento. Si ya existe un archivo con el mismo nombre en el destino, se sobrescribirá. El valor de targetUrl para cada idioma de destino debe ser único.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `POST` a:
```HTTP
POST https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0/batches
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

## <a name="request-body-batch-submission-request"></a>Cuerpo de la solicitud: solicitud de envío por lotes

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|inputs|BatchRequest[]|BatchRequest[] que aparece a continuación. La lista de entrada de documentos o carpetas que contienen documentos. Tipos de medios: "application/json", "text/json", "application/*+json".|

### <a name="inputs"></a>Entradas

Definición para la solicitud de traducción por lotes de entrada.

|Nombre|Tipo|Requerido|Descripción|
|--- |--- |--- |--- |
|source|SourceInput[]|Verdadero|inputs.source que aparece a continuación. Origen de los documentos de entrada.|
|storageType|StorageInputType[]|False|inputs.storageType que aparece a continuación. Tipo de almacenamiento de la cadena de origen de los documentos de entrada. Se requiere solo para la traducción de documentos únicos.|
|destinos|TargetInput[]|Verdadero|inputs.target que aparece a continuación. Ubicación del destino de la salida.|

**inputs.source**

Origen de los documentos de entrada.

|Nombre|Tipo|Requerido|Descripción|
|--- |--- |--- |--- |
|filter|DocumentFilter[]|False|DocumentFilter[] que aparece a continuación.|
|filter.prefix|string|False|Una cadena de prefijo que distingue entre mayúsculas y minúsculas para filtrar los documentos en la ruta de origen para su traducción. Por ejemplo, cuando se utiliza un URI de Azure Storage Blob, utilice el prefijo para restringir las subcarpetas para la traducción.|
|filter.suffix|string|False|Una cadena de sufijo que distingue entre mayúsculas y minúsculas para filtrar los documentos en la ruta de origen para su traducción. Esto se utiliza más a menudo para las extensiones de archivos.|
|language|string|False|Código de idioma. Si no se especifica ninguno, se realizará la detección automática en el documento.|
|sourceUrl|string|True|Ubicación de la carpeta/contenedor o archivo único con los documentos.|
|storageSource|StorageSource|False|StorageSource que aparece a continuación.|
|storageSource.AzureBlob|string|False||

**inputs.storageType**

Tipo de almacenamiento de la cadena de origen de los documentos de entrada.

|Nombre|Tipo|
|--- |--- |
|archivo|string|
|folder|string|

**inputs.target**

Destino de los documentos traducidos terminados.

|Nombre|Tipo|Requerido|Descripción|
|--- |--- |--- |--- |
|category|string|False|Categoría/sistema personalizado para la solicitud de traducción.|
|glossaries|Glossary[]|False|Glosario que aparece a continuación. Lista del glosario.|
|glossaries.format|string|False|Formato.|
|glossaries.glossaryUrl|string|Verdadero (si se usan glosarios).|Ubicación del glosario. Usaremos la extensión de archivo para extraer el formato si no se proporciona el parámetro de formato. Si el par de idiomas de traducción no está presente en el glosario, no se aplicará.|
|glossaries.storageSource|StorageSource|False|StorageSource que aparece a continuación.|
|glossaries.version|string|False|Versión opcional. Si no se especifica, se usa el valor predeterminado.|
|targetUrl|string|True|Ubicación de la carpeta/contenedor con sus documentos.|
|language|string|True|Código de dos letras del idioma de destino. Vea la [lista de códigos de idioma](../../language-support.md).|
|storageSource|StorageSource []|False|StorageSource [] que aparece a continuación.|

## <a name="example-request"></a>Solicitud de ejemplo

Los siguientes son ejemplos de solicitudes por lotes.

**Traducción de todos los documentos de un contenedor**

```json
{
    "inputs": [
        {
            "source": {
                "sourceUrl": "https://my.blob.core.windows.net/source-en?sv=2019-12-12&st=2021-03-05T17%3A45%3A25Z&se=2021-03-13T17%3A45%3A00Z&sr=c&sp=rl&sig=SDRPMjE4nfrH3csmKLILkT%2Fv3e0Q6SWpssuuQl1NmfM%3D"
            },
            "targets": [
                {
                    "targetUrl": "https://my.blob.core.windows.net/target-fr?sv=2019-12-12&st=2021-03-05T17%3A49%3A02Z&se=2021-03-13T17%3A49%3A00Z&sr=c&sp=wdl&sig=Sq%2BYdNbhgbq4hLT0o1UUOsTnQJFU590sWYo4BOhhQhs%3D",
                    "language": "fr"
                }
            ]
        }
    ]
}
```

**Traducción de todos los documentos de un contenedor aplicando glosarios**

Asegúrese de que ha creado la URL del glosario y el token de SAS para el blob/documento específico (no para el contenedor).

```json
{
    "inputs": [
        {
            "source": {
                "sourceUrl": "https://my.blob.core.windows.net/source-en?sv=2019-12-12&st=2021-03-05T17%3A45%3A25Z&se=2021-03-13T17%3A45%3A00Z&sr=c&sp=rl&sig=SDRPMjE4nfrH3csmKLILkT%2Fv3e0Q6SWpssuuQl1NmfM%3D"
            },
            "targets": [
                {
                    "targetUrl": "https://my.blob.core.windows.net/target-fr?sv=2019-12-12&st=2021-03-05T17%3A49%3A02Z&se=2021-03-13T17%3A49%3A00Z&sr=c&sp=wdl&sig=Sq%2BYdNbhgbq4hLT0o1UUOsTnQJFU590sWYo4BOhhQhs%3D",
                    "language": "fr",
                    "glossaries": [
                        {
                            "glossaryUrl": "https://my.blob.core.windows.net/glossaries/en-fr.xlf?sv=2019-12-12&st=2021-03-05T17%3A45%3A25Z&se=2021-03-13T17%3A45%3A00Z&sr=c&sp=rl&sig=BsciG3NWoOoRjOYesTaUmxlXzyjsX4AgVkt2AsxJ9to%3D",
                            "format": "xliff",
                            "version": "1.2"
                        }
                    ]

                }
            ]
        }
    ]
}
```

**Traducción de una carpeta específica en un contenedor**

Asegúrese de que ha especificado el nombre de la carpeta (distingue entre mayúsculas y minúsculas) como prefijo en el filtro, aunque el token de SAS sigue siendo para el contenedor.

```json
{
    "inputs": [
        {
            "source": {
                "sourceUrl": "https://my.blob.core.windows.net/source-en?sv=2019-12-12&st=2021-03-05T17%3A45%3A25Z&se=2021-03-13T17%3A45%3A00Z&sr=c&sp=rl&sig=SDRPMjE4nfrH3csmKLILkT%2Fv3e0Q6SWpssuuQl1NmfM%3D",
                "filter": {
                    "prefix": "MyFolder/"
                }
            },
            "targets": [
                {
                    "targetUrl": "https://my.blob.core.windows.net/target-fr?sv=2019-12-12&st=2021-03-05T17%3A49%3A02Z&se=2021-03-13T17%3A49%3A00Z&sr=c&sp=wdl&sig=Sq%2BYdNbhgbq4hLT0o1UUOsTnQJFU590sWYo4BOhhQhs%3D",
                    "language": "fr"
                }
            ]
        }
    ]
}
```

**Traducción de un documento específico en un contenedor**

* Asegúrese de que ha especificado "storageType": "File".
* Asegúrese de haber creado la URL de origen y el token de SAS para el blob/documento específico (no para el contenedor).
* Asegúrese de haber especificado el nombre del archivo de destino como parte de la dirección URL de destino, aunque el token de SAS sigue siendo para el contenedor.
* La solicitud de ejemplo siguiente muestra un único documento que se traduce en dos idiomas de destino.

```json
{
    "inputs": [
        {
            "storageType": "File",
            "source": {
                "sourceUrl": "https://my.blob.core.windows.net/source-en/source-english.docx?sv=2019-12-12&st=2021-01-26T18%3A30%3A20Z&se=2021-02-05T18%3A30%3A00Z&sr=c&sp=rl&sig=d7PZKyQsIeE6xb%2B1M4Yb56I%2FEEKoNIF65D%2Fs0IFsYcE%3D"
            },
            "targets": [
                {
                    "targetUrl": "https://my.blob.core.windows.net/target/try/Target-Spanish.docx?sv=2019-12-12&st=2021-01-26T18%3A31%3A11Z&se=2021-02-05T18%3A31%3A00Z&sr=c&sp=wl&sig=AgddSzXLXwHKpGHr7wALt2DGQJHCzNFF%2F3L94JHAWZM%3D",
                    "language": "es"
                },
                {
                    "targetUrl": "https://my.blob.core.windows.net/target/try/Target-German.docx?sv=2019-12-12&st=2021-01-26T18%3A31%3A11Z&se=2021-02-05T18%3A31%3A00Z&sr=c&sp=wl&sig=AgddSzXLXwHKpGHr7wALt2DGQJHCzNFF%2F3L94JHAWZM%3D",
                    "language": "de"
                }
            ]
        }
    ]
}
```

## <a name="response-status-codes"></a>Códigos de estado de respuesta

A continuación se indican los códigos de estado HTTP posibles que devuelve una solicitud.

|Código de estado|Descripción|
|--- |--- |
|202|Accepted. El servicio crea correctamente la solicitud y la solicitud por lotes. El encabezado Operation-Location indicará una dirección URL de estado con la operación ID.HeadersOperation-Location: string.|
|400|Solicitud incorrecta. Solicitud no válida. Compruebe los parámetros de entrada.|
|401|No autorizado. Compruebe las credenciales.|
|429|La tasa de solicitudes es demasiado alta.|
|500|Error interno del servidor.|
|503|El servicio no está disponible en este momento.  Vuelva a intentarlo más tarde.|
|Otros códigos de estado|<ul><li>Demasiadas solicitudes</li><li>Servidor temporal no disponible.</li></ul>|

## <a name="error-response"></a>Respuesta de error

|Nombre|Tipo|Descripción|
|--- |--- |--- |
|código|string|Enumeraciones que contiene códigos de error de alto nivel. Valores posibles:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>RequestRateTooHigh</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>No autorizado</li></ul>|
|message|string|Obtiene un mensaje de error de alto nivel.|
|innerError|InnerTranslationError|Nuevo formato de error interno que cumple las directrices de Cognitive Services API. Contiene las propiedades necesarias ErrorCode, message y las propiedades opcionales, target, details (par clave-valor), innerError (se puede anidar).|
|inner.Errorcode|string|Obtiene la cadena de error de código.|
|innerError.message|string|Obtiene un mensaje de error de alto nivel.|
|innerError.target|string|Obtiene el origen del error. Por ejemplo, sería "documentos" o "id. de documento" en el caso de un documento no válido.|

## <a name="examples"></a>Ejemplos

### <a name="example-successful-response"></a>Ejemplo de respuesta correcta

La siguiente información se devuelve en una respuesta correcta.

Puede encontrar el identificador del trabajo en el valor de la URL de Operation-Location del encabezado de respuesta. El último parámetro de la dirección URL es el identificador del trabajo de la operación (la cadena que sigue a "/operation/").

```HTTP
Operation-Location: https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0/operation/0FA2822F-4C2A-4317-9C20-658C801E0E55
```

### <a name="example-error-response"></a>Respuesta de error de ejemplo

```JSON
{
  "error": {
    "code": "ServiceUnavailable",
    "message": "Service is temporary unavailable",
    "innerError": {
      "code": "ServiceTemporaryUnavailable",
      "message": "Service is currently unavailable.  Please try again later"
    }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Siga nuestro inicio rápido para obtener más información sobre el uso de Traducción de documentos y la biblioteca cliente.

> [!div class="nextstepaction"]
> [Introducción a la traducción de documentos](../get-started-with-document-translation.md)
