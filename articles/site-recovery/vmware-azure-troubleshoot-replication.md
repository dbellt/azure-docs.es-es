---
title: Solución de problemas de replicación para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con Azure Site Recovery | Microsoft Docs
description: En este artículo se proporciona información para solucionar problemas comunes de replicación durante la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 08/2/2019
ms.author: mayg
ms.openlocfilehash: 56ee52e25c755af014de220a65ae682ea014ef1b
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949053"
---
# <a name="troubleshoot-replication-issues-for-vmware-vms-and-physical-servers"></a>Solución de problemas de replicación de máquinas virtuales de VMware y de servidores físicos

En este artículo se describen algunos problemas comunes y errores específicos que pueden surgir al replicar máquinas virtuales de VMware locales y servidores físicos en Azure mediante [Site Recovery](site-recovery-overview.md).

## <a name="step-1-monitor-process-server-health"></a>Paso 1: Supervisión del estado de los servidores de procesos

Site Recovery usa el [servidor de procesos](vmware-physical-azure-config-process-server-overview.md#process-server) para recibir y optimizar los datos replicados y enviarlos a Azure.

Le recomendamos que supervise el estado de los servidores de procesos en el portal para asegurarse de que están conectados y funcionan correctamente y, asimismo, de que hay una replicación en curso de las máquinas de origen asociadas al servidor de procesos.

- [Más información sobre](vmware-physical-azure-monitor-process-server.md) cómo supervisar servidores de proceso
- [Revisión de los procedimientos recomendados](vmware-physical-azure-troubleshoot-process-server.md#best-practices-for-process-server-deployment)
- [Solución de problemas](vmware-physical-azure-troubleshoot-process-server.md#check-process-server-health) de estado de los servidores de procesos

## <a name="step-2-troubleshoot-connectivity-and-replication-issues"></a>Paso 2: Solucionar problemas de conectividad y replicación

Los errores de replicación iniciales y continuados se deben a menudo a problemas de conectividad entre el servidor de origen y el servidor de procesos, o entre el servidor de procesos y Azure.

Para resolver estos problemas, [solucione problemas de conectividad y replicación](vmware-physical-azure-troubleshoot-process-server.md#check-connectivity-and-replication).




## <a name="step-3-troubleshoot-source-machines-that-arent-available-for-replication"></a>Paso 3: Solucionar problemas de máquinas de origen que no están disponibles para la replicación

Cuando intenta seleccionar la máquina de origen para habilitar la replicación con Site Recovery, puede que no esté disponible por uno de los siguientes motivos:

* **Dos máquinas virtuales con el mismo UUID de instancia**: Si dos máquinas virtuales en vCenter tienen el mismo UUID de instancia, en Azure Portal se muestra la primera máquina virtual detectada por el servidor de configuración. Para solucionar este problema, asegúrese de que no haya dos máquinas virtuales con el mismo UUID de instancia. Este escenario sucede habitualmente en instancias donde una máquina virtual de copia de seguridad se activa y se registra en nuestros registros de detección. Consulte [Azure Site Recovery VMware-to-Azure: How to clean up duplicate or stale entries](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx) (De VMware a Azure en Azure Site Recovery: cómo limpiar entradas duplicadas u obsoletas) para solucionarlo.
* **Credenciales de usuario de vCenter incorrectas**: Asegúrese de que ha agregado las credenciales correctas de vCenter al configurar el servidor de configuración mediante la plantilla OVF o la instalación unificada. Para comprobar las credenciales que ha agregado durante la instalación, consulte [Modificación de las credenciales para la detección automática](vmware-azure-manage-configuration-server.md#modify-credentials-for-automatic-discovery).
* **Privilegios de vCenter insuficientes**: Si los permisos proporcionados para acceder a vCenter no son los necesarios, es posible que no se puedan detectar las máquinas virtuales. Asegúrese de que los permisos descritos en [Preparación de una cuenta de detección automática](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) se agregan a la cuenta de usuario de vCenter.
* **Servidores de administración de Azure Site Recovery**: Si la máquina virtual se usa como servidor de administración con al menos uno de los roles de servidor de configuración, servidor de procesos de escalado horizontal o servidor de destino principal, no podrá seleccionar la máquina virtual en el portal. Los servidores de administración no se pueden replicar.
* **Máquina virtual ya protegida o conmutada por error mediante servicios de Azure Site Recovery**: Si la máquina virtual ya se ha conmutado por error o ya está protegida mediante Site Recovery, no estará disponible para seleccionarla para la protección en el portal. Asegúrese de que la máquina virtual que busca en el portal no esté aún protegida por ningún otro usuario o en otras suscripciones.
* **vCenter no conectado**: Compruebe si vCenter tiene el estado conectado. Para ello, vaya al almacén de Recovery Services > Infraestructura de Site Recovery > Servidores de configuración y haga clic en el servidor de configuración correspondiente; se abrirá una hoja a la derecha con detalles acerca de los servidores asociados. Compruebe si vCenter está conectado. Si está en un estado "No conectado", resuelva el problema y, después, [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server) en Portal. Una vez hecho esto, la máquina virtual se mostrará en el portal.
* **ESXi apagado**: Si el host ESXi en el que reside la máquina virtual está apagado, la máquina virtual no se mostrará ni se podrá seleccionar en Azure Portal. Encienda el host ESXi y [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server) en el portal. Una vez hecho esto, la máquina virtual se mostrará en el portal.
* **Reinicio pendiente**: Si la máquina virtual está pendiente de reiniciarse, no se podrá seleccionar en Azure Portal. Asegúrese de completar el reinicio pendiente y [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server). Una vez hecho esto, la máquina virtual se mostrará en el portal.
* **Dirección IP no encontrada**: Si la máquina virtual no tiene una dirección IP válida asociada, no se podrá seleccionar en Azure Portal. Asegúrese de asignar una dirección IP válida a la máquina virtual y [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server). Una vez hecho esto, la máquina virtual se mostrará en el portal.

### <a name="troubleshoot-protected-virtual-machines-greyed-out-in-the-portal"></a>Solución de problemas de máquinas virtuales protegidas que aparecen atenuadas en Portal

Las máquinas virtuales que se replican en Site Recovery no están disponibles en Azure Portal si hay entradas duplicadas en el sistema. Para aprender a eliminar entradas obsoletas y resolver el problema, consulte [Azure Site Recovery VMware-to-Azure: How to clean up duplicate or stale entries](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx) (De VMware a Azure en Azure Site Recovery: cómo limpiar entradas duplicadas u obsoletas).

## <a name="no-crash-consistent-recovery-point-available-for-the-vm-in-the-last-xxx-minutes"></a>"No crash consistent recovery point available for the VM in the last "XXX" minutes" (No se ha producido ningún punto de recuperación coherente con bloqueo de la máquina virtual en los últimos "XXX" minutos).

A continuación se enumeran algunos de los problemas más comunes

### <a name="initial-replication-issues-error-78169"></a>Problemas de replicación inicial [error 78169]

Aparte de confirmar que no hay problemas relacionados con la conectividad, el ancho de banda o la sincronización de hora, asegúrese de lo siguiente:

- Ningún software antivirus está bloqueando Azure Site Recovery. Obtenga [más información](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) sobre las exclusiones de carpeta necesarias para Azure Site Recovery.

### <a name="source-machines-with-high-churn-error-78188"></a>Máquinas de origen con una elevada renovación [error 78188]

Causas posibles:
- La tasa de cambio de datos (bytes de escritura/s) de los discos enumerados de la máquina virtual es superior a los [límites de Azure Site Recovery admitidos](site-recovery-vmware-deployment-planner-analyze-report.md#azure-site-recovery-limits) para el tipo de cuenta de almacenamiento de destino de replicación.
- Hay un pico repentino de la tasa de renovación, motivo por el cual hay una gran cantidad de datos pendiente de carga.

Para resolver el problema:
- Asegúrese de que el tipo de cuenta de almacenamiento de destino (Standard o Premium) se haya aprovisionado según el requisito de tasa de renovación en el origen.
- Si ya está replicando en un disco administrado Premium (tipo asrseeddisk), asegúrese de que el tamaño del disco admita la tasa de renovación observada en función de los límites de Site Recovery. Puede aumentar el tamaño de asrseeddisk si es necesario. Siga los pasos siguientes:
    - Vaya a la hoja Discos de la máquina replicada afectada y copie el nombre del disco de réplica.
    - Vaya a este disco administrado de réplica.
    - Es posible que vea un mensaje emergente en la hoja Información general que indica que se ha generado una dirección URL de SAS. Haga clic en este mensaje emergente y cancele la exportación. Omita este paso si no aparece el mensaje emergente.
    - En cuanto se revoque la dirección URL de SAS, vaya a la hoja Configuración del disco administrado y aumente el tamaño para que Azure Site Recovery admita la tasa de renovación observada en el disco de origen.
- Si la renovación observada es temporal, espere unas horas a que la carga de datos pendientes se ponga al día y se creen puntos de recuperación.
- Si el disco contiene datos no críticos como registros temporales, datos de prueba, etc., considere la posibilidad de mover estos datos a otro lugar o excluir completamente este disco de la replicación.
- Si el problema persiste, use [Deployment Planner](site-recovery-deployment-planner.md#overview) de Site Recovery como ayuda para planear la replicación.

### <a name="source-machines-with-no-heartbeat-error-78174"></a>Máquinas de origen sin latido [error 78174]

Esto sucede cuando el agente de Azure Site Recovery Mobility en la máquina de origen no se comunica con el servidor de configuración.

Para resolver el problema, use los siguientes pasos para comprobar la conectividad de red de la máquina virtual de origen con el servidor de configuración:

1. Confirme que la máquina de origen se está ejecutando.
2. Inicie sesión en la máquina de origen con una cuenta que tenga privilegios de administrador.
3. Confirme que los siguientes servicios se están ejecutando y, si no es así, reinícielos:
   - Svagents (agente de InMage Scout VX)
   - Servicio de aplicación de InMage Scout
4. En la máquina de origen, examine los registros en la ubicación de los detalles del error:

    *C:\Archivos de programa (X86)\Microsoft Azure Site Recovery\agent\svagents\*.log*

### <a name="process-server-with-no-heartbeat-error-806"></a>Servidor de procesos sin latido [error 806]
En caso de que no haya latido en el servidor de procesos, compruebe lo siguiente:
1. La máquina virtual del servidor de procesos está en funcionamiento.
2. Compruebe los siguientes registros en el servidor de procesos para obtener información detallada del error:

    *C:\Archivos de programa\ASR\home\svsystems\eventmanager\*.log*\
    y\
    *C:\Archivos de programa\ASR\home\svsystems\monitor_protection\*.log*

### <a name="master-target-server-with-no-heartbeat-error-78022"></a>Servidor de destino maestro sin latido [error 78022]

Esto sucede cuando el agente de Azure Site Recovery Mobility en el destino maestro no se comunica con el servidor de configuración.

Para resolver el problema, use los siguientes pasos para comprobar el estado del servicio:

1. Confirme que la máquina virtual del destino maestro se está ejecutando.
2. Inicie sesión en la máquina virtual del destino maestro con una cuenta que tenga privilegios de administrador.
    - Confirme que el servicio svagents se está ejecutando. Si se está ejecutando, reinicie el servicio.
    - Consulte los registros en la ubicación para obtener información detallada del error:

        *C:\Archivos de programa (X86)\Microsoft Azure Site Recovery\agent\svagents\*.log*
3. Para registrar el destino maestro en el servidor de configuración, vaya a la carpeta **%PROGRAMDATA%\ASR\Agent** y ejecute lo siguiente en el símbolo del sistema:
   ```
   cmd
   cdpcli.exe --registermt

   net stop obengine

   net start obengine

   exit
   ```

## <a name="error-id-78144---no-app-consistent-recovery-point-available-for-the-vm-in-the-last-xxx-minutes"></a>Error ID 78144: No hay ningún punto de recuperación coherente con la aplicación disponible para la VM en los XXX últimos minutos

Se han realizado mejoras en las versiones [9.23](vmware-physical-mobility-service-overview.md#mobility-service-agent-version-923-and-higher) & [9.27](site-recovery-whats-new.md#update-rollup-39) del agente de movilidad para administrar los comportamientos de error en la instalación de VSS. Asegúrese de que está en las versiones más recientes para obtener la mejor orientación sobre la solución de errores de VSS.

A continuación se enumeran algunos de los problemas más comunes

#### <a name="cause-1-known-issue-in-sql-server-20082008-r2"></a>Causa 1: Incidencia conocida en SQL Server 2008/2008 R2
**Solución:** Existe una incidencia conocida en SQL Server 2008/2008 R2. Consulte este artículo de KB [Copia de seguridad de ASR Agent u otros VSS no componente falla en un servidor que aloja SQL Server 2008 R2](https://support.microsoft.com/help/4504103/non-component-vss-backup-fails-for-server-hosting-sql-server-2008-r2)

#### <a name="cause-2-azure-site-recovery-jobs-fail-on-servers-hosting-any-version-of-sql-server-instances-with-auto_close-dbs"></a>Causa 2: Se producen errores en los trabajos de Azure Site Recovery en los servidores que alojan cualquier versión de las instancias de SQL Server con bases de datos AUTO_CLOSE
**Solución:** Consulte el [artículo](https://support.microsoft.com/help/4504104/non-component-vss-backups-such-as-azure-site-recovery-jobs-fail-on-ser) de KB


#### <a name="cause-3-known-issue-in-sql-server-2016-and-2017"></a>Causa 3: Incidencia conocida en SQL Server 2016 y 2017
**Solución:** Consulte el [artículo](https://support.microsoft.com/help/4493364/fix-error-occurs-when-you-back-up-a-virtual-machine-with-non-component) de KB

#### <a name="cause-4-app-consistency-not-enabled-on-linux-servers"></a>Causa 4: La coherencia de aplicaciones no está habilitada en servidores Linux
**Solución:** Azure Site Recovery para el sistema operativo Linux admite scripts personalizados de aplicaciones para la coherencia de aplicaciones. El agente de movilidad de Azure Site Recovery usará el script personalizado con opciones previas y posteriores para la coherencia de aplicaciones. [Aquí](/azure/site-recovery/site-recovery-faq.yml#replication) se indican los pasos necesarios para habilitarlo.

### <a name="more-causes-due-to-vss-related-issues"></a>Más causas debidas a problemas relacionados con VSS:

Para seguir solucionando el problema, compruebe los archivos en la máquina de origen para obtener el código de error exacto:

*C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\agent\Application Data\ApplicationPolicyLogs\vacp.log*

¿Cómo buscar los errores en el archivo?
Busque la cadena "vacpError" abriendo el archivo vacp.log en un editor

`Ex: `**`vacpError`**`:220#Following disks are in FilteringStopped state [\\.\PHYSICALDRIVE1=5, ]#220|^|224#FAILED: CheckWriterStatus().#2147754994|^|226#FAILED to revoke tags.FAILED: CheckWriterStatus().#2147754994|^|`

En el ejemplo anterior, **2147754994** es el código de error que indica el error como se muestra a continuación

#### <a name="vss-writer-is-not-installed---error-2147221164"></a>VSS Writer no está instalado - Error 2147221164

*Solución:* Para generar la etiqueta de coherencia de la aplicación, Azure Site Recovery usa Microsoft Volume Shadow Copy Service (VSS). Instala un proveedor de VSS para su funcionamiento y para realizar instantáneas de la coherencia de las aplicaciones. Este proveedor de VSS se instala como un servicio. En caso de que el servicio del proveedor de VSS no esté instalado, la creación de instantáneas de coherencia de las aplicaciones genera el id. de error 0x80040154 "Clase no registrada". </br>
Consulte el [artículo para solucionar problemas con la instalación de VSS Writer](./vmware-azure-troubleshoot-push-install.md#vss-installation-failures)

#### <a name="vss-writer-is-disabled---error-2147943458"></a>VSS Writer está deshabilitado - Error 2147943458

**Solución:** Para generar la etiqueta de coherencia de la aplicación, Azure Site Recovery usa Microsoft Volume Shadow Copy Service (VSS). Instala un proveedor de VSS para su funcionamiento y para realizar instantáneas de la coherencia de las aplicaciones. Este proveedor de VSS se instala como un servicio. En caso de que el servicio del proveedor de VSS esté deshabilitado, la creación de instantáneas de coherencia de las aplicaciones generará el id. de error "El servicio especificado está deshabilitado y no se puede iniciar (0x80070422)". </br>

- Si VSS está deshabilitado:
    - Compruebe que el tipo de inicio del servicio de proveedor de VSS está establecido en **Automático**.
    - Reinicie los servicios siguientes:
        - Servicio VSS
        - Proveedor VSS de Azure Site Recovery
        - Servicio VDS

####  <a name="vss-provider-not_registered---error-2147754756"></a>VSS PROVIDER NOT_REGISTERED - Error 2147754756

**Solución:** Para generar la etiqueta de coherencia de la aplicación, Azure Site Recovery usa Microsoft Volume Shadow Copy Service (VSS).
Compruebe si el servicio de proveedor de VSS de Azure Site Recovery está instalado o no. </br>

- Vuelva a intentar la instalación del proveedor con los comandos siguientes:
- Desinstale el proveedor existente: C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Uninstall.cmd
- Vuelva a instalar: C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd

Compruebe que el tipo de inicio del servicio de proveedor de VSS está establecido en **Automático**.
    - Reinicie los servicios siguientes:
        - Servicio VSS
        - Proveedor VSS de Azure Site Recovery
        - Servicio VDS

## <a name="error-id-95001---insufficient-permissions-found"></a>Id. de error 95001: se encontraron permisos insuficientes

Este error se produce al intentar habilitar la replicación y las carpetas de la aplicación no tienen permisos suficientes.

**Solución:** Para resolver este problema, asegúrese de que el usuario IUSR tiene el rol de propietario para todas las carpetas mencionadas a continuación:

- *C\ProgramData\Microsoft Azure Site Recovery\private*
- El directorio de instalación. Por ejemplo, si el directorio de instalación es la unidad F, proporcione los permisos correctos para:
    - *F:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems*
- La carpeta *\pushinstallsvc* en el directorio de instalación. Por ejemplo, si el directorio de instalación es la unidad F, proporcione los permisos correctos para:
    - *F:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc*
- La carpeta *\etc* en el directorio de instalación. Por ejemplo, si el directorio de instalación es la unidad F, proporcione los permisos correctos para:
    - *F:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\etc*
- *C:\Temp*
- *C:\thirdparty\php5nts*
- Todos los elementos de la ruta de acceso siguiente:
    - *C:\thirdparty\rrdtool-1.2.15-win32-perl58\rrdtool\Release\**

## <a name="next-steps"></a>Pasos siguientes

Si necesita más ayuda, puede publicar su pregunta en la [Página de preguntas y respuestas de Microsoft sobre Azure Site Recovery](/answers/topics/azure-site-recovery.html). Contamos con una comunidad activa y uno de nuestros ingenieros puede ayudarle.