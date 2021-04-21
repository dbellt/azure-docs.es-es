---
title: Desasociación de una dirección IP pública de una máquina virtual de Azure
titlesuffix: Azure Virtual Network
description: Aprenda a desasociar direcciones IP públicas de una máquina virtual
services: virtual-network
documentationcenter: ''
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/04/2019
ms.author: allensu
ms.openlocfilehash: e9bfadd3e2453f0241dc2f7b8bfa5c964333bcf5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776547"
---
# <a name="dissociate-a-public-ip-address-from-an-azure-vm"></a>Desasociación de una dirección IP pública de una máquina virtual de Azure 

En este artículo aprenderá a desasociar direcciones IP públicas de una máquina virtual (VM) de Azure.

Para disociar una dirección IP pública de una máquina virtual, puede usar [Azure Portal](#azure-portal), la [interfaz de la línea de comandos](#azure-cli) (CLI) de Azure o [PowerShell](#powershell).

## <a name="azure-portal"></a>Azure portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Busque o vaya a la máquina virtual de la que desea disociar la dirección IP pública y selecciónela.
3. En la página de la máquina virtual, seleccione **Información general** y, después, la dirección IP pública, como se muestra en la imagen siguiente:

   ![Seleccionar IP pública](./media/remove-public-ip-address/remove-public-ip-address-2.png)

4. En la página de la dirección IP pública, seleccione **Información general** y, después, **Desasociar**, como se muestra en la siguiente imagen:

    ![Desasociar IP pública](./media/remove-public-ip-address/remove-public-ip-address-3.png)

5. En **Desasociar dirección IP pública**, seleccione **Sí**.

## <a name="azure-cli"></a>Azure CLI

Instale la [CLI de Azure](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json) o use Azure Cloud Shell. Azure Cloud Shell es un shell de Bash gratuito que se puede ejecutar directamente en Azure Portal. Tiene la CLI de Azure preinstalada y configurada para utilizarla con la cuenta. Seleccione el botón **Probar** en los comandos siguientes de la CLI. Al seleccionar **Probar**, se invoca una instancia de Cloud Shell con la que puede iniciar sesión en la cuenta de Azure.

1. Si usa la CLI localmente en Bash, inicie sesión en Azure con `az login`.
2. Una dirección IP pública está asociada a una configuración de IP de una interfaz de red conectada a una VM. Use el comando [az network nic-ip-config update](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_update) para desasociar una dirección IP pública de la configuración de una IP. En el siguiente ejemplo se desasocia una dirección IP pública denominada *myVMPublicIP* de la configuración de IP *ipconfigmyVM* de una interfaz de red existente denominada *myVMVMNic* que está asociada a una máquina virtual denominada *myVM* de un grupo de recursos denominado *myResourceGroup*.
  
   ```azurecli-interactive
    az network nic ip-config update \
    --name ipconfigmyVM \
    --resource-group myResourceGroup \
    --nic-name myVMVMNic \
    --remove PublicIpAddress
   ```

   Si no conoce el nombre de una interfaz de red conectada a la VM, use el comando [az vm nic list](/cli/azure/vm/nic#az_vm_nic_list) para verla. Por ejemplo, el comando siguiente enumera los nombres de las interfaces de red conectadas a una VM denominada *myVM* en un grupo de recursos denominado *myResourceGroup*:

     ```azurecli-interactive
     az vm nic list --vm-name myVM --resource-group myResourceGroup
     ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente:
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

     En el ejemplo anterior, *myVMVMNic* es el nombre de la interfaz de red.

   - Si no conoce el nombre de una configuración de IP para una interfaz de red, use el comando [az network nic ip-config list](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_list) para recuperarla. Por ejemplo, el comando siguiente enumera los nombres de las configuraciones de IP públicas para una interfaz de red denominada *myVMVMNic* en un grupo de recursos denominado *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config list --nic-name myVMVMNic --resource-group myResourceGroup --out table
     ```

   - Si no conoce el nombre de una configuración de IP pública para una interfaz de red, use el comando [az network nic ip-config show](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_show) para recuperarlo. Por ejemplo, el comando siguiente enumera los nombres de las configuraciones de IP públicas para una interfaz de red denominada *myVMVMNic* en un grupo de recursos denominado *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config show --name ipconfigmyVM --nic-name myVMVMNic --resource-group myResourceGroup --query publicIPAddress.id
     ```


## <a name="powershell"></a>PowerShell

Instale [PowerShell](/powershell/azure/install-az-ps) o use Azure Cloud Shell. Azure Cloud Shell es un shell gratuito que se ejecuta directamente en Azure Portal. Tiene preinstalado y configurado PowerShell para usarlo con su cuenta. Seleccione el botón **Probar** en los comandos siguientes de PowerShell. Al seleccionar **Probar**, se invoca una instancia de Cloud Shell con la que puede iniciar sesión en la cuenta de Azure.

1. Si usa PowerShell localmente, inicie sesión en Azure con `Connect-AzAccount`.
2. Una dirección IP pública está asociada a una configuración de IP de una interfaz de red conectada a una VM. Use el comando [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) para obtener una interfaz de red. Establezca el valor de Dirección IP pública en NULL y, después, use el comando [Set-AzNetworkInterface](/powershell/module/Az.Network/Set-AzNetworkInterface) para escribir la nueva configuración de la IP en la interfaz de red.

   En el ejemplo siguiente se desasocia una dirección IP pública denominada *myVMPublicIP* de una interfaz de red denominada *myVMVMNic* que está asociada a una máquina virtual denominada *myVM*. Todos los recursos están en un grupo de recursos denominado *myResourceGroup*.
  
   ```azurepowershell
    $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroup myResourceGroup
    $nic.IpConfigurations.publicipaddress.id = $null
    Set-AzNetworkInterface -NetworkInterface $nic
   ```

  - Si no conoce el nombre de una interfaz de red conectada a la VM, use el comando [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) para verla. Por ejemplo, el comando siguiente enumera los nombres de las interfaces de red conectadas a una VM denominada *myVM* en un grupo de recursos denominado *myResourceGroup*:

    ```azurepowershell
    $vm = Get-AzVM -name myVM -ResourceGroupName myResourceGroup
    $vm.NetworkProfile
    ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente. En la salida de ejemplo, *myVMVMNic* es el nombre de la interfaz de red.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

   - Si no conoce el nombre de una configuración de IP para una interfaz de red, use el comando [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) para recuperarla. Por ejemplo, el comando siguiente enumera los nombres de las configuraciones de IP para una interfaz de red denominada *myVMVMNic* en un grupo de recursos denominado *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $nic.IPConfigurations.id
     ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente. En la salida del ejemplo, *ipconfigmyVM* es el nombre de una configuración de IP.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM"
     ```

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [asociar una dirección IP pública a una máquina virtual](associate-public-ip-address-vm.md).
