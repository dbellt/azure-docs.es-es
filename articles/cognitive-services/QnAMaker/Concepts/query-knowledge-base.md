---
title: 'Consulta a una base de conocimiento: QnA Maker'
description: Se debe publicar una base de conocimiento. Una vez publicada, se consulta la base de conocimiento a través del punto de conexión de predicción del tiempo de ejecución mediante generateAnswer API.
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 4d36e1feb0279eec638a3602c5188e8af1cb7c17
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369309"
---
# <a name="query-the-knowledge-base-for-answers"></a>Consulta a una base de conocimiento para obtener respuestas

Se debe publicar una base de conocimiento. Una vez publicada, se consulta la base de conocimiento a través del punto de conexión de predicción del tiempo de ejecución mediante generateAnswer API. La consulta incluye el texto de la pregunta y otros valores para que QnA Maker seleccione la mejor coincidencia posible con una respuesta.

## <a name="how-qna-maker-processes-a-user-query-to-select-the-best-answer"></a>Cómo QnA Maker procesa una consulta de usuario para seleccionar la mejor respuesta

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

La base de conocimiento entrenada y [publicada](../quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) de QnA Maker recibe una consulta de usuario, ya sea desde un bot o de otra aplicación cliente, en [GenerateAnswer API](../how-to/metadata-generateanswer-usage.md). En el diagrama siguiente se muestra el proceso de recepción de la consulta de usuario.

![El proceso del modelo de clasificación de una consulta de usuario](../media/qnamaker-concepts-knowledgebase/ranker-v1.png)

### <a name="ranker-process"></a>Proceso del clasificador

El proceso se explica en la tabla siguiente.

|Paso|Propósito|
|--|--|
|1|La aplicación cliente envía la consulta de usuario a [GenerateAnswer API](../how-to/metadata-generateanswer-usage.md).|
|2|QnA Maker preprocesa la consulta de usuario con la detección del idioma, los correctores ortográficos y los separadores de palabras.|
|3|Este preprocesamiento se realiza para modificar la consulta de usuario y obtener los mejores resultados de la búsqueda.|
|4|Esta consulta modificada se envía al índice de Azure Cognitive Search, que recibe el número `top` de resultados. Si no se encuentra la respuesta correcta en estos resultados, aumente levemente el valor de `top`. Por lo general, un valor de 10 para `top` funciona en el 90 % de las consultas. En este paso, la búsqueda de Azure filtra las [palabras reservadas](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/STOPWORDS.md).|
|5|QnA Maker usa características sintácticas y semánticas para determinar las similitudes entre la consulta de usuario y los resultados de QnA capturados.|
|6|El modelo de clasificador con aprendizaje automático usa las diferentes características del paso 5 para determinar las puntuaciones de confianza y el nuevo orden de clasificación.|
|7|Los resultados nuevos se devuelven a la aplicación cliente en el orden de clasificación.|
|||

Las características que se usan incluyen, entre otros, la semántica en el nivel de palabra, la importancia en el nivel de término en un conjunto y los modelos de semántica de aprendizaje profundo para determinar la similitud e importancia entre dos cadenas de texto.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

La base de conocimiento entrenada y [publicada](../quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) de QnA Maker recibe una consulta de usuario, ya sea desde un bot o de otra aplicación cliente, en [GenerateAnswer API](../how-to/metadata-generateanswer-usage.md). En el diagrama siguiente se muestra el proceso de recepción de la consulta de usuario.

![Proceso del modelo de clasificación de una consulta de usuario (versión preliminar)](../media/qnamaker-concepts-knowledgebase/ranker-v2.png)

### <a name="ranker-process"></a>Proceso del clasificador

El proceso se explica en la tabla siguiente.

