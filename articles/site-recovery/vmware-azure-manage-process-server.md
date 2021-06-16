---
title: Administración de un servidor de procesos para la recuperación ante desastres de servidores físicos o máquinas virtuales de VMware en Azure Site Recovery
description: En este artículo se describe la administración de un servidor de procesos para la recuperación ante desastres de servidores físicos o máquinas virtuales de VMware con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: 514175fec5d1fc5c7572f805c5b22465dfe6a2b7
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576885"
---
# <a name="manage-process-servers"></a>Administración de servidores de proceso

En este artículo se describen las tareas comunes para supervisar el servidor de procesos de Site Recovery.

El servidor de procesos se usa para recibir, optimizar y enviar datos de replicación a Azure. También realiza la instalación de inserción de Mobility Service en las VM de VMware y los servidores físicos que quiere replicar, además de realizar la detección automática de máquinas locales. Para replicar las VM de VMware o los servidores físicos locales en Azure, el servidor de procesos se instala de forma predeterminada en la máquina del servidor de configuración. 

- Para implementaciones de gran tamaño, es posible que necesite servidores de procesos locales adicionales para escalar la capacidad.
- Para la conmutación por recuperación de Azure al entorno local, debe configurar un servidor de procesos temporal en Azure. Esta máquina virtual se puede eliminar cuando finalice la conmutación por recuperación. 

Más información sobre el servidor de procesos.


## <a name="upgrade-a-process-server"></a>Actualizar un servidor de procesos

Al implementar un servidor de procesos de forma local o como una VM de Azure para la conmutación por recuperación, se instala la versión más reciente del servidor de procesos. Los equipos de Site Recovery publican periódicamente correcciones y mejoras y se recomienda mantener actualizados los servidores de procesos. Puede actualizar un servidor de procesos como sigue:

[!INCLUDE [site-recovery-vmware-upgrade -process-server](../../includes/site-recovery-vmware-upgrade-process-server-internal.md)]


## <a name="move-vms-to-balance-the-process-server-load"></a>Mover las VM para equilibrar la carga del servidor de procesos

Equilibre la carga moviendo las VM entre dos servidores de procesos, como sigue:

1. En el almacén, en **Administrar** haga clic en **Infraestructura de Site Recovery**. En **For VMware & Physical machines** (Para VMware y máquinas físicas), haga clic en **Servidores de configuración**.
2. Haga clic en el servidor de configuración en el que están registrados los servidores de procesos.
3. Haga clic en el servidor de procesos para el que quiere equilibrar la carga del tráfico.

    ![La captura de pantalla muestra un servidor de procesos para el que se puede equilibrar la carga del tráfico.](media/vmware-azure-manage-process-server/LoadBalance.png)

4. Haga clic en **Equilibrar la carga**, seleccione el servidor de procesos de destino al que quiere mover las máquinas. Luego, haga clic en **Aceptar**.

    ![La captura de pantalla muestra el panel de equilibrio de carga, con la opción Seleccionar servidor de procesos de destino resaltada.](media/vmware-azure-manage-process-server/LoadPS.PNG)

2. Haga clic en **Seleccionar máquinas** y elija las máquinas que quiere mover del servidor de procesos actual al de destino. Los detalles del cambio promedio de datos se muestran en cada máquina virtual. A continuación, haga clic en **Aceptar**. 
3. En el almacén, supervise el progreso del trabajo en **Supervisión** > **Trabajos de Site Recovery**.

