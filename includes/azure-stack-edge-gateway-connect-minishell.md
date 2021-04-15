---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/30/2021
ms.author: alkohli
ms.openlocfilehash: 89e648099a5ac6d905f475319cc108dd0d05a6e9
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106081115"
---
Dependiendo del sistema operativo del cliente, los procedimientos para conectarse de forma remota al dispositivo son diferentes.

### <a name="remotely-connect-from-a-windows-client"></a>Conexión de forma remota desde un cliente de Windows


#### <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

- El cliente de Windows ejecuta Windows PowerShell 5.0 o una versión posterior.
- El cliente de Windows tiene la cadena de firma (certificado raíz) que corresponde al certificado de nodo instalado en el dispositivo. Para instrucciones detalladas, consulte [Instalación de un certificado en el cliente de Windows](../articles/databox-online/azure-stack-edge-gpu-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).
- El archivo `hosts` que se encuentra en `C:\Windows\System32\drivers\etc` en el cliente de Windows tiene una entrada que corresponde al certificado de nodo en el siguiente formato:

    `<Device IP>    <Node serial number>.<DNS domain of the device>`

    Este es un ejemplo del archivo `hosts`:
 
    `10.100.10.10    1HXQG13.wdshcsso.com`
  

#### <a name="detailed-steps"></a>Pasos detallados

Siga estos pasos para conectarse de forma remota desde un cliente de Windows.

1. Ejecute una sesión de Windows PowerShell como administrador.
2. Asegúrese de que el servicio Administración remota de Windows se ejecuta en el cliente. En el símbolo del sistema, escriba:

    `winrm quickconfig`

    Para más información, vea [Instalación y configuración de Administración remota de Windows](/windows/win32/winrm/installation-and-configuration-for-windows-remote-management#quick-default-configuration).

3. Asigne una variable a la dirección IP del dispositivo.

    $ip = "<device_ip>"

    Reemplace `<device_ip>` por la dirección IP de su dispositivo.

4. Para agregar la dirección IP del dispositivo a la lista de hosts de confianza del cliente, escriba el siguiente comando:

    `Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force`

5. Inicie una sesión de Windows PowerShell en el dispositivo:

    `Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell -UseSSL`

    Si ve un error que tiene que ver con la relación de confianza, compruebe que la cadena de firma del certificado de nodo cargado en el dispositivo también esté instalada en el cliente que tiene acceso al dispositivo.

    Si no usa los certificados (se recomienda usarlos), puede omitir esta comprobación mediante las opciones de sesión: `-SkipCACheck -SkipCNCheck -SkipRevocationCheck`.

    ```powershell
    $sessOptions = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck 
    Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell -UseSSL -SessionOption $sessOptions    
    ```

    > [!NOTE] 
    > Cuando se usa la opción `-UseSSL`, la comunicación remota se realiza a través de PowerShell a través de *https*. Se recomienda usar siempre *https* para conectarse de forma remota a través de PowerShell. 

6. Proporcione la contraseña cuando se le solicite. Use la misma contraseña que se emplea para iniciar sesión en la interfaz de usuario web local. La contraseña de la interfaz de usuario web local predeterminada es *Password1*. Cuando se conecte correctamente al dispositivo mediante PowerShell remoto, verá la siguiente salida de ejemplo:  

    ```
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    
    PS C:\WINDOWS\system32> winrm quickconfig
    WinRM service is already running on this machine.
    PS C:\WINDOWS\system32> $ip = "10.100.10.10"
    PS C:\WINDOWS\system32> Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force
    PS C:\WINDOWS\system32> Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell -UseSSL

    WARNING: The Windows PowerShell interface of your device is intended to be used only for the initial network configuration. Please engage Microsoft Support if you need to access this interface to troubleshoot any potential issues you may be experiencing. Changes made through this interface without involving Microsoft Support could result in an unsupported configuration.
    [10.100.10.10]: PS>
    ```

> [!IMPORTANT]
> En la versión actual, solo puede conectarse a la interfaz de PowerShell del dispositivo mediante un cliente de Windows. La opción `-UseSSL` no funciona con los clientes de Linux.

<!--### Remotely connect from a Linux client-->

<!--On the Linux client that you'll use to connect:

- [Install the latest PowerShell Core for Linux](/powershell/scripting/install/installing-powershell-core-on-linux) from GitHub to get the SSH remoting feature. 
- [Install only the `gss-ntlmssp` package from the NTLM module](https://github.com/Microsoft/omi/blob/master/Unix/doc/setup-ntlm-omi.md). For Ubuntu clients, use the following command:
    - `sudo apt-get install gss-ntlmssp`

For more information, go to [PowerShell remoting over SSH](/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core).

Follow these steps to remotely connect from an NFS client.

1. To open PowerShell session, type:

    `pwsh`
 
2. For connecting using the remote client, type:

    `Enter-PSSession -ComputerName $ip -Authentication Negotiate -ConfigurationName Minishell -Credential ~\EdgeUser`

    When prompted, provide the password used to sign into your device.
 
> [!NOTE]
> This procedure does not work on Mac OS.-->