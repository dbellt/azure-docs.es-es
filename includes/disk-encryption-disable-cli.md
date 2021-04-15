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
ms.openlocfilehash: ed2b13077ff5809899097254dde0fa8406e1db3b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "85601398"
---
Puede deshabilitar el cifrado con Azure PowerShell, la CLI de Azure o una plantilla de Resource Manager. 

>[!IMPORTANT]
>Solo se puede deshabilitar el cifrado con Azure Disk Encryption en máquinas virtuales Linux para volúmenes de datos. No se admite en volúmenes de datos o volúmenes del sistema operativo si el volumen del sistema operativo se ha cifrado.  

- **Deshabilitar el cifrado de disco con Azure PowerShell:** para deshabilitar el cifrado, use el cmdlet [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM' [-VolumeType DATA]
     ```

- **Deshabilitar el cifrado con la CLI de Azure:** para deshabilitar el cifrado, use el comando [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type DATA
     ```
- **Deshabilitar el cifrado con una plantilla de Resource Manager:** use la plantilla [Deshabilitación del cifrado en una máquina virtual Linux en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad) para deshabilitar el cifrado.
     1. Haga clic en **Implementar en Azure**.
     2. Seleccione la suscripción, el grupo de recursos, la ubicación, la máquina virtual, los términos legales y el contrato.
