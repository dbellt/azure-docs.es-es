---
title: 'Creación de archivos de Bicep: Visual Studio Code'
description: Uso de Visual Studio Code y la extensión Bicep para archivos de Bicep a fin de implementar recursos de Azure
author: mumian
ms.date: 04/12/2021
ms.topic: quickstart
ms.author: jgao
ms.openlocfilehash: e82f047f553290c9121d91060c347c5b242c7ec2
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107306473"
---
# <a name="quickstart-create-bicep-files-with-visual-studio-code"></a>Inicio rápido: Creación de plantillas de archivos de Bicep con Visual Studio Code

La extensión Bicep para Visual Studio Code proporciona compatibilidad con el lenguaje y la finalización automática de recursos. Estas herramientas facilitan la creación y validación de archivos de [Bicep](./bicep-overview.md). En este inicio rápido usará la extensión para crear un archivo de Bicep desde cero. En el proceso, experimentará funciones de las extensiones como la validación y las finalizaciones.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

Para completar este inicio rápido, necesita [Visual Studio Code](https://code.visualstudio.com/) con la [extensión Bicep](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep) instalada. También necesita la [CLI de Azure](/cli/azure/) o el [módulo de Azure PowerShell](/powershell/azure/new-azureps-module-az) más recientes instalados y autenticados.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-bicep-file"></a>Creación de un archivo de Bicep

Cree y abra con Visual Studio Code un archivo llamado *azuredeploy.bicep*.

## <a name="add-an-azure-resource"></a>Incorporación de un recurso de Azure

Agregue un recurso de cuenta de almacenamiento básico al archivo de Bicep.

```bicep
resource stg 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: 'storageaccount1' // must be globally unique
  location: 'eastus'
  tags:{
    diplayName: 'storageaccount1'
  }
  sku: {
      name: 'Standard_LRS'
      tier: 'Standard'
  }
  kind: 'Storage'
}
```

La declaración del recurso tiene cuatro componentes:

- **resource**: palabra clave.
- **nombre simbólico** (stg): el nombre simbólico es un identificador para hacer referencia al recurso en el archivo de Bicep. No es cuál será el nombre del recurso cuando se implemente. El nombre del recurso se define mediante la propiedad **name**.  Vea el cuarto componente de esta lista.
- **tipo de recurso** (Microsoft.Storage/storageAccounts@2019-06-01): se compone del proveedor de recursos (Microsoft.Storage), el tipo de recurso (storageAccounts) y apiVersion (2019-06-01). Cada proveedor de recursos publica sus propias versiones de API, por lo que este valor es específico del tipo. Puede encontrar más tipos y valores apiVersion para varios recursos de Azure en la [referencia de plantillas de ARM](/azure/templates/).
- **propiedades** (todo dentro de = {...}): especifique las propiedades para el tipo de recurso. Cada recurso tiene una propiedad `name`. La mayoría de los recursos también tienen una propiedad `location`, que establece la región donde se implementa el recurso. Las demás propiedades varían según el tipo de recurso y la versión de API.

Para más información, consulte [Nociones sobre la estructura y la sintaxis de los archivos Bicep](./bicep-file.md).

Hay un comentario para la propiedad name.  Use `//` para los comentarios de una sola línea o `/* ... */` para los comentarios de varias líneas.

## <a name="completion-and-validation"></a>Finalización y validación

Una de las funcionalidades más eficaces de la extensión es su integración con los esquemas de Azure. Estos esquemas proporcionan a la extensión funcionalidades de validación y finalización de recursos. Vamos a modificar la cuenta de almacenamiento para ver tanto la validación como la finalización en acción.

En primer lugar, actualice el tipo de cuenta de almacenamiento a un valor no válido, como `megaStorage`. Tenga en cuenta que esta acción genera una advertencia que indica que `megaStorage` no es un valor válido.

![Imagen que muestra una configuración de almacenamiento no válida](./media/quickstart-create-bicep-use-visual-studio-code/azure-resource-manager-template-bicep-visual-studio-code-validation.png)

Para usar las funciones de finalización, quite `megaStorage`, coloque el cursor dentro de las comillas simples y presione `ctrl` + `space`. Esta acción presenta una lista de finalización de valores válidos.

![Imagen que muestra la finalización automática de la extensión](./media/quickstart-create-bicep-use-visual-studio-code/azure-resource-manager-template-bicep-visual-studio-code-auto-completion.png)

## <a name="add-parameters"></a>Incorporación de parámetros

Ahora cree y use un parámetro para especificar el nombre de la cuenta de almacenamiento.

Agregue el código siguiente al principio del archivo:

```bicep
@minLength(3)
@maxLength(24)
@description('Specify a storage account name.')
param storageAccountName string
```

Los nombres de cuenta de Azure Storage tienen una longitud mínima de 3 caracteres, mientras que el máximo son 24. Utilice `minLength` y `maxLength` para proporcionar los valores adecuados.

Ahora, en el recurso de almacenamiento, actualice la propiedad name para usar el parámetro. Para ello, quite el nombre del recurso de almacenamiento actual, incluidas las comillas simples. Presione `ctrl` + `space`. Seleccione el parámetro **storageAccountName** en la lista. Observe que en Bicep se puede hacer referencia a los parámetros directamente mediante sus nombres. Las plantillas JSON necesitan una función parameter().

![Imagen en la que se muestra la finalización automática cuando se usan parámetros en recursos de Bicep](./media/quickstart-create-bicep-use-visual-studio-code/azure-resource-manager-template-bicep-visual-studio-code-valid-param.png)

## <a name="deploy-the-bicep-file"></a>Implementación del archivo de Bicep

Abra el terminal de Visual Studio Code integrado mediante la combinación de teclas `ctrl` + ```` ` ````, cambie el directorio actual a la ubicación en la que se encuentra el archivo de Bicep y, después, use la CLI de Azure o el módulo de Azure PowerShell para implementarlo.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group create --name arm-vscode --location eastus

az deployment group create --resource-group arm-vscode --template-file azuredeploy.bicep --parameters storageAccountName={your-unique-name}
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
New-AzResourceGroup -Name arm-vscode -Location eastus

New-AzResourceGroupDeployment -ResourceGroupName arm-vscode -TemplateFile ./azuredeploy.bicep -storageAccountName "{your-unique-name}"
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure dejen de ser necesarios, use la CLI de Azure o el módulo de Azure PowerShell para eliminar el grupo de recursos y el recurso del servidor del inicio rápido.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group delete --name arm-vscode
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell
Remove-AzResourceGroup -Name arm-vscode
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutoriales de Bicep para principiantes](./bicep-tutorial-create-first-bicep.md)
