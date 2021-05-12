---
title: Creación de un panel de Azure Portal mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un panel de Azure Portal mediante una plantilla de Azure Resource Manager.
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 03/15/2021
ms.openlocfilehash: 3cb6069b13246e561de85c7a392283a5c243f1a4
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517791"
---
# <a name="quickstart-create-a-dashboard-in-the-azure-portal-by-using-an-arm-template"></a>Inicio rápido: Creación de un panel en el Azure Portal mediante una plantilla de ARM

En Azure Portal, un panel es una vista organizada de los recursos en la nube. Este inicio rápido se centra en el proceso de implementación de una plantilla de Azure Resource Manager para crear un panel. El panel muestra el rendimiento de una máquina virtual y algunos vínculos e información estática.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.portal%2Fazure-portal-dashboard%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Una máquina virtual existente.

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

El panel que se crea en la siguiente parte de este inicio rápido requiere que haya una máquina virtual. Cree una máquina virtual siguiendo estos pasos.

1. En Azure Portal, seleccione **Cloud Shell**.

    ![Seleccione Cloud Shell en la cinta de Azure Portal](media/quick-create-template/cloud-shell.png)

1. En la ventana **Cloud Shell**, seleccione **PowerShell**.

    ![Selección de PowerShell en la ventana de terminal](media/quick-create-template/powershell.png)

1. Copie el comando siguiente y escríbalo en el símbolo del sistema para crear un grupo de recursos.

    ```powershell
    New-AzResourceGroup -Name SimpleWinVmResourceGroup -Location EastUS
    ```

    ![Copia de un comando en el símbolo del sistema](media/quick-create-template/command-prompt.png)

1. Copie el comando siguiente y escríbalo en el símbolo del sistema para crear una máquina virtual en el grupo de recursos.

    ```powershell
    New-AzVm `
        -ResourceGroupName "SimpleWinVmResourceGroup" `
        -Name "SimpleWinVm" `
        -Location "East US" 
    ```

1. Escriba un nombre de usuario y una contraseña para la máquina virtual. Se trata de un nombre de usuario y una contraseña nuevos; no es, por ejemplo, la cuenta que usa para iniciar sesión en Azure. Para más información, consulte los [requisitos de nombre de usuario](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm) y los [requisitos de contraseña](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

    Ahora se inicia la implementación de la máquina virtual y normalmente tarda unos minutos en completarse. Una vez finalizada la implementación, pase a la sección siguiente.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-azure-portal-dashboard/). La plantilla de este artículo es demasiado larga para mostrarse aquí. Para ver la plantilla, consulte [azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.portal/azure-portal-dashboard/azuredeploy.json). En la plantilla se define un recurso de Azure, [Microsoft.portal/dashboards](/azure/templates/microsoft.portal/dashboards). Cree un panel en Azure Portal.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.portal%2Fazure-portal-dashboard%2Fazuredeploy.json)

1. Seleccione o escriba los siguientes valores y, a continuación, seleccione **Revisar y crear**.

    ![ARM template, create dashboard, deploy portal](media/quick-create-template/create-dashboard-using-template-portal.png)

    A menos que se especifique otra cosa, use los valores predeterminados para crear el panel.

    * **Suscripción**: seleccione una suscripción de Azure.
    * **Grupo de recursos**: seleccione **SimpleWinVmResourceGroup**.
    * **Ubicación**: seleccione **Este de EE. UU**.
    * **Nombre de máquina virtual**: escriba **SimpleWinVm**.
    * **Grupo de recursos de máquina virtual**: escriba **SimpleWinVmResourceGroup**.

1. Seleccione **Crear** o **Comprar**. Una vez implementado correctamente el panel, recibirá una notificación:

    ![ARM template, create dashboard, deploy portal notification](media/quick-create-template/resource-manager-template-portal-deployment-notification.png)

Se ha usado Azure Portal para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

[!INCLUDE [azure-portal-review-deployed-resources](../../includes/azure-portal-review-deployed-resources.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar la máquina virtual y el panel asociado, elimine el grupo de recursos que los contiene.

1. En el Azure Portal, busque **SimpleWinVmResourceGroup** y selecciónelo en los resultados de la búsqueda.

1. En la página **SimpleWinVmResourceGroup**, seleccione **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos para confirmar y, a continuación, seleccione **Eliminar**.

    ![Eliminación de un grupo de recursos](media/quick-create-template/delete-resource-group.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de los paneles en Azure Portal, consulte:

> [!div class="nextstepaction"]
> [Creación y uso compartido de paneles en Azure Portal](azure-portal-dashboards.md)
