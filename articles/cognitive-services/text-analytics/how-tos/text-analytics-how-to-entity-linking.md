---
title: Uso del reconocimiento de entidades con Text Analytics API
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo identificar y eliminar la ambigüedad de la identidad de una entidad encontrada en el texto con la API REST de Text Analytics.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 06/15/2021
ms.author: aahi
ms.openlocfilehash: 989045e552a42d6ebf06d6c0ecadcb1a1f0bb5ff
ms.sourcegitcommit: cc099517b76bf4b5421944bd1bfdaa54153458a0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/09/2021
ms.locfileid: "113549983"
---
# <a name="how-to-use-named-entity-recognition-in-text-analytics"></a>Uso del reconocimiento de entidades con nombre en Text Analytics

Text Analytics API permite tomar el texto no estructurado y devuelve una lista de entidades desambiguadas con vínculos a más información en la web. La API admite el reconocimiento de entidades con nombre (NER) para varias categorías de entidad y la vinculación de entidades.

## <a name="entity-linking"></a>Entity Linking

La vinculación de entidad es la capacidad de identificar y desambiguar la identidad de una entidad que se encuentra en el texto (por ejemplo, determinar si una aparición de la palabra "Marte" hace referencia al planeta o al dios romano de la guerra). Este proceso requiere la presencia de una base de conocimiento en un lenguaje adecuado para vincular entidades reconocidas en el texto. La vinculación de entidades utiliza [Wikipedia](https://www.wikipedia.org/) como base de conocimiento.

## <a name="named-entity-recognition-ner"></a>Reconocimiento de entidades con nombre (NER)

El reconocimiento de entidades con nombre (NER) es la capacidad para identificar diferentes entidades en el texto y clasificarlas en clases o tipos predefinidos, como: persona, ubicación, evento, producto y organización.  

## <a name="personally-identifiable-information-pii"></a>Información de identificación personal

La característica PII forma parte de NER y puede identificar y suprimir entidades confidenciales en texto que están asociadas a una persona individual, como: el número de teléfono, la dirección de correo electrónico, la dirección de correo electrónico y el número de pasaporte.

## <a name="named-entity-recognition-features-and-versions"></a>Características y versiones del reconocimiento de entidades con nombre

| Característica                                                         | NER v3.0 | NER v3.1 |
|-----------------------------------------------------------------|--------|----------|
| Métodos para solicitudes individuales y por lotes                          | X      | X        |
| Reconocimiento de entidades expandidas en varias categorías           | X      | X        |
| Separe los puntos de conexión independientes para enviar las solicitudes de vinculación de entidad y de NER. | X      | X        |
| Reconocimiento de entidades de información personal (`PII`) y de estado (`PHI`)        |        | X        |
| Supresión de `PII`        |        | X        |

Consulte [Compatibilidad de idioma](../language-support.md) para obtener más información.

El reconocimiento de entidades con nombre v3 proporciona detección expandida en varios tipos. Actualmente, NER v3.0 puede reconocer entidades en la [categoría de entidad general](../named-entity-types.md).

La versión 3.1 de Reconocimiento de entidades con nombre incluye las funcionalidades de detección de la versión 3.0 y además: 
* La capacidad para detectar información personal (`PII`) mediante el punto de conexión `v3.1/entities/recognition/pii`. 
* Un parámetro opcional `domain=phi` para detectar información de estado confidencial (`PHI`).
* El [funcionamiento asincrónico](text-analytics-how-to-call-api.md) mediante el punto de conexión `/analyze`.

Para más información, consulte el artículo sobre [categorías de entidad](../named-entity-types.md) y la sección sobre [puntos de conexión de solicitud](#request-endpoints), que encontrará a continuación. Para obtener más información sobre las puntuaciones de confianza, vea la [nota de transparencia de Text Analytics](/legal/cognitive-services/text-analytics/transparency-note?context=/azure/cognitive-services/text-analytics/context/context). 

## <a name="sending-a-rest-api-request"></a>Envío de una solicitud de API REST

### <a name="preparation"></a>Preparación

Debe tener documentos JSON en este formato: identificador, texto, idioma.

Cada documento tiene un tamaño inferior a 5120 caracteres. Puede tener hasta 1000 elementos (identificadores) por colección. La colección se envía en el cuerpo de la solicitud.

### <a name="structure-the-request"></a>Estructurar la solicitud

Cree una solicitud POST. Puede [usar Postman](text-analytics-how-to-call-api.md) o la **consola de pruebas de API** en los siguientes vínculos de referencia para estructurar y enviar rápidamente una. 

> [!NOTE]
> Puede encontrar la clave y el punto de conexión del recurso de Text Analytics en Azure Portal. Están en la página **Inicio rápido** del recurso, en **Administración de recursos**. 


### <a name="request-endpoints"></a>Puntos de conexión de solicitud

#### <a name="version-31"></a>[Versión 3.1](#tab/version-3-1)

El reconocimiento de entidades con nombre `v3.1` usa puntos de conexión independientes para las solicitudes de NER, de PII y de vinculación de entidad. Use un formato de dirección URL a continuación en función de la solicitud.

**Vinculación de entidad**
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1/entities/linking`

[Referencia de la versión 3.1 para el reconocimiento de entidades con nombre para `Linking`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/EntitiesLinking)

**Reconocimiento de entidades con nombre**
* Entidades generales: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1/entities/recognition/general`

[Referencia de la versión 3.1 para el reconocimiento de entidades con nombre para `General`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/EntitiesRecognitionGeneral)

**Información de identificación personal**
* Información personal (`PII`): `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1/entities/recognition/pii`

También se puede usar el parámetro opcional `domain=phi` para detectar información de estado (`PHI`) en el texto. 

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1/entities/recognition/pii?domain=phi`

A partir de `v3.1`, la respuesta JSON incluye una propiedad `redactedText`, que contiene el texto de entrada modificado donde las entidades PII detectadas se reemplazan por `*` en cada carácter de las entidades.

[Referencia de la versión 3.1 para el reconocimiento de entidades con nombre para `PII`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/EntitiesRecognitionPii)

La API intentará detectar las [categorías de entidad enumeradas](../named-entity-types.md?tabs=personal) para un idioma de documento determinado. Si desea especificar qué entidades se detectarán y devolverán, use el parámetro opcional `piiCategories` con las categorías de entidad adecuadas. Este parámetro también puede permitirle detectar entidades que no están habilitadas de forma predeterminada para el lenguaje del documento. En el ejemplo siguiente se detectaría un número del carné de conducir de un conductor francés que podría aparecer en inglés, junto con las entidades predeterminadas en inglés.

> [!TIP]
> Si no incluye `default` al especificar las categorías de entidad, la API solo devolverá las categorías de entidad que especifique.

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1/entities/recognition/pii?piiCategories=default,FRDriversLicenseNumber`

**Operación asincrónica**

A partir de `v3.1`, puede enviar solicitudes NER y de vinculación de entidad de forma asincrónica mediante el punto de conexión `/analyze`.

* Operación asincrónica: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1/analyze`

Consulte [Cómo llamar a la API de Text Analytics](text-analytics-how-to-call-api.md) para obtener información sobre el envío de solicitudes asincrónicas.

#### <a name="version-30"></a>[Versión 3.0](#tab/version-3)

El reconocimiento de entidades con nombre v3 usa puntos de conexión independientes para las solicitudes de NER y de vinculación de entidad. Use un formato de dirección URL a continuación en función de la solicitud:

**Vinculación de entidad**
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/linking`

[Referencia de la versión 3.1 para el reconocimiento de entidades con nombre para `Linking`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/EntitiesRecognitionGeneral)

**Reconocimiento de entidades con nombre**
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/recognition/general`

[Referencia de la versión 3.0 para el reconocimiento de entidades con nombre para `General`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/EntitiesRecognitionGeneral)

---

Establezca un encabezado de la solicitud para incluir la clave de la API Text Analytics. En el cuerpo de la solicitud, proporcione los documentos JSON que tiene preparados.

## <a name="example-requests"></a>Solicitudes de ejemplo

#### <a name="version-31"></a>[Versión 3.1](#tab/version-3-1)

### <a name="example-synchronous-ner-request"></a>Ejemplo de solicitud de NER sincrónica 

El siguiente código JSON es un ejemplo que se podría enviar a la API. El formato de la solicitud es el mismo para las dos versiones de la API.

```json
{
  "documents": [
    {
        "id": "1",
        "language": "en",
        "text": "Our tour guide took us up the Space Needle during our trip to Seattle last week."
    }
  ]
}
```

### <a name="example-synchronous-pii-request"></a>Ejemplo de solicitud de PII sincrónica

El siguiente código JSON es un ejemplo de contenido que se podría enviar a la API para detectar PII en el texto.

```json
{
  "documents": [
    {
        "id": "1",
        "language": "en",
        "text": "You can even pre-order from their online menu at www.contososteakhouse.com, call 312-555-0176 or send email to order@contososteakhouse.com!"
    }
  ]
}
```

### <a name="example-asynchronous-ner-request"></a>Ejemplo de solicitud de NER asincrónica

Si usa el punto de conexión `/analyze` en la [operación asincrónica](text-analytics-how-to-call-api.md), recibirá una respuesta que contiene las tareas que envió a la API.

```json
{
    "displayName": "My Job",
    "analysisInput": {
        "documents": [
            {
                "id": "doc1",
                "text": "It's incredibly sunny outside! I'm so happy"
            },
            {
                "id": "doc2",
                "text": "Pike place market is my favorite Seattle attraction."
            }
        ]
    },
    "tasks": {
        "entityRecognitionTasks": [
            {
                "parameters": {
                    "model-version": "latest"
                }
            }
        ],
        "entityRecognitionPiiTasks": [{
            "parameters": {
                "model-version": "latest"
            }
        }]
    }
}
```

#### <a name="version-30"></a>[Versión 3.0](#tab/version-3)

### <a name="example-synchronous-ner-request"></a>Ejemplo de solicitud de NER sincrónica 

La versión 3.0 solo incluye operaciones sincrónicas. El siguiente código JSON es un ejemplo que se podría enviar a la API. El formato de la solicitud es el mismo para las dos versiones de la API.

```json
{
  "documents": [
    {
        "id": "1",
        "language": "en",
        "text": "Our tour guide took us up the Space Needle during our trip to Seattle last week."
    }
  ]
}
```

---

## <a name="post-the-request"></a>Publicar la solicitud

El análisis se realiza tras la recepción de la solicitud. Consulte el artículo sobre [límites de datos](../overview.md#data-limits) para información sobre el tamaño y el número de solicitudes que puede enviar por minuto y segundo.

La API Text Analytics no tiene estado. No se almacenan datos en la cuenta y los resultados se devuelven inmediatamente en la respuesta.

## <a name="view-results"></a>Vista de resultados

Todas las solicitudes POST devuelven una respuesta con formato JSON con los identificadores y las propiedades de entidad detectadas.

La salida se devuelve inmediatamente. Puede transmitir los resultados a una aplicación que acepte JSON o guardar la salida en un archivo en el sistema local y, a continuación, importarlo en una aplicación que permita ordenar, buscar y manipular los datos. Debido a la compatibilidad con varios idiomas y con Emojis, la respuesta puede contener desplazamientos de texto. Para más información, consulte el artículo en que se indica [cómo procesar desplazamientos de texto](../concepts/text-offsets.md).

### <a name="example-responses"></a>Respuestas de ejemplo

La versión 3 proporciona puntos de conexión independientes para la vinculación de entidad y para NER y PII. Versión 3.1: la versión preliminar incluye un modo de análisis asincrónico. A continuación se muestran las respuestas para esas operaciones. 

#### <a name="version-31"></a>[Versión 3.1](#tab/version-3-1)

### <a name="synchronous-example-results"></a>Resultados sincrónicos de ejemplo

Respuesta de NER general de ejemplo:

```json
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "text": "tour guide",
                    "category": "PersonType",
                    "offset": 4,
                    "length": 10,
                    "confidenceScore": 0.94
                },
                {
                    "text": "Space Needle",
                    "category": "Location",
                    "offset": 30,
                    "length": 12,
                    "confidenceScore": 0.96
                },
                {
                    "text": "Seattle",
                    "category": "Location",
                    "subcategory": "GPE",
                    "offset": 62,
                    "length": 7,
                    "confidenceScore": 1.0
                },
                {
                    "text": "last week",
                    "category": "DateTime",
                    "subcategory": "DateRange",
                    "offset": 70,
                    "length": 9,
                    "confidenceScore": 0.8
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2021-06-01"
}
```

Ejemplo de una respuesta PII:

```json
{
    "documents": [
        {
            "redactedText": "You can even pre-order from their online menu at www.contososteakhouse.com, call ************ or send email to ***************************!",
            "id": "1",
            "entities": [
                {
                    "text": "312-555-0176",
                    "category": "PhoneNumber",
                    "offset": 81,
                    "length": 12,
                    "confidenceScore": 0.8
                },
                {
                    "text": "order@contososteakhouse.com",
                    "category": "Email",
                    "offset": 111,
                    "length": 27,
                    "confidenceScore": 0.8
                },
                {
                    "text": "contososteakhouse",
                    "category": "Organization",
                    "offset": 117,
                    "length": 17,
                    "confidenceScore": 0.45
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2021-01-15"
}
```

Respuesta de vinculación de entidad de ejemplo:

```json
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "bingId": "f8dd5b08-206d-2554-6e4a-893f51f4de7e",
                    "name": "Space Needle",
                    "matches": [
                        {
                            "text": "Space Needle",
                            "offset": 30,
                            "length": 12,
                            "confidenceScore": 0.4
                        }
                    ],
                    "language": "en",
                    "id": "Space Needle",
                    "url": "https://en.wikipedia.org/wiki/Space_Needle",
                    "dataSource": "Wikipedia"
                },
                {
                    "bingId": "5fbba6b8-85e1-4d41-9444-d9055436e473",
                    "name": "Seattle",
                    "matches": [
                        {
                            "text": "Seattle",
                            "offset": 62,
                            "length": 7,
                            "confidenceScore": 0.25
                        }
                    ],
                    "language": "en",
                    "id": "Seattle",
                    "url": "https://en.wikipedia.org/wiki/Seattle",
                    "dataSource": "Wikipedia"
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2021-06-01"
}
```

### <a name="example-asynchronous-result"></a>Resultado asincrónico de ejemplo

```json
{
    "jobId": "f480e1f9-0b61-4d47-93da-240f084582cf",
    "lastUpdateDateTime": "2021-07-06T19:03:15Z",
    "createdDateTime": "2021-07-06T19:02:47Z",
    "expirationDateTime": "2021-07-07T19:02:47Z",
    "status": "succeeded",
    "errors": [],
    "displayName": "My Job",
    "tasks": {
        "completed": 2,
        "failed": 0,
        "inProgress": 0,
        "total": 2,
        "entityRecognitionTasks": [
            {
                "lastUpdateDateTime": "2021-07-06T19:03:15.212633Z",
                "taskName": "NamedEntityRecognition_latest",
                "state": "succeeded",
                "results": {
                    "documents": [
                        {
                            "id": "doc1",
                            "entities": [],
                            "warnings": []
                        },
                        {
                            "id": "doc2",
                            "entities": [
                                {
                                    "text": "Pike place market",
                                    "category": "Location",
                                    "offset": 0,
                                    "length": 17,
                                    "confidenceScore": 0.95
                                },
                                {
                                    "text": "Seattle",
                                    "category": "Location",
                                    "subcategory": "GPE",
                                    "offset": 33,
                                    "length": 7,
                                    "confidenceScore": 0.99
                                }
                            ],
                            "warnings": []
                        }
                    ],
                    "errors": [],
                    "modelVersion": "2021-06-01"
                }
            }
        ],
        "entityRecognitionPiiTasks": [
            {
                "lastUpdateDateTime": "2021-07-06T19:03:03.2063832Z",
                "taskName": "PersonallyIdentifiableInformation_latest",
                "state": "succeeded",
                "results": {
                    "documents": [
                        {
                            "redactedText": "It's incredibly sunny outside! I'm so happy",
                            "id": "doc1",
                            "entities": [],
                            "warnings": []
                        },
                        {
                            "redactedText": "Pike place market is my favorite Seattle attraction.",
                            "id": "doc2",
                            "entities": [],
                            "warnings": []
                        }
                    ],
                    "errors": [],
                    "modelVersion": "2021-01-15"
                }
            }
        ]
    }
}
```


#### <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Respuesta de NER general de ejemplo:
```json
{
  "documents": [
    {
      "id": "1",
      "entities": [
        {
          "text": "tour guide",
          "category": "PersonType",
          "offset": 4,
          "length": 10,
          "confidenceScore": 0.45
        },
        {
          "text": "Space Needle",
          "category": "Location",
          "offset": 30,
          "length": 12,
          "confidenceScore": 0.38
        },
        {
          "text": "trip",
          "category": "Event",
          "offset": 54,
          "length": 4,
          "confidenceScore": 0.78
        },
        {
          "text": "Seattle",
          "category": "Location",
          "subcategory": "GPE",
          "offset": 62,
          "length": 7,
          "confidenceScore": 0.78
        },
        {
          "text": "last week",
          "category": "DateTime",
          "subcategory": "DateRange",
          "offset": 70,
          "length": 9,
          "confidenceScore": 0.8
        }
      ],
      "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2020-04-01"
}
```

---


## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo de vinculación de entidad mediante Text Analytics de Cognitive Services. En resumen:

* Los documentos JSON del cuerpo de la solicitud incluyen un identificador, un texto y el código de idioma.
* Las solicitudes POST se envían a uno o varios puntos de conexión, con una [clave de acceso y un punto de conexión](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) personalizados, que son válidos para la suscripción.
* La salida de respuesta, que consta de entidades vinculadas (incluidas las puntuaciones de confianza, los desplazamientos y los vínculos web de cada id. de documento) se puede usar en cualquier aplicación.

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Text Analytics](../overview.md)
* [Uso de la biblioteca cliente de Text Analytics](../quickstarts/client-libraries-rest-api.md)
* [Versiones del modelo](../concepts/model-versioning.md)
* [Novedades](../whats-new.md)
