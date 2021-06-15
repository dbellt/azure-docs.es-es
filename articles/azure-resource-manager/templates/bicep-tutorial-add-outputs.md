---
title: 'Tutorial: Adición de salidas al archivo de Bicep de Azure Resource Manager'
description: Agregue salidas a cualquier archivo de Bicep para simplificar la sintaxis.
author: mumian
ms.date: 03/17/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 4b7d7a1414091c516dba2c474e1681ba357b55a1
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353170"
---
# <a name="tutorial-add-outputs-to-azure-resource-manager-bicep-file"></a>Tutorial: Adición de salidas al archivo de Bicep de Azure Resource Manager

En este tutorial, aprenderá a devolver un valor de una implementación. Las salidas se usan cuando necesita un valor de un recurso implementado. Este tutorial se realiza en **7 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre variables](bicep-tutorial-add-variables.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, el archivo de Bicep tenía el siguiente contenido:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.bicep":::

Este código implementa una cuenta de almacenamiento, pero no devuelve ninguna información sobre dicha cuenta. Es posible que necesite capturar propiedades de un nuevo recurso para que estén disponibles más adelante como referencia.

## <a name="add-outputs"></a>Adición de salidas

Puede usar salidas para devolver valores de la implementación. Por ejemplo, puede resultar útil obtener los puntos de conexión de la nueva cuenta de almacenamiento.

En el ejemplo siguiente se muestra el cambio que se realiza en el archivo Bicep para agregar un valor de salida. Copie el archivo completo y reemplace el archivo de Bicep por su contenido.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.bicep" range="1-33" highlight="33":::

Hay algunos elementos importantes que se deben tener en cuenta sobre el valor de salida que ha agregado.

El tipo de valor devuelto se establece en `object`, lo que significa que devuelve un objeto de plantilla.

Para obtener la propiedad `primaryEndpoints` de la cuenta de almacenamiento, use el nombre simbólico de la cuenta de almacenamiento. La característica de autocompletar de Visual Studio Code presenta una lista completa de las propiedades:

   ![Propiedades de objeto de nombre simbólico de Bicep de Visual Studio Code](./media/bicep-tutorial-add-outputs/visual-studio-code-bicep-output-properties.png)

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Ya está listo para implementar el archivo de Bicep y examinar el valor devuelto.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addoutputs `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addoutputs \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

---

En el resultado del comando de implementación, verá un objeto similar al siguiente ejemplo solo si el resultado está en formato JSON:

```json
{
  "dfs": "https://storeluktbfkpjjrkm.dfs.core.windows.net/",
  "web": "https://storeluktbfkpjjrkm.z19.web.core.windows.net/",
  "blob": "https://storeluktbfkpjjrkm.blob.core.windows.net/",
  "queue": "https://storeluktbfkpjjrkm.queue.core.windows.net/",
  "table": "https://storeluktbfkpjjrkm.table.core.windows.net/",
  "file": "https://storeluktbfkpjjrkm.file.core.windows.net/"
}
```

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información acerca de los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="review-your-work"></a>Revisión del trabajo

En los seis últimos tutoriales ha hecho muchas cosas. Dedique un momento a revisar lo que ha hecho. Ha creado un archivo de Bicep con parámetros fáciles de proporcionar. El archivo de Bicep se puede volver a usar en diferentes entornos, ya que se puede personalizar y crea dinámicamente valores necesarios. También devuelve información sobre la cuenta de almacenamiento que puede usar en el script.

Ahora, echemos un vistazo al grupo de recursos y al historial de implementaciones.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Grupos de recursos**.
1. Seleccione el grupo de recursos en el que ha realizado la implementación.
1. En función de los pasos que haya realizado, debe tener al menos una cuenta de almacenamiento en el grupo de recursos.
1. También debe tener varias implementaciones correctas enumeradas en el historial. Seleccione ese vínculo.

   ![Seleccionar implementaciones](./media/bicep-tutorial-add-outputs/select-deployments.png)

1. Verá todas las implementaciones del historial. Seleccione la implementación llamada **addoutputs**.

   ![Mostrar el historial de implementaciones](./media/bicep-tutorial-add-outputs/show-history.png)

1. Puede revisar las entradas.

   ![Mostrar entradas](./media/bicep-tutorial-add-outputs/show-inputs.png)

1. Puede revisar las salidas.

   ![Mostrar salidas](./media/bicep-tutorial-add-outputs/show-outputs.png)

1. Puede examinar la plantilla JSON.

   ![Mostrar la plantilla](./media/bicep-tutorial-add-outputs/show-template.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado un valor devuelto al archivo de Bicep. En el siguiente tutorial, aprenderá a exportar una plantilla JSON y a usar partes de la plantilla exportada en su archivo de Bicep.

> [!div class="nextstepaction"]
> [Uso de una plantilla exportada](bicep-tutorial-export-template.md)
