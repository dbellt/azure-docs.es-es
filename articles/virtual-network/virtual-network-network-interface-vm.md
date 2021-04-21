---
title: Incorporación de interfaces de red a VM de Azure o eliminación de estas
description: Aprenda a agregar interfaces de red a máquinas virtuales o a eliminarlas de ellas.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2020
ms.author: kumud
ms.openlocfilehash: 847f8dbd2d8f4064f12333348a4f03e5c5fcc611
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774276"
---
# <a name="add-network-interfaces-to-or-remove-network-interfaces-from-virtual-machines"></a>Incorporación de interfaces de red a máquinas virtuales o su eliminación de ellas

Obtenga información sobre cómo agregar una interfaz de red existente al crear una máquina virtual (VM) de Azure. Aprenda también a agregar o quitar interfaces de red de una VM existente en estado detenido (desasignado). Una interfaz de red permite que una VM de Azure se comunique con Internet, Azure y recursos locales. Una VM puede tener una o varias interfaces de red. 

Si tiene que agregar, cambiar o quitar direcciones IP para una interfaz de red, consulte [Administración de direcciones IP de interfaz de red](virtual-network-network-interface-addresses.md). Para crear, cambiar o eliminar interfaces de red, consulte [Administración de interfaces de red](virtual-network-network-interface.md).

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si no tiene ninguna, configure una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Complete una de estas tareas antes de continuar con este artículo:

- **Usuarios de Portal**: Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

- **Usuarios de PowerShell**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/powershell), o bien ejecute PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. En la pestaña del explorador Azure Cloud Shell, busque la lista desplegable **Seleccionar entorno** y, después, elija **PowerShell** si aún no está seleccionado.

    Si ejecuta PowerShell en el entorno local, use la versión del módulo de Azure PowerShell 1.0.0 o una posterior. Ejecute `Get-Module -ListAvailable Az.Network` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Ejecute `Connect-AzAccount` para crear una conexión con Azure.

