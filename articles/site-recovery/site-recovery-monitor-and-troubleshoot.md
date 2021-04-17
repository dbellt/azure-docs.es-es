---
title: Supervisión de Azure Site Recovery | Microsoft Docs
description: Supervisión y solución de problemas de replicación y de operaciones de Azure Site Recovery mediante el portal
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/30/2019
ms.openlocfilehash: a80f68c230bfed5caad4fd5ce474d8d06fd3d093
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106579321"
---
# <a name="monitor-site-recovery"></a>Supervisión de Site Recovery

En este artículo, aprenderá a supervisar Azure [Site Recovery](site-recovery-overview.md), mediante la supervisión integrada de Site Recovery.  Puede supervisar:

- El mantenimiento y el estado de las máquinas replicadas por Site Recovery.
- El estado de conmutación por error de prueba de las máquinas.
- Los problemas y errores que afectan a la configuración y la replicación.
- Los componentes de la infraestructura, como los servidores locales.


## <a name="before-you-start"></a>Antes de comenzar

Es posible que quiera revisar las [preguntas comunes de supervisión](monitoring-common-questions.md) antes de empezar.

## <a name="monitor-in-the-dashboard"></a>Supervisión en el panel

1. En el almacén, haga clic en **Información general**. El panel de Recovery Services consolida toda la información de supervisión para el almacén en una sola ubicación. Existen páginas tanto para Site Recovery como para el servicio Azure Backup, y puede cambiar entre ellas.

    ![Panel de Site Recovery](./media/site-recovery-monitor-and-troubleshoot/dashboard.png)

2. En el panel, explore en profundidad las diferentes áreas. 

    ![Captura de pantalla en la que se muestran las áreas del panel donde puede explorar en profundidad.](./media/site-recovery-monitor-and-troubleshoot/site-recovery-overview-page.png).

3. En **Elementos replicados**, haga clic en **Ver todo** para ver todos los servidores en el almacén.
4. Haga clic en los detalles del estado de cada sección para explorar en profundidad.
5. En **Vista de la infraestructura**, ordene la información de supervisión por el tipo de máquinas que replica.

## <a name="monitor-replicated-items"></a>Supervisión de elementos replicados

En **Elementos replicados**, supervise el estado de todas las máquinas que tienen habilitada la replicación en el almacén.

**State** | **Detalles**
--- | ---
Healthy | La replicación progresa con normalidad. No se ha detectado ningún síntoma de error o advertencia.
Advertencia | Se han detectado uno o varios síntomas de advertencia que podrían afectar a la replicación.
Crítico | Se han detectado uno o varios síntomas de errores críticos de replicación.<br/><br/> Estos síntomas de error suelen ser indicadores de que la replicación se ha quedado bloqueada, o de que no está progresando a la misma velocidad que el cambio de datos.
No aplicable | Los servidores cuya replicación no se espera actualmente. Esto podría incluir las máquinas que se han conmutado por error.

## <a name="monitor-test-failovers"></a>Supervisión de conmutaciones por error de prueba

En **Prueba de conmutación por error correcta**, supervise el estado de la conmutación por error de las máquinas del almacén.

- Recomendamos que ejecute una conmutación por error de prueba en máquinas replicadas al menos una vez cada seis meses. Es una forma de comprobar que la conmutación por error funciona según lo esperado, sin interrumpir su entorno de producción. 
- Una conmutación por error de prueba se considera correcta solo después de que la conmutación por error y la limpieza posterior a la conmutación por error se hayan completado correctamente.

**State** | **Detalles**
--- | ---
Prueba recomendada | Máquinas que no han tenido una conmutación por error de prueba desde que se habilitó la protección.
Realizada correctamente | Máquinas con una o varias conmutaciones por error de prueba correctas.
No aplicable | Máquinas que, actualmente, no son aptas para una conmutación por error de prueba. Por ejemplo, las máquinas que se conmutan por error cuentan con replicación inicial, conmutación por error de prueba o conmutación por error en curso.

## <a name="monitor-configuration-issues"></a>Supervisión de problemas de configuración

En **Problemas de configuración**, supervise los problemas que puedan afectar a su capacidad de conmutar por error correctamente.

- Los problemas de configuración (salvo en el caso de la disponibilidad de actualizaciones de software) se detectan mediante una operación periódica de validador que se ejecuta cada 12 horas de forma predeterminada. Puede forzar la operación de validador para que se ejecute inmediatamente; para ello, haga clic en el icono de actualización junto al encabezado de la sección **Problemas de configuración**.
- Haga clic en los vínculos para obtener más detalles. En caso de producirse problemas que afecten a máquinas específicas, haga clic en **Necesita atención** en la columna **Configuraciones de destino**. Entre los detalles se incluyen recomendaciones de corrección.