Los cambios tardarán unos 15 minutos en verse reflejados en el portal. Para un efecto más rápido, [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="switch-an-entire-workload-to-another-process-server"></a>Cambiar una carga de trabajo completo a otro servidor de procesos

Mueva toda la carga de trabajo controlada por un servidor de procesos a otro servidor de procesos como se indica a continuación:

1. En el almacén, en **Administrar** haga clic en **Infraestructura de Site Recovery**. En **For VMware & Physical machines** (Para VMware y máquinas físicas), haga clic en **Servidores de configuración**.
2. Haga clic en el servidor de configuración en el que están registrados los servidores de procesos.
3. Haga clic en el servidor de procesos desde el que quiere cambiar la carga de trabajo.
4. Haga clic en **Cambiar**, seleccione el servidor de procesos de destino al que quiere mover la carga de trabajo. Luego, haga clic en **Aceptar**.

    ![La captura de pantalla muestra el panel Seleccionar servidor de procesos de destino.](media/vmware-azure-manage-process-server/Switch.PNG)

5. En el almacén, supervise el progreso del trabajo en **Supervisión** > **Trabajos de Site Recovery**.

Los cambios tardarán unos 15 minutos en verse reflejados en el portal. Para un efecto más rápido, [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="register-a-master-target-server"></a>Registro de un servidor de destino maestro

El servidor de destino maestro reside en el servidor de configuración y en los servidores de procesos de escalado horizontal. Debe estar registrado en el servidor de configuración. En caso de que se produzca un error en este registro, el estado de los elementos protegidos puede verse afectado. Para registrar el servidor de destino maestro en el servidor de configuración, inicie sesión en el servidor de configuración específico o en el servidor de procesos de escalado horizontal en el que se requiere el registro. Vaya a carpeta **%PROGRAMDATA%\ASR\Agent** y ejecute lo siguiente en el símbolo del sistema de administrador.

   ```
   cmd
   cdpcli.exe --registermt

   net stop obengine

   net start obengine

   exit
   ```

## <a name="reregister-a-process-server"></a>Volver a registrar un servidor de procesos

Vuelva a registrar un servidor de procesos que se ejecute de forma local o en una VM Azure con el servidor de configuración como sigue:

[!INCLUDE [site-recovery-vmware-register-process-server](../../includes/site-recovery-vmware-register-process-server.md)]

Después de guardar la configuración, haga lo siguiente:

1. En el servidor de procesos, abra un símbolo del sistema de administrador.
2. Vaya a la carpeta **%PROGRAMDATA%\ASR\Agent** y ejecute el comando:

    ```
    cdpcli.exe --registermt
    net stop obengine
    net start obengine
    ```

## <a name="modify-proxy-settings-for-an-on-premises-process-server"></a>Modificar la configuración de proxy para un servidor de procesos local

Si un servidor de procesos local usa a un proxy para conectarse a Azure, puede modificar la configuración de proxy como sigue:

1. Inicie sesión en la máquina del servidor de procesos. 
2. Abra la ventana de comandos de administrador de PowerShell y ejecute el comando siguiente:
   ```powershell
   $pwd = ConvertTo-SecureString -String MyProxyUserPassword
   Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber –ProxyUserName domain\username -ProxyPassword $pwd
   net stop obengine
   net start obengine
   ```
2. Vaya a la carpeta **%PROGRAMDATA%\ASR\Agent** y ejecute este comando:
   ```
   cmd
   cdpcli.exe --registermt

   net stop obengine

   net start obengine

   exit
   ```

## <a name="remove-a-process-server"></a>Eliminar un servidor de procesos

[!INCLUDE [site-recovery-vmware-unregister-process-server](../../includes/site-recovery-vmware-unregister-process-server.md)]

## <a name="exclude-folders-from-anti-virus-software"></a>Excluir las carpetas del software antivirus

Si hay un software antivirus en ejecución en un servidor de procesos de escalabilidad horizontal (o servidor de destino maestro), excluya las siguientes carpetas de las operaciones del programa antivirus:


- C:\Archivos de programa\Microsoft Azure Recovery Services Agent
- C:\ProgramData\ASR
- C:\ProgramData\ASRLogs
- C:\ProgramData\ASRSetupLogs
- C:\ProgramData\LogUploadServiceLogs
- C:\ProgramData\Microsoft Azure Site Recovery
- El directorio de instalación del servidor de procesos. Por ejemplo: C:\Program Files (x86)\Microsoft Azure Site Recovery