|Paso|Propósito|
|--|--|
|1|La aplicación cliente envía la consulta de usuario a [GenerateAnswer API](../how-to/metadata-generateanswer-usage.md).|
|2|QnA Maker preprocesa la consulta de usuario con la detección del idioma, los correctores ortográficos y los separadores de palabras.|
|3|Este preprocesamiento se realiza para modificar la consulta de usuario y obtener los mejores resultados de la búsqueda.|
|4|Esta consulta modificada se envía al índice de Azure Cognitive Search, que recibe el número `top` de resultados. Si no se encuentra la respuesta correcta en estos resultados, aumente levemente el valor de `top`. Por lo general, un valor de 10 para `top` funciona en el 90 % de las consultas. En este paso, la búsqueda de Azure filtra las [palabras reservadas](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/STOPWORDS.md).|
|5|QnA Maker usa un modelo basado en un innovador convertidor para determinar la similitud entre la consulta del usuario y los resultados de las instancias de QnA candidatas capturadas desde Azure Cognitive Search. El modelo basado en un convertidor es un modelo multilingüe de aprendizaje profundo, que funciona horizontalmente para todos los idiomas con el fin de determinar las puntuaciones de confianza y el nuevo orden de clasificación.|
|6|Los resultados nuevos se devuelven a la aplicación cliente en el orden de clasificación.|
|||

El clasificador funciona en todas las preguntas y respuestas alternativas para buscar los pares de preguntas y respuestas con una mayor coincidencia para la consulta de usuario. Los usuarios tienen la posibilidad de configurar el clasificador para la clasificación solo de preguntas. 

---

## <a name="http-request-and-response-with-endpoint"></a>Solicitud y respuesta HTTP con punto de conexión
Al publicar la base de conocimiento, el servicio crea un punto de conexión HTTP basado en REST que se puede integrar en una aplicación, normalmente un bot de chat.

### <a name="the-user-query-request-to-generate-an-answer"></a>Solicitud de consulta de usuario para generar una respuesta

Una consulta de usuario es la pregunta que el usuario final formula en la base de conocimiento, como por ejemplo `How do I add a collaborator to my app?`. A menudo la consulta está en un formato de lenguaje natural o se usan unas pocas palabras clave que representan la pregunta, como por ejemplo `help with collaborators`. La consulta se envía a la base de conocimiento desde una solicitud HTTP en la aplicación cliente.

```json
{
    "question": "How do I add a collaborator to my app?",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 20,
    "strictFilters": [
    {
        "name": "QuestionType",
        "value": "Support"
    }],
    "userId": "sd53lsY="
}
```
Puede controlar la respuesta estableciendo propiedades como [scoreThreshold](./confidence-score.md#choose-a-score-threshold), [top](../how-to/improve-knowledge-base.md#use-the-top-property-in-the-generateanswer-request-to-get-several-matching-answers) y [strictFilters](../how-to/query-knowledge-base-with-metadata.md).

Use [contexto de conversación](../how-to/query-knowledge-base-with-metadata.md) con la [función de varios turnos](../how-to/multiturn-conversation.md) para mantener la conversación en marcha con el fin de refinar las preguntas y respuestas y encontrar la respuesta correcta y final.

### <a name="the-response-from-a-call-to-generate-an-answer"></a>Respuesta de una llamada para generar una respuesta

La respuesta HTTP es la respuesta que se recupera de la base de conocimientos, en función de la coincidencia más alta para una consulta de usuario dada. La respuesta incluye la respuesta y la puntuación de predicción. Si ha solicitado más de una respuesta principal, con la propiedad `top`, obtendrá más de una respuesta principal, cada una con una puntuación.

```json
{
    "answers": [
        {
            "questions": [
                "How do I add a collaborator to my app?",
                "What access control is provided for the app?",
                "How do I find user management and security?"
            ],
            "answer": "Use the Azure portal to add a collaborator using Access Control (IAM)",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "QuestionType",
                    "value": "Support"
                },
                {
                    "name": "ToolDependency",
                    "value": "Azure Portal"
                }
            ]
        }
    ]
}
```


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Puntuación de confianza](./confidence-score.md)
