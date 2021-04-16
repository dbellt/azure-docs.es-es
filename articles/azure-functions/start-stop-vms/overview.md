---
title: Información general sobre Start/Stop VMs v2 (versión preliminar)
description: En este artículo se describe la versión dos de la característica Start/Stop VMs (versión preliminar), que inicia o detiene las máquinas virtuales de Azure Resource Manager y clásicas según una programación.
ms.topic: conceptual
ms.service: azure-functions
ms.subservice: ''
ms.date: 03/29/2021
ms.openlocfilehash: 44bfbaa8b18ebeab3b74bc696a16fc4cfb6c08ec
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220941"
---
# <a name="startstop-vms-v2-preview-overview"></a>Información general sobre Start/Stop VMs v2 (versión preliminar)

La característica Start/Stop VMs v2 (versión preliminar) inicia o detiene las máquinas virtuales (VM) de Azure en varias suscripciones. Inicia o detiene las máquinas virtuales de Azure según las programaciones definidas por el usuario, proporciona información a través de [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) y envía notificaciones opcionales mediante [grupos de acciones](../../azure-monitor/alerts/action-groups.md). La característica puede administrar tanto las máquinas virtuales de Azure Resource Manager como las máquinas virtuales clásicas de la mayoría de los escenarios.

Esta nueva versión de Start/Stop VMs v2 (versión preliminar) proporciona una opción de automatización económica centralizada para los clientes que quieren optimizar los costos de las máquinas virtuales. Ofrece la misma funcionalidad que la [versión original](../../automation/automation-solution-vm-management.md) disponible con Azure Automation, pero está diseñada para aprovechar las ventajas de la tecnología más reciente de Azure.

## <a name="overview"></a>Información general

Start/Stop VMs v2 (versión preliminar) se ha rediseñado y no depende de los registros de Azure Automation o Azure Monitor, como requiere la [versión anterior](../../automation/automation-solution-vm-management.md). Esta versión se basa en [Azure Functions](../../azure-functions/functions-overview.md) para controlar la ejecución del inicio y detención de la máquina virtual.

