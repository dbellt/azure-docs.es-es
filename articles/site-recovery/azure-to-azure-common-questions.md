---
title: Preguntas comunes sobre la recuperación ante desastres de máquinas virtuales de Azure con Azure Site Recovery
description: En este artículo se responde a preguntas comunes sobre la recuperación ante desastres de máquinas virtuales de Azure cuando se usa Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.date: 11/03/2019
ms.topic: conceptual
ms.openlocfilehash: 5309fd60640c45ade42bab4c5727cf1f0a8d9d70
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105025482"
---
# <a name="common-questions-azure-to-azure-disaster-recovery"></a>Preguntas frecuentes: Recuperación ante desastres de Azure a Azure

En este artículo se responde a preguntas frecuentes sobre la recuperación ante desastres de máquinas virtuales de Azure a otra región de Azure mediante el servicio [Azure Site Recovery](site-recovery-overview.md).

## <a name="general"></a>General

### <a name="how-is-site-recovery-priced"></a>¿Cómo se fija el precio de Site Recovery?

Conozca los [costos](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/) de la recuperación ante desastres de máquinas virtuales de Azure.

### <a name="how-does-the-free-tier-work"></a>¿Cómo funciona el nivel gratuito?

Todas las instancias protegidas con Site Recovery son gratuitas los primeros 31 días de protección. Después de ese período, la protección de cada instancia se cobra según las tarifas resumidas en los [detalles de precios](https://azure.microsoft.com/pricing/details/site-recovery/). Para estimar los costos, use la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/?service=site-recovery).

### <a name="do-i-incur-other-azure-charges-in-the-first-31-days"></a>Durante los primeros 31 días, ¿puedo incurrir en otros cargos de Azure?

Sí. Aunque Azure Site Recovery sea gratuito los primeros 31 días de una instancia protegida, se pueden generar cargos por Azure Storage, transacciones de almacenamiento y transferencia de datos. Una máquina virtual recuperada también puede incurrir en cargos por proceso de Azure. G

### <a name="how-do-i-get-started-with-azure-vm-disaster-recovery"></a>¿Cómo puedo empezar a usar la recuperación ante desastres de máquinas virtuales de Azure?

1. [Comprenda](azure-to-azure-architecture.md) la arquitectura de recuperación ante desastres de máquinas virtuales de Azure.
2. [Revise](azure-to-azure-support-matrix.md) los requisitos de compatibilidad.
3. [Configure](azure-to-azure-how-to-enable-replication.md) la recuperación ante desastres en máquinas virtuales de Azure
4. [Ejecute un simulacro de recuperación ante desastres](azure-to-azure-tutorial-dr-drill.md) con una conmutación por error de prueba.
5. [Ejecute una conmutación por error completa](azure-to-azure-tutorial-failover-failback.md) a una región secundaria de Azure.
6. [Realice una conmutación por recuperación](azure-to-azure-tutorial-failback.md) de la región secundaria a la primaria.

### <a name="how-do-we-ensure-capacity-in-the-target-region"></a>¿Cómo se garantiza la capacidad en la región de destino?

El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias de máquina virtual que están protegidas mediante Site Recovery se implementan en la región de destino.

## <a name="replication"></a>Replicación

### <a name="can-i-replicate-vms-with-disk-encryption"></a>¿Puedo replicar máquinas virtuales con cifrado de discos?

Sí. Site Recovery admite la recuperación ante desastres de máquinas virtuales con Azure Disk Encryption (ADE) habilitado. Cuando está habilitada la replicación, Azure copia todos los secretos y claves de cifrado de disco necesarios de la región de origen a la de destino en el contexto del usuario. Si no tiene los permisos necesarios, el administrador de seguridad puede usar un script para copiarlos.

- Site Recovery admite ADE para las máquinas virtuales de Azure que ejecutan Windows.
- Site Recovery admite:
    - ADE versión 0.1, que tiene un esquema que requiere Azure Active Directory (Azure AD).
    - ADE versión 1.1, que no requiere Azure AD. En el caso de la versión 1.1, las máquinas virtuales de Azure en Windows deben tener discos administrados.
    - [Más información](../virtual-machines/extensions/azure-disk-enc-windows.md#extension-schema). Acerca de los esquemas de extensión.

[Más información](azure-to-azure-how-to-enable-replication-ade-vms.md) sobre la habilitación de la replicación para máquinas virtuales cifradas.

### <a name="can-i-select-an-automation-account-from-a-different-resource-group"></a>¿Se puede seleccionar una cuenta de automatización de otro grupo de recursos?

Cuando permite que Site Recovery administre las actualizaciones de la extensión del servicio Mobility que se ejecuta en máquinas virtuales de Azure replicadas, se implementa un runbook global (que usan los servicios de Azure) a través de una cuenta de automatización de Azure. Puede usar la cuenta de automatización que Site Recovery crea o seleccionar usar una cuenta de automatización existente. 

Actualmente, en el portal, solo puede seleccionar una cuenta de automatización del mismo grupo de recursos que el almacén. Sin embargo, puede seleccionar una cuenta de automatización de otro grupo de recursos mediante PowerShell. [Más información](azure-to-azure-autoupdate.md#enable-automatic-updates).

### <a name="if-i-use-a-customer-automation-account-thats-not-in-the-vault-resource-group-can-i-delete-the-default-runbook"></a>Si uso la cuenta de automatización de un cliente que no está en el grupo de recursos del almacén, ¿puedo eliminar el runbook predeterminado?

Sí, puede eliminarlo si no lo necesita. 

### <a name="can-i-replicate-vms-to-another-subscription"></a>¿Se pueden replicar máquinas virtuales en otra suscripción?

Sí, puede replicar las máquinas virtuales de Azure en otra suscripción que esté en el mismo inquilino de Azure AD. Al habilitar la recuperación ante desastres en las máquinas virtuales, la suscripción de destino que se muestra es la de la máquina virtual de origen de forma predeterminada. Puede modificar la suscripción de destino, y el resto de opciones (como el grupo de recursos y la red virtual) se rellenan automáticamente a partir de la suscripción seleccionada.

### <a name="can-i-replicate-vms-in-an-availability-zone-to-another-region"></a>¿Puedo replicar las máquinas virtuales de una zona de disponibilidad en otra región?

Sí, puede replicar las máquinas virtuales de las zonas de disponibilidad en otra región de Azure. 

### <a name="can-i-replicate-non-zone-vms-to-a-zone-within-the-same-region"></a>¿Puedo replicar las máquinas virtuales que no son de zona en una zona dentro de la misma región? 

No existe esta posibilidad en el portal. Para ello, puede usar la API REST o PowerShell.

### <a name="can-i-replicate-zoned-vms-to-a-different-zone-in-the-same-region"></a>¿Puedo replicar máquinas virtuales con zonas en una zona diferente de la misma región?

La compatibilidad con esta posibilidad está limitada a unas pocas regiones. [Más información](azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery.md).

### <a name="can-i-exclude-disks-from-replication"></a>¿Se pueden excluir discos de la replicación?

Sí, puede excluir discos al configurar la replicación, mediante PowerShell. [Más información](azure-to-azure-exclude-disks.md).

### <a name="can-i-replicate-new-disks-added-to-replicated-vms"></a>¿Puedo replicar nuevos discos agregados a las máquinas virtuales replicadas?

En el caso de las máquinas virtuales replicadas con discos administrados, puede agregar nuevos discos y habilitar la replicación en ellos. Cuando se agrega un nuevo disco, la máquina virtual replicada muestra un mensaje de advertencia que indica que uno o más discos de la máquina virtual están disponibles para la protección. 

- Si habilita la replicación para los discos agregados, la advertencia desaparece después de la replicación inicial.
- Si no quiere permitir la replicación en el disco, puede hacer caso omiso de la advertencia.
- Si realiza la conmutación por error de una máquina virtual con discos agregados, los puntos de replicación muestran los discos disponibles para la recuperación. Por ejemplo, si agrega un segundo disco a una máquina virtual que tiene uno, el punto de replicación creado antes de agregarlo se muestra como "1 de 2 discos".

Site Recovery no admite la eliminación en caliente de un disco de una máquina virtual replicada. Si quita un disco de la máquina virtual, deberá deshabilitar y luego volver a habilitar la replicación en ella.

### <a name="how-often-can-i-replicate-to-azure"></a>¿Con qué frecuencia se puede replicar en Azure?

La replicación es continua cuando se replican máquinas virtuales de Azure en otra región de Azure. [Más información](./azure-to-azure-architecture.md#replication-process) sobre cómo funciona la replicación.

### <a name="can-i-replicate-virtual-machines-within-a-region"></a>¿Se pueden replicar máquinas virtuales de una misma región? 

No puede usar Site Recovery para replicar discos dentro de una región.

### <a name="can-i-replicate-vm-instances-to-any-azure-region"></a>¿Se pueden replicar instancias de máquina virtual en cualquier región de Azure?

Puede replicar y recuperar máquinas virtuales entre dos regiones cualesquiera dentro del mismo clúster geográfico. Los clústeres geográficos se definen teniendo presente la latencia y la soberanía de datos. [Más información](./azure-to-azure-support-matrix.md#region-support) sobre la compatibilidad de regiones.

### <a name="does-site-recovery-need-internet-connectivity"></a>¿Site Recovery requiere conectividad a Internet?

No, pero las máquinas virtuales necesitan acceso a las direcciones URL y los intervalos IP de Site Recovery. [Más información](./azure-to-azure-about-networking.md#outbound-connectivity-for-urls).

### <a name="can-i-replicate-an-application-tiered-across-resource-groups"></a>¿Puedo replicar una aplicación en capas entre grupos de recursos?

Sí, puede replicar la aplicación y mantener la configuración de recuperación ante desastres en un grupo de recursos distinto.

Por ejemplo, si las aplicaciones tienen tres niveles (aplicación, base de datos y web) en distintos grupos de recursos, debe habilitar la replicación tres veces para proteger todos los niveles. Site Recovery replicará estos tres niveles en tres grupos de recursos diferentes.

### <a name="can-i-move-storage-accounts-across-resource-groups"></a>¿Puedo mover cuentas de almacenamiento entre grupos de recursos?

No, esta posibilidad no se admite. Si mueve por accidente cuentas de almacenamiento a otro grupo de recursos y elimina el grupo de recursos original, puede crear un grupo de recursos con el mismo nombre que el antiguo y luego mover la cuenta de almacenamiento a este grupo de recursos.

## <a name="replication-policy"></a>Directiva de replicación

### <a name="what-is-a-replication-policy"></a>¿Qué es una directiva de replicación?

Una directiva de replicación define el historial de retención de los puntos de recuperación y la frecuencia de instantáneas coherentes con la aplicación.  Site Recovery crea una directiva de replicación predeterminada de la siguiente manera:

- Conserva los puntos de recuperación durante 24 horas.
- Toma instantáneas coherentes con la aplicación cada cuatro horas.

[Más información](azure-to-azure-how-to-enable-replication.md#customize-target-resources) sobre la configuración de replicación.

### <a name="whats-a-crash-consistent-recovery-point"></a>¿Qué es un punto de recuperación coherente con los bloqueos?

Un punto de recuperación coherente con los bloqueos contiene datos en disco, como si tirara del cable de alimentación del servidor durante la instantánea. No incluye nada de lo que había en memoria cuando se tomó la instantánea.

Hoy en día, la mayoría de las aplicaciones se pueden recuperar bien a partir de instantáneas coherentes con los bloqueos. Un punto de recuperación coherente con los bloqueos suele ser suficiente para sistemas operativos que no son de base de datos y para aplicaciones como servidores de archivos, servidores DHCP y servidores de impresión.

Site Recovery crea automáticamente un punto de recuperación coherente con los bloqueos cada cinco minutos.

### <a name="whats-an-application-consistent-recovery-point"></a>¿Qué es un punto de recuperación coherente con la aplicación?

Se crean puntos de recuperación coherentes con la aplicación a partir de instantáneas coherentes con la aplicación. Estos puntos de recuperación capturan los mismos datos que las instantáneas coherentes con los bloqueos y, además, capturan los datos en memoria y todas las transacciones en proceso.

Debido a su contenido adicional, las instantáneas coherentes con la aplicación son las más complejas y las que más tardan. Se recomiendan puntos de recuperación coherentes con la aplicación para sistemas operativos de base de datos y aplicaciones como SQL Server. En Windows, las instantáneas coherentes con la aplicación usan el Servicio de instantáneas de volumen (VSS).

### <a name="do-app-consistent-recovery-points-impact-performance"></a>¿Los puntos de recuperación coherentes con la aplicación afectan al rendimiento?

 Dado que los puntos de recuperación coherentes con la aplicación capturan todos los datos en memoria y proceso, si se capturan con frecuencia, el rendimiento puede verse afectado cuando la carga de trabajo ya está ocupada. No se recomienda capturarlos con demasiada frecuencia en el caso de cargas de trabajo que no son de base de datos. Incluso con cargas de trabajo de base de datos, una frecuencia de una hora será suficiente.

### <a name="whats-the-minimum-frequency-for-generating-app-consistent-recovery-points"></a>¿Cuál es la frecuencia mínima para generar puntos de recuperación coherentes con la aplicación?

Site Recovery puede crear puntos de recuperación coherentes con la aplicación con una frecuencia mínima de una hora.

### <a name="can-i-enable-app-consistent-replication-for-linux-vms"></a>¿Puedo habilitar la replicación coherente con la aplicación en máquinas virtuales Linux?

Sí. El agente de movilidad para Linux admite scripts personalizados para la coherencia con la aplicación. El agente utiliza un script personalizado con opciones anteriores y posteriores. [Más información](site-recovery-faq.md#can-i-enable-replication-with-app-consistency-in-linux-servers)

### <a name="how-are-recovery-points-generated-and-saved"></a>¿Cómo se generan y guardan los puntos de recuperación?

Para entender cómo Site Recovery genera puntos de recuperación, vamos a usar un ejemplo. 

- Una directiva de replicación retiene los puntos de recuperación durante 24 horas y toma una instantánea de frecuencia coherente con la aplicación cada hora.
- Site Recovery crea un punto de recuperación coherente con los bloqueos cada cinco minutos. No se puede cambiar esta frecuencia.
- Site Recovery elimina los puntos de recuperación después de una hora, lo que supone un ahorro de un punto por hora.

Por lo tanto, en la última hora, puede elegir entre 12 puntos coherentes con los bloqueos y un punto coherente con la aplicación, como se muestra en el gráfico.

   ![Lista de puntos de recuperación generados](./media/azure-to-azure-common-questions/recovery-points.png)

### <a name="how-far-back-can-i-recover"></a>¿Hasta cuánto tiempo atrás puedo recuperar?

El punto de recuperación más antiguo que puede usar es de 72 horas.

### <a name="what-happens-if-site-recovery-cant-generate-recovery-points-for-more-than-24-hours"></a>¿Qué ocurre si Site Recovery no puede generar puntos de recuperación durante más de 24 horas? 

Si tiene una directiva de replicación de 24 horas y Site Recovery no puede generar puntos de recuperación durante un número de horas superior, los puntos de recuperación antiguos permanecen. Site Recovery solo reemplaza el punto más antiguo si genera nuevos puntos. Hasta que no haya nuevos puntos de recuperación, todos los puntos anteriores permanecerán después de alcanzar la ventana de retención.

### <a name="can-i-change-the-replication-policy-after-replication-is-enabled"></a>¿Puedo cambiar la directiva de replicación después de que se ha habilitado la replicación?

Sí. En el almacén > **Infraestructura de Site Recovery** > **Directivas de replicación**, seleccione y edite la directiva. Los cambios también se aplican a las directivas existentes.

### <a name="are-all-recovery-points-a-complete-vm-copy"></a>¿Son todos los puntos de recuperación una copia completa de una máquina virtual?

El primer punto de recuperación que se genera tiene la copia completa. Los puntos de recuperación sucesivos tienen los cambios diferenciales.

### <a name="do-increases-in-recovery-point-retention-increase-storage-costs"></a>¿El aumento en la retención de los puntos de recuperación aumenta los costos de almacenamiento?

Sí. Por ejemplo, si aumenta la retención de 24 a 72 horas, Site Recovery guarda puntos de recuperación durante otras 48 horas. La hora agregada genera cambios de almacenamiento. Por ejemplo, si un único punto de recuperación tuviera cambios diferenciales de 10 GB, con un costo por GB de 0,16 USD al mes, los cargos adicionales serían de $1,60 USD × 48 al mes.

## <a name="multi-vm-consistency"></a>Coherencia con múltiples máquinas virtuales

### <a name="what-is-multi-vm-consistency"></a>¿Qué es la coherencia de múltiples máquinas virtuales?

La coherencia con múltiples máquinas virtuales garantiza que los puntos de recuperación sean coherentes entre todas las máquinas virtuales replicadas.

- Cuando se habilita la coherencia con varias máquinas virtuales, Site Recovery crea un grupo de replicación de todas las máquinas con la opción habilitada. 
- Al conmutar por error las máquinas del grupo de replicación, tienen puntos de recuperación compartidos coherentes con los bloqueos y coherentes con la aplicación.

[Aprenda](azure-to-azure-tutorial-enable-replication.md#enable-replication) cómo habilitar la coherencia con múltiples máquinas virtuales.

### <a name="can-i-fail-over-a-single-vm-in-a-replication-group"></a>¿Puedo conmutar por error una sola máquina virtual de un grupo de replicación?

No. Cuando se habilita la coherencia con múltiples máquinas virtuales, se deduce que una aplicación tiene una dependencia de todas las máquinas virtuales del grupo de replicación y no se permite la conmutación por error de una sola.

### <a name="how-many-vm-can-i-replicate-together-in-a-group"></a>¿Cuántas máquinas virtuales se pueden replicar juntas en un grupo?

Puede replicar 16 máquinas virtuales juntas en un grupo de replicación.

### <a name="when-should-i-enable-multi-vm-consistency"></a>¿Cuándo se debe habilitar la coherencia de múltiples máquinas virtuales?

Dado que la coherencia con múltiples máquinas virtuales consume mucha CPU, su habilitación puede afectar al rendimiento de la carga de trabajo. Habilítela solamente si las máquinas ejecutan la misma carga de trabajo y necesita coherencia entre varias máquinas. Por ejemplo, si tiene dos instancias de SQL Server y dos servidores web en una aplicación, habilite la coherencia con múltiples máquinas virtuales solo para las instancias de SQL Server.

### <a name="can-i-add-a-replicating-vm-to-a-replication-group"></a>¿Puedo agregar una máquina virtual de replicación a un grupo de replicación?

Al habilitar la replicación en una máquina virtual, puede agregarla a un nuevo grupo de replicación o a un grupo ya existente. No se puede agregar a un grupo una máquina virtual que ya se esté replicando. 
 
## <a name="failover"></a>Conmutación por error

### <a name="how-do-we-ensure-capacity-in-the-target-region"></a>¿Cómo se garantiza la capacidad en la región de destino?

El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias que están protegidas mediante Site Recovery se puedan implementar en la región de destino.

### <a name="is-failover-automatic"></a>¿La conmutación por error es automática?

La conmutación por error no es automática. Puede iniciar una conmutación por error con solo un clic en el portal o bien usar [PowerShell](azure-to-azure-powershell.md) para desencadenarla.

### <a name="can-i-keep-a-public-ip-address-after--failover"></a>¿Puedo conservar una dirección IP pública después de la conmutación por error?

No puede conservar la dirección IP pública de una aplicación de producción después de una conmutación por error.

Al incorporar una carga de trabajo como parte del proceso de conmutación por error, debe asignarle un recurso de dirección IP pública de Azure. El recurso debe estar disponible en la región de destino. Puede asignar el recurso de dirección IP pública manualmente o automatizarlo con un plan de recuperación. [Aprenda](concepts-public-ip-address-with-site-recovery.md#public-ip-address-assignment-using-recovery-plan) a configurar direcciones IP públicas tras la conmutación por error.

### <a name="can-i-keep-a-private-ip-address-after-failover"></a>¿Puedo conservar una dirección IP privada después de una conmutación por error?

Sí. De forma predeterminada, cuando se habilita la recuperación ante desastres en las máquinas virtuales de Azure, Site Recovery crea recursos de destino en función de la configuración de los recursos de origen. En el caso de las máquinas virtuales de Azure configuradas con direcciones IP estáticas, Site Recovery trata de aprovisionar la misma dirección IP para la máquina virtual de destino, si no está en uso.
[Más información](site-recovery-retain-ip-azure-vm-failover.md) sobre la conservación de direcciones IP después de una conmutación por error.

### <a name="why-is-a-vm-assigned-a-new-ip-address-after-failover"></a>¿Por qué se asigna una nueva dirección IP a una máquina virtual después de la conmutación por error?

Site Recovery trata de proporcionar la dirección IP en el momento de la conmutación por error. Si otra máquina usa esa dirección, Site Recovery establece la siguiente dirección IP disponible como destino.

[Más información sobre](azure-to-azure-network-mapping.md#set-up-ip-addressing-for-target-vms) cómo configurar la asignación de red y el direccionamiento IP para redes virtuales.

### <a name="whats-the-latest-recovery-point"></a>¿Qué es el punto de recuperación *más reciente*?

La opción de punto de recuperación *Más reciente (RPO más bajo)* funciona de la manera siguiente:

1. Procesa primero todos los datos que se han enviado a Site Recovery.
2. Después de que el servicio procesa los datos, se crea un punto de recuperación para cada máquina virtual antes de realizar la conmutación por error a ellas. Esta opción proporciona el objetivo de punto de recuperación (RPO) más bajo.
3. La máquina virtual creada después de la conmutación por error tiene todos los datos replicados en Site Recovery, desde el momento en que se desencadenó este proceso.

### <a name="do-latest-recovery-points-impact-failover-rto"></a>¿Afectan los puntos de recuperación *más recientes* al RTO de la conmutación por error?

Sí. Site Recovery procesa todos los datos pendientes antes de la conmutación por error, por lo que esta opción tiene un objetivo de tiempo de recuperación (RTO) mayor en comparación con otras opciones.

### <a name="whats-the-latest-processed-recovery-option"></a>¿Qué es la opción de recuperación *Procesado más recientemente*?

La opción *Procesado más recientemente* funciona de la siguiente manera:

1. Las máquinas virtuales se conmutan por error al último punto de recuperación procesado por Site Recovery. Esta opción proporciona un objetivo de tiempo de recuperación bajo, ya que no se invierte tiempo en el procesamiento de datos sin procesar.

### <a name="what-if-theres-an-unexpected-outage-in-the-primary-region"></a>¿Qué ocurre si hay una interrupción inesperada en la región primaria?

Puede iniciar la conmutación por error. Site Recovery no necesita conectividad desde la región primaria para realizar la conmutación por error.

### <a name="what-is-the-rto-of-a-vm-failover"></a>¿Qué es el RTO de la conmutación por error de una máquina virtual?

Site Recovery tiene un Acuerdo de Nivel de Servicio de RTO de [dos horas](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/). La mayoría de las veces, Site Recovery conmuta por error las máquinas virtuales en cuestión de minutos. Para calcular el RTO, revise el trabajo de conmutación por error, que muestra el tiempo que ha tardado en aparecer una máquina virtual. 

## <a name="recovery-plans"></a>Planes de recuperación

### <a name="whats-a-recovery-plan"></a>¿Qué es un plan de recuperación?

Un [plan de recuperación](site-recovery-create-recovery-plans.md) en Site Recovery coordina la conmutación por error y la recuperación de máquinas virtuales. Ayuda a que la recuperación sea siempre precisa, repetible y automatizada. Realiza las operaciones siguientes:

- Define un grupo de máquinas virtuales que se conmutan por error juntas.
- Define las dependencias entre las máquinas virtuales, de forma que la aplicación aparezca de forma precisa.
- Automatiza la recuperación, con la opción de acciones manuales personalizadas para tareas distintas de la conmutación por error de la máquina virtual. 


### <a name="how-does-sequencing-work"></a>¿Cómo funciona la secuenciación?

En un plan de recuperación, puede crear varios grupos de máquinas virtuales para lograr la secuenciación. Los grupos conmutan por error de uno en uno, de modo que las máquinas virtuales que forman parte del mismo grupo se conmutan por error juntas. [Más información](recovery-plan-overview.md#model-apps).

### <a name="how-can-i-find-the-rto-of-a-recovery-plan"></a>¿Cómo se puede encontrar el RTO de un plan de recuperación?

Para comprobar el RTO de un plan de recuperación, haga una conmutación por error de prueba en el plan de recuperación. En **Trabajos de Site Recovery**, compruebe la duración de la conmutación por error de prueba. En la captura de pantalla de ejemplo, el trabajo de conmutación por error de prueba **SAPTestRecoveryPlan** tardó 8 minutos y 59 segundos.

![Enumeración de los trabajos con la duración de la conmutación por error de prueba para el RTO](./media/azure-to-azure-common-questions/recovery-plan-rto.png)

### <a name="can-i-add-automation-runbooks-to-recovery-plans"></a>¿Puedo agregar runbooks de automatización a los planes de recuperación?

Sí. [Más información](site-recovery-runbook-automation.md).

## <a name="reprotection-and-failback"></a>Reprotección y conmutación por recuperación

### <a name="after-failover-are-vms-in-the-secondary-region-protected-automatically"></a>Después de la conmutación por error, ¿las máquinas virtuales de la región secundaria están protegidas automáticamente? 

No. Al conmutar por error las máquinas virtuales de una región a otra, estas se inician en la región de recuperación ante desastres de destino con un estado desprotegido. Para [volver a proteger](./azure-to-azure-how-to-reprotect.md) las máquinas virtuales de la región secundaria, habilite de nuevo la replicación en la región primaria.

### <a name="when-i-reprotect-is-all-data-replicated-from-the-secondary-region-to-primary"></a>Al volver a aplicar la protección, ¿todos los datos se replican de la región secundaria a la primaria? 

Depende. Si la máquina virtual de la región de origen existe, solo se sincronizan los cambios entre el disco de origen y el disco de destino. Site Recovery compara los discos para ver qué hay diferente y, luego, transfiere los datos. Este proceso suele tardar unas horas. [Más información](azure-to-azure-how-to-reprotect.md#what-happens-during-reprotection).

### <a name="how-long-does-it-take-fail-back"></a>¿Cuánto tiempo se tarda en realizar la conmutación por recuperación?

Después de la reprotección, la conmutación por recuperación tarda casi lo mismo que la conmutación por error de la región primaria a una región secundaria.

## <a name="capacity"></a><a name="capacity"></a>Capacidad

### <a name="how-do-we-ensure-capacity-in-the-target-region"></a>¿Cómo se garantiza la capacidad en la región de destino?

El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias que están protegidas mediante Site Recovery se puedan implementar en la región de destino.

### <a name="does-site-recovery-work-with-reserved-instances"></a>¿Funciona Site Recovery con instancias reservadas?

Sí, puede comprar [máquinas virtuales de Azure reservadas](https://azure.microsoft.com/pricing/reserved-vm-instances/) en la región de recuperación ante desastres, que se usarán en las operaciones de conmutación por error de Site Recovery. No se necesita ninguna configuración adicional.

## <a name="security"></a>Seguridad

### <a name="is-replication-data-sent-to-the-site-recovery-service"></a>¿Se envían mis datos de replicación al servicio de Site Recovery?

No, Site Recovery no intercepta los datos replicados ni tiene información sobre lo que se ejecuta en las máquinas virtuales. Únicamente se envían los metadatos necesarios para coordinar la replicación y la conmutación por error al servicio Site Recovery.

Site Recovery tiene la certificación ISO 27001:2013, 27018, HIPAA y DPA. El servicio está llevando a cabo evaluaciones de SOC2 y FedRAMP JAB.

### <a name="does-site-recovery-encrypt-replication"></a>¿Site Recovery cifra la replicación?

Sí, se admite tanto el cifrado en tránsito como el [cifrado en reposo de Azure](../storage/common/storage-service-encryption.md).

## <a name="next-steps"></a>Pasos siguientes

- [Revise los requisitos de recuperación ante desastres entre regiones de Azure](azure-to-azure-support-matrix.md).
- [Configure la replicación de Azure a Azure](azure-to-azure-tutorial-enable-replication.md).
- Si tiene alguna pregunta después de leer este artículo, publíquela en la [Página de preguntas y respuestas de Microsoft para Azure Recovery Services](/answers/topics/azure-site-recovery.html).
