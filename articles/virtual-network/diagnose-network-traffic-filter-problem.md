---
title: Diagnóstico de un problema de filtro del tráfico de red de una máquina virtual | Microsoft Docs
description: Aprenda a diagnosticar un problema de filtro del tráfico de red de una máquina virtual mediante la visualización de las reglas de seguridad vigentes para una máquina virtual.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: a54feccf-0123-4e49-a743-eb8d0bdd1ebc
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2018
ms.author: kumud
ms.openlocfilehash: d6835d06015923a70301c95370c76efbd0c2163e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776742"
---
# <a name="diagnose-a-virtual-machine-network-traffic-filter-problem"></a>Diagnóstico de un problema de filtro del tráfico de red de una máquina virtual

En este artículo, aprenderá a diagnosticar un problema de filtro del tráfico de red mediante la visualización de las reglas de seguridad del grupo de seguridad de red (NSG) que están vigentes para una máquina virtual (VM).

Los NSG le permiten controlar los tipos de tráfico que entran y salen de una máquina virtual. Puede asociar un NSG a una subred de una red virtual de Azure, a una interfaz de red conectada a una máquina virtual, o a ambas. Las reglas de seguridad en vigor aplicadas a una interfaz de red se agregan a las que existen en el NSG asociado a una interfaz de red y la subred en la que se encuentra la interfaz. Las reglas de diferentes NSG a veces pueden entrar en conflicto entre sí y afectar la conectividad de red de la máquina virtual. Puede ver todas las reglas de seguridad vigentes desde los NSG que se aplican a las interfaces de red de la máquina virtual. Si no está familiarizado con los conceptos de red virtual, interfaz de red o NSG, consulte [Introducción a la red virtual](virtual-networks-overview.md), [Interfaz de red](virtual-network-network-interface.md) e [Introducción a los grupos de seguridad de red](./network-security-groups-overview.md).

## <a name="scenario"></a>Escenario

