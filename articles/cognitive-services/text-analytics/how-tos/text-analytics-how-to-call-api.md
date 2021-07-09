---
title: Llamada a la API Text Analytics
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo se puede llamar a la API REST y a Postman de Text Analytics de Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 06/10/2021
ms.author: aahi
ms.openlocfilehash: b7ad200bba527d0b4b841483175b2672d94f162e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962857"
---
# <a name="how-to-call-the-text-analytics-rest-api"></a>Cómo llamar a la API REST de Text Analytics

En este artículo, se usan la API REST de Text Analytics y [Postman](https://www.postman.com/downloads/) para mostrar los conceptos clave. La API proporciona varios puntos de conexión sincrónicos y asincrónicos para usar las características del servicio. 

## <a name="create-a-text-analytics-resource"></a>Creación de un recurso de Text Analytics

> [!NOTE]
> * Necesitará un recurso de Text Analytics mediante un [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) Estándar (S) si desea utilizar los puntos de conexión `/analyze` o `/health`. El punto de conexión `/analyze` está incluido en el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/).

Antes de usar la API de Text Analytics, debe crear un recurso de Azure con una clave y un punto de conexión para las aplicaciones. 

1.  En primer lugar, vaya a [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) y cree un recurso de Text Analytics, en caso de que no tenga ninguno. Elija un [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/).

2.  Seleccione la región que quiera que use su punto de conexión.

3.  Cree el recurso de Text Analytics y vaya a la sección "Claves y punto de conexión" en Administración de recursos a la izquierda de la página. Copie la clave que va a usar más adelante cuando llame a las API. La agregará más adelante como valor en el encabezado `Ocp-Apim-Subscription-Key` .

4. Para comprobar el número de registros de texto que se han enviado mediante el recurso de Text Analytics:

    1. Vaya al recurso de Text Analytics en Azure Portal. 
    2. Haga clic en **Métricas**, que se encuentra en **Supervisión**, en el menú de navegación izquierdo. 
    3. Seleccione *Registros de texto procesados* en el cuadro desplegable de **Métrica**.
    
Un registro de texto es una unidad de texto de entrada de 1000 caracteres como máximo.  Por ejemplo, 1500 caracteres enviados como texto de entrada contarán como 2 registros de texto.

## <a name="change-your-pricing-tier"></a>Cambio del plan de tarifa 

Si tiene un recurso de Text Analytics existente que usa el plan de tarifa S0 a S4, debe actualizarlo para que use el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) Estándar. Los planes de tarifa S0 a S4 se retirarán. Para actualizar los precios de los recursos:

