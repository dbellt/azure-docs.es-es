---
title: 'Creación de archivos de Bicep: Visual Studio Code'
description: Uso de Visual Studio Code y la extensión Bicep para archivos de Bicep a fin de implementar recursos de Azure
ms.date: 06/01/2021
ms.topic: quickstart
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8369d0185977ae85e561b809200f1e7d733ea556
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111027261"
---
# <a name="quickstart-create-bicep-files-with-visual-studio-code"></a>Inicio rápido: Creación de plantillas de archivos de Bicep con Visual Studio Code

Este inicio rápido le guiará por los pasos necesarios para crear un [archivo Bicep](overview.md) con Visual Studio Code. Creará una cuenta de almacenamiento y una red virtual. También aprenderá cómo la extensión Bicep simplifica el desarrollo al proporcionar seguridad de tipos, validación de sintaxis y autocompletado.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

A fin de configurar el entorno para el desarrollo de Bicep, consulte [Instalación de las herramientas de Bicep](install.md). Después de completar esos pasos, tendrá [Visual Studio Code](https://code.visualstudio.com/) y la [extensión de Bicep](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep). También tendrá la versión más reciente de la [CLI de Azure](/cli/azure/) o la versión más reciente del [módulo de Azure PowerShell](/powershell/azure/new-azureps-module-az).

## <a name="add-resource-snippet"></a>Adición del fragmento de código de un recurso

Inicie Visual Studio Code y cree un nuevo archivo denominado *main.bicep*.

VS Code con la extensión Bicep simplifica el desarrollo al proporcionar fragmentos de código predefinidos. En este inicio rápido, agregará un fragmento de código para crear una red virtual.

En el archivo *main.bicep*, escriba **vnet**. Seleccione **res-vnet** en la lista y, a continuación, presione TAB o Entrar.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/add-snippet.png" alt-text="Agregar un fragmento de código para la red virtual":::

El archivo Bicep ahora contiene el código siguiente:

```bicep
resource virtualNetwork 'Microsoft.Network/virtualNetworks@2019-11-01' = {
  name: 'name'
  location: resourceGroup().location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'Subnet-1'
        properties: {
          addressPrefix: '10.0.0.0/24'
        }
      }
      {
        name: 'Subnet-2'
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
    ]
  }
}
```

Este fragmento de código contiene todos los valores que necesita para definir la red virtual. Sin embargo, puede modificarlo para que cumpla sus requisitos. Por ejemplo, `name` no es un buen nombre para la red virtual. Cambie la propiedad `name` a `examplevnet`.

```bicep
name: 'examplevnet'
```

Puede implementar este archivo Bicep, pero agregaremos un parámetro y una cuenta de almacenamiento antes de la implementación.

## <a name="add-parameter"></a>Agregar parámetro

Ahora, agregaremos un parámetro que se usará para el nombre de la cuenta de almacenamiento. Agregue un par de líneas arriba de la red virtual y escriba:

```bicep
param storageName
```

Al agregar un espacio después de **storageName**, observe que IntelliSense muestra los tipos de datos disponibles para el parámetro.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/add-param.png" alt-text="Agregar el tipo cadena al parámetro":::

Seleccione **string**.

Ahora tiene el siguiente parámetro:

```bicep
param storageName string
```

Este parámetro funciona bien, pero las cuentas de almacenamiento tienen límites para la longitud del nombre. El nombre debe tener al menos 3 caracteres y no debe superar los 24 caracteres. Para especificar esos requisitos, puede agregar decoradores al parámetro.

Agregue una línea arriba del parámetro y escriba **@** . Se mostrarán los decoradores disponibles. Observe que hay decoradores para **minLength** y **maxLength**.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/add-decorators.png" alt-text="Agregar decoradores al parámetro":::

Agregue ambos decoradores y especifique los límites de caracteres, como se muestra a continuación:

```bicep
@minLength(3)
@maxLength(24)
param storageName string
```

También puede agregar una descripción para el parámetro. Incluya información para que las personas que implementarán el archivo Bicep comprendan el valor que se debe especificar.

```bicep
@minLength(3)
@maxLength(24)
@description('Provide a name for the storage account. Use only lower case letters and numbers. The name must be unique across Azure.')
param storageName string
```

Vale, el parámetro está listo para usarse.

## <a name="add-resource"></a>Agregar recurso

En lugar de usar un fragmento de código para definir la cuenta de almacenamiento, usaremos IntelliSense para configurar los valores. Con IntelliSense, este paso es mucho más fácil que cuando los valores se tienen que escribir manualmente.

Defina un recurso con la palabra clave `resource`.  Debajo de la red virtual, escriba **resource exampleStorage**:

```bicep
resource exampleStorage
```

**exampleStorage** es un nombre simbólico para el recurso que se va a implementar. Con él, se facilitará el uso de referencias al recurso en otras partes del archivo Bicep.

Al agregar un espacio después del nombre simbólico, se muestra una lista de tipos de recursos. Siga escribiendo **storage** hasta que pueda seleccionarlo entre las opciones disponibles.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/select-resource-type.png" alt-text="Seleccionar cuentas de almacenamiento para el tipo de recurso":::

Después de seleccionar **Microsoft.Storage/storageAccounts**, se muestran las versiones de API disponibles. Seleccione la versión más reciente.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/select-api-version.png" alt-text="Seleccionar la versión de API para el tipo de recurso":::

Después de la comilla simple del tipo de recurso, agregue **=** y un espacio. Se mostrarán las opciones para agregar propiedades al recurso. Seleccione **required-properties**.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/select-required-properties.png" alt-text="Agregar propiedades obligatorias":::

Esta opción agrega todas las propiedades del tipo de recurso que son obligatorias para la implementación. Después de seleccionar esta opción, la cuenta de almacenamiento tendrá las siguientes propiedades:

```bicep
resource exampleStorage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name:
  location:
  sku: {
    name:
  }
  kind:

}
```

Casi ha terminado. Solo tiene que proporcionar valores para esas propiedades.

De nuevo, IntelliSense le puede ayudar. En el caso de `name`, proporcione el parámetro que contiene un nombre para la cuenta de almacenamiento. En el caso de `location`, establezca el valor en `eastus`. Al agregar el nombre y el tipo de SKU, IntelliSense presentará las opciones válidas. 

Cuando haya terminado, tendrá lo siguiente:

```bicep
@minLength(3)
@maxLength(24)
param storageName string

resource virtualNetwork 'Microsoft.Network/virtualNetworks@2019-11-01' = {
  name: 'examplevnet'
  location: resourceGroup().location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'Subnet-1'
        properties: {
          addressPrefix: '10.0.0.0/24'
        }
      }
      {
        name: 'Subnet-2'
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
    ]
  }
}

resource exampleStorage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageName
  location: 'eastus'
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

Para obtener más información sobre la sintaxis de Bicep, consulte [Estructura de Bicep](./file.md).

## <a name="visualize-resources"></a>Visualización de recursos

Puede ver una representación de los recursos en el archivo.

En la esquina superior izquierda, seleccione el botón del visualizador para abrir el visualizador de Bicep.

:::image type="content" source="./media/quickstart-create-bicep-use-visual-studio-code/bicep-visualizer.png" alt-text="Visualizador de Bicep":::

El visualizador muestra los recursos definidos en el archivo Bicep con la información sobre las dependencias entre recursos. Los dos recursos que se definieron en este inicio rápido no tienen una relación de dependencia, por lo que no verá un conector entre ellos.

## <a name="deploy-the-bicep-file"></a>Implementación del archivo de Bicep

Para implementar el archivo que creó, abra PowerShell o la CLI de Azure. Si quiere usar el terminal integrado de Visual Studio Code, seleccione la combinación de teclas `ctrl` + ```` ` ````. Cambie el directorio actual a la ubicación actual del archivo Bicep.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group create --name exampleRG --location eastus

az deployment group create --resource-group exampleRG --template-file main.bicep --parameters storageName={your-unique-name}
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
New-AzResourceGroup -Name exampleRG -Location eastus

New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./main.bicep -storageName "{your-unique-name}"
```

---

Una vez finalizada la implementación, debería mostrarse un mensaje indicando que la implementación se realizó correctamente. Si recibe un mensaje de error indicando que la cuenta de almacenamiento ya está ocupada, significa que el nombre de almacenamiento proporcionado está en uso. Especifique un nombre con mayores probabilidades de ser único.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure dejen de ser necesarios, use la CLI de Azure o el módulo de Azure PowerShell para eliminar el grupo de recursos y el recurso del servidor del inicio rápido.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group delete --name exampleRG
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
Remove-AzResourceGroup -Name exampleRG
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Bicep en Microsoft Learn](learn-bicep.md)
