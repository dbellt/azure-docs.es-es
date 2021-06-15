---
title: 'Tutorial: exportación de una plantilla JSON desde Azure Portal para el desarrollo de Bicep'
description: Aprenda a usar una plantilla JSON exportada para completar el desarrollo de Bicep.
author: mumian
ms.date: 03/10/2021
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 3bc7ed4ada4f7810e9864778c7f76a0573c9dc89
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111353147"
---
# <a name="tutorial-use-exported-json-template-from-the-azure-portal"></a>Tutorial: Uso de una plantilla JSON exportada desde Azure Portal

En esta serie de tutoriales, ha creado un archivo de Bicep para implementar una cuenta de almacenamiento de Azure. En los dos tutoriales siguientes, agregará un *plan de App Service* y un *sitio web*. En lugar de crear plantillas JSON desde cero, aprenderá a exportarlas desde Azure Portal y cómo usar las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) como ejemplo. Se puede personalizar el uso de esas plantillas. Este tutorial se centra en la exportación de plantillas y en la personalización del resultado del archivo Bicep. Se tarda en realizar unos **14 minutos**.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el [tutorial sobre las salidas](bicep-tutorial-add-outputs.md).

Debe tener Visual Studio Code con la extensión Bicep y Azure PowerShell o la CLI de Azure. Para más información, consulte la sección relativa a las [herramientas de Bicep](bicep-tutorial-create-first-bicep.md#get-tools).

## <a name="review-bicep-file"></a>Examen de un archivo de Bicep

Al final del tutorial anterior, el archivo de Bicep tenía el siguiente contenido:

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.bicep":::

Este archivo Bicep es adecuado para implementar cuentas de almacenamiento, pero puede agregarle más recursos. Puede exportar una plantilla JSON desde un recurso existente para obtener rápidamente el código JSON del mismo. Después, convierta el código JSON a Bicep antes de poder agregarlo al archivo de Bicep.

## <a name="create-app-service-plan"></a>Creación de un plan de App Service

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **Plan de App Service** y, luego, seleccione **Plan de App Service**.  No seleccione **Plan de App Service (clásico)** .
1. Seleccione **Crear**.
1. Especifique:

    - **Suscripción**: seleccione su suscripción de Azure.
    - **Grupo de recursos**: Seleccione **Crear nuevo** y, luego, especifique un nombre. Proporcione un nombre de grupo de recursos diferente al que ha usado en esta serie de tutoriales.
    - **Nombre**: escriba un nombre para el plan de App Service.
    - **Sistema operativo**: seleccione **Linux**.
    - **Región**: seleccione una ubicación de Azure. Por ejemplo, **Centro de EE. UU**.
    - **Plan de tarifa**: para ahorrar costos, cambie la SKU a **Básico B1** (en desarrollo y pruebas).

    ![Plantilla de Resource Manager: exportación de plantillas en el portal](./media/bicep-tutorial-export-template/resource-manager-template-export.png)
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**. La creación del recurso tarda algunos minutos.

## <a name="export-template"></a>Exportación de la plantilla

Actualmente, Azure Portal solo admite la exportación de plantillas JSON. Hay herramientas que puede usar para descompilar plantillas JSON en archivos de Bicep.

1. Haga clic en **Go to resource** (Ir al recurso).

    ![Ir al recurso](./media/bicep-tutorial-export-template/resource-manager-template-export-go-to-resource.png)

1. Seleccione **Exportar plantilla**.

    ![Plantilla de Resource Manager: exportación de plantillas](./media/bicep-tutorial-export-template/resource-manager-template-export-template.png)

   La característica de exportación de plantillas toma el estado actual de un recurso y genera una plantilla para implementarlo. Exportar una plantilla puede ser una forma útil de obtener rápidamente el código JSON necesario para implementar un recurso.

1. Los elementos que necesita son la `Microsoft.Web/serverfarms` definición y la definición de los parámetros.

    ![Plantilla de Resource Manager: plantillas exportadas con la exportación de plantillas](./media/bicep-tutorial-export-template/resource-manager-template-exported-template.png)

    > [!IMPORTANT]
    > Normalmente, la plantilla exportada es más detallada que cuando se crea una plantilla. Por ejemplo, el objeto SKU de la plantilla exportada tiene cinco propiedades. Esta plantilla funciona, pero podría usar simplemente la propiedad `name`. Puede comenzar con la plantilla exportada y, luego, modificarla para adaptarla a sus necesidades.

1. Seleccione **Descargar**.  El archivo zip descargado contiene un archivo **template.json** y un archivo **parameters.json**. Descomprima los archivos.
1. Vaya a **https://bicepdemo.z22.web.core.windows.net/** , seleccione **Descompilar** y, después, abra **template.json**. La plantilla se obtiene en Bicep.

## <a name="revise-existing-bicep-file"></a>Examen del archivo de Bicep existente

La plantilla exportada descompilada le proporciona la mayor parte del código de Bicep que necesita, pero debe personalizarlo para su archivo de Bicep. Preste especial atención a las diferencias, en cuanto a parámetros y variables, que existen entre el archivo de Bicep y la plantilla de Bicep exportada. Obviamente, el proceso de exportación no conoce los parámetros y las variables que ya ha definido en el archivo de Bicep.

En el ejemplo siguiente se muestran los elementos agregados al archivo de Bicep. Contiene el código exportado y algunos cambios. En primer lugar, cambia el nombre del parámetro para que coincida con su convención de nomenclatura. En segundo lugar, usa el parámetro de ubicación para la ubicación del plan de App Service. En tercer lugar, elimina algunas de las propiedades en las que el valor predeterminado es correcto.

Copie el archivo completo y reemplace el archivo de Bicep por su contenido.

:::code language="bicep" source="~/resourcemanager-templates/get-started-with-templates/export-template/azuredeploy.bicep" range="1-53" highlight="18,34-51":::

## <a name="deploy-bicep-file"></a>Implementación de un archivo de Bicep

Use la CLI de Azure o Azure PowerShell para implementar un archivo de Bicep.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](bicep-tutorial-create-first-bicep.md#create-resource-group). En el ejemplo se supone que ha establecido la variable `bicepFile` en la ruta de acceso al archivo de Bicep, como se muestra en el [primer tutorial](bicep-tutorial-create-first-bicep.md#deploy-bicep-file).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para ejecutar este cmdlet de implementación, debe tener la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addappserviceplan `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $bicepFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.

```azurecli
az deployment group create \
  --name addappserviceplan \
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
1. El grupo de recursos contiene una cuenta de almacenamiento y un plan de App Service.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a pasar al siguiente tutorial, no es necesario que elimine el grupo de recursos.

Si va a terminar ya, puede eliminar el grupo de recursos para limpiar los recursos que ha implementado.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a exportar una plantilla JSON desde Azure Portal, a convertir la plantilla JSON en un archivo de Bicep y a usar la plantilla exportada para el desarrollo de Bicep. También puede usar las plantillas de inicio rápido de Azure para simplificar el desarrollo con Bicep.

> [!div class="nextstepaction"]
> [Uso de las plantillas de inicio rápido de Azure](bicep-tutorial-quickstart-template.md)