**State** | **Detalles**
--- | ---
Configuraciones que faltan | Falta una configuración necesaria, como una red de recuperación o un grupo de recursos.
Recursos que faltan | Un recurso especificado no se encuentra o no está disponible en la suscripción. Por ejemplo, el recurso se eliminó o migró. Entre los recursos supervisados se incluían el grupo de recursos de destino, la red virtual o subred de destino, la cuenta de almacenamiento de registro o destino, el conjunto de disponibilidad de destino y la dirección IP de destino.
Cuota de suscripción |  El saldo disponible de la cuota de recursos de suscripción se compara con el saldo necesario para conmutar por error todas las máquinas virtuales en el almacén.<br/><br/> Si no hay recursos suficientes, se notificará que el saldo de la cuota es insuficiente.<br/><br/> Las cuotas se supervisan para el recuento de núcleos de máquinas virtuales, el recuento de núcleos de familia de máquinas virtuales y el recuento de tarjetas de interfaz de red (NIC).
Actualizaciones de software | La disponibilidad de nuevas actualizaciones de software e información sobre versiones de software que van a caducar.


## <a name="monitor-errors"></a>Supervisión de errores

En **Resumen de errores**, supervise los síntomas de los errores actualmente activos que pueden afectar a la replicación de servidores en el almacén, junto con el número de máquinas afectadas.

- Los errores que afectan a los componentes de la infraestructura local se muestran al principio de la sección. Por ejemplo, la falta de recepción de latido del proveedor de Azure Site Recovery en el servidor de configuración local o en el host de Hyper-V.
- A continuación, se muestran los síntomas de errores de replicación que afectan a los servidores replicados.
- Las entradas de la tabla se ordenan por orden decreciente de gravedad del error y, a continuación, por orden decreciente de recuento de las máquinas afectadas.
- El recuento de los servidores afectados es una forma útil de conocer si un único problema subyacente puede afectar a varias máquinas. Por ejemplo, un problema de red podría afectar potencialmente a todas las máquinas que se replican en Azure. 
- Pueden producirse varios errores de replicación en un solo servidor. En este caso, cada síntoma de error cuenta ese servidor en la lista de sus servidores afectados. Una vez solucionado el problema, los parámetros de replicación mejoran y el error se borra de la máquina.

## <a name="monitor-the-infrastructure"></a>Supervise la infraestructura.

En la **Vista de la infraestructura**, supervise los componentes de infraestructura implicados en la replicación, así como el estado de conectividad entre los servidores y los servicios de Azure.

- Una línea verde indica que el estado de la conexión es correcto.
- Una línea roja con el icono de error superpuesto indica la existencia de uno o varios síntomas de error que afectan a la conectividad.
-  Mantenga el puntero del mouse sobre el icono del error para mostrar el error y el número de entidades afectadas. Haga clic en el icono de una lista filtrada de entidades afectadas.

    ![Vista de la infraestructura de Site Recovery (almacén)](./media/site-recovery-monitor-and-troubleshoot/site-recovery-vault-infra-view.png)

### <a name="tips-for-monitoring-the-infrastructure"></a>Sugerencias para la supervisión de la infraestructura

