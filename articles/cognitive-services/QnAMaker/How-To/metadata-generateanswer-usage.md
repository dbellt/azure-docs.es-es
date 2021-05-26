---
title: 'Metadatos con GenerateAnswer API: QnA Maker'
titleSuffix: Azure Cognitive Services
description: QnA Maker permite agregar metadatos, en forma de pares de clave-valor, a los pares de preguntas y respuestas. Puede filtrar los resultados a las consultas de usuario y almacenar la información adicional que se puede usar en las conversaciones de seguimiento.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 0daa44f0d901dc14541785f764d5ccf8f96fc737
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376680"
---
# <a name="get-an-answer-with-the-generateanswer-api"></a>Obtención de una respuesta con GenerateAnswer API

Para obtener la respuesta prevista a la pregunta de un usuario, use GenerateAnswer API. Al publicar una base de conocimiento, puede consultar información sobre cómo usar esta API en la página **Publicar**. También puede configurar la API para filtrar las respuestas según las etiquetas de metadatos y probar la base de conocimiento desde el punto de conexión con el parámetro de cadena de consulta de prueba.

<a name="generateanswer-api"></a>

## <a name="get-answer-predictions-with-the-generateanswer-api"></a>Obtención de predicciones de respuestas con GenerateAnswer API

[GenerateAnswer API](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) se usa en un bot o aplicación para consultar la base de conocimiento con una pregunta de usuario y obtener la mejor coincidencia de los pares de preguntas y respuestas.

<a name="generateanswer-endpoint"></a>

## <a name="publish-to-get-generateanswer-endpoint"></a>Publicación para obtener el punto de conexión de GenerateAnswer

