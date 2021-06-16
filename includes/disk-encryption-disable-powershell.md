---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 06/19/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 517bde8d365793906c24d0f242d4adc1b6241e7c
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112084030"
---
Puede deshabilitar el cifrado con Azure PowerShell, la CLI de Azure o una plantilla de Resource Manager. La deshabilitación del cifrado de disco de datos en la VM de Windows cuando se han cifrado tanto los discos de datos como del SO no funciona según lo previsto. Deshabilite del cifrado en todos los discos en su lugar.

- **Deshabilitar el cifrado de disco con Azure PowerShell:** para deshabilitar el cifrado, use el cmdlet [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM' -VolumeType "all"
     ```

- **Deshabilitar el cifrado con la CLI de Azure:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az_vm_encryption_disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type "all"
     ```
- **Deshabilitar el cifrado con una plantilla de Resource Manager:** 

    1. Haga clic en **Deploy to Azure** (Implementar en Azure) en la plantilla [Deshabilitar el cifrado de disco en una máquina virtual Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/decrypt-running-windows-vm-without-aad).
    2. Seleccione la suscripción, el grupo de recursos, la ubicación, la máquina virtual, el tipo de volumen, los términos legales y el contrato.
    3.  Haga clic en **Comprar** para deshabilitar el cifrado de disco en una máquina virtual Windows en ejecución. 
