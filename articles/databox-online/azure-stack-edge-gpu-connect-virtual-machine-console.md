---
title: Conexión a la consola de la máquina virtual en el dispositivo Azure Stack Edge Pro GPU
description: Describe cómo conectarse a la consola de la máquina virtual para una máquina virtual que se ejecuta en el dispositivo Azure Stack Edge Pro GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 06/04/2021
ms.author: alkohli
ms.openlocfilehash: 5bcd23c9a515155b9dc7cfdc04a3fafe122474ae
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111986361"
---
# <a name="connect-to-a-virtual-machine-console-on-an-azure-stack-edge-pro-gpu-device"></a>Conexión a la consola de una máquina virtual en un dispositivo Azure Stack Edge Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

La solución Azure Stack Edge Pro GPU ejecuta cargas de trabajo no contenedorizadas a través de las máquinas virtuales. En este artículo se describe cómo conectarse a la consola de una máquina virtual implementada en el dispositivo. 

La consola de máquina virtual le permite acceder a las máquinas virtuales con las características de teclado, mouse y pantalla mediante las herramientas de escritorio remoto disponibles de forma habitual. Puede acceder a la consola y solucionar cualquier problema que se haya producido al implementar una máquina virtual en el dispositivo. Podrá conectarse a la consola de la máquina virtual aunque la máquina virtual no se haya aprovisionado.


## <a name="workflow"></a>Flujo de trabajo

El flujo de trabajo de alto nivel consta de los siguientes pasos:

1. Conéctese a la interfaz de PowerShell del dispositivo.
1. Habilite el acceso de la consola a la máquina virtual.
1. Conéctese a la máquina virtual mediante el protocolo de escritorio remoto (RDP).
1. Revoque el acceso de la consola a la máquina virtual.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de completar estos requisitos previos:

#### <a name="for-your-device"></a>En el dispositivo

Debe tener acceso a un dispositivo Azure Stack Edge Pro GPU que esté activado. El dispositivo debe tener una o más máquinas virtuales implementadas. Puede implementar las máquinas virtuales mediante Azure PowerShell, plantillas o Azure Portal.

#### <a name="for-client-accessing-the-device"></a>Para el cliente que va a acceder al dispositivo

Asegúrese de que tiene acceso a un sistema cliente que cumpla estos criterios:

- Puede acceder a la interfaz de PowerShell del dispositivo. El equipo cliente usa un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device).
- El cliente ejecuta Windows PowerShell 7.0 o una versión posterior. Esta versión de PowerShell funciona con clientes de Windows, Mac y Linux. Consulte las instrucciones para [instalar PowerShell 7.0](/powershell/scripting/whats-new/what-s-new-in-powershell-70?view=powershell-7.1&preserve-view=true).
- Incorpora funcionalidad de escritorio remoto. En función de si usa Windows, macOS o Linux, debe instalar uno de estos [clientes de Escritorio remoto](/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients). En este artículo se proporcionan instrucciones para [Escritorio remoto de Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop#install-the-client) y [FreeRDP](https://www.freerdp.com/). <!--Which version of FreeRDP to use?-->


## <a name="connect-to-vm-console"></a>Conexión a la consola de la máquina virtual

Siga estos pasos para conectarse a la consola de la máquina virtual en el dispositivo.

### <a name="connect-to-the-powershell-interface-on-your-device"></a>Conexión a la interfaz de PowerShell en el dispositivo

El primer paso consiste en [conectarse a la interfaz de PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) del dispositivo. 

### <a name="enable-console-access-to-the-vm"></a>Habilitación del acceso de la consola a la máquina virtual

1.  En la interfaz de PowerShell, ejecute el siguiente comando para habilitar el acceso a la consola de la máquina virtual.

    ```powershell
    Grant-HcsVMConnectAccess -ResourceGroupName <VM resource group> -VirtualMachineName <VM name>
    ```
2. En el resultado del ejemplo, tome nota del identificador de la máquina virtual. Necesitará esta información más tarde.

    ```powershell
    [10.100.10.10]: PS>Grant-HcsVMConnectAccess -ResourceGroupName mywindowsvm1rg -VirtualMachineName mywindowsvm1
        
    VirtualMachineId       : 81462e0a-decb-4cd4-96e9-057094040063
    VirtualMachineHostName : 3V78B03
    ResourceGroupName      : mywindowsvm1rg
    VirtualMachineName     : mywindowsvm1
    Id                     : 81462e0a-decb-4cd4-96e9-057094040063
    [10.100.10.10]: PS>
    ```

### <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

Ahora puede usar un cliente de Escritorio remoto para conectarse a la consola de la máquina virtual.

#### <a name="use-windows-remote-desktop"></a>Uso del Escritorio remoto de Windows

1. Cree un nuevo archivo de texto y especifique el texto siguiente.

    ```
    pcb:s:<VM ID from PowerShell>;EnhancedMode=0
    full address:s:<IP address of the device>   
    server port:i:2179
    username:s:EdgeARMUser
    negotiate security layer:i:0
    ```
1. Guarde el archivo como * *.rdp* en el sistema cliente. Usará este perfil para conectarse a la máquina virtual.
1. Haga doble clic en el perfil para conectarse a la máquina virtual. Utilice estas credenciales:

    - **Nombre de usuario**: inicie sesión como EdgeARMUser.
    - **Contraseña**: proporcione la contraseña local de Azure Resource Manager para el dispositivo. Si olvidó la contraseña, consulte [Restablecimiento de la contraseña de Azure Resource Manager mediante Azure Portal](azure-stack-edge-gpu-set-azure-resource-manager-password.md#reset-password-via-the-azure-portal). 

#### <a name="use-freerdp"></a>Uso de FreeRDP

Si usa FreeRDP en el cliente Linux, ejecute el siguiente comando: 

```powershell
./wfreerdp /u:EdgeARMUser /vmconnect:<VM ID from PowerShell> /v:<IP address of the device>
```

## <a name="revoke-vm-console-access"></a>Revocación del acceso a la consola de la máquina virtual

Para revocar el acceso a la consola de la máquina virtual, vuelva a la interfaz de PowerShell del dispositivo. Ejecute el siguiente comando:

```
Revoke-HcsVMConnectAccess -ResourceGroupName <VM resource group> -VirtualMachineName <VM name>
```
A continuación, se incluye una salida de ejemplo:

```powershell
[10.100.10.10]: PS>Revoke-HcsVMConnectAccess -ResourceGroupName mywindowsvm1rg -VirtualMachineName mywindowsvm1

VirtualMachineId       : 81462e0a-decb-4cd4-96e9-057094040063
VirtualMachineHostName : 3V78B03
ResourceGroupName      : mywindowsvm1rg
VirtualMachineName     : mywindowsvm1
Id                     : 81462e0a-decb-4cd4-96e9-057094040063

[10.100.10.10]: PS>
```
> [!NOTE] 
> Cuando termine de usar la consola de la máquina virtual, es recomendable que revoque el acceso o cierre la ventana de PowerShell para salir de la sesión. 

## <a name="next-steps"></a>Pasos siguientes

- Solución de problemas de [implementaciones de máquinas virtuales](azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning.md) en Azure Portal.
<!--Make "VM guest logs" first link when article is available.-->