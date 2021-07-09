---
title: 'Inicio rápido: Biblioteca cliente v3 de Text Analytics para Node.js | Microsoft Docs'
description: Introducción a la biblioteca cliente v3 de Text Analytics para Node.js.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 06/11/2021
ms.author: aahi
ms.reviewer: sumeh, assafi
ms.custom: devx-track-js
ms.openlocfilehash: 98e8d7862f1270977ed3eb3ab71605e440d53520
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112083865"
---
<a name="HOLTop"></a>

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

[Documentación de referencia de v3](/javascript/api/overview/azure/ai-text-analytics-readme?preserve-view=true&view=azure-node-preview) | [Código fuente de la biblioteca v3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics) | [Paquete v3 (NPM)](https://www.npmjs.com/package/@azure/ai-text-analytics/v/5.1.0-beta.6) | [Ejemplos de v3](https://github.com/Azure/azure-sdk-for-js/tree/%40azure/ai-text-analytics_5.1.0-beta.6/sdk/textanalytics/ai-text-analytics/samples)


# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

[Documentación de referencia de v3](/javascript/api/overview/azure/ai-text-analytics-readme) | [Código fuente de la biblioteca v3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics) | [Paquete v3 (NPM)](https://www.npmjs.com/package/@azure/ai-text-analytics) | [Ejemplos de v3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples)


---

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org/).
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="cree un recurso de Text Analytics"  target="_blank">cree un recurso de Text Analytics </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a API Text Analytics. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Para usar la característica Analyze, necesitará un recurso de Text Analytics con un plan de tarifa Estándar (S).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp 

cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```
### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Instale los paquetes NPM `@azure/ai-text-analytics`:

```console
npm install --save @azure/ai-text-analytics@5.1.0-beta.6
```

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo [en GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/TextAnalytics/text-analytics-v3-client-library.js), que contiene los ejemplos de código de este inicio rápido. 


# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Instale los paquetes NPM `@azure/ai-text-analytics`:

```console
npm install --save @azure/ai-text-analytics@5.0.0
```

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo [en GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/TextAnalytics/text-analytics-v3-client-library.js), que contiene los ejemplos de código de este inicio rápido. 

---

el archivo `package.json` de la aplicación se actualizará con las dependencias.
Cree un archivo llamado `index.js` y agregue los siguientes elementos:

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

```javascript
"use strict";

const { TextAnalyticsClient, AzureKeyCredential } = require("@azure/ai-text-analytics");
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

```javascript
"use strict";

const { TextAnalyticsClient, AzureKeyCredential } = require("@azure/ai-text-analytics");
```

---

Cree variables para el punto de conexión y la clave de Azure del recurso.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```javascript
const key = '<paste-your-text-analytics-key-here>';
const endpoint = '<paste-your-text-analytics-endpoint-here>';
```

## <a name="object-model"></a>Modelo de objetos

El cliente de Text Analytics es un objeto `TextAnalyticsClient` que se autentica en Azure mediante la clave. El cliente proporciona varios métodos para analizar texto, como una sola cadena o un lote.

El texto se envía a la API como una lista de `documents`, que son objetos `dictionary` que contienen una combinación de atributos `id`, `text` y `language`, según el método utilizado. El atributo `text` almacena el texto que se va a analizar en el origen `language` y `id` puede ser cualquier valor. 

El objeto de respuesta es una lista que contiene la información de análisis para cada documento. 

## <a name="code-examples"></a>Ejemplos de código

* [Autenticación de cliente](#authenticate-the-client)
* [Análisis de sentimiento](#sentiment-analysis) 
* [Minería de opiniones](#opinion-mining)
* [Detección de idioma](#language-detection)
* [Reconocimiento de entidades con nombre](#named-entity-recognition-ner)
* [Vinculación de entidad](#entity-linking)
* Información de identificación personal
* [Extracción de frases clave](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>Autenticar el cliente

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Cree un objeto `TextAnalyticsClient` con el punto de conexión y la clave como parámetros.

```javascript
const textAnalyticsClient = new TextAnalyticsClient(endpoint,  new AzureKeyCredential(key));
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Cree un objeto `TextAnalyticsClient` con el punto de conexión y la clave como parámetros.

```javascript
const textAnalyticsClient = new TextAnalyticsClient(endpoint,  new AzureKeyCredential(key));
```

---

## <a name="sentiment-analysis"></a>análisis de opiniones

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `analyzeSentiment()` del cliente y obtenga el objeto `SentimentBatchResult` devuelto. Recorra en iteración la lista de resultados e imprima el identificador de cada documento y la opinión de nivel de documento con puntuaciones de confianza. El resultado contiene, para cada documento, un sentimiento de nivel de oración junto con las puntuaciones de desplazamientos, longitud y confianza.

```javascript
async function sentimentAnalysis(client){

    const sentimentInput = [
        "I had the best day of my life. I wish you were there with me."
    ];
    const sentimentResult = await client.analyzeSentiment(sentimentInput);

    sentimentResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Sentiment: ${document.sentiment}`);
        console.log(`\tDocument Scores:`);
        console.log(`\t\tPositive: ${document.confidenceScores.positive.toFixed(2)} \tNegative: ${document.confidenceScores.negative.toFixed(2)} \tNeutral: ${document.confidenceScores.neutral.toFixed(2)}`);
        console.log(`\tSentences Sentiment(${document.sentences.length}):`);
        document.sentences.forEach(sentence => {
            console.log(`\t\tSentence sentiment: ${sentence.sentiment}`)
            console.log(`\t\tSentences Scores:`);
            console.log(`\t\tPositive: ${sentence.confidenceScores.positive.toFixed(2)} \tNegative: ${sentence.confidenceScores.negative.toFixed(2)} \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`);
        });
    });
}
sentimentAnalysis(textAnalyticsClient)
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
ID: 0
        Document Sentiment: positive
        Document Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
        Sentences Sentiment(2):
                Sentence sentiment: positive
                Sentences Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
                Sentence sentiment: neutral
                Sentences Scores:
                Positive: 0.21  Negative: 0.02  Neutral: 0.77
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `analyzeSentiment()` del cliente y obtenga el objeto `SentimentBatchResult` devuelto. Recorra en iteración la lista de resultados e imprima el identificador de cada documento y la opinión de nivel de documento con puntuaciones de confianza. El resultado contiene, para cada documento, un sentimiento de nivel de oración junto con las puntuaciones de desplazamientos, longitud y confianza.

```javascript
async function sentimentAnalysis(client){

    const sentimentInput = [
        "I had the best day of my life. I wish you were there with me."
    ];
    const sentimentResult = await client.analyzeSentiment(sentimentInput);

    sentimentResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Sentiment: ${document.sentiment}`);
        console.log(`\tDocument Scores:`);
        console.log(`\t\tPositive: ${document.confidenceScores.positive.toFixed(2)} \tNegative: ${document.confidenceScores.negative.toFixed(2)} \tNeutral: ${document.confidenceScores.neutral.toFixed(2)}`);
        console.log(`\tSentences Sentiment(${document.sentences.length}):`);
        document.sentences.forEach(sentence => {
            console.log(`\t\tSentence sentiment: ${sentence.sentiment}`)
            console.log(`\t\tSentences Scores:`);
            console.log(`\t\tPositive: ${sentence.confidenceScores.positive.toFixed(2)} \tNegative: ${sentence.confidenceScores.negative.toFixed(2)} \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`);
        });
    });
}
sentimentAnalysis(textAnalyticsClient)
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
ID: 0
        Document Sentiment: positive
        Document Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
        Sentences Sentiment(2):
                Sentence sentiment: positive
                Sentences Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
                Sentence sentiment: neutral
                Sentences Scores:
                Positive: 0.21  Negative: 0.02  Neutral: 0.77
```

---

## <a name="opinion-mining"></a>Minería de opiniones

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Para realizar el análisis de sentimiento con la minería de opiniones, cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `analyzeSentiment()` del cliente con la marca de opción de adición `includeOpinionMining: true` y obtenga el objeto `SentimentBatchResult` devuelto. Recorra en iteración la lista de resultados e imprima el identificador de cada documento y la opinión de nivel de documento con puntuaciones de confianza. Para cada documento, el resultado contiene la opinión de nivel de oración, así como el aspecto y el sentimiento de nivel de opinión.

```javascript
async function sentimentAnalysisWithOpinionMining(client){

  const sentimentInput = [
    {
      text: "The food and service were unacceptable, but the concierge were nice",
      id: "0",
      language: "en"
    }
  ];
  const results = await client.analyzeSentiment(sentimentInput, { includeOpinionMining: true });

  for (let i = 0; i < results.length; i++) {
    const result = results[i];
    console.log(`- Document ${result.id}`);
    if (!result.error) {
      console.log(`\tDocument text: ${sentimentInput[i].text}`);
      console.log(`\tOverall Sentiment: ${result.sentiment}`);
      console.log("\tSentiment confidence scores:", result.confidenceScores);
      console.log("\tSentences");
      for (const { sentiment, confidenceScores, opinions } of result.sentences) {
        console.log(`\t- Sentence sentiment: ${sentiment}`);
        console.log("\t  Confidence scores:", confidenceScores);
        console.log("\t  Mined opinions");
        for (const { target, assessments } of opinions) {
          console.log(`\t\t- Target text: ${target.text}`);
          console.log(`\t\t  Target sentiment: ${target.sentiment}`);
          console.log("\t\t  Target confidence scores:", target.confidenceScores);
          console.log("\t\t  Target assessments");
          for (const { text, sentiment } of assessments) {
            console.log(`\t\t\t- Text: ${text}`);
            console.log(`\t\t\t  Sentiment: ${sentiment}`);
          }
        }
      }
    } else {
      console.error(`\tError: ${result.error}`);
    }
  }
}
sentimentAnalysisWithOpinionMining(textAnalyticsClient)
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
- Document 0
        Document text: The food and service were unacceptable, but the concierge were nice
        Overall Sentiment: positive
        Sentiment confidence scores: { positive: 0.84, neutral: 0, negative: 0.16 }
        Sentences
        - Sentence sentiment: positive
          Confidence scores: { positive: 0.84, neutral: 0, negative: 0.16 }
          Mined opinions
                - Target text: food
                  Target sentiment: negative
                  Target confidence scores: { positive: 0.01, negative: 0.99 }
                  Target assessments
                        - Text: unacceptable
                          Sentiment: negative
                - Target text: service
                  Target sentiment: negative
                  Target confidence scores: { positive: 0.01, negative: 0.99 }
                  Target assessments
                        - Text: unacceptable
                          Sentiment: negative
                - Target text: concierge
                  Target sentiment: positive
                  Target confidence scores: { positive: 1, negative: 0 }
                  Target assessments
                        - Text: nice
                          Sentiment: positive
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Esta característica no está disponible en la versión 3.0.

---

## <a name="language-detection"></a>Detección de idiomas

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `detectLanguage()` del cliente y obtenga el objeto `DetectLanguageResultCollection` devuelto. Luego, recorra en iteración los resultados e imprima el identificador de cada documento y el idioma principal respectivo.

```javascript
async function languageDetection(client) {

    const languageInputArray = [
        "Ce document est rédigé en Français."
    ];
    const languageResult = await client.detectLanguage(languageInputArray);

    languageResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tPrimary Language ${document.primaryLanguage.name}`)
    });
}
languageDetection(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
ID: 0
        Primary Language French
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `detectLanguage()` del cliente y obtenga el objeto `DetectLanguageResultCollection` devuelto. Luego, recorra en iteración los resultados e imprima el identificador de cada documento y el idioma principal respectivo.

```javascript
async function languageDetection(client) {

    const languageInputArray = [
        "Ce document est rédigé en Français."
    ];
    const languageResult = await client.detectLanguage(languageInputArray);

    languageResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tPrimary Language ${document.primaryLanguage.name}`)
    });
}
languageDetection(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
ID: 0
        Primary Language French
```

---

## <a name="named-entity-recognition-ner"></a>Reconocimiento de entidades con nombre (NER)

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `recognizeEntities()` del cliente y obtenga el objeto `RecognizeEntitiesResult`. Recorra en iteración la lista de resultados e imprima el nombre, el tipo, el subtipo, el desplazamiento, la longitud y la puntuación de la entidad.

```javascript
async function entityRecognition(client){

    const entityInputs = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800",
        "La sede principal de Microsoft se encuentra en la ciudad de Redmond, a 21 kilómetros de Seattle."
    ];
    const entityResults = await client.recognizeEntities(entityInputs);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.text} \tCategory: ${entity.category} \tSubcategory: ${entity.subCategory ? entity.subCategory : "N/A"}`);
            console.log(`\tScore: ${entity.confidenceScore}`);
        });
    });
}
entityRecognition(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
Document ID: 0
        Name: Microsoft         Category: Organization  Subcategory: N/A
        Score: 0.29
        Name: Bill Gates        Category: Person        Subcategory: N/A
        Score: 0.78
        Name: Paul Allen        Category: Person        Subcategory: N/A
        Score: 0.82
        Name: April 4, 1975     Category: DateTime      Subcategory: Date
        Score: 0.8
        Name: 8800      Category: Quantity      Subcategory: Number
        Score: 0.8
Document ID: 1
        Name: 21        Category: Quantity      Subcategory: Number
        Score: 0.8
        Name: Seattle   Category: Location      Subcategory: GPE
        Score: 0.25
```

### <a name="personally-identifying-information-pii-recognition"></a>Reconocimiento de información de identificación personal (DCP)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `recognizePiiEntities()` del cliente y obtenga el objeto `RecognizePIIEntitiesResult`. Recorra en iteración la lista de resultados e imprima el nombre, el tipo y la puntuación de la entidad.

```javascript
async function piiRecognition(client) {

    const documents = [
        "The employee's phone number is (555) 555-5555."
    ];

    const results = await client.recognizePiiEntities(documents, "en");
    for (const result of results) {
        if (result.error === undefined) {
            console.log("Redacted Text: ", result.redactedText);
            console.log(" -- Recognized PII entities for input", result.id, "--");
            for (const entity of result.entities) {
                console.log(entity.text, ":", entity.category, "(Score:", entity.confidenceScore, ")");
            }
        } else {
            console.error("Encountered an error:", result.error);
        }
    }
}
piiRecognition(textAnalyticsClient)
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
Redacted Text:  The employee's phone number is **************.
 -- Recognized PII entities for input 0 --
(555) 555-5555 : Phone Number (Score: 0.8 )
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `recognizeEntities()` del cliente y obtenga el objeto `RecognizeEntitiesResult`. Recorra en iteración la lista de resultados e imprima el nombre, el tipo, el subtipo, el desplazamiento, la longitud y la puntuación de la entidad.

```javascript
async function entityRecognition(client){

    const entityInputs = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800",
        "La sede principal de Microsoft se encuentra en la ciudad de Redmond, a 21 kilómetros de Seattle."
    ];
    const entityResults = await client.recognizeEntities(entityInputs);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.text} \tCategory: ${entity.category} \tSubcategory: ${entity.subCategory ? entity.subCategory : "N/A"}`);
            console.log(`\tScore: ${entity.confidenceScore}`);
        });
    });
}
entityRecognition(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
Document ID: 0
        Name: Microsoft         Category: Organization  Subcategory: N/A
        Score: 0.29
        Name: Bill Gates        Category: Person        Subcategory: N/A
        Score: 0.78
        Name: Paul Allen        Category: Person        Subcategory: N/A
        Score: 0.82
        Name: April 4, 1975     Category: DateTime      Subcategory: Date
        Score: 0.8
        Name: 8800      Category: Quantity      Subcategory: Number
        Score: 0.8
Document ID: 1
        Name: 21        Category: Quantity      Subcategory: Number
        Score: 0.8
        Name: Seattle   Category: Location      Subcategory: GPE
        Score: 0.25
```


---


## <a name="entity-linking"></a>Vinculación de entidad

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `recognizeLinkedEntities()` del cliente y obtenga el objeto `RecognizeLinkedEntitiesResult`. Recorra en iteración la lista de resultados e imprima el nombre, el identificador, el origen de datos, la dirección URL y las coincidencias de la entidad. Cada objeto de la matriz `matches` contendrá el desplazamiento, la longitud y la puntuación de esa coincidencia.

```javascript
async function linkedEntityRecognition(client){

    const linkedEntityInput = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800. During his career at Microsoft, Gates held the positions of chairman, chief executive officer, president and chief software architect, while also being the largest individual shareholder until May 2014."
    ];
    const entityResults = await client.recognizeLinkedEntities(linkedEntityInput);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.name} \tID: ${entity.dataSourceEntityId} \tURL: ${entity.url} \tData Source: ${entity.dataSource}`);
            console.log(`\tMatches:`)
            entity.matches.forEach(match => {
                console.log(`\t\tText: ${match.text} \tScore: ${match.confidenceScore.toFixed(2)}`);
        })
        });
    });
}
linkedEntityRecognition(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
Document ID: 0
        Name: Altair 8800       ID: Altair 8800         URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800       Score: 0.88
        Name: Bill Gates        ID: Bill Gates  URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates        Score: 0.63
                Text: Gates     Score: 0.63
        Name: Paul Allen        ID: Paul Allen  URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen        Score: 0.60
        Name: Microsoft         ID: Microsoft   URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft         Score: 0.55
                Text: Microsoft         Score: 0.55
        Name: April 4   ID: April 4     URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4   Score: 0.32
        Name: BASIC     ID: BASIC       URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC     Score: 0.33
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `recognizeLinkedEntities()` del cliente y obtenga el objeto `RecognizeLinkedEntitiesResult`. Recorra en iteración la lista de resultados e imprima el nombre, el identificador, el origen de datos, la dirección URL y las coincidencias de la entidad. Cada objeto de la matriz `matches` contendrá el desplazamiento, la longitud y la puntuación de esa coincidencia.

```javascript
async function linkedEntityRecognition(client){

    const linkedEntityInput = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800. During his career at Microsoft, Gates held the positions of chairman, chief executive officer, president and chief software architect, while also being the largest individual shareholder until May 2014."
    ];
    const entityResults = await client.recognizeLinkedEntities(linkedEntityInput);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.name} \tID: ${entity.dataSourceEntityId} \tURL: ${entity.url} \tData Source: ${entity.dataSource}`);
            console.log(`\tMatches:`)
            entity.matches.forEach(match => {
                console.log(`\t\tText: ${match.text} \tScore: ${match.confidenceScore.toFixed(2)}`);
        })
        });
    });
}
linkedEntityRecognition(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
Document ID: 0
        Name: Altair 8800       ID: Altair 8800         URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800       Score: 0.88
        Name: Bill Gates        ID: Bill Gates  URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates        Score: 0.63
                Text: Gates     Score: 0.63
        Name: Paul Allen        ID: Paul Allen  URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen        Score: 0.60
        Name: Microsoft         ID: Microsoft   URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft         Score: 0.55
                Text: Microsoft         Score: 0.55
        Name: April 4   ID: April 4     URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4   Score: 0.32
        Name: BASIC     ID: BASIC       URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC     Score: 0.33
```

