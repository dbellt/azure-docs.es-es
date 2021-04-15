---
title: Administración de etiquetas de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU a través de Azure PowerShell
description: Se describe cómo crear y administrar etiquetas de máquinas virtuales para máquinas virtuales que se ejecutan en un dispositivo Azure Stack Edge mediante Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/06/2021
ms.author: alkohli
ms.openlocfilehash: 566ca1598857f67e25faea9f19d134c4a91bfb18
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106555986"
---
# <a name="manage-vm-tags-on-azure-stack-edge-via-azure-powershell"></a>Administración de etiquetas de máquinas virtuales en el dispositivo Azure Stack Edge a través de Azure PowerShell

En este artículo se describe cómo etiquetar máquinas virtuales (VM) que se ejecutan en los dispositivos Azure Stack Edge Pro con GPU mediante Azure PowerShell.

## <a name="about-tags"></a>Acerca de las etiquetas

Las etiquetas son pares clave-valor definidos por el usuario que se pueden asignar a un recurso o a un grupo de recursos. Puede aplicar etiquetas a las máquinas virtuales que se ejecutan en el dispositivo para organizarlas lógicamente en una taxonomía. Las etiquetas se pueden colocar en un recurso en el momento de su creación, o bien se pueden agregar a un recurso existente. Por ejemplo, puede aplicar el nombre `Organization` y el valor `Engineering` a todas las máquinas virtuales que usa el Departamento de ingeniería de su organización.

Para más información sobre las etiquetas, consulte [Administración de etiquetas a través de AzureRM PowerShell](/powershell/module/azurerm.tags/?view=azurermps-6.13.0&preserve-view=true).

## <a name="prerequisites"></a>Requisitos previos

Antes de poder implementar una máquina virtual en el dispositivo a través de PowerShell, compruebe lo siguiente:

- Tiene acceso al cliente que se usará para conectarse al dispositivo.
    - El cliente ejecuta un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device).
    - El cliente está configurado para conectarse a la instancia local de Azure Resource Manager del dispositivo según las instrucciones que se indican en [Conexión a Azure Resource Manager en el dispositivo](azure-stack-edge-gpu-connect-resource-manager.md).


## <a name="verify-connection-to-local-azure-resource-manager"></a>Comprobación de la conexión a Azure Resource Manager local

Asegúrese de que se pueden usar los pasos siguientes para acceder al dispositivo desde el cliente.

Compruebe que el cliente puede conectarse a Azure Resource Manager local. 

1. Llame a las API del dispositivo local para autenticarse:

    ```powershell
    login-AzureRMAccount -EnvironmentName <Environment Name> -TenantId c0257de7-538f-415c-993a-1b87a031879d  
    ```

1. Proporcione el nombre de usuario `EdgeArmUser` y la contraseña para conectarse mediante Azure Resource Manager. Si no recuerda la contraseña, consulte [Restablecimiento de la contraseña de Azure Resource Manager](azure-stack-edge-gpu-set-azure-resource-manager-password.md) y use esta contraseña para iniciar la sesión.


## <a name="add-a-tag-to-a-vm"></a>Adición de una etiqueta a una máquina virtual

