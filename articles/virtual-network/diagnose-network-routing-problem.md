---
title: Diagnóstico de un problema de enrutamiento de máquinas virtuales de Azure | Microsoft Docs
description: Aprenda a diagnosticar un problema de enrutamiento en una máquina virtual visualizando sus rutas eficaces.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/30/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d7b8b04093bbec23dd080802876358c1b1af33e8
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110675878"
---
# <a name="diagnose-a-virtual-machine-routing-problem"></a>Diagnóstico de problemas de enrutamiento en una máquina virtual

En este artículo, aprenderá a diagnosticar un problema de enrutamiento mediante la visualización de las rutas que son eficaces para una interfaz de red en una máquina virtual (VM). Azure crea varias rutas predeterminadas para cada subred de red virtual. Puede invalidar las rutas predeterminadas de Azure mediante la definición de rutas en una tabla de rutas y la asociación de dicha tabla a una subred. La combinación de rutas que cree, las rutas predeterminadas de Azure y las rutas que se propagan desde la red local a través de una puerta de enlace de VPN de Azure (si la red virtual está conectada a la red local) mediante el protocolo de puerta de enlace de borde (BGP), son la rutas eficaces para todas las interfaces de red de una subred. Si no está familiarizado con los conceptos de red virtual, interfaz de red o enrutamiento, consulte [Introducción a la red virtual](virtual-networks-overview.md), [Interfaz de red](virtual-network-network-interface.md) e [Introducción al enrutamiento](virtual-networks-udr-overview.md).

## <a name="scenario"></a>Escenario