En Azure Active Directory (Azure AD) se crea una identidad administrada para la aplicación de Azure Functions y permite que Start/Stop VMs v2 (versión preliminar) acceda fácilmente a otros recursos protegidos por Azure AD, como las aplicaciones lógicas y las máquinas virtuales de Azure. Para más información sobre las identidades administradas en Azure AD, consulte [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Se crea una función de punto de conexión de desencadenador HTTP para admitir los escenarios de programación y secuencia que se incluyen con la característica, tal como se muestra en la tabla siguiente.

|Nombre |Desencadenador |Descripción |
|-----|--------|------------|
|AlertAvailabilityTest |Temporizador |Esta función realiza la prueba de disponibilidad para asegurarse de que la función principal **AutoStopVM** está siempre disponible.|
|AutoStop |HTTP |Esta función admite el escenario de **detención automática**, que es la función de punto de entrada a la que se llama desde la aplicación lógica.|
|AutoStopAvailabilityTest |Temporizador |Esta función realiza la prueba de disponibilidad para asegurarse de que la función principal **AutoStop** está siempre disponible.|
|AutoStopVM |HTTP |La alerta de máquina virtual desencadena automáticamente esta función cuando se cumple la condición de alerta.|
|CreateAutoStopAlertExecutor |Cola |Esta función obtiene la información de carga de la función **AutoStop** para crear la alerta en la máquina virtual.|
|Programado |HTTP |Esta función es tanto para escenarios programados como secuenciados (diferenciados por el esquema de carga). Es la función de punto de entrada a la que se llama desde la aplicación lógica y toma la carga para procesar la operación de inicio o detención de la máquina virtual. |
|ScheduledAvailabilityTest |Temporizador |Esta función realiza la prueba de disponibilidad para asegurarse de que la función principal **Scheduled** está siempre disponible.|
|VirtualMachineRequestExecutor |Cola |Esta función realiza la operación de inicio y detención real en la máquina virtual.|
|VirtualMachineRequestOrchestrator |Cola |Esta función obtiene la información de carga de la función **Scheduled** y orquesta las solicitudes de inicio y detención de la máquina virtual.|

Por ejemplo, la función de desencadenador HTTP **Scheduled** se utiliza para controlar los escenarios de programación y secuencia. Del mismo modo, la función de desencadenador HTTP **AutoStop** controla el escenario de detención automática.

Las funciones de desencadenador basadas en cola son necesarias para admitir esta característica. Todos los desencadenadores basados en temporizador se usan para realizar la prueba de disponibilidad y supervisar el mantenimiento del sistema.

 [Azure Logic Apps](../../logic-apps/logic-apps-overview.md) se usa para configurar y administrar las programaciones de inicio y detención de la actuación de la máquina virtual con una llamada a la función mediante una carga JSON. De manera predeterminada, durante la implementación inicial, crea un total de cinco instancias de Logic Apps para los siguientes escenarios:

- Programado: las acciones de inicio y detención se basan en una programación que se especifica en las máquinas virtuales de Azure Resource Manager y clásicas. **ststv2_vms_Scheduled_start** y **ststv2_vms_Scheduled_stop** configuran el inicio y la detención programados.

- Secuenciado: las acciones de inicio y detención se basan en una programación dirigida a las máquinas virtuales con etiquetas de secuencia predefinidas. Solo se admiten dos etiquetas con nombre: **sequencestart** y **sequencestop**. **ststv2_vms_Sequenced_start** y **ststv2_vms_Sequenced_stop** configuran el inicio y la detención secuenciados.

    > [!NOTE]
    > En este escenario solo se admiten las máquinas virtuales de Azure Resource Manager.

- AutoStop: esta funcionalidad solo se usa para realizar una acción de detención en máquinas virtuales de Azure Resource Manager y clásicas en función de su uso de CPU. También puede ser una *actuación* basada en programación, que crea alertas en máquinas virtuales y en función de la condición, la alerta se desencadena para realizar la acción de detención. **ststv2_vms_AutoStop** configura la funcionalidad de detención automática.

Cada acción de inicio o detención admite la asignación de una o varias suscripciones, grupos de recursos o una lista de máquinas virtuales.

Start/Stop VMs v2 (versión preliminar) también usa una cuenta de Azure Storage, que es necesaria para Functions, para dos propósitos:

   - Usa Azure Table Storage para almacenar los metadatos de la operación de ejecución (es decir, la acción de iniciar o detener la máquina virtual).

   - Usa Azure Queue Storage para admitir los desencadenadores basados en cola de Azure Functions.

Todos los datos de telemetría, que son registros de seguimiento de la ejecución de la aplicación de funciones, se envían a la instancia de Application Insights conectada. Puede ver los datos de telemetría almacenados en Application Insights en un conjunto de visualizaciones predefinidas que se presentan en un [panel de Azure](../../azure-portal/azure-portal-dashboards.md) compartido.

:::image type="content" source="media/overview/shared-dashboard-sml.png" alt-text="Panel de estado compartido de Start/Stop VMs":::

También se envían notificaciones por correo electrónico como resultado de las acciones realizadas en las máquinas virtuales.

## <a name="new-releases"></a>Nuevas versiones

Cuando se publica una nueva versión de Start/Stop VMs v2 (versión preliminar), la instancia se actualiza automáticamente sin tener que volverla a implementar manualmente.

## <a name="supported-scoping-options"></a>Opciones de ámbito admitidas

### <a name="subscription"></a>Subscription

El ámbito de una suscripción puede usarse cuando sea necesario realizar la acción de inicio y detención en todas las máquinas virtuales de toda una suscripción, y puede seleccionar varias suscripciones si es necesario.  

También puede especificar una lista de las máquinas virtuales que se deben excluir y se omitirán de la acción. También puede usar caracteres comodín para especificar todos los nombres que se pueden omitir simultáneamente.

### <a name="resource-group"></a>Resource group

El ámbito de un grupo de recursos se puede usar cuando es necesario realizar la acción de inicio y detención en todas las máquinas virtuales mediante la especificación de uno o más nombres de grupos de recursos y de una o varias suscripciones.

También puede especificar una lista de las máquinas virtuales que se deben excluir y se omitirán de la acción. También puede usar caracteres comodín para especificar todos los nombres que se pueden omitir simultáneamente.

### <a name="vmlist"></a>VMList

Se puede especificar una lista de máquinas virtuales cuando sea necesario realizar la acción de inicio y detención en un conjunto específico de máquinas virtuales y en varias suscripciones. Esta opción no admite la especificación de una lista de máquinas virtuales que se deben excluir.

## <a name="prerequisites"></a>Requisitos previos

- Debe disponer de una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).

- A la cuenta se le debe haber concedido el permiso de [colaborador](../../role-based-access-control/built-in-roles.md#contributor) en la suscripción.

- Start/Stop VMs v2 (versión preliminar) está disponible en todas las regiones de la nube de Azure global y del Gobierno de EE. UU. que aparecen en la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=functions) para Azure Functions.

## <a name="next-steps"></a>Pasos siguientes

Para implementar esta característica, consulte [Implementación de Start/Stop VMs](deploy.md) (versión preliminar).