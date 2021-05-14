---
title: Solución de problemas de implementaciones
description: Aprenda a supervisar y solucionar los problemas asociados a las implementaciones de las plantillas de Azure Resource Manager (ARM). Muestra registros de actividad y el historial de implementación.
author: mumian
ms.date: 01/15/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 88898d3a0e42ff362a5ab6fa586c52064fcb8221
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108736746"
---
# <a name="tutorial-troubleshoot-arm-template-deployments"></a>Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager

Aprenda a solucionar errores de implementación de las plantillas de Azure Resource Manager (ARM). En este tutorial, se configuran dos errores en una plantilla y se aprende a usar los registros de actividad y el historial de implementación para resolver los problemas.

Hay dos tipos de errores relacionados con la implementación de plantillas:

- Los **errores de validación** que provienen de escenarios que se pueden determinar antes de la implementación. incluyen los errores de sintaxis en la plantilla o tratar de implementar los recursos que superarían las cuotas de suscripción.
- Los **errores de implementación** que provienen de condiciones que se producen durante el proceso de implementación. Incluyen intentar acceder a un recurso que se está implementando en paralelo.

Los dos tipos de errores devuelven un código de error que utiliza para solucionar problemas de implementación. Ambos aparecen en el registro de actividad. Sin embargo, los errores de validación no aparecen en el historial de implementación porque la implementación nunca se inicia.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> - Creación de una plantilla problemática
> - Solución de errores de validación
> - Solución de errores de implementación
> - Limpieza de recursos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

- Visual Studio Code con la extensión Resource Manager Tools. Consulte [Quickstart: Creación de plantillas de ARM mediante Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="create-a-problematic-template"></a>Creación de una plantilla problemática

Abra una plantilla llamada [Create a standard storage account](https://azure.microsoft.com/resources/templates/101-storage-account-create/) (Crear una cuenta de almacenamiento estándar) desde [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/) y configure dos problemas de plantilla.

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.
2. En **Nombre de archivo**, pegue el código URL siguiente:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json
    ```

3. Seleccione **Abrir** para abrir el archivo.
4. Cambie la línea `apiVersion` por la siguiente:

    ```json
    "apiVersion1": "2018-07-02",
    ```

    - `apiVersion1` es un nombre de elemento no válido. Es un error de validación.
    - La versión de la API será `"2018-07-01"`.  Es un error de implementación.

5. Seleccione **Archivo** > **Guardar como** para guardar el archivo como _azuredeploy.json_ en el equipo local.

## <a name="troubleshoot-the-validation-error"></a>Solución de errores de validación

Consulte la sección [Implementación de la plantilla](template-tutorial-create-multiple-instances.md#deploy-the-template) para implementar la plantilla.

Se recibe un error del shell similar al siguiente:

```azurepowershell
New-AzResourceGroupDeployment : 4:29:24 PM - Error: Code=InvalidRequestContent; Message=The request content was invalid and could not be deserialized: 'Could not find member 'apiVersion1' on object of type 'TemplateResource'. Path 'properties.template.resources[0].apiVersion1', line 36, position 24.'.
```

El mensaje de error indica que el problema es con `apiVersion1`.

Utilice Visual Studio Code para solucionar el problema cambiando `apiVersion1` por `apiVersion` y, después, guarde la plantilla.

## <a name="troubleshoot-the-deployment-error"></a>Solución de errores de implementación

Consulte la sección [Implementación de la plantilla](template-tutorial-create-multiple-instances.md#deploy-the-template) para implementar la plantilla.

Se recibe un error del shell similar al siguiente:

```azurepowershell
New-AzResourceGroupDeployment : 4:48:50 PM - Resource Microsoft.Storage/storageAccounts 'storeqii7x2rce77dc' failed with message '{
  "error": {
    "code": "NoRegisteredProviderFound",
    "message": "No registered resource provider found for location 'centralus' and API version '2018-07-02' for type 'storageAccounts'. The supported api-versions are '2018-07-01, 2018-03-01-preview, 2018-02-01, 2017-10-01, 2017-06-01, 2016-12-01, 2016-05-01, 2016-01-01, 2015-06-15, 2015-05-01-preview'. The supported locations are 'eastus, eastus2, westus, westeurope, eastasia, southeastasia, japaneast, japanwest, northcentralus, southcentralus, centralus, northeurope, brazilsouth, australiaeast, australiasoutheast, southindia, centralindia, westindia, canadaeast, canadacentral, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth, francecentral'."
  }
}'
```

El error de implementación se puede encontrar desde Azure Portal mediante el procedimiento siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Abra el grupo de recursos seleccionando **Grupos de recursos** y, después, el nombre del grupo de recursos. Verá **1 Failed** (1 error) en **Implementación**.

    ![Captura de pantalla que resalta la implementación con errores.](./media/template-tutorial-troubleshoot/resource-manager-template-deployment-error.png)
3. Seleccione **Detalles del error**.

    ![Captura de pantalla que resalta el vínculo Detalles del error.](./media/template-tutorial-troubleshoot/resource-manager-template-deployment-error-details.png)

    El mensaje de error es el mismo que el que se ha mostrado anteriormente:

    ![Captura de pantalla que muestra los detalles del error.](./media/template-tutorial-troubleshoot/resource-manager-template-deployment-error-summary.png)

El error también se puede encontrar desde los registros de actividad:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Monitor** > **Registro de actividad**.
3. Utilice los filtros para buscar el registro.

    ![Solución del problemas del tutorial de Resource Manager](./media/template-tutorial-troubleshoot/resource-manager-template-deployment-activity-log.png)

Utilice Visual Studio Code para corregir el problema y, después, vuelva a implementar la plantilla.

Para ver una lista de errores comunes, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando los recursos de Azure ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.  Verá un total de seis recursos en el grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a solucionar errores en las implementaciones de plantillas de Resource Manager.  Para más información, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).
