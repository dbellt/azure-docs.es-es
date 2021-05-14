---
title: Habilitación de la extensión de VM mediante Azure PowerShell
description: En este artículo se describe cómo implementar extensiones de máquina virtual en servidores habilitados para Azure Arc que se ejecutan en entornos de nube híbrida mediante Azure PowerShell.
ms.date: 05/06/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 59e33a8e4e5a2bc0cc8f620bd4b7fb8b65c5d7e9
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108803213"
---
# <a name="enable-azure-vm-extensions-using-azure-powershell"></a>Habilitación de las extensiones de VM de Azure mediante Azure PowerShell

En este artículo se muestra cómo implementar y desinstalar extensiones de VM de Azure, compatibles con servidores habilitados para Azure Arc, en una máquina híbrida Linux o Windows mediante Azure PowerShell.

> [!NOTE]
> Los servidores habilitados de Azure Arc no admiten la implementación y administración de extensiones de máquina virtual en máquinas virtuales de Azure. Para las máquinas virtuales de Azure, consulte el siguiente artículo de [información general sobre la extensión de máquina virtual](../../virtual-machines/extensions/overview.md).

## <a name="prerequisites"></a>Prerrequisitos

- Un equipo con Azure PowerShell. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).

Antes de usar Azure PowerShell para administrar las extensiones de máquina virtual en el servidor híbrido administrado por los servidores habilitados para Arc, tendrá que instalar el módulo `Az.ConnectedMachine`. Ejecute el comando siguiente en el servidor habilitado para Arc:

`Install-Module -Name Az.ConnectedMachine`.

Una vez que haya finalizado la instalación, se devuelve el mensaje siguiente:

`The installed extension `Az.ConnectedMachine` is experimental and not covered by customer support. Please use with discretion.`

## <a name="enable-extension"></a>Habilitación de una extensión

Para habilitar una extensión de máquina virtual en el servidor con Arc habilitado, use [New-AzConnectedMachineExtension](/powershell/module/az.connectedmachine/new-azconnectedmachineextension) con los parámetros `-Name`, `-ResourceGroupName`, `-MachineName`, `-Location`, `-Publisher`,-`ExtensionType` y `-Settings`.

En el ejemplo siguiente se habilita la extensión de VM de Log Analytics en un servidor Linux habilitado para Arc:

```powershell
PS C:\> $Setting = @{ "workspaceId" = "workspaceId" }
PS C:\> $protectedSetting = @{ "workspaceKey" = "workspaceKey" }
PS C:\> New-AzConnectedMachineExtension -Name OMSLinuxAgent -ResourceGroupName "myResourceGroup" -MachineName "myMachine" -Location "eastus" -Publisher "Microsoft.EnterpriseCloud.Monitoring" -Settings $Setting -ProtectedSetting $protectedSetting -ExtensionType "OmsAgentForLinux"
```

Para habilitar la extensión de máquina virtual de Log Analytics en un servidor Windows habilitado para Arc, cambie el valor del parámetro `-ExtensionType` a `"MicrosoftMonitoringAgent"` en el ejemplo anterior.

En el ejemplo siguiente se habilita la extensión de script personalizado en un servidor habilitado para Arc:

```powershell
PS C:\> $Setting = @{ "commandToExecute" = "powershell.exe -c Get-Process" }
PS C:\> New-AzConnectedMachineExtension -Name custom -ResourceGroupName myResourceGroup -MachineName myMachineName -Location eastus -Publisher "Microsoft.Compute"  -Settings $Setting -ExtensionType CustomScriptExtension
```

### <a name="key-vault-vm-extension-preview"></a>Extensión de máquina virtual de Key Vault (versión preliminar)

> [!WARNING]
> Los clientes de PowerShell suelen agregar `\` a `"` en el archivo settings.json, lo cual producirá en akvvm_service el error: `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`.

En el ejemplo siguiente se habilita la extensión de máquina virtual de Key Vault (versión preliminar) en un servidor habilitado para Arc:

```powershell
# Build settings
    $settings = @{
      secretsManagementSettings = @{
       observedCertificates = @(
        "observedCert1"
       )
      certificateStoreLocation = "myMachineName" # For Linux use "/var/lib/waagent/Microsoft.Azure.KeyVault.Store/"
      certificateStore = "myCertificateStoreName"
      pollingIntervalInS = "pollingInterval"
      }
    authenticationSettings = @{
     msiEndpoint = "http://localhost:40342/metadata/identity"
     }
    }

    $resourceGroup = "resourceGroupName"
    $machineName = "myMachineName"
    $location = "regionName"

    # Start the deployment
    New-AzConnectedMachineExtension -ResourceGroupName $resourceGRoup -Location $location -MachineName $machineName -Name "KeyVaultForWindows or KeyVaultforLinux" -Publisher "Microsoft.Azure.KeyVault" -ExtensionType "KeyVaultforWindows or KeyVaultforLinux" -Setting (ConvertTo-Json $settings)
```

## <a name="list-extensions-installed"></a>Enumeración de extensiones instaladas

Para obtener una lista de las extensiones de VM en el servidor habilitado para Arc, use [Get-AzConnectedMachineExtension](/powershell/module/az.connectedmachine/get-azconnectedmachineextension) con los parámetros `-MachineName` y `-ResourceGroupName`.

Ejemplo:

```powershell
Get-AzConnectedMachineExtension -ResourceGroupName myResourceGroup -MachineName myMachineName

Name    Location  PropertiesType        ProvisioningState
----    --------  --------------        -----------------
custom  westus2   CustomScriptExtension Succeeded
```

## <a name="remove-an-installed-extension"></a>Eliminación de una extensión instalada

Para quitar una extensión de VM instalada en el servidor habilitado para Arc, use [Remove-AzConnectedMachineExtension](/powershell/module/az.connectedmachine/remove-azconnectedmachineextension) con los parámetros `-Name`, `-MachineName` y `-ResourceGroupName`.

Por ejemplo, para quitar la extensión de VM de Log Analytics para Linux, ejecute el siguiente comando:

```powershell
Remove-AzConnectedMachineExtension -MachineName myMachineName -ResourceGroupName myResourceGroup -Name OmsAgentforLinux
```

## <a name="next-steps"></a>Pasos siguientes

- Puede implementar, administrar y quitar extensiones de VM con la [CLI de Azure](manage-vm-extensions-cli.md), desde [Azure Portal](manage-vm-extensions-portal.md) o con [plantillas de Azure Resource Manager](manage-vm-extensions-template.md).

- Puede encontrar información de solución de problemas en la [guía de solución de problemas de las extensiones de máquina virtual](troubleshoot-vm-extensions.md).
