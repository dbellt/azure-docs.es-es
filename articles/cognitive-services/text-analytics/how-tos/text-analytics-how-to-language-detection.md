---
title: Detección del idioma con la API REST de Text Analytics
titleSuffix: Azure Cognitive Services
description: Detecte el idioma con la API de REST de Text Analytics desde Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: sample
ms.date: 06/10/2021
ms.author: aahi
ms.openlocfilehash: c9ff3bdc7e7076846ffb44239bf5654512f88eea
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111967964"
---
# <a name="example-detect-language-with-text-analytics"></a>Ejemplo: Detectar idioma con Text Analytics

La característica [Detección de idioma](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) de la API de REST de Azure Text Analytics valora la entrada de texto para cada documento y devuelve identificadores de idioma con una puntuación que indica la solidez del análisis.

Esta capacidad es útil para los almacenes de contenido que recopilan texto arbitrario, donde el idioma es desconocido. Puede analizar los resultados del análisis para determinar el idioma que se usa en el documento de entrada. La respuesta también devuelve una puntuación que refleja la confianza del modelo. El valor de puntuación se encuentra entre 0 y 1.

La característica Detección de idioma puede detectar una amplia gama de idiomas, variantes, dialectos y algunos idiomas regionales o culturales. La lista exacta de idiomas para esta característica no está publicada.

Si tiene contenido que se expresa en un idioma que se usa con menos frecuencia, puede probar la característica Detección de idioma para ver si devuelve un código. La respuesta para los idiomas que no se pueden detectar es `unknown`.

> [!TIP]
> Text Analytics proporciona también una imagen de contenedor de Docker basada en Linux para la detección del lenguaje, por lo que puede [instalar y ejecutar el contenedor de Text Analytics](text-analytics-how-to-install-containers.md) cerca de los datos.

## <a name="preparation"></a>Preparación

Debe tener documentos JSON en este formato: Identificador y texto.

El tamaño del documento debe ser inferior a 5.120 caracteres por documento. Puede tener hasta 1000 elementos (identificadores) por colección. La colección se envía en el cuerpo de la solicitud. En el siguiente ejemplo se presenta contenido que se podría enviar para la detección del idioma:

```json
{
    "documents": [
        {
            "id": "1",
            "text": "This document is in English."
        },
        {
            "id": "2",
            "text": "Este documento está en inglés."
        },
        {
            "id": "3",
            "text": "Ce document est en anglais."
        },
        {
            "id": "4",
            "text": "本文件为英文"
        },
        {
            "id": "5",
            "text": "Этот документ на английском языке."
        }
    ]
}
```

## <a name="step-1-structure-the-request"></a>Paso 1: Estructurar la solicitud

Para obtener más información sobre la definición de la solicitud, consulte [Llamada a la API de Text Analytics](text-analytics-how-to-call-api.md). Recapitulamos los siguientes puntos para su comodidad:

+ Cree una solicitud POST. Para revisar la documentación de la API para esta solicitud, consulte [API de Detección de idioma](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages).

