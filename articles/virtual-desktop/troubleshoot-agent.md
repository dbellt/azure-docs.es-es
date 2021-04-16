---
title: 'Solución de problemas del agente de Windows Virtual Desktop: Azure'
description: Obtenga información sobre cómo resolver problemas comunes de agente y conectividad.
author: Sefriend
ms.topic: troubleshooting
ms.date: 12/16/2020
ms.author: sefriend
manager: clarkn
ms.openlocfilehash: 2f321413a275676d0abb1a075ba958885ffcd821
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106505032"
---
# <a name="troubleshoot-common-windows-virtual-desktop-agent-issues"></a>Solución de problemas comunes del agente de Windows Virtual Desktop

El agente de Windows Virtual Desktop puede provocar problemas de conexión debido a varios factores:
   - Un error en el agente que hace que detenga el servicio.
   - Problemas con las actualizaciones.
   - Problemas durante la instalación del agente, que interrumpe la conexión con el host de sesión.

Este artículo le guiará a través de las soluciones para estos escenarios comunes y de cómo solucionar los problemas de conexión.

>[!NOTE]
>Para solucionar problemas relacionados con la conectividad de sesión y el agente de Windows Virtual Desktop, se recomienda que revise los registros de eventos en **Visor de eventos** > **Registros de Windows** > **Aplicación**. Busque eventos que tengan uno de los orígenes siguientes para identificar el problema:
>
>- WVD-Agent
>- WVD-Agent-Updater
>- RDAgentBootLoader
>- MsiInstaller

## <a name="error-the-rdagentbootloader-andor-remote-desktop-agent-loader-has-stopped-running"></a>Error: El cargador del agente de Escritorio remoto o RDAgentBootLoader ha dejado de ejecutarse

Si ve alguno de los siguientes problemas, significa que el cargador de arranque, que carga el agente, no pudo instalar el agente correctamente y el servicio del agente no se está ejecutando:
- **RDAgentBootLoader** se ha detenido o no se está ejecutando.
- No hay ningún estado para el **cargador del agente de Escritorio remoto**.

Para resolver este problema, inicie el cargador de arranque RDAgent:

1. En la ventana Servicios, haga clic con el botón derecho en **Cargador del agente de Escritorio remoto**.
2. Seleccione **Inicio**. Si esta opción está atenuada, significa que no tiene permisos de administrador y deberá obtenerlos para iniciar el servicio.
3. Espere 10 segundos y haga clic con el botón derecho en **Cargador del agente de Escritorio remoto**.
4. Seleccione **Refresh** (Actualizar).
5. Si el servicio se detiene después de iniciarlo y actualizarlo, es posible que se produzca un error de registro. Para más información, consulte [INVALID_REGISTRATION_TOKEN](#error-invalid_registration_token).

## <a name="error-invalid_registration_token"></a>Error: INVALID_REGISTRATION_TOKEN

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277, que indica **INVALID_REGISTRATION_TOKEN** en la descripción, significa que el token de registro no se reconoce como válido.

Para resolver este problema, cree un token de registro válido:

1. Para crear un nuevo token de registro, siga los pasos descritos en la sección [Generación de una nueva clave de registro para la máquina virtual](#step-3-generate-a-new-registration-key-for-the-vm).
2. Abra el Editor del Registro. 
3. Vaya a **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **Microsoft** > **RDInfraAgent**.
4. Seleccione **IsRegistered**. 
5. En el cuadro de entrada **Datos del valor:** , escriba **0** y seleccione **Aceptar**. 
6. Seleccione **RegistrationToken**. 
7. En el cuadro de entrada **Datos del valor:** , pegue el token de registro del paso 1. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de IsRegistered 0](media/isregistered-token.png)

8. Abra un símbolo del sistema como administrador.
9. Escriba **net stop RDAgentBootLoader**. 
10. Escriba **net start RDAgentBootLoader**. 
11. Abra el Editor del Registro.
12. Vaya a **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **Microsoft** > **RDInfraAgent**.
13. Compruebe que **IsRegistered** está establecido en 1 y no hay nada en la columna de datos para **RegistrationToken**. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de IsRegistered 1](media/isregistered-registry.png)

