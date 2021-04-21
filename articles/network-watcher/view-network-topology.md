---
title: Visualización de una topología de redes virtuales de Azure | Microsoft Docs
description: Aprenda a ver los recursos en una red virtual y las relaciones entre ellos.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2018
ms.author: damendo
ms.openlocfilehash: f20fa22dac3fba4d01cbc5e398bafa4113e94a96
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780306"
---
# <a name="view-the-topology-of-an-azure-virtual-network"></a>Visualización de la topología de una red virtual de Azure

En este artículo, aprenderá a ver los recursos de una red virtual de Microsoft Azure y las relaciones entre ellos. Por ejemplo, una red virtual contiene subredes. Las subredes contienen recursos, como máquinas virtuales de Azure. Las máquinas virtuales pueden tener una o varias interfaces de red. Cada subred puede tener un grupo de seguridad de red y una tabla de rutas asociada a él. La funcionalidad de topología de Azure Network Watcher le permite ver todos los recursos de una red virtual, los recursos asociados a los recursos de una red virtual y las relaciones entre los recursos.

Puede usar [Azure Portal](#azure-portal), la [CLI de Azure](#azure-cli) o [PowerShell](#powershell) para ver una topología.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="view-topology---azure-portal"></a><a name = "azure-portal"></a>Visualización de topología: Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta que disponga de los [permisos](required-rbac-permissions.md) necesarios.
2. En la esquina superior izquierda del portal, seleccione **Todos los servicios**.
3. En el cuadro de filtro **Todos los servicios** escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados, selecciónela.
4. Seleccione **Topología**. La generación de una topología requiere una instancia de Network Watcher en la misma región en la que existe la red virtual para la que desea generar la topología. Si no tiene habilitada una instancia de Network Watcher en la región en la que se encuentra la red virtual para la que desea generar una topología, se crean automáticamente instancias de Network Watcher en todas las regiones. Las instancias de Network Watcher se crean en un grupo de recursos denominado **NetworkWatcherRG**.
5. Seleccione una suscripción, el grupo de recursos de una red virtual para la que desea ver la topología y, después, seleccione la red virtual. En la siguiente imagen, se muestra una topología para una red virtual denominada *MyVnet*, en el grupo de recursos denominado *MyResourceGroup*:

    ![Visualización de la topología](./media/view-network-topology/view-topology.png)

    Como puede ver en la figura anterior, la red virtual contiene tres subredes. Una subred tiene una máquina virtual implementada. La máquina virtual tiene una interfaz de red asociada a él y una dirección IP pública asociadas también. Las otras dos subredes tienen una tabla de rutas asociada a ellas. Cada tabla de rutas contiene dos rutas. Una subred dispone de un grupo de seguridad de red asociado a ella. Solo se muestra la información de topología para recursos que se encuentran:
    
    - Dentro del mismo grupo de recursos y región que la red virtual *myVnet*. Por ejemplo, no se muestra un grupo de seguridad de red que existe en un grupo de recursos distinto de *MyResourceGroup*, incluso si el grupo de seguridad de red está asociado a una subred en la red virtual *MyVnet*.
    - Dentro, o asociados a recursos que se encuentran dentro, de la red virtual *myVnet*. Por ejemplo, un grupo de seguridad de red que no esté asociado a una subred o interfaz de red en la red virtual *myVnet* no se muestra, incluso si el grupo de seguridad de red está en el grupo de recursos *MyResourceGroup*.

   La topología mostrada en la imagen es para la red virtual creada después de implementar el **ejemplo de script para el enrutamiento del tráfico mediante una aplicación virtual de red**, que puede implementar con la [CLI de Azure](../virtual-network/scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) o [PowerShell](../virtual-network/scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

6. Seleccione **Descargar topología** para descargar la imagen como archivo editable, en formato svg.

Los recursos mostrados en el diagrama son un subconjunto de los componentes de red en la red virtual. Por ejemplo, mientras se muestra un grupo de seguridad de red, las reglas de seguridad dentro de él no se muestran en el diagrama. Aunque no se diferencian en el diagrama, las líneas representan una de dos relaciones: *independencia* o *asociado*. Para ver la lista completa de recursos en la red virtual y el tipo de relación entre los recursos, genere la topología con [PowerShell ](#powershell) o la [CLI de Azure](#azure-cli).

## <a name="view-topology---azure-cli"></a><a name = "azure-cli"></a>Visualización de la topología: CLI de Azure

Puede ejecutar los comandos en los pasos siguientes:
- En Azure Cloud Shell, mediante la selección **Pruébelo** en la parte superior derecha de cualquier comando. Azure Cloud Shell es un shell interactivo gratis que tiene herramientas comunes de Azure preinstaladas y configuradas para usar con la cuenta.
- Mediante la ejecución de la CLI desde el equipo. Si ejecuta la CLI desde su equipo, los pasos de este artículo requieren la versión 2.0.31 o posterior de la CLI de Azure. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.

La cuenta que utilice debe tener los [permisos](required-rbac-permissions.md) necesarios.

1. Si ya tiene una instancia de Network Watcher en la misma región que la red virtual para la que desea crear una topología, vaya al paso 3. Cree un grupo de recursos que contenga una instancia de Network Watcher con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos en la región *eastus*:

    ```azurecli-interactive
    az group create --name NetworkWatcherRG --location eastus
    ```

2. Cree una instancia de Network Watcher con [az network watcher configure](/cli/azure/network/watcher#az_network_watcher_configure). En el ejemplo siguiente, se crea una instancia de Network Watcher en la región *eastus*:

    ```azurecli-interactive
    az network watcher configure \
      --resource-group NetworkWatcherRG \
      --location eastus \
      --enabled true
    ```

3. Vea la topología con [az network watcher show-topology](/cli/azure/network/watcher#az_network_watcher_show_topology). En el ejemplo siguiente se ve la topología de un grupo de recursos denominado *myResourceGroup*:

    ```azurecli-interactive
    az network watcher show-topology --resource-group MyResourceGroup
    ```

    La información de topología solo se devuelve para los recursos que están dentro del mismo grupo de recursos que el grupo de recursos *MyResourceGroup* y la misma región que la instancia de Network Watcher. Por ejemplo, no se muestra un grupo de seguridad de red que existe en un grupo de recursos distinto de *MyResourceGroup*, incluso si el grupo de seguridad de red está asociado a una subred en la red virtual *MyVnet*.

   Obtenga más información sobre las relaciones y [propiedades](#properties) en la salida devuelta. Si no tiene una red virtual existente para ver una topología, puede crear una utilizando el ejemplo de script [para el enrutamiento del tráfico mediante una aplicación virtual de red](../virtual-network/scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Para ver un diagrama de la topología y descargar en un archivo editable, use el [portal](#azure-portal).

## <a name="view-topology---powershell"></a><a name = "powershell"></a>Visualización de topología: PowerShell

Puede ejecutar los comandos en los pasos siguientes:
- En Azure Cloud Shell, mediante la selección **Pruébelo** en la parte superior derecha de cualquier comando. Azure Cloud Shell es un shell interactivo gratis que tiene herramientas comunes de Azure preinstaladas y configuradas para usar con la cuenta.
- Mediante la ejecución de PowerShell en el equipo. Si ejecuta PowerShell desde el equipo, se necesita el módulo `Az` de Azure PowerShell para este artículo. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

La cuenta que utilice debe tener los [permisos](required-rbac-permissions.md) necesarios.

1. Si ya tiene una instancia de Network Watcher en la misma región que la red virtual para la que desea crear una topología, vaya al paso 3. Cree un grupo de recursos que contenga una instancia de Network Watcher con [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). En el ejemplo siguiente, se crea un grupo de recursos en la región *eastus*:

    ```azurepowershell-interactive
    New-AzResourceGroup -Name NetworkWatcherRG -Location EastUS
    ```

2. Cree una instancia de Network Watcher con [New-AzNetworkWatcher](/powershell/module/az.network/new-aznetworkwatcher). En el ejemplo siguiente, se crea una instancia de Network Watcher en la región eastus:

    ```azurepowershell-interactive
    New-AzNetworkWatcher `
      -Name NetworkWatcher_eastus `
      -ResourceGroupName NetworkWatcherRG
    ```

3. Recupere una instancia de Network Watcher con [Get-AzNetworkWatcher](/powershell/module/az.network/get-aznetworkwatcher). En el ejemplo siguiente se recupera una instancia de Network Watcher en la región Este de EE. UU.:

    ```azurepowershell-interactive
    $nw = Get-AzResource `
      | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "EastUS" }
    $networkWatcher = Get-AzNetworkWatcher `
      -Name $nw.Name `
      -ResourceGroupName $nw.ResourceGroupName
    ```

4. Recupere una topología con [Get-AzNetworkWatcherTopology](/powershell/module/az.network/get-aznetworkwatchertopology). En el siguiente ejemplo, se recupera una topología para una red virtual en el grupo de recursos denominada *MyResourceGroup*:

    ```azurepowershell-interactive
    Get-AzNetworkWatcherTopology `
      -NetworkWatcher $networkWatcher `
      -TargetResourceGroupName MyResourceGroup
    ```

   La información de topología solo se devuelve para los recursos que están dentro del mismo grupo de recursos que el grupo de recursos *MyResourceGroup* y la misma región que la instancia de Network Watcher. Por ejemplo, no se muestra un grupo de seguridad de red que existe en un grupo de recursos distinto de *MyResourceGroup*, incluso si el grupo de seguridad de red está asociado a una subred en la red virtual *MyVnet*.

   Obtenga más información sobre las relaciones y [propiedades](#properties) en la salida devuelta. Si no tiene una red virtual existente para ver una topología, puede crear una utilizando el ejemplo de script [para el enrutamiento del tráfico mediante una aplicación virtual de red](../virtual-network/scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Para ver un diagrama de la topología y descargar en un archivo editable, use el [portal](#azure-portal).

## <a name="relationships"></a>Relaciones

Todos los recursos devueltos en una topología tienen uno de los siguientes tipos de relación con otro recurso:

| Tipo de relación | Ejemplo                                                                                                |
| ---               | ---                                                                                                    |
| Containment       | Una red virtual contiene subredes. Una subred contiene una interfaz de red.                            |
| Asociado        | Una interfaz de red está asociada a una máquina virtual. Una dirección IP pública está asociada a una interfaz de red. |

## <a name="properties"></a>Propiedades

Todos los recursos que se devuelven en una topología tienen las siguientes propiedades:

- **Name**: nombre del recurso.
- **Id**: identificador URI del recurso.
- **Location**: región de Azure en la que existe el recurso.
- **Associations**: lista de asociaciones para el objeto al que se hace referencia. Cada asociación tiene las siguientes propiedades:
    - **AssociationType**: hace referencia a la relación entre los objetos secundario y principal. Los valores válidos son *Contains* o *Associated*.
    - **Name**: nombre del recurso al que se hace referencia.
    - **ResourceId**: identificador URI del recurso al que se hace referencia en la asociación.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [diagnosticar un problema de filtrado de tráfico de red hacia o desde una máquina virtual](diagnose-vm-network-traffic-filtering-problem.md) mediante la funcionalidad de comprobación de flujo IP de Network Watcher.
- Aprenda a [diagnosticar un problema de enrutamiento de tráfico de red desde una máquina virtual](diagnose-vm-network-routing-problem.md) mediante la funcionalidad de próximo salto de Network Watcher.
