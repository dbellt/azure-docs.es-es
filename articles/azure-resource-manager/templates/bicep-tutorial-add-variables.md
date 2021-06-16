---
title: 'Tutorial: Incorporación de variables a un archivo de Bicep de Azure Resource Manager'
description: Agregue variables a cualquier archivo de Bicep para simplificar la sintaxis.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: da2755c1f2c0f9fa891fe1a99b1fed21f64492c8
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353152"
---
# <a name="tutorial-add-variables-to-azure-resource-manager-bicep-file"></a>Tutorial: Incorporación de variables a un archivo de Bicep de Azure Resource Manager

En este tutorial, aprenderá a agregar una variable a un archivo Bicep. Las variables simplifican los archivos de Bicep, ya que le permiten escribir una expresión una vez y volver a usarla siempre que lo desee en todo el archivo de Bicep. Este tutorial se realiza en **7 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre las funciones](bicep-tutorial-add-functions.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, el archivo de Bicep tenía el siguiente contenido:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-location/azuredeploy.bicep":::

El parámetro del nombre de la cuenta de almacenamiento es difícil de usar porque tiene que proporcionar un nombre único. Si ha realizado los tutoriales anteriores de esta serie, probablemente esté cansado de pensar en un nombre único. Para solucionar este problema, agregue una variable que construya un nombre único para la cuenta de almacenamiento.

## <a name="use-variable"></a>Uso de la variable

En el ejemplo siguiente se muestran los cambios que hay que realizar para agregar una variable al archivo de Bicep que crea un nombre de cuenta de almacenamiento único. Copie el archivo completo y reemplace el archivo de Bicep por su contenido.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.bicep" range="1-31" highlight="1-3,19,22":::

Observe que se incluye una variable llamada `uniqueStorageName`. Esta variable usa tres funciones para construir un valor de cadena.

Ya conoce la función [resourceGroup](template-functions-resource.md#resourcegroup). En este caso, obtendrá la propiedad `id` en lugar de la propiedad `location`, tal como se muestra en el tutorial anterior. La propiedad `id` devuelve el identificador completo del grupo de recursos, incluido el identificador de la suscripción y el nombre del grupo de recursos.

La función [uniqueString](template-functions-string.md#uniquestring) crea un valor hash de 13 caracteres. El valor devuelto viene determinado por los parámetros que se pasan. En este tutorial, usará el identificador del grupo de recursos como entrada del valor hash. lo que significa que puede implementar este archivo de Bicep en distintos grupos de recursos y obtener un valor de cadena único diferente. Sin embargo, se obtiene el mismo valor si la implementa en el mismo grupo de recursos.

Bicep admite una sintaxis de [interpolación de cadena](https://en.wikipedia.org/wiki/String_interpolation#). En el caso de esta variable, toma la cadena del parámetro y la cadena de la función `uniqueString` y las combina en una cadena.

El parámetro `storagePrefix` le permite pasar un prefijo que le ayuda a identificar las cuentas de almacenamiento. Puede crear su propia convención de nomenclatura que facilite la identificación de las cuentas de almacenamiento entre una larga lista de recursos después de la implementación.

Por último, observe que el nombre de almacenamiento ahora está establecido en la variable en vez de en un parámetro.

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Vamos a implementar el archivo de Bicep. La implementación de este archivo Bicep es más sencilla que la de los anteriores, ya que solo se especifica el prefijo del nombre de almacenamiento.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addnamevariable `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addnamevariable \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

---

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información acerca de los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Verá que se ha implementado un recurso de cuenta de almacenamiento. El nombre de la cuenta de almacenamiento es **store** más una cadena de caracteres aleatorios.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado una variable que crea un nombre único para una cuenta de almacenamiento. En el siguiente tutorial, devolverá un valor de la cuenta de almacenamiento implementada.

> [!div class="nextstepaction"]
> [Adición de salidas](bicep-tutorial-add-outputs.md)
