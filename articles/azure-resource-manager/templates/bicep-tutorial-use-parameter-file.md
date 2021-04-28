---
title: 'Tutorial: Uso de un archivo de parámetros para implementar un archivo de Bicep de Azure Resource Manager'
description: Use archivos de parámetros que contengan los valores que se usarán para implementar el archivo de Bicep.
author: mumian
ms.date: 04/27/2021
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 73af451292462d9963720924df7718116a78baaa
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108071494"
---
# <a name="tutorial-use-parameter-files-to-deploy-azure-resource-manager-bicep-file"></a>Tutorial: Uso de un archivo de parámetros para implementar un archivo de Bicep de Azure Resource Manager

En este tutorial, aprenderá a usar [archivos de parámetros](parameter-files.md) para almacenar los valores que se pasan durante la implementación. En los tutoriales anteriores, usó parámetros insertados con el comando de implementación. Este enfoque funcionó para probar el archivo de Bicep, pero al automatizar las implementaciones puede ser más fácil pasar un conjunto de valores del entorno. Los archivos de parámetros facilitan el empaquetado de los valores de los parámetros de un entorno específico. Use el mismo archivo de parámetros JSON que se usa para implementar una plantilla JSON. En este tutorial, creará archivos de parámetros para entornos de desarrollo y producción. Su tiempo de realización es de unos **12 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre etiquetas](bicep-tutorial-add-tags.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

El archivo de Bicep tiene muchos parámetros que puede proporcionar durante la implementación. Al final del tutorial anterior, el archivo de Bicep tenía el siguiente aspecto:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.bicep":::

Este archivo de Bicep funciona bien, pero ahora desea administrar fácilmente los parámetros que pasa al archivo de Bicep.

## <a name="add-parameter-files"></a>Adición de archivos de parámetros

Los archivos de parámetros son archivos JSON con una estructura similar a la de las plantillas JSON. En el archivo, proporcione los valores de los parámetros que desea pasar durante la implementación.

Especifique los valores de los parámetros del archivo de Bicep en el archivo de parámetros. El nombre de cada uno de los parámetros del archivo de parámetros debe coincidir con el nombre de un parámetro del archivo de Bicep. El nombre no distingue entre mayúsculas y minúsculas, pero para ver fácilmente los valores coincidentes se recomienda que haga coincidir el uso de mayúsculas y minúsculas del archivo de Bicep.

No es necesario proporcionar un valor para cada parámetro. Si un parámetro no especificado tiene un valor predeterminado, se usa ese valor durante la implementación. Si un parámetro no tiene un valor predeterminado y no se especifica en el archivo de parámetros, se le pedirá que proporcione un valor durante la implementación.

No se puede especificar un nombre de parámetro en el archivo de parámetros que no coincida con un nombre de parámetro en el archivo de Bicep. Recibirá un error cuando se proporcionan parámetros desconocidos.

En Visual Studio Code, cree un archivo con el siguiente contenido. Guarde el archivo con el nombre _azuredeploy.parameters.dev.json_.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.dev.json":::

Este archivo es el archivo de parámetros para el entorno de desarrollo. Observe que usa **Standard_LRS** para la cuenta de almacenamiento, asigna un nombre a los recursos con el prefijo **dev** y establece la etiqueta `Environment` en **Dev**.

De nuevo, cree un nuevo archivo con el siguiente contenido. Guarde el archivo con el nombre _azuredeploy.parameters.prod.json_.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.prod.json":::

Este archivo es el archivo de parámetros para el entorno de producción. Observe que usa **Standard_GRS** para la cuenta de almacenamiento, asigna un nombre a los recursos con el prefijo **contoso** y establece la etiqueta `Environment` en **Production**. En un entorno de producción real, también deseará usar una instancia de App Service con una SKU que no sea gratuita, pero seguiremos con esa SKU para este tutorial.

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Utilice la CLI de Azure o Azure PowerShell para implementar el archivo de Bicep.

En este tutorial, vamos a crear dos grupos de recursos. Uno para el entorno de desarrollo y otro para el entorno de producción.

En el caso de las variables de plantilla y parámetro, reemplace `{path-to-the-bicep-file}`, `{path-to-azuredeploy.parameters.dev.json}`, `{path-to-azuredeploy.parameters.prod.json}` y las llaves `{}` por las rutas de acceso al archivo del archivo de Bicep y del archivo de parámetros.

En primer lugar, se implementará en el entorno de desarrollo.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
$bicepFile = "{path-to-the-bicep-file}"
$parameterFile="{path-to-azuredeploy.parameters.dev.json}"
New-AzResourceGroup `
  -Name myResourceGroupDev `
  -Location "East US"
New-AzResourceGroupDeployment `
  -Name devenvironment `
  -ResourceGroupName myResourceGroupDev `
  -TemplateFile $bicepFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
bicepFile="{path-to-the-bicep-file}"
devParameterFile="{path-to-azuredeploy.parameters.dev.json}"
az group create \
  --name myResourceGroupDev \
  --location "East US"
az deployment group create \
  --name devenvironment \
  --resource-group myResourceGroupDev \
  --template-file $bicepFile \
  --parameters $devParameterFile
```

---

Ahora, se implementará en el entorno de producción.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$parameterFile="{path-to-azuredeploy.parameters.prod.json}"
New-AzResourceGroup `
  -Name myResourceGroupProd `
  -Location "West US"
New-AzResourceGroupDeployment `
  -Name prodenvironment `
  -ResourceGroupName myResourceGroupProd `
  -TemplateFile $bicepFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
prodParameterFile="{path-to-azuredeploy.parameters.prod.json}"
az group create \
  --name myResourceGroupProd \
  --location "West US"
az deployment group create \
  --name prodenvironment \
  --resource-group myResourceGroupProd \
  --template-file $bicepFile \
  --parameters $prodParameterFile
```

---

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información sobre los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Verá los dos nuevos grupos de recursos que implementó en este tutorial.
1. Seleccione cualquier grupo de recursos y consulte los recursos implementados. Observe que coinciden con los valores especificados en el archivo de parámetros para ese entorno.

## <a name="clean-up-resources"></a>Limpieza de recursos

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se usa un archivo de parámetros para implementar el archivo de Bicep. En el siguiente tutorial, aprenderá a modular archivos de Bicep.

> [!div class="nextstepaction"]
> [Adición de módulos](./bicep-tutorial-add-modules.md)
