---
title: 'Tutorial: uso de plantillas de inicio rápido para el desarrollo de Bicep de Azure Resource Manager'
description: Aprenda a usar las plantillas de inicio rápido de Azure para completar el desarrollo de Bicep.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: c20a8307fc61e180b378fee034e086a5f54ba03f
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353144"
---
# <a name="tutorial-use-azure-quickstart-templates-for-azure-resource-manager-bicep-development"></a>Tutorial: uso de plantillas de inicio rápido de Azure para el desarrollo de Bicep de Azure Resource Manager

[Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) es un repositorio de plantillas JSON aportadas por la comunidad. Puede usar las plantillas de ejemplo en el desarrollo de Bicep. En este tutorial, encontrará una definición de recursos de sitio web, la descompilará en Bicep y la agregará al archivo de Bicep. Su tiempo de realización es de unos **12 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre plantillas exportadas](bicep-tutorial-export-template.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, el archivo de Bicep tenía el siguiente contenido:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/export-template/azuredeploy.bicep":::

Este archivo de Bicep funciona para implementar cuentas de almacenamiento y planes de App Service, pero puede agregarle un sitio web. Puede usar plantillas predefinidas para averiguar rápidamente el código JSON necesario para implementar un recurso. Para que las plantillas de inicio rápido de Azure ofrezcan ejemplos de Bicep, puede usar herramientas de conversión para convertir plantillas JSON en archivos de Bicep.

## <a name="find-template"></a>Búsqueda de la plantilla

Actualmente, las plantillas de inicio rápido de Azure solo proporcionan plantillas JSON. Hay herramientas que se pueden usar para descompilar plantillas JSON en archivos de Bicep.

1. Abra [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/).
1. En **Buscar**, escriba _deploy linux web app_ (implementar aplicación web Linux).
1. Seleccione el icono que tenga el título **Deploy a basic Linux web app** (Implementación de una aplicación web Linux básica). Si tiene problemas para encontrarla, este es el [vínculo directo](https://azure.microsoft.com/resources/templates/webapp-basic-linux/).
1. Seleccione **Explorar en GitHub**.
1. Seleccione _azuredeploy.json_. Esta es la plantilla que puede usar.
1. Seleccione **Sin formato** y, después, haga una copia de la dirección URL.
1. Vaya a **https://bicepdemo.z22.web.core.windows.net/** , seleccione **Descompilar** y, después, especifique la dirección URL de la plantilla sin formato.
1. Revise la plantilla de Bicep. En particular, busque el recurso `Microsoft.Web/sites`.

    ![Sitio web de inicio rápido de la plantilla de Resource Manager](./media/bicep-tutorial-quickstart-template/resource-manager-template-quickstart-template-web-site.png)

    Hay un par de características importantes de Bicep importantes que se deben tener en cuenta en este nuevo recurso si ha trabajado en plantillas JSON.

    En las plantillas de código JSON de Azure Resource Manager, debe especificar manualmente las dependencias de los recursos con la propiedad _dependsOn_. El recurso del sitio web depende del recurso del plan de App Service. Con Bicep, si hace referencia a cualquier propiedad del recurso mediante el nombre simbólico, la propiedad dependsOn se agrega automáticamente.

    Puede hacer referencia fácilmente al identificador de recurso desde el nombre simbólico del plan de App Service (appServicePlanName.id), que se traducirá en la función resourceId (...) en la plantilla JSON compilada.

## <a name="revise-existing-bicep-file"></a>Revise el archivo de Bicep existente

Combine la plantilla de inicio rápido descompilada con el archivo de Bicep existente. Tal como hizo en el tutorial anterior, actualice el nombre simbólico del recurso y el nombre del recurso para que coincida con su convención de nomenclatura.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/quickstart-template/azuredeploy.bicep" range="1-73" highlight="20-25,28,61-71":::

El nombre de la aplicación web debe ser único en Azure. Para evitar tener nombres duplicados, la variable `webAppPortalName` se ha actualizado de `var webAppPortalName_var = '${webAppName}-webapp'` a `var webAppPortalName = '${webAppName}${uniqueString(resourceGroup().id)}'` .

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Use la CLI de Azure o Azure PowerShell para implementar una plantilla de Bicep.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable **bicepFile** en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addwebapp `
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
  --name addwebapp \
  --resource-group myResourceGroup \
  --template-file $bicepFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS webAppName=demoapp
```

---

> [!NOTE]
> Si se produjo un error en la implementación, use el modificador `verbose` para obtener información acerca de los recursos que se están creando. Utilice el modificador `debug` para más información sobre la depuración.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a usar una plantilla de inicio rápido para el desarrollo de Bicep. En el siguiente tutorial, agregará etiquetas a los recursos.

> [!div class="nextstepaction"]
> [Adición de etiquetas](bicep-tutorial-add-tags.md)
