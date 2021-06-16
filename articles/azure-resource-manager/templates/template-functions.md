---
title: Funciones de plantillas
description: Se describen las funciones que se van a usar en una plantilla de Azure Resource Manager para recuperar valores, trabajar con cadenas y valores numéricos y recuperar información de implementación.
ms.topic: conceptual
ms.date: 10/12/2020
ms.openlocfilehash: 012e570a769e8ea969d0c527a71053f55ecdf5a9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959402"
---
# <a name="arm-template-functions"></a>Funciones de plantilla de ARM

En este artículo se describen todas las funciones que puede utilizar en una plantilla de Azure Resource Manager. Para obtener información sobre el uso de funciones en la plantilla, consulte la [sintaxis de la plantilla](template-expressions.md).

Para crear sus propias funciones, consulte [Funciones definidas por el usuario](./syntax.md#functions).

La mayoría de las funciones operan de la misma forma cuando se implementan en un grupo de recursos, en una suscripción, en un grupo de administración o en un inquilino. Hay algunas funciones que no pueden utilizarse en todos los ámbitos. Se indican en las listas siguientes.

<a id="array" aria-hidden="true"></a>
<a id="concatarray" aria-hidden="true"></a>
<a id="contains" aria-hidden="true"></a>
<a id="createarray" aria-hidden="true"></a>
<a id="empty" aria-hidden="true"></a>
<a id="first" aria-hidden="true"></a>
<a id="intersection" aria-hidden="true"></a>
<a id="last" aria-hidden="true"></a>
<a id="length" aria-hidden="true"></a>
<a id="min" aria-hidden="true"></a>
<a id="max" aria-hidden="true"></a>
<a id="range" aria-hidden="true"></a>
<a id="skip" aria-hidden="true"></a>
<a id="take" aria-hidden="true"></a>
<a id="union" aria-hidden="true"></a>

## <a name="any-function"></a>Cualquier función

La [función any](../bicep/bicep-functions-any.md) está disponible en Bicep para ayudar a resolver problemas relacionados con las advertencias de tipos de datos.

## <a name="array-functions"></a>Funciones de matriz

Resource Manager ofrece varias funciones para trabajar con matrices.

* [array](template-functions-array.md#array)
* [concat](template-functions-array.md#concat)
* [contains](template-functions-array.md#contains)
* [createArray](template-functions-array.md#createarray)
* [empty](template-functions-array.md#empty)
* [first](template-functions-array.md#first)
* [intersection](template-functions-array.md#intersection)
* [last](template-functions-array.md#last)
* [length](template-functions-array.md#length)
* [min](template-functions-array.md#min)
* [max](template-functions-array.md#max)
* [range](template-functions-array.md#range)
* [skip](template-functions-array.md#skip)
* [take](template-functions-array.md#take)
* [union](template-functions-array.md#union)

<a id="coalesce" aria-hidden="true"></a>
<a id="equals" aria-hidden="true"></a>
<a id="less" aria-hidden="true"></a>
<a id="lessorequals" aria-hidden="true"></a>
<a id="greater" aria-hidden="true"></a>
<a id="greaterorequals" aria-hidden="true"></a>

## <a name="comparison-functions"></a>Funciones de comparación

Resource Manager proporciona varias funciones para realizar comparaciones en las plantillas.

* [coalesce](template-functions-comparison.md#coalesce)
* [equals](template-functions-comparison.md#equals)
* [less](template-functions-comparison.md#less)
* [lessOrEquals](template-functions-comparison.md#lessorequals)
* [greater](template-functions-comparison.md#greater)
* [greaterOrEquals](template-functions-comparison.md#greaterorequals)

<a id="deployment" aria-hidden="true"></a>
<a id="parameters" aria-hidden="true"></a>
<a id="variables" aria-hidden="true"></a>

## <a name="date-functions"></a>Funciones de fecha

Resource Manager ofrece las siguientes funciones para trabajar con fechas.

* [dateTimeAdd](template-functions-date.md#datetimeadd)
* [utcNow](template-functions-date.md#utcnow)

## <a name="deployment-value-functions"></a>Funciones con valores de implementación

El Administrador de recursos ofrece las siguientes funciones para obtener valores de las secciones de la plantilla y valores relacionados con la implementación:

* [deployment](template-functions-deployment.md#deployment)
* [environment](template-functions-deployment.md#environment)
* [parameters](template-functions-deployment.md#parameters)
* [variables](template-functions-deployment.md#variables)

<a id="and" aria-hidden="true"></a>
<a id="bool" aria-hidden="true"></a>
<a id="if" aria-hidden="true"></a>
<a id="not" aria-hidden="true"></a>
<a id="or" aria-hidden="true"></a>

## <a name="logical-functions"></a>Funciones lógicas

Resource Manager ofrece las siguientes funciones para trabajar con condiciones lógicas:

* [and](template-functions-logical.md#and)
* [bool](template-functions-logical.md#bool)
* [false](template-functions-logical.md#false)
* [if](template-functions-logical.md#if)
* [not](template-functions-logical.md#not)
* [or](template-functions-logical.md#or)
* [true](template-functions-logical.md#true)

<a id="add" aria-hidden="true"></a>
<a id="copyindex" aria-hidden="true"></a>
<a id="div" aria-hidden="true"></a>
<a id="float" aria-hidden="true"></a>
<a id="int" aria-hidden="true"></a>
<a id="minint" aria-hidden="true"></a>
<a id="maxint" aria-hidden="true"></a>
<a id="mod" aria-hidden="true"></a>
<a id="mul" aria-hidden="true"></a>
<a id="sub" aria-hidden="true"></a>

## <a name="numeric-functions"></a>Funciones numéricas

El Administrador de recursos ofrece las siguientes funciones para trabajar con números enteros:

* [add](template-functions-numeric.md#add)
* [copyIndex](template-functions-numeric.md#copyindex)
* [div](template-functions-numeric.md#div)
* [float](template-functions-numeric.md#float)
* [int](template-functions-numeric.md#int)
* [min](template-functions-numeric.md#min)
* [max](template-functions-numeric.md#max)
* [mod](template-functions-numeric.md#mod)
* [mul](template-functions-numeric.md#mul)
* [sub](template-functions-numeric.md#sub)

<a id="json" aria-hidden="true"></a>

## <a name="object-functions"></a>Funciones de objeto

Resource Manager ofrece varias funciones para trabajar con objetos.

* [contains](template-functions-object.md#contains)
* [createObject](template-functions-object.md#createobject)
* [empty](template-functions-object.md#empty)
* [intersection](template-functions-object.md#intersection)
* [json](template-functions-object.md#json)
* [length](template-functions-object.md#length)
* [null](template-functions-object.md#null)
* [union](template-functions-object.md#union)

<a id="extensionResourceId" aria-hidden="true"></a>
<a id="listkeys" aria-hidden="true"></a>
<a id="list" aria-hidden="true"></a>
<a id="providers" aria-hidden="true"></a>
<a id="reference" aria-hidden="true"></a>
<a id="resourcegroup" aria-hidden="true"></a>
<a id="resourceid" aria-hidden="true"></a>
<a id="subscription" aria-hidden="true"></a>
<a id="subscriptionResourceId" aria-hidden="true"></a>
<a id="tenantResourceId" aria-hidden="true"></a>

## <a name="resource-functions"></a>Funciones de recursos

El Administrador de recursos ofrece las siguientes funciones para obtener valores de recursos:

* [extensionResourceId](template-functions-resource.md#extensionresourceid)
* [listAccountSas](template-functions-resource.md#list)
* [listKeys](template-functions-resource.md#listkeys)
* [listSecrets](template-functions-resource.md#list)
* [list*](template-functions-resource.md#list)
* [pickZones](template-functions-resource.md#pickzones)
* [reference](template-functions-resource.md#reference)
* [resourceGroup](template-functions-resource.md#resourcegroup): solo puede utilizarse cuando la implementación se realiza en un grupo de recursos.
* [resourceId](template-functions-resource.md#resourceid): puede utilizarse con cualquier ámbito, pero los valores correctos cambiarán en función de este.
* [subscription](template-functions-resource.md#subscription): solo puede utilizarse cuando la implementación se realiza en un grupo de recursos o una suscripción.
* [subscriptionResourceId](template-functions-resource.md#subscriptionresourceid)
* [tenantResourceId](template-functions-resource.md#tenantresourceid)

<a id="base64" aria-hidden="true"></a>
<a id="base64tojson" aria-hidden="true"></a>
<a id="base64tostring" aria-hidden="true"></a>
<a id="concat" aria-hidden="true"></a>
<a id="containsstring" aria-hidden="true"></a>
<a id="datauri" aria-hidden="true"></a>
<a id="datauritostring" aria-hidden="true"></a>
<a id="emptystring" aria-hidden="true"></a>
<a id="endswith" aria-hidden="true"></a>
<a id="firststring" aria-hidden="true"></a>
<a id="guid" aria-hidden="true"></a>
<a id="indexof" aria-hidden="true"></a>
<a id="laststring" aria-hidden="true"></a>
<a id="lastindexof" aria-hidden="true"></a>
<a id="lengthstring" aria-hidden="true"></a>
<a id="padleft" aria-hidden="true"></a>
<a id="replace" aria-hidden="true"></a>
<a id="skipstring" aria-hidden="true"></a>
<a id="split" aria-hidden="true"></a>
<a id="startswith" aria-hidden="true"></a>
<a id="string" aria-hidden="true"></a>
<a id="substring" aria-hidden="true"></a>
<a id="takestring" aria-hidden="true"></a>
<a id="tolower" aria-hidden="true"></a>
<a id="toupper" aria-hidden="true"></a>
<a id="trim" aria-hidden="true"></a>
<a id="uniquestring" aria-hidden="true"></a>
<a id="uri" aria-hidden="true"></a>
<a id="uricomponent" aria-hidden="true"></a>
<a id="uricomponenttostring" aria-hidden="true"></a>

## <a name="string-functions"></a>Funciones de cadena

El Administrador de recursos ofrece las siguientes funciones para trabajar con cadenas:

* [base64](template-functions-string.md#base64)
* [base64ToJson](template-functions-string.md#base64tojson)
* [base64ToString](template-functions-string.md#base64tostring)
* [concat](template-functions-string.md#concat)
* [contains](template-functions-string.md#contains)
* [dataUri](template-functions-string.md#datauri)
* [dataUriToString](template-functions-string.md#datauritostring)
* [empty](template-functions-string.md#empty)
* [endsWith](template-functions-string.md#endswith)
* [first](template-functions-string.md#first)
* [format](template-functions-string.md#format)
* [guid](template-functions-string.md#guid)
* [indexOf](template-functions-string.md#indexof)
* [last](template-functions-string.md#last)
* [lastIndexOf](template-functions-string.md#lastindexof)
* [length](template-functions-string.md#length)
* [newGuid](template-functions-string.md#newguid)
* [padLeft](template-functions-string.md#padleft)
* [replace](template-functions-string.md#replace)
* [skip](template-functions-string.md#skip)
* [split](template-functions-string.md#split)
* [startsWith](template-functions-string.md#startswith)
* [string](template-functions-string.md#string)
* [substring](template-functions-string.md#substring)
* [take](template-functions-string.md#take)
* [toLower](template-functions-string.md#tolower)
* [toUpper](template-functions-string.md#toupper)
* [trim](template-functions-string.md#trim)
* [uniqueString](template-functions-string.md#uniquestring)
* [uri](template-functions-string.md#uri)
* [uriComponent](template-functions-string.md#uricomponent)
* [uriComponentToString](template-functions-string.md#uricomponenttostring)

## <a name="next-steps"></a>Pasos siguientes

* Puede encontrar una descripción de las secciones de una plantilla de Azure Resource Manager en [Nociones sobre la estructura y la sintaxis de las plantillas de Resource Manager](./syntax.md).
* Para combinar varias plantillas, consulte [Uso de plantillas vinculadas y anidadas al implementar recursos de Azure](linked-templates.md).
* Para iterar un número especificado de veces al crear un tipo de recurso, consulte [Iteración de recursos en las plantillas de Resource Manager](copy-resources.md).
* Para ver cómo implementar la plantilla que ha creado, consulte [Implementación de recursos con plantillas de Resource Manager y Azure PowerShell](deploy-powershell.md).