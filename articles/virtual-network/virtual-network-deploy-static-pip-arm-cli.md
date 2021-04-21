---
title: 'Crear una VM con una dirección IP pública estática: CLI de Azure | Microsoft Docs'
description: Aprenda a crear una VM con una dirección IP pública estática mediante la interfaz de la línea de comandos (CLI) de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: 55bc21b0-2a45-4943-a5e7-8d785d0d015c
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/08/2018
ms.author: kumud
ms.openlocfilehash: f0f61cc4ef02033a2c21ce5acde68caea483e743
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790138"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-the-azure-cli"></a>Creación de una máquina virtual con una dirección IP pública estática mediante la CLI de Azure

Puede crear una máquina virtual con una dirección IP pública estática. Una dirección IP pública le permite comunicarse con una máquina virtual desde Internet. Asigne una dirección IP pública estática, en lugar de una dirección dinámica, para garantizar que la dirección no cambie nunca. Más información sobre [direcciones IP públicas estáticas](./public-ip-addresses.md#allocation-method). Para cambiar una dirección IP pública asignada a una máquina virtual existente de dinámica a estática, o para trabajar con direcciones IP privadas, consulte [Incorporación, cambio o eliminación de direcciones IP](virtual-network-network-interface-addresses.md). Las direcciones IP públicas tienen un [costo nominal](https://azure.microsoft.com/pricing/details/ip-addresses) y hay un [límite](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits) en el número de direcciones IP públicas que se pueden usar por suscripción.

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Puede realizar los pasos siguientes desde el equipo local o mediante Azure Cloud Shell. Para usar el equipo local, asegúrese de tener [instalada la CLI de Azure](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json). Para usar Azure Cloud Shell, seleccione **Probar** en la esquina superior derecha de cualquier cuadro de comando que sigue. Cloud Shell inicia su sesión en Azure.

1. Si usa Cloud Shell, continúe al paso 2. Abra una sesión de comandos e inicie sesión en Azure con `az login`.
2. Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create). En el siguiente ejemplo se crea un grupo de recursos en la región Este de EE. UU. de Azure:

   ```azurecli-interactive
   az group create --name myResourceGroup --location eastus
   ```

3. Cree la máquina virtual con el comando [az vm create](/cli/azure/vm#az_vm_create). La opción `--public-ip-address-allocation=static` asigna una dirección IP pública estática a la máquina virtual. En el ejemplo siguiente se crea una máquina virtual Ubuntu con una dirección IP pública estática, con una SKU básica, denominada *myPublicIpAddress*:

   ```azurecli-interactive
   az vm create \
     --resource-group myResourceGroup \
     --name myVM \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys \
     --public-ip-address myPublicIpAddress \
     --public-ip-address-allocation static
   ```

   Si la dirección IP pública debe ser una SKU estándar, agregue `--public-ip-sku Standard` al comando anterior. Más información sobre las [SKU de dirección IP pública](./public-ip-addresses.md#sku). Si la máquina virtual se va a agregar al grupo back-end de una instancia pública de Azure Load Balancer, la SKU de la dirección IP pública de la máquina virtual debe coincidir con la SKU de la dirección IP del equilibrador de carga. Para más información, consulte [Azure Load Balancer](../load-balancer/skus.md).

4. Vea la dirección IP pública asignada y confirme que se creó como una dirección estática con una SKU básica, con el comando [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show):

   ```azurecli-interactive
   az network public-ip show \
     --resource-group myResourceGroup \
     --name myPublicIpAddress \
     --query [ipAddress,publicIpAllocationMethod,sku] \
     --output table
   ```

   Azure asigna una dirección IP pública de las direcciones usadas en la región en la que creó la máquina virtual. Puede descargar la lista de intervalos (prefijos) para las nubes de Azure [Pública](https://www.microsoft.com/download/details.aspx?id=56519), [Gobierno de Estados Unidos](https://www.microsoft.com/download/details.aspx?id=57063), [China](https://www.microsoft.com/download/details.aspx?id=57062) y [Alemania](https://www.microsoft.com/download/details.aspx?id=57064).

> [!WARNING]
> No modifique la configuración de direcciones IP dentro del sistema operativo de la máquina virtual. El sistema operativo no conoce las direcciones IP públicas de Azure. Aunque puede agregar la configuración de dirección IP privada al sistema operativo, se recomienda no hacerlo a menos que sea necesario y no hasta después de haber leído [Incorporación o eliminación de direcciones IP privadas a un sistema operativo](virtual-network-network-interface-addresses.md#private).

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede utilizar [az group delete](/cli/azure/group#az_group_delete) para eliminar el grupo de recursos y todos los recursos que contenga:

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [direcciones IP públicas](./public-ip-addresses.md#public-ip-addresses) en Azure
- Más información acerca de toda la [configuración de dirección IP pública](virtual-network-public-ip-address.md#create-a-public-ip-address)
- Más información acerca de [direcciones IP privadas](./private-ip-addresses.md) y la asignación de una [dirección IP privada estática](virtual-network-network-interface-addresses.md#add-ip-addresses) a una máquina virtual de Azure
- Más información acerca de cómo crear máquinas virtuales [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