## <a name="error-agent-cannot-connect-to-broker-with-invalid_form"></a>Error: El agente no se puede conectar al agente con INVALID_FORM.

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277, que indica "INVALID_FORM" en la descripción, significa que se produjo un error con la comunicación entre los agentes. El agente no se puede conectar al agente o tener acceso a una dirección URL determinada debido a determinados valores de configuración de DNS o firewall.

Para resolver este problema, compruebe que puede acceder a BrokerURI y BrokerURIGlobal:
1. Abra el Editor del Registro. 
2. Vaya a **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **Microsoft** > **RDInfraAgent**. 
3. Tome nota de los valores de **BrokerURI** y **BrokerURIGlobal**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla del URI del agente y URI del agente global](media/broker-uri.png)

 
4. Abra un explorador y vaya a *\<BrokerURI\>api/health*. 
   - Asegúrese de usar el valor del paso 3 en **BrokerURI**. En el ejemplo de esta sección, sería <https://rdbroker-g-us-r0.wvd.microsoft.com/api/health>.
5. Abra otra pestaña en el explorador y vaya a *\<BrokerURIGlobal\>api/health*. 
   - Asegúrese de usar el valor del paso 3 en el vínculo de **BrokerURIGlobal**. En el ejemplo de esta sección, sería <https://rdbroker.wvd.microsoft.com/api/health>.
6. Si la red no está bloqueando la conexión del agente, ambas páginas se cargarán correctamente y mostrarán un mensaje que indica **"El agente de escritorio remoto es correcto"** , tal como se muestra en las siguientes capturas de pantallas. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de acceso al URI del agente cargado correctamente](media/broker-uri-web.png)

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de acceso al URI del agente global cargado correctamente](media/broker-global.png)
 

7. Si la red está bloqueando la conexión del agente, las páginas no se cargarán, tal como se muestra en la siguiente captura de pantalla. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de acceso al agente cargado incorrectamente](media/unsuccessful-broker-uri.png)

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de acceso al agente global cargado incorrectamente](media/unsuccessful-broker-global.png)

