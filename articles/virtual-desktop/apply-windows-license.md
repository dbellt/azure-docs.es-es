---
title: 'Aplicación de la licencia de Windows a máquinas virtuales de host de sesión: Azure'
description: En este artículo se describe cómo aplicar la licencia de Windows a máquinas virtuales de Azure Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 08/14/2019
ms.author: helohr
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: eaf8bd06493fb105207024b3b78c584ad58d4149
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745410"
---
# <a name="apply-windows-license-to-session-host-virtual-machines"></a>Aplicación de la licencia de Windows a máquinas virtuales de host de sesión

Los clientes con una licencia adecuada para ejecutar cargas de trabajo de Azure Virtual Desktop pueden aplicar una licencia de Windows a sus máquinas virtuales de host de sesión y ejecutarlas sin necesidad de pagar por otra licencia. Para más información, consulte [Precios de Azure Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="ways-to-use-your-azure-virtual-desktop-license"></a>Formas de usar la licencia de Azure Virtual Desktop
Las licencias de Azure Virtual Desktop permiten aplicar una licencia a cualquier máquina virtual de Windows o de Windows Server que esté registrada como un host de sesión en un grupo de hosts y que reciba conexiones de usuario. Esta licencia no se aplica a las máquinas virtuales que se ejecuten como servidores de recursos compartidos de archivos, controladores de dominio, etc.

Hay varias maneras de usar la licencia de Azure Virtual Desktop:
- Puede crear un grupo de hosts y sus máquinas virtuales de host de sesión mediante la [oferta de Azure Marketplace](./create-host-pools-azure-marketplace.md). Las máquinas virtuales creadas de esta manera tienen la licencia aplicada automáticamente.
- Puede crear un grupo de hosts y sus máquinas virtuales de host de sesión mediante la [plantilla de Azure Resource Manager de GitHub](./virtual-desktop-fall-2019/create-host-pools-arm-template.md). Las máquinas virtuales creadas de esta manera tienen la licencia aplicada automáticamente.
- Puede aplicar una licencia a una máquina virtual de host de sesión existente. Para ello, siga primero las instrucciones indicadas en [Creación de un grupo host con PowerShell](./create-host-pools-powershell.md) para crear un grupo de hosts y las máquinas virtuales asociadas. Después, vuelva a este artículo para obtener información sobre cómo aplicar la licencia.

## <a name="apply-a-windows-license-to-a-session-host-vm"></a>Aplicación de una licencia de Windows a una máquina virtual de host de sesión
Asegúrese de que tiene [la versión de Azure PowerShell más reciente instalada y configurada](/powershell/azure/). Ejecute el siguiente cmdlet de PowerShell para aplicar la licencia de Windows:

```powershell
$vm = Get-AzVM -ResourceGroup <resourceGroupName> -Name <vmName>
$vm.LicenseType = "Windows_Client"
Update-AzVM -ResourceGroupName <resourceGroupName> -VM $vm
```

## <a name="verify-your-session-host-vm-is-utilizing-the-licensing-benefit"></a>Comprobación de que la máquina virtual de host de sesión usa la ventaja de licencia
Después de implementar la máquina virtual, ejecute este cmdlet y compruebe el tipo de licencia:
```powershell
Get-AzVM -ResourceGroupName <resourceGroupName> -Name <vmName>
```

Si la máquina virtual de host de sesión tiene aplicada la licencia de Windows, aparecerá algo parecido a esto:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Client
```

Si la máquina virtual no tiene aplicada la licencia de Windows, aparecerá algo parecido a esto:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

Ejecute el siguiente cmdlet para ver una lista de todas las máquinas virtuales de host de sesión que tienen aplicada la licencia de Windows en la suscripción de Azure:

```powershell
$vms = Get-AzVM
$vms | Where-Object {$_.LicenseType -like "Windows_Client"} | Select-Object ResourceGroupName, Name, LicenseType
```

## <a name="requirements-for-deploying-windows-server-remote-desktop-services"></a>Requisitos para implementar Servicios de Escritorio remoto de Windows Server

Si implementa Windows Server 2019, 2016 o 2012 R2 como hosts de Azure Virtual Desktop en la implementación, un servidor de licencias de Servicios de Escritorio remoto debe ser accesible desde esas máquinas virtuales. El servidor de licencias de Servicios de Escritorio remoto se puede encontrar en el entorno local o en Azure. Para obtener más información, consulte [Activación del servidor de licencias de Servicios de Escritorio remoto](/windows-server/remote/remote-desktop-services/rds-activate-license-server).
