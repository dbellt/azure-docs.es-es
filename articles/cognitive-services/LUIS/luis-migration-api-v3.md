---
title: Cambios de punto de conexión para predicciones en la versión 3 de la API
description: Se cambiaron las API v3 del punto de conexión para la predicción de consultas. Use esta guía para aprender a migrar a la versión 3 de las API de punto de conexión.
ms.service: cognitive-services
author: aahill
ms.manager: nitinme
ms.author: aahi
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/28/2021
ms.openlocfilehash: 6eb5e54f2c1d8a4e7d05204fcfa111ddcb9f951a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692683"
---
# <a name="prediction-endpoint-changes-for-v3"></a>Cambios de punto de conexión para predicciones para la versión 3

Se cambiaron las API v3 del punto de conexión para la predicción de consultas. Use esta guía para aprender a migrar a la versión 3 de las API de punto de conexión. Actualmente no hay ninguna fecha antes de la que sea necesario completar la migración.

**Estado de disponibilidad general**: esta versión 3 de la API incluye cambios importantes en la solicitud JSON y la respuesta en comparación con la versión 2 de la API.

La versión 3 de la API proporciona las características nuevas siguientes:

* [Entidades externas](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time)
* [Listas dinámicas](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Cambios de JSON en las entidades precompiladas](#prebuilt-entity-changes)

La [solicitud](#request-changes) y [respuesta](#response-changes) del punto de conexión para predicciones recibió cambios importantes para admitir las nuevas características enumeradas anteriormente, incluido lo siguiente:

* [Cambios al objeto de respuesta](#top-level-json-changes)
* [Referencias al nombre del rol de entidad en lugar del nombre de entidad](#entity-role-name-instead-of-entity-name)
* [Propiedades para marcar las entidades en expresiones](#marking-placement-of-entities-in-utterances)

Puede consultar la [documentación de referencia](https://aka.ms/luis-api-v3) para la v3.

## <a name="v3-changes-from-preview-to-ga"></a>Cambios en la versión 3 desde la versión preliminar a la disponibilidad general

La versión 3 realizó los cambios siguientes como parte de la migración a la disponibilidad general:

* Estas entidades creadas previamente tienen respuestas JSON distintas:
    * [OrdinalV1](luis-reference-prebuilt-ordinal.md)
    * [GeographyV2](luis-reference-prebuilt-geographyv2.md)
    * [DatetimeV2](luis-reference-prebuilt-datetimev2.md)
    * Nombre de clave de unidad medible de `units` a `unit`

* Cambio de JSON del cuerpo de la solicitud:
    * de `preferExternalEntities` a `preferExternalEntities`
    * parámetro `score` opcional para entidades externas

* Cambios de JSON del cuerpo de la respuesta:
    * se quitó `normalizedQuery`

## <a name="suggested-adoption-strategy"></a>Estrategia de adopción sugerida

Si usa Bot Framework, Bing Spell Check V7 o solo quiere migrar la creación de aplicaciones de LUIS, siga usando la versión 2 del punto de conexión.

Si sabe que ninguna de las aplicaciones cliente o integraciones (Bot Framework y Bing Spell Check V7) se ven afectadas y está familiarizado con la migración de la creación de aplicaciones de LUIS y el punto de conexión para predicciones al mismo tiempo, empiece a usar la versión 3 del punto de conexión para predicciones. La versión 2 del punto de conexión para predicciones seguirá disponible y es una buena estrategia de retroceso.

Para obtener información sobre el uso de Bing Spell Check API, consulte [Corrección de palabras mal escritas.](luis-tutorial-bing-spellcheck.md)


## <a name="not-supported"></a>No compatible

### <a name="bot-framework-and-azure-bot-service-client-applications"></a>Aplicaciones cliente de Azure Bot Service y Bot Framework

Siga usando la versión 2 de la API del punto de conexión para predicciones hasta que aparezca la versión 4.7 de Bot Framework.


## <a name="endpoint-url-changes"></a>Cambios en la dirección URL del punto de conexión

### <a name="changes-by-slot-name-and-version-name"></a>Cambios por nombre de ranura y nombre de versión

[Se cambió el formato para la llamada HTTP al punto de conexión v3.](developer-reference-resource.md#rest-endpoints)

Si quiere realizar una consulta por visión, primero debe [publicarla a través de la API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c3b) con `"directVersionPublish":true`. Consulte el punto de conexión que hace referencia al Id. de versión en lugar del nombre de la ranura.

|Valores válidos para `SLOT-NAME`|
|--|
|`production`|
|`staging`|

## <a name="request-changes"></a>Solicitud de cambios

### <a name="query-string-changes"></a>Cambios en la cadena de consulta

[!INCLUDE [V3 query params](./includes/v3-prediction-query-params.md)]

### <a name="v3-post-body"></a>Cuerpo de POST de la versión 3

```JSON
{
    "query":"your utterance here",
    "options":{
        "datetimeReference": "2019-05-05T12:00:00",
        "preferExternalEntities": true
    },
    "externalEntities":[],
    "dynamicLists":[]
}
```

|Propiedad|Tipo|Versión|Valor predeterminado|Propósito|
|--|--|--|--|--|
|`dynamicLists`|array|Solo v3|No se requiere.|Las [listas dinámicas](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time) permiten extender una entidad de lista existente entrenada y publicada, ya presente en la aplicación de LUIS.|
|`externalEntities`|array|Solo v3|No se requiere.|Las [entidades externas](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time) permiten que la aplicación de LUIS identifique y etiquete las entidades durante el tiempo de ejecución, que pueden usarse como características para las entidades existentes. |
|`options.datetimeReference`|string|Solo v3|Sin valor predeterminado|Permite determinar el [desplazamiento de datetimeV2](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity). El formato de datetimeReference es [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).|
|`options.preferExternalEntities`|boolean|Solo v3|false|Especifica si la [entidad externa (con el mismo nombre que la entidad existente)](schema-change-prediction-runtime.md#override-existing-model-predictions) del usuario o la entidad existente en el modelo se usa para la predicción. |
|`query`|string|Solo v3|Necesario.|**En la versión 2**, la expresión que va a predecirse se almacena en el parámetro `q`. <br><br>**En la versión 3**, la funcionalidad se pasa en el parámetro `query`.|

## <a name="response-changes"></a>Cambios de respuesta

El elemento JSON de la respuesta de consulta se cambió para permitir el acceso mediante programación a los datos usados con más frecuencia.

### <a name="top-level-json-changes"></a>Cambios JSON de nivel superior



Las siguientes son las principales propiedades JSON para la v2, cuando `verbose` está establecido en true, de forma que devuelve todas las intenciones y sus puntuaciones en la propiedad `intents`:

```JSON
{
    "query":"this is your utterance you want predicted",
    "topScoringIntent":{},
    "intents":[],
    "entities":[],
    "compositeEntities":[]
}
```

Las principales propiedades JSON para la versión 3 son:

```JSON
{
    "query": "this is your utterance you want predicted",
    "prediction":{
        "topIntent": "intent-name-1",
        "intents": {},
        "entities":{}
    }
}
```

El objeto `intents` es una lista sin ordenar. No suponga que el primer elemento secundario en `intents` corresponde a `topIntent`. En su lugar, use el valor `topIntent` para buscar la puntuación:

```nodejs
const topIntentName = response.prediction.topIntent;
const score = intents[topIntentName];
```

Los cambios en el esquema JSON de la respuesta permiten lo siguiente:

* Una clara distinción entre la expresión original, `query`, y la predicción devuelta, `prediction`.
* Fácil acceso mediante programación a los datos previstos. En lugar de realizar la enumeración a través de una matriz en la versión 2, puede acceder a los valores por **nombre** en ambas intenciones y entidades. Para los roles de entidad prevista, se devuelve el nombre del rol porque es único en toda la aplicación.
* Los tipos de datos, si están establecidos, se respetan. Los valores numéricos ya no se devuelven como cadenas.
* Distinción entre la información de predicciones de máxima prioridad y los metadatos adicionales, que se devuelven en el objeto `$instance`.

### <a name="entity-response-changes"></a>Cambios en la respuesta de la entidad

#### <a name="marking-placement-of-entities-in-utterances"></a>Selección de la ubicación de las entidades en las expresiones

**En la versión 2**, una entidad se marca dentro de una expresión con `startIndex` y `endIndex`.

**En la versión 3**, la entidad se marca con `startIndex` y `entityLength`.

#### <a name="access-instance-for-entity-metadata"></a>Acceso a `$instance` para obtener metadatos de entidad

Si tiene metadatos de entidad, la cadena de consulta debe usar la marca `verbose=true` y la respuesta contiene los metadatos en el objeto `$instance`. En las respuestas JSON de las secciones siguientes se muestran ejemplos.

#### <a name="each-predicted-entity-is-represented-as-an-array"></a>Todas las entidades de predicción se representan como una matriz.

El objeto `prediction.entities.<entity-name>` contiene una matriz, dado que cada entidad se puede predecir más de una vez en la expresión.

<a name="prebuilt-entities-with-new-json"></a>

#### <a name="prebuilt-entity-changes"></a>Cambios de entidad generados previamente

El objeto de respuesta de la versión 3 incluye cambios en las entidades compiladas previamente. Revise las [entidades compiladas previamente específicas](luis-reference-prebuilt-entities.md) para más información.

#### <a name="list-entity-prediction-changes"></a>Cambios en la predicción de entidad de lista

El código JSON de una predicción de entidad la lista se cambió para que sea una matriz de matrices:

```JSON
"entities":{
    "my_list_entity":[
        ["canonical-form-1","canonical-form-2"],
        ["canonical-form-2"]
    ]
}
```
Cada matriz interior corresponde al texto dentro de la expresión. El objeto interior es una matriz porque el mismo texto puede aparecer en más de una sublista de una entidad de lista.

Cuando se realiza una asignación del objeto `entities` y el objeto `$instance`, se conserva el orden de los objetos para las predicciones de entidad de lista.

```nodejs
const item = 0; // order preserved, use same enumeration for both
const predictedCanonicalForm = entities.my_list_entity[item];
const associatedMetadata = entities.$instance.my_list_entity[item];
```

#### <a name="entity-role-name-instead-of-entity-name"></a>Nombre del rol de entidad en lugar del nombre de entidad

En la versión 2, la matriz `entities` devuelve todas las entidades previstas, cuyo el nombre de entidad es el identificador único. En la versión 3, si la entidad usa roles y la predicción es para un rol de entidad, el identificador principal es el nombre del rol. Esto es posible porque los nombres de rol de entidad deben ser únicos en toda la aplicación, incluidos otros nombres del modelo (entidad, intención).

En el siguiente ejemplo: considere la posibilidad de que una expresión incluya el texto `Yellow Bird Lane`. Este texto se prevé como el rol `Destination` de una entidad de `Location` personalizada.

|Texto de expresión|Nombre de entidad|Nombre de rol|
|--|--|--|
|`Yellow Bird Lane`|`Location`|`Destination`|

En la versión 2, la entidad se identifica mediante el _nombre de entidad_, con el rol como una propiedad del objeto:

```JSON
"entities":[
    {
        "entity": "Yellow Bird Lane",
        "type": "Location",
        "startIndex": 13,
        "endIndex": 20,
        "score": 0.786378264,
        "role": "Destination"
    }
]
```

En la versión 3, se hace referencia a la entidad mediante el _rol de entidad_ si la predicción es para el rol:

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ]
}
```

En la versión 3, el mismo resultado con la marca `verbose` para devolver los metadatos de entidad:

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ],
    "$instance":{
        "Destination": [
            {
                "role": "Destination",
                "type": "Location",
                "text": "Yellow Bird Lane",
                "startIndex": 25,
                "length":16,
                "score": 0.9837309,
                "modelTypeId": 1,
                "modelType": "Entity Extractor"
            }
        ]
    }
}
```

<a name="external-entities-passed-in-at-prediction-time"></a>
<a name="override-existing-model-predictions"></a>

## <a name="extend-the-app-at-prediction-time"></a>Extensión de la aplicación en el momento de la predicción

Conozca los [conceptos](schema-change-prediction-runtime.md) sobre cómo ampliar la aplicación en el entorno de ejecución de predicción.


## <a name="next-steps"></a>Pasos siguientes

Use la documentación de la API v3 para actualizar las llamadas REST existentes al [punto de conexión](https://westcentralus.dev.cognitive.microsoft.com/docs/services/luis-endpoint-api-v3-0/operations/5cb0a9459a1fe8fa44c28dd8) de LUIS.
