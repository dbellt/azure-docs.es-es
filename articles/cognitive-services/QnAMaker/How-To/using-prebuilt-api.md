---
title: API precompilada QnA Maker
titleSuffix: Azure Cognitive Services
description: Uso de la API precompilada para obtener respuestas sobre un texto
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 05/05/2021
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 0aa26cc0d78d6ca20dccb97b3936bad840c989b7
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382929"
---
# <a name="prebuilt-question-answering"></a>Respuesta a preguntas precompilada

La característica de respuesta a preguntas precompilada ofrece al usuario la posibilidad de responder preguntas de un fragmento de texto sin tener que crear knowledge bases, mantener pares de preguntas y respuestas ni incurrir en los costos de una infraestructura infrautilizada. Esta funcionalidad se proporciona como una API y se puede usar para satisfacer las necesidades de preguntas y respuestas sin tener que aprender los detalles sobre QnA Maker ni el almacenamiento adicional.

Dada una consulta de usuario y un bloque de texto o fragmento, la API devolverá una respuesta y una respuesta precisa (si está disponible). 

<a name="qna-entity"></a>


## <a name="example-usage-of-prebuilt-question-answering"></a>Ejemplo de uso de la funcionalidad de respuesta a preguntas precompilada

Imagine que tiene uno o varios bloques de texto de los que le gustaría obtener las respuestas a una pregunta determinada. En un sistema convencional, habría tenido que crear tantos orígenes como número de bloques de texto. Sin embargo, ahora con la funcionalidad de respuesta a preguntas precompilada puede consultar los bloques de texto sin tener que definir orígenes de contenido en una knowledge base. 

A continuación se indican otros escenarios en los que se puede usar la API precompilada:

* Está desarrollando una aplicación de lector de libros electrónicos para usuarios finales que les permite resaltar texto, escribir una pregunta y encontrar respuestas en el texto resaltado. 
* Una extensión del navegador permite a los usuarios hacer una pregunta sobre el contenido que se muestra actualmente en la página del navegador.
* Un bot de mantenimiento recibe las consultas de los usuarios y proporciona respuestas basadas en el contenido médico que el bot identifica como más pertinente para la consulta del usuario. 

A continuación se muestra un ejemplo de una solicitud:

## <a name="sample-request"></a>Solicitud de ejemplo
```
POST https://{Endpoint}/qnamaker/v5.0-preview.2/generateanswer
```

### <a name="sample-query-over-a-single-block-of-text"></a>Consulta de ejemplo sobre un único bloque de texto

Cuerpo de la solicitud

```json
{
    "question": "How long it takes to charge surface pro 4?",
    "documents": [
        {
            "text": "### The basics #### Power and charging It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state. It can take longer if you’re using your Surface for power-intensive activities like gaming or video streaming while you’re charging it. You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges.",
            "id": "doc1"
        }
    ],
    "Language": "en"
}
```
## <a name="sample-response"></a>Respuesta de ejemplo

En el cuerpo de la solicitud anterior, se consulta sobre un único bloque de texto. A continuación se muestra una respuesta de ejemplo recibida para la consulta anterior.

```json
{
    "answers": [
        {
            "answer": "### The basics #### Power and charging It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state. It can take longer if you’re using your Surface for power-intensive activities like gaming or video streaming while you’re charging it. You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges.",
            "answerSpan": {
                "text": "two to four hours",
                "score": 0.0,
                "startIndex": 47,
                "endIndex": 64
            },
            "score": 0.9599020481109619,
            "id": "doc1",
            "answerStartIndex": 0,
            "answerEndIndex": 390
        },
        {
            "answer": "It can take longer if you’re using your Surface for power-intensive activities like gaming or video streaming while you’re charging it. You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges.",
            "score": 0.06749606877565384,
            "id": "doc1",
            "answerStartIndex": 129,
            "answerEndIndex": 390
        },
        {
            "answer": "You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges.",
            "score": 0.011389964260160923,
            "id": "doc1",
            "answerStartIndex": 265,
            "answerEndIndex": 390
        }
    ]
}
```
Vemos que se reciben varias respuestas como parte de la respuesta global de la API. Cada respuesta tiene una puntuación de confianza específica que ayuda a comprender su relevancia general. Los usuarios pueden usar esta puntuación de confianza para mostrar las respuestas a la consulta.

## <a name="prebuilt-api-limits"></a>Límites de la API precompilada 

Visite la documentación sobre los [límites de la API precompilada](https://docs.microsoft.com/azure/cognitive-services/qnamaker/limits?#prebuilt-question-answering-limits). 

## <a name="prebuilt-api-reference"></a>Referencia de la API precompilada
Visite la documentación de [referencia de la API precompilada](https://docs.microsoft.com/rest/api/cognitiveservices-qnamaker/qnamaker5.0preview2/prebuilt/generateanswer) para conocer los parámetros de entrada y salida necesarios para llamar a la API.
