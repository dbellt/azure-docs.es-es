---
title: Creación de un nuevo conmutador virtual en Azure Stack Edge a través de PowerShell
description: Se describe cómo crear un conmutador virtual en un dispositivo Azure Stack Edge mediante Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/06/2021
ms.author: alkohli
ms.openlocfilehash: 1ad86695510a8fe93bbeeab27db53f5afbef92fd
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106555980"
---
# <a name="create-a-new-virtual-switch-in-azure-stack-edge-pro-gpu-via-powershell"></a>Creación de un nuevo conmutador virtual en Azure Stack Edge Pro con GPU a través de PowerShell

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe cómo crear un nuevo conmutador virtual en un dispositivo Azure Stack Edge Pro con GPU. Por ejemplo, si desea que las máquinas virtuales se conecten a través de un puerto de red físico diferente, debe crear un nuevo conmutador virtual.

## <a name="vm-deployment-workflow"></a>Flujo de trabajo de implementación de una máquina virtual

1. Conéctese a la interfaz de PowerShell del dispositivo.
2. Consulte las interfaces de red físicas disponibles.
3. Crear un conmutador virtual.
4. Compruebe la red virtual y la subred que se crean automáticamente.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

- Tiene acceso a un equipo cliente que puede tener acceso a la interfaz de PowerShell del dispositivo. Consulte [Conectarse a la interfaz de PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface). 

    El equipo cliente debe ejecutar un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device).

- Usa la interfaz de usuario local para habilitar el proceso en una de las interfaces de red físicas en el dispositivo según las instrucciones de [Habilitación de la red de proceso](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#enable-compute-network) en el dispositivo. 


## <a name="connect-to-the-powershell-interface"></a>Conectarse a la interfaz de PowerShell

[Conéctese a la interfaz de PowerShell del dispositivo](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).

## <a name="query-available-network-interfaces"></a>Consulta de las interfaces de red disponibles

1. Use el siguiente comando para mostrar una lista de interfaces de red físicas en las que puede crear un nuevo conmutador virtual. Seleccionará una de estas interfaces de red.

    ```powershell
    Get-NetAdapter -Physical
    ```
    A continuación, se incluye una salida de ejemplo:
    
    ```powershell
        [10.100.10.10]: PS>Get-NetAdapter -Physical
        
        Name                      InterfaceDescription                    ifIndex Status       MacAddress       LinkSpeed
        ----                      --------------------                    ------- ------       ----------        -----
        Port2                     QLogic 2x1GE+2x25GE QL41234HMCU NIC ...      12 Up           34-80-0D-05-26-EA ...ps
        Ethernet                  Remote NDIS Compatible Device                11 Up           F4-02-70-CD-41-39 ...ps
        Port1                     QLogic 2x1GE+2x25GE QL41234HMCU NI...#3       9 Up           34-80-0D-05-26-EB ...ps
        Port5                     Mellanox ConnectX-4 Lx Ethernet Ad...#2       8 Up           0C-42-A1-C0-E3-99 ...ps
        Port3                     QLogic 2x1GE+2x25GE QL41234HMCU NI...#4       7 Up           34-80-0D-05-26-E9 ...ps
        Port6                     Mellanox ConnectX-4 Lx Ethernet Adapter       6 Up           0C-42-A1-C0-E3-98 ...ps
        Port4                     QLogic 2x1GE+2x25GE QL41234HMCU NI...#2       4 Up           34-80-0D-05-26-E8 ...ps
        
        [10.100.10.10]: PS>
    ```
2. Elija una interfaz de red que:

    - Esté en estado **Activo**. 
    - No se use en ningún conmutador virtual existente. Actualmente, solo se puede configurar un conmutador virtual por interfaz de red. 
    
    Para comprobar el conmutador virtual existente y la asociación de la interfaz de red, ejecute el comando `Get-HcsExternalVirtualSwitch`.
 
    Esta es una salida de ejemplo.

    ```powershell
    [10.100.10.10]: PS>Get-HcsExternalVirtualSwitch

    Name                          : vSwitch1
    InterfaceAlias                : {Port2}
    EnableIov                     : True
    MacAddressPools               :
    IPAddressPools                : {}
    ConfigurationSource           : Dsc
    EnabledForCompute             : True
    SupportsAcceleratedNetworking : False
    DbeDhcpHostVnicName           : f4a92de8-26ed-4597-a141-cb233c2ba0aa
    Type                          : External
    
    [10.100.10.10]: PS>
    ```
    En esta instancia, el puerto 2 está asociado a un conmutador virtual existente y no debe usarse.

## <a name="create-a-virtual-switch"></a>Crear un conmutador virtual

Use el siguiente cmdlet para crear un nuevo conmutador virtual en la interfaz de red especificada. Una vez completada esta operación, las instancias de proceso pueden usar la nueva red virtual.

```powershell
Add-HcsExternalVirtualSwitch -InterfaceAlias <Network interface name> -WaitForSwitchCreation $true
```

Use el comando `Get-HcsExternalVirtualSwitch` para identificar el conmutador recién creado. El nuevo conmutador que se ha creado se denomina `vswitch-<InterfaceAlias>`. 

A continuación, se incluye una salida de ejemplo:

```powershell
[10.100.10.10]: P> Add-HcsExternalVirtualSwitch -InterfaceAlias Port5 -WaitForSwitchCreation $true
[10.100.10.10]: PS>Get-HcsExternalVirtualSwitch

Name                          : vSwitch1
InterfaceAlias                : {Port2}
EnableIov                     : True
MacAddressPools               :
IPAddressPools                : {}
ConfigurationSource           : Dsc
EnabledForCompute             : True
SupportsAcceleratedNetworking : False
DbeDhcpHostVnicName           : f4a92de8-26ed-4597-a141-cb233c2ba0aa
Type                          : External

Name                          : vswitch-Port5
InterfaceAlias                : {Port5}
EnableIov                     : True
MacAddressPools               :
IPAddressPools                :
ConfigurationSource           : Dsc
EnabledForCompute             : False
SupportsAcceleratedNetworking : False
DbeDhcpHostVnicName           : 9b301c40-3daa-49bf-a20b-9f7889820129
Type                          : External

[10.100.10.10]: PS>
```

## <a name="verify-network-subnet"></a>Comprobación de la red y la subred 

Después de crear el nuevo conmutador virtual, Azure Stack Edge Pro con GPU crea automáticamente una red virtual y una subred que le corresponde. Puede usar esta red virtual al crear máquinas virtuales.

<!--To identify the virtual network and subnet associated with the new switch that you created, use the `Get-HcsVirtualNetwork` command. This cmdlet will be released in April some time. -->

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU a través de Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md)

- [Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU a través de Azure Portal](azure-stack-edge-gpu-deploy-virtual-machine-portal.md)
