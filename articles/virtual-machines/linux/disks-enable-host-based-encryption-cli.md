---
title: Habilitación del cifrado de un extremo a otro mediante el cifrado en el host - CLI de Azure - discos administrados
description: Use el cifrado en el host para habilitar el cifrado de un extremo a otro en los discos administrados de Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 08/24/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 10fe02b0deb505fcedc6fc3119150709b6613b04
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110673029"
---
# <a name="use-the-azure-cli-to-enable-end-to-end-encryption-using-encryption-at-host"></a>Use la CLI de Azure para habilitar el cifrado de un extremo a otro mediante el cifrado en host

Cuando se habilita el cifrado en el host, los datos almacenados en el host de máquina virtual se cifran en reposo y se transmiten cifrados al servido Storage. Para obtener información conceptual sobre el cifrado en el host, así como otros tipos de cifrado de disco administrado, consulte [Cifrado en host: cifrado de un extremo a otro para los datos de la VM](../disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## <a name="restrictions"></a>Restricciones

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-vm-sizes"></a>Tamaños de máquinas virtuales que se admiten

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

La lista completa de tamaños de máquina virtual admitidos se puede obtener mediante programación. Para obtener información sobre cómo recuperarlos mediante programación, consulte la sección [Búsqueda de tamaños de máquinas virtuales admitidos](#finding-supported-vm-sizes).
La actualización del tamaño de la máquina virtual producirá una validación para comprobar si el nuevo tamaño de máquina virtual admite la característica EncryptionAtHost.

## <a name="prerequisites"></a>Requisitos previos

Debe habilitar la característica para su suscripción antes de usar la propiedad EncryptionAtHost para su VM/VMSS. Siga los pasos que se indican a continuación para habilitar la característica para su suscripción:

1.  Ejecute el siguiente comando para registrar la característica para su suscripción

    ```azurecli
    az feature register --namespace Microsoft.Compute --name EncryptionAtHost
    ```
 
2.  Compruebe que el estado de registro es Registrado (tarda unos minutos) mediante el comando siguiente antes de probar la característica.

    ```azurecli
    az feature show --namespace Microsoft.Compute --name EncryptionAtHost
    ```


### <a name="create-an-azure-key-vault-and-diskencryptionset"></a>Creación de una instancia de Azure Key Vault y DiskEncryptionSet

Una vez habilitada la característica, deberá configurar un almacén de Azure Key Vault y un elemento DiskEncryptionSet, si no lo ha hecho aún.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-cli](../../../includes/virtual-machines-disks-encryption-create-key-vault-cli.md)]

## <a name="examples"></a>Ejemplos

### <a name="create-a-vm-with-encryption-at-host-enabled-with-customer-managed-keys"></a>Cree una máquina virtual con cifrado en el host habilitado con claves administradas por el cliente. 

Cree una máquina virtual con discos administrados mediante el identificador URI del recurso de DiskEncryptionSet creado anteriormente para cifrar la caché de los discos de datos y del sistema operativo con claves administradas por el cliente. Los discos temporales se cifran con claves administradas por la plataforma. 

```azurecli
rgName=yourRGName
vmName=yourVMName
location=eastus
vmSize=Standard_DS2_v2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskEncryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vm create -g $rgName \
-n $vmName \
-l $location \
--encryption-at-host \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--os-disk-encryption-set $diskEncryptionSetId \
--data-disk-sizes-gb 128 128 \
--data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-a-vm-with-encryption-at-host-enabled-with-platform-managed-keys"></a>Cree una máquina virtual con cifrado en el host habilitado con claves administradas por la plataforma. 

Cree una máquina virtual con cifrado en el host habilitado para cifrar la caché de discos de datos o del sistema operativo, y discos temporales con claves administradas por la plataforma. 

```azurecli
rgName=yourRGName
vmName=yourVMName
location=eastus
vmSize=Standard_DS2_v2
image=UbuntuLTS 

az vm create -g $rgName \
-n $vmName \
-l $location \
--encryption-at-host \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--data-disk-sizes-gb 128 128 \
```

### <a name="update-a-vm-to-enable-encryption-at-host"></a>Actualización de una máquina virtual para habilitar el cifrado en el host 

```azurecli
rgName=yourRGName
vmName=yourVMName

