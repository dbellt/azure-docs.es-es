---
title: Detalles de la estructura de definición de directivas
description: Describe cómo se usan las definiciones de directiva para establecer convenciones para los recursos de Azure de su organización.
ms.date: 05/01/2021
ms.topic: conceptual
ms.openlocfilehash: 594dbfe3dda919e4d8dcbf3047fac78bad600127
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108326208"
---
# <a name="azure-policy-definition-structure"></a>Estructura de definición de Azure Policy

Azure Policy establece las convenciones de los recursos. Las definiciones de directiva describen las [condiciones](#conditions) de cumplimiento de los recursos y qué sucederá si se cumple una condición. Una condición compara un [campo](#fields) o un [valor](#value) de propiedad de recurso con un valor requerido. Para acceder a los campos de propiedad de recurso, se usa [alias](#aliases). Cuando el campo de una propiedad de recurso es una matriz, se puede usar un [alias de matriz](#understanding-the--alias) especial para seleccionar valores de todos los miembros de la matriz y aplicar una condición a cada uno de ellos. Más información sobre las [condiciones](#conditions).

La definición de convenciones permite controlar los costes y administrar los recursos más fácilmente. Por ejemplo, puede especificar que se permitan solo determinados tipos de máquinas virtuales. O bien, puede exigir que todos los recursos tengan una etiqueta concreta. Los recursos secundarios heredan las asignaciones de directivas. Si una asignación de directiva se aplica a un grupo de recursos, será aplicable a todos los recursos de dicho grupo de recursos.

El esquema de definición de directiva _policyRule_ se encuentra aquí: [https://schema.management.azure.com/schemas/2019-09-01/policyDefinition.json](https://schema.management.azure.com/schemas/2019-09-01/policyDefinition.json)

Para crear una definición de directiva se utiliza JSON. La definición de directiva contiene elementos para:

- nombre para mostrar
- description
- mode
- metadata
- parámetros
- regla de directiva
  - evaluación lógica
  - efecto

Por ejemplo, el siguiente JSON muestra una directiva que limita las ubicaciones donde se implementan los recursos:

```json
{
    "properties": {
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "mode": "Indexed",
        "metadata": {
            "version": "1.0.0",
            "category": "Locations"
        },
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                },
                "defaultValue": [ "westus2" ]
            }
        },
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

Los patrones y elementos integrados de Azure Policy se encuentran en [Ejemplos de Azure Policy](../samples/index.md).

## <a name="display-name-and-description"></a>Nombre para mostrar y descripción

Use los valores **displayName** y **description** para identificar la definición de directiva y proporcionar el contexto para su uso. **displayName** tiene una longitud máxima de _128_ caracteres y **description** tiene una longitud máxima de _512_ caracteres.

> [!NOTE]
> Durante la creación o actualización de una definición de directiva, las propiedades **id**, **type** y **name** se definen mediante propiedades externas al JSON y no son necesarias en el archivo JSON. Al capturar la definición de directiva mediante el SDK, se devuelven las propiedades **id**, **type** y **name** como parte del JSON, pero cada una de ellas es información de solo lectura relacionada con la definición de directiva.

## <a name="type"></a>Tipo

Aunque no se puede establecer la propiedad **type**, hay tres valores que devuelve el SDK y que son visibles en el portal:

- `Builtin`: Microsoft proporciona y mantiene estas definiciones de directiva.
- `Custom`: todas las definiciones de directiva creadas por los clientes tienen este valor.
- `Static`: indica una definición de directiva [Cumplimiento normativo](./regulatory-compliance.md) con **Propiedad** de Microsoft. Los resultados de cumplimiento de estas definiciones de directiva son los resultados de las auditorías de terceros en la infraestructura de Microsoft. En Azure Portal, este valor se muestra a veces como **Administrado por Microsoft**. Para más información, consulte [Responsabilidad compartida en la nube](../../../security/fundamentals/shared-responsibility.md).

## <a name="mode"></a>Mode

El **modo** se configura en función de si la directiva tiene como destino una propiedad de Azure Resource Manager o una propiedad del proveedor de recursos.

### <a name="resource-manager-modes"></a>Modos de Resource Manager

El **modo** determina qué tipos de recurso se evalúan para una definición de directiva. Los modos admitidos son:

- `all`: evalúe los grupos de recursos, las suscripciones y todos los tipos de recurso
- `indexed`: evalúe solo los tipos de recurso que admitan las etiquetas y la ubicación

Por ejemplo, en un recurso, `Microsoft.Network/routeTables` admite etiquetas y ubicación, y se evalúa en ambos modos. Sin embargo, `Microsoft.Network/routeTables/routes` no se puede etiquetar y no se evalúa en el modo `Indexed`.

Se recomienda que establezca **mode** en `all` en la mayoría de los casos. Todas las definiciones de directivas creadas a través del portal usan el modo `all`. Si usa PowerShell o la CLI de Azure, puede especificar el parámetro **mode** de forma manual. Si la definición de directiva no incluye un valor de **modo**, el valor predeterminado es `all` en Azure PowerShell y `null` en la CLI de Azure. Un modo `null` es lo mismo que usar `indexed` para la compatibilidad con versiones anteriores.

`indexed` debe usarse al crear directivas que apliquen etiquetas o ubicaciones. Aunque no es obligatorio, impide que los recursos que no son compatibles con etiquetas y ubicaciones aparezcan como no compatibles en los resultados de cumplimiento. La excepción son los **grupos de recursos** y las **suscripciones**. Las directivas que aplican la ubicación o etiquetas en un grupo de recursos o suscripción deben establecer **mode** en `all` y tener como destino específico el tipo `Microsoft.Resources/subscriptions/resourceGroups` o `Microsoft.Resources/subscriptions`. Para obtener un ejemplo, consulte [Patrón: Etiquetas: ejemplo n.º 1](../samples/pattern-tags.md). Para obtener una lista de los recursos que admiten etiquetas, consulte [Compatibilidad con etiquetas de los recursos de Azure](../../../azure-resource-manager/management/tag-support.md).

### <a name="resource-provider-modes"></a>Modos del proveedor de recursos

El siguiente modo del proveedor de recursos es totalmente compatible:

- `Microsoft.Kubernetes.Data` para administrar los clústeres de Kubernetes en o fuera de Azure. Las definiciones que utilizan este modo del proveedor de recursos usan los efectos _auditoría_, _denegar_ y _deshabilitado_. El uso del efecto [EnforceOPAConstraint](./effects.md#enforceopaconstraint) está _en desuso_.

Actualmente se admiten los siguientes modos del proveedor de recursos como **versión preliminar**:

- `Microsoft.ContainerService.Data` para administrar reglas del controlador de admisión en [Azure Kubernetes Service](../../../aks/intro-kubernetes.md). Las definiciones que usan este modo del proveedor de recursos **deben** utilizar el efecto [EnforceRegoPolicy](./effects.md#enforceregopolicy). Este modo está _en desuso_.
- `Microsoft.KeyVault.Data` para administrar almacenes y certificados en [Azure Key Vault](../../../key-vault/general/overview.md). Para más información sobre estas definiciones de directiva, consulte [Integrar Azure Key Vault con Azure Policy](../../../key-vault/general/azure-policy.md).

> [!NOTE]
> Los modos del proveedor de recursos solo admiten definiciones de directivas integradas y no admiten [exenciones](./exemption-structure.md).

## <a name="metadata"></a>Metadatos

La propiedad `metadata` opcional almacena información acerca de la definición de la directiva. Los clientes pueden definir las propiedades y los valores útiles para su organización en `metadata`. Aun así, hay algunas propiedades _comunes_ que se usan en Azure Policy y los elementos integrados. Cada propiedad `metadata` tiene un límite de 1024 caracteres.

### <a name="common-metadata-properties"></a>Propiedades de metadatos comunes

- `version` (cadena): realiza el seguimiento de los detalles sobre la versión del contenido de una definición de directiva.
- `category` (cadena): determina en qué categoría de Azure Portal se muestra la definición de directiva.
- `preview` (booleano): marca true o false si la definición de directiva es _versión preliminar_.
- `deprecated` (booleano): marca true o false si la definición de directiva está marcada como _en desuso_.

> [!NOTE]
> El servicio Azure Policy usa las propiedades `version`, `preview` y `deprecated` para transmitir el nivel de cambio a una definición o iniciativa de directiva integradas y el estado. El formato de `version` es: `{Major}.{Minor}.{Patch}`. Determinados estados, como _en desuso_ o _versión preliminar_, están anexados a la propiedad `version` o están en otra propiedad como **booleano**. Para obtener más información sobre la forma en que Azure Policy crea versiones los elementos integrados, vea [Control de versiones integradas](https://github.com/Azure/azure-policy/blob/master/built-in-policies/README.md).

## <a name="parameters"></a>Parámetros

Los parámetros ayudan a simplificar la administración de directivas mediante la reducción del número de definiciones de directiva. Piense en los parámetros como si fueran los campos de un formulario: `name`, `address`, `city`, `state`. Estos parámetros no cambian, pero sí sus valores en función del individuo que rellena el formulario.
Los parámetros funcionan del mismo modo al crear las directivas. Con la inclusión de parámetros en una definición de directiva, puede volver a usar esa directiva en distintos escenarios con valores diferentes.

> [!NOTE]
> Pueden añadirse parámetros a una definición existente y asignada. El nuevo parámetro debe incluir la propiedad **defaultValue**. Así se evita que las asignaciones existentes de la directiva o la iniciativa realizadas indirectamente no sean válidas.

### <a name="parameter-properties"></a>Propiedades del parámetro

Un parámetro tiene las siguientes propiedades que se usan en la definición de directiva:

- `name`: El nombre del parámetro. Lo utiliza la función de la implementación `parameters` dentro de la regla de directiva. Para más información, consulte [Uso de un valor de parámetro](#using-a-parameter-value).
- `type`: Determina si el parámetro es **string**, **array**, **object**, **boolean**, **integer**, **float** o **datetime**.
- `metadata`: Define las subpropiedades que usa principalmente Azure Portal para mostrar información intuitiva:
  - `description`: La explicación de para qué se usa el parámetro. Puede utilizarse para proporcionar ejemplos de valores aceptables.
  - `displayName`: El nombre descriptivo que se muestra en el portal para el parámetro.
  - `strongType`: (Opcional) Se usa al asignar la definición de directiva mediante el portal. Proporciona una lista que tiene en cuenta el contexto. Para más información, consulte [strongType](#strongtype).
  - `assignPermissions`: (Opcional) Establecer como _true_ para que Azure Portal cree asignaciones de roles durante la asignación de directivas. Esta propiedad es útil en caso de que desee asignar permisos fuera del ámbito de asignación. Hay una asignación de roles por cada definición de roles de la directiva (o por cada definición de roles de todas las directivas de la iniciativa). El valor del parámetro debe ser un recurso o un ámbito válidos.
- `defaultValue`: (Opcional) Establece el valor del parámetro en una asignación, si no se especifica ningún valor.
  Requerido cuando se actualiza una definición de directiva existente que está asignada.
- `allowedValues`: (Opcional) Proporciona una matriz de los valores que acepta el parámetro durante la asignación. Las comparaciones de valores permitidas distinguen mayúsculas de minúsculas. 

Por ejemplo, podría definir una definición de directiva para limitar las ubicaciones en las que se pueden implementar los recursos. Un parámetro para esa definición de directiva podría ser **allowedLocations**. Este parámetro podría utilizarse por cada asignación de la definición de directiva para limitar los valores aceptados. El uso de **strongType** proporciona una experiencia mejorada al completar la asignación mediante el portal:

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="using-a-parameter-value"></a>Uso de un valor de parámetro

En la regla de directiva, se hace referencia a los parámetros con la siguiente sintaxis de función `parameters`:

```json
{
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

En este ejemplo se hace referencia al parámetro **allowedLocations** que se mostró en las [ propiedades del parámetro](#parameter-properties).

### <a name="strongtype"></a>strongType

Dentro de la propiedad `metadata`, puede usar **strongType** para proporcionar una lista de opciones de selección múltiple en Azure Portal. **strongType** puede ser un _tipo de recurso_ compatible o un valor permitido. Para determinar si un _tipo de recurso_ es válido para **strongType**, use [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider). El formato de un _tipo de recurso_ **strongType** es `<Resource Provider>/<Resource Type>`. Por ejemplo, `Microsoft.Network/virtualNetworks/subnets`.

Se admiten algunos _tipos de recursos_ no devueltos por **Get-AzResourceProvider**. Estos tipos son:

- `Microsoft.RecoveryServices/vaults/backupPolicies`

Los valores admitidos para **strongType** que no son _tipo de recurso_ son:

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`

## <a name="definition-location"></a>Ubicación de definición

Al crear una iniciativa o directiva, es necesario especificar la ubicación de la definición. La ubicación de la definición puede especificarse como un grupo de administración o una suscripción. La ubicación determina el ámbito al que pueden asignarse la directiva o la iniciativa. Los recursos deben ser miembros directos o elementos secundarios dentro de la jerarquía de la ubicación de la definición para que puedan ser destino de asignación.

Si la ubicación de la definición es:

- **Suscripción**: la definición de la directiva solo se puede asignar a los recursos incluidos dentro de esa suscripción.
- **Grupo de administración**: la definición de la directiva solo se puede asignar a los recursos incluidos dentro de grupos de administración secundarios y suscripciones secundarias. Si planea aplicar la definición de directiva a varias suscripciones, la ubicación debe ser un grupo de administración que contenga cada una de las suscripciones.

Para obtener más información, vea [Descripción del ámbito de Azure Policy](./scope.md#definition-location).

## <a name="policy-rule"></a>Regla de directiva

La regla de directiva se compone de los bloques **If** y **Then**. En el bloque **If**, defina una o varias condiciones que especifican cuándo se aplica la directiva. Puede aplicar operadores lógicos a estas condiciones para definir con precisión el escenario de una directiva.

En el bloque **Then**, defina el efecto que se produce cuando se cumplen las condiciones de **If**.

```json
{
    "if": {
        <condition> | <logical operator>
    },
    "then": {
        "effect": "deny | audit | modify | append | auditIfNotExists | deployIfNotExists | disabled"
    }
}
```

### <a name="logical-operators"></a>Operadores lógicos

Los operadores lógicos admitidos son:

- `"not": {condition  or operator}`
- `"allOf": [{condition or operator},{condition or operator}]`
- `"anyOf": [{condition or operator},{condition or operator}]`

La sintaxis **not** invierte el resultado de la condición. La sintaxis **allOf** (similar a la operación lógica **And**) requiere que se cumplan todas las condiciones. La sintaxis **anyOf** (similar a la operación lógica **Or**) requiere que se cumplan una o varias condiciones.

Puede anidar los operadores lógicos. El ejemplo siguiente muestra una operación **not** que está anidada dentro de una operación **allOf**.

```json
"if": {
    "allOf": [{
            "not": {
                "field": "tags",
                "containsKey": "application"
            }
        },
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        }
    ]
},
```

### <a name="conditions"></a>Condiciones

Una condición evalúa si un campo cumple determinados criterios. Estas son las condiciones que se admiten:

- `"equals": "stringValue"`
- `"notEquals": "stringValue"`
- `"like": "stringValue"`
- `"notLike": "stringValue"`
- `"match": "stringValue"`
- `"matchInsensitively": "stringValue"`
- `"notMatch": "stringValue"`
- `"notMatchInsensitively": "stringValue"`
- `"contains": "stringValue"`
- `"notContains": "stringValue"`
- `"in": ["stringValue1","stringValue2"]`
- `"notIn": ["stringValue1","stringValue2"]`
- `"containsKey": "keyName"`
- `"notContainsKey": "keyName"`
- `"less": "dateValue"` | `"less": "stringValue"` | `"less": intValue`
- `"lessOrEquals": "dateValue"` | `"lessOrEquals": "stringValue"` | `"lessOrEquals": intValue`
- `"greater": "dateValue"` | `"greater": "stringValue"` | `"greater": intValue`
- `"greaterOrEquals": "dateValue"` | `"greaterOrEquals": "stringValue"` |
  `"greaterOrEquals": intValue`
- `"exists": "bool"`

Para **less**, **lessOrEquals**, **greater** y **greaterOrEquals**, si el tipo de propiedad no coincide con el tipo de condición, se produce un error. La comparación de cadenas se realiza con `InvariantCultureIgnoreCase`.

Cuando se usan las condiciones **like** y **notLike**, incluya un carácter comodín (`*`) en el valor. El valor no debe contener más de un carácter comodín `*`.

Cuando se usan las condiciones **match** y **notMatch**, proporcione `#` para que coincida un dígito, `?` para una letra, `.` para que coincida cualquier carácter y cualquier otro carácter para que coincida ese carácter en sí. Mientras que **match** y **notMatch** distinguen mayúsculas de minúsculas, el resto de las condiciones que evalúan un elemento _stringValue_ no lo hacen. Las alternativas de distinción entre mayúsculas y minúsculas están disponibles en **matchInsensitively** y **notMatchInsensitively**.

### <a name="fields"></a>Campos

Las condiciones que evalúan si los valores de las propiedades de la carga de solicitudes de recursos cumplen determinados criterios se pueden formar usando una expresión **field**. Se admiten los siguientes campos:

- `name`
- `fullName`
  - Devuelve el nombre completo del recurso. El nombre completo de un recurso es el nombre del recurso precedido por los nombres de recurso principal (por ejemplo "myServer/myDatabase").
- `kind`
- `type`
- `location`
  - Los campos de ubicación se normalizan para admitir varios formatos. Por ejemplo, `East US 2` se considera igual que `eastus2`.
  - Use **global** para los recursos que son independientes de la ubicación.
- `id`
  - Devuelve el id. del recurso que se está evaluando.
  - Ejemplo: `/subscriptions/06be863d-0996-4d56-be22-384767287aa2/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault`
- `identity.type`
  - Devuelve el tipo de [identidad administrada](../../../active-directory/managed-identities-azure-resources/overview.md) habilitado en el recurso.
- `tags`
- `tags['<tagName>']`
  - Esta sintaxis con corchetes admite nombres de etiquetas que tienen signos de puntuación como guion, punto o espacio.
  - Donde **\<tagName\>** es el nombre de la etiqueta de la que se va a validar la condición.
  - Ejemplos: `tags['Acct.CostCenter']` donde **Acct.CostCenter** es el nombre de la etiqueta.
- `tags['''<tagName>''']`
  - Esta sintaxis con corchetes admite nombres de etiquetas con apóstrofos mediante secuencias de escape con dobles apóstrofos.
  - Donde **'\<tagName\>'** es el nombre de la etiqueta de la que se va a validar la condición.
  - Ejemplo: `tags['''My.Apostrophe.Tag''']` donde **'My.Apostrophe.Tag'** es el nombre de la etiqueta.
- alias de propiedad: para obtener una lista, vea [Alias](#aliases).

> [!NOTE]
> `tags.<tagName>`, `tags[tagName]` y `tags[tag.with.dots]` son todavía formas aceptables de declarar un campo de etiquetas. Sin embargo, las expresiones preferidas son las mencionadas anteriormente.

> [!NOTE]
> En las expresiones **field** que hacen referencia a **\[\*\] alias**, cada elemento de la matriz se evalúa de forma individual con el operador **and** lógico entre elementos. Para obtener más información, consulte [Referencia a las propiedades de recursos de matriz](../how-to/author-policies-for-arrays.md#referencing-array-resource-properties).

#### <a name="use-tags-with-parameters"></a>Uso de etiquetas con parámetros

Un valor de parámetro se puede pasar a un campo de etiqueta. Al pasar un parámetro a un campo de etiqueta aumenta la flexibilidad de la definición de directiva durante la asignación de directivas.

En el ejemplo siguiente, `concat` se usa para crear una búsqueda de campos de etiquetas para la etiqueta denominada con el valor del parámetro **tagName**. Si esa etiqueta no existe, se usa el efecto **modify** para agregarla con el valor de la misma etiqueta con nombre establecida en el grupo de recursos principal de los recursos auditados mediante la función de búsqueda `resourcegroup()`.

```json
{
    "if": {
        "field": "[concat('tags[', parameters('tagName'), ']')]",
        "exists": "false"
    },
    "then": {
        "effect": "modify",
        "details": {
            "operations": [{
                "operation": "add",
                "field": "[concat('tags[', parameters('tagName'), ']')]",
                "value": "[resourcegroup().tags[parameters('tagName')]]"
            }],
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/4a9ae827-6dc8-4573-8ac7-8239d42aa03f"
            ]
        }
    }
}
```

### <a name="value"></a>Value

Las condiciones que evalúan si un valor cumple determinados criterios se pueden formar utilizando una expresión **value**. Los valores pueden ser literales, los valores de [parámetros](#parameters) o los valores devueltos de cualquier [función de plantilla compatible](#policy-functions).

> [!WARNING]
> Si el resultado de una _función de plantilla_ es un error, no se pude realizar la evaluación de directivas. Una evaluación con errores es una **denegación** implícita. Para más información, consulte cómo [evitar los errores de plantilla](#avoiding-template-failures). Use [enforcementMode](./assignment-structure.md#enforcement-mode) de **DoNotEnforce** para evitar el efecto de una evaluación con errores en los recursos nuevos o actualizados mientras se prueba y valida una nueva definición de directiva.

#### <a name="value-examples"></a>Ejemplos de value

En este ejemplo de regla de directiva se utiliza **value** para comparar el resultado de la función `resourceGroup()` y la propiedad **name** devuelta con una condición  **like** de `*netrg`. La regla niega cualquier recurso que no sea de **tipo** `Microsoft.Network/*` en cualquier grupo de recursos cuyo nombre termine en `*netrg`.

```json
{
    "if": {
        "allOf": [{
                "value": "[resourceGroup().name]",
                "like": "*netrg"
            },
            {
                "field": "type",
                "notLike": "Microsoft.Network/*"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

En este ejemplo de regla de directiva se utiliza **value** para comprobar si el resultado de varias funciones anidadas es **igual a** `true`. La regla deniega cualquier recurso que no tenga al menos tres etiquetas.

```json
{
    "mode": "indexed",
    "policyRule": {
        "if": {
            "value": "[less(length(field('tags')), 3)]",
            "equals": "true"
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

#### <a name="avoiding-template-failures"></a>Evitar los errores de plantilla

El uso de _funciones de plantilla_ en el **valor** permite muchas funciones anidadas complejas. Si el resultado de una _función de plantilla_ es un error, no se pude realizar la evaluación de directivas. Una evaluación con errores es una **denegación** implícita. Un ejemplo de un **valor** que produce un error en ciertos escenarios:

```json
{
    "policyRule": {
        "if": {
            "value": "[substring(field('name'), 0, 3)]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

La regla de directiva del ejemplo anterior usa [substring()](../../../azure-resource-manager/templates/template-functions-string.md#substring) para comparar los tres primeros caracteres del **nombre** con **abc**. Si el **nombre** tiene menos de tres caracteres, la función `substring()` produce un error. Este error hace que la directiva tenga el efecto de **denegar**.

En su lugar, use la función [if()](../../../azure-resource-manager/templates/template-functions-logical.md#if) para comprobar si los tres primeros caracteres del **nombre** son igual a **abc** sin permitir que un **nombre** menor de tres caracteres produzca un error:

```json
{
    "policyRule": {
        "if": {
            "value": "[if(greaterOrEquals(length(field('name')), 3), substring(field('name'), 0, 3), 'not starting with abc')]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

Con la regla de directivas revisada, `if()` comprueba la longitud del **nombre** antes de intentar obtener un elemento `substring()` de un valor con menos de tres caracteres. Si el **nombre** es demasiado corto, se devuelve en su lugar el valor "no comienza por abc" en lugar de compararlo con **abc**. Un recurso con un nombre corto que no comienza por **abc** sigue produciendo un error en la regla de directivas, pero ya no se produce un error durante la evaluación.

### <a name="count"></a>Count

Las condiciones que cuentan el número de miembros de una matriz que cumplen determinados criterios se pueden formar usando una expresión **count**. Algunos escenarios comunes consisten en comprobar si "al menos uno de", "exactamente uno de", "todos" o "ninguno de" los miembros de la matriz satisfacen la condición. **count** evalúa una expresión de condición en cada miembro de la matriz y suma los resultados que son _true_ para luego compararlos con el operador de la expresión.

#### <a name="field-count"></a>Field count

Cuente el número de miembros de una matriz de la carga de solicitudes que satisfacen una expresión de condición. La estructura de la expresión **field count** es la siguiente:

```json
{
    "count": {
        "field": "<[*] alias>",
        "where": {
            /* condition expression */
        }
    },
    "<condition>": "<compare the count of true condition expression array members to this value>"
}
```

Las siguientes propiedades se utilizan con **field count**:

- **count.field** (requerido): Contiene la ruta de acceso a la matriz y debe ser un alias de matriz.
- **count.where** (opcional): la expresión de condición para evaluar de forma individual cada miembro de matriz de [\[\* alias\]](#understanding-the--alias) de `count.field`. Si no se proporciona esta propiedad, todos los miembros de la matriz con la ruta de acceso "field" se evalúan como _true_. En esta propiedad se puede usar cualquier [condición](../concepts/definition-structure.md#conditions).
  Los [operadores lógicos](#logical-operators) pueden usarse dentro de esta propiedad para crear requisitos de evaluación complejos.
- **\<condition\>** (obligatorio): El valor se compara con el número de elementos que cumplieron la expresión de condición **count.where**. Se debe usar una condición [numérica](../concepts/definition-structure.md#conditions).

Las expresiones **field count** pueden enumerar la misma matriz de campos hasta tres veces en una sola definición de **policyRule**.

Para más información sobre cómo trabajar con propiedades de matrices en Azure Policy, incluida una explicación detallada sobre cómo se evalúa la expresión **field count**, consulte [Referencia a las propiedades de recursos de matriz](../how-to/author-policies-for-arrays.md#referencing-array-resource-properties).

#### <a name="value-count"></a>Value count

Cuente el número de miembros de una matriz que satisfacen una condición. La matriz puede ser una matriz de literales o una [referencia a un parámetro de matriz](#using-a-parameter-value). La estructura de la expresión **value count** es la siguiente:

```json
{
    "count": {
        "value": "<literal array | array parameter reference>",
        "name": "<index name>",
        "where": {
            /* condition expression */
        }
    },
    "<condition>": "<compare the count of true condition expression array members to this value>"
}
```

Las siguientes propiedades se utilizan con **value count**:

- **count.value** (requerido): la matriz que se va a evaluar.
- **count.name** (requerido): el nombre del índice, compuesto por letras y dígitos en inglés. Define un nombre para el valor del miembro de la matriz evaluado en la iteración actual. El nombre se usa para hacer referencia al valor actual dentro de la condición `count.where`. Opcional cuando la expresión **count** no está en un elemento secundario de otra expresión **count**. Cuando no se proporciona, el nombre del índice se establece implícitamente en `"default"`.
- **count.where** (opcional): la expresión de condición para evaluar de forma individual cada miembro de la matriz de `count.value`. Si no se proporciona esta propiedad, todos los miembros de la matriz se evalúan como _true_. En esta propiedad se puede usar cualquier [condición](../concepts/definition-structure.md#conditions). Los [operadores lógicos](#logical-operators) pueden usarse dentro de esta propiedad para crear requisitos de evaluación complejos. Se puede acceder al valor del miembro de la matriz enumerado actualmente mediante la llamada a la función [current](#the-current-function).
- **\<condition\>** (obligatorio): El valor se compara con el número de elementos que cumplieron la expresión de condición `count.where`. Se debe usar una condición [numérica](../concepts/definition-structure.md#conditions).

Se aplican los límites siguientes:
- Se pueden usar hasta 10 expresiones **value count** en una única definición de **policyRule**.
- Cada expresión **value count** puede realizar hasta 100 iteraciones. Este número incluye el número de iteraciones realizadas por cualquier expresión **value count** principal.

#### <a name="the-current-function"></a>Función current

La función `current()` solo está disponible dentro de la condición `count.where`. Esta función devuelve el valor del miembro de la matriz que está enumerado actualmente en la evaluación de la expresión **count**.

**Uso de value count**

- `current(<index name defined in count.name>)`. Por ejemplo: `current('arrayMember')`.
- `current()`. Solo se permite cuando la expresión **value count** no es un elemento secundario de otra expresión **count**. Devuelve el mismo valor que antes.

Si el valor devuelto por la llamada es un objeto, se admiten los descriptores de acceso de propiedad. Por ejemplo: `current('objectArrayMember').property`.

**Uso de field count**

- `current(<the array alias defined in count.field>)`. Por ejemplo, `current('Microsoft.Test/resource/enumeratedArray[*]')`.
- `current()`. Solo se permite cuando la expresión **field count** no es un elemento secundario de otra expresión **count**. Devuelve el mismo valor que antes.
- `current(<alias of a property of the array member>)`. Por ejemplo, `current('Microsoft.Test/resource/enumeratedArray[*].property')`.

#### <a name="field-count-examples"></a>Ejemplos de field count

Ejemplo 1: Comprobar si una matriz está vacía

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]"
    },
    "equals": 0
}
```

Ejemplo 2: Comprobar si solo un miembro de la matriz cumple la expresión de condición

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "My unique description"
        }
    },
    "equals": 1
}
```

Ejemplo 3: Comprobar si por lo menos un miembro de la matriz cumple la expresión de condición

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "My common description"
        }
    },
    "greaterOrEquals": 1
}
```

Ejemplo 4: Comprobar si todos los miembros de la matriz del objeto cumplen la expresión de condición

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "description"
        }
    },
    "equals": "[length(field('Microsoft.Network/networkSecurityGroups/securityRules[*]'))]"
}
```

Ejemplo 5: Comprobar si al menos un miembro de la matriz coincide con varias propiedades de la expresión de condición

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "allOf": [
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction",
                    "equals": "Inbound"
                },
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
                    "equals": "Allow"
                },
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange",
                    "equals": "3389"
                }
            ]
        }
    },
    "greater": 0
}
```

Ejemplo 6: Use la función `current()` dentro de las condiciones `where` para acceder al valor del miembro de la matriz actualmente enumerado en una función de plantilla. Esta condición comprueba si una red virtual contiene un prefijo de dirección que no está en el intervalo de CIDR 10.0.0.0/24.

```json
{
    "count": {
        "field": "Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]",
        "where": {
          "value": "[ipRangeContains('10.0.0.0/24', current('Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]'))]",
          "equals": false
        }
    },
    "greater": 0
}
```

Ejemplo 7: Use la función `field()` dentro de las condiciones `where` para acceder al valor del miembro de la matriz actualmente enumerado. Esta condición comprueba si una red virtual contiene un prefijo de dirección que no está en el intervalo de CIDR 10.0.0.0/24.

```json
{
    "count": {
        "field": "Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]",
        "where": {
          "value": "[ipRangeContains('10.0.0.0/24', first(field(('Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]')))]",
          "equals": false
        }
    },
    "greater": 0
}
```

#### <a name="value-count-examples"></a>Ejemplos de value count

Ejemplo 1: Compruebe si el nombre de un recurso coincide con alguno de los patrones de nombre especificados.

```json
{
    "count": {
        "value": [ "prefix1_*", "prefix2_*" ],
        "name": "pattern",
        "where": {
            "field": "name",
            "like": "[current('pattern')]"
        }
    },
    "greater": 0
}
```

Ejemplo 2: Compruebe si el nombre de un recurso coincide con alguno de los patrones de nombre especificados. La función `current()` no especifica un nombre de índice. El resultado es el mismo que el del ejemplo anterior.

```json
{
    "count": {
        "value": [ "prefix1_*", "prefix2_*" ],
        "where": {
            "field": "name",
            "like": "[current()]"
        }
    },
    "greater": 0
}
```

Ejemplo 3: Compruebe si el nombre de un recurso coincide con alguno de los patrones de nombre proporcionados por un parámetro de matriz.

```json
{
    "count": {
        "value": "[parameters('namePatterns')]",
        "name": "pattern",
        "where": {
            "field": "name",
            "like": "[current('pattern')]"
        }
    },
    "greater": 0
}
```

Ejemplo 4: Compruebe si alguno de los prefijos de dirección de red virtual no está en la lista de prefijos aprobados.

```json
{
    "count": {
        "field": "Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]",
        "where": {
            "count": {
                "value": "[parameters('approvedPrefixes')]",
                "name": "approvedPrefix",
                "where": {
                    "value": "[ipRangeContains(current('approvedPrefix'), current('Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]'))]",
                    "equals": true
                },
            },
            "equals": 0
        }
    },
    "greater": 0
}
```

Ejemplo 5: Compruebe que todas las reglas del grupo de seguridad de red reservadas están definidas en un grupo de seguridad de red. Las propiedades de las reglas de grupo de seguridad de red reservadas se definen en un parámetro de matriz que contiene objetos.

Valor del parámetro:

```json
[
    {
        "priority": 101,
        "access": "deny",
        "direction": "inbound",
        "destinationPortRange": 22
    },
    {
        "priority": 102,
        "access": "deny",
        "direction": "inbound",
        "destinationPortRange": 3389
    }
]
```

Directiva:
```json
{
    "count": {
        "value": "[parameters('reservedNsgRules')]",
        "name": "reservedNsgRule",
        "where": {
            "count": {
                "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
                "where": {
                    "allOf": [
                        {
                            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].priority",
                            "equals": "[current('reservedNsgRule').priority]"
                        },
                        {
                            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
                            "equals": "[current('reservedNsgRule').access]"
                        },
                        {
                            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction",
                            "equals": "[current('reservedNsgRule').direction]"
                        },
                        {
                            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange",
                            "equals": "[current('reservedNsgRule').destinationPortRange]"
                        }
                    ]
                }
            },
            "equals": 1
        }
    },
    "equals": "[length(parameters('reservedNsgRules'))]"
}
```

### <a name="effect"></a>Efecto

Azure Policy admite los siguientes tipos de efecto:

- **Append**: agrega el conjunto de campos definido a la solicitud.
- **Audit**: genera un evento de advertencia en el registro de actividad pero no genera un error en la solicitud.
- **AuditIfNotExists**: genera un evento de advertencia en el registro de actividad en caso de no existir un recurso relacionado.
- **Deny**: genera un evento en el registro de actividad y genera un error en la solicitud.
- **DeployIfNotExists**: implementa un recurso relacionado si todavía no existe.
- **Disabled**: no se evalúa el cumplimiento de la regla de directivas en los recursos.
- **Modify**: agrega, actualiza o quita las etiquetas definidas de un recurso o suscripción.
- **EnforceOPAConstraint** (en desuso): configura el controlador de admisiones Open Policy Agent con Gatekeeper v3 para clústeres de Kubernetes autoadministrados en Azure.
- **EnforceRegoPolicy** (en desuso): configura el controlador de admisiones Open Policy Agent con Gatekeeper v2 en Azure Kubernetes Service.

Para obtener información detallada sobre cada efecto, el orden de evaluación, las propiedades y algunos ejemplos, consulte [Descripción de los efectos de Azure Policy](effects.md).

### <a name="policy-functions"></a>Funciones de directiva

Se pueden usar todas las [funciones de plantilla de Resource Manager](../../../azure-resource-manager/templates/template-functions.md) dentro de una regla de directivas, excepto las siguientes funciones y funciones definidas por el usuario:

- copyIndex()
- deployment()
- lista*
- newGuid()
- pickZones()
- providers()
- reference()
- resourceId()
- variables()

> [!NOTE]
> Estas funciones siguen estando disponibles en la parte `details.deployment.properties.template` de la implementación de la plantilla en una definición de directiva **deployIfNotExists**.

La siguiente función está disponible para su uso en una regla de directivas, pero difieren del uso en una plantilla de Azure Resource Manager (Plantilla de ARM):

- `utcNow()`: a diferencia de una plantilla de ARM, esta propiedad se puede usar con un valor distinto a _defaultValue_.
  - Devuelve una cadena que se establece en la fecha y hora actuales en formato DateTime universal ISO 8601 `yyyy-MM-ddTHH:mm:ss.fffffffZ`.

Las siguientes funciones solo están disponibles en las reglas de directiva:

- `addDays(dateTime, numberOfDaysToAdd)`
  - **dateTime** (fecha y hora): [Obligatorio] cadena: cadena con el formato de fecha y hora universal ISO 8601 "yyyy-MM-ddTHH:mm:ss.FFFFFFFZ"
  - **numberOfDaysToAdd** (número de días para agregar): [Obligatorio] entero: número de días que se desea agregar

- `field(fieldName)`
  - **fieldName**: [obligatorio] cadena. Es el nombre del [campo](#fields) que se va a recuperar.
  - Devuelve el valor de ese campo del recurso que se va a evaluar con la condición If.
  - `field` se usa principalmente con **AuditIfNotExists** y **DeployIfNotExists** para hacer referencia a los campos del recurso que se van a evaluar. Este uso se puede observar en el [ejemplo de DeployIfNotExists](effects.md#deployifnotexists-example).

- `requestContext().apiVersion`
  - Devuelve la versión de la API de la solicitud que desencadenó la evaluación de la directiva (por ejemplo: `2019-09-01`).
    Este valor es la versión de API que se usó en la solicitud PUT/PATCH para las evaluaciones de la creación o actualización de recursos. La versión más reciente de la API siempre se usa durante la evaluación del cumplimiento de los recursos existentes.

- `policy()`
  - Devuelve la siguiente información acerca de la directiva que se está evaluando. Es posible acceder a las propiedades desde el objeto devuelto (ejemplo: `[policy().assignmentId]`).
  
    ```json
    {
      "assignmentId": "/subscriptions/ad404ddd-36a5-4ea8-b3e3-681e77487a63/providers/Microsoft.Authorization/policyAssignments/myAssignment",
      "definitionId": "/providers/Microsoft.Authorization/policyDefinitions/34c877ad-507e-4c82-993e-3452a6e0ad3c",
      "setDefinitionId": "/providers/Microsoft.Authorization/policySetDefinitions/42a694ed-f65e-42b2-aa9e-8052e9740a92",
      "definitionReferenceId": "StorageAccountNetworkACLs"
    }
    ```

- `ipRangeContains(range, targetRange)`
  - **range**: [requerido] cadena: cadena que especifica un intervalo de direcciones IP para comprobar si _targetRange_ está dentro.
  - **targetRange**: [requerido] cadena: cadena que especifica un intervalo de direcciones IP que se van a validar como incluidas dentro del _intervalo_.
  - Devuelve un _valor booleano_ para determinar si el intervalo de direcciones IP del _intervalo_ contiene el intervalo de direcciones IP _targetRange_. No se permiten rangos vacíos ni mezclas entre familias de direcciones IP, lo que genera un error en la evaluación.

  Formatos compatibles:
  - Dirección IP única (ejemplos: `10.0.0.0`, `2001:0DB8::3:FFFE`)
  - Intervalo de CIDR (ejemplos: `10.0.0.0/24`, `2001:0DB8::/110`)
  - Intervalo definido por direcciones IP de inicio y final (ejemplos: `192.168.0.1-192.168.0.9`, `2001:0DB8::-2001:0DB8::3:FFFF`)

- `current(indexName)`
  - Función especial que solo se puede usar dentro de [expresiones count](#count).

#### <a name="policy-function-example"></a>Ejemplo de función de directiva

Este ejemplo de regla de directiva usa la función de recurso `resourceGroup` para obtener la propiedad **nombre**, combinada con la matriz `concat` y la función de objeto para compilar una condición `like` que exige que el nombre del recurso empiece con el nombre del grupo de recursos.

```json
{
    "if": {
        "not": {
            "field": "name",
            "like": "[concat(resourceGroup().name,'*')]"
        }
    },
    "then": {
        "effect": "deny"
    }
}
```

## <a name="aliases"></a>Alias

Los alias de propiedad se usan para tener acceso a propiedades específicas de un tipo de recurso. Los alias le permiten restringir los valores o condiciones que se permiten para una propiedad en un recurso. Cada alias se asigna a las rutas de acceso en las diferentes versiones de la API para un tipo de recurso determinado. Durante la evaluación de directivas, el motor de directiva obtiene la ruta de acceso de propiedad para esa versión de la API.

La lista de alias siempre está en aumento. Para descubrir qué alias son compatibles actualmente con Azure Policy, use uno de los métodos siguientes:

- Extensión de Azure Policy para Visual Studio Code (recomendado)

  Use la [extensión de Azure Policy para Visual Studio Code](../how-to/extension-for-vscode.md) a fin de ver y detectar alias para las propiedades de recursos.

  :::image type="content" source="../media/extension-for-vscode/extension-hover-shows-property-alias.png" alt-text="Captura de pantalla de la extensión Azure Policy para Visual Studio Code al mantener el puntero sobre una propiedad para mostrar los nombres de alias." border="false":::

- Azure PowerShell

  ```azurepowershell-interactive
  # Login first with Connect-AzAccount if not using Cloud Shell

  # Use Get-AzPolicyAlias to list available providers
  Get-AzPolicyAlias -ListAvailable

  # Use Get-AzPolicyAlias to list aliases for a Namespace (such as Azure Compute -- Microsoft.Compute)
  (Get-AzPolicyAlias -NamespaceMatch 'compute').Aliases
  ```

  > [!NOTE]
  > Para buscar alias que se puedan usar con el efecto [modificar](./effects.md#modify), use el siguiente comando en Azure PowerShell **4.6.0** o superior:
  >
  > ```azurepowershell-interactive
  > Get-AzPolicyAlias | Select-Object -ExpandProperty 'Aliases' | Where-Object { $_.DefaultMetadata.Attributes -eq 'Modifiable' }
  > ```

- Azure CLI

  ```azurecli-interactive
  # Login first with az login if not using Cloud Shell

  # List namespaces
  az provider list --query [*].namespace

  # Get Azure Policy aliases for a specific Namespace (such as Azure Compute -- Microsoft.Compute)
  az provider show --namespace Microsoft.Compute --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
  ```

- API de REST/ARMClient

  ```http
  GET https://management.azure.com/providers/?api-version=2019-10-01&$expand=resourceTypes/aliases
  ```

### <a name="understanding-the--alias"></a>Descripción del alias [*]

Algunos de los alias disponibles tienen una versión que aparece como un nombre "normal" y otra que tiene agregado **\[\*\]** . Por ejemplo:

- `Microsoft.Storage/storageAccounts/networkAcls.ipRules`
- `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]`

El alias "normal" representa el campo como un valor único. Este campo se utiliza en escenarios de comparación de coincidencia exacta, cuando todo el conjunto de valores debe ser exactamente como se define, nada más y nada menos.

El alias **\[\*\]** representa una colección de valores seleccionados de los elementos de una propiedad de recurso de matriz. Por ejemplo:

| Alias | Valores seleccionados |
|:---|:---|
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | Los elementos de la matriz `ipRules`. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | Los valores de la propiedad `action` de cada elemento de la matriz `ipRules`. |

Cuando se usan en una condición de [campo](#fields), los alias de matriz permiten comparar cada elemento individual de la matriz con un valor de destino. Cuando se usan con la expresión [count](#count), se pueden realizar las siguientes comprobaciones:

- Comprobar el tamaño de una matriz
- Comprobar si todos, alguno o ninguno de los elementos de la matriz cumple una condición compleja
- Comprobar si exactamente ***n*** elementos de la matriz cumplen una condición compleja

Para obtener más información y acceder a ejemplos, consulte [Referencia a las propiedades de recursos de matriz](../how-to/author-policies-for-arrays.md#referencing-array-resource-properties).

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [estructura de definición de la iniciativa](./initiative-definition-structure.md).
- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Vea la [Descripción de los efectos de directivas](effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
