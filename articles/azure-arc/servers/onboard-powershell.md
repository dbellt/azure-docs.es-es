---
title: Conexión de máquinas híbridas a Azure mediante PowerShell
description: En este artículo se aprende a instalar el agente y a conectar una máquina a Azure mediante servidores habilitados para Azure Arc. Puede hacer esto con PowerShell.
ms.date: 10/28/2020
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d6963a53ac14c9d6727a8d53e781bc8b8389b76e
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831625"
---
# <a name="connect-hybrid-machines-to-azure-by-using-powershell"></a>Conexión de máquinas híbridas a Azure mediante PowerShell

En el caso de los servidores habilitados para Azure Arc, puede llevar a cabo una serie de pasos manuales a fin de habilitarlos para admitir una o varias máquinas Windows o Linux en el entorno. Si no, puede usar el cmdlet de PowerShell [Connect-AzConnectedMachine](/powershell/module/az.connectedmachine/remove-azconnectedmachine) para descargar el agente de Connected Machine, instalarlo y registrar la máquina en Azure Arc. El cmdlet descarga el paquete del agente de Windows (Windows Installer) del Centro de descarga de Microsoft y el paquete del agente de Linux desde el repositorio de paquetes de Microsoft.

Este método requiere que tenga permisos de administrador en la máquina para instalar y configurar el agente. En Linux, mediante la cuenta raíz, y en Windows, usted es miembro del grupo local de administradores. Puede realizar este proceso de forma interactiva o remota en un servidor Windows mediante la [Comunicación remota de PowerShell](/powershell/scripting/learn/ps101/08-powershell-remoting).

Antes de comenzar, revise los [requisitos previos](agent-overview.md#prerequisites) y compruebe que la suscripción y los recursos los cumplen. Para obtener información sobre las regiones admitidas y otras consideraciones relacionadas, consulte [Regiones de Azure admitidas](overview.md#supported-regions).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

- Una máquina con Azure PowerShell. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).

PowerShell se usa para administrar las extensiones de máquina virtual en los servidores híbridos administrados por los servidores habilitados para Azure Arc. Antes de usar PowerShell, instale el módulo `Az.ConnectedMachine`. Ejecute el comando siguiente en el servidor habilitado para Azure Arc:

```powershell
Install-Module -Name Az.ConnectedMachine
```

Cuando finaliza la instalación, se ve el mensaje siguiente:

`The installed extension ``Az.ConnectedMachine`` is experimental and not covered by customer support. Please use with discretion.`

## <a name="install-the-agent-and-connect-to-azure"></a>Instalación del agente y conexión a Azure

1. Abra una consola de PowerShell con privilegios elevados.

2. Inicie sesión en Azure para ejecutar el comando `Connect-AzAccount`.

3. Para instalar el Agente de Connected Machine, use `Connect-AzConnectedMachine` con los parámetros `-Name`, `-ResourceGroupName` y `-Location`. Use el parámetro `-SubscriptionId` para invalidar la suscripción predeterminada como resultado del contexto de Azure creado después del inicio de sesión. Ejecute uno de los siguientes comandos:

    * Para instalar el Agente de Connected Machine en el equipo de destino que se puede comunicar directamente con Azure, ejecute:

        ```azurepowershell
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Name myMachineName -Location <region>
        ```
    
    * Para instalar el Agente de Connected Machine en el equipo de destino que se comunica a través de un servidor proxy, ejecute:
        
        ```azurepowershell
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Name myMachineName -Location <region> -Proxy http://<proxyURL>:<proxyport>
        ```

Si el agente no se inicia una vez completada la instalación, compruebe los registros para obtener información detallada del error. En Windows, compruebe este archivo: *%ProgramData%\AzureConnectedMachineAgent\Log\himds.log*. En Linux, compruebe este archivo: */var/opt/azcmagent/log/himds.log*.

## <a name="install-and-connect-by-using-powershell-remoting"></a>Instalación y conexión mediante la comunicación remota de PowerShell

Aquí se explica cómo configurar uno o más servidores Windows con servidores habilitados para Azure Arc. Es necesario habilitar la comunicación remota de PowerShell en la máquina remota. Para ello, use el cmdlet `Enable-PSRemoting`.

1. Abra una consola de PowerShell como administrador.

2. Inicie sesión en Azure para ejecutar el comando `Connect-AzAccount`.

3. Para instalar el Agente de Connected Machine, use `Connect-AzConnectedMachine` con los parámetros `-ResourceGroupName` y `-Location`. Los nombres de los recursos de Azure usarán automáticamente el nombre de host de cada servidor. Use el parámetro `-SubscriptionId` para invalidar la suscripción predeterminada como resultado del contexto de Azure creado después del inicio de sesión.

    * Para instalar el Agente de Connected Machine en la máquina de destino que se puede comunicar directamente con Azure, ejecute el comando siguiente:
    
        ```azurepowershell
        $sessions = New-PSSession -ComputerName myMachineName
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Location <region> -PSSession $sessions
        ```
    
    * Para instalar el agente de Connected Machine en varias máquinas remotas al mismo tiempo, agregue una lista de nombres de máquinas remotas separados por una coma.

        ```azurepowershell
        $sessions = New-PSSession -ComputerName myMachineName1, myMachineName2, myMachineName3
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Location <region> -PSSession $sessions
        ```

    En el ejemplo siguiente se muestra el resultado del comando con una sola máquina como destino:
    
    ```azurepowershell
    time="2020-08-07T13:13:25-07:00" level=info msg="Onboarding Machine. It usually takes a few minutes to complete. Sometimes it may take longer depending on network and server load status."
    time="2020-08-07T13:13:25-07:00" level=info msg="Check network connectivity to all endpoints..."
    time="2020-08-07T13:13:29-07:00" level=info msg="All endpoints are available... continue onboarding"
    time="2020-08-07T13:13:50-07:00" level=info msg="Successfully Onboarded Resource to Azure" VM Id=f65bffc7-4734-483e-b3ca-3164bfa42941
    
    Name           Location OSName   Status     ProvisioningState
    ----           -------- ------   ------     -----------------
    myMachineName  eastus   windows  Connected  Succeeded
    ```

## <a name="verify-the-connection-with-azure-arc"></a>Comprobación de la conexión con Azure Arc

Después de instalar y configurar el agente para que se registre en los servidores habilitados para Azure Arc, vaya a Azure Portal a fin de comprobar que el servidor se ha conectado correctamente. Vea la máquina en [Azure Portal](https://portal.azure.com).

![Captura de pantalla del panel Servidores que muestra una conexión de servidor correcta.](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>Pasos siguientes

* Si es necesario, vea la guía [Solución de problemas de conexión del agente de Connected Machine](troubleshoot-agent-onboard.md).

* Obtenga información sobre cómo administrar la máquina mediante [Azure Policy](../../governance/policy/overview.md). Para usar la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de máquina virtual, compruebe que la máquina está notificando al área de trabajo de Log Analytics esperada y habilite la supervisión con [Azure Monitor para VM](../../azure-monitor/vm/vminsights-enable-policy.md).

* Más información sobre el [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md). Se necesita el agente de Log Analytics para Windows y Linux si se quieren recopilar datos de supervisión del sistema operativo y las cargas de trabajo o administrarlos con runbooks de Azure Automation o características como Update Management. Este agente también es necesario para usar otros servicios de Azure, como [Azure Security Center](../../security-center/security-center-introduction.md).
