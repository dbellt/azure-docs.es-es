---
title: Filtrado de eventos para Azure Event Grid
description: Se describe cómo filtrar eventos al crear una suscripción de Azure Event Grid.
ms.topic: conceptual
ms.date: 03/04/2021
ms.openlocfilehash: 986e31c1213dc05862c95d87a0bc7733f7a2f9c1
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111372149"
---
# <a name="understand-event-filtering-for-event-grid-subscriptions"></a>Descripción del filtrado de eventos para suscripciones de Event Grid

En este artículo se describen las distintas formas de filtrar los eventos que se envían al punto de conexión. Al crear una suscripción de eventos, se le presentan tres opciones de filtrado:

* Tipos de eventos
* El asunto empieza o termina con
* Operadores y campos avanzados

## <a name="event-type-filtering"></a>Filtrado de tipos de evento

De forma predeterminada, todos los [tipos de evento](event-schema.md) del origen de eventos se envían al punto de conexión. Puede decidir enviar solo determinados tipos de eventos al punto de conexión. Por ejemplo, puede recibir una notificación de las actualizaciones de sus recursos pero no de otras operaciones, como las eliminaciones. En ese caso, filtre por el tipo de evento `Microsoft.Resources.ResourceWriteSuccess`. Proporcione una matriz con los tipos de eventos, o especifique `All` para obtener todos los tipos de evento para el origen del evento.

La sintaxis JSON para el filtrado por tipo de evento es:

```json
"filter": {
  "includedEventTypes": [
    "Microsoft.Resources.ResourceWriteFailure",
    "Microsoft.Resources.ResourceWriteSuccess"
  ]
}
```

## <a name="subject-filtering"></a>Filtrado de asunto

Para filtrar solo por asunto, especifique un valor inicial o final para el asunto. Por ejemplo, puede especificar que el asunto termina con `.txt` para obtener solo los eventos relacionados con la carga de un archivo de texto a la cuenta de almacenamiento. O bien, puede filtrar que el sujeto empieza por `/blobServices/default/containers/testcontainer` para obtener todos los eventos de ese contenedor, pero no otros contenedores en la cuenta de almacenamiento.

Al publicar eventos en temas personalizados, cree asuntos para los eventos que faciliten que los suscriptores sepan si les interesa el evento. Los suscriptores usan la propiedad del asunto para filtrar y redirigir eventos. Considere la posibilidad de agregar la ruta de acceso de donde se produjo el evento, para que los suscriptores pueden filtrar por los segmentos de esa ruta de acceso. La ruta de acceso permite que los suscriptores filtren eventos de una manera más amplia o más restringida. Si proporciona una ruta de acceso de tres segmentos como `/A/B/C` en el asunto, los suscriptores pueden filtrar por el primer segmento `/A` para obtener un amplio conjunto de eventos. Esos suscriptores obtienen eventos con asuntos como `/A/B/C` o `/A/D/E`. Otros suscriptores pueden filtrar por `/A/B` para obtener un conjunto de eventos más reducido.

La sintaxis JSON para el filtrado por asunto es:

```json
"filter": {
  "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
  "subjectEndsWith": ".jpg"
}

```

## <a name="advanced-filtering"></a>Filtrado avanzado

Para filtrar por valores en los campos de datos y especificar el operador de comparación, use la opción de filtrado avanzada. En el filtrado avanzado, se especifica lo siguiente:

* tipo de operador: el tipo de comparación.
* clave: el campo de datos de evento que se usa para filtrar. Puede ser un número, un valor booleano, una cadena o una matriz.
* valores: el valor o los valores que se compararán con la clave.

## <a name="key"></a>Clave
Clave es el campo de datos de evento que se usa para filtrar. Puede ser uno de los siguientes tipos:

- Number
- Boolean
- String
- Matriz. Debe establecer la propiedad `enableAdvancedFilteringOnArrays` en true para usar esta característica. 

    ```json
    "filter":
    {
        "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
        "subjectEndsWith": ".jpg",
        "enableAdvancedFilteringOnArrays": true
    }
    ```

