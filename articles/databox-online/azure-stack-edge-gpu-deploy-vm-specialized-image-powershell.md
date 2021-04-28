---
title: Creación de imágenes de máquina virtual a partir de imágenes especializadas de discos duros virtuales (VHD) de Windows para el dispositivo de GPU de Azure Stack Edge Pro
description: Se describe cómo crear imágenes de máquina virtual a partir de imágenes especializadas desde un VHD o VHDX en Windows. Utilice esta imagen especializada para crear imágenes de máquina virtual que se usarán con las máquinas virtuales implementadas en el dispositivo de GPU de Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/15/2021
ms.author: alkohli
ms.openlocfilehash: 6bfa42e99f295b429eba40a27eb59becb8aa80a1
ms.sourcegitcommit: d3bcd46f71f578ca2fd8ed94c3cdabe1c1e0302d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107575961"
---
# <a name="deploy-a-vm-from-a-specialized-image-on-your-azure-stack-edge-pro-gpu-device-via-azure-powershell"></a>Implementación de una máquina virtual a partir de una imagen especializada en el dispositivo Azure Stack Edge Pro GPU a través de Azure PowerShell 

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describen los pasos necesarios para implementar una máquina virtual (VM) en el dispositivo Azure Stack Edge Pro GPU a partir de una imagen especializada. 

Para preparar una imagen generalizada para la implementación de máquinas virtuales en Azure Stack Edge Pro GPU, consulte [Preparación de una imagen generalizada desde un VHD de Windows](azure-stack-edge-gpu-prepare-windows-vhd-generalized-image.md) o [Preparación de una imagen generalizada a partir de una imagen ISO](azure-stack-edge-gpu-prepare-windows-generalized-image-iso.md).

## <a name="about-vm-images"></a>Acerca de las imágenes de máquinas virtuales

Se puede utilizar un archivo VHD o VHDX de Windows para crear una imagen *especializada* o *generalizada*. En la tabla siguiente se resumen las diferencias principales entre las imágenes *especializadas* y *generalizadas*.

[!INCLUDE [about-vm-images-for-azure-stack-edge](../../includes/azure-stack-edge-about-vm-images.md)]

## <a name="workflow"></a>Flujo de trabajo

En líneas generales, el flujo de trabajo para implementar una máquina virtual a partir de una imagen especializada es el siguiente:

1. Copie el VHD en una cuenta de almacenamiento local en el dispositivo de GPU de Azure Stack Edge Pro.
1. Cree un disco administrado a partir del VHD.
1. Cree una máquina virtual a partir del disco administrado y asocie el disco administrado.

## <a name="prerequisites"></a>Requisitos previos

Antes de poder implementar una máquina virtual en el dispositivo a través de PowerShell, compruebe lo siguiente:

- Tiene acceso al cliente que se usará para conectarse al dispositivo.
    - El cliente ejecuta un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device).
    - El cliente está configurado para conectarse a la instancia local de Azure Resource Manager del dispositivo según las instrucciones que se indican en [Conexión a Azure Resource Manager en el dispositivo](azure-stack-edge-gpu-connect-resource-manager.md).

## <a name="verify-the-local-azure-resource-manager-connection"></a>Comprobación de la conexión de Azure Resource Manager local

Compruebe que el cliente puede conectarse a Azure Resource Manager local. 

1. Llame a las API del dispositivo local para autenticarse:

    ```powershell
    Login-AzureRMAccount -EnvironmentName <Environment Name>
    ```

2. Proporcione el nombre de usuario `EdgeArmUser` y la contraseña para conectarse mediante Azure Resource Manager. Si no recuerda la contraseña, consulte [Restablecimiento de la contraseña de Azure Resource Manager](azure-stack-edge-gpu-set-azure-resource-manager-password.md) y use esta contraseña para iniciar la sesión.

## <a name="deploy-vm-from-specialized-image"></a>Implementación de una máquina virtual a partir de una imagen especializada

Las secciones siguientes contienen instrucciones paso a paso para implementar una máquina virtual a partir de una imagen especializada.

## <a name="copy-vhd-to-local-storage-account-on-device"></a>Copia del VHD en la cuenta de almacenamiento local del dispositivo

Siga estos pasos para copiar el VHD en la cuenta de almacenamiento local:

1. Copie el VHD de origen en una cuenta de almacenamiento de blobs local en Azure Stack Edge.