Después de publicar la base de conocimiento, ya sea desde el [portal de QnA Maker](https://www.qnamaker.ai) o mediante la [API](/rest/api/cognitiveservices/qnamaker/knowledgebase/publish), puede obtener los detalles de su punto de conexión de GenerateAnswer.

Para obtener los detalles del punto de conexión:
1. Inicie sesión en [https://www.qnamaker.ai](https://www.qnamaker.ai).
1. En **My knowledge bases** (Mis bases de conocimiento), seleccione **View Code** (Ver código) correspondiente a su base de conocimiento.
    ![Captura de pantalla de My knowledge bases (Mis bases de conocimiento)](../media/qnamaker-how-to-metadata-usage/my-knowledge-bases.png)
1. Obtenga los detalles del punto de conexión de GenerateAnswer.

    # <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

    ![Captura de los detalles del punto de conexión](../media/qnamaker-how-to-metadata-usage/view-code.png)

    # <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

    ![Captura de pantalla de los detalles del punto de conexión administrados](../media/qnamaker-how-to-metadata-usage/view-code-managed.png)

    ---

También puede obtener los detalles del punto de conexión en la pestaña **Settings** (Configuración) de la base de conocimiento.

<a name="generateanswer-request"></a>

## <a name="generateanswer-request-configuration"></a>Configuración de la solicitud GenerateAnswer

Para llamar a GenerateAnswer, se utiliza una solicitud HTTP POST. Para ver código de ejemplo que muestra cómo llamar a GenerateAnswer, consulte las [guías de inicio rápido](../quickstarts/quickstart-sdk.md#generate-an-answer-from-the-knowledge-base).

La solicitud POST usa:

* Los [parámetros de URI](/rest/api/cognitiveservices/qnamakerruntime/runtime/train#uri-parameters) obligatorios.
* La propiedad del encabezado obligatoria, `Authorization`, por seguridad.
* Las [propiedades del cuerpo](/rest/api/cognitiveservices/qnamakerruntime/runtime/train#feedbackrecorddto) obligatorias.

La dirección URL de GenerateAnswer tiene el formato siguiente:

```
https://{QnA-Maker-endpoint}/knowledgebases/{knowledge-base-ID}/generateAnswer
```

Recuerde establecer la propiedad del encabezado HTTP de `Authorization` con un valor de la cadena `EndpointKey` con un espacio final y, luego, la clave del punto de conexión que se encontró en la página **Configuración**.

Un cuerpo JSON de ejemplo tiene el siguiente aspecto:

```json
{
    "question": "qna maker and luis",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 30,
    "rankerType": "" // values: QuestionOnly
    "strictFilters": [
    {
        "name": "category",
        "value": "api"
    }],
    "userId": "sd53lsY="
}
```

Más información acerca de [rankerType](../concepts/best-practices.md#choosing-ranker-type).

El código JSON anterior solo solicitó aquellas respuestas con una puntuación del 30 % o que están por encima de la puntuación del umbral.

<a name="generateanswer-response"></a>

## <a name="generateanswer-response-properties"></a>Propiedades de la respuesta de GenerateAnswer

La [respuesta](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#successful-query) es un objeto JSON que incluye toda la información necesaria para mostrar la respuesta y el siguiente turno en la conversación, si está disponible.

```json
{
    "answers": [
        {
            "score": 38.54820341616869,
            "Id": 20,
            "answer": "There is no direct integration of LUIS with QnA Maker. But, in your bot code, you can use LUIS and QnA Maker together. [View a sample bot](https://github.com/Microsoft/BotBuilder-CognitiveServices/tree/master/Node/samples/QnAMaker/QnAWithLUIS)",
            "source": "Custom Editorial",
            "questions": [
                "How can I integrate LUIS with QnA Maker?"
            ],
            "metadata": [
                {
                    "name": "category",
                    "value": "api"
                }
            ]
        }
    ]
}
```

El código JSON anterior respondió con una respuesta con una puntuación del 38,5 %.

## <a name="match-questions-only-by-text"></a>Coincidencia solo de preguntas, por texto

De manera predeterminada, QnA Maker busca a través de preguntas y respuestas. Si quiere buscar solo en las preguntas, para generar una respuesta, utilice el objeto `RankerType=QuestionOnly` del cuerpo de POST de la solicitud GenerateAnswer.

Puede buscar en la base de conocimiento publicada, con `isTest=false`, o bien en la base de conocimiento de prueba, con `isTest=true`.

```json
{
  "question": "Hi",
  "top": 30,
  "isTest": true,
  "RankerType":"QuestionOnly"
}

```
## <a name="use-qna-maker-with-a-bot-in-c"></a>Uso de QnA Maker con un bot en C#

Bot Framework brinda acceso a las propiedades de QnA Maker con [getAnswer API](/dotnet/api/microsoft.bot.builder.ai.qna.qnamaker.getanswersasync#Microsoft_Bot_Builder_AI_QnA_QnAMaker_GetAnswersAsync_Microsoft_Bot_Builder_ITurnContext_Microsoft_Bot_Builder_AI_QnA_QnAMakerOptions_System_Collections_Generic_Dictionary_System_String_System_String__System_Collections_Generic_Dictionary_System_String_System_Double__):

```csharp
using Microsoft.Bot.Builder.AI.QnA;
var metadata = new Microsoft.Bot.Builder.AI.QnA.Metadata();
var qnaOptions = new QnAMakerOptions();

metadata.Name = Constants.MetadataName.Intent;
metadata.Value = topIntent;
qnaOptions.StrictFilters = new Microsoft.Bot.Builder.AI.QnA.Metadata[] { metadata };
qnaOptions.Top = Constants.DefaultTop;
qnaOptions.ScoreThreshold = 0.3F;
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext, qnaOptions);
```

El código JSON anterior solo solicitó aquellas respuestas con una puntuación del 30 % o que están por encima de la puntuación del umbral.

## <a name="use-qna-maker-with-a-bot-in-nodejs"></a>Uso de QnA Maker con un bot en Node.js

Bot Framework brinda acceso a las propiedades de QnA Maker con [getAnswer API](/javascript/api/botbuilder-ai/qnamaker?preserve-view=true&view=botbuilder-ts-latest#generateanswer-string---undefined--number--number-):

```javascript
const { QnAMaker } = require('botbuilder-ai');
this.qnaMaker = new QnAMaker(endpoint);

// Default QnAMakerOptions
var qnaMakerOptions = {
    ScoreThreshold: 0.30,
    Top: 3
};
var qnaResults = await this.qnaMaker.getAnswers(stepContext.context, qnaMakerOptions);
```

El código JSON anterior solo solicitó aquellas respuestas con una puntuación del 30 % o que están por encima de la puntuación del umbral.

## <a name="get-precise-answers-with-generateanswer-api"></a>Obtención de respuestas precisas con GenerateAnswer API

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

Solo ofrecemos una característica de respuesta precisa con la versión administrada de QnA Maker.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

El usuario puede habilitar [respuestas precisas](../reference-precise-answering.md) al usar el recurso de Text Analytics con la característica de respuesta a preguntas personalizada. El parámetro answerSpanRequest debe actualizarse por lo mismo.

```json
{
    "question": "How long it takes to charge surface pro 4?",
    "top": 3,
    "answerSpanRequest": {
        "enable": true,
        "topAnswersWithSpan": 1
    }
}
```

Igualmente, los usuarios pueden optar por no establecer el parámetro answerSpanRequest para deshabilitar las respuestas precisas.

```json
{
    "question": "How long it takes to charge surface pro 4?",
    "top": 3
}
```
### <a name="bot-settings"></a>Configuración de bot

Si desea configurar los valores de respuesta precisos para el servicio de bot, vaya al recurso de App Service para el bot. Después, agregue el siguiente valor para actualizar las configuraciones.

- EnablePreciseAnswer
- DisplayPreciseAnswerOnly

|Configuración de pantalla|EnablePreciseAnswer|DisplayPreciseAnswerOnly|
|:--|--|--|
|Solo respuestas precisas|true|true|
|Solo respuestas largas|false|false|
|Respuestas largas y precisas|true|false|

---

## <a name="common-http-errors"></a>Errores HTTP comunes

|Código|Explicación|
|:--|--|
|2xx|Correcto|
|400|Los parámetros de la solicitud no son correctos, lo que significa que faltan parámetros necesarios, o bien que tienen un formato incorrecto o son demasiado grandes|
|400|El cuerpo de la solicitud es incorrecto, lo que significa que falta el código JSON, o bien que tiene un formato incorrecto o es demasiado grande|
|401|Clave no válida|
|403|Prohibido: no tiene los permisos correctos|
|404|KB no existe|
|410|Esta API está en desuso y ya no está disponible|

## <a name="next-steps"></a>Pasos siguientes

La página **Publicar** también proporciona información para [generar una respuesta](../Quickstarts/get-answer-from-knowledge-base-using-url-tool.md) con Postman o cURL.

> [!div class="nextstepaction"]
> [Obtener análisis en la base de conocimiento](../how-to/get-analytics-knowledge-base.md)
> [!div class="nextstepaction"]
> [Puntuación de confianza](../Concepts/confidence-score.md)