Para los eventos del **esquema de Event Grid**, utilice los valores siguientes para la clave: `ID`, `Topic`, `Subject`, `EventType`, `DataVersion` o datos de evento (como `data.key1`).

Para los eventos del **esquema de Cloud Events**, utilice los valores siguientes para la clave: `eventid`, `source`, `eventtype`, `eventtypeversion` o datos de evento (como `data.key1`).

Para el **esquema de entrada personalizado**, use los campos de datos de evento (como `data.key1`). Para acceder a los campos de la sección de datos, use la notación `.` (punto). Por ejemplo, `data.sitename`, `data.appEventTypeDetail.action` para acceder a `sitename` o `action` para el siguiente evento de ejemplo.

```json
    "data": {
        "appEventTypeDetail": {
            "action": "Started"
        },
        "siteName": "<site-name>",
        "clientRequestId": "None",
        "correlationRequestId": "None",
        "requestId": "292f499d-04ee-4066-994d-c2df57b99198",
        "address": "None",
        "verb": "None"
    },
```

## <a name="values"></a>Valores
Los valores pueden ser: número, cadena, booleano o matriz.

## <a name="operators"></a>Operadores

Los operadores disponibles para los **números** son:

## <a name="numberin"></a>NumberIn
El operador NumberIn se evalúa en true si el valor de la **clave** es uno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `counter` de la sección `data` es 5 o 1. 

```json
"advancedFilters": [{
    "operatorType": "NumberIn",
    "key": "data.counter",
    "values": [
        5,
        1
    ]
}]
```


Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a, b, c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF filter == key
            MATCH