1. Anote el URI resultante. Se usará en un paso posterior.

    Para crear una cuenta de almacenamiento local y acceder a ella, consulte las secciones [Creación de una cuenta de almacenamiento](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#create-a-storage-account) hasta [Carga de un archivo VHD](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#upload-a-vhd) en el artículo [Implementación de máquinas virtuales en el dispositivo Azure Stack Edge mediante Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md). 

## <a name="create-a-managed-disk-from-vhd"></a>Creación de un disco administrado a partir de un archivo VHD

Siga estos pasos para crear un disco administrado a partir de un archivo VHD que se cargó en la cuenta de almacenamiento anteriormente:

1. Configure algunos parámetros.

    ```powershell
    $VhdURI = <URI of VHD in local storage account>
    $DiskRG = <managed disk resource group>
    $DiskName = <managed disk name>    
    ```
    Esta es una salida de ejemplo.

    ```powershell
    PS C:\WINDOWS\system32> $VHDURI = "https://myasevmsa.blob.myasegpudev.wdshcsso.com/vhds/WindowsServer2016Datacenter.vhd"
    PS C:\WINDOWS\system32> $DiskRG = "myasevm1rg"
    PS C:\WINDOWS\system32> $DiskName = "myasemd1"
    ```
1. Cree un disco administrado.

    ```powershell
    $DiskConfig = New-AzureRmDiskConfig -Location DBELocal -CreateOption Import -SourceUri $VhdURI
    $disk = New-AzureRMDisk -ResourceGroupName $DiskRG -DiskName $DiskName -Disk $DiskConfig
    ```

    Esta es una salida de ejemplo. Aquí la ubicación se establece en la de la cuenta de almacenamiento local y siempre es `DBELocal` en todas las cuentas de almacenamiento local del dispositivo de GPU de Azure Stack Edge Pro. 

    ```powershell
    PS C:\WINDOWS\system32> $DiskConfig = New-AzureRmDiskConfig -Location DBELocal -CreateOption Import -SourceUri $VHDURI
    PS C:\WINDOWS\system32> $disk = New-AzureRMDisk -ResourceGroupName $DiskRG -DiskName $DiskName -Disk $DiskConfig
    PS C:\WINDOWS\system32>    
    ```
## <a name="create-a-vm-from-managed-disk"></a>Creación de una máquina virtual a partir de un disco administrado

Siga estos pasos para crear una máquina virtual a partir de un disco administrado:

1. Configure algunos parámetros.

    ```powershell
    $NicRG = <NIC resource group>
    $NicName = <NIC name>
    $IPConfigName = <IP config name>
    $PrivateIP = <IP address> #Optional
    
    $VMRG = <VM resource group>
    $VMName = <VM name>
    $VMSize = <VM size> 
    ```

    >[!NOTE]
    > El `PrivateIP` es opcional. Use este parámetro para asignar una dirección IP estática. De lo contrario, el valor predeterminado es una dirección IP dinámica que usa DHCP.

    Esta es una salida de ejemplo. En este ejemplo, se especifica el mismo grupo de recursos para todos los recursos de máquina virtual, aunque puede crear y especificar grupos de recursos distintos para los recursos, si es necesario.

    ```powershell
    PS C:\WINDOWS\system32> $NicRG = "myasevm1rg"
    PS C:\WINDOWS\system32> $NicName = "myasevmnic1"
    PS C:\WINDOWS\system32> $IPConfigName = "myaseipconfig1" 

    PS C:\WINDOWS\system32> $VMRG = "myasevm1rg"
    PS C:\WINDOWS\system32> $VMName = "myasetestvm1"
    PS C:\WINDOWS\system32> $VMSize = "Standard_D1_v2"   
    ```

1. Obtenga la información de la red virtual y cree una interfaz de red.

    En este ejemplo se supone que está creando una sola interfaz de red en la red virtual predeterminada `ASEVNET` que está asociada con el grupo de recursos predeterminado `ASERG`. Si es necesario, puede especificar una red virtual alternativa o crear varias interfaces de red. Para más información, consulte [Adición de una interfaz de red a una máquina virtual mediante Azure Portal](azure-stack-edge-gpu-manage-virtual-machine-network-interfaces-portal.md).

    ```powershell
    $armVN = Get-AzureRMVirtualNetwork -Name ASEVNET -ResourceGroupName ASERG
    $ipConfig = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName -SubnetId $armVN.Subnets[0].Id [-PrivateIpAddress $PrivateIP]
    $nic = New-AzureRmNetworkInterface -Name $NicName -ResourceGroupName $NicRG -Location DBELocal -IpConfiguration $ipConfig
    ```

    Esta es una salida de ejemplo.

    ```powershell
    PS C:\WINDOWS\system32> $armVN = Get-AzureRMVirtualNetwork -Name ASEVNET -ResourceGroupName ASERG
    PS C:\WINDOWS\system32> $ipConfig = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName -SubnetId $armVN.Subnets[0].Id
    PS C:\WINDOWS\system32> $nic = New-AzureRmNetworkInterface -Name $NicName -ResourceGroupName $NicRG -Location DBELocal -IpConfiguration $ipConfig
    WARNING: The output object type of this cmdlet will be modified in a future release.
    PS C:\WINDOWS\system32>    
    ```
1. Cree un objeto de configuración de máquina virtual.

    ```powershell
    $vmConfig = New-AzureRmVMConfig -VMName $VMName -VMSize $VMSize
    ```

    
1. Agregue la interfaz de red a la máquina virtual.

    ```powershell
    $vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id
    ```

1. Establezca las propiedades del disco del sistema operativo de la máquina virtual.

    ```powershell
    $vm = Set-AzureRmVMOSDisk -VM $vm -ManagedDiskId $disk.Id -StorageAccountType StandardLRS -CreateOption Attach –[Windows/Linux]
    ```
    La última marca de este comando será `-Windows` o `-Linux` en función del sistema operativo que se use para la máquina virtual.

1. Cree la máquina virtual.

    ```powershell
    New-AzureRmVM -ResourceGroupName $VMRG -Location DBELocal -VM $vm 
    ```

    Esta es una salida de ejemplo. 

    ```powershell
    PS C:\WINDOWS\system32> $vmConfig = New-AzureRmVMConfig -VMName $VMName -VMSize $VMSize
    PS C:\WINDOWS\system32> $vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id
    PS C:\WINDOWS\system32> $vm = Set-AzureRmVMOSDisk -VM $vm -ManagedDiskId $disk.Id -StorageAccountType StandardLRS -CreateOption Attach -Windows
    PS C:\WINDOWS\system32> New-AzureRmVM -ResourceGroupName $VMRG -Location DBELocal -VM $vm
    WARNING: Since the VM is created using premium storage or managed disk, existing standard storage account, myasevmsa, is used for
    boot diagnostics.    
    RequestId IsSuccessStatusCode StatusCode ReasonPhrase
    --------- ------------------- ---------- ------------
                             True         OK OK        
    PS C:\WINDOWS\system32>
    ```

## <a name="delete-vm-and-resources"></a>Eliminación de la máquina virtual y los recursos

En este artículo solo se usa un grupo de recursos para crear todos los recursos de la máquina virtual. Si elimina ese grupo de recursos, se eliminará la máquina virtual y todos los recursos asociados. 

1. En primer lugar, vea todos los recursos creados en el grupo de recursos.

    ```powershell
    Get-AzureRmResource -ResourceGroupName <Resource group name>
    ```
    Esta es una salida de ejemplo.
    
    ```powershell
    PS C:\WINDOWS\system32> Get-AzureRmResource -ResourceGroupName myasevm1rg
    
    
    Name              : myasemd1
    ResourceGroupName : myasevm1rg
    ResourceType      : Microsoft.Compute/disks
    Location          : dbelocal
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myasevm1rg/providers/Microsoft.Compute/disk
                        s/myasemd1
    
    Name              : myasetestvm1
    ResourceGroupName : myasevm1rg
    ResourceType      : Microsoft.Compute/virtualMachines
    Location          : dbelocal
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myasevm1rg/providers/Microsoft.Compute/virt
                        ualMachines/myasetestvm1
    
    Name              : myasevmnic1
    ResourceGroupName : myasevm1rg
    ResourceType      : Microsoft.Network/networkInterfaces
    Location          : dbelocal
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myasevm1rg/providers/Microsoft.Network/netw
                        orkInterfaces/myasevmnic1
    
    Name              : myasevmsa
    ResourceGroupName : myasevm1rg
    ResourceType      : Microsoft.Storage/storageaccounts
    Location          : dbelocal
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myasevm1rg/providers/Microsoft.Storage/stor
                        ageaccounts/myasevmsa
    
    PS C:\WINDOWS\system32>
    ```

1. Elimine el grupo de recursos y todos los recursos asociados.

    ```powershell
    Remove-AzureRmResourceGroup -ResourceGroupName <Resource group name>
    ```
    Esta es una salida de ejemplo.
    
    ```powershell
    PS C:\WINDOWS\system32> Remove-AzureRmResourceGroup -ResourceGroupName myasevm1rg
    
    Confirm
    Are you sure you want to remove resource group 'myasevm1rg'
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
    True
    PS C:\WINDOWS\system32>
    ```

1. Compruebe la fecha y hora en que se eliminó el grupo de recursos. Obtenga todos los grupos de recursos que existen en el dispositivo. 

    ```powershell
    Get-AzureRmResourceGroup
    ```
    Esta es una salida de ejemplo.

    ```powershell
    PS C:\WINDOWS\system32> Get-AzureRmResourceGroup

    ResourceGroupName : ase-image-resourcegroup
    Location          : dbelocal
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/ase-image-resourcegroup
    
    ResourceGroupName : ASERG
    Location          : dbelocal
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/ASERG
    
    ResourceGroupName : myaserg
    Location          : dbelocal
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myaserg
        
    PS C:\WINDOWS\system32>
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Preparación de una imagen generalizada desde un VHD de Windows para implementar máquinas virtuales en Azure Stack Edge Pro GPU](azure-stack-edge-gpu-prepare-windows-vhd-generalized-image.md)
- [Preparación de una imagen generalizada desde una ISO para implementar máquinas virtuales en Azure Stack Edge Pro GPU](azure-stack-edge-gpu-prepare-windows-generalized-image-iso.md)