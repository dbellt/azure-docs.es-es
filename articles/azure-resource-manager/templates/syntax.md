---
title: Estructura y sintaxis de plantillas
description: Describe la estructura y las propiedades de plantillas de Azure Resource Manager (plantillas de ARM) mediante la sintaxis declarativa de JSON.
ms.topic: conceptual
ms.date: 05/17/2021
ms.openlocfilehash: 9a1ead39ed680921f444068e8e52136247272ac5
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959915"
---
# <a name="understand-the-structure-and-syntax-of-arm-templates"></a>Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager

En este artículo se describe la estructura de una plantilla de Azure Resource Manager (plantilla de ARM). Presenta las distintas secciones de una plantilla y las propiedades que están disponibles en esas secciones.

Este artículo está destinado a los usuarios que ya estén familiarizados con las plantillas de Azure Resource Manager. Proporciona información detallada sobre la estructura de la plantilla. Para obtener un tutorial paso a paso que le guíe en el proceso de creación de una plantilla, consulte [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](template-tutorial-create-first-template.md). Para una guía en módulos con información sobre las plantillas de ARM en Microsoft Learn, consulte [Implementación y administración de recursos en Azure mediante plantillas de ARM](/learn/paths/deploy-manage-resource-manager-templates/).

## <a name="template-format"></a>Formato de plantilla

