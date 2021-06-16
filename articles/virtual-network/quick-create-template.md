---
title: 'Inicio rápido: Creación de una red virtual mediante una plantilla de Resource Manager'
titleSuffix: Azure Virtual Network
description: Aprenda a usar una plantilla de Resource Manager para crear una red virtual de Azure.
services: virtual-network
author: KumudD
ms.service: virtual-network
ms.topic: quickstart
ms.date: 06/09/2021
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a686eacf8e3caa162cfca765e60d2268fe2066e7
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078085"
---
# <a name="quickstart-create-a-virtual-network---resource-manager-template"></a>Inicio rápido: Creación de una red virtual: plantilla de Resource Manager

En este inicio rápido aprenderá a crear una red virtual con dos subredes mediante la plantilla de Resource Manager. Una red virtual es el bloque de creación básico de una red privada en Azure. Permite que los recursos de Azure, como las máquinas virtuales, se comuniquen de manera segura entre sí y con Internet.


[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

También se puede completar este inicio rápido mediante [Azure Portal](quick-create-portal.md), [Azure PowerShell](quick-create-powershell.md) o la [CLI de Azure](quick-create-cli.md).

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.network/vnet-two-subnets/azuredeploy.json).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.network/vnet-two-subnets/azuredeploy.json" range="001-98" highlight="56-97":::

En la plantilla se han definido los siguientes recursos de Azure:
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks): cree una red virtual de Azure.
-  [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/microsoft.network/virtualnetworks/subnets): cree una subred.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una red virtual con dos subredes.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.network%2Fvnet-two-subnets%2Fazuredeploy.json)

2. En el portal, en la página **Create a Virtual Network with two Subnets** (Crear una red virtual con dos subredes), escriba o seleccione los valores siguientes:
   - **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre para el grupo de recursos y seleccione **Aceptar**.
   - **Nombre de la red virtual**: escriba el nombre de la nueva red virtual.
3. Seleccione **Revisar y crear** y, luego, **Crear**.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Explore los recursos que se crearon con la red virtual.

Para obtener información sobre la sintaxis y las propiedades de JSON de una red virtual en una plantilla, consulte [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con la red virtual, elimine el grupo de recursos. Así se quitan tanto la red virtual como todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, ha implementado una red virtual de Azure con dos subredes. Para más información sobre las redes virtuales de Azure, pase al tutorial de redes virtuales.

> [!div class="nextstepaction"]
> [Filtrado del tráfico de red](tutorial-filter-network-traffic.md)
