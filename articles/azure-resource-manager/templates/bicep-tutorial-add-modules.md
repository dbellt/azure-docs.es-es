---
title: 'Tutorial: Incorporación de variables a un archivo de Bicep de Azure Resource Manager'
description: Use módulos para encapsular los detalles complejos de la declaración de recursos sin procesar.
author: mumian
ms.date: 03/25/2021
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 2d8ae81ac6ce9d53688499518c57b1c4a82cd055
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107949675"
---
# <a name="tutorial-add-modules-to-azure-resource-manager-bicep-file"></a>Tutorial: Incorporación de módulos a un archivo de Bicep de Azure Resource Manager

En el [tutorial anterior](bicep-tutorial-use-parameter-file.md), aprendió a usar un archivo de parámetros para implementar un archivo de Bicep. En este tutorial, aprenderá a usar los módulos de Bicep para encapsular los detalles complejos de la declaración de recursos sin procesar. Los módulos se pueden compartir y reutilizar en la solución.  Su tiempo de realización es de unos **12 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda completar el [tutorial sobre el archivo de parámetros](bicep-tutorial-use-parameter-file.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, el archivo de Bicep tenía el siguiente contenido:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.bicep":::

Este archivo de Bicep funciona bien. Pero para soluciones de mayor tamaño, puede dividir el archivo de Bicep en muchos módulos relacionados, con el fin de poder compartir estos módulos y volver a usarlos. El archivo de Bicep actual implementa una cuenta de almacenamiento, un plan de App Service y un sitio web.  Vamos a separar la cuenta de almacenamiento en un módulo.

## <a name="create-bicep-module"></a>Creación de un módulo de Bicep

Todos los archivos de Bicep se pueden consumir como si fueran módulo, por lo que no hay ninguna sintaxis específica para definir un módulo. Cree un archivo _storage.bicep_ con el siguiente contenido:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-module/storage.bicep":::

Este módulo contiene el recurso de la cuenta de almacenamiento y los parámetros y variables relacionados. Se han quitado los valores del parámetro _location_ y los parámetros _resourceTags_. Estos valores se pasarán del archivo de Bicep principal.

## <a name="consume-bicep-module"></a>Consumo de un módulo de Bicep

Reemplace la definición de recursos de la cuenta de almacenamiento en el archivo _azuredeploy.bicep_ existente por el contenido de Bicep siguiente:

```bicep
module stg './storage.bicep' = {
  name: 'storageDeploy'
  params: {
    storagePrefix: storagePrefix
    location: location
    resourceTags: resourceTags
  }
}
```

- **modulo**: palabra clave.
- **nombre simbólico** (stg): es un identificador del módulo.
- **archivo de módulo**: en este ejemplo, la ruta de acceso al módulo se especifica mediante una ruta de acceso relativa (./storage.bicep). Todas las rutas de acceso de Bicep deben especificarse mediante el separador de directorios de barra diagonal (/), con el fin de garantizar una compilación coherente entre plataformas. El carácter de barra diagonal inversa de Windows (\) no se admite.

Para recuperar el punto de conexión de almacenamiento, actualice la salida de _azuredeploy.bicep_ al siguiente archivo de Bicep:

```bicep
output storageEndpoint object = stg.outputs.storageEndpoint
```

El archivo azuredeploy.bicep completado tiene el siguiente contenido:

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-module/azuredeploy.bicep":::

## <a name="deploy-template"></a>Implementar plantilla

Use la CLI de Azure o Azure PowerShell para implementar la plantilla.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addmodule `
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
  --name addmodule \
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
1. Verá el nuevo grupo de recursos que implementó en este tutorial.
1. Seleccione el grupo de recursos y consulte los recursos implementados. Observe que coinciden con los valores especificados en el archivo de plantilla.

## <a name="clean-up-resources"></a>Limpieza de recursos

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**. Si ha completado esta serie, tiene tres grupos de recursos que eliminar: **myResourceGroup**, **myResourceGroupDev** y **myResourceGroupProd**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Enhorabuena, ha terminado esta introducción a la implementación de archivos de Bicep en Azure. Si tiene comentarios y sugerencias, háganoslo saber en la sección de comentarios. Gracias.

En la siguiente serie de tutoriales encontrará más información sobre la implementación de plantillas.

> [!div class="nextstepaction"]
> [Implementación de una plantilla local](./deployment-tutorial-local-template.md)
