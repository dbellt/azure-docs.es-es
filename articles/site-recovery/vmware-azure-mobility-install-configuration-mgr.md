---
title: Automatización del servicio Mobility para la recuperación ante desastres de la instalación en Azure Site Recovery
description: Cómo instalar automáticamente el servicio Mobility para la recuperación ante desastres de VMware/servidor físico con Azure Site Recovery.
services: site-recovery
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: how-to
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: edab60c800aee772ed95980515e8d381c96a2bb6
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576713"
---
# <a name="automate-mobility-service-installation"></a>Automatización de la instalación del servicio Mobility

En este artículo se explica cómo automatizar la instalación y las actualizaciones del agente del servicio Mobility en [Azure Site Recovery](site-recovery-overview.md).

Al implementar Site Recovery para la recuperación ante desastres de máquinas virtuales de VMware locales y servidores físicos en Azure, se instala el agente del servicio Mobility en cada máquina que se quiere replicar. El servicio Mobility captura las escrituras de datos en la máquina y las reenvía al servidor de procesos de Site Recovery para su replicación. Puede implementar el servicio Mobility de varias maneras:

- **Instalación de inserción**: Deje que Site Recovery instale el agente del servicio Mobility cuando habilite la replicación para una máquina en Azure Portal.
- **Instalación manual**: Instale el servicio Mobility manualmente en cada máquina. [Obtenga más información](vmware-physical-mobility-service-overview.md) acerca de la instalación de inserción y manual.
- **Implementación automatizada**: Automatice la instalación con herramientas de implementación de software, como Microsoft Endpoint Configuration Manager, o herramientas de terceros, como JetPatch.

La instalación y actualización automatizadas proporcionan una solución si:

- Su organización no permite la instalación de inserción en servidores protegidos.
- La directiva de la empresa requiere que las contraseñas se cambien periódicamente. Debe especificar una contraseña para la instalación de inserción.
- Su directiva de seguridad no permite agregar excepciones de firewall para máquinas específicas.
- Actúa como proveedor de servicios de hosting y no quiere proporcionar las credenciales de las máquinas de los clientes que se necesitan para la instalación de inserción con Site Recovery.
- Tiene que escalar las instalaciones del agente a muchos servidores de forma simultánea.
- Quiere programar las instalaciones y las actualizaciones durante las ventanas de mantenimiento planeado.

## <a name="prerequisites"></a>Prerrequisitos

Para automatizar la instalación, necesita los siguientes elementos:

- Una solución de instalación de software implementada como [Configuration Manager](/configmgr/) o [JetPatch](https://jetpatch.com/microsoft-azure-site-recovery/).
- Los requisitos previos de implementación en [Azure](tutorial-prepare-azure.md) y [locales](vmware-azure-tutorial-prepare-on-premises.md) para la recuperación ante desastres de VMware, o para la recuperación ante desastres de [servidores físicos](physical-azure-disaster-recovery.md). Revise los [requisitos de compatibilidad](vmware-physical-azure-support-matrix.md) para la recuperación ante desastres.

## <a name="prepare-for-automated-deployment"></a>Preparación para la implementación automatizada

En la tabla siguiente se resumen las herramientas y los procesos necesarios para automatizar la implementación del servicio Mobility.

**Herramienta** | **Detalles** | **Instrucciones**
--- | --- | ---
**Administrador de configuración** | 1. Compruebe que cumple los [requisitos previos](#prerequisites) enumerados anteriormente. <br/><br/> 2. Implemente la recuperación ante desastres mediante la configuración del entorno de origen, incluyendo la descarga de un archivo OVA para implementar el servidor de configuración de Site Recovery como una máquina virtual de VMware mediante una plantilla OVF.<br/><br/> 3. Registre el servidor de configuración con el servicio Site Recovery, configure el entorno de Azure de destino y configure una directiva de replicación.<br/><br/> 4. Para la implementación automatizada del servicio Mobility, cree un recurso compartido de red que contenga la frase de contraseña del servidor de configuración y los archivos de instalación del servicio Mobility.<br/><br/> 5. Cree un paquete de Configuration Manager que contenga la instalación o las actualizaciones y prepárese para la implementación del servicio Mobility.<br/><br/> 6. Luego puede habilitar la replicación en Azure para las máquinas que tienen instalado el servicio Mobility. | [Automatizar con Configuration Manager](#automate-with-configuration-manager)
**JetPatch** | 1. Compruebe que cumple los [requisitos previos](#prerequisites) enumerados anteriormente. <br/><br/> 2. Implemente la recuperación ante desastres mediante la configuración del entorno de origen, incluyendo la descarga e implementación de Agent Manager de JetPatch para Azure Site Recovery en su entorno de Site Recovery, mediante una plantilla OVF.<br/><br/> 3. Registre el servidor de configuración con Site Recovery, configure el entorno de Azure de destino y configure una directiva de replicación.<br/><br/> 4. Para la implementación automatizada, inicialice y complete la configuración de Agent Manager de JetPatch.<br/><br/> 5. En JetPatch, puede crear una directiva de Site Recovery para automatizar la implementación y la actualización del agente del servicio Mobility. <br/><br/> 6. Luego puede habilitar la replicación en Azure para las máquinas que tienen instalado el servicio Mobility. | [Automatización con JetPatch Agent Manager](https://jetpatch.com/microsoft-azure-site-recovery-deployment-guide/)<br/><br/> [Solución de problemas de instalación del agente en JetPatch](https://kc.jetpatch.com/hc/articles/360035981812)

## <a name="automate-with-configuration-manager"></a>Automatizar con Configuration Manager

### <a name="prepare-the-installation-files"></a>Preparar los archivos de instalación

1. Asegúrese de que cumple los requisitos previos.
1. Cree un recurso compartido de archivos de red seguro (recurso compartido de SMB) al que pueda acceder mediante la máquina que ejecuta el servidor de configuración.
1. En Configuration Manager, [clasifique los servidores](/sccm/core/clients/manage/collections/automatically-categorize-devices-into-collections) en los que quiere instalar o actualizar el servicio Mobility. Una colección debe contener todos los servidores de Windows y la otra todos los servidores de Linux.
1. En el recurso compartido de red, cree una carpeta:

   - Para la instalación en máquinas con Windows, cree una carpeta denominada _MobSvcWindows_.
   - Para la instalación en máquinas con Linux, cree una carpeta denominada _MobSvcLinux_.

1. Inicie sesión en la máquina del servidor de configuración.
1. En la máquina del servidor de configuración, abra un símbolo del sistema administrativo.
1. Para generar el archivo de frase de contraseña, ejecute este comando:

    ```Console
    cd %ProgramData%\ASR\home\svsystems\bin
    genpassphrase.exe -v > MobSvc.passphrase
    ```

1. Copie el archivo _MobSvc.passphrase_ en la carpeta de Windows y en la carpeta de Linux.
1. Para ir a la carpeta que contiene los archivos de instalación, ejecute este comando:

    ```Console
    cd %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository
    ```

1. Copie estos archivos de instalación en el recurso compartido de red:

   - En Windows, copie _Microsoft-ASR_UA_version_Windows_GA_date_Release.exe_ en _MobSvcWindows_.
   - En Linux, copie los siguientes archivos en _MobSvcLinux_:
     - _Microsoft-ASR_UARHEL6-64release.tar.gz_
     - _Microsoft-ASR_UARHEL7-64release.tar.gz_
     - _Microsoft-ASR_UASLES11-SP3-64release.tar.gz_
     - _Microsoft-ASR_UASLES11-SP4-64release.tar.gz_
     - _Microsoft-ASR_UAOL6-64release.tar.gz_
     - _Microsoft-ASR_UAUBUNTU-14.04-64release.tar.gz_

1. Como se ha explicado en los procedimientos anteriores, copie el código en las carpetas de Windows o Linux. Damos por hecho que:

   - La dirección IP del servidor de configuración es `192.168.3.121`.
   - El recurso compartido de archivos de red seguro es `\\ContosoSecureFS\MobilityServiceInstallers`.

### <a name="copy-code-to-the-windows-folder"></a>Copiar código en la carpeta de Windows

Copie el siguiente código:

- Guarde el código en la carpeta _MobSvcWindows_ como _install.bat_.
- Reemplace los marcadores de posición `[CSIP]` de este script por los valores reales de la dirección IP del servidor de configuración.
- El script admite nuevas instalaciones del agente del servicio Mobility y actualizaciones de agentes ya instalados.

```DOS
Time /t >> C:\Temp\logfile.log
REM ==================================================
REM ==== Clean up the folders ========================
RMDIR /S /q %temp%\MobSvc
MKDIR %Temp%\MobSvc
MKDIR C:\Temp
REM ==================================================

REM ==== Copy new files ==============================
COPY M*.* %Temp%\MobSvc
CD %Temp%\MobSvc
REN Micro*.exe MobSvcInstaller.exe
REM ==================================================

REM ==== Extract the installer =======================
MobSvcInstaller.exe /q /x:%Temp%\MobSvc\Extracted
REM ==== Wait 10s for extraction to complete =========
TIMEOUT /t 10
REM =================================================

REM ==== Perform installation =======================
REM =================================================

CD %Temp%\MobSvc\Extracted
whoami >> C:\Temp\logfile.log
SET PRODKEY=HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
REG QUERY %PRODKEY%\{275197FC-14FD-4560-A5EB-38217F80CBD1}
IF NOT %ERRORLEVEL% EQU 0 (
    echo "Product is not installed. Goto INSTALL." >> C:\Temp\logfile.log
    GOTO :INSTALL
) ELSE (
    echo "Product is installed." >> C:\Temp\logfile.log

    echo "Checking for Post-install action status." >> C:\Temp\logfile.log
    GOTO :POSTINSTALLCHECK
)

:POSTINSTALLCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "PostInstallActions" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Post-install actions succeeded. Checking for Configuration status." >> C:\Temp\logfile.log
        GOTO :CONFIGURATIONCHECK
    ) ELSE (
        echo "Post-install actions didn't succeed. Goto INSTALL." >> C:\Temp\logfile.log
        GOTO :INSTALL
    )

:CONFIGURATIONCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "AgentConfigurationStatus" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded. Goto UPGRADE." >> C:\Temp\logfile.log
        GOTO :UPGRADE
    ) ELSE (
        echo "Configuration didn't succeed. Goto CONFIGURE." >> C:\Temp\logfile.log
        GOTO :CONFIGURE
    )


:INSTALL
    echo "Perform installation." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Role MS /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Installation has succeeded." >> C:\Temp\logfile.log
        (GOTO :CONFIGURE)
    ) ELSE (
        echo "Installation has failed." >> C:\Temp\logfile.log
        GOTO :ENDSCRIPT
    )

:CONFIGURE
    echo "Perform configuration." >> C:\Temp\logfile.log
    cd "C:\Program Files (x86)\Microsoft Azure Site Recovery\agent"
    UnifiedAgentConfigurator.exe  /CSEndPoint "[CSIP]" /PassphraseFilePath %Temp%\MobSvc\MobSvc.passphrase
    IF %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Configuration has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:UPGRADE
    echo "Perform upgrade." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Upgrade has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Upgrade has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:ENDSCRIPT
    echo "End of script." >> C:\Temp\logfile.log
```

### <a name="copy-code-to-the-linux-folder"></a>Copiar código en la carpeta de Linux

Copie el siguiente código:

- Guarde el código en la carpeta _MobSvcLinux_ como _install_linux.sh_.
- Reemplace los marcadores de posición `[CSIP]` de este script por los valores reales de la dirección IP del servidor de configuración.
- El script admite nuevas instalaciones del agente del servicio Mobility y actualizaciones de agentes ya instalados.

```Bash
#!/usr/bin/env bash

rm -rf /tmp/MobSvc
mkdir -p /tmp/MobSvc
INSTALL_DIR='/usr/local/ASR'
VX_VERSION_FILE='/usr/local/.vx_version'

echo "=============================" >> /tmp/MobSvc/sccm.log
echo `date` >> /tmp/MobSvc/sccm.log
echo "=============================" >> /tmp/MobSvc/sccm.log

if [ -f /etc/oracle-release ] && [ -f /etc/redhat-release ]; then
    if grep -q 'Oracle Linux Server release 6.*' /etc/oracle-release; then
        if uname -a | grep -q x86_64; then
            OS="OL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *OL6*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/redhat-release ]; then
    if grep -q 'Red Hat Enterprise Linux Server release 6.* (Santiago)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 6.* (Final)' /etc/redhat-release || \
        grep -q 'CentOS release 6.* (Final)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL6*.tar.gz /tmp/MobSvc
        fi
    elif grep -q 'Red Hat Enterprise Linux Server release 7.* (Maipo)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 7.* (Core)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL7-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL7*.tar.gz /tmp/MobSvc
                fi
    fi
elif [ -f /etc/SuSE-release ] && grep -q 'VERSION = 11' /etc/SuSE-release; then
    if grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 3' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP3-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP3*.tar.gz /tmp/MobSvc
        fi
    elif grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 4' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP4-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP4*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/lsb-release ] ; then
    if grep -q 'DISTRIB_RELEASE=14.04' /etc/lsb-release ; then
       if uname -a | grep -q x86_64; then
           OS="UBUNTU-14.04-64"
           echo $OS >> /tmp/MobSvc/sccm.log
           cp *UBUNTU-14*.tar.gz /tmp/MobSvc
       fi
    fi
else
    exit 1
fi

if [ -z "$OS" ]; then
    exit 1
fi

Install()
{
    echo "Perform Installation." >> /tmp/MobSvc/sccm.log
    ./install -q -d ${INSTALL_DIR} -r MS -v VmWare
    RET_VAL=$?
    echo "Installation Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Installation has succeeded. Proceed to configuration." >> /tmp/MobSvc/sccm.log
        Configure
    else
        echo "Installation has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Configure()
{
    echo "Perform configuration." >> /tmp/MobSvc/sccm.log
    ${INSTALL_DIR}/Vx/bin/UnifiedAgentConfigurator.sh -i [CSIP] -P MobSvc.passphrase
    RET_VAL=$?
    echo "Configuration Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Configuration has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Configuration has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Upgrade()
{
    echo "Perform Upgrade." >> /tmp/MobSvc/sccm.log
    ./install -q -v VmWare
    RET_VAL=$?
    echo "Upgrade Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Upgrade has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Upgrade has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

cp MobSvc.passphrase /tmp/MobSvc
cd /tmp/MobSvc

tar -zxvf *.tar.gz

if [ -e ${VX_VERSION_FILE} ]; then
    echo "${VX_VERSION_FILE} exists. Checking for configuration status." >> /tmp/MobSvc/sccm.log
    agent_configuration=$(grep ^AGENT_CONFIGURATION_STATUS "${VX_VERSION_FILE}" | cut -d"=" -f2 | tr -d " ")
    echo "agent_configuration=$agent_configuration" >> /tmp/MobSvc/sccm.log
     if [ "$agent_configuration" == "Succeeded" ]; then
        echo "Agent is already configured. Proceed to Upgrade." >> /tmp/MobSvc/sccm.log
        Upgrade
    else
        echo "Agent is not configured. Proceed to Configure." >> /tmp/MobSvc/sccm.log
        Configure
    fi
else
    Install
fi

cd /tmp

```

### <a name="create-a-package"></a>Creación de un paquete

1. Inicie sesión en la consola de Configuration Manager y vaya a **Biblioteca de software** > **Administración de aplicaciones** > **Paquetes**.
1. Haga clic con el botón derecho en **Paquetes** > **Crear paquete**.
1. Proporcione los detalles del paquete, incluyendo el nombre, descripción, fabricante, lenguaje y versión.
1. Seleccione **Este paquete contiene archivos de origen**.
1. Haga clic en **Examinar** y seleccione el recurso compartido de red y la carpeta que contiene el instalador correspondiente (_MobSvcWindows_ o _MobSvcLinux_). Después, seleccione **Siguiente**.

   ![Captura de pantalla del Asistente para crear paquetes y programas](./media/vmware-azure-mobility-install-configuration-mgr/create_sccm_package.png)

1. En la página **Elija el tipo de programa que desea crear**, seleccione **Programa estándar** > **Siguiente**.

   ![Captura de pantalla del Asistente para crear paquetes y programas en la que se muestra la opción Programa estándar.](./media/vmware-azure-mobility-install-configuration-mgr/sccm-standard-program.png)

1. En la página **Especifique la información sobre este programa estándar**, especifique los valores siguientes:

    **Parámetro** | **Valor de Windows** | **Valor de Linux**
    --- | --- | ---
    **Nombre** | Instalación del Servicio de movilidad de Microsoft Azure (Windows) | Instalación del Servicio de movilidad de Microsoft Azure (Linux).
    **Línea de comandos** | install.bat | ./install_linux.sh
    **El programa se puede ejecutar** | Tanto si un usuario inició sesión como si no | Tanto si un usuario inició sesión como si no
    **Otros parámetros** | Usar el valor predeterminado | Usar el valor predeterminado

   ![Captura de pantalla en la que se muestra la información que se puede especificar para el programa estándar.](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties.png)

1. En **Especifique los requisitos para este programa estándar**, haga las siguientes tareas:

   - Para máquinas con Windows, seleccione **Este programa solo puede ejecutarse en las plataformas especificadas**. Luego seleccione los [sistemas operativos Windows compatibles](vmware-physical-azure-support-matrix.md#replicated-machines) y después **Siguiente**.
   - Para máquinas con Linux, seleccione **Este programa puede ejecutarse en cualquier plataforma**. Luego, seleccione **Siguiente**.

1. Finalice al asistente.

### <a name="deploy-the-package"></a>Implementación del paquete

1. En la consola de Configuration Manager, haga clic con el botón derecho en el paquete y seleccione **Distribuir contenido**.

   ![Captura de pantalla de la consola de Configuration Manager](./media/vmware-azure-mobility-install-configuration-mgr/sccm_distribute.png)

1. Seleccione los puntos de distribución en los que se deben copiar los paquetes. [Más información](/sccm/core/servers/deploy/configure/install-and-configure-distribution-points).
1. Finalice el asistente. El paquete comienza a replicarse en los puntos de distribución especificados.
1. Cuando la distribución del paquete finalice, haga clic con el botón derecho en el paquete > **Implementar**.

   ![Captura de pantalla de la consola de Configuration Manager en la que se muestra la opción de menú Implementar.](./media/vmware-azure-mobility-install-configuration-mgr/sccm_deploy.png)

1. Seleccione la colección de dispositivos Windows o Linux que creó anteriormente.
1. En la página **Especifique el destino del contenido**, seleccione **Puntos de distribución**.
1. En la página **Especificar configuración para controlar cómo se implementará este software**, defina **Propósito** como **Requerido**.

   ![Captura de pantalla del Asistente para implementar software](./media/vmware-azure-mobility-install-configuration-mgr/sccm-deploy-select-purpose.png)

1. En **Especifique la programación de esta implementación**, configure una programación. [Más información](/sccm/apps/deploy-use/deploy-applications#bkmk_deploy-sched).

   - El servicio Mobility se instala de acuerdo con la programación que especifique.
   - Para evitar los reinicios innecesarios, programe la instalación del paquete durante la ventana de mantenimiento mensual o la ventana de actualizaciones de software.

1. En la página **Puntos de distribución**, configure las opciones y finalice el asistente.
1. Puede supervisar el progreso de la implementación en la consola de Configuration Manager. Vaya a **Supervisión** > **Implementaciones** >  _\<your package name\>_ .

### <a name="uninstall-the-mobility-service"></a>Desinstalación del servicio Mobility

Puede crear paquetes de Configuration Manager para desinstalar el servicio Mobility. Por ejemplo, el siguiente script desinstala el servicio Mobility:

```DOS
Time /t >> C:\logfile.log
REM ==================================================
REM ==== Check if Mob Svc is already installed =======
REM ==== If not installed no operation required ========
REM ==== Else run uninstall command =====================
REM ==== {275197FC-14FD-4560-A5EB-38217F80CBD1} is ====
REM ==== guid for Mob Svc Installer ====================
whoami >> C:\logfile.log
NET START | FIND "InMage Scout Application Service"
IF  %ERRORLEVEL% EQU 1 (GOTO :INSTALL) ELSE GOTO :UNINSTALL
:NOOPERATION
                echo "No Operation Required." >> c:\logfile.log
                GOTO :ENDSCRIPT
:UNINSTALL
                echo "Uninstall" >> C:\logfile.log
                MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
:ENDSCRIPT
```

## <a name="next-steps"></a>Pasos siguientes

[Habilitar la replicación](vmware-azure-enable-replication.md) para máquinas virtuales.