- Asegúrese de que los componentes de infraestructura local (servidor de configuración, servidores de procesos, servidores VMM, hosts de Hyper-V y máquinas de VMware) estén ejecutando las versiones más recientes del proveedor o los agentes de Site Recovery.
- Para usar todas las características en la vista de la infraestructura, debe ejecutar el [paquete acumulativo de actualizaciones 22](https://support.microsoft.com/help/4072852) para estos componentes.
- Para utilizar la vista de la infraestructura, seleccione el escenario de replicación adecuado en su entorno. Puede explorar en profundidad en la vista para obtener más detalles. En la siguiente tabla se muestran los escenarios representados.

    **Escenario** | **State**  | **¿Vista disponible?**
    --- |--- | ---
    **Replicación entre sitios locales** | Todos los estados | No 
    **Replicación de máquina virtual de Azure entre las regiones de Azure**  | Replicación habilitada/replicación inicial en curso | Sí
    **Replicación de máquina virtual de Azure entre las regiones de Azure** | Conmutada por error/conmutación por recuperación | No   
    **Replicación de VMware en Azure** | Replicación habilitada/replicación inicial en curso | Sí     
    **Replicación de VMware en Azure** | Conmutada por error/conmutada por recuperación | No      
    **Replicación de Hyper-V en Azure** | Conmutada por error/conmutada por recuperación | No

- Para ver la vista de la infraestructura para una sola máquina de replicación, en el menú Almacén, haga clic en **Elementos replicados** y seleccione un servidor.  




## <a name="monitor-recovery-plans"></a>Supervisión de planes de recuperación

En **Planes de recuperación**, supervise el número de planes, cree nuevos planes y modifique los existentes.  

## <a name="monitor-jobs"></a>Supervisión de trabajos

En **Trabajos**, supervise el estado de las operaciones de Site Recovery.

- La mayoría de las operaciones de Azure Site Recovery se ejecutan de forma asincrónica, con un trabajo de seguimiento que se crea y usa para realizar un seguimiento del progreso de la operación. 
- El objeto de trabajo tiene toda la información que necesita para realizar un seguimiento del estado y el progreso de la operación. 

Supervise los trabajos de la siguiente manera:

1. En el panel > sección **Trabajos**, puede ver un resumen de los trabajos que se han completado, están en curso o en espera de entrada en las últimas 24 horas. Puede hacer clic en cualquier estado para obtener más información sobre los trabajos pertinentes.
2. Haga clic en **Ver todo** para ver todos los trabajos en las últimas 24 horas.

    > [!NOTE]
    > También puede tener acceso a información del trabajo en el menú Almacén > **Trabajos de Site Recovery**. 

2. En la lista **Trabajos de Site Recovery**, se muestra una lista de trabajos. En el menú superior, puede obtener detalles del error para trabajos específicos, filtrar la lista de trabajos en función de criterios específicos y exportar los detalles del trabajo seleccionado a Excel.
3. Puede profundizar en un trabajo haciendo clic en él. 

## <a name="monitor-virtual-machines"></a>Supervisión de máquinas virtuales

En **Elementos replicados**, obtenga una lista de las máquinas replicadas. 
    ![Vista de lista de elementos replicados de Site Recovery](./media/site-recovery-monitor-and-troubleshoot/site-recovery-virtual-machine-list-view.png)

2. Puede ver y filtrar la información. En el menú Acción de la parte superior, puede realizar acciones para una máquina determinada, como ejecutar una conmutación por error de prueba o ver errores específicos.
3. Haga clic en **Columnas** para mostrar columnas adicionales. Por ejemplo, para mostrar RPO, problemas de configuración de destino y errores de replicación.
4. Haga clic en **Filtrar** para ver información basada en parámetros específicos como el estado de la replicación, o bien una directiva de replicación determinada.
5. Haga clic con el botón derecho en una máquina para iniciar operaciones como la conmutación por error de prueba para ella o ver detalles del error específicos asociados a ella.
6. Haga clic en una máquina para profundizar en más detalles para ella. Los detalles incluyen:
   - **Información de replicación**: estado y mantenimiento actual de la máquina.
   - **RPO** (objetivo de punto de recuperación): el RPO actual de la máquina virtual y la hora a la que se calculó por última vez el RPO.
   - **Puntos de recuperación**: puntos de recuperación disponibles más recientes de la máquina.
   - **Preparación de la conmutación por error**: indica si se ejecutó una conmutación por error de prueba para la máquina, la versión del agente que se ejecuta en la máquina (para las máquinas que ejecutan Mobility Service) y cualquier problema de configuración.
   - **Errores**: lista de síntomas de errores de replicación observados actualmente en la máquina y posibles causas o acciones.
   - **Eventos**: una lista cronológica de eventos recientes que afectan a la máquina. Los detalles del error muestran los síntomas de errores observados actualmente, mientras que los eventos son un registro histórico de problemas que han afectado a la máquina.
   - **Vista de la infraestructura**: muestra el estado de la infraestructura para el escenario al replicarse las máquinas en Azure.

     ![Información general/detalles de elementos replicados de Site Recovery](./media/site-recovery-monitor-and-troubleshoot/site-recovery-virtual-machine-details.png)

## <a name="subscribe-to-email-notifications"></a>Suscribirse a las notificaciones por correo electrónico

Puede suscribirse para recibir notificaciones por correo electrónico para estos eventos críticos:
 
- Estado crítico para la máquina replicada.
- Ausencia de conectividad entre los componentes de infraestructura local y el servicio Site Recovery. La conectividad entre Site Recovery y los servidores locales registrados en un almacén se detecta mediante un mecanismo de latido.
- Errores de conmutación por error.

Suscríbase de la siguiente manera:

En el almacén > sección **Supervisión**, haga clic en **Eventos de Site Recovery**.
1. Haga clic en **Notificaciones por correo electrónico**.
1. En **Notificaciones por correo electrónico**, active las notificaciones y especifique a quién deben enviarse. Puede enviarlas a todos los administradores de suscripción y, de forma opcional, a direcciones de correo electrónico específicas.

    ![Notificaciones por correo electrónico](./media/site-recovery-monitor-and-troubleshoot/email.png)

## <a name="next-steps"></a>Pasos siguientes

[Más información](monitor-log-analytics.md) sobre la supervisión de Site Recovery con Azure Monitor.
