---
title: Creación de archivo de parámetros
description: Creación de un archivo de parámetros para pasar valores durante la implementación de una plantilla de Azure Resource Manager
ms.topic: conceptual
ms.date: 04/15/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 50404df278da22352344bbd12e139cc86f0a0615
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321652"
---
# <a name="create-resource-manager-parameter-file"></a>Creación de un archivo de parámetros de Resource Manager

En lugar de pasar parámetros como valores en línea en el script, puede usar un archivo JSON que contenga los valores de parámetro. En este artículo se muestra cómo crear un archivo de parámetros que se usa con una plantilla JSON o un archivo Bicep.

## <a name="parameter-file"></a>Archivo de parámetros

Un archivo de parámetros usa el siguiente formato:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "value": "<second-value>"
    }
  }
}
```

Tenga en cuenta que el archivo de parámetros almacena los valores de parámetro como texto sin formato. Este enfoque funciona para los valores que no son confidenciales, como la SKU de un recurso. El texto sin formato no funciona para valores confidenciales, como las contraseñas. Si necesita pasar un parámetro que contiene un valor confidencial, almacene el valor en un almacén de claves. A continuación, puede hacer referencia al almacén de claves en el archivo de parámetros. El valor confidencial se recupera de forma segura durante la implementación.

El archivo de parámetros siguiente incluye un valor de texto sin formato y un valor confidencial que está almacenado en un almacén de claves.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "reference": {
        "keyVault": {
          "id": "<resource-id-key-vault>"
        },
        "secretName": "<secret-name>"
      }
    }
  }
}
```

Para más información sobre el uso de valores de una instancia de Key Vault, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](key-vault-parameter.md).

## <a name="define-parameter-values"></a>Definición de los valores de parámetro

Para determinar cómo definir los nombres y valores de los parámetros, abra la plantilla JSON o Bicep. Examine la sección de parámetros de la plantilla. En los ejemplos siguientes se muestran los parámetros de las plantillas JSON y Bicep.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "storagePrefix": {
    "type": "string",
    "maxLength": 11
  },
  "storageAccountType": {
    "type": "string",
    "defaultValue": "Standard_LRS",
    "allowedValues": [
    "Standard_LRS",
    "Standard_GRS",
    "Standard_ZRS",
    "Premium_LRS"
    ]
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@maxLength(11)
param storagePrefix string

@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_ZRS'
  'Premium_LRS'
])
param storageAccountType string = 'Standard_LRS'
```

---

En el archivo de parámetros, el primer detalle que se debe tener en cuenta es el nombre de cada parámetro. Los nombres de los parámetros del archivo de parámetros deben coincidir con los nombres de los parámetros de la plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
    },
    "storageAccountType": {
    }
  }
}
```

Tenga en cuenta el tipo del parámetro. Los tipos de parámetros del archivo de parámetros deben usar los mismos tipos que la plantilla. En este ejemplo, ambos tipos de parámetros son cadenas.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": ""
    },
    "storageAccountType": {
      "value": ""
    }
  }
}
```

Compruebe en la plantilla los parámetros con un valor predeterminado. Si un parámetro tiene un valor predeterminado, puede proporcionar un valor en el archivo de parámetros, pero no es necesario. El valor del archivo de parámetros invalida el valor predeterminado de la plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "" // This value must be provided.
    },
    "storageAccountType": {
      "value": "" // This value is optional. Template will use default value if not provided.
    }
  }
}
```

Compruebe los valores permitidos y las restricciones de la plantilla, como la longitud máxima. Esos valores especifican el intervalo de valores que puede proporcionar para un parámetro. En este ejemplo, `storagePrefix` puede tener un máximo de 11 caracteres y `storageAccountType` debe especificar un valor permitido.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "storage"
    },
    "storageAccountType": {
      "value": "Standard_ZRS"
    }
  }
}
```

> [!NOTE]
> El archivo de parámetros solo puede contener valores para los parámetros que se definan en la plantilla. Si el archivo de parámetros contiene parámetros adicionales que no coinciden con los de la plantilla, recibirá un error.

## <a name="parameter-type-formats"></a>Formatos de tipo de parámetro

En el ejemplo siguiente se muestran los formatos de distintos tipos de parámetros: cadena, entero, booleano, matriz y objeto.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "exampleString": {
      "value": "test string"
    },
    "exampleInt": {
      "value": 4
    },
    "exampleBool": {
      "value": true
    },
    "exampleArray": {
      "value": [
        "value 1",
        "value 2"
      ]
    },
    "exampleObject": {
      "value": {
        "property1": "value1",
        "property2": "value2"
      }
    }
  }
}
```