- **Usuarios de la interfaz de la línea de comandos (CLI) de Azure**: ejecute los comandos en [Azure Cloud Shell](https://shell.azure.com/bash), o bien ejecute la CLI en el equipo. Use la versión 2.0.26 de la CLI de Azure o una posterior si ejecuta la CLI de Azure en el entorno local. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Ejecute `az login` para crear una conexión con Azure.

## <a name="add-existing-network-interfaces-to-a-new-vm"></a>Adición de interfaces de red existentes a una nueva máquina virtual

Al crear una máquina virtual a través del portal, este crea una interfaz de red con la configuración predeterminada y asocia la interfaz de red a la VM automáticamente. No se puede usar el portal para agregar interfaces de red existentes a una VM nueva ni para crear una VM con varias interfaces de red. Para ello, puede usar la CLI o PowerShell. Asegúrese de familiarizarse con las [restricciones](#constraints). Si crea una máquina virtual con varias interfaces de red, también debe configurar el sistema operativo para usarlas correctamente después de crear la máquina virtual. Aprenda cómo configurar [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) o [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) para varias interfaces de red.

### <a name="commands"></a>Comandos:

Antes de crear una VM, [cree una interfaz de red](virtual-network-network-interface.md#create-a-network-interface).

|Herramienta|Get-Help|
|---|---|
|CLI|[az network nic create](/cli/azure/network/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az_network_nic_create)|
|PowerShell|[New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="add-a-network-interface-to-an-existing-vm"></a>Incorporación de una interfaz de red a una VM existente

Para agregar una interfaz de red a la máquina virtual:

1. Vaya a [Azure Portal](https://portal.azure.com) para buscar una máquina virtual existente. Busque y seleccione **Máquinas virtuales**.

2. Seleccione el nombre de la VM. La máquina virtual debe admitir el número de interfaces de red que quiere agregar. Para averiguar cuántas interfaces de red admite cada tamaño de VM, consulte los tamaños de Azure para [VM Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [VM Windows](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

3. En la barra de comandos de la VM, seleccione **Detener** y, a continuación, **Aceptar** en el cuadro de diálogo de confirmación. Espere a que el valor de **Estado** de la VM cambie a **Detenido (desasignado)** .

4. En la barra de menús de la VM, elija **Redes** > **Asociar interfaz de red**. A continuación, en **Asociar una interfaz de red existente**, elija la interfaz de red que desea asociar y seleccione **Aceptar**.

    >[!NOTE]
    >La interfaz de red que seleccione no puede tener habilitadas las redes aceleradas, no puede tener asignada una dirección IPv6 y debe existir en la misma red virtual que la que tiene la interfaz de red actualmente asociada a la VM.

    Si no tiene una interfaz de red existente, debe crear una. Para ello, seleccione **Crear interfaz de red**. Para aprender más sobre cómo crear una interfaz de red, consulte [Crear una interfaz de red](virtual-network-network-interface.md#create-a-network-interface). Para más información sobre las restricciones adicionales cuando se agregan interfaces de red a las máquinas virtuales, consulte [Restricciones](#constraints).

5. En la barra de menús de la VM, elija **Información general** > **Iniciar** para reiniciar la máquina virtual.

Ahora puede configurar el sistema operativo de la VM para usar varias interfaces de red correctamente. Aprenda cómo configurar [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) o [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) para varias interfaces de red.

### <a name="commands"></a>Comandos:

|Herramienta|Get-Help|
|---|---|
|CLI|[az vm nic add](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_nic_add) (referencia); [pasos detallados](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-a-vm)|
|PowerShell|[Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (referencia); [pasos detallados](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-an-existing-vm)|

## <a name="view-network-interfaces-for-a-vm"></a>Visualización de las interfaces de red de una máquina virtual

Puede ver las interfaces de red asociadas actualmente a una máquina virtual para conocer su configuración y sus direcciones IP asignadas. 

1. Vaya a [Azure Portal](https://portal.azure.com) para buscar una máquina virtual existente. Busque y seleccione **Máquinas virtuales**.

    >[!NOTE]
    >Inicie sesión con una cuenta que tenga asignado el rol de propietario, colaborador o colaborador de red para la suscripción. Para más información sobre la asignación de roles a las cuentas, consulte [Roles integrados para el control de acceso basado en roles de Azure](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor).

2. Seleccione el nombre de la VM cuyas interfaces de red asociadas desea ver.

3. En la barra de menús de la VM, seleccione **Redes**.

Para información sobre la configuración de la interfaz de red y cómo modificarla, consulte [Administración de interfaces de red](virtual-network-network-interface.md). Para saber cómo agregar, cambiar o quitar direcciones IP asignadas a una interfaz de red, consulte [Administración de direcciones IP de interfaz de red](virtual-network-network-interface-addresses.md).

### <a name="commands"></a>Comandos:

|Herramienta|Get-Help|
|---|---|
|CLI|[az vm nic list](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_nic_list)|
|PowerShell|[Get-AzVM](/powershell/module/az.compute/get-azvm?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="remove-a-network-interface-from-a-vm"></a>Eliminación de una interfaz de red de una máquina virtual

1. Vaya a [Azure Portal](https://portal.azure.com) para buscar una máquina virtual existente. Busque y seleccione **Máquinas virtuales**.

2. Seleccione el nombre de la VM cuyas interfaces de red asociadas desea ver.

3. En la barra de herramientas de la VM, seleccione **Detener**.

4. Espere a que el **estado** de la máquina virtual cambie a **Detenida (desasignada)** .

5. En la barra de menús de la VM, seleccione **Redes** > **Desasociar interfaz de red**.

6. En el cuadro de diálogo **Desasociar interfaz de red**, seleccione la interfaz de red que desea desasociar. Después, seleccione **Aceptar**.

    >[!NOTE]
    >Si solo se muestra una interfaz de red, no puede desasociarla porque una máquina virtual siempre debe tener al menos una interfaz de red asociada.

### <a name="commands"></a>Comandos:

|Herramienta|Get-Help|
|---|---|
|CLI|[az vm nic remove](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az_vm_nic_remove) (referencia); [pasos detallados](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-a-vm)|
|PowerShell|[Remove-AzVMNetworkInterface](/powershell/module/az.compute/remove-azvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (referencia); [pasos detallados](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-an-existing-vm)|

## <a name="constraints"></a>Restricciones

- Una VM debe tener al menos una interfaz de red asociada.

- Una VM solo puede tener tantas interfaces de red asociadas como admita el tamaño de la VM. Para averiguar cuántas interfaces de red admite cada tamaño de VM, consulte los tamaños de Azure para [VM Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [VM Windows](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Todos los tamaños admiten al menos dos interfaces de red.

- Actualmente, las interfaces de red que se agregan a una VM no se pueden asociar a otra. Para más información sobre cómo crear interfaces de red, consulte [Crear una interfaz de red](virtual-network-network-interface.md#create-a-network-interface).

- Anteriormente, solo se podían agregar interfaces de red a las VM que admitían varias interfaces de red y que se habían creado con al menos dos. No se podía agregar una interfaz de red a una VM que se hubiera creado con una interfaz de red, aunque el tamaño de la VM admitiera más de una interfaz de red. Y a la inversa, solo se podían quitar interfaces de red de una máquina virtual que tuviera al menos tres, ya que las máquinas virtuales creadas con al menos dos interfaces de red siempre debían tener al menos dos. Estas restricciones ya no se aplican. Ahora puede crear una máquina virtual con cualquier cantidad de interfaces de red (hasta el máximo que permita el tamaño de la máquina virtual).

- De manera predeterminada, la primera interfaz de red asociada a una VM es la interfaz de red *principal*. Todas las demás interfaces de red de la máquina virtual son interfaces de red *secundarias*.

- Puede controlar a qué interfaz de red se envía el tráfico de salida. Sin embargo, una VM envía todo el tráfico de salida a la dirección IP asignada a la configuración de IP principal de la interfaz de red principal de forma predeterminada.

- En el pasado, era un requisito que todas las máquinas virtuales situadas en el mismo conjunto de disponibilidad debían tener una o varias interfaces de red. Ahora puede haber máquinas virtuales con cualquier cantidad de interfaces de red en el mismo conjunto de disponibilidad, hasta el máximo que permita su tamaño. Solo se pueden agregar máquinas virtuales a un conjunto de disponibilidad cuando se crean. Para más información sobre los conjuntos de disponibilidad, consulte [Administración de la disponibilidad de las máquinas virtuales en Azure](../virtual-machines/availability.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

- Puede conectar interfaces de red de la misma VM a diferentes subredes de una red virtual. Sin embargo, todas las interfaces de red deben estar conectadas a la misma red virtual.

- Puede agregar cualquier dirección IP para cualquier configuración de IP de cualquier interfaz de red principal o secundaria a un grupo de servidores back-end de Azure Load Balancer. En el pasado, solo la dirección IP principal de la interfaz de red principal podía agregarse a un grupo de servidores back-end. Para más información sobre las direcciones IP y las configuraciones, consulte [incorporación, cambio o eliminación de direcciones IP](virtual-network-network-interface-addresses.md).

- Al eliminar una VM, no se eliminan las interfaces de red que tiene asociadas. Cuando se elimina una máquina virtual, las interfaces de red se desasocian de ella. Puede agregar esas interfaces de red a diferentes VM o eliminarlas.

- La obtención de un rendimiento óptimo documentado requiere redes aceleradas. En algunos casos, debe habilitar explícitamente las redes aceleradas para máquinas virtuales [Windows](create-vm-accelerated-networking-powershell.md) o [Linux](create-vm-accelerated-networking-cli.md).

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="next-steps"></a>Pasos siguientes

Para crear una VM con varias interfaces de red o direcciones IP, consulte:

|Tarea|Herramienta|
|---|---|
|Creación de una máquina virtual con varias NIC|[CLI](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
|Creación de una máquina virtual con una sola interfaz de red y varias direcciones IPv4|[CLI](virtual-network-multiple-ip-addresses-cli.md), [PowerShell](virtual-network-multiple-ip-addresses-powershell.md)|
|Creación de una máquina virtual con una sola interfaz de red y una dirección IPv6 privada (detrás de Azure Load Balancer)|[CLI](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [Plantilla de Azure Resource Manager](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|