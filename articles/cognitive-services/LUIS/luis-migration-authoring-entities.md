---
title: Migración a una entidad de aprendizaje automático de V3
description: La creación de V3 proporciona un nuevo tipo de entidad, la entidad de aprendizaje automático, junto con la capacidad de agregar relaciones a la entidad de aprendizaje automático y a otras entidades o características de la aplicación.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/28/2021
ms.openlocfilehash: 6808c5d84a1be6a2e9b262eab00010d4a962c259
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701158"
---
# <a name="migrate-to-v3-authoring-entity"></a>Migración a la creación de V3

La creación de V3 proporciona un nuevo tipo de entidad, la entidad de aprendizaje automático, junto con la capacidad de agregar relaciones a la entidad de aprendizaje automático y a otras entidades o características de la aplicación. Actualmente no hay ninguna fecha antes de la que sea necesario completar la migración.

## <a name="entities-are-decomposable-in-v3"></a>Las entidades se pueden descomponer en V3

Las entidades creadas con las API de creación de V3, ya sea mediante las [API](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview) o con el portal, le permiten crear un modelo de entidad con capas con un elemento principal y elementos secundarios. Al elemento primario se le conoce como **entidad de aprendizaje automático** y a los secundarios se les conoce como **subentidades** de la entidad de aprendizaje automático.

Cada subentidad es también una entidad de aprendizaje automático, pero con las opciones de configuración agregadas de las características.

* Las **características necesarias** son reglas que garantizan que una entidad se extraiga cuando coincida con una característica. La regla se define mediante la característica necesaria para el modelo:
    * [Entidad precompilada](luis-reference-prebuilt-entities.md)
    * [Entidad de expresión regular](reference-entity-regular-expression.md)
    * [Entidad de lista](reference-entity-list.md)

## <a name="how-do-these-new-relationships-compare-to-v2-authoring"></a>Comparación de las nuevas relaciones con la creación de V2

La creación de V2 proporciona entidades jerárquicas y compuestas junto con roles y características para realizar esta misma tarea. Dado que las entidades, las características y los roles no se relacionaban explícitamente entre sí, era difícil entender cómo LUIS deducía las relaciones durante la predicción.

Con V3, la relación es explícita y está diseñada por los autores de la aplicación. Esto le permite, como autor de la aplicación:

* Ver visualmente cómo LUIS está prediciendo estas relaciones en las expresiones de ejemplo
* Probar estas relaciones con el [panel de prueba interactiva](luis-interactive-test.md) o en el punto de conexión
* Usar estas relaciones en la aplicación cliente, a través de un [objeto .json](reference-entity-machine-learned-entity.md) anidado bien estructurado y con nombre

## <a name="planning"></a>Planificación

Cuando migre, tenga en cuenta lo siguiente en el plan de migración:

* Realice una copia de seguridad de la aplicación de LUIS y realice la migración en una aplicación independiente. Tener una aplicación de V2 y V3 disponible al mismo tiempo le permite validar los cambios necesarios y el impacto en los resultados de la predicción.
* Capture las métricas de éxito de predicción actuales
* Capture la información del panel actual como una instantánea del estado de la aplicación
* Revise las intenciones, entidades, listas de frases, patrones y pruebas por lotes existentes
* Los siguientes elementos se pueden migrar **sin necesidad de cambios**:
    * Intenciones
    * Entidades
        * Entidad de expresión regular
        * Entidad de lista
    * Características
        * Lista de frases
* Los siguientes elementos se deben migrar **con cambios**:
    * Entidades
        * Entidad jerárquica
        * Entidad compuesta
    * Roles: los roles solo se pueden aplicar a una entidad de aprendizaje automático (primaria). Los roles no se pueden aplicar a las subentidades.
    * Pruebas por lotes y patrones que usan las entidades jerárquicas y compuestas

Al diseñar el plan de migración, deje tiempo para revisar las entidades finales de aprendizaje automático, después de que se hayan migrado todas las entidades jerárquicas y compuestas. Aunque una migración directa funcionará, después de realizar el cambio y revisar los resultados de las pruebas por lotes, y el JSON de predicción, el JSON más unificado puede hacer que realice cambios para que la información final que se entrega a la aplicación del lado del cliente se organice de manera diferente. Esto es similar a la refactorización de código y debe tratarse con el mismo proceso de revisión que tiene su organización.

Si no tiene pruebas por lotes para su modelo de V2 y migra las pruebas por lotes al modelo de V3 como parte de la migración, no podrá validar cómo afectará la migración a los resultados de predicción de puntos de conexión.

## <a name="migrating-from-v2-entities"></a>Migración desde entidades de V2

Cuando se empiece a migrar al modelo de creación de V3, se debe tener en cuenta cómo migrar a la entidad de aprendizaje automático, así como sus subentidades y características.

En la tabla siguiente se indican las entidades que se deben migrar de un diseño de entidad de V2 a V3.