---

## <a name="key-phrase-extraction"></a>Extracción de la frase clave

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `extractKeyPhrases()` del cliente y obtenga el objeto `ExtractKeyPhrasesResult` devuelto. Itere los resultados e imprima el identificador de cada documento y todas las frases clave detectadas.

```javascript
async function keyPhraseExtraction(client){

    const keyPhrasesInput = [
        "My cat might need to see a veterinarian.",
    ];
    const keyPhraseResult = await client.extractKeyPhrases(keyPhrasesInput);
    
    keyPhraseResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Key Phrases: ${document.keyPhrases}`);
    });
}
keyPhraseExtraction(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
ID: 0
        Document Key Phrases: cat,veterinarian
```

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Cree una matriz de cadenas que contenga el documento que quiere analizar. Llame al método `extractKeyPhrases()` del cliente y obtenga el objeto `ExtractKeyPhrasesResult` devuelto. Itere los resultados e imprima el identificador de cada documento y todas las frases clave detectadas.

```javascript
async function keyPhraseExtraction(client){

    const keyPhrasesInput = [
        "My cat might need to see a veterinarian.",
    ];
    const keyPhraseResult = await client.extractKeyPhrases(keyPhrasesInput);
    
    keyPhraseResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Key Phrases: ${document.keyPhrases}`);
    });
}
keyPhraseExtraction(textAnalyticsClient);
```

Ejecute el código con `node index.js` en la ventana de la consola.

### <a name="output"></a>Output

```console
ID: 0
        Document Key Phrases: cat,veterinarian
```


---

## <a name="use-the-api-asynchronously-with-the-analyze-operation"></a>Uso de la API de forma asincrónica con la operación Analyze

# <a name="version-31-preview"></a>[Versión 3.1 (versión preliminar)](#tab/version-3-1)

Puede usar la operación Analyze para realizar solicitudes por lotes asincrónicas para: NER, extracción de frases clave, análisis de sentimiento y detección de información de identificación personal. En el ejemplo siguiente se muestra un ejemplo básico sobre una operación. Puede encontrar ejemplos más avanzados de [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/textanalytics/ai-text-analytics/samples/v5/javascript/beginAnalyzeActions.js) y [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/textanalytics/ai-text-analytics/samples/v5/typescript/src/beginAnalyzeActions.ts) en GitHub.

[!INCLUDE [Analyze Batch Action pricing](../analyze-operation-pricing-caution.md)]

Cree una nueva función llamada `analyze_example()`, que llame a la función `beginAnalyze()`. El resultado será una operación de larga duración que se sondeará para encontrar resultados.

```javascript
async function analyze_example(client) {
  const documents = [
    "Microsoft was founded by Bill Gates and Paul Allen.",
  ];

  const actions = {
    recognizeEntitiesActions: [{ modelVersion: "latest" }],
  };
  const poller = await client.beginAnalyzeActions(documents, actions, "en");

  console.log(
    `The analyze batch actions operation was created on ${poller.getOperationState().createdOn}`
  );
  console.log(
    `The analyze batch actions operation results will expire on ${
      poller.getOperationState().expiresOn
    }`
  );
  const resultPages = await poller.pollUntilDone();
  for await (const page of resultPages) {
    const entitiesAction = page.recognizeEntitiesResults[0];
    if (!entitiesAction.error) {
      for (const doc of entitiesAction.results) {
        console.log(`- Document ${doc.id}`);
        if (!doc.error) {
          console.log("\tEntities:");
          for (const entity of doc.entities) {
            console.log(`\t- Entity ${entity.text} of type ${entity.category}`);
          }
        } else {
          console.error("\tError:", doc.error);
        }
      }
    }
  }
}

analyze_example(textAnalyticsClient);
```

### <a name="output"></a>Output

```console
The analyze batch actions operation was created on Fri Mar 12 2021 09:53:49 GMT-0800 (Pacific Standard Time)
The analyze batch actions operation results will expire on Sat Mar 13 2021 09:53:49 GMT-0800 (Pacific Standard Time)
- Document 0
        Entities:
        - Entity Microsoft of type Organization
        - Entity Bill Gates of type Person
        - Entity Paul Allen of type Person
```

También puede usar la operación Analyze para ejecutar NER, la extracción de frases clave, el análisis de sentimiento y la detección de información de identificación personal. Consulte los ejemplos de Analyze para [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/textanalytics/ai-text-analytics/samples/v5/javascript/beginAnalyzeActions.js) y [TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples/v5/typescript/src) en GitHub.

# <a name="version-30"></a>[Versión 3.0](#tab/version-3)

Esta característica no está disponible en la versión 3.0.

---

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```
