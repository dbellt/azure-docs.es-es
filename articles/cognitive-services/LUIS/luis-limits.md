---
title: 'Límites: LUIS'
description: En este artículo contiene los límites conocidos de Language Understanding (LUIS) de Azure Cognitive Services. LUIS tiene varias áreas de límites. El límite de modelo controla las intenciones, las entidades y las características de LUIS. Los límites de cuota se basan en el tipo de clave. La combinación de teclado controla el sitio web de LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 06/04/2020
ms.openlocfilehash: 1f917087eb15d8c77356995299e27dfc1657cb5d
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107497207"
---
# <a name="limits-for-your-luis-model-and-keys"></a>Límites de las claves y el modelo de LUIS
LUIS tiene varias áreas de límites. La primera es el [límite de modelo](#model-limits), que controla las intenciones, las entidades y las características de LUIS. La segunda área son los [límites de cuota](#key-limits) según el tipo de clave. Una tercera área de límites es la [combinación de teclas](#keyboard-controls), para controlar el sitio web de LUIS. Una cuarta área es la [asignación de regiones del mundo](luis-reference-regions.md) entre el sitio web de creación de LUIS y las API de [punto de conexión](luis-glossary.md#endpoint) de LUIS.

<a name="model-boundaries"></a>

## <a name="model-limits"></a>Límites del modelo

Si la aplicación supera los límites de modelo de LUIS, puede usar una aplicación de [envíos de LUIS](luis-concept-enterprise.md#dispatch-tool-and-model) o un [contenedor de LUIS](luis-container-howto.md).

|Área|Límite|
|--|:--|
| [Nombre de la aplicación][luis-get-started-create-app] | *Máximo de caracteres predeterminado |
| APLICACIONES| 500 aplicaciones por recurso de creación de Azure |
| [Pruebas por lotes][batch-testing]| 10 conjuntos de datos, 1000 expresiones por cada conjunto de datos|
| Lista explícita | 50 por aplicación|
| Entidades externas | sin límites |
| [Intenciones][intents]|500 por aplicación: 499 intenciones personalizadas y la intención _Ninguno_ necesaria.<br>La aplicación [basada en la distribución](https://aka.ms/dispatch-tool) tiene 500 orígenes de distribución correspondientes.|
| [Listar entidades](./luis-concept-entity-types.md) | Elemento principal: 50, elemento secundario: 20 000 elementos. El nombre canónico es *Máximo de caracteres predeterminado. Los valores de los sinónimos no tienen ninguna restricción de longitud. |
| [Entidades de aprendizaje automático y roles](./luis-concept-entity-types.md):<br> compuesta,<br>simple,<br>rol de entidad|Un límite de 100 entidades primarias o 330 entidades, el que sea que alcance primero el usuario. Un rol cuenta como una entidad para los fines de este límite. Un ejemplo es una composición con una entidad sencilla que tiene dos roles: 1 compuesta + 1 simple + 2 roles = 4 de las 330 entidades.<br>Las entidades secundarias se pueden anidar hasta 5 niveles, con un máximo de 20 elementos secundarios por nivel.|
|Modelo como característica| El número máximo de modelos que se pueden usar como característica para un modelo específico es de 10 modelos. El número máximo de listas de frases utilizadas como característica para un modelo específico es de 10 listas de frases.|
| [Versión preliminar: entidades de lista dinámica](./luis-migration-api-v3.md)|2 listas de ~1 K por solicitud de punto de conexión de predicción de consulta|
| [Patrones](luis-concept-patterns.md)|500 patrones por aplicación.<br>El patrón puede contener 400 caracteres como máximo.<br>3 entidades Pattern.any por patrón<br>Máximo de 2 textos opcionales anidados en el patrón|
| [Pattern.any](./luis-concept-entity-types.md)|100 por aplicación, 3 entidades Pattern.any por patrón |
| [Lista de frases][phrase-list]|500 listas de frases. 10 listas de frases globales debido al modelo como límite de características. La lista de frases no intercambiable tiene un máximo de 5000 frases. La lista de frases intercambiables tiene un máximo de 50 000 frases. Número máximo de frases totales por aplicación de 500 000 frases.|
| [Entidades precompiladas](./howto-add-prebuilt-models.md) | ilimitado|
| [Entidades de expresión regular](./luis-concept-entity-types.md)|20 entidades<br>Máximo de 500 caracteres por patrón de la entidad de expresiones regulares|
| [Roles](./luis-concept-entity-types.md)|300 roles por aplicación. 10 roles por entidad|
| [Expresión][utterances] | 500 caracteres<br><br>Si tiene texto cuyo número de caracteres supera este límite, tiene que segmentar la expresión antes de especificar en LUIS para recibir respuestas de intención individuales por segmento. Hay interrupciones obvias con las que puede trabajar, como los signos de puntuación y las pausas prolongadas al hablar.|
| [Ejemplos de expresiones][utterances] | 15 000 por aplicación: no hay ningún límite en el número de expresiones por intención<br><br>Si necesita entrenar la aplicación con más ejemplos, use un enfoque de modelo de [envío](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch). Puede entrenar aplicaciones de LUIS individuales (conocidas como aplicaciones secundarias en la aplicación de envío principal) con una o varias intenciones y, después, entrenar una aplicación de envío que cree ejemplos de las expresiones de cada una de las aplicaciones de LUIS secundarias para dirigir la solicitud de predicción a la aplicación secundaria correcta. |
| [Versiones](./luis-concept-app-iteration.md)| 100 versiones por aplicación |
| [Nombre de versión][luis-how-to-manage-versions] | 128 caracteres |

*El valor máximo predeterminado de caracteres es 50.

<a name="intent-and-entity-naming"></a>

## <a name="name-uniqueness"></a>Exclusividad del nombre

Los nombres de objeto deben ser únicos al compararse con otros objetos del mismo nivel.

|Objetos|Restricciones|
|--|--|
|Intención, entidad|Todos los nombres de intenciones y entidades deben ser únicos en una versión de una aplicación.|
|Componentes de la entidad de ML|Todos los componentes de una entidad de aprendizaje automático (entidades secundarias) tienen que ser únicos, dentro de esa entidad, para los componentes del mismo nivel.|
|Características | Todas las características con nombre, como las listas de frases, deben ser únicas dentro de una versión de una aplicación.|
|Roles de entidad|Todos los roles de una entidad o de un componente de una entidad deben ser únicos cuando están en el mismo nivel de entidad (primario, secundario, terciario, etc.).|

## <a name="object-naming"></a>Denominación de objeto

No use los siguientes caracteres en los siguientes nombres.

|Object|Excluir caracteres|
|--|--|
|Nombres de intención, entidad y rol|`:`<br>`$` <br> `&`|
|Nombre de versión|`\`<br> `/`<br> `:`<br> `?`<br> `&`<br> `=`<br> `*`<br> `+`<br> `(`<br> `)`<br> `%`<br> `@`<br> `$`<br> `~`<br> `!`<br> `#`|

## <a name="resource-usage-and-limits"></a>Uso y límites de recursos

Language Understanding tiene recursos independientes, un tipo para la creación y un tipo para las consultas en el punto de conexión de la predicción. Para más información sobre las diferencias entre los tipos de claves, consulte [Claves de creación y del punto de conexión de consulta de predicciones en LUIS](luis-how-to-azure-subscription.md).

<a name="key-limits"></a>

### <a name="authoring-resource-limits"></a>Límites de recursos de creación

Use el elemento _kind_, `LUIS.Authoring`, al filtrar los recursos en Azure Portal. LUIS tiene un límite de 500 aplicaciones por recurso de creación de Azure.

|Recurso de creación|TPS de creación|
|--|--|
|Inicio|1 millón/mes, 5/segundo|
|F0: nivel Gratis |1 millón/mes, 5/segundo|

* TPS: transacciones por segundo

[Más información sobre precios.][pricing]

### <a name="query-prediction-resource-limits"></a>Consulta de los límites de recurso de predicción

Use el elemento _kind_, `LUIS`, al filtrar los recursos de Azure Portal. El recurso de punto de conexión para predicción de consulta de LUIS, que se utiliza en tiempo de ejecución, solo es válido para las consultas de punto de conexión.

|Consulta de recurso de predicción|TPS de consulta|
|--|--|
|F0: nivel Gratis |10 000/mes, 5/segundo|
|S0: nivel Estándar|50/segundo|

### <a name="sentiment-analysis"></a>análisis de opiniones

La [integración de análisis de opiniones](luis-how-to-publish-app.md#enable-sentiment-analysis), que proporciona información de opiniones, se ofrece sin necesidad de otro recurso de Azure.

### <a name="speech-integration"></a>Integración de voz

La [integración de voz](../speech-service/how-to-recognize-intents-from-speech-csharp.md) proporciona 1000 solicitudes de punto de conexión por costo de unidad.

[Más información sobre precios.][pricing]

## <a name="keyboard-controls"></a>Controles de teclado

|Entrada de teclado | Descripción |
|--|--|
|Control+E|cambia entre los tokens y entidades en la lista de expresiones|

## <a name="website-sign-in-time-period"></a>Período de tiempo de inicio de sesión en el sitio web

El acceso de inicio de sesión es válido durante **60 minutos**. Después de este tiempo, recibirá este error. Debe volver a iniciar sesión.

[luis-get-started-create-app]: ./luis-get-started-create-app.md
[batch-testing]: ./luis-concept-test.md#batch-testing
[intents]: ./luis-concept-intent.md
[phrase-list]: ./luis-concept-feature.md
[utterances]: ./luis-concept-utterance.md
[luis-how-to-manage-versions]: ./luis-how-to-manage-versions.md
[pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
<!-- TBD: fix this link -->
[speech-to-intent-pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
