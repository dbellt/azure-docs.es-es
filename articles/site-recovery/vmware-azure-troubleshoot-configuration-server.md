---
title: Solución de problemas con el servidor de configuración durante la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con Azure Site Recovery | Microsoft Docs
description: En este artículo se proporciona información para solucionar problemas de la implementación del servidor de configuración para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: b943013de867016478a6e126d244974a8505a7cd
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576577"
---
# <a name="troubleshoot-configuration-server-issues"></a>Solución de problemas del servidor de configuración

En este artículo se le ayudará a solucionar problemas al implementar y administrar el servidor de configuración de [Azure Site Recovery](site-recovery-overview.md). El servidor de configuración actúa como servidor de administración. Use el servidor de configuración para configurar la recuperación ante desastres de VM de VMware y servidores físicos locales en Azure con Site Recovery. En las siguientes secciones se habla de los errores más habituales que podría experimentar al agregar un nuevo servidor de configuración y al administrar un servidor de configuración.

## <a name="registration-failures"></a>Errores de registro

La máquina de origen se registra con el servidor de configuración al instalar el agente de movilidad. Puede depurar los errores durante este paso siguiendo estas directrices:

1. Abra el archivo C:\ProgramData\ASR\home\svsystems\var\configurator_register_host_static_info.log (la carpeta ProgramData podría ser una carpeta oculta. Si no ve la carpeta ProgramData, en el Explorador de archivos, en la pestaña **Ver**, en la sección **Mostrar/ocultar**, active la casilla **Elementos ocultos**). Los errores podrían deberse a varios problemas.

2. Busque la cadena **No se encuentra ninguna dirección IP válida**. Si se encuentra la cadena:
   1. Compruebe que el identificador de host solicitado es el mismo que el identificador de host de la máquina de origen.
   2. Compruebe que la máquina de origen tiene al menos una dirección IP asignada a la NIC física. Para que el registro del agente con el servidor de configuración se realice correctamente, la máquina de origen debe tener al menos una dirección IPv4 asignada a la NIC física.
   3. Ejecute uno de los siguientes comandos en la máquina de origen para obtener todas las direcciones IP de la máquina de origen:
      - En Windows: `> ipconfig /all`
      - En Linux: `# ifconfig -a`

