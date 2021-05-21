---
title: Sintaxis y formato de las condiciones de asignación de roles (versión preliminar) de Azure RBAC
description: Obtenga información general sobre el formato y la sintaxis de las condiciones de asignación de roles de Azure en el control de acceso basado en atributos de Azure (Azure ABAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: conceptual
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.openlocfilehash: b271da3cf8e591df8557133abcff248a737645e5
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490040"
---
# <a name="azure-role-assignment-condition-format-and-syntax-preview"></a>Sintaxis y formato de las condiciones de asignación de roles de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Una condición es una comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso. Por ejemplo, puede agregar una condición que requiera que un objeto tenga una etiqueta específica para poder leerlo. En este artículo se describen el formato y la sintaxis de las condiciones de asignación de roles.

## <a name="condition-format"></a>Formato de condición

Para comprender mejor las condiciones de asignación de roles, examine el formato.

### <a name="simple-condition"></a>Condición simple

La condición más básica consta de una acción de destino y una expresión. Una acción es una operación que un usuario puede realizar en un tipo de recurso. Una expresión es una instrucción que se evalúa como verdadera o falsa, que determina si se permite realizar la acción.

A continuación, se muestra el formato de una condición simple.

![Formato de una condición simple con una sola acción y una sola expresión.](./media/conditions-format/format-simple.png)

La siguiente condición tiene una acción de "Leer un blob". La expresión comprueba si el nombre del contenedor es blobs-example-container.

![Ejemplo de condición simple con una acción de lectura de blobs y una expresión de nombre de contenedor.](./media/conditions-format/format-simple-example.png)

![Diagrama que muestra el acceso de lectura a blobs con un nombre de contenedor determinado.](./media/conditions-format/format-simple-example-diagram.png)

### <a name="how-a-condition-is-evaluated"></a>Cómo se evalúa una condición

Si un usuario intenta realizar una acción en la asignación de roles que no es `<action>`, `!(ActionMatches)` se evalúa como verdadera y la condición general se evalúa como verdadera para permitir que se realice la acción.

Si un usuario intenta realizar `<action>` en la asignación de roles, `!(ActionMatches)` se evalúa como falsa, por lo que se evalúa la expresión. Si la expresión se evalúa como verdadera, la condición general se evalúa como verdadera para permitir que se realice `<action>`. De lo contrario, no se permite realizar `<action>`.

El pseudocódigo siguiente muestra otra manera de leer esta condición.

```
if a user tries to perform an action in the role assignment that does not match <action>
{
    Allow action to be performed
}
else
{
    if <attribute> <operator> <value> is true
    {
        Allow <action> to be performed
    }
    else
    {
        Do not allow <action> to be performed
    }
}
```

### <a name="suboperations"></a>Suboperaciones

Algunas acciones tienen suboperaciones. Por ejemplo, la acción de datos "Leer un blob" tiene la suboperación "Leer contenido de un blob con condiciones de etiqueta". Las condiciones con suboperaciones tienen el formato siguiente.

![Formato de una acción con una suboperación.](./media/conditions-format/format-suboperation.png)

### <a name="multiple-actions"></a>Varias acciones

Una condición puede incluir varias acciones que quiere permitir si la condición es verdadera. Si selecciona varias acciones para una sola condición, puede que haya menos atributos para elegir para la condición, ya que los atributos deben estar disponibles en las acciones seleccionadas.

![Formato de varias acciones que se pueden permitir si la condición es verdadera.](./media/conditions-format/format-multiple-actions.png)

### <a name="multiple-expressions"></a>Varias expresiones

Una condición puede incluir varias expresiones. Según el operador, los atributos se pueden comprobar con varios valores.

![Formato para varias expresiones mediante operadores booleanos y varios valores.](./media/conditions-format/format-multiple-expressions.png)

### <a name="multiple-conditions"></a>Varias condiciones

También puede combinar condiciones para dirigirse a varias acciones.

![Formato para varias condiciones mediante operadores booleanos.](./media/conditions-format/format-multiple-conditions.png)

## <a name="condition-syntax"></a>Sintaxis de condición

A continuación, se muestra la sintaxis de una condición de asignación de roles.

```
(
    (
        !(ActionMatches{'<action>'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'<subOperation>'})
        AND
        !(ActionMatches{'<action>'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'<subOperation>'})
        AND
        ...
    )
    OR
    (
        <attribute> <operator> {<value, <value>, ...}
        AND | OR
        <attribute> <operator> {<value>, <value>, ...}
        AND | OR
        ...
    )
)
AND
(
    (
        !(ActionMatches{'<action>'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'<subOperation>'})
        AND
        !(ActionMatches{'<action>'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'<subOperation>'})
        AND
        ...
    )
    OR
    (
        <attribute> <operator> {<value, <value>, ...}
        AND | OR
        <attribute> <operator> {<value>, <value>, ...}
        AND | OR
        ...
    )
)
AND
...
```

## <a name="actions"></a>Acciones

Actualmente, se pueden agregar condiciones a asignaciones de roles integradas o personalizadas que tienen blobs de almacenamiento. Entre estas se incluyen los siguientes roles integrados:

- [Colaborador de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-contributor)
- [Propietario de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-owner)
- [Lector de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-reader)

Para ver una lista de las acciones de los blobs de almacenamiento que puede usar en las condiciones, consulte [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](../storage/common/storage-auth-abac-attributes.md).

## <a name="attributes"></a>Atributos

En función de las acciones seleccionadas, el atributo puede encontrarse en distintos lugares. Si selecciona varias acciones para una sola condición, puede que haya menos atributos para elegir para la condición, ya que los atributos deben estar disponibles en las acciones seleccionadas. Para especificar un atributo, debe incluir el origen como prefijo.

> [!div class="mx-tableFixed"]
> | Origen de atributo | Descripción | Código |
> | --- | --- | --- |
> | Resource | Indica que el atributo está en el recurso, como el nombre del contenedor. | `@Resource` |
> | Solicitud | Indica que el atributo forma parte de la solicitud de acción, como establecer la etiqueta de índice del blob. | `@Request` |

Para ver una lista de los atributos de blob de almacenamiento que puede usar en las condiciones, consulte [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](../storage/common/storage-auth-abac-attributes.md).

## <a name="operators"></a>Operadores

En la tabla siguiente se enumeran los operadores que están disponibles para construir condiciones.

| Category | Operator | Descripción |
| --- | --- | --- |
| Comparación lógica |`AND`<br/>`&&` | Operador AND |
|  | `OR`<br/>`||` | Operador OR |
|  | `NOT`<br/>`!` | Operador NOT o de negación |
| Comparación de cadenas | `StringEquals`<br/>`StringEqualsIgnoreCase` | Coincidencia que distingue mayúsculas de minúsculas (o no distingue mayúsculas de minúsculas) Los valores deben coincidir exactamente con la cadena. |
|  | `StringNotEquals`<br/>`StringNotEqualsIgnoreCase` | Operador de negación de `StringEquals` (o `StringEqualsIgnoreCase`) |
|  | `StringStartsWith`<br/>`StringStartsWithIgnoreCase` | Coincidencia que distingue mayúsculas de minúsculas (o no distingue mayúsculas de minúsculas) Los valores comienzan con la cadena. |
|  | `StringNotStartsWith`<br/>`StringNotStartsWithIgnoreCase` | Operador de negación de `StringStartsWith` (o `StringStartsWithIgnoreCase`) |
|  | `StringLike`<br/>`StringLikeIgnoreCase` | Coincidencia que distingue mayúsculas de minúsculas (o no distingue mayúsculas de minúsculas) Los valores pueden incluir un carácter comodín de coincidencia con varios caracteres (`*`) o un carácter comodín de coincidencia con un solo carácter (`?`) en cualquier parte de la cadena. Si es necesario, estos caracteres se pueden escapar agregando una barra diagonal inversa `\*` y `\?`. |
|  | `StringNotLike`<br/>`StringNotLikeIgnoreCase` | Operador de negación de `StringLike` (o `StringLikeIgnoreCase`) |
| Comparación numérica | `NumericEquals`<br/>`NumericNotEquals`<br/>`NumericLessThan`<br/>`NumericLessThanEquals`<br/>`NumericGreaterThan`<br/>`NumericGreaterThanEquals` | Actualmente, solo se admiten números enteros. |
| Funciones de nivel superior | `ActionMatches` | Comprueba si el valor Action[ID] coincide con el patrón de acción especificado. Este operador es equivalente a la lógica de coincidencia de acciones que usa el SDK al comparar una acción con un patrón de acción dentro de un permiso. |
| Comparación de productos | `ForAnyOfAnyValues:StringEquals`<br/>`ForAnyOfAnyValues:StringEqualsIgnoreCase`<br/>`ForAnyOfAnyValues:StringNotEquals`<br/>`ForAnyOfAnyValues:StringNotEqualsIgnoreCase`<br/>`ForAnyOfAnyValues:StringLike`<br/>`ForAnyOfAnyValues:StringLikeIgnoreCase`<br/>`ForAnyOfAnyValues:StringNotLike`<br/>`ForAnyOfAnyValues:StringNotLikeIgnoreCase`<br/>`ForAnyOfAnyValues:NumericEquals`<br/>`ForAnyOfAnyValues:NumericNotEquals`<br/>`ForAnyOfAnyValues:NumericGreaterThan`<br/>`ForAnyOfAnyValues:NumericGreaterThanEquals`<br/>`ForAnyOfAnyValues:NumericLessThan`<br/>`ForAnyOfAnyValues:NumericLessThanEquals` | Si al menos un valor del lado izquierdo satisface la comparación con al menos un valor del lado derecho, la expresión se evalúa como verdadera. Tiene el formato `ForAnyOfAnyValues:<BooleanFunction>`. Admite varias cadenas y números. |
|  | `ForAllOfAnyValues:StringEquals`<br/>`ForAllOfAnyValues:StringEqualsIgnoreCase`<br/>`ForAllOfAnyValues:StringNotEquals`<br/>`ForAllOfAnyValues:StringNotEqualsIgnoreCase`<br/>`ForAllOfAnyValues:StringLike`<br/>`ForAllOfAnyValues:StringLikeIgnoreCase`<br/>`ForAllOfAnyValues:StringNotLike`<br/>`ForAllOfAnyValues:StringNotLikeIgnoreCase`<br/>`ForAllOfAnyValues:NumericEquals`<br/>`ForAllOfAnyValues:NumericNotEquals`<br/>`ForAllOfAnyValues:NumericGreaterThan`<br/>`ForAllOfAnyValues:NumericGreaterThanEquals`<br/>`ForAllOfAnyValues:NumericLessThan`<br/>`ForAllOfAnyValues:NumericLessThanEquals` | Si cada valor del lado izquierdo satisface la comparación con al menos un valor del lado derecho, la expresión se evalúa como verdadera. Tiene el formato `ForAllOfAnyValues:<BooleanFunction>`. Admite varias cadenas y números. |
|  | `ForAnyOfAllValues:StringEquals`<br/>`ForAnyOfAllValues:StringEqualsIgnoreCase`<br/>`ForAnyOfAllValues:StringNotEquals`<br/>`ForAnyOfAllValues:StringNotEqualsIgnoreCase`<br/>`ForAnyOfAllValues:StringLike`<br/>`ForAnyOfAllValues:StringLikeIgnoreCase`<br/>`ForAnyOfAllValues:StringNotLike`<br/>`ForAnyOfAllValues:StringNotLikeIgnoreCase`<br/>`ForAnyOfAllValues:NumericEquals`<br/>`ForAnyOfAllValues:NumericNotEquals`<br/>`ForAnyOfAllValues:NumericGreaterThan`<br/>`ForAnyOfAllValues:NumericGreaterThanEquals`<br/>`ForAnyOfAllValues:NumericLessThan`<br/>`ForAnyOfAllValues:NumericLessThanEquals` | Si al menos un valor del lado izquierdo satisface la comparación con cada valor del lado derecho, la expresión se evalúa como verdadera. Tiene el formato `ForAnyOfAllValues:<BooleanFunction>`. Admite varias cadenas y números. |
|  | `ForAllOfAllValues:StringEquals`<br/>`ForAllOfAllValues:StringEqualsIgnoreCase`<br/>`ForAllOfAllValues:StringNotEquals`<br/>`ForAllOfAllValues:StringNotEqualsIgnoreCase`<br/>`ForAllOfAllValues:StringLike`<br/>`ForAllOfAllValues:StringLikeIgnoreCase`<br/>`ForAllOfAllValues:StringNotLike`<br/>`ForAllOfAllValues:StringNotLikeIgnoreCase`<br/>`ForAllOfAllValues:NumericEquals`<br/>`ForAllOfAllValues:NumericNotEquals`<br/>`ForAllOfAllValues:NumericGreaterThan`<br/>`ForAllOfAllValues:NumericGreaterThanEquals`<br/>`ForAllOfAllValues:NumericLessThan`<br/>`ForAllOfAllValues:NumericLessThanEquals` | Si cada valor del lado izquierdo satisface la comparación con cada valor del lado derecho, la expresión se evalúa como verdadera. Tiene el formato `ForAllOfAllValues:<BooleanFunction>`. Admite varias cadenas y números. |

## <a name="operator-examples"></a>Ejemplos de operadores

> [!div class="mx-tableFixed"]
> | Ejemplo | Resultado |
> | --- | --- |
> | `ActionMatches{'Microsoft.Authorization/roleAssignments/*'}` | Si la acción que se comprueba es igual a "Microsoft.Authorization/roleAssignments/write", entonces es verdadera. |
> | `ActionMatches{'Microsoft.Authorization/roleDefinitions/*'}` | Si la acción que se comprueba es igual a "Microsoft.Authorization/roleAssignments/write", entonces es falsa. |
> | `Resource[name1] StringLike 'a*c?'` | Si Resource[name1] es igual a "abcd", entonces es verdadero. |
> | `Resource[name1] StringLike 'A*C?'` | Si Resource[name1] es igual a "abcd", entonces es falso. |
> | `Resource[name1] StringLike 'a*c'` | Si Resource[name1] es igual a "abcd", entonces es falso. |
> | `{'red', 'blue'} ForAnyOfAnyValues:StringEquals {'blue', 'green'}` | true |
> | `{'red', 'blue'} ForAnyOfAnyValues:StringEquals {'orange', 'green'}` | false |
> | `{'red', 'blue'} ForAllOfAnyValues:StringEquals {'orange', 'red', 'blue'}` | true |
> | `{'red', 'blue'} ForAllOfAnyValues:StringEquals {'red', 'green'}` | false |
> | `{10, 20} ForAnyOfAllValues:NumericLessThan {15, 18}` | true |
> | `{10, 20} ForAllOfAllValues:NumericLessThan {5, 15, 18}` | false |
> | `{10, 20} ForAllOfAllValues:NumericLessThan {25, 30}` | true |
> | `{10, 20} ForAllOfAllValues:NumericLessThan {15, 25, 30}` | false |

## <a name="special-characters"></a>Caracteres especiales

| Carácter | Descripción |
| --- | --- |
| `*` | Un asterisco (*) representa una coincidencia de carácter comodín con varios caracteres que se pueden usar con operadores `Like`. Si es necesario, puede escapar un asterisco agregando una barra diagonal inversa `\*`. |
| `?` | Un signo de interrogación (*) representa una coincidencia de carácter comodín con un solo carácter que se puede usar con operadores `Like`. Si es necesario, puede escapar un signo de interrogación agregando una barra diagonal inversa `\?`. |
| `$` | Se usa un signo de dólar ($) para ayudar a delinear las claves de etiqueta. En Azure PowerShell, si una cadena entre comillas dobles (") incluye un signo de dólar, debe ir precedido por una comilla simple (\`). Por ejemplo: ``tags:Project<`$key_case_sensitive`$>``. | 

## <a name="grouping-and-precedence"></a>Agrupación y precedencia

Los paréntesis `()` se usan para agrupar expresiones y definir la precedencia en una condición. Si tiene tres o más expresiones para una acción de destino, debe agregar paréntesis para definir el orden en que se evalúan las expresiones. Las expresiones entre paréntesis tienen mayor precedencia. Por ejemplo, si tiene la siguiente expresión:

```
a AND b OR c
```

Debe agregar paréntesis de una de las maneras siguientes:

```
(a AND b) OR c
```

```
a AND (b OR c)
```

## <a name="next-steps"></a>Pasos siguientes

- [Ejemplo de condiciones de asignación de roles de Azure (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](../storage/common/storage-auth-abac-attributes.md)