Intenta conectarse a una máquina virtual en el puerto 80 desde Internet, pero se produce un error en la conexión. Para determinar por qué no puede acceder al puerto 80 desde Internet, puede ver las reglas de seguridad vigentes para una interfaz de red mediante [Azure Portal](#diagnose-using-azure-portal), [PowerShell](#diagnose-using-powershell) o la [CLI de Azure](#diagnose-using-azure-cli).

En los pasos que se indican a continuación se da por hecho que tiene una maquina virtual para ver las reglas de seguridad vigentes. Si no tiene una máquina virtual, implemente primero una máquina virtual [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para completar las tareas de este artículo. Los ejemplos de este artículo son para una máquina virtual denominada *myVM* con una interfaz de red llamada *myVMVMNic*. La máquina virtual y la interfaz de red están en un grupo de recursos llamado *myResourceGroup* y en la región *Este de EE. UU*. Cambie los valores de los pasos, según corresponda, por los de la máquina virtual cuyos problemas va a diagnosticar.

## <a name="diagnose-using-azure-portal"></a>Diagnóstico mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de Azure que disponga de los [permisos necesarios](virtual-network-network-interface.md#permissions).
2. En la parte superior de Azure Portal, vaya al cuadro de búsqueda y escriba el nombre de la máquina virtual. Cuando el nombre de la máquina virtual aparezca en los resultados de búsqueda, selecciónela.
3. En **CONFIGURACIÓN**, seleccione **Redes**, como se muestra en la imagen siguiente:

   ![Captura de pantalla que muestra la configuración de Redes de Azure Portal para la interfaz myVMVMNic.](./media/diagnose-network-traffic-filter-problem/view-security-rules.png)

   Las reglas que se ven en la imagen anterior son para una interfaz de red denominada **myVMVMNic**. Verá que hay **REGLAS DE PUERTO DE ENTRADA** para la interfaz de red de dos grupos de seguridad de red diferentes:
   
   - **mySubnetNSG**: asociado a la subred que se encuentra en la interfaz de red.
   - **myVMNSG**: asociado a la interfaz de red de la máquina virtual denominada **myVMVMNic**.

   La regla denominada **DenyAllInBound** es lo que impide la comunicación entrante con la máquina virtual en el puerto 80, desde Internet, tal como se describe en el [escenario](#scenario). La regla enumera *0.0.0.0/0* para **ORIGEN**, lo que incluye Internet. Ninguna otra regla con una prioridad más alta (número menor) permite el puerto 80 de entrada. Para permitir el puerto 80 de entrada a la máquina virtual desde Internet, consulte [Resolve a problem](#resolve-a-problem) (Resolución de un problema). Para más información acerca de las reglas de seguridad y de la forma en que Azure las aplica, consulte [Grupos de seguridad de red](./network-security-groups-overview.md).

   En la parte inferior de la imagen, también ve **REGLAS DEL PUERTO DE SALIDA**. Debajo están las reglas del puerto de salida para la interfaz de red. Aunque la imagen solo muestra cuatro reglas de entrada para cada NSG, sus NSG pueden tener más de cuatro. En la imagen, ve **VirtualNetwork** en **SOURCE** (ORIGEN) y **DESTINATION** (DESTINO) y **AzureLoadBalancer** en **SOURCE** (ORIGEN). **VirtualNetwork** y **AzureLoadBalancer** son [etiquetas de servicio](./network-security-groups-overview.md#service-tags). Las etiquetas de servicio representan un grupo de prefijos de direcciones IP que ayudan a reducir la complejidad de la creación de reglas de seguridad.

4. Asegúrese de que la máquina virtual está en estado de ejecución y luego seleccione **Effective security rules** (Reglas de seguridad eficaces), como se muestra en la imagen anterior, para ver las reglas de seguridad vigentes que se muestran en la siguiente imagen:

   ![Captura de pantalla que muestra el panel Reglas de seguridad vigentes con la opción Descargar selección y la regla de entrada AllowAzureLoadBalancerInbound seleccionada.](./media/diagnose-network-traffic-filter-problem/view-effective-security-rules.png)

   Las reglas que se enumeran son las mismas que vio en el paso 3, aunque hay pestañas diferentes para el grupo de seguridad de red asociado a la interfaz de red y la subred. Como puede ver en la imagen, solo se muestran las 50 primeras reglas. Para descargar un archivo .csv que contiene todas las reglas, seleccione **Descargar**.

   Para ver los prefijos que representa cada etiqueta de servicio, seleccione una regla, por ejemplo, la regla denominada **AllowAzureLoadBalancerInbound**. La siguiente imagen muestra los prefijos de la etiqueta de servicio **AzureLoadBalancer**:

   ![Captura de pantalla que muestra los prefijos de dirección de AllowAzureLoadBalancerInbound especificados.](./media/diagnose-network-traffic-filter-problem/address-prefixes.png)

   Aunque la etiqueta de servicio **AzureLoadBalancer** solo representa un prefijo, otras etiquetas de servicio representan varios prefijos.

5. Los pasos anteriores mostraban las reglas de seguridad de una interfaz de red denominada **myVMVMNic**, pero también ha visto una interfaz de red llamada **myVMVMNic2** en algunas de las imágenes anteriores. La máquina virtual de este ejemplo tiene dos interfaces de red conectadas. Las reglas de seguridad vigentes pueden ser diferentes para cada interfaz de red.

   Para ver las reglas de la interfaz de red **myVMVMNic2**, selecciónela. Como se muestra en la imagen siguiente, la interfaz de red tiene las mismas reglas asociadas a su subred que la interfaz de red **myVMVMNic**, ya que las dos interfaces de red se encuentran en la misma subred. Al asociar un NSG a una subred, sus reglas se aplican a todas las interfaces de red de la subred.

   ![Captura de pantalla que muestra la configuración de Redes de Azure Portal para la interfaz myVMVMNic2.](./media/diagnose-network-traffic-filter-problem/view-security-rules2.png)

   A diferencia de la interfaz de red **myVMVMNic**, la interfaz de red **myVMVMNic2** no tiene ningún grupo de seguridad de red asociado. Cada interfaz de red y subred pueden tener un grupo de seguridad de red asociado, o ninguno. El grupo de seguridad de red asociados a cada interfaz de red o subred puede ser el mismo o diferente. El mismo grupo de seguridad de red se puede asociar a tantas interfaces de red y subredes individuales como se desee.

Aunque las reglas de seguridad vigentes se vieron a través de la máquina virtual, también se puede ver a través de un usuario individual:
- **Interfaz de red**: aprenda a [ver una interfaz de red](virtual-network-network-interface.md#view-network-interface-settings).
- **NSG**: aprenda a [ver un grupo de seguridad de red](manage-network-security-group.md#view-details-of-a-network-security-group).

## <a name="diagnose-using-powershell"></a>Diagnóstico mediante PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/powershell), o mediante la ejecución de PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta PowerShell desde el equipo, necesita el módulo Azure PowerShell versión 1.0.0 o posterior. Ejecute `Get-Module -ListAvailable Az` en el equipo para encontrar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si ejecuta PowerShell localmente, también debe ejecutar `Connect-AzAccount` para iniciar sesión en Azure con una cuenta que tenga los [permisos necesarios](virtual-network-network-interface.md#permissions)].

Obtenga las reglas de seguridad vigentes para una interfaz de red con [Get-AzEffectiveNetworkSecurityGroup](/powershell/module/az.network/get-azeffectivenetworksecuritygroup). En el ejemplo siguiente se obtienen las reglas vigentes de una interfaz de red denominada *myVMVMNic*, que se encuentra en un grupo de recursos llamado *myResourceGroup*:

```azurepowershell-interactive
Get-AzEffectiveNetworkSecurityGroup `
  -NetworkInterfaceName myVMVMNic `
  -ResourceGroupName myResourceGroup
```

La salida se devuelve en formato json. Para comprender la salida, consulte la sección [Interpretación de la salida de los comandos](#interpret-command-output).
La salida solo se devuelve si un NSG está asociado a la interfaz de red, la subred en la que está la interfaz de red, o a ambas. La máquina virtual debe estar en estado de ejecución. Una máquina virtual puede tener varias interfaces de red con diferentes grupos de seguridad de red aplicados. Cuando esté intentando solucionar un problema, ejecute el comando para cada interfaz de red.

Si sigue teniendo problemas de conectividad, consulte las secciones [Diagnóstico adicional](#additional-diagnosis) y [Consideraciones](#considerations).

Si no conoce el nombre de una interfaz de red, pero conoce el nombre de la máquina virtual a la que está asociada la interfaz de red, los comandos siguientes devuelven los identificadores de todas las interfaces de red asociadas a una máquina virtual:

```azurepowershell-interactive
$VM = Get-AzVM -Name myVM -ResourceGroupName myResourceGroup
$VM.NetworkProfile
```

Recibirá una salida similar a la del ejemplo siguiente:

```output
NetworkInterfaces
-----------------
{/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic
```

En la salida anterior, el nombre de la interfaz de red es *myVMVMNic*.

## <a name="diagnose-using-azure-cli"></a>Diagnóstico mediante la CLI de Azure

Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. En este artículo se requiere la CLI de Azure versión 2.0.32 o posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta la CLI de Azure localmente, también debe ejecutar `az login` e iniciar sesión en Azure con una cuenta que tenga los [permisos necesarios](virtual-network-network-interface.md#permissions).

Obtenga las reglas de seguridad vigentes de una interfaz de red con [az network nic list-effective-nsg](/cli/azure/network/nic#az_network_nic_list_effective_nsg). En el ejemplo siguiente se obtienen las reglas vigentes de una interfaz de red denominada *myVMVMNic*, que se encuentra en un grupo de recursos llamado *myResourceGroup*:

```azurecli-interactive
az network nic list-effective-nsg \
  --name myVMVMNic \
  --resource-group myResourceGroup
```

La salida se devuelve en formato json. Para comprender la salida, consulte la sección [Interpretación de la salida de los comandos](#interpret-command-output).
La salida solo se devuelve si un NSG está asociado a la interfaz de red, la subred en la que está la interfaz de red, o a ambas. La máquina virtual debe estar en estado de ejecución. Una máquina virtual puede tener varias interfaces de red con diferentes grupos de seguridad de red aplicados. Cuando esté intentando solucionar un problema, ejecute el comando para cada interfaz de red.

Si sigue teniendo problemas de conectividad, consulte las secciones [Diagnóstico adicional](#additional-diagnosis) y [Consideraciones](#considerations).

Si no conoce el nombre de una interfaz de red, pero conoce el nombre de la máquina virtual a la que está asociada la interfaz de red, los comandos siguientes devuelven los identificadores de todas las interfaces de red asociadas a una máquina virtual:

```azurecli-interactive
az vm show \
  --name myVM \
  --resource-group myResourceGroup
```

En la salida devuelta, verá información similar a la del ejemplo siguiente:

```output
"networkProfile": {
    "additionalProperties": {},
    "networkInterfaces": [
      {
        "additionalProperties": {},
        "id": "/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
        "primary": true,
        "resourceGroup": "myResourceGroup"
      },
```

En la salida anterior, el nombre de la interfaz de red es *myVMVMNic interface*.

## <a name="interpret-command-output"></a>Interpretación de la salida de los comandos

Independientemente de si ha utilizado [PowerShell](#diagnose-using-powershell) o la [CLI de Azure](#diagnose-using-azure-cli) para diagnosticar el problema, recibirá una salida que contiene la siguiente información:

- **NetworkSecurityGroup**: el identificador del grupo de seguridad de red.
- **Asociación**: si el grupo de seguridad de red está asociado a una *interfaz de red*  o una *subred*. Si un NSG está asociado a ambos, se devuelve la salida con **NetworkSecurityGroup**, **Association** y **EffectiveSecurityRules**, para cada NSG. Si el grupo de seguridad de red se asocia o disocia inmediatamente antes de ejecutar el comando para ver las reglas de seguridad vigentes, es posible que tenga que esperar unos segundos para que el cambio se refleje en la salida del comando.
- **EffectiveSecurityRules**: en [Creación de una regla de seguridad](manage-network-security-group.md#create-a-security-rule) se proporciona una explicación de cada propiedad. Los nombres de regla precedidos por *defaultSecurityRules/* son las reglas de seguridad predeterminadas que existen en todos los grupos de seguridad de red. Los nombres de regla precedidos por *securityRules/* son reglas que ha creado. Las reglas que especifican una [etiqueta de servicio](./network-security-groups-overview.md#service-tags), como **Internet**, **VirtualNetwork** y **AzureLoadBalancer** para las propiedades  **destinationAddressPrefix** o **sourceAddressPrefix** también tienen valores para la propiedad **expandedDestinationAddressPrefix**. La propiedad **expandedDestinationAddressPrefix** enumera todos los prefijos de dirección representados por la etiqueta de servicio.

Si ve reglas duplicadas en la salida, es porque hay un grupo de seguridad de red asociado a la interfaz de red y la subred. Ambos NSG tienen las mismas reglas predeterminadas y pueden tener reglas duplicadas adicionales si ha creado sus propias reglas y son las mismas en ambos grupos de seguridad de red.

La regla denominada **defaultSecurityRules/DenyAllInBound** es lo que impide la comunicación entrante con la máquina virtual en el puerto 80, desde Internet, tal como se describe en el [escenario](#scenario). Ninguna otra regla con una prioridad más alta (número menor) permite el puerto 80 de entrada desde Internet.

## <a name="resolve-a-problem"></a>Resolución de un problema

Independientemente de que utilice Azure [Portal](#diagnose-using-azure-portal), [PowerShell](#diagnose-using-powershell) o la [CLI de Azure](#diagnose-using-azure-cli) para diagnosticar el problema que se presenta en el [escenario](#scenario) de este artículo, la solución consiste en crear una regla de seguridad de red con las siguientes propiedades:

| Propiedad                | Value                                                                              |
|---------                |---------                                                                           |
| Source                  | Any                                                                                |
| Source port ranges      | Any                                                                                |
| Destination             | La dirección IP de la máquina virtual, un intervalo de direcciones IP o todas las direcciones de la subred. |
| Intervalos de puertos de destino | 80                                                                                 |
| Protocolo                | TCP                                                                                |
| Acción                  | Allow                                                                              |
| Priority                | 100                                                                                |
| Nombre                    | Allow-HTTP-All                                                                     |

Después de crear la regla, el puerto 80 está permitido de entrada desde Internet, ya que la prioridad de la regla es mayor que la regla de seguridad predeterminada llamada *DenyAllInBound*, que deniega el tráfico. Aprenda a [crear una regla de seguridad](manage-network-security-group.md#create-a-security-rule). Si hay diferentes grupos de seguridad de red asociados tanto a la interfaz de red como a la subred, debe crear la misma regla en los dos grupos de seguridad de red.

Cuando Azure procesa el tráfico entrante, procesa las reglas del grupo de seguridad de red asociado a la subred (si hay alguno) y, después, procesa las reglas del grupo de seguridad de red asociado a la interfaz de red. Si hay un grupo de seguridad de red asociado tanto a la interfaz de red como la subred, el puerto debe estar abierto en ambos grupos de seguridad de red para que el tráfico llegue a la máquina virtual. Para solucionar los problemas de administración y la comunicación, se recomienda asociar un grupo de seguridad de red a una subred, en lugar de a interfaces de red individuales. Si las máquinas virtuales de una subred necesitan reglas de seguridad diferentes, puede hacer que los miembros de las interfaces de red de un grupo de seguridad de aplicaciones (ASG) y especifique un ASG como origen y destino de una regla de seguridad. Más información acerca de los [grupos de seguridad de aplicaciones](./network-security-groups-overview.md#application-security-groups).

Si sigue teniendo problemas de comunicación, consulte las secciones [Consideraciones](#considerations) y Diagnóstico adicional.

## <a name="considerations"></a>Consideraciones

Tenga en cuenta los puntos siguientes cuando tenga que solucionar problemas de conectividad:

* Las reglas de seguridad predeterminadas bloquean el acceso de entrada desde Internet y solo permiten el tráfico entrante desde la red virtual. Para permitir el tráfico entrante desde Internet, agregue reglas de seguridad con una prioridad más alta que las reglas predeterminadas. Obtenga más información acerca de las [reglas de seguridad predeterminadas](./network-security-groups-overview.md#default-security-rules) o de cómo [agregar una regla de seguridad](manage-network-security-group.md#create-a-security-rule).
* Si ha emparejado redes virtuales, de forma predeterminada, la etiqueta de servicio **VIRTUAL_NETWORK** se expande automáticamente para incluir prefijos para las redes virtuales emparejadas. Para solucionar los problemas relacionados con el emparejamiento de redes virtuales, puede ver los prefijos en la lista **ExpandedAddressPrefix**. Obtenga más información acerca del [emparejamiento de redes virtuales](virtual-network-peering-overview.md) y de las [etiquetas de servicio](./network-security-groups-overview.md#service-tags).
* Las reglas de seguridad vigentes solo se muestran para una interfaz de red si hay un grupo de seguridad de red asociado a la interfaz de red de la máquina virtual o a la subred, y si la máquina virtual está en estado de ejecución.
* Si no hay ningún grupo de seguridad de red asociado a la interfaz de red o a la subred y tiene una [dirección IP pública](virtual-network-public-ip-address.md) asignada a una máquina virtual, todos los puertos estarán abiertos para el acceso entrante y saliente a y desde cualquier lugar. Si la máquina virtual tiene una dirección IP pública, es aconsejable aplicar un grupo de seguridad de red a la subred o a la interfaz de red.

## <a name="additional-diagnosis"></a>Diagnóstico adicional

* Para ejecutar una prueba rápida para determinar si se permite que haya tráfico que llegué a una máquina virtual, o que salga de ella, use la funcionalidad de [comprobación del flujo de IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) de Azure Network Watcher. Comprobación del flujo de IP indica si el tráfico se permite o se deniega. Si se deniega, Comprobación del flujo de IP indica la regla de seguridad que deniega el tráfico.
* Si no reglas de seguridad de provoquen errores en la conectividad de red de una máquina virtual, el problema puede deberse a:
  * Software de firewall que se esté ejecutando en el sistema operativo de la máquina virtual
  * Rutas configuradas para dispositivos virtuales o para el tráfico local. El tráfico de Internet puede redirigirse a una red local a través de la [tunelización forzada](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si se fuerza el tráfico de Internet del túnel a una aplicación virtual, o local, es posible que no pueda conectarse a la máquina virtual desde internet. Para aprender a diagnosticar los problemas de ruta que pueden impedir el flujo del tráfico que sale de la máquina virtual, consulte [Diagnose a virtual machine routing problem](diagnose-network-routing-problem.md) (Diagnóstico de un problema de enrutamiento del tráfico de red de una máquina virtual).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información acerca de todas las tareas, propiedades y valores de un [grupo de seguridad de red](manage-network-security-group.md#work-with-network-security-groups) y de las [reglas de seguridad](manage-network-security-group.md#work-with-security-rules).
- Obtenga información acerca de las [reglas de seguridad predeterminadas](./network-security-groups-overview.md#default-security-rules), las [etiquetas de servicio](./network-security-groups-overview.md#service-tags), así como de [la forma en que Azure procesa las reglas de seguridad para el tráfico entrante y saliente](./network-security-groups-overview.md#network-security-groups) en una máquina virtual.