1. Configure algunos parámetros.

    ```powershell
    $VMName = <VM Name>
    $VMRG = <VM Resource Group>
    $TagName = <Tag Name>
    $TagValue = <Tag Value>   
    ```

    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $VMName = "myasetestvm1"
    PS C:\WINDOWS\system32> $VMRG = "myaserg2"
    PS C:\WINDOWS\system32> $TagName = "Organization"
    PS C:\WINDOWS\system32> $TagValue = "Sales"
    ```

2. Obtenga el objeto de máquina virtual y sus etiquetas.

   ```powershell
   $VirtualMachine = Get-AzureRmVM -ResourceGroupName $VMRG -Name $VMName
   $tags = $VirtualMachine.Tags
   ```

3. Agregue la etiqueta y actualice la máquina virtual. La actualización de la máquina virtual puede tardar unos minutos.

    Puede usar la marca **-Force** opcional para ejecutar el comando sin la confirmación del usuario.

    ```powershell
    $tags.Add($TagName, $TagValue)
    Set-AzureRmResource -ResourceId $VirtualMachine.Id -Tag $tags [-Force]
    ```

    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $VirtualMachine = Get-AzureRMVM -ResourceGroupName $VMRG -Name $VMName
    PS C:\WINDOWS\system32> $tags = $VirtualMachine.Tags
    PS C:\WINDOWS\system32> $tags.Add($TagName, $TagValue)
    PS C:\WINDOWS\system32> Set-AzureRmResource -ResourceID $VirtualMachine.ID -Tag $tags -Force
    
    Name              : myasetestvm1
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myaserg2/providers/Microsoft.Compute/virtua
                        lMachines/myasetestvm1
    ResourceName      : myasetestvm1
    ResourceType      : Microsoft.Compute/virtualMachines
    ResourceGroupName : myaserg2
    Location          : dbelocal
    SubscriptionId    : 992601bc-b03d-4d72-598e-d24eac232122
    Tags              : {Organization}
    Properties        : @{vmId=958c0baa-e143-4d8a-82bd-9c6b1ba45e86; hardwareProfile=; storageProfile=; osProfile=; networkProfile=;
                        provisioningState=Succeeded}
    
    PS C:\WINDOWS\system32>
    ```

Para más información, consulte [Add-AzureRMTag](/powershell/module/azurerm.tags/remove-azurermtag?view=azurermps-6.13.0&preserve-view=true).

## <a name="view-tags-of-a-vm"></a>Visualización de las etiquetas de una máquina virtual

Puede ver las etiquetas aplicadas a una máquina virtual específica que se ejecuta en el dispositivo. 

1. Defina los parámetros asociados a la máquina virtual cuyas etiquetas desea ver.

   ```powershell
   $VMName = <VM Name>
   $VMRG = <VM Resource Group>
   ```
    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $VMName = "myasetestvm1"
    PS C:\WINDOWS\system32> $VMRG = "myaserg2"
    PS C:\WINDOWS\system32> $TagName = "Organization"
    PS C:\WINDOWS\system32> $TagValue = "Sales"
    ```
1. Obtenga el objeto de máquina virtual y sus etiquetas.

   ```powershell
   $VirtualMachine = Get-AzureRmVM -ResourceGroupName $VMRG -Name $VMName
   $VirtualMachine.Tags
   ```
    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $VirtualMachine = Get-AzureRMVM -ResourceGroupName $VMRG -Name $VMName
    PS C:\WINDOWS\system32> $VirtualMachine

    ResourceGroupName : myaserg2
    Id                : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myaserg2/providers/Microsoft.Compute/virtua
    lMachines/myasetestvm1
    VmId              : 958c0baa-e143-4d8a-82bd-9c6b1ba45e86
    Name              : myasetestvm1
    Type              : Microsoft.Compute/virtualMachines
    Location          : dbelocal
    Tags              : {"Organization":"Sales"}
    HardwareProfile   : {VmSize}
    NetworkProfile    : {NetworkInterfaces}
    OSProfile         : {ComputerName, AdminUsername, LinuxConfiguration, Secrets}
    ProvisioningState : Succeeded
    StorageProfile    : {ImageReference, OsDisk, DataDisks}
    
    PS C:\WINDOWS\system32>
    ```
## <a name="view-tags-for-all-resources"></a>Visualización de las etiquetas de todos los recursos

Para ver la lista actual de etiquetas de todos los recursos de la suscripción de Azure Resource Manager local (diferente de la suscripción de Azure) del dispositivo, use el comando `Get-AzureRMTag`.


Este es un ejemplo de salida cuando se ejecutan varias máquinas virtuales en el dispositivo y desea ver todas las etiquetas en todas las máquinas virtuales.

```powershell
PS C:\WINDOWS\system32> Get-AzureRMTag

Name         Count
----         -----
Organization 3

PS C:\WINDOWS\system32>
```

