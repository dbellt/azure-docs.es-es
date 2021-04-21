---
title: 'Extracción de datos: LUIS'
description: Extraiga datos de texto de expresiones con intenciones y entidades. Conozca qué tipo de datos se pueden extraer de Language Understanding (LUIS)
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/13/2021
ms.openlocfilehash: dd7d113b1c23a0afec82a346e0f7baa1254ebbed
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107500148"
---
# <a name="extract-data-from-utterance-text-with-intents-and-entities"></a>Extracción de datos de texto de expresiones con intenciones y entidades
LUIS ofrece la capacidad de obtener información de expresiones de lenguaje natural de un usuario. La información se extrae de manera que pueda ser usada por un programa, una aplicación o un bot de chat para tomar medidas. En las secciones siguientes, obtendrá información sobre qué datos se devuelven de las intenciones y entidades con ejemplos de JSON.

Los datos más difíciles de extraer son los datos de aprendizaje automático, porque no corresponden a una coincidencia de texto exacta. El proceso de extracción de datos de [entidades](luis-concept-entity-types.md) de aprendizaje automático debe formar parte del [ciclo de creación](luis-concept-app-iteration.md) hasta que esté seguro de que recibirá los datos que espera.

## <a name="data-location-and-key-usage"></a>Ubicación de los datos y uso de la clave
LUIS extrae datos de la expresión del usuario en el [punto de conexión](luis-glossary.md#endpoint) publicado. La **solicitud HTTPS** (POST o GET) contiene la expresión, así como algunas configuraciones opcionales como los entornos de producción o de almacenamiento provisional.

**Solicitud de punto de conexión de predicción de V2**

`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&q=book 2 tickets to paris`

**Solicitud de punto de conexión de predicción de V3**

`https://westus.api.cognitive.microsoft.com/luis/v3.0-preview/apps/<appID>/slots/<slot-type>/predict?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&query=book 2 tickets to paris`

El valor de `appID` está disponible en la página **Configuración** de la aplicación LUIS, así como parte de la dirección URL (después de `/apps/`) cuando edite esa aplicación LUIS. El valor de `subscription-key` es la clave de punto de conexión que se ha usado para consultar a la aplicación. Aunque puede usar la clave de inicio o creación gratis mientras se familiariza con LUIS, es importante que cambie la clave del punto de conexión por una clave que admita el [uso esperado de LUIS](luis-limits.md#key-limits). La unidad de `timezoneOffset` es minutos.

La **respuesta HTTPS** contiene toda la información de la intención y la entidad que LUIS puede determinar en función del modelo actual publicado de un punto de conexión de producción o de almacenamiento provisional. La dirección URL del punto de conexión se encuentra en el sitio web de [LUIS](luis-reference-regions.md), en la sección **Administrar**, en la página **Claves y puntos de conexión**.

## <a name="data-from-intents"></a>Datos de intenciones
Los datos principales son el **nombre de la intención** de puntuación superior. La respuesta del punto de conexión es:

#### <a name="v2-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V2](#tab/V2)

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V3](#tab/V3)

```JSON
{
  "query": "when do you open next?",
  "prediction": {
    "normalizedQuery": "when do you open next?",
    "topIntent": "GetStoreInfo",
    "intents": {
        "GetStoreInfo": {
            "score": 0.984749258
        }
    }
  },
  "entities": []
}
```

Más información acerca del [punto de conexión de predicción de V3](luis-migration-api-v3.md).

* * *

|Objeto de datos|Tipo de datos|Ubicación de los datos|Value|
|--|--|--|--|
|Intención|String|topScoringIntent.intent|"GetStoreInfo"|

Si el bot de chat o aplicación de llamada a LUIS toma una decisión en función de más de una puntuación de intención, devuelve todas las puntuaciones de intenciones.


#### <a name="v2-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V2](#tab/V2)

Establezca el parámetro querystring, `verbose=true`. La respuesta del punto de conexión es:

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "intents": [
    {
      "intent": "GetStoreInfo",
      "score": 0.984749258
    },
    {
      "intent": "None",
      "score": 0.2040639
    }
  ],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V3](#tab/V3)

Establezca el parámetro querystring, `show-all-intents=true`. La respuesta del punto de conexión es:

```JSON
{
    "query": "when do you open next?",
    "prediction": {
        "normalizedQuery": "when do you open next?",
        "topIntent": "GetStoreInfo",
        "intents": {
            "GetStoreInfo": {
                "score": 0.984749258
            },
            "None": {
                 "score": 0.2040639
            }
        },
        "entities": {
        }
    }
}
```

Más información acerca del [punto de conexión de predicción de V3](luis-migration-api-v3.md).

* * *

Las intenciones se ordenan de mayor a menor puntuación.

|Objeto de datos|Tipo de datos|Ubicación de los datos|Value|Score|
|--|--|--|--|:--|
|Intención|String|intents[0].intent|"GetStoreInfo"|0.984749258|
|Intención|String|intents[1].intent|"None"|0.0168218873|

Si agrega dominios creados previamente, el nombre de la intención indica el dominio, como `Utilties` o `Communication`, así como la intención:

#### <a name="v2-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V2](#tab/V2)

```JSON
{
  "query": "Turn on the lights next monday at 9am",
  "topScoringIntent": {
    "intent": "Utilities.ShowNext",
    "score": 0.07842206
  },
  "intents": [
    {
      "intent": "Utilities.ShowNext",
      "score": 0.07842206
    },
    {
      "intent": "Communication.StartOver",
      "score": 0.0239675418
    },
    {
      "intent": "None",
      "score": 0.0168218873
    }],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Respuesta de punto de conexión de predicción de V3](#tab/V3)

```JSON
{
    "query": "Turn on the lights next monday at 9am",
    "prediction": {
        "normalizedQuery": "Turn on the lights next monday at 9am",
        "topIntent": "Utilities.ShowNext",
        "intents": {
            "Utilities.ShowNext": {
                "score": 0.07842206
            },
            "Communication.StartOver": {
                "score": 0.0239675418
            },
            "None": {
                "score": 0.00085447653
            }
        },
        "entities": []
    }
}
```

Más información acerca del [punto de conexión de predicción de V3](luis-migration-api-v3.md).

* * *

|Domain|Objeto de datos|Tipo de datos|Ubicación de los datos|Value|
|--|--|--|--|--|
|Sectores públicos|Intención|String|intents[0].intent|"<b>Utilities</b>.ShowNext"|
|Comunicación|Intención|String|intents[1].intent|<b>Communication</b>.StartOver"|
||Intención|String|intents[2].intent|"None"|


## <a name="data-from-entities"></a>Datos de entidades
La mayoría de los bots de chat y las aplicaciones necesitan algo más que el nombre de la intención. Estos datos adicionales y opcionales proceden de las entidades que se han detectado en la expresión. Cada tipo de entidad devuelve otra información sobre la coincidencia.

Una sola palabra o frase en una expresión puede coincidir con más de una entidad. En ese caso, se devuelve cada entidad coincidente con su puntuación.

Se devuelven todas las entidades de la matriz **entities** de la respuesta desde el punto de conexión:

## <a name="tokenized-entity-returned"></a>Entidad tokenizada devuelta

Revise la [compatibilidad de los tokens](luis-language-support.md#tokenization) en LUIS.


## <a name="prebuilt-entity-data"></a>Datos de entidades creadas previamente
Las entidades [creadas previamente](luis-concept-entity-types.md) se detectan en función de una coincidencia de expresión regular mediante el proyecto de código abierto [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text). Las entidades creadas previamente se devuelven en la matriz de entidades y usan el nombre de tipo con el prefijo `builtin::`.

## <a name="list-entity-data"></a>Datos de entidad de lista

Las [entidades de lista](reference-entity-list.md) representan un conjunto fijo y cerrado de palabras relacionadas y sus sinónimos. LUIS no detecta valores adicionales para las entidades de lista. Use la característica **Recommend** (Recomendar) para ver sugerencias de palabras nuevas en función de la lista actual. Si hay más de una entidad de lista con el mismo valor, se devolverá cada entidad en la consulta de punto de conexión.

## <a name="regular-expression-entity-data"></a>Datos de entidades de expresiones regulares

Una [entidad de expresión regular](reference-entity-regular-expression.md) extrae una entidad basada en una expresión regular que se proporciona.

## <a name="extracting-names"></a>Extraer nombres
Obtener nombres de una expresión es difícil porque un nombre puede ser casi cualquier combinación de letras y palabras. En función de qué tipo de nombre vaya a extraer, tiene varias opciones. Las sugerencias siguientes no son reglas a seguir, si no más bien instrucciones.

### <a name="add-prebuilt-personname-and-geographyv2-entities"></a>Agregar las entidades PersonName y GeographyV2 creadas previamente

Las entidades [PersonName](luis-reference-prebuilt-person.md) y [GeographyV2](luis-reference-prebuilt-geographyV2.md) están disponibles en algunas [referencias culturales del idioma](luis-reference-prebuilt-entities.md).

### <a name="names-of-people"></a>Nombres de personas

Los nombres de personas pueden tener un pequeño formato en función del idioma y la referencia cultural. Use una entidad **[personName](luis-reference-prebuilt-person.md)** precompilada o una **[entidad sencilla](luis-concept-entity-types.md)** con roles de nombre y apellido.

Si usa la entidad sencilla, asegúrese de proporcionar ejemplos que usen el nombre y el apellido en diferentes partes de la expresión, en expresiones de distintas longitudes y expresiones en todas las intenciones, incluida la intención None. [Revise](./luis-how-to-review-endpoint-utterances.md) las expresiones del punto de conexión de forma regular para etiquetar los nombres que no se predijeron correctamente.

### <a name="names-of-places"></a>Nombres de lugares

Los nombres de ubicaciones se establecen y conocen, por ejemplo, ciudades, condados, estados, provincias y países o regiones. Use la entidad precompilada **[geographyV2](luis-reference-prebuilt-geographyv2.md)** para extraer información de ubicación.

### <a name="new-and-emerging-names"></a>Nombres nuevos y emergentes

Algunas aplicaciones necesitan poder encontrar nombres nuevos y emergentes, como productos o empresas. Estos tipos de nombres son el tipo más difícil de extracción de datos. Empiece con una **[entidad simple](luis-concept-entity-types.md)** y agregue una [lista de frases](luis-concept-feature.md). [Revise](./luis-how-to-review-endpoint-utterances.md) las expresiones del punto de conexión de forma regular para etiquetar los nombres que no se predijeron correctamente.

## <a name="patternany-entity-data"></a>Datos de la entidad Pattern.any

[Pattern.any](reference-entity-pattern-any.md) es un marcador de posición de longitud variable que solo se usa en la expresión de plantilla de un patrón para marcar dónde empieza y acaba la entidad. Debe encontrarse la entidad que se usa en el patrón para poder aplicar el patrón.

## <a name="sentiment-analysis"></a>análisis de opiniones
Si está configurado el análisis de sentimiento durante la [publicación](luis-how-to-publish-app.md#sentiment-analysis), la respuesta JSON de LUIS incluye el análisis de sentimiento. Obtenga más información sobre el análisis de sentimiento en la documentación de [Text Analytics](../text-analytics/index.yml).

## <a name="key-phrase-extraction-entity-data"></a>Datos de entidad de extracción de frases clave
La [entidad de extracción de frases clave](luis-reference-prebuilt-keyphrase.md) devuelve frases clave en la expresión, proporcionadas por [Text Analytics](../text-analytics/index.yml).

## <a name="data-matching-multiple-entities"></a>Datos que coinciden con varias entidades

LUIS devuelve todas las entidades que ha detectado la expresión. Como consecuencia, es posible que el bot de chat tenga que tomar una decisión en función de los resultados.

## <a name="data-matching-multiple-list-entities"></a>Datos que coinciden con varias entidades de lista

Si una palabra o frase coincide con más de una entidad de lista, la consulta de punto de conexión devolverá todas las entidades de lista.

Para la consulta `when is the best time to go to red rock?`, si la aplicación tiene la palabra `red` en más de una lista, LUIS reconocerá todas las entidades y devolverá una matriz de entidades como parte de la respuesta del punto de conexión JSON.

## <a name="next-steps"></a>Pasos siguientes

Vea [Add entities](luis-how-to-add-entities.md) (Agregar entidades) para obtener más información sobre cómo agregar entidades a la aplicación de LUIS.