1. Vaya al recurso de Text Analytics en [Azure Portal](https://portal.azure.com/).
2. En el menú de navegación de la izquierda, seleccione **Plan de tarifa**. Estará debajo de **ADMINISTRACIÓN DE RECURSOS**. 
3. Elija el plan de tarifa Estándar (S). Después, haga clic en **Seleccionar**.

También puede crear un nuevo recurso de Text Analytics con el plan de tarifa Estándar (S) y migrar las aplicaciones para que usen las credenciales del nuevo recurso. 

## <a name="using-the-api-synchronously"></a>Uso de la API de forma sincrónica

Puede llamar a Text Analytics sincrónicamente (para escenarios de baja latencia). Recuerde que tiene que llamar a cada API (característica) por separado cuando use la API sincrónica. Si necesita llamar a varias características, consulte la sección siguiente sobre cómo llamar a Text Analytics de forma asincrónica. 

## <a name="using-the-api-asynchronously"></a>Uso de la API de forma asincrónica

La API Text Analytics v3.1-preview.5 proporciona dos puntos de conexión asincrónicos: 

* El punto de conexión `/analyze` para Text Analytics permite analizar el mismo conjunto de documentos de texto con varias características de análisis de texto en una llamada API. Anteriormente, para usar varias características sería preciso realizar llamadas API independientes para cada operación. Tenga en cuenta esta funcionalidad cuando necesite analizar grandes conjuntos de documentos con más de una característica de Text Analytics.

* El punto de conexión `/health` para Text Analytics fo Health, que puede extraer y etiquetar información médica relevante de documentos clínicos.  

En la tabla siguiente puede ver qué características se pueden usar de forma asincrónica. Tenga en cuenta que desde el punto de conexión `/analyze` solo se puede llamar a unas pocas características. 

| Característica | Sincrónica | Asincrónica |
|--|--|--|
| Detección de idiomas | ✔ |  |
| análisis de opiniones | ✔ | ✔* |
| Minería de opiniones | ✔ | ✔* |
| Extracción de la frase clave | ✔ | ✔* |
| Reconocimiento de entidades con nombre (se incluyen PII y PHI) | ✔ | ✔* |
| Vinculación de entidad | ✔ | ✔* |
| Text Analytics for Health (contenedor) | ✔ |  |
| Text Analytics for Health (API) |  | ✔  |

`*`: se le llama de forma asincrónica a través del punto de conexión `/analyze` .


[!INCLUDE [text-analytics-api-references](../includes/text-analytics-api-references.md)]

<a name="json-schema"></a>

## <a name="api-request-formats"></a>Formato de solicitudes de API

Puede enviar llamadas sincrónicas y asincrónicas a la API de Text Analytics.

#### <a name="synchronous"></a>[Sincrónica](#tab/synchronous)

### <a name="synchronous-requests"></a>Solicitudes sincrónicas

El formato de las solicitudes de API es el mismo para todas las operaciones sincrónicas. Los documentos se envían en un objeto JSON como texto no estructurado sin formato. XML no se admite. El esquema JSON consta de los elementos que se describen a continuación.

| Elemento | Valores válidos | ¿Necesario? | Uso |
|---------|--------------|-----------|-------|
|`id` |El tipo de datos es una cadena, pero en la práctica, los identificadores de documento tienden a ser números enteros. | Obligatorio | El sistema usa los identificadores que proporcione para estructurar la salida. Se generan códigos de idioma, frases clave y puntuaciones de opinión para cada identificador de la solicitud.|
|`text` | Texto sin formato no estructurado, hasta 5120 caracteres. | Obligatorio | Para la detección de idioma, el texto se puede expresar en cualquier idioma. Para el análisis de opiniones, la extracción de frases clave y la identificación de entidades, el texto debe estar en un [idioma admitido](../language-support.md). |
|`language` | Código [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) de dos caracteres para un [idioma admitido](../language-support.md) | Varía | Análisis de opiniones, extracción de frases clave, detección de idioma y vinculación de entidades; opcional para la detección de idioma. No hay ningún error si se excluye, pero sin él se debilita el análisis. El código de idioma se debe corresponder al `text` que proporcione. |

A continuación verá un ejemplo de una solicitud de API para los puntos de conexión de Text Analytics sincrónicos. 

```json
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Sample text to be sent to the text analytics api."
    }
  ]
}
```

#### <a name="asynchronous"></a>[Asincrónica](#tab/asynchronous)

### <a name="asynchronous-requests-to-the-analyze-endpoint"></a>Solicitudes asincrónicas al punto de conexión `/analyze`

> [!NOTE]
> La versión preliminar más reciente de la biblioteca cliente de Text Analytics permite llamar a operaciones asincrónicas de Analyze mediante un objeto de cliente. Puede encontrar ejemplos en GitHub:
* [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
* [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
* [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)

El punto de conexión `/analyze` le permite elegir cuál de las características compatibles de Text Analytics desea usar en una llamada API individual. Este punto de conexión actualmente admite:

* Extracción de frases clave 
* Reconocimiento de entidades con nombre (se incluyen PII y PHI)
* Entity Linking
* Análisis de sentimiento
* Minería de opiniones

| Elemento | Valores válidos | ¿Necesario? | Uso |
|---------|--------------|-----------|-------|
|`displayName` | String | Opcional | Se usa como nombre para mostrar para el identificador único en el trabajo.|
|`analysisInput` | Incluye el campo `documents` siguiente | Obligatorio | Contiene la información de los documentos que desea enviar. |
|`documents` | Incluye los campos `id` y `text` siguientes | Obligatorio | Contiene información de cada documento que se envía y el texto sin formato del documento. |
|`id` | String | Obligatorio | Los identificadores que proporcione se usan para estructurar la salida. |
|`text` | Texto sin formato no estructurado, hasta 125 000 caracteres. | Obligatorio | Debe estar en inglés, que es el único idioma que se admite actualmente. |
|`tasks` | Incluye las siguientes características de Text Analytics: `entityRecognitionTasks`, `entityLinkingTasks`, `keyPhraseExtractionTasks`, `entityRecognitionPiiTasks` o `sentimentAnalysisTasks`. | Obligatorio | Una o varias de las características Text Analytics que desea usar. Tenga en cuenta que `entityRecognitionPiiTasks` tiene un parámetro opcional `domain` que se puede establecer en `pii` o `phi` y `pii-categories` para la detección de los tipos de entidad seleccionados. Si el parámetro `domain` no se especifica, el valor predeterminado del sistema es `pii`. De forma similar, `sentimentAnalysisTasks` tiene el parámetro booleano `opinionMining` para incluir los resultados de minería de opiniones en la salida de Análisis de sentimiento. |
|`parameters` | Incluye los campos `model-version` y `stringIndexType` siguientes | Obligatorio | Este campo está incluido en las tareas de características anteriores que elija. Contienen información no solo sobre la versión del modelo que desea utilizar, sino también sobre el tipo de índice. |
|`model-version` | String | Obligatorio | Especifique la versión del modelo que desea usar que se va a llamar.  |
|`stringIndexType` | String | Obligatorio | Especifique el descodificador de texto que se ajuste a su entorno de programación.  Los tipos que se admiten son: `textElement_v8` (predeterminado), `unicodeCodePoint` y `utf16CodeUnit`. Para más información, consulte el [artículo sobre desplazamientos de texto](../concepts/text-offsets.md#offsets-in-api-version-31-preview).  |
|`domain` | String | Opcional | Solo se aplica como parámetro a la tarea `entityRecognitionPiiTasks` y se puede establecer en `pii` o `phi`. Si no se especifica, su valor predeterminado es `pii`.  |

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
                "text": "Pike place market is my favorite Seattle attraction. The shops have very good food."
            }
        ]
    },
    "tasks": {
        "entityRecognitionTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElement_v8",
                    "loggingOptOut": "false"
                }
            }
        ],
        "entityRecognitionPiiTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElement_v8",
                    "loggingOptOut": "true",
                    "domain": "phi",
                    "piiCategories":["default"]
                }
            }
        ],
        "entityLinkingTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElement_v8",
                    "loggingOptOut": "false"
                }
            }
        ],
        "keyPhraseExtractionTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "loggingOptOut": "false"
                }
            }
        ],
        "sentimentAnalysisTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElement_v8",
                    "loggingOptOut": "false",
                    "opinionMining": "true"
                }
            }
        ]
    }
}