## <a name="deploy-template-with-parameter-file"></a>Implementación de la plantilla con el archivo de parámetros

Desde la CLI de Azure se pasa un archivo de parámetros local mediante `@` y el nombre del archivo de parámetros. Por ejemplo, `@storage.parameters.json`.

```azurecli
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

Para más información, consulte [Implementación de recursos con plantillas de ARM y la CLI de Azure](./deploy-cli.md#parameters). Para implementar archivos _.bicep_, necesita la CLI de Azure versión 2.20 o posterior.

Desde Azure PowerShell se pasa un archivo de parámetros local mediante el parámetro `TemplateParameterFile`.

```azurepowershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile C:\MyTemplates\storage.json `
  -TemplateParameterFile C:\MyTemplates\storage.parameters.json
```

Para más información, consulte [Implementación de recursos con las plantillas de ARM y Azure PowerShell](./deploy-powershell.md#pass-parameter-values). Para implementar archivos _.bicep_, necesita Azure PowerShell versión 5.6.0 o posterior.

> [!NOTE]
> No es posible usar un archivo de parámetros con la hoja de plantilla personalizada en el portal.

> [!TIP]
> Si usa el [proyecto de grupo de recursos de Azure en Visual Studio](create-visual-studio-deployment-project.md), asegúrese de que el archivo de parámetros tiene su **acción de compilación** establecida en **Contenido**.

## <a name="file-name"></a>Nombre de archivo

La convención de nomenclatura general para el archivo de parámetros consiste en agregar _parameters_ al nombre de la plantilla. Por ejemplo, si la plantilla se denomina _azuredeploy.json_, el archivo de parámetros se denomina _azuredeploy.parameters.json_. Esta convención de nomenclatura le ayuda a ver la conexión entre la plantilla y los parámetros.

Para realizar la implementación en entornos diferentes, cree más de un archivo de parámetros. Cuando asigne un nombre a los archivos de parámetros, identifique su uso, desarrollo y producción. Por ejemplo, use _azuredeploy.parameters-dev.json_ y _azuredeploy.parameters-prod.json_ para implementar recursos.

## <a name="parameter-precedence"></a>Prioridad de parámetros

Puede usar parámetros en línea y un archivo de parámetros local en la misma operación de implementación. Por ejemplo, puede especificar algunos valores en el archivo de parámetros local y agregar otros valores en línea durante la implementación. Si proporciona valores para un parámetro en el archivo de parámetros local y en línea, el valor en línea tiene prioridad.

Es posible usar un archivo de parámetros externo proporcionando el URI al archivo. Cuando se utiliza un archivo de parámetros externo, no se pueden pasar otros valores, tanto si se hace en línea como desde un archivo local. Se omiten todos los parámetros insertados. Proporcione todos los valores de parámetro en el archivo externo.

## <a name="parameter-name-conflicts"></a>Conflictos de nombres de parámetro

Si la plantilla incluye un parámetro con el mismo nombre que uno de los parámetros del comando de PowerShell, PowerShell presenta el parámetro de la plantilla con el postfijo `FromTemplate`. Por ejemplo, un parámetro denominado `ResourceGroupName` de su plantilla entra en conflicto con el parámetro `ResourceGroupName` del cmdlet [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) nuevo. Se le pedirá que proporcione un valor para `ResourceGroupNameFromTemplate`. Para evitar esta confusión, use nombres de parámetros que no se usan para los comandos de implementación.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre cómo definir parámetros en una plantilla, vea [Parámetros en plantillas de ARM](template-parameters.md).
- Para más información sobre el uso de valores de una instancia de Key Vault, consulte [Uso de Azure Key Vault para pasar el valor de parámetro seguro durante la implementación](key-vault-parameter.md).
