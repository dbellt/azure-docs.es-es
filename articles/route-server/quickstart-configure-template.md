---
title: 'Inicio rápido: Creación de una instancia de Azure Route Server mediante una plantilla de Azure Resource Manager (plantilla de ARM)'
description: En este inicio rápido se muestra cómo crear un circuito de Azure Route Server mediante una plantilla de Azure Resource Manager.
services: route-server
author: duongau
ms.service: route-server
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 04/05/2021
ms.author: duau
ms.openlocfilehash: 3476e5fa2c274f0fc2c180711480375b0ebefaf2
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388047"
---
# <a name="quickstart-create-an-azure-route-server-using-an-arm-template"></a>Inicio rápido: Creación de una instancia de Azure Route Server mediante una plantilla de Resource Manager

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para implementar una instancia de Azure Route Server en una red virtual nueva o existente.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-route-server%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-route-server).

En este inicio rápido, implementará una instancia de Azure Route Server en una red virtual nueva o existente. Se creará una subred dedicada denominada `RouteServerSubnet` para hospedar la instancia de Route Server. Route Server también se configurará con el ASN del mismo nivel y la dirección IP del mismo nivel para establecer un emparejamiento BGP.

:::code language="json" source="~/quickstart-templates/101-route-server/azuredeploy.json" range="001-145" highlight="105-142":::

En la plantilla se han definido varios recursos de Azure:

* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks)
* [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/microsoft.network/virtualNetworks/subnets) (dos subredes, una denominada `routeserversubnet`)
* [**Microsoft.Network/virtualHubs**](/azure/templates/microsoft.network/virtualhubs) (implementación de Route Server)
* [**Microsoft.Network/virtualHubs/ipConfigurations**](/azure/templates/microsoft.network/virtualhubs/ipConfigurations)
* [**Microsoft.Network/virtualHubs/bgpConnections**](/azure/templates/microsoft.network/virtualhubs/bgpconnections) (Configuración de ASN y dirección IP del mismo nivel)


Para encontrar más plantillas relacionadas con ExpressRoute, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione **Try It** (Probarlo) en el bloque de código siguiente para abrir Azure Cloud Shell y siga las instrucciones para iniciar sesión en Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-route-server/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Espere hasta que vea el aviso de la consola.

1. Seleccione **Copiar** en el bloque de código anterior para copiar el script de PowerShell.

1. Haga clic con el botón derecho en el panel de consola del shell y, a continuación, seleccione **Pegar**.

1. Escriba los valores.

    El nombre del grupo de recursos es el nombre del proyecto con **rg** anexado.

    La plantilla tarda aproximadamente 20 en implementarse. Al finalizar, la salida es parecida a esta:

    :::image type="content" source="./media/quickstart-configure-template/powershell-output.png" alt-text="Salida de la implementación de PowerShell de la plantilla de Resource Manager para Route Server.":::

Azure PowerShell se usa para implementar la plantilla. Además de Azure PowerShell, también puede usar Azure Portal, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Validación de la implementación

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Grupos de recursos** en el panel izquierdo.

1. Seleccione el grupo de recursos que creó en la sección anterior. El nombre del grupo de recursos predeterminado es el nombre del proyecto con **rg** anexado.

1. El grupo de recursos debe contener solo la red virtual:

     :::image type="content" source="./media/quickstart-configure-template/resource-group.png" alt-text="Grupo de recursos de implementación de Route Server con la red virtual.":::

1. Ir a https://aka.ms/routeserver.

1. Seleccione la instancia de Route Server denominada **routeserver** para comprobar que la implementación se realizó correctamente.

    :::image type="content" source="./media/quickstart-configure-template/deployment.png" alt-text="Captura de pantalla de la página de información general del servidor de rutas.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con Route Server, elimine el grupo de recursos. Esta acción elimina la instancia de Route Server y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:

* Route Server
* Virtual Network
* Subnet

Después de crear la instancia de Azure Route Server, continúe para obtener información sobre cómo interactúa Azure Route Server con las puertas de enlace de VPN y ExpressRoute: 

> [!div class="nextstepaction"]
> [Compatibilidad de Azure Route Server (versión preliminar) con ExpressRoute y VPN de Azure](expressroute-vpn-support.md)