En la estructura más simple, una plantilla tiene los siguientes elementos:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "",
  "apiProfile": "",
  "parameters": {  },
  "variables": {  },
  "functions": [  ],
  "resources": [  ],
  "outputs": {  }
}
```

| Nombre del elemento | Obligatorio | Descripción |
|:--- |:--- |:--- |
| $schema |Sí |Ubicación del archivo de esquema de notación de objetos JavaScript (JSON) que describe la versión del lenguaje de plantilla. El número de versión que use dependerá del ámbito de la implementación y del editor de JSON.<br><br>Si usa [Visual Studio Code con la extensión de herramientas de Azure Resource Manager](quickstart-create-templates-use-visual-studio-code.md), utilice la versión más reciente de las implementaciones del grupo de recursos:<br>`https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#`<br><br>Es posible que otros editores (incluido Visual Studio) no puedan procesar este esquema. Para esos editores, use:<br>`https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#`<br><br>Para implementaciones de suscripciones, use:<br>`https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#`<br><br>Para implementaciones del grupo de administración, use:<br>`https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#`<br><br>Para implementaciones de inquilino, use:<br>`https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#` |
| contentVersion |Sí |Versión de la plantilla (por ejemplo, 1.0.0.0). Puede especificar cualquier valor para este elemento. Use este valor para documentar los cambios importantes de la plantilla. Al implementar los recursos con la plantilla, este valor se puede usar para asegurarse de que se está usando la plantilla correcta. |
| apiProfile |No | Una versión de API que actúa como una colección de versiones de API para los tipos de recursos. Use este valor para evitar tener que especificar las versiones de API para cada recurso de la plantilla. Si especifica una versión del perfil de API y no especifica una versión de API para el tipo de recurso, Resource Manager usa la versión de API para el tipo de recurso que está definido en el perfil.<br><br>La propiedad del perfil de API es especialmente útil al implementar una plantilla en diferentes entornos, como Azure Stack y Azure global. Use la versión del perfil de API para asegurarse de que la plantilla utilice automáticamente las versiones que se admiten en ambos entornos. Para obtener una lista de las versiones del perfil de API actuales y de las versiones de API de recursos definidas en el perfil, consulte [API Profile](https://github.com/Azure/azure-rest-api-specs/tree/master/profile) (Perfil de API).<br><br>Para más información, consulte [Seguimiento de versiones mediante perfiles de API](./template-cloud-consistency.md#track-versions-using-api-profiles). |
| [parameters](#parameters) |No |Valores que se proporcionan cuando se ejecuta la implementación para personalizar la implementación de recursos. |
| [variables](#variables) |No |Valores que se usan como fragmentos JSON en la plantilla para simplificar expresiones de idioma de la plantilla. |
| [functions](#functions) |No |Funciones definidas por el usuario que están disponibles dentro de la plantilla. |
| [resources](#resources) |Sí |Tipos de servicios que se implementan o actualizan en un grupo de recursos o suscripción. |
| [outputs](#outputs) |No |Valores que se devuelven después de la implementación. |

Cada elemento tiene propiedades que puede configurar. En este artículo se describen las secciones de la plantilla con más detalle.

## <a name="parameters"></a>Parámetros

En la sección `parameters` de la plantilla, especifique los valores que el usuario puede introducir al implementar los recursos. Está limitado a 256 parámetros en una plantilla. Puede reducir el número de parámetros mediante el uso de objetos que contienen varias propiedades.

Las propiedades disponibles para un parámetro son:

```json
"parameters": {
  "<parameter-name>" : {
    "type" : "<type-of-parameter-value>",
    "defaultValue": "<default-value-of-parameter>",
    "allowedValues": [ "<array-of-allowed-values>" ],
    "minValue": <minimum-value-for-int>,
    "maxValue": <maximum-value-for-int>,
    "minLength": <minimum-length-for-string-or-array>,
    "maxLength": <maximum-length-for-string-or-array-parameters>,
    "metadata": {
      "description": "<description-of-the parameter>"
    }
  }
}
```

| Nombre del elemento | Obligatorio | Descripción |
|:--- |:--- |:--- |
| nombre-del-parámetro |Sí |Nombre del parámetro. Debe ser un identificador válido de JavaScript. |
| type |Sí |Tipo del valor del parámetro. Los tipos y valores permitidos son **string**, **secureString**, **int**, **bool**, **objet**, **secureObject** y **array**. Consulte [Tipos de datos en plantillas de ARM](data-types.md). |
| defaultValue |No |Valor predeterminado del parámetro, si no se proporciona ningún valor. |
| allowedValues |No |Matriz de valores permitidos para el parámetro para asegurarse de que se proporciona el valor correcto. |
| minValue |No |El valor mínimo de parámetros de tipo int, este valor es inclusivo. |
| maxValue |No |El valor máximo de parámetros de tipo int, este valor es inclusivo. |
| minLength |No |Longitud mínima de los parámetros de tipo cadena, cadena segura y matriz; este valor es inclusivo. |
| maxLength |No |Longitud máxima de los parámetros de tipo cadena, cadena segura y matriz; este valor es inclusivo. |
| description |No |Descripción del parámetro que se muestra a los usuarios a través del portal. Para más información, consulte [Comentarios en plantillas](#comments). |

Para ejemplos de cómo usar los parámetros, ve [Parámetros en plantillas de ARM](./parameters.md).

## <a name="variables"></a>variables

En la sección `variables`, se crean valores que pueden usarse en toda la plantilla. No es necesario definir las variables, pero a menudo simplifican la plantilla reduciendo expresiones complejas. El formato de cada variable coincide con uno de los [tipos de datos](data-types.md).

En el ejemplo siguiente se muestran las opciones disponibles para definir una variable:

```json
"variables": {
  "<variable-name>": "<variable-value>",
  "<variable-name>": {
    <variable-complex-type-value>
  },
  "<variable-object-name>": {
    "copy": [
      {
        "name": "<name-of-array-property>",
        "count": <number-of-iterations>,
        "input": <object-or-value-to-repeat>
      }
    ]
  },
  "copy": [
    {
      "name": "<variable-array-name>",
      "count": <number-of-iterations>,
      "input": <object-or-value-to-repeat>
    }
  ]
}
```

Si desea información sobre el uso de `copy` para crear varios valores para una variable, consulte [Iteración de variables](copy-variables.md).

Para ejemplos de cómo usar las variables, vea [Variables en plantillas de ARM](./variables.md).

## <a name="functions"></a>Functions

Dentro de la plantilla, puede crear sus propias funciones. Estas funciones están disponibles para su uso en la plantilla. Normalmente, definirá expresiones complejas que no desea repetir en toda la plantilla. Creará las funciones definidas por el usuario a partir de las expresiones y [funciones](template-functions.md) que se admiten en las plantillas.

Al definir una función de usuario, hay algunas restricciones:

* La función no puede acceder a las variables.
* La función solo puede usar los parámetros que se definen en la función. Cuando usa la función [parameters](template-functions-deployment.md#parameters) dentro de una función definida por el usuario, los parámetros de esa función serán los que limiten sus acciones.
* La función no puede llamar a otras funciones definidas por el usuario.
* La función no puede usar la [función de referencia](template-functions-resource.md#reference).
* Los parámetros de la función no pueden tener valores predeterminados.

```json
"functions": [
  {
    "namespace": "<namespace-for-functions>",
    "members": {
      "<function-name>": {
        "parameters": [
          {
            "name": "<parameter-name>",
            "type": "<type-of-parameter-value>"
          }
        ],
        "output": {
          "type": "<type-of-output-value>",
          "value": "<function-return-value>"
        }
      }
    }
  }
],
```

| Nombre del elemento | Obligatorio | Descripción |
|:--- |:--- |:--- |
| espacio de nombres |Sí |Espacio de nombres para las funciones personalizadas. Se usa para evitar conflictos de nomenclatura con funciones de plantilla. |
| nombre-de-la-función |Sí |Nombre de la función personalizada. Al llamar a la función, combine el nombre de la función con el espacio de nombres. Por ejemplo, para llamar a una función denominada `uniqueName` en el espacio de nombres contoso, use `"[contoso.uniqueName()]"`. |
| nombre-del-parámetro |No |Nombre del parámetro que se va a usar en la función personalizada. |
| valor-del-parámetro |No |Tipo del valor del parámetro. Los tipos y valores permitidos son **string**, **secureString**, **int**, **bool**, **objet**, **secureObject** y **array**. |
| tipo de salida |Sí |Tipo del valor de salida. Los valores de salida admiten los mismos tipos que los parámetros de entrada de función. |
| valor de salida |Sí |Expresión de lenguaje de plantilla que se evalúa y devuelve desde la función. |

Para ejemplos de cómo usar las funciones personalizadas, vea [Funciones definidas por el usuario de la plantilla de ARM](./user-defined-functions.md).

## <a name="resources"></a>Recursos

En la sección `resources`, se definen los recursos que se implementan o se actualizan.

Defina recursos con la siguiente estructura:

```json
"resources": [
  {
      "condition": "<true-to-deploy-this-resource>",
      "type": "<resource-provider-namespace/resource-type-name>",
      "apiVersion": "<api-version-of-resource>",
      "name": "<name-of-the-resource>",
      "comments": "<your-reference-notes>",
      "location": "<location-of-resource>",
      "dependsOn": [
          "<array-of-related-resource-names>"
      ],
      "tags": {
          "<tag-name1>": "<tag-value1>",
          "<tag-name2>": "<tag-value2>"
      },
      "sku": {
          "name": "<sku-name>",
          "tier": "<sku-tier>",
          "size": "<sku-size>",
          "family": "<sku-family>",
          "capacity": <sku-capacity>
      },
      "kind": "<type-of-resource>",
      "scope": "<target-scope-for-extension-resources>",
      "copy": {
          "name": "<name-of-copy-loop>",
          "count": <number-of-iterations>,
          "mode": "<serial-or-parallel>",
          "batchSize": <number-to-deploy-serially>
      },
      "plan": {
          "name": "<plan-name>",
          "promotionCode": "<plan-promotion-code>",
          "publisher": "<plan-publisher>",
          "product": "<plan-product>",
          "version": "<plan-version>"
      },
      "properties": {
          "<settings-for-the-resource>",
          "copy": [
              {
                  "name": ,
                  "count": ,
                  "input": {}
              }
          ]
      },
      "resources": [
          "<array-of-child-resources>"
      ]
  }
]
```

| Nombre del elemento | Obligatorio | Descripción |
|:--- |:--- |:--- |
| condición | No | Valor booleano que indica si el recurso se aprovisionará durante esta implementación. Si es `true`, el recurso se crea durante la implementación. Si es `false`, el recurso se omite para esta implementación. Consulte [condition](conditional-resource-deployment.md). |
| type |Sí |Tipo de recurso. Este valor es una combinación del espacio de nombres del proveedor de recursos y el tipo de recurso (por ejemplo, `Microsoft.Storage/storageAccounts`). Para determinar los valores disponibles, consulte la [referencia de plantilla](/azure/templates/). Para un recurso secundario, el formato del tipo depende de si está anidado dentro del recurso primario o se ha definido fuera del recurso primario. Consulte [Establecimiento del nombre y el tipo de recursos secundarios](child-resource-name-type.md). |
| apiVersion |Sí |Versión de la API de REST que debe usar para crear el recurso. Al crear una plantilla, establezca este valor en la última versión del recurso que va a implementar. Siempre que la plantilla funcione según sea necesario, siga usando la misma versión de la API. Al seguir usando la misma versión de la API, se minimiza el riesgo de que una nueva versión de la API cambie el funcionamiento de la plantilla. Considere la posibilidad de actualizar la versión de la API solo cuando desee usar una característica nueva que se presenta en una versión posterior. Para determinar los valores disponibles, consulte la [referencia de plantilla](/azure/templates/). |
| name |Sí |Nombre del recurso. El nombre debe cumplir las restricciones de componente URI definidas en RFC3986. Los servicios de Azure que exponen el nombre del recurso a partes externas validan el nombre para asegurarse de que no es un intento de suplantar otra identidad. Para un recurso secundario, el formato del nombre depende de si está anidado dentro del recurso primario o se ha definido fuera del recurso primario. Consulte [Establecimiento del nombre y el tipo de recursos secundarios](child-resource-name-type.md). |
| comments |No |Notas para documentar los recursos de la plantilla. Para más información, consulte [Comentarios en plantillas](#comments). |
| ubicación |Varía |Ubicaciones geográficas compatibles del recurso proporcionado. Puede seleccionar cualquiera de las ubicaciones disponibles, pero normalmente tiene sentido elegir aquella que esté más cerca de los usuarios. Normalmente, también tiene sentido colocar los recursos que interactúan entre sí en la misma región. La mayoría de los tipos de recursos requieren una ubicación, pero algunos (por ejemplo, una asignación de roles) no la necesitan. Consulte [Establecimiento de la ubicación del recurso](resource-location.md). |
| dependsOn |No |Recursos que se deben implementar antes de implementar este. Resource Manager evalúa las dependencias entre recursos y los implementa en su orden correcto. Cuando no hay recursos dependientes entre sí, se implementan en paralelo. El valor puede ser una lista separada por comas de nombres de recursos o identificadores de recursos únicos. Solo los recursos de lista que se implementan en esta plantilla. Deben existir los recursos que no estén definidos en esta plantilla. Evite agregar dependencias innecesarias, ya que pueden ralentizar la implementación y crear dependencias circulares. Para obtener instrucciones sobre cómo establecer dependencias, vea [Definición del orden de implementación de recursos en plantillas de Azure Resource Manager](./resource-dependency.md). |
| etiquetas |No |Etiquetas asociadas al recurso. Aplique etiquetas para organizar de forma lógica los recursos en su suscripción. |
| sku | No | Algunos recursos permiten valores que definen la SKU que se va a implementar. Por ejemplo, puede especificar el tipo de redundancia para una cuenta de almacenamiento. |
| kind | No | Algunos recursos permiten un valor que define el tipo de recurso que va a implementar. Por ejemplo, puede especificar el tipo de instancia de Cosmos DB que va a crear. |
| scope | No | La propiedad scope solo está disponible para los [tipos de recursos de extensión](../management/extension-resource-types.md). Úsela al especificar un ámbito diferente del ámbito de implementación. Consulte [Establecimiento del ámbito de los recursos de extensión en las plantillas de Resource Manager](scope-extension-resources.md). |
| copy |No |Si se necesita más de una instancia, el número de recursos que se crearán. El modo predeterminado es paralelo. Si no desea que todos los recursos se implementen al mismo tiempo, especifique el modo serie. Para obtener más información, consulte [Creación de varias instancias de recursos en Azure Resource Manager](copy-resources.md). |
| plan | No | Algunos recursos permiten valores que definen el plan que se va a implementar. Por ejemplo, puede especificar la imagen de Marketplace para una máquina virtual. |
| properties |No |Opciones de configuración específicas de recursos. Los valores de las propiedades son exactamente los mismos valores que se especifican en el cuerpo de la solicitud de la operación de API de REST (método PUT) para crear el recurso. También puede especificar una matriz de copia para crear varias instancias de una propiedad. Para determinar los valores disponibles, consulte la [referencia de plantilla](/azure/templates/). |
| resources |No |Recursos secundarios que dependen del recurso que se está definiendo. Proporcione solo tipos de recursos que permita el esquema del recurso principal. La dependencia del recurso principal no está implícita. Debe definirla explícitamente. Consulte [Establecimiento del nombre y el tipo de recursos secundarios](child-resource-name-type.md). |

## <a name="outputs"></a>Salidas

En la sección `outputs`, especifique los valores que se devuelven de la implementación. Normalmente, devuelve valores de los recursos implementados.

En el ejemplo siguiente se muestra la estructura de una definición de salida:

```json
"outputs": {
  "<output-name>": {
    "condition": "<boolean-value-whether-to-output-value>",
    "type": "<type-of-output-value>",
    "value": "<output-value-expression>",
    "copy": {
      "count": <number-of-iterations>,
      "input": <values-for-the-variable>
    }
  }
}
```

| Nombre del elemento | Obligatorio | Descripción |
|:--- |:--- |:--- |
| nombre de salida |Sí |Nombre del valor de salida. Debe ser un identificador válido de JavaScript. |
| condición |No | Valor booleano que indica si se va a devolver este valor de salida. Si es `true`, el valor se incluye en la salida de la implementación. Si es `false`, el recurso se omite en esta implementación. Si no se especifica, el valor predeterminado es `true`. |
| type |Sí |Tipo del valor de salida. Los valores de salida admiten los mismos tipos que los parámetros de entrada de plantilla. Si especifica **securestring** para el tipo de salida, el valor no se muestra en el historial de implementación y no se puede recuperar desde otra plantilla. Para usar un valor de secreto en más de una plantilla, almacene el secreto en un almacén de claves y haga referencia al secreto en el archivo de parámetros. Para más información, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](key-vault-parameter.md). |
| value |No |Expresión de lenguaje de plantilla que se evaluará y devolverá como valor de salida. Especifique **value** o **copy**. |
| copy |No | Se usa para devolver más de un valor para una salida. Especifique **value** o **copy**. Para más información, vea [Iteración de salida en plantillas de ARM](copy-outputs.md). |

Para ejemplos sobre cómo usar las salidas, vea [Salidas en una plantilla de ARM](./outputs.md).

<a id="comments"></a>

## <a name="comments-and-metadata"></a>Comentarios y metadatos

Tiene unas cuantas opciones para agregar comentarios y metadatos a la plantilla.

### <a name="comments"></a>Comentarios

Para los comentarios insertados, puede usar `//` o `/* ... */`.

> [!NOTE]
>
> Para implementar plantillas con comentarios, use Azure PowerShell o la CLI de Azure. Para la CLI, use la versión 2.3.0 o posterior y especifique el modificador `--handle-extended-json-format`.
>
> No se admiten comentarios al implementar la plantilla mediante Azure Portal, una canalización de DevOps o la API REST.

```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2018-10-01",
  "name": "[variables('vmName')]", // to customize name, change it in variables
  "location": "[parameters('location')]", //defaults to resource group location
  "dependsOn": [ /* storage account and network interface must be deployed first */
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
  ],
```

En Visual Studio Code, la [extensión de herramientas de Azure Resource Manager](quickstart-create-templates-use-visual-studio-code.md) puede detectar automáticamente una plantilla de ARM y cambiar el modo de lenguaje. Si ve **Plantilla de Azure Resource Manager** en la esquina inferior derecha de Visual Studio Code, puede usar los comentarios en línea. Los comentarios insertados ya no están marcados como no válidos.

![Modo de plantillas de Azure Resource Manager en Visual Studio Code](./media/template-syntax/resource-manager-template-editor-mode.png)

### <a name="metadata"></a>Metadatos

Puede agregar un objeto `metadata` prácticamente en cualquier parte de la plantilla. Resource Manager omite el objeto, pero el editor JSON puede advertirle de que la propiedad no es válida. En el objeto, defina las propiedades que necesita.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "comments": "This template was developed for demonstration purposes.",
    "author": "Example Name"
  },
```

Para `parameters`, agregue un objeto `metadata` con una propiedad `description`.

```json
"parameters": {
  "adminUsername": {
    "type": "string",
    "metadata": {
      "description": "User name for the Virtual Machine."
    }
  },
```

Al implementar la plantilla a través del portal, el texto que proporciona en la descripción se usa automáticamente como sugerencia para ese parámetro.

![Mostrar sugerencia de parámetros](./media/template-syntax/show-parameter-tip.png)

Para `resources`, agregue un elemento `comments` o un objeto `metadata`. En el ejemplo siguiente se muestra un elemento `comments` y un objeto `metadata`.

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2018-07-01",
    "name": "[concat('storage', uniqueString(resourceGroup().id))]",
    "comments": "Storage account used to store VM disks",
    "location": "[parameters('location')]",
    "metadata": {
      "comments": "These tags are needed for policy compliance."
    },
    "tags": {
      "Dept": "[parameters('deptName')]",
      "Environment": "[parameters('environment')]"
    },
    "sku": {
      "name": "Standard_LRS"
    },
    "kind": "Storage",
    "properties": {}
  }
]
```

Para `outputs`, agregue un objeto `metadata` al valor de salida.

```json
"outputs": {
  "hostname": {
    "type": "string",
    "value": "[reference(variables('publicIPAddressName')).dnsSettings.fqdn]",
    "metadata": {
      "comments": "Return the fully qualified domain name"
    }
  },
```

No se puede agregar un objeto `metadata` a las funciones definidas por el usuario.

## <a name="multi-line-strings"></a>Cadenas de varias líneas

Una cadena se puede dividir en varias líneas. Por ejemplo, consulte la propiedad `location` y uno de los comentarios del ejemplo de JSON siguiente.

> [!NOTE]
>
> Para implementar plantillas con cadenas multilínea, use Azure PowerShell o la CLI de Azure. Para la CLI, use la versión 2.3.0 o posterior y especifique el modificador `--handle-extended-json-format`.
>
> No se admiten cadenas multilínea al implementar la plantilla mediante Azure Portal, una canalización de DevOps o la API REST.


```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2018-10-01",
  "name": "[variables('vmName')]", // to customize name, change it in variables
  "location": "[
    parameters('location')
    ]", //defaults to resource group location
  /*
    storage account and network interface
    must be deployed first
  */
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
  ],
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver plantillas completas de muchos tipos diferentes de soluciones, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/).
* Para obtener información detallada sobre las funciones que se pueden usar dentro de una plantilla, vea [Funciones de plantilla de ARM](template-functions.md).
* Para combinar varias plantillas durante la implementación, vea [Uso de plantillas vinculadas y anidadas al implementar recursos de Azure](linked-templates.md).
* Para recomendaciones sobre la creación de platillas, consulte [Procedimientos recomendados de plantillas de ARM](./best-practices.md).
* Para obtener respuestas a preguntas comunes, consulte [Preguntas más frecuentes sobre las plantillas de ARM](frequently-asked-questions.md).