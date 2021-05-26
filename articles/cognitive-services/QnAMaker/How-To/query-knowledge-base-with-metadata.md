---
title: Filtrado contextual mediante metadatos
titleSuffix: Azure Cognitive Services
description: QnA Maker filtra los pares de preguntas y respuestas por metadatos.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 268cb05b7399c6fc812310f451797df60e283557
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376312"
---
# <a name="filter-responses-with-metadata"></a>Filtrado de las respuestas con metadatos

QnA Maker permite agregar metadatos, en forma de pares clave-valor, a los pares de preguntas y respuestas. Luego puede usar esta información para filtrar los resultados de las consultas de usuario y para almacenar información adicional que se puede usar en las conversaciones de seguimiento.

<a name="qna-entity"></a>

## <a name="store-questions-and-answers-with-a-qna-entity"></a>Almacenamiento de preguntas y respuestas con una entidad QnA

Resulta importante entender cómo QnA Maker almacena los datos de preguntas y respuestas. En la siguiente ilustración se muestra una entidad QnA:

![Ilustración de una entidad QnA](../media/qnamaker-how-to-metadata-usage/qna-entity.png)

Cada entidad QnA tiene un identificador único y persistente. Puede usar el identificador para realizar actualizaciones en una entidad QnA determinada.

## <a name="use-metadata-to-filter-answers-by-custom-metadata-tags"></a>Uso de metadatos para filtrar las respuestas por etiquetas de metadatos personalizadas

Agregar los metadatos le permite filtrar las respuestas según estas etiquetas de metadatos. Agregue la columna de metadatos desde el menú **View Options** (Ver opciones). Para agregar los metadatos a la base de conocimiento, seleccione el icono **+** de los metadatos para agregar un par de metadatos. Este par consta de una clave y un valor.

![Captura de pantalla de incorporación de metadatos](../media/qnamaker-how-to-metadata-usage/add-metadata.png)

<a name="filter-results-with-strictfilters-for-metadata-tags"></a>

## <a name="filter-results-with-strictfilters-for-metadata-tags"></a>Filtrar los resultados con strictFilters para las etiquetas de metadatos

Considere la pregunta del usuario "¿Cuándo cierra este hotel?", donde la intención está implícita para el restaurante "Paradise".

Puesto que los resultados solo son necesarios para el restaurante "Paradise", puede establecer un filtro en la llamada GenerateAnswer para los metadatos "Nombre del restaurante". Esto se muestra en el ejemplo siguiente:

```json
{
    "question": "When does this hotel close?",
    "top": 1,
    "strictFilters": [ { "name": "restaurant", "value": "paradise"}]
}
```

## <a name="filter-by-source"></a>Filtrar por origen

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

En la versión de disponibilidad general de QnA Maker no se pueden filtrar los resultados por un origen de contenido.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

En caso de que tenga varios orígenes de contenido en la base de conocimiento y quiera limitar los resultados a un conjunto determinado de orígenes, puede hacerlo con la palabra clave reservada `source_name_metadata`, como se muestra a continuación.

```json
"strictFilters": [
    {
        "name": "category",
        "value": "api"
    },
   {
        "name": "source_name_metadata",
        "value": "boby_brown_docx"
    },
   {
        "name": "source_name_metadata",
        "value": "chitchat.tsv"
   }
]
```

---

### <a name="logical-and-by-default"></a>AND lógico de manera predeterminada

Para combinar varios filtros de metadatos en la consulta, agregue los filtros de metadatos adicionales a la matriz de la propiedad `strictFilters`. De manera predeterminada, los valores se combinan lógicamente (AND). Una combinación lógica requiere que todos los filtros coincidan con los pares PyR para que se devuelva el par en la respuesta.

Esto es equivalente a utilizar la propiedad `strictFiltersCompoundOperationType` con el valor de `AND`.

### <a name="logical-or-using-strictfilterscompoundoperationtype-property"></a>OR lógico mediante la propiedad strictFiltersCompoundOperationType

Al combinar varios filtros de metadatos, si solo le preocupa uno o algunos de los filtros que coinciden, use la propiedad `strictFiltersCompoundOperationType` con el valor de `OR`.

Esto permite que knowledge base devuelva respuestas cuando algún filtro coincida, pero no devolverá respuestas que no tengan metadatos.

```json
{
    "question": "When do facilities in this hotel close?",
    "top": 1,
    "strictFilters": [
      { "name": "type","value": "restaurant"},
      { "name": "type", "value": "bar"},
      { "name": "type", "value": "poolbar"}
    ],
    "strictFiltersCompoundOperationType": "OR"
}
```

### <a name="metadata-examples-in-quickstarts"></a>Ejemplos de metadatos en los inicios rápidos

Obtenga más información sobre los metadatos en el inicio rápido del portal de QnA Maker sobre metadatos:
* [Creación: adición de metadatos a un par de QnA](../quickstarts/add-question-metadata-portal.md#add-metadata-to-filter-the-answers)
* [Predicción de consultas: filtrar respuestas por metadatos](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md)

<a name="keep-context"></a>

## <a name="use-question-and-answer-results-to-keep-conversation-context"></a>Uso de los resultados de las preguntas y respuestas para mantener el contexto de la conversación

La respuesta a GenerateAnswer contiene la información de los metadatos correspondientes al par de pregunta y respuesta coincidente. Puede usar esta información en la aplicación cliente para almacenar el contexto de la conversación anterior para su uso en conversiones posteriores.

```json
{
    "answers": [
        {
            "questions": [
                "What is the closing time?"
            ],
            "answer": "10.30 PM",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "restaurant",
                    "value": "paradise"
                },
                {
                    "name": "location",
                    "value": "secunderabad"
                }
            ]
        }
    ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Analice su base de conocimiento](../How-to/get-analytics-knowledge-base.md)