```

### <a name="asynchronous-requests-to-the-health-endpoint"></a>Solicitudes asincrónicas al punto de conexión `/health`

El formato de las solicitudes de API a la API hospedada de Text Analytics for Health es el mismo que el de su contenedor. Los documentos se envían en un objeto JSON como texto no estructurado sin formato. XML no se admite. El esquema JSON consta de los elementos que se describen a continuación.  Rellene y envíe el [formulario de solicitud de Cognitive Services](https://aka.ms/csgate) para solicitar acceso a la versión preliminar pública de Text Analytics for Health. El uso de Text Analytics for Health no se facturará. 

| Elemento | Valores válidos | ¿Necesario? | Uso |
|---------|--------------|-----------|-------|
|`id` |El tipo de datos es una cadena, pero en la práctica, los identificadores de documento tienden a ser números enteros. | Obligatorio | El sistema usa los identificadores que proporcione para estructurar la salida. |
|`text` | Texto sin formato no estructurado, hasta 5120 caracteres. | Obligatorio | Tenga en cuenta que en la actualidad solo se admite texto en inglés. |
|`language` | Código [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) de dos caracteres para un [idioma admitido](../language-support.md) | Obligatorio | Actualmente solo se admite `en`. |

A continuación verá un ejemplo de una solicitud de API para los puntos de conexión de Text Analytics for Health. 

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Subject was administered 100mg remdesivir intravenously over a period of 120 min"
    }
  ]
}
```

---

