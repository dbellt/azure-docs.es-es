---
title: Búsqueda semántica
titleSuffix: Azure Cognitive Search
description: Conozca cómo Cognitive Search usa modelos de búsqueda semántica de aprendizaje profundo de Bing para crear resultados de búsqueda más intuitivos.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/01/2021
ms.custom: references_regions
ms.openlocfilehash: 04300b8d148bb22bf585aa81481c475b347ad462
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222051"
---
# <a name="semantic-search-in-azure-cognitive-search"></a>Búsqueda semántica en Azure Cognitive Search

> [!IMPORTANT]
> La búsqueda semántica se encuentra en versión preliminar pública y solo está disponible mediante la API REST en versión preliminar y el portal. Las características en versión preliminar se ofrecen tal cual, según las [Condiciones de uso complementarias](https://azure.microsoft.com/support/legal/preview-supplemental-terms/), y no se garantiza que tengan la misma implementación en la disponibilidad general. Estas características son facturables. Para más información, consulte [Disponibilidad y precios](semantic-search-overview.md#availability-and-pricing).

La búsqueda semántica es una colección de funcionalidades relacionadas con las consultas que agrega relevancia semántica y comprensión del lenguaje a los resultados de la búsqueda. La *clasificación semántica* busca el contexto y la relación entre los términos, elevando las coincidencias que tienen más sentido en función de la consulta. Language Understanding busca *descripciones* y *respuestas* en el contenido que resuman el documento coincidente o respondan a una pregunta, que luego se puede representar en una página de resultados de búsqueda para una experiencia de búsqueda más productiva.

Los modelos previamente entrenados de última generación se usan para el resumen y la clasificación. Para mantener el rápido rendimiento que los usuarios esperan de la búsqueda, el resumen semántico y la clasificación se aplican solo a los 50 resultados principales, en función de la puntuación del [algoritmo de puntuación de similitud predeterminada](index-similarity-and-scoring.md#similarity-ranking-algorithms). Al usar los resultados como un corpus de documento, la clasificación semántica vuelve a puntuar los resultados en función de la intensidad semántica de la coincidencia.

La tecnología subyacente es de Bing y Microsoft Research, y se integra con la infraestructura de Cognitive Search como una característica de complemento. Para más información sobre las inversiones en investigación e inteligencia artificial que respaldan la búsqueda semántica, consulte [Formas en que la inteligencia artificial de Bing Azure se usa en Cognitive Search (blog de Microsoft Research)](https://www.microsoft.com/research/blog/the-science-behind-semantic-search-how-ai-from-bing-is-powering-azure-cognitive-search/).

En el vídeo siguiente se proporciona información general de las funcionalidades.

> [!VIDEO https://www.youtube.com/embed/yOf0WfVd_V0]

## <a name="components-and-workflow"></a>Componentes y flujo de trabajo

La búsqueda semántica mejora la precisión y la coincidencia gracias a la adición de las siguientes funcionalidades:

| Característica | Descripción |
|---------|-------------|
| [Corrector ortográfico](speller-how-to-add.md) | Corrige los errores tipográficos antes de que los términos de consulta lleguen al motor de búsqueda. |
| [Clasificación semántica](semantic-ranking.md) | Usa el contexto o el significado semántico para calcular una nueva puntuación de relevancia. |
| [Títulos y resaltados semánticos](semantic-how-to-query-request.md) | Son las oraciones y frases de un documento que mejor resumen el contenido; se resaltan los pasajes clave para facilitar el análisis. Los títulos que resumen un resultado son útiles cuando los campos de contenido individuales son demasiado densos para la página de resultados. El texto resaltado eleva los términos y frases más relevantes para que los usuarios puedan determinar rápidamente por qué se consideró relevante una coincidencia. |
| [Respuestas semánticas](semantic-answers.md) | Es una subestructura opcional y adicional que se devuelve desde una consulta semántica. Proporciona una respuesta directa a una consulta que se parece a una pregunta. |

### <a name="order-of-operations"></a>Orden de las operaciones

Los componentes de la búsqueda semántica amplían la canalización de ejecución de consultas existente en ambas direcciones. Si habilita la corrección ortográfica, el [corrector ortográfico](speller-how-to-add.md) corrige los errores tipográficos al principio de la consulta, antes de que los términos de la consulta lleguen al motor de búsqueda.

:::image type="content" source="media/semantic-search-overview/semantic-workflow.png" alt-text="Componentes semánticos de una ejecución de consulta" border="true":::

La ejecución de la consulta se realiza de la manera habitual, con el análisis de términos, varios exámenes y el análisis de los índices invertidos. El motor recupera los documentos mediante la coincidencia de tokens y puntúa los resultados mediante el [algoritmo de puntuación de similitud predeterminada](index-similarity-and-scoring.md#similarity-ranking-algorithms). Las puntuaciones se calculan en función del grado de similitud lingüística entre los términos de consulta y los términos coincidentes del índice. Si los ha definido, los perfiles de puntuación también se aplican en esta fase. A continuación, los resultados se pasan al subsistema de búsqueda semántica.

En el paso de preparación, el corpus del documento que se ha devuelto desde el conjunto de resultados inicial se analiza a nivel de oración y de párrafo para buscar pasajes que resuman cada documento. A diferencia de la búsqueda de palabras clave, este paso usa la lectura y la comprensión automáticas para evaluar el contenido. Con esta fase de procesamiento de contenido, una consulta semántica devuelve [descripciones](semantic-how-to-query-request.md) y [respuestas](semantic-answers.md). Para formular estos elementos, la búsqueda semántica usa la representación del lenguaje para extraer y resaltar los pasajes principales que mejor resuman un resultado. Si la consulta de búsqueda es una pregunta y se solicitan respuestas, la respuesta también incluirá el pasaje de texto que mejor responda a la pregunta, según se exprese en la consulta de búsqueda. 

En el caso de los títulos y las respuestas, se usa el texto existente en la formulación. Tenga en cuenta que los modelos semánticos no componen nuevas oraciones o frases a partir del contenido disponible, ni aplican la lógica para llegar a nuevas conclusiones. En resumen, el sistema nunca devolverá contenido que no exista.

A continuación, los resultados se vuelven a puntuar en función de la [similitud conceptual](semantic-ranking.md) de los términos de la consulta.

Para usar las funcionalidades semánticas en las consultas, deberá realizar pequeñas modificaciones en la [solicitud de búsqueda](semantic-how-to-query-request.md), pero no necesitará realizar más configuraciones ni nuevas indexaciones.

## <a name="availability-and-pricing"></a>Disponibilidad y precios

Puede obtener las funcionalidades semánticas mediante el [registro de inicio de sesión](https://aka.ms/SemanticSearchPreviewSignup), en los servicios de búsqueda que se crean en un nivel Estándar (S1, S2, S3) y que estén ubicados en una de estas regiones: Centro-norte de EE. UU., Oeste de EE. UU., Oeste de EE. UU. 2, Este de EE. UU. 2, Norte de Europa y Oeste de Europa. 

La corrección ortográfica está disponible en las mismas regiones, pero no tiene restricciones de nivel. Si tiene un servicio existente que cumpla los criterios de nivel y región, solo es necesario que se registre.

Entre el lanzamiento de la versión preliminar del 2 de marzo hasta finales de abril, la corrección ortográfica y la clasificación semántica se ofrecen de forma gratuita. Después de abril, los costos computacionales de ejecutar esta funcionalidad pasarán a ser un evento facturable. El costo esperado es de aproximadamente 500 USD/mes por 250 000 consultas. Puede encontrar información detallada de los costos documentada en la [página de precios de Cognitive Search](https://azure.microsoft.com/pricing/details/search/) y en [Estimación y administración de los costos](search-sku-manage-costs.md).

## <a name="next-steps"></a>Pasos siguientes

Un nuevo tipo de consulta permite la clasificación de relevancia y las estructuras de respuesta de la búsqueda semántica.

Para comenzar, [cree una consulta semántica](semantic-how-to-query-request.md). O bien, consulte los siguientes artículos para obtener información relacionada.

+ [Adición de la corrección ortográfica a los términos de consulta](speller-how-to-add.md)
+ [Devolución de una respuesta semántica](semantic-answers.md)
+ [Clasificación semántica](semantic-ranking.md)
+ [Introducción a la búsqueda semántica (entrada de blog)](https://techcommunity.microsoft.com/t5/azure-ai/introducing-semantic-search-bringing-more-meaningful-results-to/ba-p/2175636)
+ [Información significativa con las funcionalidades semánticas (vídeo de The AI Show)](https://channel9.msdn.com/Shows/AI-Show/Find-meaningful-insights-using-semantic-capabilities-in-Azure-Cognitive-Search)