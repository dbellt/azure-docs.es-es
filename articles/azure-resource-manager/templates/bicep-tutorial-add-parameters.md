---
title: 'Tutorial: Adición de parámetros a un archivo de Bicep de Azure Resource Manager'
description: Agregue parámetros a un archivo de Bicep para que se pueda volver a usar.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c017595d33856ba7a2902ec6d86f7c36a8c53107
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353167"
---
# <a name="tutorial-add-parameters-to-azure-resource-manager-bicep-file"></a>Tutorial: Adición de parámetros a un archivo de Bicep de Azure Resource Manager

En el [tutorial anterior](bicep-tutorial-create-first-bicep.md), aprendió a implementar una cuenta de almacenamiento. En este tutorial, aprenderá a mejorar el archivo de Bicep la plantilla mediante la incorporación de parámetros. Este tutorial se realiza en unos **14 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Es aconsejable consultar antes el tutorial de [creación e implementación del primer archivo de Bicep](bicep-tutorial-create-first-bicep.md), pero no es necesario.

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, así era el archivo de Bicep:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-storage/azuredeploy.bicep":::

Es posible que haya observado que existe un problema con este archivo de Bicep. El nombre de la cuenta de almacenamiento está codificado de forma rígida. Este archivo de Bicep solo se puede usar para implementar siempre la misma cuenta de almacenamiento. Para implementar una cuenta de almacenamiento con otro nombre, tendría que crear un archivo de Bicep, lo que obviamente no es una forma práctica de automatizar las implementaciones.

## <a name="make-bicep-file-reusable"></a>Reutilización de un archivo de Bicep

Para que el archivo de Bicep se pueda volver a usar, vamos a agregar un parámetro que puede usar para pasar un nombre de cuenta de almacenamiento. El siguiente archivo Bicep muestra lo que ha cambiado en el archivo. El parámetro `storageName` se identifica como una cadena. La longitud máxima se establece en 24 caracteres para evitar que los nombres sean demasiado largos.

Copie todo el archivo y reemplácelo por el contenido siguiente.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-name/azuredeploy.bicep" range="1-15" highlight="1-3,6":::

Observe que se puede hacer referencia a los parámetros directamente mediante sus nombres en Bicep, en comparación con la plantilla JSON de Azure Resource Manager, que requiere [parámetros("nombreDeAlmacenamiento")].

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Vamos a implementar el archivo de Bicep. En el ejemplo siguiente se implementa el archivo de Bicep con la CLI de Azure o PowerShell. Observe que se proporciona el nombre de la cuenta de almacenamiento como uno de los valores en el comando de implementación. Para el nombre de la cuenta de almacenamiento, proporcione el mismo nombre que usó en el tutorial anterior.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addnameparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addnameparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageName={your-unique-name}
```

---

## <a name="understand-resource-updates"></a>Descripción de actualizaciones de recursos

En la sección anterior, ha implementado una cuenta de almacenamiento con el mismo nombre que creó anteriormente. Es posible que se pregunte cómo afecta al recurso la reimplementación.

Si el recurso ya existe y no se detecta ningún cambio en las propiedades, no se realiza ninguna acción. Si el recurso ya existe y se produce un cambio en alguna propiedad, el recurso se actualiza. Si el recurso no existe, se creará.

Esta forma de controlar las actualizaciones significa que el archivo de Bicep puede incluir todos los recursos necesarios para una solución de Azure. Puede volver a implementar el archivo de Bicep de forma segura con la certeza de que los recursos se cambiarán o se crearán solo cuando sea necesario. Por ejemplo, si ha agregado archivos a su cuenta de almacenamiento, puede volver a implementar la cuenta de almacenamiento sin perder esos archivos.

## <a name="customize-by-environment"></a>Personalización por entorno

Los parámetros le permiten personalizar la implementación al proporcionar valores que son específicos para un entorno concreto. Por ejemplo, puede pasar diferentes valores en función de si está implementando en un entorno de desarrollo, de prueba o de producción.

El archivo de Bicep anterior siempre implementaba una cuenta de almacenamiento **Standard_LRS**. Es posible que desee disponer de flexibilidad para implementar diferentes SKU según el entorno. En el ejemplo siguiente se resaltan los cambios para agregar un parámetro para la SKU. Copie el archivo completo y péguelo en el archivo de Bicep.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-sku/azuredeploy.bicep" range="1-27" highlight="5-15,21":::

El parámetro `storageSKU` tiene un valor predeterminado. Este valor se usa cuando no se especifica ningún valor durante la implementación. También tiene una lista de valores permitidos. Estos valores coinciden con los valores necesarios para crear una cuenta de almacenamiento. No desea que los usuarios de su archivo de Bicep pasen referencias de almacén que no funcionan.

## <a name="redeploy-bicep-file"></a>Nueva implementación de un archivo de Bicep

Está listo para volver a implementar. Dado que la SKU predeterminada está establecida en **Standard_LRS**, no es necesario proporcionar un valor para ese parámetro.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --name addskuparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageName={your-unique-name}
```

---

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información sobre los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

Para ver la flexibilidad del archivo de Bicep, vamos a implementarlo de nuevo. Esta vez, establezca el parámetro de SKU en **Standard_GRS**. Puede pasar un nuevo nombre para crear una cuenta de almacenamiento diferente, o bien usar el mismo nombre para actualizar la cuenta de almacenamiento existente. Ambas opciones funcionan.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name usenondefaultsku `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}" `
  -storageSKU Standard_GRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --name usenondefaultsku \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageSKU=Standard_GRS storageName={your-unique-name}
```

---

Por último, vamos a realizar una prueba más y ver lo que sucede cuando se pasa una SKU que no es uno de los valores permitidos. En este caso, probaremos un escenario en el que un usuario del archivo de Bicep piensa que **basic** es una de las referencias de almacén.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name testskuparameter `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storageName "{your-unique-name}" `
  -storageSKU basic
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az deployment group create \
  --name testskuparameter \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storageSKU=basic storageName={your-unique-name}
```

---

Se produce un error en el comando inmediatamente con un mensaje de error que indica qué valores se permiten. Resource Manager identifica el error antes de que se inicie la implementación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha mejorado el archivo de Bicep creado en el [primer tutorial](bicep-tutorial-create-first-bicep.md) mediante la adición de parámetros. En el siguiente tutorial, obtendrá información sobre las funciones de Bicep.

> [!div class="nextstepaction"]
> [Adición de funciones](bicep-tutorial-add-functions.md)