```

## <a name="numbernotin"></a>NumberNotIn
El operador NumberNotIn se evalúa en true si el valor de la **clave** **no** es ninguno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `counter` de la sección `data` no es 41 y 0. 

```json
"advancedFilters": [{
    "operatorType": "NumberNotIn",
    "key": "data.counter",
    "values": [
        41,
        0
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a, b, c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF filter == key
            FAIL_MATCH
```

## <a name="numberlessthan"></a>NumberLessThan
El operador NumberLessThan se evalúa en true si el valor de la **clave** es **menor** que el valor de **filtro** especificado. En el ejemplo siguiente, comprueba si el valor del atributo `counter` de la sección `data` es menor que 100. 

```json
"advancedFilters": [{
    "operatorType": "NumberLessThan",
    "key": "data.counter",
    "value": 100
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con el valor de filtro. Este es el pseudocódigo con la clave: `[v1, v2, v3]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH key IN (v1, v2, v3)
    IF key < filter
        MATCH
```

## <a name="numbergreaterthan"></a>NumberGreaterThan
El operador NumberGreaterThan se evalúa en true si el valor de la **clave** es **mayor** que el valor de **filtro** especificado. En el ejemplo siguiente, comprueba si el valor del atributo `counter` de la sección `data` es mayor que 20. 

```json
"advancedFilters": [{
    "operatorType": "NumberGreaterThan",
    "key": "data.counter",
    "value": 20
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con el valor de filtro. Este es el pseudocódigo con la clave: `[v1, v2, v3]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH key IN (v1, v2, v3)
    IF key > filter
        MATCH
```

## <a name="numberlessthanorequals"></a>NumberLessThanOrEquals
El operador NumberLessThanOrEquals se evalúa en true si el valor de la **clave** es **menor o igual** que el valor de **filtro** especificado. En el ejemplo siguiente, comprueba si el valor del atributo `counter` de la sección `data` es menor o igual que 100. 

```json
"advancedFilters": [{
    "operatorType": "NumberLessThanOrEquals",
    "key": "data.counter",
    "value": 100
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con el valor de filtro. Este es el pseudocódigo con la clave: `[v1, v2, v3]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH key IN (v1, v2, v3)
    IF key <= filter
        MATCH
```

## <a name="numbergreaterthanorequals"></a>NumberGreaterThanOrEquals
El operador NumberGreaterThanOrEquals se evalúa en true si el valor de la **clave** es **mayor o igual** que el valor de **filtro** especificado. En el ejemplo siguiente, comprueba si el valor del atributo `counter` de la sección `data` es mayor o igual que 30. 

```json
"advancedFilters": [{
    "operatorType": "NumberGreaterThanOrEquals",
    "key": "data.counter",
    "value": 30
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con el valor de filtro. Este es el pseudocódigo con la clave: `[v1, v2, v3]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH key IN (v1, v2, v3)
    IF key >= filter
        MATCH
```

## <a name="numberinrange"></a>NumberInRange
El operador NumberInRange se evalúa en true si el valor de la **clave** es uno de los **intervalos de filtros** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` se encuentra dentro de uno de los dos intervalos: 3.14159 - 999.95, 3000 - 4000. 

```json
{
    "operatorType": "NumberInRange",
    "key": "data.key1",
    "values": [[3.14159, 999.95], [3000, 4000]]
}
```

La propiedad `values` es una matriz de intervalos. En el ejemplo anterior, es una matriz de dos intervalos. Este es un ejemplo de una matriz con un intervalo para comprobar. 

**Matriz con un intervalo:** 
```json
{
    "operatorType": "NumberInRange",
    "key": "data.key1",
    "values": [[3000, 4000]]
}
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y una matriz de intervalos como filtro. En este pseudocódigo, `a` y `b` son los valores inferior y superior de cada intervalo de la matriz. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH (a,b) IN filter.Values
    FOR_EACH key IN (v1, v2, v3)
       IF key >= a AND key <= b
           MATCH
```


## <a name="numbernotinrange"></a>NumberNotInRange
El operador NumberNotInRange se evalúa en true si el valor de la **clave** **no** se encuentra dentro de ninguno de los **intervalos de filtros** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` se encuentra dentro de uno de los dos intervalos: 3.14159 - 999.95, 3000 - 4000. Si es así, el operador devuelve false. 

```json
{
    "operatorType": "NumberNotInRange",
    "key": "data.key1",
    "values": [[3.14159, 999.95], [3000, 4000]]
}
```
La propiedad `values` es una matriz de intervalos. En el ejemplo anterior, es una matriz de dos intervalos. Este es un ejemplo de una matriz con un intervalo para comprobar.

**Matriz con un intervalo:** 
```json
{
    "operatorType": "NumberNotInRange",
    "key": "data.key1",
    "values": [[3000, 4000]]
}
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y una matriz de intervalos como filtro. En este pseudocódigo, `a` y `b` son los valores inferior y superior de cada intervalo de la matriz. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH (a,b) IN filter.Values
    FOR_EACH key IN (v1, v2, v3)
        IF key >= a AND key <= b
            FAIL_MATCH
```


El operador disponible para los **booleanos** es: 

## <a name="boolequals"></a>BoolEquals
El operador BoolEquals se evalúa en true si el valor de la **clave** es el **filtro** del valor booleano especificado. En el ejemplo siguiente, comprueba si el valor del atributo `isEnabled` de la sección `data` es `true`. 

```json
"advancedFilters": [{
    "operatorType": "BoolEquals",
    "key": "data.isEnabled",
    "value": true
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con el valor booleano del filtro. Este es el pseudocódigo con la clave: `[v1, v2, v3]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH key IN (v1, v2, v3)
    IF filter == key
        MATCH
```

Los operadores disponibles para las **cadenas** son:

## <a name="stringcontains"></a>StringContains
El operador **StringContains** se evalúa en true si el valor de la **clave** **contiene** cualquiera de los valores de **filtro** especificados (como subcadenas). En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` contiene una de las subcadenas especificadas: `microsoft` o `azure`. Por ejemplo, `azure data factory` contiene `azure`. 

```json
"advancedFilters": [{
    "operatorType": "StringContains",
    "key": "data.key1",
    "values": [
        "microsoft", 
        "azure"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF key CONTAINS filter
            MATCH
```

## <a name="stringnotcontains"></a>StringNotContains
El operador **StringNotContains** se evalúa en true si la **clave** **no contiene** los valores de **filtro** especificados como subcadenas. Si la clave contiene uno de los valores especificados como una subcadena, el operador se evalúa en false. En el ejemplo siguiente, el operador devuelve true solo si el valor del atributo `key1` de la sección `data` no tiene `contoso` y `fabrikam` como subcadenas. 

```json
"advancedFilters": [{
    "operatorType": "StringNotContains",
    "key": "data.key1",
    "values": [
        "contoso", 
        "fabrikam"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF key CONTAINS filter
            FAIL_MATCH
```
Consulte la sección [Limitaciones](#limitations) para ver las limitaciones actuales de este operador.

## <a name="stringbeginswith"></a>StringBeginsWith
El operador **StringBeginsWith** se evalúa en true si el valor de la **clave** **comienza por** cualquiera de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` comienza por `event` o `grid`. Por ejemplo, `event hubs` comienza por `event`.  

```json
"advancedFilters": [{
    "operatorType": "StringBeginsWith",
    "key": "data.key1",
    "values": [
        "event", 
        "message"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF key BEGINS_WITH filter
            MATCH
```

## <a name="stringnotbeginswith"></a>StringNotBeginsWith
El operador **StringNotBeginsWith** se evalúa en true si el valor de la **clave** **no comienza por** ninguno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` no comienza por `event` ni `message`.

```json
"advancedFilters": [{
    "operatorType": "StringNotBeginsWith",
    "key": "data.key1",
    "values": [
        "event", 
        "message"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF key BEGINS_WITH filter
            FAIL_MATCH
```

## <a name="stringendswith"></a>StringEndsWith
El operador **StringEndsWith** se evalúa en true si el valor de la **clave** **termina con** uno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` termina con `jpg`, `jpeg` o `png`. Por ejemplo, `eventgrid.png` termina con `png`.


```json
"advancedFilters": [{
    "operatorType": "StringEndsWith",
    "key": "data.key1",
    "values": [
        "jpg", 
        "jpeg", 
        "png"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF key ENDS_WITH filter
            MATCH
```

## <a name="stringnotendswith"></a>StringNotEndsWith
El operador **StringNotEndsWith** se evalúa en true si el valor de la **clave** **no termina con** ninguno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` no termina con `jpg`, `jpeg` ni `png`. 


```json
"advancedFilters": [{
    "operatorType": "StringNotEndsWith",
    "key": "data.key1",
    "values": [
        "jpg", 
        "jpeg", 
        "png"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF key ENDS_WITH filter
            FAIL_MATCH
```

## <a name="stringin"></a>StringIn
El operador **StringIn** comprueba si el valor de la **clave** **coincide exactamente con** alguno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` es `contoso`, `fabrikam` o `factory`. 

```json
"advancedFilters": [{
    "operatorType": "StringIn",
    "key": "data.key1",
    "values": [
        "contoso", 
        "fabrikam", 
        "factory"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF filter == key
            MATCH
```

## <a name="stringnotin"></a>StringNotIn
El operador **StringNotIn** comprueba si el valor de la **clave** **no coincide** con ninguno de los valores de **filtro** especificados. En el ejemplo siguiente, comprueba si el valor del atributo `key1` de la sección `data` no es `aws` ni `bridge`. 

```json
"advancedFilters": [{
    "operatorType": "StringNotIn",
    "key": "data.key1",
    "values": [
        "aws", 
        "bridge"
    ]
}]
```

Si la clave es una matriz, todos los valores de la matriz se comprueban con la matriz de valores de filtro. Este es el pseudocódigo con la clave `[v1, v2, v3]` y el filtro `[a,b,c]`. Los valores de clave con tipos de datos que no coinciden con el tipo de datos del filtro se ignoran.

```
FOR_EACH filter IN (a, b, c)
    FOR_EACH key IN (v1, v2, v3)
        IF filter == key
            FAIL_MATCH
```


Las comparaciones de cadenas no distinguen mayúsculas de minúsculas.

> [!NOTE]
> Si el JSON del evento no contiene la clave de filtro avanzada, el filtro se evalúa en **no coincidente** para los operadores siguientes: NumberGreaterThan, NumberGreaterThanOrEquals, NumberLessThan, NumberLessThanOrEquals, NumberIn, BoolEquals, StringContains, StringNotContains, StringBeginsWith, StringNotBeginsWith, StringEndsWith, StringNotEndsWith y StringIn.
> 
>El filtro se evalúa en **coincidente** para los operadores siguientes: NumberNotIn y StringNotIn.


## <a name="isnullorundefined"></a>IsNullOrUndefined
El operador IsNullOrUndefined se evalúa en true si el valor de la clave es NULL o no está definido. 

```json
{
    "operatorType": "IsNullOrUndefined",
    "key": "data.key1"
}
```

En el ejemplo siguiente, falta key1, por lo que el operador se evaluaría en true. 

```json
{ 
    "data": 
    { 
        "key2": 5 
    } 
}
```

En el ejemplo siguiente, key1 está establecido en NULL, por lo que el operador se evaluaría en true.

```json
{
    "data": 
    { 
        "key1": null
    }
}
```

Si key1 tiene cualquier otro valor en estos ejemplos, el operador se evaluaría en false. 

## <a name="isnotnull"></a>IsNotNull
El operador IsNotNull se evalúa en true si el valor de la clave no es NULL o no está definido. 

```json
{
    "operatorType": "IsNotNull",
    "key": "data.key1"
}
```

## <a name="or-and-and"></a>OR y AND
Si especifica un único filtro con varios valores, se realiza una operación **OR**, por lo que el valor del campo de clave debe ser uno de estos valores. Este es un ejemplo:

```json
"advancedFilters": [
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/microsoft.devtestlab/",
            "/providers/Microsoft.Compute/virtualMachines/"
        ]
    }
]
```

Si especifica varios filtros diferentes, se realiza una operación **AND**, por lo que se debe cumplir cada condición de filtro. Este es un ejemplo: 

```json
"advancedFilters": [
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/microsoft.devtestlab/"
        ]
    },
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/Microsoft.Compute/virtualMachines/"
        ]
    }
]
```

## <a name="cloudevents"></a>CloudEvents 
Para los eventos del **esquema de CloudEvents**, utilice los valores siguientes para la clave: `eventid`, `source`, `eventtype`, `eventtypeversion` o datos de evento (como `data.key1`). 

También puede usar [atributos de contexto de extensión en CloudEvents 1.0](https://github.com/cloudevents/spec/blob/v1.0.1/spec.md#extension-context-attributes). En el ejemplo siguiente, `comexampleextension1` y `comexampleothervalue` son atributos de contexto de extensión. 

```json
{
    "specversion" : "1.0",
    "type" : "com.example.someevent",
    "source" : "/mycontext",
    "id" : "C234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "subject": null,
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "application/json",
    "data" : {
        "appinfoA" : "abc",
        "appinfoB" : 123,
        "appinfoC" : true
    }
}
```

Este es un ejemplo del uso de un atributo de contexto de extensión en un filtro.

```json
"advancedFilters": [{
    "operatorType": "StringBeginsWith",
    "key": "comexampleothervalue",
    "values": [
        "5", 
        "1"
    ]
}]
```


## <a name="limitations"></a>Limitaciones

El filtrado avanzado tiene las siguientes limitaciones:

* 5 filtros avanzados y 25 valores de filtro en todos los filtros por suscripción de Event Grid
* 512 caracteres por valor de cadena
* 5 valores para los operadores **in** y **not in**
* El operador `StringNotContains` no está disponible actualmente en el portal.
* Claves que incluyen el carácter **`.` (punto)** . Por ejemplo, `http://schemas.microsoft.com/claims/authnclassreference` o `john.doe@contoso.com`. Actualmente, las claves no admiten caracteres de escape. 

La misma clave se puede usar en más de un filtro.





## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre el filtrado de eventos con PowerShell y la CLI de Azure, consulte [Filter events for Event Grid](how-to-filter-events.md) (Filtrado de eventos para Event Grid).
* Para comenzar a usar rápidamente Event Grid, vea [Creación y enrutamiento de eventos personalizados con Azure Event Grid](custom-event-quickstart.md).
