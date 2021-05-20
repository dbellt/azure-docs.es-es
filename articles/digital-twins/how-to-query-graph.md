---
title: Consulta del grafo de gemelos
titleSuffix: Azure Digital Twins
description: Vea cómo consultar el grafo gemelo de Azure Digital Twins para información.
author: baanders
ms.author: baanders
ms.date: 11/19/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: 67d09eb3420143c362914a7903b181a419d7a388
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786886"
---
# <a name="query-the-azure-digital-twins-twin-graph"></a>Consulta del grafo gemelo de Azure Digital Twins

En este artículo se ofrecen ejemplos de consultas e instrucciones sobre el uso del **lenguaje de consultas de Azure Digital Twins** para consultar un [grafo de gemelos](concepts-twins-graph.md) para obtener información. (Para obtener una introducción al lenguaje de consulta, consulte [Conceptos: lenguaje de consulta](concepts-query-language.md)).

Contiene consultas de ejemplo que muestran la estructura del lenguaje de consulta y las operaciones de consulta habituales para los gemelos digitales. También describe cómo ejecutar las consultas una vez escritas, mediante [Query API](/rest/api/digital-twins/dataplane/query) de Azure Digital Twins o un [SDK](concepts-apis-sdks.md#overview-data-plane-apis).

> [!NOTE]
> Si ejecuta las consultas de ejemplo siguientes con la llamada a una API o un SDK, deberá condensar el texto de la consulta en una sola línea.

[!INCLUDE [digital-twins-query-reference.md](../../includes/digital-twins-query-reference.md)]

## <a name="show-all-digital-twins"></a>Mostrar todos los gemelos digitales

Esta es la consulta básica que devolverá una lista de todos los gemelos digitales de la instancia:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="GetAllTwins":::

## <a name="query-by-property"></a>Consulta por propiedad

Obtenga instancias de Digital Twins por **propiedades** (incluidos el identificador y los metadatos):

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByProperty1":::

Como se muestra en la consulta anterior, el identificador de un gemelo digital se consulta mediante el campo de metadatos `$dtId`.

>[!TIP]
> Cuando se usa Cloud Shell para ejecutar una consulta con campos de metadatos que comienzan por `$`, se debe usar un acento grave como carácter de escape para que Cloud Shell sepa que no es una variable y se debe consumir como literal en el texto de la consulta.

También puede obtener instancias de Digital Twins en función de **si una propiedad determinada está definida**. Esta es una consulta que obtiene instancias de Digital Twins que tienen una propiedad *Location* definida:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByProperty2":::

Esto le puede ayudar a obtener instancias de Digital Twins por sus propiedades de *etiqueta*, como se describe en [Incorporación de etiquetas a gemelos digitales](how-to-use-tags.md). Esta es una consulta que obtiene todas las instancias de Digital Twins etiquetadas con *red*:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryMarkerTags1":::

También puede obtener instancias de Digital Twins en función del **tipo de una propiedad**. Esta es una consulta que obtiene instancias de Digital Twins cuya propiedad *Temperature* es un número:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByProperty3":::

>[!TIP]
> Si una propiedad es de tipo `Map`, puede usar las claves y los valores de mapa directamente en la consulta, de la siguiente manera:
> :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByProperty4":::

## <a name="query-by-model"></a>Consulta por modelo

El operador `IS_OF_MODEL` se puede utilizar para filtrar en función del [modelo](concepts-models.md) del gemelo.

Tiene en cuenta la [herencia](concepts-models.md#model-inheritance) y el [control de versiones](how-to-manage-model.md#update-models) del modelo, y se evalúa como **true** para un gemelo dado si este cumple cualquiera de estas condiciones:

* El gemelo implementa directamente el modelo proporcionado a `IS_OF_MODEL()` y el número de versión del modelo en el gemelo es *mayor o igual que* el número de versión del modelo proporcionado.
* El gemelo implementa un modelo que *amplía* el modelo proporcionado a `IS_OF_MODEL()` y el número de versión del modelo extendido del gemelo es *mayor o igual que* el número de versión del modelo proporcionado.

Por ejemplo, si consulta los gemelos del modelo `dtmi:example:widget;4`, la consulta devolverá todos los gemelos basados en la **versión 4 o superior** del modelo de **widget**, y también los gemelos basados en la versión **4 o superior** de los **modelos que heredan del widget**.

`IS_OF_MODEL` puede aceptar varios parámetros diferentes, y el resto de esta sección se dedica a sus distintas opciones de sobrecarga.

El uso más simple de `IS_OF_MODEL` solo toma un parámetro `twinTypeName`: `IS_OF_MODEL(twinTypeName)`.
A continuación, se muestra un ejemplo de consulta que pasa un valor en este parámetro:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByModel1":::

Para especificar una colección gemela para buscar cuando hay más de uno (como cuando se usa `JOIN`), agregue el parámetro `twinCollection`: `IS_OF_MODEL(twinCollection, twinTypeName)`.
A continuación, se muestra un ejemplo de consulta que agrega un valor en este parámetro:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByModel2":::

Para realizar una coincidencia exacta, agregue el parámetro `exact`: `IS_OF_MODEL(twinTypeName, exact)`.
A continuación, se muestra un ejemplo de consulta que agrega un valor en este parámetro:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByModel3":::

También puede pasar los tres argumentos juntos: `IS_OF_MODEL(twinCollection, twinTypeName, exact)`.
A continuación, se muestra un ejemplo de consulta que especifica un valor para los tres parámetros:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByModel4":::

## <a name="query-by-relationship"></a>Consulta por relación

Al realizar consultas basadas en **relaciones** de Digital Twins, el lenguaje de consultas de Azure Digital Twins tiene una sintaxis especial.

Las relaciones se extraen en el ámbito de la consulta en la cláusula `FROM`. A diferencia de los lenguajes de tipo SQL "clásico", cada expresión de esta cláusula `FROM` no es una tabla; en su lugar, la cláusula `FROM` expresa un recorrido de relación entre entidades. Para recorrer las relaciones, Azure Digital Twins usa una versión personalizada de `JOIN`.

Recuerde que, con las funcionalidades del [modelo](concepts-models.md) de Azure Digital Twins, las relaciones no existen de forma independiente de los gemelos. Esto implica que las relaciones no se pueden consultar de forma independiente y deben estar vinculadas a un gemelo.
Para controlar esto, la palabra clave `RELATED` se usa en la cláusula `JOIN` para extraer el conjunto de un tipo determinado de relación procedente de la colección de gemelos. A continuación, la consulta debe filtrar en la cláusula `WHERE` qué gemelos específicos se usarán en la consulta de relación (mediante los valores `$dtId` de los gemelos).

En las siguientes secciones se proporcionan ejemplos de su aspecto.

### <a name="basic-relationship-query"></a>Consulta de relación básica

A continuación se muestra un ejemplo de consulta basada en relaciones. Este fragmento de código selecciona todos los gemelos digitales con una propiedad *ID* "ABC" y todos los gemelos digitales relacionados con estos gemelos digitales a través de una relación *contains*.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByRelationship1":::

> [!NOTE]
> El desarrollador no necesita poner en correlación esta operación `JOIN` con un valor de clave en la cláusula `WHERE` (ni especificar un valor de clave insertado con la definición de `JOIN`). El sistema calcula esta correlación automáticamente, ya que las propias propiedades de la relación identifican la entidad de destino.

### <a name="query-by-the-source-or-target-of-a-relationship"></a>Consulta por el origen o destino de una relación

Puede usar la estructura de consulta de la relación para identificar un gemelo digital que sea el origen o destino de una relación.

Por ejemplo, puede empezar con un gemelo de origen y seguir sus relaciones para buscar los gemelos de destino de las relaciones. Este es un ejemplo de una consulta que busca los gemelos de destino de las relaciones de *fuentes* que proceden del gemelo source-twin.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByRelationshipSource":::

También puede empezar con el destino de la relación y realizar un seguimiento de la relación para encontrar al gemelo de origen. Este es un ejemplo de una consulta que busca los gemelos de origen de las relaciones de *fuentes* con el gemelo target-twin.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByRelationshipTarget":::

### <a name="query-the-properties-of-a-relationship"></a>Consulta de las propiedades de una relación

Del mismo modo que los gemelos digitales tienen propiedades que se describen a través de DTDL, las relaciones también pueden tener propiedades. Puede consultar instancias de Digital Twins **en función de las propiedades de sus relaciones**.
El lenguaje de consultas de Azure Digital Twins permite filtrar y proyectar relaciones mediante la asignación de un alias a la relación dentro de la cláusula `JOIN`.

Como ejemplo, considere una relación *servicedBy* que tiene una propiedad *reportedCondition*. En la consulta siguiente, a esta relación se le asigna el alias "R" para hacer referencia a su propiedad.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByRelationship2":::

En el ejemplo anterior, observe cómo *reportedCondition* es una propiedad de la propia relación *servicedBy* (NO de un gemelo digital que tiene una relación *servicedBy*).

### <a name="query-with-multiple-joins"></a>Consulta con varias combinaciones JOIN

En una sola consulta se admiten hasta cinco elementos `JOIN`. Esto le permite atravesar varios niveles de relaciones a la vez. 

Para consultar varios niveles de relaciones, use una única instrucción `FROM` seguida de N instrucciones `JOIN`, donde las instrucciones `JOIN` expresan relaciones en el resultado de una instrucción `FROM` o `JOIN` anterior.

Este es un ejemplo de una consulta de varias combinaciones, que obtiene todas las bombillas contenidas en los paneles de luz de las salas 1 y 2.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="QueryByRelationship3":::

## <a name="count-items"></a>Recuento de elementos

Puede contar el número de elementos de un conjunto de resultados mediante la cláusula `Select COUNT`:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="SelectCount1":::

Agregue una cláusula `WHERE` para contar el número de elementos que cumplen determinados criterios. Estos son algunos ejemplos de recuento con un filtro aplicado basado en el tipo de modelo gemelo (para obtener más información sobre esta sintaxis, vea [Consulta por modelo](#query-by-model) a continuación):

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="SelectCount2":::

También puede usar `COUNT` junto con la cláusula `JOIN`. Esta es una consulta que cuenta todas las bombillas incluidas en los paneles de luz de los salones 1 y 2:

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="SelectCount3":::

## <a name="filter-results-select-top-items"></a>Resultados del filtro: seleccionar elementos principales

Puede seleccionar los diversos elementos "principales" en una consulta mediante la cláusula `Select TOP`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="SelectTop":::

## <a name="filter-results-specify-return-set-with-projections"></a>Resultados del filtro: especificación del conjunto de devoluciones con proyecciones

El uso de las proyecciones en la instrucción `SELECT` permite elegir las columnas que devolverá una consulta.

>[!NOTE]
>En este momento, no se admiten las propiedades complejas. Para asegurarse de que las propiedades de proyección son válidas, combine las proyecciones con una comprobación de `IS_PRIMITIVE`.

Este es un ejemplo de una consulta que usa una proyección para devolver gemelos y relaciones. La siguiente consulta proyecta los valores de consumidor, fábrica y borde de un escenario en el que una fábrica con el identificador *ABC* está relacionada con el consumidor mediante una relación *Factory.customer*, y esa relación se presenta como el *Borde*.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="Projections1":::

También puede usar la proyección para devolver una propiedad de un gemelo. La siguiente consulta proyecta la propiedad *Nombre* de los Consumidores que están relacionados con la Fábrica con un identificador de *ABC* mediante una relación de *Factory.customer*.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="Projections2":::

También puede usar la proyección para devolver una propiedad de una relación. Al igual que en el ejemplo anterior, la siguiente consulta proyecta la propiedad *Nombre* de los Consumidores relacionados con la Fábrica con un identificador de *ABC* mediante una relación de *Factory.customer*; pero ahora también devuelve dos propiedades de esa relación, *Prop1* y *prop2*. Para ello, se asigna un nombre a la relación *Perimetral* y se recopilan sus propiedades.  

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="Projections3":::

También puede utilizar alias para simplificar las consultas con proyección.

La siguiente consulta realiza las mismas operaciones que el ejemplo anterior, pero asigna a los nombres de propiedades los alias `consumerName`, `first`, `second` y `factoryArea`.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="Projections4":::

Esta es una consulta similar que consulta el mismo conjunto que en el caso anterior, pero solo proyecta la propiedad *Consumer.name* como `consumerName` y proyecta la Fábrica completa como gemela.

:::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="Projections5":::

## <a name="build-efficient-queries-with-the-in-operator"></a>Creación de consultas eficientes con el operador IN

Puede reducir significativamente el número de consultas que necesita si crea una matriz de gemelos y consulta con el operador `IN`. 

Por ejemplo, considere un escenario en el que Buildings contenga a Floors y Floors contenga a Rooms. Una forma de buscar las habitaciones que estén activas dentro de un edificio, es seguir estos pasos.

1. Busque los pisos en el edificio con la relación `contains`.

    :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="INOperatorWithout":::

2. Para buscar las habitaciones, en lugar de tener que considerar los pisos uno por uno y ejecutar una consulta `JOIN` para buscar las habitaciones de cada uno, puede realizar consultas con una colección de los pisos del edificio (con el nombre Floor en la consulta siguiente).

    En la aplicación cliente:
    
    ```csharp
    var floors = "['floor1','floor2', ..'floorn']"; 
    ```
    
    En la consulta:
    
    :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="INOperatorWith":::

## <a name="other-compound-query-examples"></a>Otros ejemplos de consultas compuestas

Puede **combinar** cualquiera de los tipos de consulta anteriores mediante operadores de combinación con el fin de incluir más detalles en una sola consulta. A continuación se muestran algunos ejemplos adicionales de consultas compuestas que realizan consultas para más de un tipo de descriptor de instancia de Digital Twins a la vez.

* De entre los dispositivos que tiene Room 123, se devuelven los dispositivos MxChip que tienen el rol de operador.
    :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="OtherExamples1":::
* Se obtienen las instancias de Digital Twins que tienen una relación denominada *Contains* con otra instancia que tiene un identificador *id1*
    :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="OtherExamples2":::
* Se obtienen todas las salas de este modelo de sala contenidos en floor11
    :::code language="sql" source="~/digital-twins-docs-samples/queries/examples.sql" id="OtherExamples3":::

## <a name="run-queries-with-the-api"></a>Ejecución de consultas con la API

Una vez que haya decidido una cadena de consulta, puede ejecutarla realizando una llamada a la [API de consulta](/rest/api/digital-twins/dataplane/query).

Puede llamar a la API directamente, o bien usar uno de los [SDK](concepts-apis-sdks.md#overview-data-plane-apis) disponibles para Azure Digital Twins.

En el fragmento de código siguiente se muestra la llamada al [SDK de .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client) desde una aplicación cliente:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/queries.cs" id="RunQuery":::

La consulta utilizada en esta llamada devuelve una lista de gemelos digitales, que el ejemplo anterior representa con objetos [BasicDigitalTwin](/dotnet/api/azure.digitaltwins.core.basicdigitaltwin). El tipo de valor devuelto de los datos para cada consulta dependerá de los términos que especifique con la instrucción `SELECT`:
* Las consultas que comienzan por `SELECT * FROM ...` devolverán una lista de gemelos digitales (que se pueden serializar como objetos `BasicDigitalTwin` u otros tipos de gemelos digitales personalizados que haya creado).
* Las consultas que comienzan con el formato `SELECT <A>, <B>, <C> FROM ...` devolverán un diccionario con las claves `<A>`, `<B>` y `<C>`.
* Se pueden diseñar otros formatos de instrucciones `SELECT` para devolver datos personalizados. Considere la posibilidad de crear sus propias clases para administrar conjuntos de resultados muy personalizados. 

### <a name="query-with-paging"></a>Consulta con paginación

Las llamadas de consulta admiten la paginación. Este es un ejemplo completo donde se usa `BasicDigitalTwin` como tipo de resultado de la consulta con control de errores y paginación:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/queries.cs" id="FullQuerySample":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las [API y los SDK de Azure Digital Twins](concepts-apis-sdks.md), incluida Query API, que se usa para ejecutar las consultas de este artículo.
