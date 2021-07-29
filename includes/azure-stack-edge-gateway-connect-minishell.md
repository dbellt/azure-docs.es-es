---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/15/2021
ms.author: alkohli
ms.openlocfilehash: 6f732f7c59f76c99d82d5c4f7b062532c60a5a78
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110165009"
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

    ```powershell
    winrm quickconfig
    ```

    Para más información, vea [Instalación y configuración de Administración remota de Windows](/windows/win32/winrm/installation-and-configuration-for-windows-remote-management#quick-default-configuration).

3. Asigne una variable a la cadena de conexión utilizada en el archivo `hosts`.

    ```powershell
    $Name = "<Node serial number>.<DNS domain of the device>"
    ``` 

    Reemplace `<Node serial number>` y `<DNS domain of the device>` por el número de serie del nodo y el dominio DNS del dispositivo. Puede obtener los valores del número de serie del nodo en la página **Certificates** (Certificados) y el dominio DNS en la página **Device** (Dispositivo) de la interfaz de usuario web local del dispositivo.

4. Para agregar esta cadena de conexión del dispositivo a la lista de hosts de confianza del cliente, escriba el siguiente comando:

    ```powershell
    Set-Item WSMan:\localhost\Client\TrustedHosts $Name -Concatenate -Force
    ```

5. Inicie una sesión de Windows PowerShell en el dispositivo:

    ```powershell
    Enter-PSSession -ComputerName $Name -Credential ~\EdgeUser -ConfigurationName Minishell -UseSSL
    ```

    Si ve un error que tiene que ver con la relación de confianza, compruebe que la cadena de firma del certificado de nodo cargado en el dispositivo también esté instalada en el cliente que tiene acceso al dispositivo.

6. Proporcione la contraseña cuando se le solicite. Use la misma contraseña que se emplea para iniciar sesión en la interfaz de usuario web local. La contraseña de la interfaz de usuario web local predeterminada es *Password1*. Cuando se conecte correctamente al dispositivo mediante PowerShell remoto, verá la siguiente salida de ejemplo:  

    ```
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    
    PS C:\WINDOWS\system32> winrm quickconfig
    WinRM service is already running on this machine.
    PS C:\WINDOWS\system32> $Name = "1HXQG13.wdshcsso.com"
    PS C:\WINDOWS\system32> Set-Item WSMan:\localhost\Client\TrustedHosts $Name -Concatenate -Force
    PS C:\WINDOWS\system32> Enter-PSSession -ComputerName $Name -Credential ~\EdgeUser -ConfigurationName Minishell -UseSSL

    WARNING: The Windows PowerShell interface of your device is intended to be used only for the initial network configuration. Please engage Microsoft Support if you need to access this interface to troubleshoot any potential issues you may be experiencing. Changes made through this interface without involving Microsoft Support could result in an unsupported configuration.
    [1HXQG13.wdshcsso.com]: PS>
    ```

Cuando se usa la opción `-UseSSL`, la comunicación remota se realiza a través de PowerShell a través de *https*. Se recomienda usar siempre *https* para conectarse de forma remota a través de PowerShell. En redes de confianza, la comunicación remota mediante PowerShell a través de HTTP es aceptable. En primer lugar, habilite PowerShell remoto a través de HTTP en la interfaz de usuario local. A continuación, puede conectarse a la interfaz de PowerShell del dispositivo mediante el procedimiento anterior sin la opción `-UseSSL`.

Si no usa los certificados (se recomienda usarlos), puede omitir la comprobación de la validación de certificados mediante las opciones de sesión: `-SkipCACheck -SkipCNCheck -SkipRevocationCheck`.

```powershell
$sessOptions = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck 
Enter-PSSession -ComputerName $Name -Credential ~\EdgeUser -ConfigurationName Minishell -UseSSL -SessionOption $sessOptions    
```
Este es un ejemplo de salida al omitir la comprobación de certificados:

```powershell
PS C:\WINDOWS\system32> $Name = "1HXQG13.wdshcsso.com"
PS C:\WINDOWS\system32> $sessOptions = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
PS C:\WINDOWS\system32> $sessOptions

MaximumConnectionRedirectionCount : 5
NoCompression                     : False
NoMachineProfile                  : False
ProxyAccessType                   : None
ProxyAuthentication               : Negotiate
ProxyCredential                   :
SkipCACheck                       : True
SkipCNCheck                       : True
SkipRevocationCheck               : True
OperationTimeout                  : 00:03:00
NoEncryption                      : False
UseUTF16                          : False
IncludePortInSPN                  : False
OutputBufferingMode               : None
MaxConnectionRetryCount           : 0
Culture                           :
UICulture                         :
MaximumReceivedDataSizePerCommand :
MaximumReceivedObjectSize         :
ApplicationArguments              :
OpenTimeout                       : 00:03:00
CancelTimeout                     : 00:01:00
IdleTimeout                       : -00:00:00.0010000

PS C:\WINDOWS\system32> Enter-PSSession -ComputerName $Name -Credential ~\EdgeUser -ConfigurationName Minishell -UseSSL -SessionOption $sessOptions
WARNING: The Windows PowerShell interface of your device is intended to be used only for the initial network configuration. Please
engage Microsoft Support if you need to access this interface to troubleshoot any potential issues you may be experiencing.
Changes made through this interface without involving Microsoft Support could result in an unsupported configuration.
[1HXQG13.wdshcsso.com]: PS>
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