---
title: Actualización de Azure Disk Encryption en un disco
description: En este artículo se proporcionan instrucciones sobre cómo actualizar Azure Disk Encryption en un disco
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.topic: how-to
ms.author: mbaldwin
ms.date: 05/27/2021
ms.custom: seodec18, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 0cdc32e303de7d9a26a99d1f08188dc54539930b
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111758315"
---
# <a name="upgrading-the-azure-disk-encryption-version"></a>Actualización de la versión de Azure Disk Encryption

La primera versión de Azure Disk Encryption (ADE) se basaba en Azure Active Directory (AAD) para la autenticación. Sin embargo, la versión actual no lo hace.  Se recomienda encarecidamente el uso de la versión más reciente.

## <a name="determine-ade-version"></a>Determinación de la versión de ADE

Las versiones de ADE en el ámbito de la migración son:
- **Windows:** versión 1.1.* (ADE en la máquina virtual debe actualizarse a la versión 2.2)
- **Linux:** versión 0.1.* (ADE en la máquina virtual debe actualizarse a la versión 1.2)

Puede determinar la versión de ADE con la que se ha cifrado una máquina virtual mediante la CLI de Azure, Azure PowerShell o Azure Portal.

# <a name="cli"></a>[CLI](#tab/CLI)

Para determinar la versión de ADE, ejecute el comando [az vm get-instance-view](/cli/azure/vm#az_vm_get_instance_view) de la CLI de Azure.

```azurecli-interactive
az vm get-instance-view --resource-group  <ResourceGroupName> --name <VMName> 
```

Busque la extensión AzureDiskEncryption en la salida e identifique el número de versión en el campo "TypeHandlerVersion" en la salida.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Para determinar la versión de ADE, ejecute el comando [Get-AzVM](/powershell/module/az.compute/get-azvm) de Azure PowerShell.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName <ResourceGroupName> -Name <VMName> -Status
```

Busque la extensión AzureDiskEncryption en la salida e identifique el número de versión en el campo "TypeHandlerVersion" en la salida.

# <a name="portal"></a>[Portal](#tab/Portal)

Vaya a la hoja "Extensiones" de la máquina virtual en Azure Portal.

:::image type="content" source="../media/disk-encryption/ade-version-1.png" alt-text="Captura de pantalla 1 de la búsqueda en el portal de la versión de ADE":::

Elija la extensión "AzureDiskEncryption" para Windows o la extensión "AzureDiskEncryptionForLinux" para Linux y busque el número de versión en el campo "Versión".

:::image type="content" source="../media/disk-encryption/ade-version-2.png" alt-text="Captura de pantalla 2 de la búsqueda en el portal de la versión de ADE":::

---

## <a name="how-to-migrate"></a>Cómo migrar

La migración de Azure Disk Encryption (con AAD) a Azure Disk Encryption (sin AAD) solo está disponible a través de Azure PowerShell. Asegúrese de que tiene instalada la versión más reciente de Azure PowerShell y al menos la [versión 5.9.0 del módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

Para actualizar de Azure Disk Encryption (con AAD) a Azure Disk Encryption (sin AAD), use el cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) de PowerShell. 

> [!WARNING]
> El cmdlet Set-AzVMDiskEncryptionExtension solo se debe usar en máquinas virtuales cifradas con Azure Disk Encryption (con AAD). Al intentar migrar una máquina virtual sin cifrar o una máquina virtual cifrada con Azure Disk Encryption (sin AAD), se producirá un error de terminal.

```azurepowershell-interactive
Set-AzVMDiskEncryptionExtension -ResourceGroupName <resourceGroupName> -VMName <vmName> -Migrate
```

Cuando el cmdlet le pida confirmación, escriba "Y".  Se actualizará la versión de ADE y se reiniciará la máquina virtual. La salida tendrá una apariencia parecida a la siguiente:

```bash
> Set-AzVMDiskEncryptionExtension -ResourceGroupName myResourceGroup -VMName myVM -Migrate

Update AzureDiskEncryption version?
This cmdlet updates Azure Disk Encryption version to single pass (Azure Disk Encryption without AAD). This may reboot
the machine and takes 10-15 minutes to finish. Are you sure you want to continue?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
Azure Disk Encryption Extension Public Settings
"KeyVaultResourceId": /subscriptions/ea500758-3163-4849-bd2c-3e50f06efa7a/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myKeyVault
"SequenceVersion":
"MigrateFlag": Migrate
"KeyVaultURL": https://myKeyVault.vault.azure.net/
"AADClientID": d29edf8c-3fcb-42e7-8410-9e39fdf0dd70
"KeyEncryptionKeyURL":
"KekVaultResourceId":
"EncryptionOperation": EnableEncryption
"AADClientCertThumbprint":
"VolumeType":
"KeyEncryptionAlgorithm":

Running ADE extension (with AAD) for -Migrate..
ADE extension (with AAD) is now complete. Updating VM model..
Running ADE extension (without AAD) for -Migrate..
ADE extension (without AAD) is now complete. Clearing VM model..

RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```


> [!IMPORTANT]
> La actualización llevará al menos entre 10 y 15 minutos en completarse. No cancele el cmdlet mientras la actualización está en curso. Si lo hace, pondrá en riesgo el estado de la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas de Azure Disk Encryption](disk-encryption-troubleshooting.md)