La salida anterior indica que hay tres etiquetas `Organization` en las máquinas virtuales que se ejecutan en el dispositivo.

Para ver más detalles, use el parámetro `-Detailed`.

```powershell
PS C:\WINDOWS\system32> Get-AzureRMTag -Detailed |fl

Name        : Organization
ValuesTable :
              Name         Count
              ===========  =====
              Engineering  2
              Sales        1

Count       : 3
Values      : {Engineering, Sales}

PS C:\WINDOWS\system32>
```

La salida anterior indica que fuera de las tres etiquetas, dos máquinas virtuales se etiquetan como `Engineering` y una se etiqueta como perteneciente a `Sales`.

## <a name="remove-a-tag-from-a-vm"></a>Eliminación de una etiqueta de una máquina virtual

1. Configure algunos parámetros. 

    ```powershell
    $VMName = <VM Name>
    $VMRG = <VM Resource Group>
    $TagName = <Tag Name>
   ``` 
    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $VMName = "myaselinuxvm1"
    PS C:\WINDOWS\system32> $VMRG = "myaserg1"
    PS C:\WINDOWS\system32> $TagName = "Organization" 
    ```
2. Obtenga el objeto de VM.

    ```powershell
    $VirtualMachine = Get-AzureRmVM -ResourceGroupName $VMRG -Name $VMName
    $VirtualMachine   
    ```   

    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $VirtualMachine = Get-AzureRMVM -ResourceGroupName $VMRG -Name $VMName
    ResourceGroupName : myaserg1
    Id                : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGroups/myaserg1/providers/Microsoft.Compute/virtualMachines/myaselinuxvm1
    VmId              : 290b3fdd-0c99-4905-9ea1-cf93cd6f25ee
    Name              : myaselinuxvm1
    Type              : Microsoft.Compute/virtualMachines
    Location          : dbelocal
    Tags              : {"Organization":"Engineering"}
    HardwareProfile   : {VmSize}
    NetworkProfile    : {NetworkInterfaces}
    OSProfile         : {ComputerName, AdminUsername, LinuxConfiguration, Secrets}
    ProvisioningState : Succeeded
    StorageProfile    : {ImageReference, OsDisk, DataDisks}
                                                                              PS C:\WINDOWS\system32> 
    ```
3. Quite la etiqueta y actualice la máquina virtual. Use la marca `-Force` opcional para ejecutar el comando sin la confirmación del usuario.

    ```powershell
    $tags = $VirtualMachine.Tags
    $tags.Remove($TagName)
    Set-AzureRmResource -ResourceId $VirtualMachine.Id -Tag $tags [-Force]
    ```
    A continuación, se incluye una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> $tags = $Virtualmachine.Tags
    PS C:\WINDOWS\system32> $tags
    Key          Value
    ---          -----
    Organization Engineering
    PS C:\WINDOWS\system32> $tags.Remove($TagName)
    True
    PS C:\WINDOWS\system32> Set-AzureRMResource -ResourceID $VirtualMachine.ID -Tag $tags -Force
    Name              : myaselinuxvm1
    ResourceId        : /subscriptions/992601bc-b03d-4d72-598e-d24eac232122/resourceGrou
                        ps/myaserg1/providers/Microsoft.Compute/virtualMachines/myaselin
                        uxvm1
    ResourceName      : myaselinuxvm1
    ResourceType      : Microsoft.Compute/virtualMachines
    ResourceGroupName : myaserg1
    Location          : dbelocal
    SubscriptionId    : 992601bc-b03d-4d72-598e-d24eac232122
    Tags              : {}
    Properties        : @{vmId=290b3fdd-0c99-4905-9ea1-cf93cd6f25ee; hardwareProfile=;
                        storageProfile=; osProfile=; networkProfile=;
                        provisioningState=Succeeded}
    PS C:\WINDOWS\system32>
    ```


## <a name="next-steps"></a>Pasos siguientes

Aprenda a [administrar etiquetas a través de AzureRM PowerShell](/powershell/module/azurerm.tags/?view=azurermps-6.13.0&preserve-view=true).
