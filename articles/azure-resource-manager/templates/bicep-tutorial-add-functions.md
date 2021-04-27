---
title: 'Tutorial: Adición de funciones a archivos de Bicep de Azure Resource Manager'
description: Agregue funciones a archivos de Bicep para construir valores.
author: mumian
ms.date: 04/20/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: references_regions
ms.openlocfilehash: 43dec6ceb21a6604bc0034b3f14b79ffd2cbe263
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773808"
---
# <a name="tutorial-add-functions-to-azure-resource-manager-bicep-file"></a>Tutorial: Adición de funciones a un archivo de Bicep de Azure Resource Manager

En este tutorial, aprenderá a agregar [funciones de plantilla](template-functions.md) a un archivo de Bicep. Las funciones se usan para construir valores de forma dinámica. Actualmente, Bicep no admite funciones definidas por el usuario. Este tutorial se realiza en **7 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre parámetros](bicep-tutorial-add-parameters.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, el archivo de Bicep tenía el siguiente contenido:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-sku/azuredeploy.bicep":::

La ubicación de la cuenta de almacenamiento está codificada de forma rígida en **Este de EE. UU.** . Sin embargo, puede que tenga que implementar la cuenta de almacenamiento en otras regiones. Una vez más, se enfrenta al problema de que un archivo de Bicep no tiene flexibilidad. Podría agregar un parámetro para la ubicación, pero sería aún mejor si su valor predeterminado fuera algo más que un simple valor codificado de forma rígida.

## <a name="use-function"></a>Uso de una función

Las funciones agregan flexibilidad a cualquier archivo de Bicep mediante la obtención dinámica de valores durante la implementación. En este tutorial, usará una función para obtener la ubicación del grupo de recursos que está usando para la implementación.

En el ejemplo siguiente se muestran los cambios que hay que realizar para agregar un parámetro denominado `location`. El valor predeterminado del parámetro llama a la función [resourceGroup](template-functions-resource.md#resourcegroup). Esta función devuelve un objeto con información sobre el grupo de recursos que se utiliza para la implementación. Una de las propiedades del objeto es una propiedad de ubicación. Cuando se usa el valor predeterminado, la cuenta de almacenamiento tiene la misma ubicación que el grupo de recursos. No es necesario que los recursos incluidos en un grupo de recursos compartan la misma ubicación. También puede proporcionar una ubicación diferente cuando sea necesario.

Copie el archivo completo y reemplace el archivo de Bicep por su contenido.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-location/azuredeploy.bicep" range="1-30" highlight="18,22":::

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

En los tutoriales anteriores, creó una cuenta de almacenamiento en Este de EE. UU., pero el grupo de recursos se creó en Centro de EE. UU. En este tutorial, la cuenta de almacenamiento se crea en la misma región que el grupo de recursos. Use el valor predeterminado para la ubicación, con lo que no será necesario proporcionar el valor de ese parámetro. Debe proporcionar un nuevo nombre para la cuenta de almacenamiento porque está creando una cuenta de almacenamiento en una ubicación diferente. Por ejemplo, use **store2** como prefijo en lugar de **store1**.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addlocationparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{new-unique-name}"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addlocationparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageName={new-unique-name}
```

---

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información acerca de los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Verá que un recurso de la cuenta de almacenamiento se ha implementado y tiene la misma ubicación que el grupo de recursos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha usado una función al definir el valor predeterminado de un parámetro. En esta serie de tutoriales, seguirá usando funciones. Al final de la serie, agregará funciones a cada sección del archivo de Bicep.

> [!div class="nextstepaction"]
> [Adición de variables](bicep-tutorial-add-variables.md)
