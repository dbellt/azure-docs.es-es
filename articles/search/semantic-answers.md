---
title: Devolución de una respuesta semántica
titleSuffix: Azure Cognitive Search
description: Se describe la composición de una respuesta semántica y cómo se obtienen respuestas de un conjunto de resultados.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/27/2021
ms.openlocfilehash: d0390bd70080ea0174a81cce9538396321dec658
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111539345"
---
# <a name="return-a-semantic-answer-in-azure-cognitive-search"></a>Devolución de una respuesta semántica en Azure Cognitive Search

> [!IMPORTANT]
> La búsqueda semántica está en versión preliminar pública en los [términos de uso complementarios](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Está disponible a través de Azure Portal, la API REST en versión preliminar y los SDK en versión beta. Estas características son facturables. Para obtener más información, consulte [Disponibilidad y precios](semantic-search-overview.md#availability-and-pricing).

Al invocar una [clasificación semántica y leyendas semánticas](semantic-how-to-query-request.md), puede extraer el contenido de los documentos con mayor coincidencia que "respondan" directamente a la consulta. En la respuesta de la consulta se pueden incluir una o más respuestas, que posteriormente puede representar en una página de búsqueda para mejorar la experiencia del usuario de la aplicación.

En este artículo, aprenderá a solicitar una respuesta semántica, a desempaquetar la respuesta y a saber qué características de contenido son más favorables para generar respuestas de alta calidad.

## <a name="prerequisites"></a>Requisitos previos

Todos los requisitos previos que se aplican a las [consultas semánticas](semantic-how-to-query-request.md#prerequisites) también son aplicables a las respuestas, incluidos el [nivel de servicio y la región](semantic-search-overview.md#availability-and-pricing).

+ La lógica de consulta debe incluir los parámetros de consulta semántica "queryType=semantic", además del parámetro "answers". En este artículo se describen los parámetros necesarios.

+ Las cadenas de consulta que especifique el usuario deben ser reconocibles como pregunta (qué, dónde, cuándo, cómo).

+ Los documentos de búsqueda del índice deben contener texto que tenga las características de una respuesta y dicho texto debe existir en uno de los campos enumerados en "searchFields". Por ejemplo, dada una consulta "Qué es una tabla hash", si ninguno de los campos searchField contiene pasajes que incluyan "Una tabla hash es...", es poco probable que se devuelva una respuesta.

## <a name="what-is-a-semantic-answer"></a>¿Qué es una respuesta semántica?

Una respuesta semántica es una subestructura de una [respuesta de consulta semántica](semantic-how-to-query-request.md). Se compone de uno o varios pasajes textuales de un documento de búsqueda, formulados como una respuesta a una consulta que tiene el aspecto de una pregunta. Para que se devuelva una respuesta, deben existir frases u oraciones en un documento de búsqueda que presente las características del lenguaje de una respuesta y la propia consulta debe plantearse como una pregunta.

Cognitive Search usa un modelo de comprensión de lectura automática para formular respuestas. El modelo genera un conjunto de posibles respuestas a partir del contenido disponible y, cuando llega a un nivel de confianza lo suficientemente alto, propone una respuesta.

Las respuestas se devuelven como un objeto de nivel superior e independiente en la carga de respuesta de la consulta, el cual puede elegir para su representación en páginas de búsqueda, junto con los resultados de búsqueda. Estructuralmente, es un elemento de matriz dentro de la respuesta que consta de texto, una clave de documento y una puntuación de confianza.

<a name="query-params"></a>

## <a name="how-to-specify-answers-in-a-query-request"></a>Cómo especificar "respuestas" en una solicitud de consulta

Para devolver una respuesta semántica, la consulta debe tener el "queryType" semántica, "queryLanguage", "searchFields" y el parámetro "answers". El hecho de especificar el parámetro "answers" no garantiza que se vaya a obtener una respuesta, pero la solicitud debe incluirlo para que se invoque el procesamiento de respuestas.

El parámetro "searchFields" es fundamental para devolver una respuesta de alta calidad, tanto en términos de contenido como de orden (consulte a continuación). 

```json
{
    "search": "how do clouds form",
    "queryType": "semantic",
    "queryLanguage": "en-us",
    "searchFields": "title,locations,content",
    "answers": "extractive|count-3",
    "count": "true"
}
```

+ Una cadena de consulta no debe ser NULL y debe formularse como una pregunta.

+ El valor de "queryType" debe establecerse en "semantic".

+ El elemento "queryLanguage" debe tener uno de los valores de la [lista de idiomas admitidos (API REST)](/rest/api/searchservice/preview-api/search-documents#queryLanguage).

+ El elemento "searchFields" determina qué campos de cadena proporcionan tokens al modelo de extracción. Los mismos campos que producen leyendas también generan respuestas. Para obtener una guía precisa sobre cómo establecer este campo para que funcione con títulos y respuestas, consulte [Configurar searchFields](semantic-how-to-query-request.md#searchfields). 

+ En el caso de "answers", la construcción de parámetros es `"answers": "extractive"`, donde el número de respuestas predeterminado devuelto es uno. Puede aumentar el número de respuestas si agrega un elemento `count` como se muestra en el ejemplo anterior, hasta un máximo de cinco.  En función de la experiencia del usuario de la aplicación y de cómo quiera representar los resultados, podría necesitar más de una respuesta.

## <a name="deconstruct-an-answer-from-the-response"></a>Deconstrucción de una respuesta a partir de la respuesta de la consulta

Las respuestas se proporcionan en la matriz @search.answers, que aparece primero en la respuesta de la consulta. Si una respuesta es indeterminada, la respuesta de la consulta se mostrará como `"@search.answers": []`. Cuando diseñe una página de resultados de la búsqueda que incluya respuestas, asegúrese de controlar los casos en los que no se encuentran respuestas.

En @search.answers, la "clave" es la clave del documento o el identificador de la coincidencia. Dada una clave de documento, puede usar la API para [buscar documento](/rest/api/searchservice/lookup-document) a fin de recuperar algunas o todas las partes del documento de búsqueda que se van a incluir en la página de búsqueda o en una página de detalles.

Tanto "text" como "highlights" proporcionan contenido idéntico en texto sin formato y con resaltado. De forma predeterminada, los resaltados tienen el estilo `<em>`, que se puede invalidar mediante los parámetros highlightPreTag y highlightPostTag. Como ya se ha indicado anteriormente, la sustancia de una respuesta es el contenido textual de un documento de búsqueda. El modelo de extracción busca las características de una respuesta para encontrar el contenido adecuado, pero no crea un lenguaje nuevo en la respuesta de la consulta.

El valor "score" es la puntuación de confianza que refleja la intensidad de la respuesta. Si hay varias respuestas en la respuesta de la consulta, se usa esta puntuación para determinar el orden. Las respuestas y los títulos principales pueden derivar de documentos de búsqueda diferentes (por ejemplo, la respuesta principal procede de un documento y el título principal de otro), pero en general verá los mismos documentos en las posiciones superiores de cada matriz.

Las respuestas van seguidas de la matriz "value", que siempre incluye puntuaciones, títulos y todos los campos que se recuperen de forma predeterminada. Si especificó el parámetro select, la matriz "value" se limita a los campos especificados. Para obtener más información sobre los elementos de la respuesta de la consulta, vea [Creación de consultas semánticas](semantic-how-to-query-request.md).

Dada la consulta "how do clouds form" ("¿Cómo se forman las nubes?"), se devuelve la respuesta siguiente en la respuesta de la consulta:

```json
{
    "@search.answers": [
        {
            "key": "4123",
            "text": "Sunlight heats the land all day, warming that moist air and causing it to rise high into the   atmosphere until it cools and condenses into water droplets. Clouds generally form where air is ascending (over land in this case),   but not where it is descending (over the river).",
            "highlights": "Sunlight heats the land all day, warming that moist air and causing it to rise high into the   atmosphere until it cools and condenses into water droplets. Clouds generally form<em> where air is ascending</em> (over land in this case),   but not where it is<em> descending</em> (over the river).",
            "score": 0.94639826
        }
    ],
    "value": [
        {
            "@search.score": 0.5479723,
            "@search.rerankerScore": 1.0321671911515296,
            "@search.captions": [
                {
                    "text": "Like all clouds, it forms when the air reaches its dew point—the temperature at which an air mass is cool enough for its water vapor to condense into liquid droplets. This false-color image shows valley fog, which is common in the Pacific Northwest of North America.",
                    "highlights": "Like all<em> clouds</em>, it<em> forms</em> when the air reaches its dew point—the temperature at    which an air mass is cool enough for its water vapor to condense into liquid droplets. This false-color image shows valley<em> fog</em>, which is common in the Pacific Northwest of North America."
                }
            ],
            "title": "Earth Atmosphere",
            "content": "Fog is essentially a cloud lying on the ground. Like all clouds, it forms when the air reaches its dew point—the temperature at  \n\nwhich an air mass is cool enough for its water vapor to condense into liquid droplets.\n\nThis false-color image shows valley fog, which is common in the Pacific Northwest of North America. On clear winter nights, the \n\nground and overlying air cool off rapidly, especially at high elevations. Cold air is denser than warm air, and it sinks down into the \n\nvalleys. The moist air in the valleys gets chilled to its dew point, and fog forms. If undisturbed by winds, such fog may persist for \n\ndays. The Terra satellite captured this image of foggy valleys northeast of Vancouver in February 2010.\n\n\n",
            "locations": [
                "Pacific Northwest",
                "North America",
                "Vancouver"
            ]
    ]
        }
}

```

## <a name="tips-for-producing-high-quality-answers"></a>Sugerencias para generar respuestas de alta calidad

Para obtener los mejores resultados, devuelva respuestas semánticas en un corpus de documentos que tenga las siguientes características:

+ "searchFields" debe proporcionar campos que ofrezcan un texto suficiente en el que es probable que se encuentre una respuesta. Solo el texto textual de un documento puede aparecer como una respuesta.

+ Las cadenas de consulta no deben ser NULL (search=`*`) y la cadena debe tener las características de una pregunta, a diferencia de una búsqueda de palabras clave (una lista secuencial de términos o frases arbitrarios). Si la cadena de consulta no parece ser una pregunta, se omitirá el procesamiento de respuestas, aunque la solicitud especifique "answers" como parámetro de consulta.

+ La extracción semántica y el resumen tienen límites respecto al número de tokens por documento que se pueden analizar de manera oportuna. En términos prácticos, si tiene documentos grandes que se ejecutan en cientos de páginas, debe intentar dividir el contenido en documentos más pequeños en primer lugar.

## <a name="next-steps"></a>Pasos siguientes

+ [Introducción a la búsqueda semántica](semantic-search-overview.md)
+ [Algoritmo de clasificación semántica](semantic-ranking.md)
+ [Algoritmo de clasificación de similitud](index-ranking-similarity.md)
+ [Creación de consultas semánticas](semantic-how-to-query-request.md)
