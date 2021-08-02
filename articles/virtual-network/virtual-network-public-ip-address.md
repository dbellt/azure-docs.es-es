---
title: Administrar direcciones IP públicas | Microsoft Docs
titleSuffix: Azure Virtual Network
description: Administrar direcciones IP públicas.  Además, aprenda de qué forma una dirección IP pública es un recurso con sus propios valores de configuración.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: bb71abaf-b2d9-4147-b607-38067a10caf6
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/06/2019
ms.author: kumud
ms.openlocfilehash: 80976d3c73a20f7f152f33a8d70cbbd4687a7bd9
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111810362"
---
# <a name="manage-public-ip-addresses"></a>Administración de direcciones IP públicas

Obtenga información sobre una dirección IP pública y cómo crearla, modificarla y eliminarla. Una dirección IP pública es un recurso con sus propios valores de configuración. Asignar una dirección IP pública a un recurso de Azure que admita direcciones IP públicas permite:
- La comunicación entrante desde Internet a los recursos, como Azure Virtual Machines, Azure Application Gateway, Azure Load Balancer, Azure VPN Gateway y otros. Todavía puede comunicarse con recursos como máquinas virtuales desde Internet, si una máquina virtual no tiene asignada una dirección IP pública, y siempre que la máquina virtual forme parte de un grupo de back-end de un equilibrador de carga, y el equilibrador de carga tenga asignada una dirección IP pública. Para determinar si puede asignarse una dirección IP pública a un recurso para un servicio específico de Azure, o si es posible comunicarse con él a través de la dirección IP pública de otro recurso de Azure, consulte la documentación del servicio.
- La conectividad saliente a Internet usa una dirección IP predecible. Por ejemplo, una máquina virtual puede establecer una comunicación saliente a Internet sin tener asignada una dirección IP pública, pero su dirección es una dirección de red que Azure traduce a una dirección pública impredecible, de manera predeterminada. Asignar una dirección IP pública a un recurso le permite saber cuál es la dirección IP que se usa para la conexión saliente. Si bien la dirección es predecible, puede cambiar en función del método de asignación que se elija. Para más información, consulte [Creación de una dirección IP pública](#create-a-public-ip-address). Para obtener más información sobre las conexiones salientes de recursos de Azure, consulte [Conexiones salientes](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Complete las tareas siguientes antes de seguir los pasos de las secciones de este artículo:

- Si todavía no tiene una cuenta de Azure, regístrese para obtener una [cuenta de evaluación gratuita](https://azure.microsoft.com/free).
- Si usa el portal, abra https://portal.azure.com e inicie sesión con la cuenta de Azure.
- Si usa comandos de PowerShell para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/powershell) o ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para realizar este tutorial, se necesita la versión 1.0.0 del módulo de Azure PowerShell u otra posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.
- Si usa la interfaz de la línea de comandos (CLI) de Azure para completar las tareas de este artículo, ejecute los comandos que se encuentran en [Azure Cloud Shell](https://shell.azure.com/bash) o ejecute la CLI en el equipo. Para realizar este tutorial, es necesaria la versión 2.0.31 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta de forma local la CLI de Azure, también debe ejecutar `az login` para crear una conexión con Azure.

La cuenta en la que inicia sesión o con la que se conecta a Azure debe tener asignado el rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

Las direcciones IP públicas tienen un precio simbólico. Para ver los precios, consulte la página [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Para obtener instrucciones sobre cómo crear direcciones IP públicas mediante el portal, PowerShell o la CLI, consulte las siguientes páginas:

 * [Creación de direcciones IP públicas: Azure Portal](./create-public-ip-portal.md?tabs=option-create-public-ip-standard-zones)
 * [Creación de direcciones IP públicas: PowerShell](./create-public-ip-powershell.md?tabs=option-create-public-ip-standard-zones)
 * [Creación de direcciones IP públicas: CLI de Azure](./create-public-ip-cli.md?tabs=option-create-public-ip-standard-zones)

>[!NOTE]
>Si bien el portal proporciona la opción de crear dos recursos de direcciones IP públicas (una IPv4 y una IPv6), los comandos de PowerShell y la CLI crean un recurso con una dirección para una versión de dirección IP o la otra. Si desea dos recursos de direcciones IP públicas, uno para cada versión de dirección IP, debe ejecutar dos veces el comando y especificar distintos nombres y versiones de dirección IP para los recursos de direcciones IP públicas.

Para obtener detalles adicionales sobre los atributos específicos de una dirección IP pública durante la creación, consulte la tabla siguiente.

   |Configuración|¿Necesario?|Detalles|
   |---|---|---|
   |Versión de la dirección IP|Sí| Seleccione IPv4 o IPv6 o ambas. Seleccionar ambas se traducirá en la creación de dos direcciones IP públicas: una dirección IPv4 y una dirección IPv6. Obtenga más información sobre [IPv6 en las redes virtuales de Azure](../virtual-network/ipv6-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).|
   |SKU|Sí|Todas las direcciones IP públicas creadas antes de la introducción de SKU son direcciones IP públicas de SKU **básica**. No se puede cambiar la SKU después de crear la dirección IP pública. Una máquina virtual independiente, las máquinas virtuales dentro de un conjunto de disponibilidad o los conjuntos de escalado de máquinas virtuales pueden usar SKU básicas o estándar. No se permite la mezcla de SKU entre máquinas virtuales dentro de conjuntos de disponibilidad, conjuntos de escalado o VM independientes. SKU **básico**: si va a crear una dirección IP pública en una región que admite zonas de disponibilidad, el ajuste **Zona disponibilidad** se establece en *Ninguno* de forma predeterminada. Las IP públicas básicas no admiten zonas de disponibilidad. SKU **estándar**: una dirección IP pública de SKU estándar se puede asociar a una máquina virtual o una front-end de equilibrador de carga. Si va a crear una dirección IP pública en una región que admite zonas de disponibilidad, el ajuste **Zona disponibilidad** se establece en *Con redundancia de zona* de forma predeterminada. Para obtener información sobre las zonas de disponibilidad, consulte el ajuste **Zona de disponibilidad**. La SKU estándar es necesaria si se asocia la dirección a un equilibrador de carga estándar. Para obtener más información sobre equilibradores de carga estándar, consulte [Azure load balancer standard SKU](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (SLU estándar para Azure Load Balancer). Cuando asigna una dirección IP pública de SKU estándar a una interfaz de red de una máquina virtual, debe permitir explícitamente el tráfico previsto con un [grupo de seguridad de red](./network-security-groups-overview.md#network-security-groups). Para evitar que se produzca un error en la comunicación con el recurso, debe crear un grupo de seguridad de red, asociarlo y permitir explícitamente el tráfico deseado.|
   |Nivel|Sí|Indica si la dirección IP está asociada a una región (**regional**) o si se trata de una "difusión por proximidad (anycast)" desde varias regiones (**global**). *Tenga en cuenta que una IP de "nivel global" es una funcionalidad de versión preliminar para las direcciones IP estándar y actualmente solo se emplea para Load Balancer entre regiones*.|
   |Nombre|Sí|El nombre debe ser único dentro del grupo de recursos que seleccione.|
   |Asignación de dirección IP|Sí|**Dinámica:** las direcciones dinámicas se asignan solo después de que una dirección IP pública se asocia a un recurso de Azure y el recurso se inicia por primera vez. Las direcciones dinámicas pueden cambiar si se asignan a un recurso, como una máquina virtual, y la máquina virtual se detiene (desasigna) y luego se reinicia. La dirección sigue siendo la misma si la máquina virtual se reinicia o se detiene (pero no se desasigna). Las direcciones dinámicas se liberan cuando un recurso de dirección IP pública se desasocia de un recurso o se asocia a él. **Estática:** las direcciones estáticas se asignan cuando se crea la dirección IP pública. Las direcciones estáticas no se liberan hasta que se elimina un recurso de dirección IP pública. Si la dirección no está asociada a un recurso, puede cambiar el método de asignación una vez creada la dirección. Si la dirección está asociada a un recurso, es posible que no pueda cambiar el método de asignación. Si selecciona *IPv6* como la **versión de dirección IP**, el método de asignación debe ser *Dinámica* para el SKU básico.  Las direcciones de SKU estándar son *estáticas* tanto para IPv4 como para IPv6. |
   |Tiempo de espera de inactividad (minutos)|No|Cantidad de minutos para mantener una conexión TCP o HTTP abierta sin que dependa del envío de mensajes de mantenimiento de los clientes. Si selecciona IPv6 como la **versión de dirección IP**, no se puede cambiar este valor. |
   |Etiqueta de nombre DNS|No|Debe ser único dentro de la ubicación de Azure en la que cree el nombre (entre todas las suscripciones y todos los clientes). Azure registra automáticamente el nombre y la dirección IP en el DNS para que pueda conectarse a un recurso con el nombre. Azure anexa una subred predeterminada, como *location.cloudapp.azure.com* (donde location es la ubicación seleccionada), al nombre que proporcione para crear el nombre DNS completo. Si elige crear ambas versiones de direcciones, el mismo nombre DNS se asigna tanto a la dirección IPv4 como a la IPv6. El DNS predeterminado de Azure contiene registros de nombres AAAA tanto de IPv4 como de IPv6 y responde con ambos registros cuando se busca el nombre DNS. El cliente elige con qué dirección (IPv4 o IPv6) comunicarse. En lugar de usar la etiqueta de nombre DNS con el sufijo predeterminado, o además de ello, puede usar el servicio Azure DNS para configurar un nombre DNS con un sufijo personalizado que se resuelva en la dirección IP pública. Para obtener más información, vea los detalles relativos al [uso de Azure DNS con una dirección IP pública de Azure](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address).|
   |Nombre (solo visible si selecciona la versión de IP de **ambas**)|Sí, si selecciona la versión de IP de **ambas**|El nombre debe ser distinto del nombre que escribe para el **nombre** de esta lista. Si elige crear tanto una dirección IPv4 como IPv6, el portal crea dos recursos de direcciones IP públicas independientes, uno con cada versión de dirección IP asignada.|
   |Asignación de direcciones IP (solo visible si selecciona la versión de IP de **ambas**)|Sí, si selecciona la versión de IP de **ambas**|Las mismas restricciones que la asignación de direcciones IP anterior|
   |Subscription|Sí|Debe existir en la misma [suscripción](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) que el recurso al cual asociará las direcciones IP públicas.|
   |Resource group|Sí|Puede existir en el mismo [grupo de recursos](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group) o en otro diferente como recurso al que asociará las direcciones IP públicas.|
   |Location|Sí|Debe existir en la misma [ubicación](https://azure.microsoft.com/regions), a la que también se hace referencia como región, que el recurso al que asociará las direcciones públicas.|
   |Zona de disponibilidad| No | Este ajuste solo aparece si selecciona una ubicación admitida. Para una lista de ubicaciones admitidas, consulte [Introducción a las zonas de disponibilidad](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si seleccionó la SKU **básica**, la opción *Ninguna* se selecciona automáticamente. Si prefiere garantizar una zona específica, puede seleccionarla. Cualquiera de las opciones no es con redundancia de zona. Si seleccionó la SKU **estándar**: la opción con redundancia de zona se selecciona automáticamente y hace que la ruta de acceso de datos sea resistente a errores de zona. Si prefiere garantizar una zona específica, que no sea resistente a errores de zona, puede seleccionarla.

## <a name="view-modify-settings-for-or-delete-a-public-ip-address"></a>Visualización, cambio de la configuración o eliminación de una dirección IP pública

   - **Vista o lista**: para revisar la configuración de una dirección IP pública, incluida la SKU, la dirección, cualquier asociación aplicable (por ejemplo, una NIC de máquina virtual o un front-end de Load Balancer).
   - **Modificar**: para modificar la configuración con la información del paso 4 para [crear una dirección IP pública](#create-a-public-ip-address), como el tiempo de espera de inactividad, la etiqueta de nombre DNS o el método de asignación.  (Para obtener el proceso completo de actualización de una SKU de IP pública de Básica a Estándar, vea [Actualización de direcciones IP públicas de Azure](./virtual-network-public-ip-address-upgrade.md)).
   >[!WARNING]
   >Para cambiar la asignación de una dirección IP pública de estática a dinámica, primero debe desasociar la dirección de cualquier configuración de IP aplicable (vea la sección **Eliminar**).  Asimismo, tenga en cuenta que, al cambiar el método de asignación de estático a dinámico, perderá la dirección IP que se asignó a la dirección IP pública. Aunque los servidores DNS públicos de Azure mantienen una asignación entre las direcciones estáticas o dinámicas y cualquier etiqueta de nombre DNS (si se han definido), las direcciones IP dinámicas pueden cambiar cuando se inicia la máquina virtual desde el estado detenido (desasignado). Para evitar que la dirección cambie, asigne una dirección IP estática.
   
|Operación|Azure portal|Azure PowerShell|Azure CLI|
|---|---|---|---|
|Ver | En la sección de **Información general** de una dirección IP pública. |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para recuperar un objeto de dirección IP pública y ver su configuración.| [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show) para mostrar la configuración.|
|List | En la categoría de **Direcciones IP públicas**. |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para recuperar uno o varios objetos de dirección IP pública y ver su configuración.|[az network public-ip list](/cli/azure/network/public-ip#az_network_public_ip_list) para enumerar las direcciones IP públicas.|
|Modificar | En el caso de una dirección IP desasociada, seleccione **Configuración** para modificar el tiempo de espera de inactividad, la etiqueta de nombre DNS o cambiar la asignación de la dirección IP básica de estática a dinámica.  |[Set-AzPublicIpAddress](/powershell/module/az.network/set-azpublicipaddress) para actualizar la configuración. |[az network public-ip update](/cli/azure/network/public-ip#az_network_public_ip_update) para actualizar. |

   - **Eliminar**: La eliminación de direcciones IP públicas requiere que el objeto de dirección IP pública no esté asociado a ninguna configuración de IP o NIC de máquina virtual. Consulte la tabla siguiente para obtener más detalles.

|Resource|Azure portal|Azure PowerShell|Azure CLI|
|---|---|---|---|
|[Máquina virtual](./remove-public-ip-address-vm.md)|Seleccione **Desasociar** para desasociar la dirección IP de la configuración de NIC y, a continuación, seleccione **Eliminar**.|[Set-AzPublicIpAddress](/powershell/module/az.network/set-azpublicipaddress) para desasociar la dirección IP de la configuración de NIC; [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) para eliminarla.|[az network public-ip update --remove](/cli/azure/network/public-ip#az_network_public_ip_update) para desasociar la dirección IP de la configuración de NIC; [az network public-ip delete](/cli/azure/network/public-ip#az_network_public_ip_delete) para eliminarla. |
|Front-end de Load Balancer | Vaya a una dirección IP pública que no se haya usado y seleccione **Asociar**; elija la instancia de Load Balancer que tenga la configuración de IP de front-end pertinente para reemplazarla (más adelante se podá eliminar la dirección IP antigua con el mismo método que para la VM).  | [Set-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/set-azloadbalancerfrontendipconfig) para asociar la nueva configuración de IP de front-end con la instancia pública de Load Balancer; [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) para eliminarla; también puede usar [Remove-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/remove-azloadbalancerfrontendipconfig) para quitar la configuración de IP de front-end si hay más de una. |[az network lb frontend-ip update](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_update) para asociar la nueva configuración de IP de front-end con la instancia pública de Load Balancer; [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) para eliminarla; también puede usar [az network lb frontend-ip delete](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_delete) para quitar la configuración de IP de front-end si hay más de una.|
|Firewall|N/D| [Deallocate()](../firewall/firewall-faq.yml#how-can-i-stop-and-start-azure-firewall) para desasignar el firewall y quitar todas las configuraciones de IP. | [az network firewall ip-config delete](/cli/azure/network/firewall/ip-config#az_network_firewall_ip_config_delete) para quitar la IP (pero debe usar PowerShell para desasignarla primero).|

## <a name="virtual-machine-scale-sets"></a>Virtual Machine Scale Sets

Al usar un conjunto de escalado de máquinas virtuales con direcciones IP públicas, no hay objetos de IP pública independientes asociados a las instancias de máquina virtual individuales. Sin embargo, se puede usar un objeto de prefijo de dirección IP pública [para generar las direcciones IP de la instancia](https://azure.microsoft.com/resources/templates/vmms-with-public-ip-prefix/).

Para enumerar las direcciones IP públicas de un conjunto de escalado de máquinas virtuales, puede usar PowerShell ([Get-AzPublicIpAddress -VirtualMachineScaleSetName](/powershell/module/az.network/get-azpublicipaddress)) o la CLI ([az vmss list-instance-public-ips](/cli/azure/vmss#az_vmss_list_instance_public_ips)).

Para más información, consulte [Redes para conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md#public-ipv4-per-virtual-machine).

## <a name="assign-a-public-ip-address"></a>Asignación de una dirección IP pública

Aprenda a asignar una dirección IP pública a los siguientes recursos:

- Una máquina virtual [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (al crearla) o a una [máquina virtual existente](virtual-network-network-interface-addresses.md#add-ip-addresses)
- [Equilibrador de carga público](../load-balancer/quickstart-load-balancer-standard-public-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Application Gateway](../application-gateway/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Conexión de sitio a sitio con VPN Gateway](../vpn-gateway/tutorial-site-to-site-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="permissions"></a>Permisos

Para realizar tareas en direcciones IP públicas, su cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que se muestran en la tabla siguiente:

| Acción                                                             | Nombre                                                           |
| ---------                                                          | -------------                                                  |
| Microsoft.Network/publicIPAddresses/read                           | Leer una dirección IP pública                                          |
| Microsoft.Network/publicIPAddresses/write                          | Crear o actualizar una dirección IP pública                           |
| Microsoft.Network/publicIPAddresses/delete                         | Eliminar una dirección IP pública                                     |
| Microsoft.Network/publicIPAddresses/join/action                    | Asociar una dirección IP pública a un recurso                    |

## <a name="next-steps"></a>Pasos siguientes

- Crear una dirección IP pública con scripts de ejemplo de [PowerShell](powershell-samples.md) o [CLI de Azure](cli-samples.md) o con [plantillas de Azure Resource Manager](template-samples.md)
- Crear y asignar [definiciones de Azure Policy](./policy-reference.md) para direcciones IP públicas