+ Establezca el punto de conexión HTTP para la detección de idioma. Use un recurso de Text Analytics en Azure o un [contenedor de Text Analytics](text-analytics-how-to-install-containers.md) con instancias. Debe incluir `/text/analytics/v3.0/languages` en la dirección URL. Por ejemplo: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/languages`.

+ Establezca un encabezado de solicitud para incluir la [clave de acceso](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) para las operaciones de Text Analytics.

+ En el cuerpo de la solicitud, proporcione la colección de documentos JSON que preparó para este análisis.

> [!Tip]
> Use [Postman](text-analytics-how-to-call-api.md) o abra la **consola de prueba de la API** en la [documentación](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) para estructurar y enviar una solicitud POST al servicio.

## <a name="step-2-post-the-request"></a>Paso 2: Publicar la solicitud

El análisis se realiza tras la recepción de la solicitud. Para información sobre el tamaño y el número de solicitudes que puede enviar por minuto y segundo, consulte el artículo sobre [límites de datos](../concepts/data-limits.md).

Recuerde que el servicio no tiene estado. No se almacena ningún dato en su cuenta. Los resultados se devuelven inmediatamente en la respuesta.


## <a name="step-3-view-the-results"></a>Paso 3: View the results

Todas las solicitudes POST devolverán una respuesta con formato JSON con los identificadores y las propiedades detectadas.

La salida se devuelve inmediatamente. Puede transmitir los resultados a una aplicación que acepte JSON o guardar la salida en un archivo en el sistema local. Después, importe el resultado en una aplicación que pueda usar para ordenar los datos, realizar búsquedas en ellos y manipularlos.

Los resultados de la solicitud de ejemplo deben parecerse al siguiente documento JSON. Tenga en cuenta que se trata de un documento JSON con varios elementos, y cada uno de ellos representa el resultado de la detección de cada documento que se envía. La salida está disponible en inglés. 

La detección de idioma devolverá un idioma predominante para un documento, junto con su nombre [ISO 639-1](https://www.iso.org/standard/22109.html), su nombre descriptivo y la puntuación de confianza. Una puntuación positiva de 1,0 expresa el nivel más alto de confianza posible del análisis.

```json
{
    "documents":[
        {
            "detectedLanguage":{
                "confidenceScore":0.99,
                "iso6391Name":"en",
                "name":"English"
            },
            "id":"1",
            "warnings":[
                
            ]
        },
        {
            "detectedLanguage":{
                "confidenceScore":1.0,
                "iso6391Name":"es",
                "name":"Spanish"
            },
            "id":"2",
            "warnings":[
                
            ]
        },
        {
            "detectedLanguage":{
                "confidenceScore":1.0,
                "iso6391Name":"fr",
                "name":"French"
            },
            "id":"3",
            "warnings":[
                
            ]
        },
        {
            "detectedLanguage":{
                "confidenceScore":1.0,
                "iso6391Name":"zh_chs",
                "name":"Chinese_Simplified"
            },
            "id":"4",
            "warnings":[
                
            ]
        },
        {
            "detectedLanguage":{
                "confidenceScore":1.0,
                "iso6391Name":"ru",
                "name":"Russian"
            },
            "id":"5",
            "warnings":[
                
            ]
        }
    ],
    "errors":[
        
    ],
    "modelVersion":"2020-09-01"
}
```

### <a name="ambiguous-content"></a>Contenido ambiguo

En algunos casos, puede ser difícil eliminar la ambigüedad de los idiomas en función de la entrada. Puede usar el parámetro `countryHint` para especificar un código de país o región [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2). De forma predeterminada, la API usa "US" como valor de countryHint predeterminado; para quitar este comportamiento, puede restablecer este parámetro y configurarlo como una cadena `countryHint = ""` vacía.

Por ejemplo, "impossible" es igual en inglés que en francés, y si se proporciona con un contexto limitado, la respuesta se basará en la sugerencia de país o región de "US". Si se sabe que el origen del texto procede de Francia, eso se puede proporcionar como sugerencia.

**Entrada**

```json
{
    "documents": [
        {
            "id": "1",
            "text": "impossible"
        },
        {
            "id": "2",
            "text": "impossible",
            "countryHint": "fr"
        }
    ]
}
```

El servicio tiene ahora contexto adicional para hacer un mejor juicio: 

**Salida**

```json
{
    "documents":[
        {
            "detectedLanguage":{
                "confidenceScore":0.62,
                "iso6391Name":"en",
                "name":"English"
            },
            "id":"1",
            "warnings":[
                
            ]
        },
        {
            "detectedLanguage":{
                "confidenceScore":1.0,
                "iso6391Name":"fr",
                "name":"French"
            },
            "id":"2",
            "warnings":[
                
            ]
        }
    ],
    "errors":[
        
    ],
    "modelVersion":"2020-09-01"
}
```

Si el analizador no puede analizar la entrada, devuelve `(Unknown)`. Un ejemplo es si se envía un bloque de texto que consta únicamente de números arábigos.

```json
{
    "documents":[
        {
            "detectedLanguage":{
                "confidenceScore":0.0,
                "iso6391Name":"(Unknown)",
                "name":"(Unknown)"
            },
            "id":"1",
            "warnings":[
                
            ]
        }
    ],
    "errors":[
        
    ],
    "modelVersion":"2020-09-01"
}
```

### <a name="mixed-language-content"></a>Contenido en varios idiomas

El contenido en varios idiomas dentro del mismo documento devuelve el idioma con mayor representación en el contenido, pero con una clasificación positiva inferior. La clasificación refleja la fuerza marginal de la evaluación. En el ejemplo siguiente, la entrada es una combinación de inglés, español y francés. El analizador cuenta los caracteres de cada segmento para determinar el idioma predominante.

**Entrada**

```json
{
    "documents": [
        {
            "id": "1",
            "text": "Hello, I would like to take a class at your University. ¿Se ofrecen clases en español? Es mi primera lengua y más fácil para escribir. Que diriez-vous des cours en français?"
        }
    ]
}
```

**Salida**

La salida resultante está formada por el idioma predominante, con una puntuación de menos de 1,0, que indica un nivel de confianza más débil.

```json
{
    "documents":[
        {
            "detectedLanguage":{
                "confidenceScore":0.94,
                "iso6391Name":"es",
                "name":"Spanish"
            },
            "id":"1",
            "warnings":[
                
            ]
        }
    ],
    "errors":[
        
    ],
    "modelVersion":"2020-09-01"
}
```

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo de la detección del idioma con Text Analytics de Azure Cognitive Services. Se explicaron y mostraron los siguientes puntos:

+ [Detección de idioma](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) está disponible para una amplia gama de idiomas, variantes, dialectos y algunos idiomas regionales o culturales.
+ Los documentos JSON del cuerpo de la solicitud incluyen un identificador y texto.
+ La solicitud POST se realiza a un punto de conexión `/languages`, mediante una [clave de acceso y un punto de conexión](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) personalizados, que sean válidos para la suscripción.
+ La salida de respuesta consta de los identificadores de idioma para cada identificador de documento. La salida se puede transmitir a cualquier aplicación que acepte JSON. Entre las aplicaciones de ejemplo se incluyen Excel y Power BI, por nombrar algunas.

## <a name="see-also"></a>Consulte también

* [Información general de Text Analytics](../overview.md)
* [Uso de la biblioteca cliente de Text Analytics](../quickstarts/client-libraries-rest-api.md)
* [Novedades](../whats-new.md)
* [Versiones del modelo](../concepts/model-versioning.md)
