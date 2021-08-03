---
title: Búsqueda semántica
titleSuffix: Azure Cognitive Search
description: Conozca cómo Cognitive Search usa modelos de búsqueda semántica de aprendizaje profundo de Bing para crear resultados de búsqueda más intuitivos.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/25/2021
ms.custom: references_regions
ms.openlocfilehash: c55ca73d63fb380dece0350de482dcc9087bd315
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111744816"
---
# <a name="semantic-search-in-azure-cognitive-search"></a>Búsqueda semántica en Azure Cognitive Search

> [!IMPORTANT]
> La búsqueda semántica se encuentra en versión preliminar pública y solo está disponible mediante la API REST en versión preliminar y el portal. Las características en versión preliminar se ofrecen tal cual, según las [Condiciones de uso complementarias](https://azure.microsoft.com/support/legal/preview-supplemental-terms/), y no se garantiza que tengan la misma implementación en la disponibilidad general. Estas características son facturables. Para más información, consulte [Disponibilidad y precios](semantic-search-overview.md#availability-and-pricing).

La búsqueda semántica es una colección de funcionalidades relacionadas con las consultas que agrega relevancia semántica y comprensión del lenguaje a los resultados de la búsqueda. Este artículo es una introducción general a la búsqueda semántica, con descripciones de cada característica y su funcionamiento colectivo. La tecnología se describe en el vídeo insertado y la disponibilidad y los precios aparece en la sección del final.

La búsqueda semántica es una característica Prémium. Si necesita información general, le recomendamos este artículo, pero si prefiere empezar a trabajar, siga estos pasos:

1. [Compruebe los requisitos regionales y de nivel de servicio](#availability-and-pricing).
1. [Regístrese para el programa de versión preliminar](https://aka.ms/SemanticSearchPreviewSignup). La solicitud puede tardar hasta dos días laborables en procesarse.
1. Tras la aceptación, cree o modifique las consultas para devolver [títulos y resaltados semánticos](semantic-how-to-query-request.md).
1. Agregue algunas propiedades más para devolver también [respuestas semánticas](semantic-answers.md).
1. También puede incluir una propiedad de [corrección ortográfica](speller-how-to-add.md) para maximizar la precisión y la coincidencia.

## <a name="what-is-semantic-search"></a>¿Qué es la búsqueda semántica?

La búsqueda semántica es una capa opcional de inteligencia artificial relacionada con las consultas. Cuando se habilita en el servicio de búsqueda, extiende la canalización de ejecución de consultas tradicional de dos maneras. En primer lugar, agrega un modelo de clasificación semántica opcional; y, en segundo lugar, devuelve títulos y respuestas en la respuesta.

La *clasificación semántica* busca el contexto y la relación entre los términos, elevando las coincidencias que tienen más sentido en función de la consulta. Language Understanding busca resúmenes o *títulos* y *respuestas* en el contenido, y los incluye en la respuesta, que luego se puede representar en una página de resultados de búsqueda para una experiencia de búsqueda más productiva.

Los modelos previamente entrenados de última generación se usan para el resumen y la clasificación. Para mantener el rápido rendimiento que los usuarios esperan de la búsqueda, el resumen semántico y la clasificación se aplican solo a los 50 resultados principales, en función de la puntuación del [algoritmo de puntuación de similitud predeterminada](index-similarity-and-scoring.md#similarity-ranking-algorithms). Al usar los resultados como un corpus de documento, la clasificación semántica vuelve a puntuar los resultados en función de la intensidad semántica de la coincidencia.

La tecnología subyacente es de Bing y Microsoft Research, y se integra con la infraestructura de Cognitive Search como una característica de complemento. Para más información sobre las inversiones en investigación e inteligencia artificial que respaldan la búsqueda semántica, consulte [Formas en que la inteligencia artificial de Bing Azure se usa en Cognitive Search (blog de Microsoft Research)](https://www.microsoft.com/research/blog/the-science-behind-semantic-search-how-ai-from-bing-is-powering-azure-cognitive-search/).

En el vídeo siguiente se proporciona información general de las funcionalidades.

> [!VIDEO https://www.youtube.com/embed/yOf0WfVd_V0]

## <a name="feature-description-and-workflow"></a>Descripción y flujo de trabajo de características

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

## <a name="semantic-capabilities-and-limitations"></a>Funcionalidades y limitaciones semánticas

La búsqueda semántica es una tecnología novedosa, por lo que es importante establecer expectativas sobre lo que puede y no puede hacer.

Mejora la calidad de los resultados de la búsqueda de dos maneras. En primer lugar, la promoción de documentos que están semánticamente más cerca de la intención de la consulta original es una ventaja importante. En segundo lugar, los resultados son de uso más inmediato cuando los títulos, y posiblemente las respuestas, están presentes en la página. En todo momento, el motor está trabajando con contenido existente. Los modelos de lenguaje usados en la búsqueda semántica están diseñados para extraer una cadena intacta que parece una respuesta, pero no intentará crear una nueva cadena como respuesta a una consulta o como un título para un documento coincidente.

La búsqueda semántica no es un motor lógico y no deduce información de diferentes fragmentos de contenido del documento o corpus de documentos. Por ejemplo, dada una consulta sobre "complejos hoteleros en el desierto", que no tiene ninguna entrada geográfica, el motor no producirá coincidencias para hoteles ubicados en Arizona o Nevada, aunque ambos estados tengan desiertos. De igual manera, si la consulta incluye la cláusula "en los últimos 5 años", el motor no calculará un intervalo de tiempo basado en la fecha actual que se va a devolver.

En Cognitive Search, los mecanismos que podrían ser útiles para los escenarios anteriores incluyen [mapas de sinónimos](search-synonyms.md) que le permiten crear asociaciones entre términos que son distintos externamente, o [filtros de fecha](search-query-odata-filter.md) especificados como una expresión de OData.

## <a name="availability-and-pricing"></a>Disponibilidad y precios

La búsqueda semántica está disponible a través del [registro de suscripción](https://aka.ms/SemanticSearchPreviewSignup). Desde el lanzamiento de la versión preliminar del 2 de marzo hasta primeros de julio, las características semánticas se ofrecen de forma gratuita.

| Característica | Nivel | Region (Región) | Suscripción | Precios proyectados |
|---------|------|--------|---------------------|-------------------|
| Búsqueda semántica (títulos, resaltados, respuestas) | Nivel estándar (S1, S2, S3) | Centro-norte de EE. UU., Oeste de EE. UU., Oeste de EE. UU. 2, Este de EE. UU. 2, Norte de Europa, Oeste de Europa | Obligatorio | Desde principios de julio, el precio esperado de las primeras 250 000 consultas son 500 USD al mes y 2 USD por cada 1000 consultas adicionales.  |
| Corrector ortográfico | Any | Centro-norte de EE. UU., Oeste de EE. UU., Oeste de EE. UU. 2, Este de EE. UU. 2, Norte de Europa, Oeste de Europa | Obligatorio | Ninguno (gratis) |

Hay un [registro de suscripción](https://aka.ms/SemanticSearchPreviewSignup) para las características semánticas y la revisión ortográfica. 

Puede usar la revisión ortográfica sin búsqueda semántica, de forma gratuita. Los cargos por la búsqueda semántica se aplican cuando las solicitudes de consulta incluyen `queryType=semantic` y la cadena de búsqueda no está vacía (por ejemplo, `search=pet friendly hotels in new york`. Las búsquedas vacías (consultas donde `search=*`) no se cobran.

La información de los precios finales se documentará en la [página de precios de Cognitive Search](https://azure.microsoft.com/pricing/details/search/) y en [Estimación y administración de los costos](search-sku-manage-costs.md).

## <a name="next-steps"></a>Pasos siguientes

[Regístrese](https://aka.ms/SemanticSearchPreviewSignup) para obtener la versión preliminar en un servicio de búsqueda que cumple con los requisitos regionales y de nivel indicados en la sección anterior.

La solicitud puede tardar hasta dos días laborables en procesarse. Cuando el servicio esté listo, [cree una consulta semántica](semantic-how-to-query-request.md) para ver la clasificación semántica en acción.