8. Si la red está bloqueando estas direcciones URL, tendrá que desbloquear las direcciones URL necesarias. Para más información, consulte [Lista de direcciones URL requeridas](safe-url-list.md).
9. Si esto no resuelve el problema, asegúrese de que no tiene ninguna directiva de grupo con cifrados que bloqueen la conexión entre los agentes. Windows Virtual Desktop usa los mismos cifrados TLS 1.2 que [Azure Front Door](../frontdoor/front-door-faq.yml#what-are-the-current-cipher-suites-supported-by-azure-front-door-). Para obtener más información, consulte [Seguridad de conexión](network-connectivity.md#connection-security).

## <a name="error-3703"></a>Error: 3703

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3703, que indica "Dirección URL de puerta de enlace de Escritorio remoto: no es accesible" en la descripción, significa que el agente no puede tener acceso a las direcciones URL de puerta de enlace. Para conectarse correctamente al host de sesión y permitir que el tráfico de red a estos puntos de conexión omita las restricciones, debe desbloquear las direcciones URL de la [lista de direcciones URL requeridas](safe-url-list.md). Además, asegúrese de que el firewall o la configuración de proxy no bloqueen estas direcciones URL. El desbloqueo de estas direcciones URL es necesario para usar Windows Virtual Desktop.

Para resolver este problema, compruebe que la configuración de firewall o DNS no está bloqueando estas direcciones URL:
1. [Uso de Azure Firewall para proteger las implementaciones de Windows Virtual Desktop](../firewall/protect-windows-virtual-desktop.md).
2. Defina la [configuración DNS de Azure Firewall](../firewall/dns-settings.md).

## <a name="error-3019"></a>Error: 3019

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3019, significa que el agente no puede tener acceso a las direcciones URL de transporte de socket web. Para conectarse correctamente al host de sesión y permitir que el tráfico de red omita las restricciones, debe desbloquear las direcciones URL de la [lista de direcciones URL requeridas](safe-url-list.md). Trabaje con el equipo de Redes de Azure para asegurarse de que la configuración de firewall, proxy y DNS no esté bloqueando estas direcciones URL. También puede comprobar los registros de seguimiento de red para identificar dónde se está bloqueando el servicio Windows Virtual Desktop. Si abre una solicitud de soporte técnico para este problema en particular, asegúrese de adjuntar los registros de seguimiento de red a la solicitud.

## <a name="error-installationhealthcheckfailedexception"></a>Error: InstallationHealthCheckFailedException

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277 que indica "InstallationHealthCheckFailedException" en la descripción, significa que el cliente de escucha de la pila no funciona porque el servidor de Terminal Server ha activado la clave del Registro para el cliente de escucha de la pila.

Para solucionar este problema:
1. Compruebe si [el cliente de escucha de la pila está funcionando](#error-stack-listener-isnt-working-on-windows-10-2004-vm).
2. Si no funciona, [desinstale y vuelva a instalar manualmente el componente de pila](#error-vms-are-stuck-in-unavailable-or-upgrading-state).

## <a name="error-endpoint_not_found"></a>Error: ENDPOINT_NOT_FOUND

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277 que dice "ENDPOINT_NOT_FOUND" en la descripción, significa que el agente no pudo encontrar un punto de conexión con el que establecer una conexión. Este problema de conexión puede deberse a una de las siguientes causas:

- No hay máquinas virtuales en el grupo de hosts.
- Las máquinas virtuales del grupo de hosts no están activas.
- Todas las máquinas virtuales del grupo de hosts han superado el límite de sesión máximo
- Ninguna de las máquinas virtuales del grupo de hosts tienen el servicio del agente en ejecución.

Para solucionar este problema:

1. Asegúrese de que la máquina virtual esté encendida y no se haya quitado del grupo de hosts.
2. Asegúrese de que la máquina virtual no haya superado el límite máximo de sesiones.
3. Asegúrese de que el [servicio del agente se esté ejecutando](#error-the-rdagentbootloader-andor-remote-desktop-agent-loader-has-stopped-running) y de que el [cliente de escucha de la pila esté funcionando](#error-stack-listener-isnt-working-on-windows-10-2004-vm).
4. Asegúrese [de que el agente pueda conectarse al agente](#error-agent-cannot-connect-to-broker-with-invalid_form).
5. Asegúrese [de que la máquina virtual tenga un token de registro válido](#error-invalid_registration_token).
6. Asegúrese [de que el token de registro de la máquina virtual no haya expirado](faq.md#how-often-should-i-turn-my-vms-on-to-prevent-registration-issues). 

## <a name="error-installmsiexception"></a>Error: InstallMsiException

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277, que indica **InstallMsiException** en la descripción, significa que el instalador ya se está ejecutando para otra aplicación mientras intenta instalar el agente o que una directiva está bloqueando la ejecución del programa msiexec.exe.

Para resolver este problema, deshabilite la siguiente directiva:
   - Desactive Windows Installer.  
      - Ruta de acceso de la categoría: Configuración del equipo\Plantillas administrativas\Componentes de Windows\Windows Installer
   
>[!NOTE]
>Esta no es una lista completa de directivas, solo de las que tenemos conocimiento.

Para deshabilitar una directiva:
1. Abra un símbolo del sistema como administrador.
2. Escriba y ejecute **rsop.msc**.
3. En la ventana **Conjunto resultante de directivas** que aparece, vaya a la ruta de acceso de la categoría.
4. Seleccione la directiva.
5. Seleccione **Deshabilitado**.
6. Seleccione **Aplicar**.   

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la directiva de Windows Installer en Conjunto resultante de directivas](media/gpo-policy.png)

## <a name="error-win32exception"></a>Error: Win32Exception

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277, que indica **InstallMsiException** en la descripción, significa que una directiva está bloqueando el inicio de cmd.exe. El bloqueo de este programa impide ejecutar la ventana de la consola, que es lo que necesita usar para reiniciar el servicio cada vez que se actualiza el agente.

Para resolver este problema, deshabilite la siguiente directiva:
   - Impedir el acceso al símbolo del sistema   
      - Ruta de acceso de la categoría: Configuración de usuario\Plantillas administrativas\Sistema
    
>[!NOTE]
>Esta no es una lista completa de directivas, solo de las que tenemos conocimiento.

Para deshabilitar una directiva:
1. Abra un símbolo del sistema como administrador.
2. Escriba y ejecute **rsop.msc**.
3. En la ventana **Conjunto resultante de directivas** que aparece, vaya a la ruta de acceso de la categoría.
4. Seleccione la directiva.
5. Seleccione **Deshabilitado**.
6. Seleccione **Aplicar**.   

## <a name="error-stack-listener-isnt-working-on-windows-10-2004-vm"></a>Error: El cliente de escucha de la pila no funciona en la máquina virtual 10 2004 de Windows

Ejecute **qwinsta** en el símbolo del sistema y anote el número de versión que aparece junto a **rdp-sxs**. Si no ve que los componentes **rdp-tcp** y **rdp-sxs** indican **Escuchar** junto a ellos o no aparecen en absoluto después de ejecutar **qwinsta**, significa que hay un problema en la pila. Las actualizaciones de la pila se instalan junto con las actualizaciones del agente y, cuando esta instalación no funciona, el cliente de escucha de Windows Virtual Desktop tampoco funciona.

Para solucionar este problema:
1. Abra el Editor del Registro.
2. Vaya a **HKEY_LOCAL_MACHINE** > **SYSTEM** > **CurrentControlSet** > **Control** > **Terminal Server** > **WinStations**.
3. En **WinStations** puede ver varias carpetas para distintas versiones de pila, seleccione la carpeta que coincida con la información de versión que vio al ejecutar **qwinsta** en el símbolo del sistema.
4. Busque **fReverseConnectMode** y asegúrese de que su valor de datos es **1**. Asegúrese también de que **fEnableWinStation** está establecido en **1**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de fReverseConnectMode](media/fenable-2.png)

5. Si **fReverseConnectMode** no está establecido en **1**, seleccione **fReverseConnectMode** y escriba **1** en el campo de valor. 
6. Si **fEnableWinStation** no está establecido en **1**, seleccione **fEnableWinStation** y escriba **1** en su campo valor.
7. Reinicie la máquina virtual. 

>[!NOTE]
>Para cambiar el modo de **fReverseConnectMode** o **fEnableWinStation** para varias máquinas virtuales a la vez, puede realizar una de las dos acciones siguientes:
>
>- Exporte la clave del Registro del equipo en el que ya está trabajando e impórtela en el resto de máquinas que necesitan este cambio.
>- Cree un objeto de directiva de grupo (GPO) que establezca el valor de clave del Registro para las máquinas que necesitan el cambio.

7. Vaya a **HKEY_LOCAL_MACHINE** > **SYSTEM** > **CurrentControlSet** > **Control** > **Terminal Server** > **ClusterSettings**.
8. En **ClusterSettings**, busque **SessionDirectoryListener** y asegúrese de que su valor de datos es **rdp-sxs...** .
9. Si **SessionDirectoryListener** no está establecido en **rdp-sxs...** , deberá seguir los pasos descritos en la sección [Desinstalación del agente y el cargador de arranque](#step-1-uninstall-all-agent-boot-loader-and-stack-component-programs) para desinstalar primero el agente, el cargador de arranque y los componentes de la pila y, a continuación, [volver a instalar el agente y el cargador de arranque](#step-4-reinstall-the-agent-and-boot-loader). De esta forma, se volverá a instalar la pila en paralelo.

## <a name="error-heartbeat-issue-where-users-keep-getting-disconnected-from-session-hosts"></a>Error: Problema de latido en el que los usuarios siguen desconectándose de los hosts de sesión

Si el servidor no selecciona un latido del servicio Windows Virtual Desktop, deberá cambiar el umbral de latido. Esto mitigará temporalmente los síntomas del problema, pero no solucionará el problema de red subyacente. Si se aplican uno o varios de los siguientes escenarios, siga estas instrucciones:

- Está recibiendo un error **CheckSessionHostDomainIsReachableAsync**.
- Está recibiendo un error **ConnectionBrokenMissedHeartbeatThresholdExceeded**.
- Está recibiendo un error **ConnectionEstablished: UnexpectedNetworkDisconnect**.
- Los clientes de usuario siguen sin conexión.
- Los usuarios siguen desconectándose de los hosts de sesión.

Para cambiar el umbral de latidos:
1. Abra el símbolo del sistema como administrador.
2. Escriba el comando **qwinsta** y ejecútelo.
3. Deben mostrarse dos componentes de la pila: **rdp-tcp** y **rdp-sxs**. 
   - Dependiendo de la versión del sistema operativo que use, **rdp-sxs** puede ir seguido del número de compilación. Si es así, asegúrese de anotar este número para más adelante.
4. Abra el Editor del Registro.
5. Vaya a **HKEY_LOCAL_MACHINE** > **SYSTEM** > **CurrentControlSet** > **Control** > **Terminal Server** > **WinStations**.
6. En **WinStations** puede ver varias carpetas para distintas versiones de pila. Seleccione la carpeta que coincida con el número de versión del paso 3.
7. Cree un nuevo valor DWORD del registro. Para ello, haga clic con el botón derecho en el editor del registro y, a continuación, seleccione **Nuevo** > **Valor DWORD (32 bits)** . Al crear el valor DWORD, escriba los siguientes valores:
   - HeartbeatInterval: 10 000
   - HeartbeatWarnCount: 30 
   - HeartbeatDropCount: 60 
8. Reinicie la máquina virtual.

>[!NOTE]
>Si al cambiar el umbral del latido no se resuelve el problema, es posible que tenga un problema de red subyacente sobre el que tendrá que ponerse en contacto con el equipo de Redes de Azure.

## <a name="error-downloadmsiexception"></a>Error: DownloadMsiException

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3277, que indica **DownloadMsiException** en la descripción, significa que no hay suficiente espacio en el disco para RDAgent.

Para resolver este problema, haga lo siguiente para crear espacio en el disco:
   - Elimine archivos que ya no están en el usuario.
   - Aumente la capacidad de almacenamiento de la máquina virtual.

## <a name="error-agent-fails-to-update-with-missingmethodexception"></a>Error: El agente no se puede actualizar con MissingMethodException.

Vaya a **Visor de eventos** > **Registros de Windows** > **Aplicación**. Si ve un evento con el identificador 3389 que indica "MissingMethodException: método no encontrado" en la descripción, significa que el agente de Windows Virtual Desktop no se ha actualizado correctamente y se ha revertido a una versión anterior. Esto puede deberse a que el número de versión de .NET Framework instalado actualmente en las máquinas virtuales es inferior a 4.7.2. Para resolver este problema, debe actualizar .NET a la versión 4.7.2 o versiones posteriores siguiendo las instrucciones de instalación de la [documentación de .NET Framework](https://support.microsoft.com/topic/microsoft-net-framework-4-7-2-offline-installer-for-windows-05a72734-2127-a15d-50cf-daf56d5faec2).


## <a name="error-vms-are-stuck-in-unavailable-or-upgrading-state"></a>Error: Las máquinas virtuales están atascadas en estado No disponible o Actualizando

Abra una ventana de PowerShell como administrador y ejecute el siguiente cmdlet:

```powershell
Get-AzWvdSessionHost -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> | Select-Object *
```

Si el estado que se muestra para el host de sesión o los hosts en el grupo de hosts siempre indica "No disponible" o "Actualizando", significa que el agente o la pila no se han instalado correctamente.

Para resolver este problema, reinstale la pila en paralelo:
1. Abra un símbolo del sistema como administrador.
2. Escriba **net stop RDAgentBootLoader**. 
3. Vaya a **Panel de control** > **Programas** > **Programas y características**.
4. Desinstale la versión más reciente de la **pila de red de Servicios de Escritorio remoto SxS** o la versión que aparece en **HKEY_LOCAL_MACHINE** > **SYSTEM** > **CurrentControlSet** > **Control** > **Terminal Server** > **WinStations** en **ReverseConnectListener**.
5. Abra una ventana de consola como administrador y vaya a **Archivos de programa** > **Microsoft RDInfra**.
6. Seleccione el componente **SxSStack** o ejecute el comando **msiexec /i SxsStack-<version>.msi** para instalar el MSI.
8. Reinicie la máquina virtual.
9. Vuelva al símbolo del sistema y ejecute el comando **qwinsta**.
10. Compruebe que el componente de pila instalado en el paso 6 indica **Escuchar** junto a él.
   - En ese caso, escriba **net start RDAgentBootLoader** en el símbolo del sistema y reinicie la máquina virtual.
   - Si no es así, tendrá que [volver a registrar la máquina virtual y reinstalar el componente del agente](#your-issue-isnt-listed-here-or-wasnt-resolved).

## <a name="error-connection-not-found-rdagent-does-not-have-an-active-connection-to-the-broker"></a>Error: No se encuentra la conexión: RDAgent no tiene una conexión activa con el agente

Las máquinas virtuales pueden estar en su límite de conexiones, por lo que no pueden aceptar nuevas conexiones.

Para solucionar este problema:
   - Disminuya el límite máximo de sesión. Esto garantiza que los recursos se distribuyan de forma más equitativa entre los hosts de sesión y evita el agotamiento de los recursos.
   - Aumente la capacidad de los recursos de las máquinas virtuales.

## <a name="error-operating-a-pro-vm-or-other-unsupported-os"></a>Error: Funcionamiento de una VM Pro u otro sistema operativo no admitido

La pila en paralelo solo es compatible con las SKU de Windows Enterprise o Windows Server, lo que significa que los sistemas operativos como la VM Pro no la admiten. Si no tiene una SKU de Enterprise o Server, la pila se instalará en la máquina virtual, pero no se activará, por lo que no la verá aparecer al ejecutar **qwinsta** en la línea de comandos.

Para resolver este problema, cree una máquina virtual que sea Windows Enterprise o Windows Server.
1. Vaya a [Detalles de máquina virtual](create-host-pools-azure-marketplace.md#virtual-machine-details) y siga los pasos del 1 al 12 para configurar una de las siguientes imágenes recomendadas:
   - Windows 10 Enterprise multisesión, versión 1909
   - Windows 10 Enterprise multisesión, versión 1909 + Aplicaciones de Microsoft 365
   - Windows Server 2019 Datacenter
   - Windows 10 Enterprise multisesión, versión 2004
   - Windows 10 Enterprise multisesión, versión 2004 + Aplicaciones de Microsoft 365
2. Seleccione **Revisar y crear**.

## <a name="error-name_already_registered"></a>Error: NAME_ALREADY_REGISTERED

El nombre de la VM ya se ha registrado y probablemente es un duplicado.

Para solucionar este problema:
1. Siga los pasos de la sección [Eliminación del host de sesión del grupo de hosts](#step-2-remove-the-session-host-from-the-host-pool).
2. [Cree otra máquina virtual](expand-existing-host-pool.md#add-virtual-machines-with-the-azure-portal). Asegúrese de elegir un nombre único para esta máquina virtual.
3. Vaya a [Azure Portal](https://portal.azure.com) y abra la página **Información general** del grupo de hosts en el que se encontraba la máquina virtual. 
4. Abra la pestaña **Hosts de sesión** y asegúrese de que todos los hosts de sesión se encuentran en ese grupo de hosts.
5. Espere de 5 a 10 minutos para que el estado del host de sesión indique **Disponible**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla del host de sesión disponible](media/hostpool-portal.png)

## <a name="your-issue-isnt-listed-here-or-wasnt-resolved"></a>El problema no aparece aquí o no se resolvió

Si no encuentra el problema en este artículo o las instrucciones no le ayudaron, se recomienda desinstalar, reinstalar y volver a registrar el agente de Windows Virtual Desktop. En las instrucciones de esta sección se muestra cómo volver a registrar la máquina virtual en el servicio de Windows Virtual Desktop mediante la desinstalación de todos los componentes del agente, el cargador de arranque y la pila, la eliminación del host de sesión del grupo de hosts, la generación de una nueva clave de registro para la máquina virtual y la reinstalación del agente y el cargador de arranque. Si se aplican uno o varios de los siguientes escenarios, siga estas instrucciones:
- La máquina virtual está bloqueada en **Actualizando** o **No disponible**.
- El cliente de escucha de la pila no funciona y se está ejecutando en Windows 10 1809, 1903 o 1909.
- Recibe un error **EXPIRED_REGISTRATION_TOKEN**.
- No ve las máquinas virtuales en la lista de hosts de sesión.
- No ve **Cargador del agente de Escritorio remoto** en la ventana Servicios.
- No ve el componente **RdAgentBootLoader** en el administrador de tareas.
- Está recibiendo un error que indica que el **agente de conexión no pudo validar la configuración** en las máquinas virtuales de imágenes personalizadas.
- Las instrucciones de este artículo no resolvieron el problema.

### <a name="step-1-uninstall-all-agent-boot-loader-and-stack-component-programs"></a>Paso 1: Desinstalación de todos los programas del agente, el cargador de arranque y los componentes de la pila

Antes de volver a instalar el agente, el cargador de arranque y la pila, debe desinstalar todos los programas de componentes existentes de la máquina virtual. Para desinstalar todos los programas del agente, el cargador de arranque y los componentes de la pila:
1. Inicie sesión en la máquina virtual como administrador. 
2. Vaya a **Panel de control** > **Programas** > **Programas y características**.
3. Quite los programas siguientes:
   - Cargador de arranque del agente de Escritorio remoto
   - Agente de infraestructura de Servicios de Escritorio remoto
   - Agente de Geneva de infraestructura de Servicios de Escritorio remoto
   - Pila de red de Servicios de Escritorio remoto SxS
   
>[!NOTE]
>Es posible que vea varias instancias de estos programas. Asegúrese de quitar todos.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la desinstalación de programas](media/uninstall-program.png)

### <a name="step-2-remove-the-session-host-from-the-host-pool"></a>Paso 2: Eliminación del host de sesión del grupo de hosts

Cuando se quita el host de sesión del grupo de hosts, el host de sesión ya no está registrado en ese grupo de hosts. Esto actúa como un restablecimiento para el registro del host de sesión. Para quietar el host de sesión del grupo de hosts:
1. En [Azure Portal](https://portal.azure.com), vaya a la página **Información general** del grupo de hosts en el que se encuentra la máquina virtual. 
2. Vaya a la pestaña **Hosts de sesión** para ver la lista de todos los hosts de sesión de ese grupo de hosts.
3. Examine la lista de hosts de sesión y seleccione la máquina virtual que quiere quitar.
4. Seleccione **Quitar**.  

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la eliminación de la máquina virtual del grupo de hosts](media/remove-sh.png)

### <a name="step-3-generate-a-new-registration-key-for-the-vm"></a>Paso 3: Generación de una nueva clave de registro para la máquina virtual

Debe generar una nueva clave de registro que se usa para volver a registrar la máquina virtual en el grupo de hosts y en el servicio. Para generar una nueva clave de registro para la máquina virtual:
1. Abra [Azure Portal](https://portal.azure.com) y vaya a la página **Información general** del grupo de hosts de la máquina virtual que quiere editar.
2. Seleccione **Clave de registro**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la clave de registro en el portal](media/reg-key.png)

3. Abra la pestaña **Clave de registro** y seleccione **Generar nueva clave**.
4. Escriba la fecha de expiración y, a continuación, seleccione **Aceptar**.  

>[!NOTE]
>La fecha de expiración no puede ser inferior a una hora ni más de 27 días a partir de su fecha y hora de generación. Le recomendamos encarecidamente que establezca la fecha de expiración en el máximo de 27 días.

5. Copie la clave recién generada en el portapapeles. La necesitará más adelante.

### <a name="step-4-reinstall-the-agent-and-boot-loader"></a>Paso 4: Reinstalación del agente y el cargador de arranque

Al volver a instalar la versión más actualizada del agente y el cargador de arranque, la pila en paralelo y el agente de supervisión de Geneva también se instalan automáticamente. Para reinstalar el agente y el cargador de arranque:
1. Inicie sesión en la máquina virtual como administrador y use la versión correcta del instalador del agente para su implementación, en función de la versión de Windows en la que se ejecute la máquina virtual. Si tiene una máquina virtual de Windows 10, siga las instrucciones que se indican en [Registro de las máquinas virtuales](create-host-pools-powershell.md#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool) para descargar el **agente de Windows Virtual Desktop** y el **cargador de arranque del agente de Windows Virtual Desktop**. Si tiene una máquina virtual de Windows 7, siga los pasos 13 y 14 de [Registro de máquinas virtuales](deploy-windows-7-virtual-machine.md#configure-a-windows-7-virtual-machine) para descargar el **agente de Windows Virtual Desktop** y **Agent Manager de Windows Virtual Desktop**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la página de descarga del agente y el cargador de arranque](media/download-agent.png)

2. Haga clic con el botón derecho en los instaladores de agente y cargador de arranque que ha descargado.
3. Seleccione **Propiedades**.
4. Seleccione **Unblock** (Desbloquear).
5. Seleccione **Aceptar**.
6. Ejecute el instalador del agente.
7. Cuando el instalador le pida el token de registro, pegue la clave de registro del portapapeles. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla del token de registro pegado](media/pasted-agent-token.png)

8. Ejecute el instalador del cargador de arranque.
9. Reinicie la máquina virtual. 
10. Vaya a [Azure Portal](https://portal.azure.com) y abra la página **Información general** del grupo de hosts al que pertenece la máquina virtual.
11. Vaya a la pestaña **Hosts de sesión** para ver la lista de todos los hosts de sesión de ese grupo de hosts.
12. Ahora debería ver el host de sesión registrado en el grupo de hosts con el estado **Disponible**. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla del host de sesión disponible](media/hostpool-portal.png)

## <a name="next-steps"></a>Pasos siguientes

Si el problema continúa, cree un caso de soporte técnico e incluya información detallada sobre el problema que está experimentando y las acciones que ha llevado a cabo para intentar resolverlo. En la lista siguiente se incluyen otros recursos que puede usar para solucionar problemas en la implementación de Windows Virtual Desktop.

- Para obtener información general sobre cómo solucionar problemas de Windows Virtual Desktop y las pistas de escalación, consulte [Introducción, comentarios y soporte técnico para solucionar problemas](troubleshoot-set-up-overview.md).
- Para solucionar problemas durante la creación de un grupo de hosts en un entorno de Windows Virtual Desktop, consulte [Creación de entornos y grupos de hosts](troubleshoot-set-up-issues.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Windows Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration.md).
- Para solucionar problemas con conexiones de cliente de Windows Virtual Desktop, consulte [Conexiones de servicios de Windows Virtual Desktop](troubleshoot-service-connection.md).
- Para solucionar problemas con los clientes de Escritorio remoto, consulte [Solución de problemas del cliente de Escritorio remoto](troubleshoot-client.md).
- Para solucionar problemas al usar PowerShell con Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell.md).
- Para más información sobre el servicio, consulte [Entorno de Windows Virtual Desktop](environment-setup.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
- Si desea conocer más detalles sobre las acciones que permiten determinar los errores durante la implementación, consulte [Visualización de operaciones de implementación con el Portal de Azure](../azure-resource-manager/templates/deployment-history.md).