az vm update -n $vmName \
-g $rgName \
--set securityProfile.encryptionAtHost=true
```

### <a name="check-the-status-of-encryption-at-host-for-a-vm"></a>Comprobación del estado del cifrado en el host de una máquina virtual

```azurecli
rgName=yourRGName
vmName=yourVMName

az vm show -n $vmName \
-g $rgName \
--query [securityProfile.encryptionAtHost] -o tsv
```

### <a name="create-a-virtual-machine-scale-set-with-encryption-at-host-enabled-with-customer-managed-keys"></a>Cree un conjunto de escalado de máquinas virtuales con cifrado en el host habilitado con claves administradas por el cliente. 

Cree un conjunto de escalado de máquinas virtuales con discos administrados mediante el identificador URI del recurso de DiskEncryptionSet creado anteriormente para cifrar la caché de los discos de datos y del sistema operativo con claves administradas por el cliente. Los discos temporales se cifran con claves administradas por la plataforma. 

```azurecli
rgName=yourRGName
vmssName=yourVMSSName
location=westus2
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskEncryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--admin-username azureuser \
--generate-ssh-keys \
--os-disk-encryption-set $diskEncryptionSetId \
--data-disk-sizes-gb 64 128 \
--data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-a-virtual-machine-scale-set-with-encryption-at-host-enabled-with-platform-managed-keys"></a>Cree un conjunto de escalado de máquinas virtuales con cifrado en el host habilitado con claves administradas por la plataforma. 

Cree un conjunto de escalado de máquinas virtuales con cifrado en el host habilitado para cifrar la caché de discos de datos o del sistema operativo, y discos temporales con claves administradas por la plataforma. 

```azurecli
rgName=yourRGName
vmssName=yourVMSSName
location=westus2
vmSize=Standard_DS3_V2
image=UbuntuLTS 

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--admin-username azureuser \
--generate-ssh-keys \
--data-disk-sizes-gb 64 128 \
```

### <a name="update-a-virtual-machine-scale-set-to-enable-encryption-at-host"></a>Actualice un conjunto de escalado de máquinas virtuales para habilitar el cifrado en el host. 

```azurecli
rgName=yourRGName
vmssName=yourVMName

az vmss update -n $vmssName \
-g $rgName \
--set virtualMachineProfile.securityProfile.encryptionAtHost=true
```

### <a name="check-the-status-of-encryption-at-host-for-a-virtual-machine-scale-set"></a>Comprobación del estado del cifrado en el host de un conjunto de escalado de máquinas virtuales

```azurecli
rgName=yourRGName
vmssName=yourVMName

az vmss show -n $vmssName \
-g $rgName \
--query [virtualMachineProfile.securityProfile.encryptionAtHost] -o tsv
```

## <a name="finding-supported-vm-sizes"></a>Búsqueda de tamaños de máquinas virtuales admitidos

No se admiten los tamaños de máquina virtual heredados. Para consultar la lista de tamaños de máquina virtual admitidos, puede hacer lo siguiente:

Llamar a la [API Resource Skus](/rest/api/compute/resourceskus/list) y comprobar que la capacidad `EncryptionAtHostSupported` está configurada en **True**.

```json
    {
        "resourceType": "virtualMachines",
        "name": "Standard_DS1_v2",
        "tier": "Standard",
        "size": "DS1_v2",
        "family": "standardDSv2Family",
        "locations": [
        "CentralUSEUAP"
        ],
        "capabilities": [
        {
            "name": "EncryptionAtHostSupported",
            "value": "True"
        }
        ]
    }
```

O bien, llamar al cmdlet de PowerShell [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku).

```powershell
$vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.Contains('CentralUSEUAP')} 

foreach($vmSize in $vmSizes)
{
    foreach($capability in $vmSize.capabilities)
    {
        if($capability.Name -eq 'EncryptionAtHostSupported' -and $capability.Value -eq 'true')
        {
            $vmSize

        }

    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado y configurado estos recursos, puede usarlos para proteger los discos administrados. En los vínculos siguientes encontrará scripts de ejemplo, cada uno con un escenario correspondiente, que puede usar para proteger sus discos administrados.

[Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)