>[!TIP]
> Consulte el artículo [Límites de datos y velocidad de la API Text Analytics](../concepts/data-limits.md) para obtener información sobre los límites de tamaño y velocidad para enviar datos a la API Text Analytics.


## <a name="set-up-a-request"></a>Configuración de una solicitud 

En Postman (o cualquier otra herramienta para pruebas de API web), agregue el punto de conexión de la característica que desea usar. Use la tabla siguiente para buscar el formato de punto de conexión adecuado y reemplace `<your-text-analytics-resource>` por el punto de conexión del recurso. Por ejemplo:

`https://my-resource.cognitiveservices.azure.com/text/analytics/v3.0/languages`

#### <a name="synchronous"></a>[Sincrónica](#tab/synchronous)

### <a name="endpoints-for-sending-synchronous-requests"></a>Puntos de conexión para enviar solicitudes sincrónicas

| Característica | Tipo de solicitud | Puntos de conexión de recursos |
|--|--|--|
| Detección de idiomas | POST | `<your-text-analytics-resource>/text/analytics/v3.0/languages` |
| Análisis de sentimiento | POST | `<your-text-analytics-resource>/text/analytics/v3.0/sentiment` |
| Minería de opiniones | POST | `<your-text-analytics-resource>/text/analytics/v3.1-preview.5/sentiment?opinionMining=true` |
| Extracción de frases clave | POST | `<your-text-analytics-resource>/text/analytics/v3.0/keyPhrases` |
| Reconocimiento de entidades con nombre: general | POST | `<your-text-analytics-resource>/text/analytics/v3.0/entities/recognition/general` |
| Reconocimiento de entidades con nombre: PII | POST | `<your-text-analytics-resource>/text/analytics/v3.1-preview.5/entities/recognition/pii` |
| Reconocimiento de entidades con nombre: PHI | POST |  `<your-text-analytics-resource>/text/analytics/v3.1-preview.5/entities/recognition/pii?domain=phi` |
| Entity Linking | POST | `<your-text-analytics-resource>/text/analytics/v3.0/entities/linking` |

#### <a name="asynchronous"></a>[Asincrónica](#tab/asynchronous)

### <a name="endpoints-for-sending-asynchronous-requests-to-the-analyze-endpoint"></a>Puntos de conexión para enviar solicitudes asincrónicas al punto de conexión `/analyze`

| Característica | Tipo de solicitud | Puntos de conexión de recursos |
|--|--|--|
| Envío de trabajo de análisis | POST | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.5/analyze` |
| Obtención del estado y los resultados del análisis | GET | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.5/analyze/jobs/<Operation-Location>` |

### <a name="endpoints-for-sending-asynchronous-requests-to-the-health-endpoint"></a>Puntos de conexión para enviar solicitudes asincrónicas al punto de conexión `/health`

| Característica | Tipo de solicitud | Puntos de conexión de recursos |
|--|--|--|
| Envío del trabajo de Text Analytics for Health  | POST | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.5/entities/health/jobs` |
| Obtención del estado y resultados del trabajo | GET | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.5/entities/health/jobs/<Operation-Location>` |
| Cancelar trabajo | DELETE | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.5/entities/health/jobs/<Operation-Location>` |

--- 

Una vez que tenga el punto de conexión, en Postman (o en otra herramienta para pruebas de API web):

1. Elija el tipo de solicitud de la característica que desea usar.
2. Pegue el punto de conexión de la operación adecuada que desee de la tabla anterior.
3. Establezca los tres encabezados de solicitud:

   + `Ocp-Apim-Subscription-Key`: la clave de acceso, se obtiene en Azure Portal
   + `Content-Type`: application/json
   + `Accept`: application/json

    Si utiliza Postman, la solicitud debe ser similar a la captura de pantalla siguiente, siempre que el punto de conexión sea `/keyPhrases`.
    
    ![Captura de pantalla de solicitud con el punto de conexión y los encabezados](../media/postman-request-keyphrase-1.png)
    
4. Elija **raw** (sin formato) como formato en **Body** (Cuerpo).
    
    ![Captura de pantalla de la solicitud con la configuración del cuerpo](../media/postman-request-body-raw.png)

5. Pegue algunos documentos JSON con un formato válido. Use los ejemplos de la sección **Formato de solicitud de API** anterior y para más información y ejemplos, vea los temas siguientes:

      + [Detección de idioma](text-analytics-how-to-language-detection.md)
      + [Extracción de frases clave](text-analytics-how-to-keyword-extraction.md)
      + [Análisis de opiniones](text-analytics-how-to-sentiment-analysis.md)
      + [Reconocimiento de entidades](text-analytics-how-to-entity-linking.md)

## <a name="send-the-request"></a>Envío de la solicitud

Envíe la solicitud de API. Si realizó la llamada a un punto de conexión sincrónico, la respuesta se mostrará inmediatamente, en forma de un documento JSON individual, con un elemento para cada identificador de documento proporcionado en la solicitud.

Si realizó la llamada a los puntos de conexión `/analyze` o `/health` asincrónicos, compruebe que recibió un código de respuesta 202. Tendrá que obtener la respuesta para ver los resultados:

1. En la respuesta de la API, busque `Operation-Location` en el encabezado, que identifica el trabajo que envió a la API. 
2. Cree una solicitud GET para el punto de conexión que usó. En la [tabla anterior](#set-up-a-request) encontrará el formato del punto de conexión.Consulte la [documentación de referencia de la API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-5/operations/AnalyzeStatus). Por ejemplo:

    `https://my-resource.cognitiveservices.azure.com/text/analytics/v3.1-preview.5/analyze/jobs/<Operation-Location>`

