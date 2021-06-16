---
title: Planeamiento de la capacidad para la recuperación ante desastres de VMware con Azure Site Recovery
description: Este artículo puede ayudarle a planear la capacidad y el escalado cuando configure la recuperación ante desastres de máquinas virtuales de VMware en Azure con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: 9a8706ee361ea7cb34b1e843b3bdd29ce381d48e
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110577367"
---
# <a name="plan-capacity-and-scaling-for-vmware-disaster-recovery-to-azure"></a>Planeamiento de la capacidad y escalado para la recuperación ante desastres de VMware en Azure

Use este artículo para planear la capacidad y el escalado cuando se replican VM de VMware locales y servidores físicos en Azure con [Azure Site Recovery](site-recovery-overview.md).

## <a name="how-do-i-start-capacity-planning"></a>¿Cómo se puede iniciar el planeamiento de la capacidad?

Para conocer los requisitos de la infraestructura de Azure Site Recovery, recopile información sobre el entorno de replicación ejecutando [Azure Site Recovery Deployment Planner](./site-recovery-deployment-planner.md) para la replicación de VMware. Para obtener más información, consulte [Información sobre Azure Site Recovery Deployment Planner para VMware en Azure](site-recovery-deployment-planner.md). 

Site Recovery Deployment Planner proporciona un informe con toda la información sobre las máquinas virtuales compatibles e incompatibles, los discos por máquina virtual y la tasa de cambio de datos por disco. También resume los requisitos de ancho de banda de red para satisfacer el RPO de destino y la infraestructura de Azure necesaria para una replicación y conmutación por error de prueba correctas.

## <a name="capacity-considerations"></a>Consideraciones de capacidad

Componente | Detalles
--- | ---
**Replicación** | **Tasa máxima de cambios diaria:** un equipo protegido solo puede utilizar un único servidor de procesos. Un servidor de un solo proceso puede administrar una tasa de cambio diaria de hasta 2 TB. Por lo tanto, la tasa máxima de cambios diaria que admite una máquina protegida es de 2 TB.<br /><br /> **Capacidad de proceso máxima:** una máquina replicada puede pertenecer a una cuenta de almacenamiento en Azure. Una cuenta de Azure Storage estándar puede controlar un máximo de 20 000 solicitudes por segundo. Se recomienda que limite el número de operaciones de entrada/salida por segundo (IOPS) a través de una máquina de origen a 20 000. Por ejemplo, si tiene una máquina de origen con 5 discos y cada disco genera 120 E/S por segundo (8 K de tamaño) en la máquina de origen, esta se encontrará dentro del límite de Azure de 500 E/S por segundo por disco. (El número de cuentas de almacenamiento necesario es igual al IOPS de máquina de origen total dividido por 20 000).
**Servidor de configuración** | El servidor de configuración debe ser capaz de controlar la capacidad de tasa de cambios diaria en todas las cargas de trabajo que se ejecutan en máquinas protegidas. La máquina de configuración debe tener suficiente ancho de banda para replicar continuamente los datos a Azure Storage.<br /><br /> Como procedimiento recomendado, coloque el servidor de configuración en la misma red y el mismo segmento de LAN que las máquinas que desea proteger. Puede colocar el servidor de configuración en otra red, pero las máquinas que desea proteger deben contar con la visibilidad de red de nivel L3 en ella.<br /><br /> En la tabla de la siguiente sección se resumen las recomendaciones de tamaño para el servidor de configuración.
**Servidor de proceso** | El primer servidor de procesos está instalado en el servidor de configuración de forma predeterminada. Puede implementar servidores de procesos adicionales para escalar el entorno. <br /><br /> El servidor de procesos recibe los datos de replicación de las máquinas protegidas. Optimiza los datos mediante almacenamiento en caché, compresión y cifrado. A continuación, envía los datos a Azure. La máquina del servidor de procesos debe tener los recursos suficientes para realizar estas tareas.<br /><br /> El servidor de procesos utiliza una caché basada en disco. Utilice un disco de caché independiente con 600 GB o más de capacidad para controlar los cambios en los datos almacenados ante la eventualidad de una interrupción o un cuello de botella en la red.

## <a name="size-recommendations-for-the-configuration-server-and-inbuilt-process-server"></a>Recomendaciones para el tamaño del servidor de configuración y el servidor de procesos integrado

Un servidor de configuración en el que se utiliza el servidor de procesos integrado para proteger la carga de trabajo puede administrar hasta 200 máquinas virtuales en función de las siguientes configuraciones:

CPU | Memoria | Tamaño de disco de caché | Frecuencia de cambio de datos | Máquinas protegidas
--- | --- | --- | --- | ---
8 vCPU (2 sockets * 4 núcleos \@ 2,5 GHz) | 16 GB | 300 GB | 500 GB o menos | Úselo para replicar menos de 100 máquinas.
12 vCPU (2 sockets * 6 núcleos \@ 2,5 GHz) | 18 GB | 600 GB | De 501 GB a 1 TB | Úselo para replicar de 100 a 150 máquinas.
16 vCPU (2 sockets * 8 núcleos \@ 2,5 GHz) | 32 GB | 1 TB | >1 TB a 2 TB | Úselo para replicar de 151 a 200 máquinas.
Implemente otro servidor de configuración a través de la [plantilla OVF](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template). | | | | Implemente un nuevo servidor de configuración si se van a replicar más de 200 máquinas.
Implemente otro [servidor de procesos](vmware-azure-set-up-process-server-scale.md#download-installation-file). | | | >2 TB| Implemente un nuevo servidor de procesos de escalabilidad horizontal si la tasa de cambios de datos diaria global supera los 2 TB.

En estas configuraciones:

* Cada máquina de origen tiene tres discos de 100 GB.
* Usamos almacenamiento de pruebas comparativas de 8 unidades de firma de acceso compartido de 10 K RPM con RAID 10 para las mediciones de disco de caché.

## <a name="size-recommendations-for-the-process-server"></a>Recomendaciones de tamaño para el servidor de procesos

El servidor de procesos es el componente que controla la replicación de datos en Azure Site Recovery. Si la tasa de cambios diaria es mayor que 2 TB, necesita agregar servidores de procesos de escalabilidad horizontal para controlar la carga de replicación. Para escalar horizontalmente, puede:

* Incremente el número de servidores de configuración mediante la implementación a través de una [plantilla OVF](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template). Por ejemplo, puede proteger hasta 400 máquinas con dos servidores de configuración.
* Agregue [servidores de procesos de escalabilidad horizontal](vmware-azure-set-up-process-server-scale.md#download-installation-file). Use los servidores de procesos de escalabilidad horizontal para controlar el tráfico de replicación en lugar (o además) del servidor de configuración.

En la siguiente tabla se describe este escenario:

* Ha configurado un servidor de procesos de escalabilidad horizontal.
* Ha configurado máquinas virtuales protegidas para utilizar el servidor de procesos de escalabilidad horizontal.
* Cada máquina de origen protegida tiene tres discos de 100 GB.

Servidor de procesos adicionales | Tamaño de disco de caché | Frecuencia de cambio de datos | Máquinas protegidas
--- | --- | --- | ---
4 vCPU (2 sockets * 2 núcleos \@ 2,5 GHz), 8 GB de memoria | 300 GB | 250 GB o menos | Úselo para replicar 85 máquinas o menos.
8 vCPU (2 sockets * 4 núcleos \@ 2,5 GHz), 12 GB de memoria | 600 GB | De 251 GB a 1 TB | Úselo para replicar de 86 a 150 máquinas.
12 vCPU (2 sockets * 6 núcleos \@ 2,5 GHz), 24 GB de memoria | 1 TB | >1 TB a 2 TB | Úselo para replicar de 151 a 225 máquinas.

La forma de escalar los servidores depende de si prefiere un modelo de escalado vertical u horizontal. Para escalar verticalmente, implemente algunos servidores de configuración y servidores de procesos de tecnología avanzada. Para escalar horizontalmente, implemente más servidores que tengan menos recursos. Por ejemplo, si desea proteger 200 máquinas con una tasa diaria de cambios de datos global de 1,5 TB, podría realizar alguna de las siguientes acciones:

* Configurar un servidor de procesos (16 vCPU, 24 GB de RAM).
* Configurar dos servidores de procesos (2 x 8 vCPU y 2* 12 GB de RAM).

## <a name="control-network-bandwidth"></a>Ancho de banda de red de control

Después de ejecutar la herramienta [Site Recovery Deployment Planner](site-recovery-deployment-planner.md) para calcular el ancho de banda necesario para la replicación (la replicación inicial y la diferencial), tiene un par de opciones para controlar la cantidad de ancho de banda que se utiliza para la replicación:

* **Limitar el ancho de banda**: el tráfico de VMware que se replica en Azure pasa por un servidor de procesos específico. También puede limitar el ancho de banda en las máquinas que se ejecutan como servidores de procesos.
* **Influir en el ancho de banda**: puede influir en el ancho de banda utilizado para la replicación mediante un par de claves del Registro:
  * El valor del Registro **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication\UploadThreadsPerVM** especifica el número de subprocesos que se utilizan para la transferencia de datos (replicación inicial o diferencial) de un disco. Un valor mayor aumenta el ancho de banda de red utilizado para la replicación.
  * El valor del Registro **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication\DownloadThreadsPerVM** especifica el número de subprocesos usados para la transferencia de datos durante la conmutación por recuperación.

### <a name="throttle-bandwidth"></a>Limitar el ancho de banda

1. Abra el complemento MMC de Azure Backup en la máquina que utiliza como el servidor de procesos. De forma predeterminada, hay un acceso directo a Backup en el escritorio o en la siguiente carpeta: C:\Archivos de programa\Microsoft Azure Recovery Services Agent\bin.
2. En el complemento, seleccione **Cambiar propiedades**.

    ![Captura de pantalla de la opción de complemento MMC de Azure Backup para cambiar las propiedades](./media/site-recovery-vmware-to-azure/throttle1.png)
3. En la pestaña **Limitación**, seleccione la opción **Habilitar el límite de uso del ancho de banda de Internet para operaciones de copia de seguridad**. Establezca los límites para las horas laborables y no laborables. Los intervalos válidos van de 512 Kbps a 1023 Mbps.

    ![Captura de pantalla del cuadro de diálogo de propiedades de Azure Backup](./media/site-recovery-vmware-to-azure/throttle2.png)

También puede utilizar el cmdlet [Set-OBMachineSetting](/previous-versions/windows/powershell-scripting/hh770409(v=wps.640)) para establecer la limitación. Este es un ejemplo:

```azurepowershell-interactive
$mon = [System.DayOfWeek]::Monday
$tue = [System.DayOfWeek]::Tuesday
Set-OBMachineSetting -WorkDay $mon, $tue -StartWorkHour "9:00:00" -EndWorkHour "18:00:00" -WorkHourBandwidth  (512*1024) -NonWorkHourBandwidth (2048*1024)
```

**Set-OBMachineSetting -NoThrottle** indica que no se requiere ninguna limitación.

### <a name="alter-the-network-bandwidth-for-a-vm"></a>Modificación del ancho de banda de red para una VM

1. En el Registro de la VM, vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication**.
   * Para modificar el tráfico de ancho de banda en un disco de replicación, modifique el valor de **UploadThreadsPerVM**. Cree la clave en caso de que no exista.
   * Para modificar el ancho de banda para el tráfico de conmutación por recuperación de Azure, modifique el valor de **DownloadThreadsPerVM**.
2. El valor predeterminado para cada clave es **4**. En una red "sobreaprovisionada", se deben cambiar los valores predeterminados de estas claves de registro. El valor máximo que puede usar es **32**. Supervise el tráfico para optimizar el valor.

## <a name="set-up-the-site-recovery-infrastructure-to-protect-more-than-500-vms"></a>Configuración de la infraestructura de Site Recovery para proteger más de 500 máquinas virtuales

Antes de configurar la infraestructura de Site Recovery, acceda al entorno para medir los factores siguientes: las máquinas virtuales compatibles, la tasa de cambios de datos diaria, el ancho de banda de red necesario para el RPO que desea alcanzar, el número de componentes de Site Recovery necesarios y el tiempo requerido para completar la replicación inicial. Complete los pasos siguientes para recopilar la información necesaria:

1. Para medir estos parámetros, ejecute Site Recovery Deployment Planner en su entorno. Para obtener más información, consulte [Información sobre Azure Site Recovery Deployment Planner para VMware en Azure](site-recovery-deployment-planner.md).
2. Implemente un servidor de configuración que cumpla las [recomendaciones de tamaño para el servidor de configuración](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-configuration-server-and-inbuilt-process-server). Si la carga de trabajo de producción supera 650 máquinas virtuales, implemente otro servidor de configuración.
3. Según la tasa de cambios de datos diaria medida, implemente [servidores de procesos de escalabilidad horizontal](vmware-azure-set-up-process-server-scale.md#download-installation-file) con ayuda de las [directrices de tamaño](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-process-server).
4. Si espera que la tasa de cambios de datos para una máquina virtual de disco supere los 2 MBps, asegúrese de usar discos administrados Premium. Site Recovery Deployment Planner se ejecuta durante un período de tiempo específico. Los picos de la tasa de cambio de datos en otros momentos podrían no reflejarse en el informe.
5. [Configure el ancho de banda de red](site-recovery-plan-capacity-vmware.md#control-network-bandwidth) según el RPO que desea conseguir.
6. Cuando la infraestructura esté configurada, habilite la recuperación ante desastres para la carga de trabajo. Para ver cómo hacerlo, consulte [Configuración del entorno de origen para la replicación de VMware en Azure](vmware-azure-set-up-source.md).

## <a name="deploy-additional-process-servers"></a>Implementar servidores de procesos adicionales

Si debe escalar horizontalmente la implementación a más de 200 máquinas de origen o si la tasa de renovación diaria total supera los 2 TB, debe agregar servidores de procesos para controlar el volumen del tráfico. Mejoramos el producto en su versión 9.24 para ofrecerle [alertas del servidor de procesos](vmware-physical-azure-monitor-process-server.md#process-server-alerts) sobre cuándo configurar un servidor de procesos de escalabilidad horizontal. [Configure un servidor de procesos](vmware-azure-set-up-process-server-scale.md) para proteger las nuevas máquinas de origen o [equilibrar las cargas](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load).

### <a name="migrate-machines-to-use-the-new-process-server"></a>Migrar máquinas para utilizar el nuevo servidor de procesos

1. Seleccione **Configuración** > **Site Recovery servers** (Servidores de Site Recovery). Seleccione el servidor de configuración y luego expanda **Servidores de procesos**.

    ![Captura de pantalla del cuadro de diálogo Servidor de procesos](./media/site-recovery-vmware-to-azure/migrate-ps2.png)
2. Haga clic con el botón derecho en el servidor de procesos que se utiliza actualmente y seleccione **Cambiar**.

    ![Captura de pantalla del cuadro de diálogo Servidor de configuración](./media/site-recovery-vmware-to-azure/migrate-ps3.png)
3. En **Seleccionar servidor de procesos de destino**, seleccione el nuevo servidor de procesos que desea usar. A continuación, seleccione las máquinas virtuales que va a controlar el servidor. Seleccione el icono de información para obtener detalles sobre el servidor. Aparece el espacio promedio que se necesita para replicar cada máquina virtual seleccionada en el nuevo servidor de procesos para ayudarlo a tomar decisiones relacionadas con la carga. Seleccione la marca de verificación para empezar a replicar en un nuevo servidor de procesos.

## <a name="deploy-additional-master-target-servers"></a>Implementar servidores de destino maestros adicionales

En los escenarios siguientes, se requiere más de un servidor de destino maestro:

*   Desea proteger una máquina virtual basada en Linux.
*   Si el servidor de destino maestro disponible en el servidor de configuración no tiene acceso al almacén de datos de la máquina virtual.
*   El número total de discos en el servidor de destino maestro (el número de discos locales en el servidor más el número de discos que se van a proteger) es mayor que 60 discos.

Para obtener información sobre cómo agregar un servidor de destino maestro para una máquina virtual basada en Linux, consulte [Instalación de un servidor de destino maestro de Linux para la conmutación por recuperación](vmware-azure-install-linux-master-target.md).

Para agregar un servidor de destino maestro nuevo a una máquina virtual basada en Linux:

1. Vaya a **Almacén de Recovery Services** > **Infraestructura de Site Recovery** > **Servidores de configuración**.
2. Seleccione el servidor de configuración requerido y, a continuación, seleccione **Servidor de destino maestro**.

    ![Captura de pantalla que muestra el botón Agregar servidor de destino maestro](media/site-recovery-plan-capacity-vmware/add-master-target-server.png)
3. Descargue el archivo de instalación unificada y, a continuación, ejecute el archivo en la máquina virtual para configurar el servidor de destino maestro.
4. Seleccione **Instalación del destino principal** > **Siguiente**.

    ![Captura de pantalla que muestra la selección de la opción Instalación del destino principal](media/site-recovery-plan-capacity-vmware/choose-MT.PNG)
5. Seleccione la ubicación de instalación predeterminada y, a continuación, seleccione **Instalar**.

     ![Captura de pantalla que muestra la ubicación de instalación predeterminada](media/site-recovery-plan-capacity-vmware/MT-installation.PNG)
6. Seleccione **Continuar con la configuración** para registrar el destino maestro con el servidor de configuración.

    ![Captura de pantalla que muestra el botón Continuar con la configuración](media/site-recovery-plan-capacity-vmware/MT-proceed-configuration.PNG)
7. Escriba la dirección IP del servidor de configuración y luego escriba la frase de contraseña. Para obtener información sobre cómo generar una frase de contraseña, consulte [Generación de frase de contraseña del servidor de configuración](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase). 

    ![Captura de pantalla que muestra dónde escribir la dirección IP y la frase de contraseña para el servidor de configuración](media/site-recovery-plan-capacity-vmware/cs-ip-passphrase.PNG)
8. Seleccione **Registrar**. Cuando se complete el registro, seleccione **Finalizar**.

Después de registrarse correctamente, este servidor aparece en Azure Portal en **Almacén de Recovery Services** > **Infraestructura de Site Recovery** > **Servidores de configuración**, en los servidores de destino maestros del servidor de configuración.

 > [!NOTE]
 > Descargue la versión más reciente del [archivo de la instalación unificada del servidor de destino maestro para Windows](https://aka.ms/latestmobsvc).

## <a name="next-steps"></a>Pasos siguientes

Descargue y ejecute [Site Recovery Deployment Planner](https://aka.ms/asr-deployment-planner).