3. Si no se encuentra la cadena **No se encuentra ninguna dirección IP válida** , busque la cadena **Motivo => NULL**. Se produce este error si la máquina de origen utiliza un host vacío para registrarse con el servidor de configuración. Si se encuentra la cadena:
    - Tras resolver los problemas, siga las directrices en [Registrar la máquina de origen con el servidor de configuración](vmware-azure-troubleshoot-configuration-server.md#register-source-machine-with-configuration-server) para volver a intentar el registro manualmente.

4. Si no se encuentra la cadena **Motivo => NULL**, en la máquina de origen, abra el archivo C:\ProgramData\ASRSetupLogs\UploadedLogs\ASRUnifiedAgentInstaller.log (la carpeta ProgramData podría ser una carpeta oculta. Si no ve la carpeta ProgramData, en el Explorador de archivos, en la pestaña **Ver**, en la sección **Mostrar/ocultar**, active la casilla **Elementos ocultos**). Los errores podrían deberse a varios problemas. 

5. Busque la cadena **post request: (7) - Couldn't connect to server**. Si se encuentra la cadena:
    1. Resuelva los problemas de red entre el servidor de configuración y la máquina de origen. Compruebe que el servidor de configuración es accesible desde la máquina de origen con herramientas de red como ping, traceroute o un explorador web. Asegúrese de que la máquina de origen pueda acceder al servidor de configuración a través del puerto 443.
    2. Compruebe si alguna regla de firewall de la máquina de origen bloquea la conexión entre el servidor de configuración y la máquina de origen. Trabaje con el administrador de red para desbloquear los problemas de conexión.
    3. Asegúrese de que las carpetas incluidas en [Exclusiones de carpetas de Site Recovery de programas antivirus](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) se excluyan del software antivirus.
    4. Cuando se resuelvan los problemas de red, vuelva a intentar el registro siguiendo las directrices en [Registrar la máquina de origen con el servidor de configuración](vmware-azure-troubleshoot-configuration-server.md#register-source-machine-with-configuration-server).

6. Si no se encuentra la cadena **post request: (7) - Couldn't connect to server**, en el mismo archivo de registro, busque la cadena **request: (60) - Peer certificate cannot be authenticated with given CA certificates**. Este error podría producirse porque el certificado de servidor de configuración ha expirado o la máquina de origen no admite TLS 1.0 o protocolos posteriores. También puede producirse si un firewall bloquea la comunicación TLS entre la máquina de origen y el servidor de configuración. Si se encuentra la cadena: 
    1. Para resolverlo, conéctese a la dirección IP del servidor de configuración mediante un explorador web en la máquina de origen. Use la URI https:\/\/<dirección IP del servidor de configuración\>:443/. Asegúrese de que la máquina de origen pueda acceder al servidor de configuración a través del puerto 443.
    2. Compruebe si es necesario agregar o quitar alguna regla de firewall de la máquina de origen para que la máquina de origen se comunique con el servidor de configuración. Dada la variedad de software de firewall que podría estar en uso, no podemos incluir todas las configuraciones de firewall necesarias. Trabaje con el administrador de red para desbloquear los problemas de conexión.
    3. Asegúrese de que las carpetas incluidas en [Exclusiones de carpetas de Site Recovery de programas antivirus](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) se excluyan del software antivirus.  
    4. Tras resolver los problemas, vuelva a intentar el registro siguiendo las directrices en [Registrar la máquina de origen con el servidor de configuración](vmware-azure-troubleshoot-configuration-server.md#register-source-machine-with-configuration-server).

7. En Linux, si el valor de la plataforma de <INSTALLATION_DIR\>/etc/drscout.conf está dañado, se producirá un error en el registro. Para identificar este problema, abra el archivo /var/log/ua_install.log. Busque la cadena **Aborting configuration as VM_PLATFORM value is either null or it is not VmWare/Azure**. La plataforma debe establecerse en **VmWare** o **Azure**. Si el archivo drscout.conf está dañado, recomendamos que [desinstale el agente de movilidad](vmware-physical-manage-mobility-service.md#uninstall-mobility-service) y, a continuación, vuelva a instalarlo. Si se produce un error en la desinstalación, complete los siguientes pasos: a. Abra el archivo Installation_Directory/uninstall.sh y convierta en comentario la llamada a la función **StopServices**.
    b. Abra el archivo Installation_Directory/Vx/bin/uninstall.sh y convierta en comentario la llamada a la función **stop_services**.
    c. Abra el archivo Installation_Directory/Fx/uninstall.sh y convierta en comentario la sección completa que intenta detener el servicio Fx.
    d. [Desinstale](vmware-physical-manage-mobility-service.md#uninstall-mobility-service) el agente de movilidad. Después de la desinstalación correcta, reinicie el sistema e intente volver a instalar el agente de movilidad.

8. Asegúrese de que la autenticación multifactor no está habilitada en la cuenta de usuario. Azure Site Recovery no admite la autenticación multifactor para la cuenta de usuario a partir de ahora. Registre el servidor de configuración sin la cuenta de usuario habilitada para la autenticación multifactor.  

## <a name="installation-failure-failed-to-load-accounts"></a>Error de instalación: No se pudieron cargar las cuentas

Este error se produce cuando el servicio no puede leer los datos de la conexión de transporte al instalar el agente de movilidad y registrarse con el servidor de configuración. Para resolver el problema, asegúrese de que TLS 1.0 esté habilitado en su máquina de origen.

## <a name="vcenter-discovery-failures"></a>Errores de detección de vCenter

Para solucionar errores de detección de vCenter, agregue el servidor de vCenter a la configuración de proxy de la lista de omisión. 

- Descargue la herramienta PsExec desde [aquí](/sysinternals/downloads/psexec) para acceder al contenido de usuario del sistema.
- Abra Internet Explorer en el contenido de usuario del sistema mediante la ejecución de la siguiente línea de comandos: psexec -s -i "%programfiles%\Internet Explorer\iexplore.exe"
- Agregue la configuración de proxy en Internet Explorer y reinicie el servicio tmanssvc.
- Para editar la configuración del proxy DRA, ejecute cd C:\Archivos de programa\Microsoft Azure Site Recovery Provider
- A continuación, ejecute DRCONFIGURATOR. EXE /configure /AddBypassUrls [Agregue la dirección IP/FQDN de vCenter Server que se proporcionó durante el paso **Configurar vCenter Server o vSphere ESXi Server** durante la [implementación del servidor de configuración](vmware-azure-deploy-configuration-server.md#configure-settings)]

## <a name="change-the-ip-address-of-the-configuration-server"></a>Cambie la dirección IP del servidor de configuración

Recomendamos encarecidamente no cambiar la dirección IP de un servidor de configuración. Asegúrese de que todas las direcciones IP que se asignan al servidor de configuración sean direcciones IP estáticas. No use direcciones IP DHCP.

## <a name="acs50008-saml-token-is-invalid"></a>ACS50008: El token SAML no es válido

Para evitar este error, asegúrese de que la hora de su reloj del sistema no sea distinta de la hora local en más de 15 minutos. Vuelva a ejecutar el instalador para completar el registro.

## <a name="failed-to-create-a-certificate"></a>Error al crear un certificado

No se puede crear el certificado necesario para autenticarse en Site Recovery. Vuelva a ejecutar el programa de instalación tras asegurarse de que lo ejecuta como un administrador local.

## <a name="failure-to-activate-windows-license-from-server-standard-evaluation-to-server-standard"></a>Error al activar la licencia de Windows de Server Standard EVALUATION a Server Standard

1. Como parte de la implementación del servidor de configuración a través de OVF, se usa una licencia de evaluación, que es válida durante 180 días. Debe activar esta licencia antes de que expire. En caso contrario, esto puede provocar que la configuración del servidor se detenga y, por lo tanto, se entorpezcan las actividades de replicación.
2. Si no puede activar la licencia de Windows, póngase en contacto con el [equipo de soporte técnico de Windows](https://aka.ms/Windows_Support) para solucionar el problema.

## <a name="register-source-machine-with-configuration-server"></a>Registrar la máquina de origen con el servidor de configuración

### <a name="if-the-source-machine-runs-windows"></a>Si la máquina de origen ejecuta Windows

Ejecute el siguiente comando en la máquina de origen:

```
  cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
  UnifiedAgentConfigurator.exe  /CSEndPoint <configuration server IP address> /PassphraseFilePath <passphrase file path>
```

Configuración | Detalles
--- | ---
Uso | UnifiedAgentConfigurator.exe  /CSEndPoint <dirección IP del servidor de configuración\> /PassphraseFilePath <ruta de acceso del archivo de frase de contraseña\>
Registros de configuración del agente | Se encuentran en %ProgramData%\ASRSetupLogs\ASRUnifiedAgentConfigurator.log.
/CSEndPoint | Parámetro obligatorio. Especifique la dirección IP del servidor de configuración. Use una dirección IP válida.
/PassphraseFilePath |  Mandatory. La ubicación de la frase de contraseña. Use cualquier ruta de acceso de archivo local o UNC válida.

### <a name="if-the-source-machine-runs-linux"></a>Si la máquina de origen ejecuta Linux

Ejecute el siguiente comando en la máquina de origen:

```
  /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <configuration server IP address> -P /var/passphrase.txt
  ```

Configuración | Detalles
--- | ---
Uso | cd /usr/local/ASR/Vx/bin<br /><br /> UnifiedAgentConfigurator.sh -i <dirección IP del servidor de configuración\> -P <ruta de acceso del archivo de frase de contraseña\>
-i | Parámetro obligatorio. Especifique la dirección IP del servidor de configuración. Use una dirección IP válida.
-P |  Mandatory. La ruta de acceso completa del archivo en que se guarda la frase de contraseña. Uso de cualquier carpeta válida.

## <a name="unable-to-configure-the-configuration-server"></a>No se puede configurar el servidor de configuración

Si instala aplicaciones distintas del servidor de configuración en la máquina virtual, es posible que no pueda configurar el destino maestro. 

El servidor de configuración debe ser un servidor de propósito único y no está permitido usarlo como servidor compartido. 

Para obtener más información, consulte las preguntas frecuentes sobre configuración en [Implementar un servidor de configuración](vmware-azure-deploy-configuration-server.md#faqs). 

## <a name="remove-the-stale-entries-for-protected-items-from-the-configuration-server-database"></a>Eliminación de las entradas obsoletas de los elementos protegidos de la base de datos del servidor de configuración 

Para quitar la máquina protegida obsoleta del servidor de configuración, siga estos pasos. 
 
1. Para determinar la máquina de origen y la dirección IP de la entrada obsoleta: 

    1. Abra la línea de comandos MYSQL en modo de administrador. 
    2. Ejecute los siguientes comandos. 
   
        ```
        mysql> use svsdb1;
        mysql> select id as hostid, name, ipaddress, ostype as operatingsystem, from_unixtime(lasthostupdatetime) as heartbeat from hosts where name!='InMageProfiler'\G;
        ```

        Esta acción devuelve la lista de máquinas registradas junto con sus direcciones IP y el último latido. Busque el host que tiene pares de replicación obsoletos.

2. Abra un símbolo del sistema con privilegios elevados y vaya a C:\ProgramData\ASR\home\svsystems\bin. 
4. Para quitar los detalles de hosts registrados y la información de entrada obsoleta del servidor de configuración, ejecute el comando siguiente con la máquina de origen y la dirección IP de la entrada obsoleta. 
   
    `Syntax: Unregister-ASRComponent.pl -IPAddress <IP_ADDRESS_OF_MACHINE_TO_UNREGISTER> -Component <Source/ PS / MT>`
 
    Si tiene una entrada de servidor de origen de "OnPrem-VM01" con una dirección IP 10.0.0.4, use el comando siguiente en su lugar.
 
    `perl Unregister-ASRComponent.pl -IPAddress 10.0.0.4 -Component Source`
 
5. Reinicie los servicios siguientes en la máquina de origen para volver a realizar el registro con el servidor de configuración. 
 
    - Servicio de aplicación de InMage Scout
    - InMage Scout VX Agent - Sentinel/Outpost

## <a name="upgrade-fails-when-the-services-fail-to-stop"></a>Error de actualización cuando no pueden detener los servicios

Se produce un error en la actualización del servidor de configuración cuando determinados servicios no se detienen. 

Para identificar el problema, vaya a C:\ProgramData\ASRSetupLogs\CX_TP_InstallLogFile en el servidor de configuración. Si encuentra los siguientes errores, use los pasos siguientes para resolver el problema: 

```output
2018-06-28 14:28:12.943   Successfully copied php.ini to C:\Temp from C:\thirdparty\php5nts
2018-06-28 14:28:12.943   svagents service status - SERVICE_RUNNING
2018-06-28 14:28:12.944   Stopping svagents service.
2018-06-28 14:31:32.949   Unable to stop svagents service.
2018-06-28 14:31:32.949   Stopping svagents service.
2018-06-28 14:34:52.960   Unable to stop svagents service.
2018-06-28 14:34:52.960   Stopping svagents service.
2018-06-28 14:38:12.971   Unable to stop svagents service.
2018-06-28 14:38:12.971   Rolling back the install changes.
2018-06-28 14:38:12.971   Upgrade has failed.
```

Para resolver el problema:

Detenga manualmente los servicios siguientes:

- cxprocessserver
- InMage Scout VX Agent - Sentinel/Outpost 
- Agente de Microsoft Azure Recovery Services 
- Servicio Microsoft Azure Site Recovery 
- tmansvc
  
Para actualizar el servidor de configuración, ejecute la [instalación unificada](service-updates-how-to.md#links-to-currently-supported-update-rollups) de nuevo.

## <a name="azure-active-directory-application-creation-failure"></a>Error de creación de aplicación de Azure Active Directory

No tiene permisos suficientes para crear una aplicación en Azure Active Directory (AAD) utilizando la plantilla de [aplicación de virtualización abierta (OVA)](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template
).

Para resolver el problema, inicie sesión en Azure Portal y realice una de las siguientes acciones:

- Solicite el rol de desarrollador de aplicaciones en AAD. Para obtener más información sobre los distintos roles de desarrollador de aplicaciones, consulte [Permisos de roles de administrador en Azure Active Directory](../active-directory/roles/permissions-reference.md).
- Compruebe que la marca de que el **usuario puede crear la aplicación** se establece en *True* en AAD. Para más información, vea: [Cómo: portal para crear una aplicación de Azure AD y una entidad de servicio que puedan acceder a los recursos](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="process-servermaster-target-are-unable-to-communicate-with-the-configuration-server"></a>El destino maestro o el servidor de procesos no puede comunicarse con el servidor de configuración 

Los módulos de destino maestro (MT) y el servidor de procesos (PS) se pueden comunicar con el servidor de configuración (CS) y su estado se muestra como No conectado en Azure Portal.

Normalmente, esto se debe a un error con el puerto 443. Siga estos pasos para desbloquear el puerto y volver a habilitar la comunicación con CS.

**Comprobación de que el agente de MARS lo invoca el agente de destino maestro**

Para comprobar que el agente de destino maestro puede crear una sesión TCP para la dirección IP del servidor de configuración, busque un seguimiento similar al siguiente en los registros del agente de destino maestro:

TCP \<Replace IP with CS IP here>:52739 \<Replace IP with CS IP here>:443 SYN_SENT 

TCP    192.168.1.40:52739     192.168.1.40:443      SYN_SENT  // Replace IP with CS IP here

Si observa seguimientos similares a los siguientes en los registros del agente de destino maestro, el agente del destino maestro informa de errores en el puerto 443:

```output
#~> (11-20-2018 20:31:51):   ERROR  2508 8408 313 FAILED : PostToSVServer with error [at curlwrapper.cpp:CurlWrapper::processCurlResponse:212]   failed to post request: (7) - Couldn't connect to server
#~> (11-20-2018 20:31:54):   ERROR  2508 8408 314 FAILED : PostToSVServer with error [at curlwrapper.cpp:CurlWrapper::processCurlResponse:212]   failed to post request: (7) - Couldn't connect to server
```
 
Este error puede encontrarse cuando otras aplicaciones también usan el puerto 443 o debido a una configuración de firewall que bloquea el puerto.

Para resolver el problema:

- Compruebe que el puerto 443 no está bloqueado por el firewall.
- Si el puerto es inaccesible debido a otra aplicación con ese puerto, detenga y desinstale la aplicación.
  - Si no es posible detener la aplicación, configure un nuevo CS limpio.
- Reinicie el servidor de configuración.
- Reinicie el servicio IIS.

### <a name="configuration-server-is-not-connected-due-to-incorrect-uuid-entries"></a>El servidor de configuración no está conectado debido a entradas UUID incorrectas

Este error puede producirse cuando hay varias entradas UUID de instancia del servidor de configuración (CS) en la base de datos. El problema se produce con frecuencia al clonar la máquina virtual del servidor de configuración.

Para resolver el problema:

1. Quite la máquina virtual del servidor de configuración antigua u obsoleta de vCenter. Para obtener más información, consulte [Quitar servidores y deshabilitar la protección](site-recovery-manage-registration-and-protection.md).
2. Inicie sesión en la máquina virtual del servidor de configuración y conéctese a la base de datos svsdb1 de MySQL. 
3. Ejecute la siguiente consulta:

    > [!IMPORTANT]
    >
    > Compruebe que está escribiendo los detalles de UUID del servidor de configuración clonado o la entrada obsoleta del servidor de configuración que ya no se usa para proteger las máquinas virtuales. Si escribe un valor UUID incorrecto, se perderá información de todos los elementos protegidos existentes.
   
    ```
        MySQL> use svsdb1;
        MySQL> delete from infrastructurevms where infrastructurevmid='<Stale CS VM UUID>';
        MySQL> commit; 
    ```
4. Actualice la página del portal.

## <a name="an-infinite-sign-in-loop-occurs-when-entering-your-credentials"></a>Se produce un bucle infinito de inicio de sesión al escribir las credenciales

Después de escribir el nombre de usuario y la contraseña correctos en el OVF del servidor de configuración, el inicio de sesión de Azure continúa solicitando las credenciales correctas.

Este problema puede producirse cuando la hora del sistema es incorrecta.

Para resolver el problema:

Establezca la hora correcta en el equipo y vuelva a intentar iniciar de sesión. 
