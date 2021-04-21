---
title: Prácticas recomendadas para la creación de una aplicación de LUIS
description: Obtenga información sobre los procedimientos recomendados para conseguir los mejores resultados del modelo de la aplicación de LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/13/2021
ms.openlocfilehash: d5fa2a1e865a4f54de268e7ad756d1d4363f3b78
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107500216"
---
# <a name="best-practices-for-building-a-language-understanding-luis-app"></a>Procedimientos recomendados para crear aplicaciones de Language Understanding (LUIS)
Use el proceso de creación de aplicaciones para compilar la aplicación de LUIS:

* Creación de modelos de idioma (intenciones y entidades)
* Adición de algunas expresiones de ejemplo de entrenamiento (15-30 por intención)
* Publicación en el punto de conexión
* Probar desde el punto de conexión

Cuando la aplicación está [publicada](luis-how-to-publish-app.md), use el ciclo de vida de creación para agregar características, publicar y probar desde el punto de conexión. No empiece el siguiente ciclo de creación agregando más expresiones de ejemplo, ya que no permite que LUIS aprenda del modelo con expresiones de usuario del mundo real.

No expanda las expresiones hasta que el conjunto actual de expresiones de ejemplo y punto de conexión devuelva resultados seguros y con predicciones altas. Mejore las puntuaciones mediante el [aprendizaje activo](luis-concept-review-endpoint-utterances.md).




## <a name="do-and-dont"></a>Qué se debe hacer y qué no
En la lista siguiente, se incluyen los procedimientos recomendados para las aplicaciones de LUIS:

|Lo que es necesario hacer:|Lo que debe evitar:|
|--|--|
|[Planear el esquema](#do-plan-your-schema)|[Compilar y publicar sin un plan](#dont-publish-too-quickly)|
|[Definir diferentes intenciones](#do-define-distinct-intents)<br>[Agregar características a intenciones](#do-add-features-to-intents)<br>
[Usar entidades con aprendizaje automático](#do-use-machine-learned-entities) |[Agregar varias expresiones de ejemplo a las intenciones](#dont-add-many-example-utterances-to-intents)<br>[Usar unas pocas entidades sencillas](#dont-use-few-or-simple-entities) |
|[Buscar un punto óptimo entre ser demasiado genérico y demasiado específico para cada intención](#do-find-sweet-spot-for-intents)|[Usar LUIS como una plataforma de aprendizaje](#dont-use-luis-as-a-training-platform)|
|[Compilar la aplicación de forma iterativa con versiones](#do-build-your-app-iteratively-with-versions)<br>[Crear entidades para la descomposición del modelo](#do-build-for-model-decomposition)|[Agregar varias expresiones de ejemplo del mismo formato, omitiendo otros formatos](#dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats)|
|[Agregar patrones en iteraciones posteriores](#do-add-patterns-in-later-iterations)|[Combinar la definición de intenciones y entidades](#dont-mix-the-definition-of-intents-and-entities)|
|[Equilibrar las expresiones en todas las intenciones](#balance-your-utterances-across-all-intents), excepto en la intención None.<br>[Agregar expresiones de ejemplo a la intención None](#do-add-example-utterances-to-none-intent)|[Crear listas de frases con todos los valores posibles](#dont-create-phrase-lists-with-all-the-possible-values)|
|[Aprovechar la característica de sugerencia para un aprendizaje activo](#do-leverage-the-suggest-feature-for-active-learning)|[Agregar demasiados patrones](#dont-add-many-patterns)|
|[Supervisar el rendimiento de la aplicación con la prueba por lotes](#do-monitor-the-performance-of-your-app)|[Entrenar y publicar al agregar cada expresión de ejemplo](#dont-train-and-publish-with-every-single-example-utterance)|

## <a name="do-plan-your-schema"></a>Planee el esquema

Antes de empezar a compilar el esquema de la aplicación, debe identificar qué y dónde planea usar esta aplicación. Cuanto más exhaustivo y específico sea el planeamiento, mejor será la aplicación.

* Analice los usuarios de destino
* Defina los roles en su totalidad para representar la aplicación: voz, avatar y tratamiento de problemas (proactivo, reactivo)
* Identifique las interacciones de los usuarios (texto, voz) y a través de qué canales se efectúa, la entrega a las soluciones existentes o la creación de una nueva solución para esta aplicación.
* Recorrido completo del usuario
    * ¿Qué puede esperar que haga esta aplicación y qué no? * ¿Cuáles son las prioridades de lo que debe hacer?
    * ¿Cuáles son los principales casos de uso?
* Recopile datos: [obtenga más información](data-collection.md) acerca de la recopilación y preparación de los datos

## <a name="do-define-distinct-intents"></a>Definir diferentes intenciones
Asegúrese de que el vocabulario de cada intención sea solo para esa intención y no se superponga con otra intención. Por ejemplo, si quiere que una aplicación controle preparativos de viaje como vuelos y hoteles, puede elegir que estas áreas de asuntos sean intenciones independientes o que tengan una misma intención con entidades de datos específicos dentro de la expresión.

Si el vocabulario entre dos intenciones es el mismo, combine la intención y use entidades.

Considere las siguientes expresiones de ejemplo:

|Expresiones de ejemplo|
|--|
|Reservar un vuelo|
|Reservar un hotel|

`Book a flight` y `Book a hotel` utilizan el mismo vocabulario de `book a `. Este formato es el mismo, por lo que debería tratarse de la misma intención con las diferentes palabras de `flight` y `hotel` que las entidades extraídas.

## <a name="do-add-features-to-intents"></a>Agregar características a intenciones

Las características describen los conceptos de una intención. Una característica puede ser una lista de frases de palabras que son significativas para esa intención o una entidad que es significativa para dicha intención.

## <a name="do-find-sweet-spot-for-intents"></a>Buscar un punto óptimo para las intenciones
Use datos de predicción de LUIS para determinar si las intenciones se superponen. Las intenciones que se superponen confunden a LUIS. El resultado es que la intención con mayor puntuación es demasiado cercana a otra intención. Dado que LUIS no usa exactamente la misma ruta a través de los datos para el aprendizaje todas las veces, una intención que se superpone tiene una posibilidad de ser la primera o segunda en el aprendizaje. Quiere que la puntuación de la expresión de cada intención esté más alejada para que esto no suceda. Una buena distinción de las intenciones debería dar como resultado cada vez la intención superior que se esperaba.

## <a name="do-use-machine-learned-entities"></a>Use entidades con aprendizaje automático

Las entidades con aprendizaje automático se adaptan a la aplicación y requieren que el etiquetado sea correcto. Si no va a usar entidades con aprendizaje automático, es posible que esté usando la herramienta equivocada.

Las entidades con aprendizaje automático pueden usar otras entidades como características. Estas otras entidades pueden ser entidades personalizadas como entidades de expresiones regulares o entidades de lista, o bien se pueden usar como características las entidades creadas previamente.

Obtenga más información sobre las [entidades con aprendizaje automático eficaces](luis-concept-entity-types.md#machine-learned-ml-entity).

<a name="#do-build-the-app-iteratively"></a>

## <a name="do-build-your-app-iteratively-with-versions"></a>Compilar la aplicación de forma iterativa con versiones

Cada ciclo de creación debe estar en una nueva [versión](./luis-concept-app-iteration.md), clonada de una versión existente.

## <a name="do-build-for-model-decomposition"></a>Compilar para la descomposición del modelo

La descomposición del modelo tiene el siguiente proceso típico:

* crear una **intención** basada en las intenciones del usuario de la aplicación cliente.
* agregar entre 15 a 30 expresiones de ejemplo basadas en la entrada de usuario del mundo real
* etiquetar el concepto de datos de nivel superior en la expresión de ejemplo
* dividir el concepto de datos en subentidades
* agregar características a subentidades
* agregar características a intenciones

Cuando haya creado la intención y agregado las expresiones de ejemplo, en el siguiente ejemplo se describe la descomposición de entidades.

Comience por identificar los conceptos de datos completos que quiere extraer en una expresión. Esta es la entidad con aprendizaje automático. A continuación, descomponga la frase en partes. Esto incluye la identificación de subentidades y las características.

Por ejemplo, si desea extraer una dirección, la entidad superior con aprendizaje automático podría llamarse `Address`. Al crear la dirección, identifique algunas de sus subentidades, como la dirección postal, la ciudad, el estado y el código postal.

Continúe descomponiendo esos elementos mediante:
* La adición de una característica obligatoria del código postal como una entidad de expresión regular.
* La descomposición de la dirección de la calle en partes:
    * Un **número de calle** con una característica obligatoria de una entidad de número precompilada.
    * Un **nombre de la calle**.
    * Un **tipo de calle** con una característica obligatoria de una entidad de lista que incluya palabras como avenida, plaza, carretera y vía.

La API de creación V3 permite la descomposición del modelo.

## <a name="do-add-patterns-in-later-iterations"></a>Agregar patrones en iteraciones posteriores

Es necesario entender cómo se comporta la aplicación antes de agregar [patrones](luis-concept-patterns.md), ya que los patrones tienen mayor peso que las expresiones de ejemplo y sesgarán la confianza.

Una vez que comprenda cómo se comporta la aplicación, agregue los patrones que se aplican a la aplicación. No es necesario agregarlos con cada [iteración](luis-concept-app-iteration.md).

No existe ningún inconveniente en agregarlas al principio del diseño del modelo, pero es más fácil ver cómo cada patrón cambia el modelo después de que el modelo se haya probado con expresiones.

<a name="balance-your-utterances-across-all-intents"></a>

## <a name="do-balance-your-utterances-across-all-intents"></a>Equilibrar las expresiones en todas las intenciones

Para que las predicciones de LUIS sean precisas, la cantidad de expresiones de ejemplo en cada intención (excepto para la intención None), debe ser relativamente igual.

Si tiene una intención con 100 expresiones de ejemplo y una intención con 20 expresiones de ejemplo, la intención de las 100 expresiones tendrá una mayor tasa de predicción.

## <a name="do-add-example-utterances-to-none-intent"></a>Agregar expresiones de ejemplo a la intención None

Esta es la intención de reserva, que indica todo lo que está fuera de la aplicación. Agregue una expresión de ejemplo en la intención None por cada 10 expresiones de ejemplo del resto de la aplicación de LUIS.

## <a name="do-leverage-the-suggest-feature-for-active-learning"></a>Aprovechar la característica de sugerencia para un aprendizaje activo

Use la **revisión de las expresiones del punto de conexión** del [aprendizaje activo](luis-how-to-review-endpoint-utterances.md) de forma habitual, en lugar de agregar más expresiones de ejemplo a las intenciones. Dado que la aplicación recibe de forma constante expresiones de punto de conexión, esta lista crece y cambia.

## <a name="do-monitor-the-performance-of-your-app"></a>Supervisar el rendimiento de la aplicación

Supervise la precisión de las predicciones mediante un conjunto de [pruebas de lote](./luis-how-to-batch-test.md).

Mantenga un conjunto de expresiones que no se use como [expresiones de ejemplo](luis-concept-utterance.md) o de punto de conexión. Siga mejorando la aplicación del conjunto de pruebas. Adapte el conjunto de pruebas para que refleje expresiones de usuarios reales. Use este conjunto de pruebas para evaluar cada iteración o versión de la aplicación.

## <a name="dont-publish-too-quickly"></a>No publique demasiado rápido

Publicar la aplicación demasiado rápido, sin una [planeación adecuada](#do-plan-your-schema), puede dar lugar a varios problemas como, por ejemplo:

* La aplicación no funcionará en su escenario real con un nivel aceptable de rendimiento.
* El esquema (intenciones y entidades) no sería adecuado y, si ha desarrollado la lógica de la aplicación cliente después del esquema, puede que tenga que volver a escribirla desde el principio. Esto provocaría retrasos inesperados y un costo adicional para el proyecto en el que está trabajando.
* Las expresiones que agregue al modelo podrían provocar un sesgo hacia el conjunto de expresiones de ejemplo que es difícil de depurar e identificar. Esto también dificultará la eliminación de ambigüedades una vez que se haya confirmado un esquema determinado.

## <a name="dont-add-many-example-utterances-to-intents"></a>Agregar varias expresiones de ejemplo a las intenciones

Una vez publicada la aplicación, agregue solo expresiones de aprendizaje activo en el proceso de ciclo de vida del desarrollo. Si las expresiones son demasiado similares, agregue un patrón.

## <a name="dont-use-few-or-simple-entities"></a>No usar pocas entidades o entidades simples

Las entidades se crean para la extracción y predicción de datos. Es importante que cada intención tenga entidades con aprendizaje automático que describan los datos en la intención. Esto ayuda a LUIS a predecir la intención, incluso si la aplicación cliente no necesita usar la entidad extraída.

## <a name="dont-use-luis-as-a-training-platform"></a>No usar LUIS como una plataforma de aprendizaje

LUIS es específico de un dominio de modelo de lenguaje. Esto no está pensado para que funcione como una plataforma general de aprendizaje de lenguaje natural.

## <a name="dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats"></a>No agregar varias expresiones de ejemplo del mismo formato, omitiendo otros formatos

LUIS espera variaciones en las expresiones de una intención. Las expresiones pueden variar y conservar el mismo significado general. Las variaciones pueden incluir la longitud de la expresión, así como la elección y ubicación de las palabras.

|No usar el mismo formato|Usar un formato diferente|
|--|--|
|Comprar un billete a Seattle<br>Comprar un billete a París<br>Comprar un billete a Orlando|Comprar 1 billete a Seattle<br>Reservar dos asientos en el vuelo nocturno a París el próximo lunes<br>Quiero adquirir 3 billetes a Orlando para las vacaciones de primavera|

La segunda columna usa diferentes verbos (comprar, reservar y adquirir), distintas cantidades (1, dos y 3) y diferentes colocaciones de las palabras, pero todas tienen la misma intención de comprar billetes de avión para viajar.

## <a name="dont-mix-the-definition-of-intents-and-entities"></a>No combinar la definición de intenciones y entidades

Cree una intención para cualquier acción que llevará a cabo el bot. Use entidades como parámetros que hacen posible esa acción.

Para un bot que reservará vuelos, cree la intención **BookFlight**. No cree una intención para cada compañía aérea o destino. Use esos datos como [entidades](luis-concept-entity-types.md) y márquelos en las expresiones de ejemplo.

## <a name="dont-create-phrase-lists-with-all-the-possible-values"></a>No crear listas de frases con todos los valores posibles

Proporcione algunos ejemplos en las [listas de frases](luis-concept-feature.md), pero no todas las palabras o frases. LUIS generaliza y tiene en cuenta el contexto.

## <a name="dont-add-many-patterns"></a>No agregar muchos patrones

No agregue demasiados [patrones](luis-concept-patterns.md). LUIS está pensado para aprender rápidamente con pocos ejemplos. No sobrecargue el sistema innecesariamente.

## <a name="dont-train-and-publish-with-every-single-example-utterance"></a>No entrenar y publicar con cada expresión de ejemplo

Agregue 10 o 15 expresiones antes de entrenar y publicar. De esta forma, puede ver el impacto en la precisión de la predicción. Si agrega una única expresión, puede que no vea ningún impacto en la puntuación.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [planear la aplicación](luis-how-plan-your-app.md) en la aplicación de LUIS.