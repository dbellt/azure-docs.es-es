---
title: Habilitación de máquinas virtuales de VMware para la recuperación ante desastres con Azure Site Recovery
description: En este artículo, se describe cómo habilitar la replicación de máquinas virtuales de VMware para la recuperación ante desastres mediante el servicio Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: 4c25baba45431e2e7af6b9803337a5af051d75d2
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576100"
---
# <a name="enable-replication-to-azure-for-vmware-vms"></a>Habilitación de máquinas virtuales de VMware en Azure

En este artículo se describe cómo habilitar la replicación de máquinas virtuales (VM) locales de VMware en Azure.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por supuesto que el sistema cumple los siguientes criterios:

- [Configurado el entorno de origen local](vmware-azure-set-up-source.md).
- [Configurado el entorno de destino en Azure](vmware-azure-set-up-target.md).
- [Comprobado los requisitos y requisitos previos](vmware-physical-azure-support-matrix.md) antes de empezar. Aspectos importantes para tener en cuenta:
  - [Sistemas operativos admitidos](vmware-physical-azure-support-matrix.md#replicated-machines) para máquinas replicadas.
  - Compatibilidad con [almacenamiento y disco](vmware-physical-azure-support-matrix.md#storage).
  - [Requisitos de Azure](vmware-physical-azure-support-matrix.md#azure-vm-requirements) con los que deben cumplir las máquinas locales.

### <a name="resolve-common-issues"></a>Resolución de problemas comunes

- Cada disco debe tener menos de 4 TB.
- El disco del sistema operativo debe ser un disco básico, no uno dinámico.
- En el caso de las máquinas virtuales de generación 2 o habilitadas para UEFI, la familia del sistema operativo debe ser Windows y el disco de arranque debe tener menos de 300 GB.

## <a name="before-you-start"></a>Antes de comenzar

A la hora de replicar máquinas virtuales de VMware, tenga en cuenta esta información:

- Su cuenta de usuario de Azure debe tener ciertos [permisos](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) para habilitar la replicación de una nueva máquina virtual en Azure.
- Las máquinas virtuales de VMware se detectan cada 15 minutos. Tras la detección, es posible que las máquinas virtuales tarden 15 minutos o más en aparecer en Azure Portal. Al agregar un servidor de vCenter o un host de vSphere nuevos, la detección podría tardar 15 minutos o más.
- Los cambios en el entorno en la máquina virtual pueden tardar 15 minutos o más en actualizarse en el portal. Por ejemplo, la instalación de las herramientas de VMware.
- Puede comprobar la hora de la última detección de máquinas virtuales de VMware: Vea el campo **Último contacto a las** en la página **Servidores de configuración** del servidor vCenter o el host de vSphere.
- Para agregar máquinas virtuales para la replicación sin esperar a la detección programada, resalte el servidor de configuración (pero no haga clic en él) y seleccione **Actualizar**.
- Al habilitar la replicación, si la máquina virtual está preparada, el servidor de procesos instala automáticamente Mobility Service de Azure Site Recovery en ella.

## <a name="enable-replication"></a>Habilitar replicación

Antes de seguir los pasos de esta sección, revise la siguiente información:

- Azure Site Recovery ahora replica directamente en discos administrados para todas las nuevas replicaciones. El servidor de procesos escribe los registros de replicación en una cuenta de almacenamiento en caché de la región de destino. Estos registros se usan para crear puntos de recuperación en las réplicas de discos administrados que tengan la convención de nomenclatura de `asrseeddisk`.
- La compatibilidad con PowerShell para la replicación en discos administrados está disponible a partir de la [versión 2.0.0 del módulo de Az.RecoveryServices](https://www.powershellgallery.com/packages/Az.RecoveryServices/2.0.0-preview).
- En el momento de la conmutación por error, el punto de recuperación que se selecciona se usa para crear el disco administrado de destino.
- Las máquinas virtuales configuradas previamente para replicar en cuentas de almacenamiento de destino no se ven afectadas.
- La replicación en cuentas de almacenamiento para una nueva máquina virtual solo está disponible a través de una API de transferencia de estado representacional (REST) y PowerShell. Use las versiones 2016-08-10 o 2018-01-10 de la API REST de Azure para la replicación en cuentas de almacenamiento.

Para habilitar la replicación, siga estos pasos:

1. Vaya al **Paso 2: Replicar la aplicación** > **Origen**. Después de habilitar la replicación por primera vez, seleccione **+Replicar** en el almacén para habilitar la replicación de otras máquinas virtuales.
1. En la página **Origen** > **Origen**, seleccione el servidor de configuración.
1. En **Tipo de máquina**, seleccione **Máquinas virtuales** o **Máquinas físicas**.
1. En **vCenter/vSphere Hypervisor**, seleccione el servidor vCenter que administra el host de vSphere, o bien seleccione el host. Este valor no es relevante si va a replicar equipos físicos.
1. Seleccione el servidor de procesos. Si no se han creado otros servidores de procesos, el servidor de procesos integrado del servidor de configuración está disponible en el menú desplegable. El estado de mantenimiento de cada servidor de procesos se indica según los límites recomendados y otros parámetros. Elija un servidor de procesos correcto. No se puede elegir un servidor de procesos [crítico](vmware-physical-azure-monitor-process-server.md#process-server-alerts). Puede [localizar y solucionar](vmware-physical-azure-troubleshoot-process-server.md) los errores **o** configurar un [servidor de procesos de escalabilidad horizontal](vmware-azure-set-up-process-server-scale.md).

   :::image type="content" source="./media/vmware-azure-enable-replication/ps-selection.png" alt-text="Ventana Habilitar replicación: Origen":::

   > [!NOTE]
   > A partir de la [versión 9.24](site-recovery-whats-new.md), se han incorporado alertas adicionales para mejorar las alertas de estado del servidor de procesos. Actualice los componentes de Site Recovery a la versión 9.24 o posteriores para que se generen todas las alertas.

1. En **Destino**, seleccione la suscripción y el grupo de recursos donde quiere crear las máquinas virtuales conmutadas por error. Elija el modelo de implementación que quiera usar en Azure para las máquinas virtuales conmutadas por error.
1. Seleccione la red y la subred de Azure a las que se van a conectar las máquinas virtuales de Azure después de la conmutación por error. La red debe estar en la misma región que el almacén del servicio Site Recovery.

   Seleccione **Configurar ahora para las máquinas seleccionadas** para aplicar la configuración de red a todas las máquinas virtuales que seleccione para protección. Seleccione **Configurar más tarde** para seleccionar la red de Azure por máquina virtual. Si no dispone de una red, debe crear una. Para crear una red mediante Azure Resource Manager, seleccione **Crear nueva**. Seleccione una subred si es aplicable y luego **Aceptar**.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-rep3.png" alt-text="Ventana Habilitar la replicación: Destino":::

1. En **Máquinas virtuales** > **Seleccionar máquinas virtuales**, seleccione cada máquina virtual que quiera replicar. Solo puede seleccionar aquellas máquinas virtuales para las que se pueda habilitar la replicación. Después, seleccione **Aceptar**. Si no puede ver o seleccionar una máquina virtual concreta, vea [La máquina de origen no aparece en Azure Portal](vmware-azure-troubleshoot-replication.md#step-3-troubleshoot-source-machines-that-arent-available-for-replication) para resolver el problema.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-replication5.png" alt-text="Ventana Habilitar la replicación: Seleccionar máquinas virtuales":::

1. En **Propiedades** > **Configurar propiedades**, seleccione la cuenta que usa el servidor de procesos para instalar automáticamente Mobility Service de Site Recovery en la máquina virtual. Además, elija el tipo de disco administrado de destino que se usará para la replicación según los patrones de frecuencia de modificación de los datos.
1. De forma predeterminada, se replican todos los discos de una máquina virtual de origen. Para excluir discos de la replicación, desactive la casilla **Incluir** de aquellos discos que no quiera replicar. Después, seleccione **Aceptar**. Puede establecer propiedades adicionales más adelante. [Obtenga más información](vmware-azure-exclude-disk.md) sobre cómo excluir discos.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-replication6.png" alt-text="Ventana Habilitar la replicación: Configurar propiedades":::

1. En **Configuración de replicación** > **Establecer configuración de replicación**, compruebe que se haya seleccionado la directiva de replicación correcta. Puede modificar la configuración de la directiva de replicación en **Configuración** > **Directivas de replicación** > _nombre de directiva_ > **Editar configuración**. Los cambios aplicados a una directiva también se aplican a las máquinas virtuales nuevas y replicadas.
1. Si quiere recopilar las máquinas virtuales en un grupo de replicación, habilite **Coherencia con múltiples máquinas virtuales**. Especifique un nombre para el grupo y seleccione **Aceptar**.

   > [!NOTE]
   > - Las máquinas virtuales de un grupo de replicación se replican al mismo tiempo y comparten puntos de recuperación coherentes con los bloqueos y coherentes con la aplicación cuando conmutan por error.
   > - Recopile las máquinas virtuales y los servidores físicos juntos para que reflejen las cargas de trabajo. La habilitación de la coherencia entre varias VM puede afectar al rendimiento de la carga de trabajo. Hágalo únicamente si las máquinas virtuales ejecutan la misma carga de trabajo y necesita coherencia.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-replication7.png" alt-text="Ventana Habilitar la replicación":::

1. Seleccione **Habilitar replicación**. Puede hacer un seguimiento del progreso del trabajo **Habilitar protección** en **Configuración** > **Trabajos** > **Trabajos de Site Recovery**. La máquina virtual estará preparada para la conmutación por error después de que finalice el trabajo de **Finalizar protección**.

## <a name="monitor-initial-replication"></a>Supervisión de la replicación inicial

Una vez que se han completado en el elemento protegido todos los pasos indicados en el apartado "Habilitación de la replicación", Azure Site Recovery inicia la replicación (que es sinónimo de sincronización) de los datos de la máquina de origen en la región de destino. En este periodo, se crean replicas de los discos de origen. Los cambios diferenciales no se copian en la región de destino hasta que haya finalizado de la copia de los discos originales. El tiempo necesario para copiar los discos originales depende de varios parámetros, como:

- El tamaño de los discos de la máquina de origen.
- El ancho de banda disponible para transferir los datos a Azure (se puede sacar provecho de Deployment Planner para identificar el ancho de banda óptimo necesario).
- Los recursos del servidor de procesos, como la memoria, el espacio libre en disco, la CPU disponible para almacenar en caché y procesar los datos recibidos de elementos protegidos (asegúrese de que el servidor de procesos está en [buen estado](vmware-physical-azure-monitor-process-server.md#monitor-proactively)).

Para realizar un seguimiento del progreso de la replicación inicial, vaya al almacén de Recovery Services en Azure Portal > elementos replicados-> supervisar el valor de la columna "Estado" del elemento replicado. El estado muestra el porcentaje de finalización de la replicación inicial. Al mantener el puntero sobre la columna de estado, estará disponible el "total de datos transferidos". Al hacer clic en el estado, se abre una página contextual y se muestran los siguientes parámetros:

- Last refreshed at (Última actualización): indica la última hora a la que el servicio ha actualizado la información de replicación de toda la máquina.
- Completed percentage (Porcentaje de finalización): indica el porcentaje de la replicación inicial completado en la máquina virtual.
- Total data transferred (Total de datos transferidos): cantidad de datos transferidos desde la máquina virtual a Azure

    :::image type="content" source="media/vmware-azure-enable-replication/initial-replication-state.png" alt-text="Estado de la replicación" lightbox="media/vmware-azure-enable-replication/initial-replication-state.png":::

- Progreso de la sincronización (para realizar un seguimiento de los detalles a nivel de disco)
    - Estado de la replicación
      - Si la replicación aún no se ha iniciado, el estado se actualiza, pasa a ser "In queue" (En cola). Durante la replicación inicial, solo se replican tres discos a la vez. Este mecanismo se sigue para evitar la limitación en el servidor de procesos.
      - Una vez iniciada la replicación, el estado se actualiza, pasa a ser "In progress" (En curso).
      - Después de la finalización de la replicación inicial, el estado se marca como "Complete" (Completo).        
   - Site Recovery lee el disco original, transfiere los datos a Azure y captura el progreso en el nivel de disco. Tenga en cuenta que Site Recovery omite la replicación del tamaño no ocupado del disco y lo agrega a los datos completados. Por consiguiente, es posible que la suma de los datos transferidos de todos los discos no se agregue hasta el "total de datos transferidos" en el nivel de máquina virtual.
   - Al hacer clic en el globo de información de un disco, se puede saber cuándo se desencadenó la replicación (sinónimo de sincronización) en el disco, los datos transferidos a Azure en los últimos 15 min y la última marca de tiempo que se ha actualizado. Esta marca de tiempo indica la última hora a la que el servicio de Azure recibió información desde la máquina de origen :::image type="content" source="media/vmware-azure-enable-replication/initial-replication-info-balloon.png" alt-text="Datos que muestra el globo de información de la replicación inicial" lightbox="media/vmware-azure-enable-replication/initial-replication-info-balloon.png":::.
   - Se muestra el estado de cada disco
      - Si la replicación es más lenta de lo esperado, el disco pasa al estado de advertencia
      - Si la replicación no progresa, el estado del disco cambia a crítico

Si el estado es de advertencia o crítico, asegúrese de que tanto el estado de replicación de la máquina como el [servidor de procesos](vmware-physical-azure-monitor-process-server.md) sean correctos. 

En cuanto se complete la habilitación del trabajo de replicación, el progreso de la replicación sería del 0 % y el valor del total de datos transferidos sería NA (no disponible). Al hacer clic, el valor de los datos de cada disco identificado sería "NA" (no disponible). Esto indica que la replicación aún no se ha iniciado y Azure Site Recovery aún tiene que recibir las estadísticas más recientes. El progreso se actualiza cada 30 minutos.

> [!NOTE]
> Asegúrese de actualizar los servidores de configuración, los servidores de procesos de escalabilidad horizontal y los agentes de movilidad a la versión 9.36, o posteriores, para asegurarse de que se captura el progreso preciso y se envía a los servicios de Site Recovery.


## <a name="view-and-manage-vm-properties"></a>Visualización y administración de las propiedades de la máquina virtual

Después, compruebe las propiedades de la máquina virtual de origen. Recuerde que el nombre de la máquina virtual de Azure debe cumplir los [requisitos para las máquinas virtuales de Azure](vmware-physical-azure-support-matrix.md#replicated-machines).

1. Vaya a **Configuración** > **Elementos replicados** y seleccione la máquina virtual. En la página **Información esencial** se muestra información sobre la configuración y el estado de las máquinas virtuales.
1. En **Propiedades** puede ver la información de replicación y conmutación por error de la máquina virtual.
1. En **Proceso y red** > **Propiedades de Compute**, puede cambiar varias propiedades de la máquina virtual.

   :::image type="content" source="./media/vmware-azure-enable-replication/vmproperties.png" alt-text="Ventana Propiedades de proceso y red":::

   - **Nombre de la máquina virtual de Azure**: modifique el nombre en caso necesario para que cumpla los requisitos de Azure.
   - **Tamaño de la VM de destino o Tipo de máquina virtual**: el tamaño predeterminado de la máquina virtual se elige según unos parámetros que incluyen el número de discos, el número de tarjetas NIC, el recuento de núcleos de CPU, la memoria y los tamaños de rol de máquina virtual disponibles en la región de Azure de destino. Azure Site Recovery elige el primer tamaño de máquina virtual disponible que satisface todos los criterios. Puede seleccionar otro tamaño de máquina virtual según sea necesario en cualquier momento anterior a la conmutación por error. El tamaño del disco de la máquina virtual también se basa en el tamaño del disco de origen y solo puede modificarse después de la conmutación por error. Obtenga más información sobre los tamaños de disco y las tasas de IOPS en [Objetivos de escalabilidad y rendimiento para discos de máquinas virtuales](../virtual-machines/disks-scalability-targets.md).
   - **Grupo de recursos**: puede seleccionar un [grupo de recursos](../azure-resource-manager/management/overview.md#resource-groups) desde el que la máquina virtual forme parte de una conmutación por error posterior. Puede cambiar este valor en cualquier momento antes de la conmutación por error. Después de la conmutación por error, si se migra la máquina virtual a otro grupo de recursos, la configuración de protección de dicha máquina virtual se interrumpe.
   - **Conjunto de disponibilidad**: puede seleccionar un [conjunto de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md) si la máquina debe formar parte de una conmutación por error posterior. Al seleccionar un conjunto de disponibilidad, tenga en cuenta la siguiente información:
     - Solo se muestran los conjuntos de disponibilidad que pertenecen al grupo de recursos especificado.
     - Las máquinas virtuales de distintas redes virtuales no pueden formar parte del mismo conjunto de disponibilidad.
     - Solo las máquinas virtuales del mismo tamaño pueden formar parte de un conjunto de disponibilidad.

1. También puede agregar información sobre la red, la subred y la dirección IP de destino asignadas a la máquina virtual de Azure.
1. En **Discos** puede ver los discos de datos y del sistema operativo en la máquina virtual que se va a replicar.

### <a name="configure-networks-and-ip-addresses"></a>Configuración de redes y direcciones IP

Puede establecer la dirección IP de destino:

- Si no proporciona ninguna dirección, la máquina virtual conmutada por error usa DHCP.
- Si establece una dirección que no esté disponible en el momento de la conmutación por error, esta operación no funcionará.
- Si la dirección está disponible en la red de conmutación por error de prueba, se puede usar la misma dirección IP de destino para la conmutación por error de prueba.

El número de adaptadores de red viene determinado por el tamaño que se especifique para la máquina virtual de destino, de la siguiente manera:

- Si el número de adaptadores de red en la máquina virtual de origen es menor o igual que el número de adaptadores permitido para el tamaño de la máquina virtual de destino, el destino tiene el mismo número de adaptadores que el origen.
- Si el número de adaptadores de la máquina virtual de origen supera el número permitido para el tamaño de la máquina virtual de destino, se usa el tamaño máximo de destino. Por ejemplo, si una máquina virtual de origen tiene dos adaptadores de red y el tamaño de la máquina virtual de destino admite cuatro, la máquina virtual de destino tiene dos adaptadores. Si la máquina virtual de origen tiene dos adaptadores pero el tamaño de destino solo admite uno, la máquina virtual de destino tiene solo un adaptador.
- Si la máquina virtual tiene varios adaptadores de red, todos ellos se conectan a la misma red. Además, el primero de ellos que se muestra en la lista se convierte en el adaptador de red predeterminado en la máquina virtual de Azure.

### <a name="azure-hybrid-benefit"></a>Ventaja híbrida de Azure

Los clientes de Microsoft Software Assurance pueden usar la Ventaja híbrida de Azure para ahorrar en los costos de licencia de equipos Windows Server que se migran a Azure. La ventaja también se aplica a la recuperación ante desastres de Azure. Si es beneficiario, puede asignar la ventaja a la máquina virtual que Site Recovery crea si se produce una conmutación por error.

1. Vaya a **Propiedades de proceso y red** en la máquina virtual replicada.
1. Responda cuando se le pregunte si tiene una licencia de Windows Server que le convierta en beneficiario de la Ventaja híbrida de Azure.
1. Confirme que tiene una licencia de Windows Server con Software Assurance que puede usar para aplicar la ventaja a la máquina virtual que se va a crear en la conmutación por error.
1. Guarde la configuración de la máquina virtual replicada.

[Más información](https://azure.microsoft.com/pricing/hybrid-benefit/) sobre la Ventaja híbrida de Azure.

## <a name="next-steps"></a>Pasos siguientes

Una vez que la máquina virtual alcance un estado protegido, pruebe una [conmutación por error](site-recovery-failover.md) para comprobar si la aplicación aparece en Azure.

- [Más información](site-recovery-manage-registration-and-protection.md) sobre cómo borrar el registro y la configuración de protección para deshabilitar la replicación.
- [Más información](vmware-azure-disaster-recovery-powershell.md) sobre cómo automatizar la replicación de las máquinas virtuales con Powershell.
