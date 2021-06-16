---
title: 'Tutorial: Adición de etiquetas a recursos en un archivo de Bicep de Azure Resource Manager'
description: Agregue etiquetas a los recursos que implemente en los archivos de Bicep. Las etiquetas permiten organizar los recursos de manera lógica.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: ea5e078eb692d002b3f86cd43663dd042d692611
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353158"
---
# <a name="tutorial-add-tags-in-azure-resource-manager-bicep-files"></a>Tutorial: Adición de etiquetas en archivos de Bicep de Azure Resource Manager

En este tutorial, aprenderá a agregar etiquetas a archivos de Bicep. Las [etiquetas](../management/tag-resources.md) le ayudan a organizar los recursos de forma lógica. Los valores de la etiqueta se muestran en los informes de costos. Este tutorial se realiza en **8 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre las plantillas de inicio rápido](bicep-tutorial-quickstart-template.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

El archivo de Bicep implementó una cuenta de almacenamiento, un plan de App Service y una aplicación web.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.bicep":::

Después de implementar estos recursos, es posible que tenga que realizar un seguimiento de los costos y encontrar los recursos que pertenecen a una categoría. Para ayudar a resolver estos problemas, puede agregar etiquetas.

## <a name="add-tags"></a>Incorporación de etiquetas

Etiquete los recursos para agregar valores que le ayuden a identificar su uso. Por ejemplo, puede agregar etiquetas que muestren el entorno y el proyecto. Podría agregar etiquetas que identificaran un centro de costos o el equipo que posee el recurso. Agregue valores que tengan sentido para su organización.

En el siguiente ejemplo se muestran los cambios realizados en el archivo de Bicep. Copie el archivo completo y reemplace el archivo de Bicep por su contenido.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.bicep" range="1-81" highlight="27-30,38,51,71":::

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Es el momento de implementar el archivo de Bicep y examinar los resultados.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addtags `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS `
  -webAppName demoapp
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addtags \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información acerca de los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="verify-deployment"></a>Comprobación de la implementación

Para comprobar la implementación, explore el grupo de recursos desde Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. Seleccione uno de los recursos, por ejemplo, el recurso de cuenta de almacenamiento. Verá que ahora tiene etiquetas.

   ![Mostrar etiquetas](./media/bicep-tutorial-add-tags/show-tags.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado etiquetas a los recursos. En el siguiente tutorial, aprenderá a usar archivos de parámetros para simplificar el paso de valores a la implementación.

> [!div class="nextstepaction"]
> [Uso de archivos de parámetros](bicep-tutorial-use-parameter-file.md)