Intenta conectarse a una máquina virtual, pero se produce un error en la conexión. Para determinar por qué no puede conectarse a la máquina virtual, puede ver las rutas eficaces para una interfaz de red mediante [Azure Portal](#diagnose-using-azure-portal), [PowerShell](#diagnose-using-powershell) o la [CLI de Azure](#diagnose-using-azure-cli).

En los pasos que vienen a continuación se da por hecho que tiene una máquina virtual para la que ver las rutas eficaces. Si no tiene una máquina virtual, implemente primero una máquina virtual [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para completar las tareas de este artículo. Los ejemplos de este artículo son para una VM denominada *myVM* con una interfaz de red llamada *myVMNic1*. La máquina virtual y la interfaz de red están en un grupo de recursos llamado *myResourceGroup* y en la región *Este de EE. UU*. Cambie los valores de los pasos, según corresponda, por los de la máquina virtual cuyos problemas va a diagnosticar.

## <a name="diagnose-using-azure-portal"></a>Diagnóstico mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de Azure que disponga de los [permisos necesarios](virtual-network-network-interface.md#permissions).
2. En la parte superior de Azure Portal, vaya al cuadro de búsqueda y escriba el nombre de una máquina virtual que se encuentre en estado de ejecución. Cuando el nombre de la máquina virtual aparezca en los resultados de búsqueda, selecciónela.
3. En **Configuración** a la izquierda, seleccione **Redes** y navegue hasta el recurso de interfaz de red seleccionando su nombre.
     ![Visualización de las interfaces de red](./media/diagnose-network-routing-problem/view-nics.png)
4. A la izquierda, seleccione **Rutas eficaces**. Se muestran las rutas eficaces de una interfaz de red llamada **myVMNic1**, como se ilustra en la imagen siguiente: ![Visualización de las rutas eficaces](./media/diagnose-network-routing-problem/view-effective-routes.png)

    Si hay varias interfaces de red asociadas a la máquina virtual, puede ver las rutas eficaces de cada interfaz de red; para ello, basta con seleccionarla. Puesto que cada interfaz de red puede estar en una subred diferente, también puede tener cada una diferentes rutas eficaces.

    En el ejemplo mostrado en la imagen anterior, las rutas enumeradas son rutas predeterminadas que crea Azure para cada subred. Su lista tiene como mínimo estas rutas, pero puede tener rutas adicionales, según las funcionalidades que pueda haber habilitado para la red virtual; por ejemplo, estar emparejada con otra red virtual o conectada a la red local mediante una puerta de enlace VPN de Azure. Para más información sobre cada una de las rutas y otras rutas que puede ver para la red virtual, consulte [Enrutamiento del tráfico de redes virtuales](virtual-networks-udr-overview.md). Si la lista tiene un gran número de rutas, quizá le resulte más fácil seleccionar **Descargar** para descargar un archivo .csv con la lista de rutas.

Aunque las rutas eficaces se vieron mediante la máquina virtual en los pasos anteriores, también puede verlas mediante una:
- **Interfaz de red individual**: aprenda a [ver una interfaz de red](virtual-network-network-interface.md#view-network-interface-settings).
- **Tabla de rutas individual**: aprenda cómo [ver una tabla de rutas](manage-route-table.md#view-details-of-a-route-table).

## <a name="diagnose-using-powershell"></a>Diagnóstico mediante PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/powershell), o mediante la ejecución de PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta PowerShell desde el equipo, necesita el módulo Azure PowerShell versión 1.0.0 o posterior. Ejecute `Get-Module -ListAvailable Az` en el equipo para encontrar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si ejecuta PowerShell localmente, también debe ejecutar `Connect-AzAccount` para iniciar sesión en Azure con una cuenta que tenga los [permisos necesarios](virtual-network-network-interface.md#permissions).

Obtenga rutas eficaces para una interfaz de red con [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable). En el ejemplo siguiente se obtienen las rutas eficaces de una interfaz de red llamada *myVMNic1*, que se encuentra en un grupo de recursos denominado *myResourceGroup*:

```azurepowershell-interactive
Get-AzEffectiveRouteTable `
  -NetworkInterfaceName myVMNic1 `
  -ResourceGroupName myResourceGroup `
  | Format-Table
```

Para entender la información devuelta en la salida, consulte [Introducción al enrutamiento](virtual-networks-udr-overview.md). Solo se devuelve la salida si la máquina virtual está en estado de ejecución. Si hay varias interfaces de red asociadas a la máquina virtual, puede revisar las rutas eficaces para cada interfaz de red. Puesto que cada interfaz de red puede estar en una subred diferente, también puede tener cada una diferentes rutas eficaces. Si sigue teniendo problemas de comunicación, consulte las secciones [Diagnóstico adicional](#additional-diagnosis) y [Consideraciones](#considerations).

Si no conoce el nombre de una interfaz de red, pero conoce el nombre de la máquina virtual a la que está asociada la interfaz de red, los comandos siguientes devuelven los identificadores de todas las interfaces de red asociadas a una máquina virtual:

```azurepowershell-interactive
$VM = Get-AzVM -Name myVM `
  -ResourceGroupName myResourceGroup
$VM.NetworkProfile
```

Recibirá una salida similar a la del ejemplo siguiente:

```powershell
NetworkInterfaces
-----------------
{/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMNic1
```

En la salida anterior, el nombre de la interfaz de red es *myVMNic1*.

## <a name="diagnose-using-azure-cli"></a>Diagnóstico mediante la CLI de Azure

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/bash), o mediante la ejecución de la CLI en el equipo. En este artículo se requiere la CLI de Azure versión 2.0.32 o posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta la CLI de Azure localmente, también debe ejecutar `az login` e iniciar sesión en Azure con una cuenta que tenga los [permisos necesarios](virtual-network-network-interface.md#permissions).

Obtenga las rutas eficaces de una interfaz de red con [az network nic show-effective-route-table](/cli/azure/network/nic#az_network_nic_show_effective_route_table). En el ejemplo siguiente se obtienen las rutas eficaces de una interfaz de red llamada *myVMNic1*, que se encuentra en un grupo de recursos denominado *myResourceGroup*:

```azurecli-interactive
az network nic show-effective-route-table \
  --name myVMNic1 \
  --resource-group myResourceGroup
```

Para entender la información devuelta en la salida, consulte [Introducción al enrutamiento](virtual-networks-udr-overview.md). Solo se devuelve la salida si la máquina virtual está en estado de ejecución. Si hay varias interfaces de red asociadas a la máquina virtual, puede revisar las rutas eficaces para cada interfaz de red. Puesto que cada interfaz de red puede estar en una subred diferente, también puede tener cada una diferentes rutas eficaces. Si sigue teniendo problemas de comunicación, consulte las secciones [Diagnóstico adicional](#additional-diagnosis) y [Consideraciones](#considerations).

Si no conoce el nombre de una interfaz de red, pero conoce el nombre de la máquina virtual a la que está asociada la interfaz de red, los comandos siguientes devuelven los identificadores de todas las interfaces de red asociadas a una máquina virtual:

```azurecli-interactive
az vm show \
  --name myVM \
  --resource-group myResourceGroup
```

## <a name="resolve-a-problem"></a>Resolución de un problema

La resolución de problemas de enrutamiento supone normalmente:

- Agregar una ruta personalizada para invalidar una de las rutas predeterminadas de Azure. Aprenda cómo [agregar una ruta personalizada](manage-route-table.md#create-a-route).
- Cambiar o quitar una ruta personalizada que puede provocar el enrutamiento a una ubicación no deseada. Aprenda cómo [cambiar](manage-route-table.md#change-a-route) o [eliminar](manage-route-table.md#delete-a-route) una ruta personalizada.
- Asegurarse de que la tabla de rutas que contiene las rutas personalizadas que ha definido está asociada a la subred en la que se encuentra la interfaz de red. Aprenda cómo [asociar una tabla de rutas a una subred](manage-route-table.md#associate-a-route-table-to-a-subnet).
- Asegurarse de que los dispositivos, como las aplicaciones virtuales de red o de puerta de enlace de VPN que ha implementado se pueden usar. Use la funcionalidad [Diagnósticos de VPN](../network-watcher/diagnose-communication-problem-between-networks.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Network Watcher para determinar los problemas con una puerta de enlace de VPN de Azure.

Si sigue teniendo problemas de comunicación, consulte las secciones [Consideraciones](#considerations) y Diagnóstico adicional.

## <a name="considerations"></a>Consideraciones

Tenga en cuenta los puntos siguientes cuando tenga que solucionar problemas de comunicación:

- El enrutamiento se basa en la coincidencia con el prefijo más largo (LPM) entre las rutas que ha definido, el protocolo de puerta de enlace de borde (BGP) y las rutas del sistema. Si hay más de una ruta con la misma coincidencia LPM, la ruta se selecciona entonces en función de su origen en el orden enumerado en [Introducción al enrutamiento](virtual-networks-udr-overview.md#how-azure-selects-a-route). En el caso de las rutas eficaces, solo puede ver aquellas que tienen una coincidencia LPM basada en todas las rutas disponibles. Ver cómo se evalúan las rutas de una interfaz de red facilita en gran medida la solución de problemas con rutas específicas que podrían estar afectando a la comunicación desde la máquina virtual.
- Si ha definido rutas personalizadas a una aplicación virtual de red (NVA), con *Aplicación virtual* como el siguiente tipo de salto, asegúrese de que el reenvío de direcciones IP esté habilitado en la NVA que recibe el tráfico o se perderán los paquetes. Aprenda más sobre cómo [habilitar el reenvío de direcciones IP para una interfaz de red](virtual-network-network-interface.md#enable-or-disable-ip-forwarding). Además, el sistema operativo o la aplicación de la NVA también deben poder reenviar el tráfico de red y se deben configurar para ello.
- Si ha creado una ruta a 0.0.0.0/0, todo el tráfico saliente de Internet se enruta al próximo salto que especifique, por ejemplo, a una NVA o una puerta de enlace de VPN. La creación de una ruta de este tipo se conoce a veces como tunelización forzada. Puede que las conexiones remotas que usan los protocolos RDP o SSH desde Internet hasta la máquina virtual no funcionen con esta ruta, en función de cómo el próximo salto administre el tráfico. Puede habilitar la tunelización forzada:
    - Cuando se usa VPN de sitio a sitio, mediante la creación de una ruta con un tipo de próximo salto de *Puerta de enlace de VPN*. Aprenda más sobre [cómo configurar la tunelización forzada](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
    - Si 0.0.0.0/0 (ruta predeterminada) se anuncia sobre BGP mediante una puerta de enlace de red virtual al usar VPN de sitio a sitio o un circuito ExpressRoute. Aprenda más sobre el uso de BGP con una [VPN de sitio a sitio](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [ExpressRoute](../expressroute/expressroute-routing.md?toc=%2fazure%2fvirtual-network%2ftoc.json#ip-addresses-used-for-azure-private-peering).
- Para que el tráfico de emparejamiento de red virtual funcione correctamente, debe existir una ruta del sistema con un tipo de próximo salto de *Emparejamiento de VNet* para el intervalo de prefijos de la red virtual emparejada. Si la ruta no existe y el vínculo de emparejamiento de la red virtual es **Conectado**:
    - Espere unos segundos e inténtelo de nuevo. Si es un vínculo de emparejamiento recién establecido, en ocasiones las rutas tardan más tiempo en propagarse a todas las interfaces de red de una subred. Para más información sobre el emparejamiento de redes virtuales, consulte [Introducción al emparejamiento de redes virtuales](virtual-network-peering-overview.md) y [Administración del emparejamientos de redes virtuales](virtual-network-manage-peering.md).
    - Las reglas del grupo de seguridad podrían estar afectando a la comunicación. Para más información, consulte [Diagnose a virtual machine network traffic filter problem](diagnose-network-traffic-filter-problem.md) (Diagnóstico de problemas de filtro del tráfico de red de la máquina virtual).
- Aunque Azure asigna rutas predeterminadas a cada interfaz de red de Azure, si tiene varias interfaces de red asociadas a la máquina virtual, solo a la interfaz de red principal se le asigna una ruta predeterminada (0.0.0.0/0), o puerta de enlace, dentro del sistema operativo de la máquina virtual. Aprenda a crear una ruta predeterminada para las interfaces de red secundarias asociadas a una máquina virtual [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) o [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics). Aprenda más sobre las [interfaces de red principales y secundarias](virtual-network-network-interface-vm.md#constraints).

## <a name="additional-diagnosis"></a>Diagnóstico adicional

* Para ejecutar una prueba rápida con el fin de determinar el tipo de próximo salto para el tráfico destinado a una ubicación, use la funcionalidad [Próximo salto](../network-watcher/diagnose-vm-network-routing-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure Network Watcher. El próximo salto indica cuál es el tipo de próximo salto para el tráfico destinado a una ubicación especificada.
* Si no es una ruta la causante del error de comunicación de red de la máquina virtual, el problema puede deberse a que se está ejecutando software de firewall dentro del sistema operativo de la máquina virtual.
* Si está realizando la [tunelización forzada](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) del tráfico a un dispositivo local mediante una puerta de enlace de VPN o una NVA, es posible que no pueda conectarse a una máquina virtual desde Internet, según cómo haya configurado el enrutamiento en los dispositivos. Confirme que el enrutamiento que ha configurado para el dispositivo enruta el tráfico a una dirección IP pública o privada de la máquina virtual.
* Use la funcionalidad de [solución de problemas de conexión](../network-watcher/network-watcher-connectivity-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Network Watcher para determinar las causas de los problemas de comunicación salientes relacionados con el enrutamiento, el filtrado y el sistema operativo.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda sobre todas las tareas, propiedades y configuraciones de [tablas de rutas y rutas](manage-route-table.md).
- Aprenda sobre todos los [tipos de próximo salto y rutas del sistema, y cómo Azure selecciona las rutas](virtual-networks-udr-overview.md).