3. Agregue `Operation-Location` a la solicitud.

4. La respuesta será un solo documento JSON, con un elemento para cada identificador de documento proporcionado en la solicitud.

Tenga en cuenta que para las operaciones asincrónicas `/analyze` o `/health`, los resultados de la solicitud GET en el paso 2 anterior están disponibles durante 24 horas desde el momento en que se creó el trabajo.  Esta hora se indica mediante el valor `expirationDateTime` de la respuesta de GET.  Después de este período de tiempo, los resultados se purgan y ya no están disponibles para la recuperación.    

## <a name="example-api-responses"></a>Respuestas de ejemplo de la API
 
# <a name="synchronous"></a>[Sincrónica](#tab/synchronous)

### <a name="example-responses-for-synchronous-operation"></a>Respuestas de ejemplo para la operación sincrónica

Las respuestas sincrónicas del punto de conexión variarán en función del punto de conexión que se use. Para ver algunas respuestas de ejemplo, consulte los siguientes artículos.

+ [Detección de idioma](text-analytics-how-to-language-detection.md#step-3-view-the-results)
+ [Extracción de frases clave](text-analytics-how-to-keyword-extraction.md#step-3-view-results)
+ [Análisis de opiniones](text-analytics-how-to-sentiment-analysis.md#view-the-results)
+ [Reconocimiento de entidades](text-analytics-how-to-entity-linking.md#view-results)

# <a name="asynchronous"></a>[Asincrónica](#tab/asynchronous)

### <a name="example-responses-for-asynchronous-operations"></a>Respuestas de ejemplo para las operaciones asincrónicas

Si se ha realizado correctamente, la solicitud GET al punto de conexión `/analyze` devolverá un objeto que contiene las tareas asignadas. Por ejemplo, `keyPhraseExtractionTasks`. Estas tareas contienen el objeto de respuesta de la característica de Text Analytics adecuada. Para más información, consulta los siguientes artículos.

+ [Extracción de frases clave](text-analytics-how-to-keyword-extraction.md#step-3-view-results)
+ [Análisis de opiniones](text-analytics-how-to-sentiment-analysis.md#view-the-results)
+ [Reconocimiento de entidades](text-analytics-how-to-entity-linking.md#view-results)
+ [Text Analytics for Health](text-analytics-for-health.md#hosted-asynchronous-web-api-response)

--- 

## <a name="see-also"></a>Consulte también

* [Información general de Text Analytics](../overview.md)
* [Versiones del modelo](../concepts/model-versioning.md)
* [Preguntas más frecuentes](../text-analytics-resource-faq.md)</br>
* [Página del producto de Text Analytics](//go.microsoft.com/fwlink/?LinkID=759712)
* [Uso de la biblioteca cliente de Text Analytics](../quickstarts/client-libraries-rest-api.md)
* [Novedades](../whats-new.md)