|Tipo de entidad de creación de V2|Tipo de entidad de creación de V3|Ejemplo|
|--|--|--|
|Entidad compuesta|Entidad de aprendizaje automático|[más información](#migrate-v2-composite-entity)|
|Entidad jerárquica|rol de la entidad de aprendizaje automático|[más información](#migrate-v2-hierarchical-entity)|

## <a name="migrate-v2-composite-entity"></a>Migración de una entidad compuesta de V2

Cada elemento secundario de la entidad compuesta de V2 se debe representar con un subentidad de la entidad de aprendizaje automático de V3. Si el elemento secundario compuesto es una expresión regular o una entidad de lista generada previamente, debe aplicarse como característica necesaria en la subentidad.

Consideraciones a la hora de planear la migración de una entidad compuesta a una entidad de aprendizaje automático:
* Las entidades secundarias no se pueden usar en patrones
* Las entidades secundarias ya no se comparten
* Las entidades secundarias se deben etiquetar si no eran de aprendizaje automático

### <a name="existing-features"></a>Características existentes

Cualquier lista de frases utilizada para potenciar las palabras en la entidad compuesta debe aplicarse como característica a la entidad de aprendizaje automático (primaria), la subentidad (secundaria) o la intención (si la lista de frases solo se aplica a una intención). Planee agregar la característica a la entidad que debe potenciar de forma significativa. No agregue la característica genéricamente a la entidad de aprendizaje automático (primaria), en caso de que potencie de forma significativa la predicción de una subentidad (secundaria).

### <a name="new-features"></a>Nuevas características

En la creación de V3, agregue un paso de planificación para evaluar las entidades como posibles características para todas las entidades e intenciones.

### <a name="example-entity"></a>Entidad de ejemplo

Esta entidad es solo un ejemplo. Su propia migración de entidades puede requerir otras consideraciones.

Considere una entidad compuesta de V2 para modificar un `order` de pizza que usa:
* entidad datetimeV2 creada previamente para la hora de entrega
* lista de frases para potenciar ciertas palabras, como pizza, tarta, corteza e ingrediente
* lista de entidades para detectar los ingredientes, como setas, aceitunas, pepperoni.

Un ejemplo de expresión para esta entidad es:

`Change the toppings on my pie to mushrooms and delivery it 30 minutes later`

En la tabla siguiente se muestra la migración:

|Modelos de V2|Modelos de V3|
|--|--|
|Principal: entidad de componente llamada `Order`|Primaria: entidad de aprendizaje automático llamada `Order`|
|Secundario: entidad datetimeV2 creada previamente|* Migre la entidad creada previamente a la nueva aplicación.<br>* Agregue una característica necesaria en el elemento principal para la entidad datetimeV2 creada previamente.|
|Secundario: entidad de lista para los ingredientes|* Migre la entidad de lista a la nueva aplicación.<br>* A continuación, agregue una característica necesaria en el elemento principal para la entidad de lista.|


## <a name="migrate-v2-hierarchical-entity"></a>Migración de una entidad jerárquica de V2

En la creación de V2, se proporcionaba una entidad jerárquica antes de los roles existentes en LUIS. Ambos tenían el mismo propósito de extraer entidades según el uso del contexto. Si tiene entidades jerárquicas, puede considerarlas como entidades simples con roles.

En la creación de V3:
* Un rol se puede aplicar a la entidad de aprendizaje automático (primaria).
* Los roles no se pueden aplicar a las subentidades.

Esta entidad es solo un ejemplo. Su propia migración de entidades puede requerir otras consideraciones.

Considere una entidad jerárquica de V2 para modificar un `order` de pizza:
* donde cada elemento secundario determina un ingrediente original o el ingrediente final

Un ejemplo de expresión para esta entidad es:

`Change the topping from mushrooms to olives`

En la tabla siguiente se muestra la migración:

|Modelos de V2|Modelos de V3|
|--|--|
|Principal: entidad de componente llamada `Order`|Primaria: entidad de aprendizaje automático llamada `Order`|
|Secundario: entidad jerárquica con el ingrediente de la pizza original y final|* Agregue un rol a `Order` para cada ingrediente.|

## <a name="api-change-constraint-replaced-with-required-feature"></a>Restricción de cambios de API reemplazada por característica necesaria

Este cambio se realizó en mayo de 2020, en la conferencia de //Build y solo se aplica a las API de creación de v3 en las que una aplicación usa una característica restringida. Si va a migrar desde la creación de v2 a la creación de v3 o no ha usado características restringidas de v3, omita esta sección.

**Funcionalidad**: Capacidad de exigir una entidad existente como característica de otro modelo y extraer solo ese modelo si se detecta la entidad. La funcionalidad no ha cambiado, pero la API y la terminología sí.

|Terminología anterior|Nueva terminología|
|--|--|
|`constrained feature`<br>`constraint`<br>`instanceOf`|`required feature`<br>`isRequired`|

#### <a name="automatic-migration"></a>Migración automática

A **partir del 19 de junio de 2020**, no se le permitirá crear restricciones mediante programación con la API de creación anterior que exponía esta funcionalidad.

Todas las características de restricción existentes se migrarán automáticamente a la marca de características necesaria. No se requieren cambios mediante programación en la API de predicción ni ningún cambio resultante en la calidad de la precisión de predicción.

#### <a name="luis-portal-changes"></a>Cambios en el portal de LUIS

El portal de versión preliminar de LUIS hacía referencia a esta funcionalidad como **restricción**. El portal de LUIS actual nombra esta funcionalidad como **característica necesaria**.

#### <a name="previous-authoring-api"></a>API de creación anterior

Esta funcionalidad se aplicó en la versión preliminar de creación **[API Create Entity Child](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5d86cf3c6a25a45529767d77)** como parte de la definición de una entidad, mediante la propiedad `instanceOf` del elemento secundario de una entidad:

```json
{
    "name" : "dayOfWeek",
    "instanceOf": "datetimeV2",
    "children": [
        {
           "name": "dayNumber",
           "instanceOf": "number",
           "children": []
        }
    ]
}
```

#### <a name="new-authoring-api"></a>Nueva API de creación

Esta funcionalidad ahora se aplica con la **[API Add entity feature relation](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5d9dc1781e38aaec1c375f26)** mediante las propiedades `featureName` y `isRequired`. El valor de la propiedad `featureName` es el nombre del modelo.

```json
{
    "featureName": "YOUR-MODEL-NAME-HERE",
    "isRequired" : true
}
```


## <a name="next-steps"></a>Pasos siguientes

* [Recursos para desarrolladores](developer-reference-resource.md)
