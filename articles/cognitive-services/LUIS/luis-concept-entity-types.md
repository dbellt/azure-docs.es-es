---
title: 'Tipos de entidades: LUIS'
description: Una entidad extrae datos de una expresión de usuario en tiempo de ejecución de predicción. Un propósito _opcional_ y secundario es impulsar la predicción de la intención o de otras entidades usando la entidad como una característica.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/26/2021
ms.openlocfilehash: a075a84322dc11be352470d50478979b975f0292
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108140900"
---
# <a name="entities-in-luis"></a>Entidades de LUIS

Una entidad es un elemento que es importante para la intención del usuario. Las entidades definen los datos que se pueden extraer de la expresión y son esenciales para completar la acción necesaria de un usuario. Por ejemplo:

|Expresión|Intención pronosticada|Entidades extraídas|Explicación|
|--|--|--|--|
|Hola, ¿cómo está?|Saludo|-|No hay nada que extraer.|
|Quiero pedir una pizza pequeña|pedirPizza| "pequeña" | La entidad "tamaño" se extrae como "pequeña".|
|Apagar la luz del dormitorio|apagar| "dormitorio" | La entidad "Habitación" se extrae como "dormitorio".|
|Comprobar el saldo de mi cuenta de ahorro que termina en 4406|comprobarSaldo| "ahorro", "4406" | La entidad "tipoDeCuenta" se extrae como "ahorro" y la entidad "númeroDeCuenta" como "4406".|
|Comprar 3 billetes a Nueva York|comprarBilletes| "3", "Nueva York" | La entidad "númeroDeBilletes" se extrae como "3" y la entidad "Destino" como "Nueva York".|

Las entidades son opcionales, pero muy recomendables. No es necesario que cree entidades para cada concepto de la aplicación, solo para aquellos en los que:

* La aplicación cliente necesita los datos. 
* La entidad actúa como una sugerencia o señal para otra entidad o intención. Para más información sobre las entidades como características, vaya a [Entidades como características](#entities-as-features).

## <a name="entity-types"></a>Tipos de entidades

Para crear una entidad, debe darle un nombre y un tipo. Existen varios tipos de entidades en LUIS. 

## <a name="list-entity"></a>Entidad de lista

Una entidad de lista representa un conjunto fijo y cerrado de palabras relacionadas y sus sinónimos. Puede usar entidades de lista para reconocer varios sinónimos o variaciones, y extraer una salida normalizada para ellas. Use la opción *Recommend* (Recomendar) para ver sugerencias de palabras nuevas en función de la lista actual. 

Una entidad de lista no es de aprendizaje automático, lo que significa que LUIS no detecta valores adicionales para las entidades de lista. LUIS marca todas las coincidencias de un elemento de cualquier lista como una entidad en la respuesta.

La coincidencia en las entidades de lista distingue mayúsculas de minúsculas y debe ser una coincidencia exacta para que se extraiga. Los valores normalizados también se usan cuando coinciden con la entidad de lista. Por ejemplo:

|Valor normalizado|Sinónimos|
|--|--|
|Pequeña|P, pq, mini, mínima|
|Media|M, md, normal, mediana|
|grande|G, gr, maxi|

Consulte el [artículo de referencia de entidades de lista](reference-entity-list.md) para más información.

## <a name="regex-entity"></a>Entidad regex

Una entidad de expresión regular extrae una entidad basada en un patrón de expresión regular que se proporciona. No distingue entre mayúsculas y minúsculas e ignora la variante cultural. La expresión regular es mejor para textos estructurados o para una secuencia predefinida de valores alfanuméricos que se esperan en un determinado formato. Por ejemplo:

|Entidad|Expresión regular|Ejemplo|
|--|--|--|
|Número del vuelo|vuelo [A-Z]{2} [0-9]{4}| vuelo AS 1234|
|Número de la tarjeta de crédito|[0-9]{16}|5478789865437632|

Consulte el [artículo de referencia de entidades de expresión regular](reference-entity-regular-expression.md) para más información.

## <a name="prebuilt-entity"></a>Entidad creada previamente

LUIS ofrece un conjunto de entidades pregeneradas para reconocer tipos comunes de datos como nombre, fecha, número y moneda.  El comportamiento de las entidades pregeneradas es fijo. La compatibilidad con entidades pregeneradas varía según la referencia cultural de la aplicación de LUIS. Por ejemplo:

|Entidad creada previamente|Valor de ejemplo|
|--|--|
|PersonName|James, Bill, Tom|
|DatetimeV2|02-05-2019, 2 de mayo, a las 8 a.m del 2 de mayo de 2019|

Consulte el [artículo de referencia de entidades pregeneradas](./luis-reference-prebuilt-entities.md) para más información.

## <a name="patternany-entity"></a>Entidad Pattern.Any

Una entidad Pattern.Any es un marcador de posición de longitud variable que solo se usa en la expresión de plantilla de un patrón para marcar dónde empieza y acaba la entidad. Sigue una regla o un patrón específicos y se usa mejor para las oraciones con estructura léxica fija. Por ejemplo:

|Expresión de ejemplo|Modelo|Entidad|
|--|--|--|
|¿Puedo tomar una hamburguesa por favor?|¿Puedo tomar una {comida} [por favor][?]| hamburguesa
|¿Puedo tomar una pizza?|¿Puedo tomar una {comida} [por favor][?]| pizza
|¿Dónde puedo encontrar El gran Gatsby?|¿Dónde puedo encontrar {nombreDelLibro}?| El gran Gatsby|

Consulte el [artículo de referencia de las entidades Pattern.Any](./reference-entity-pattern-any.md) para más información.

## <a name="machine-learned-ml-entity"></a>Entidad con aprendizaje automático (ML)

Las entidades con aprendizaje automático usan el contexto para extraer entidades basadas en ejemplos etiquetados. Son las entidades preferidas para crear aplicaciones de LUIS. Se basan en algoritmos de aprendizaje automático y requieren que el etiquetado se adapte correctamente a la aplicación. Use una entidad con aprendizaje automático para identificar datos que no siempre tienen un formato correcto, pero que tienen el mismo significado. 

|Expresión de ejemplo|Entidad de *producto* extraída|
|--|--|
|Quiero comprar un libro.|"libro"|
|¿Puedo comprar estos zapatos por favor?|"zapatos"|
|Agregue esos pantalones a mi cesta.|"pantalones"|

Puede obtener más información sobre las entidades con aprendizaje automático [aquí](./reference-entity-machine-learned-entity.md).

Consulte el [artículo de referencia sobre entidades con aprendizaje automático](./reference-entity-pattern-any.md) para más información.

#### <a name="ml-entity-with-structure"></a>Entidad de aprendizaje automático con estructura

Una entidad con aprendizaje automático se puede componer de subentidades más pequeñas, cada una de las cuales puede tener sus propias propiedades. Por ejemplo, *Dirección* podría tener la siguiente estructura:

* Dirección: 4567 Main Street, NY, 98052, USA
    * Número de edificio: 4567
    * Nombre de la calle: Main Street
    * Estado: NY
    * Código postal: 98052
    * País: USA


## <a name="building-effective-ml-entities"></a>Creación eficaz de entidades con aprendizaje automático

Para crear entidades con aprendizaje automático de forma eficaz, siga estos procedimientos recomendados:

* Si tiene una entidad de aprendizaje automático con subentidades, asegúrese de que los distintos órdenes y variantes de la entidad y las subentidades se presenten en las expresiones etiquetadas. Las expresiones etiquetadas de ejemplo deben incluir todas las formas válidas e incluir entidades que aparecen y que están ausentes, y que también se reordenan dentro de la expresión.

* Evite sobreajustar las entidades a un conjunto muy fijo. El sobreajuste tiene lugar cuando el modelo no está bien generalizado, y es un problema común en los modelos de Machine Learning. Esto implica que la aplicación no funcionaría correctamente con tipos de ejemplos nuevos. A su vez, debe variar las expresiones etiquetadas de ejemplo para que la aplicación pueda generalizar más allá de los ejemplos limitados que proporcione.

* El etiquetado debe ser coherente en todas las intenciones. Esto incluye también las expresiones que proporcione en la intención *None* que incluyan esta entidad. De lo contrario, el modelo no podrá determinar las secuencias de forma eficaz.

## <a name="entities-as-features"></a>Entidades como características

Otra función importante de las entidades es usarlas como características o rasgos distintivos para otras intenciones o entidades para que el sistema las observe y aprenda a través de ellas.

## <a name="entities-as-features-for-intents"></a>Entidades como características para intenciones

Puede usar entidades como señal para una intención. Por ejemplo, la presencia de una entidad determinada en la expresión puede distinguir en qué intención se encuentra.

|Expresión de ejemplo|Entidad|Intención|
|--|--|--|
|Resérveme un *vuelo a Nueva York*.|City (Ciudad)|Reservar vuelo|
|Resérveme la *sala de conferencias principal*.|Sala|Reservar sala|

## <a name="entities-as-feature-for-entities"></a>Entidades como característica para entidades

También puede usar entidades como indicador de la presencia de otras entidades. Un ejemplo habitual es el uso de una entidad pregenerada como característica para otra entidad con aprendizaje automático.
Si va a crear un sistema de reserva de vuelos y su expresión se parece a "Resérveme un vuelo del Cairo a Seattle", tendrá *Ciudad de origen* y *Ciudad de destino* como entidades con aprendizaje automático. Un procedimiento recomendado sería usar la entidad pregenerada `GeographyV2` como característica para ambas entidades.

Consulte el [artículo de referencia sobre las entidades GeographyV2](./luis-reference-prebuilt-geographyv2.md) para más información.

También puede usar entidades como características necesarias para otras entidades. Esto ayuda a la resolución de las entidades extraídas. Por ejemplo, si va a crear una aplicación de pedidos de pizzas y tiene una entidad `Size` con aprendizaje automático, puede crear una entidad de lista `SizeList` y usarla como una característica necesaria para la entidad `Size`. La aplicación devolverá el valor normalizado como la entidad extraída de la expresión. 

Consulte [características](luis-concept-feature.md) para más información y [entidades pregeneradas](./luis-reference-prebuilt-entities.md) para obtener más información sobre la resolución de las entidades pregeneradas disponibles en su referencia cultural. 


## <a name="entity-prediction-status-and-errors"></a>Estado de predicción de entidad y errores

En el portal de LUIS aparece lo siguiente cuando la entidad tiene una predicción de entidad diferente a la que se ha etiquetado para una expresión de ejemplo. Esta puntuación diferente se basa en el modelo entrenado actual. 

:::image type="content" source="./media/luis-concept-entities/portal-entity-prediction-error.png" alt-text="En el portal de LUIS se muestra cuando la entidad tiene una predicción de entidad diferente a la que se ha seleccionado para una expresión de ejemplo":::.

En la expresión de ejemplo, el texto que ha provocado el error está resaltado, y la línea de la expresión de ejemplo tiene un indicador de error a la derecha, que se muestra como un triángulo rojo. 

Para resolver errores de entidad, pruebe uno o varios de los siguientes pasos:

* El texto resaltado tiene una etiqueta incorrecta. Para solucionarlo, revise la etiqueta, corríjala y vuelva a entrenar la aplicación. 
* Creación de una [característica](luis-concept-feature.md) para la entidad a fin de ayudar a identificar el concepto de la entidad.
* Adición de [expresiones de ejemplo](luis-concept-utterance.md) y etiquetado con la entidad.
* [Revisión de las sugerencias de aprendizaje activas](luis-concept-review-endpoint-utterances.md) para cualquier expresión recibida en el punto de conexión de predicción que pueda ayudar a identificar el concepto de la entidad.


## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [expresiones](luis-concept-utterance.md) de ejemplo correctas.
* Vea [Add entities](luis-how-to-add-entities.md) (Agregar entidades) para obtener más información sobre cómo agregar entidades a la aplicación de LUIS.
* Obtenga más información sobre los [límites de aplicación](./luis-limits.md) de LUIS. 
* Use un [tutorial](tutorial-machine-learned-entity.md) para aprender a extraer datos estructurados de una expresión mediante la entidad con aprendizaje automático.
