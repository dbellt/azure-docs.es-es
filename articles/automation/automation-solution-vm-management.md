---
title: Introducción a la característica Start/Stop VMs during off-hours de Azure Automation
description: En este artículo se describe la característica Start/Stop VMs during off-hours, que inicia o detiene las máquinas virtuales según una programación y las supervisa de forma proactiva desde los registros de Azure Monitor.
services: automation
ms.subservice: process-automation
ms.date: 05/25/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0ac3a2dccecf50b53917d878535ce62e124f8f8e
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479554"
---
# <a name="startstop-vms-during-off-hours-overview"></a>Introducción a la característica Start/Stop VMs during off-hours

La característica Start/Stop VMs during off-hours inicia o detiene las máquinas virtuales de Azure habilitadas. Inicia o detiene las máquinas en las programaciones definidas por el usuario, proporciona información mediante los registros de Azure Monitor y envía mensajes de correo electrónico, si se desea, mediante [grupos de acciones](../azure-monitor/alerts/action-groups.md). La característica se puede habilitar en la mayoría de los escenarios tanto en máquinas virtuales clásicas como de Azure Resource Manager.

> [!NOTE]
> Antes de instalar esta versión (v1), nos gustaría que conozca la [siguiente versión](../azure-functions/start-stop-vms/overview.md), que se encuentra en versión preliminar en este momento. La nueva versión (v2) ofrece la misma funcionalidad que esta, pero está diseñada para aprovechar las ventajas de la tecnología más reciente de Azure. Agrega algunas de las características solicitadas de forma común por los clientes, como la compatibilidad con varias suscripciones desde una única instancia de inicio y detención. 
>
> Start/Stop VMs during off-hours (v1) se retirará el 21 de mayo de 2022. 

Esta característica usa el cmdlet [Start-AzVm](/powershell/module/az.compute/start-azvm) para iniciar las máquinas virtuales. Usa [Stop-AzVm](/powershell/module/az.compute/stop-azvm) para detener las máquinas virtuales.

> [!NOTE]
> La característica Start/Stop VMs during off-hours se ha actualizado para admitir las versiones de los módulos de Azure más recientes disponibles. La versión actualizada de esta característica, disponible en Marketplace, no es compatible con los módulos de AzureRM porque hemos migrado de los módulos de AzureRM a los de Az. Aunque los runbooks se hayan actualizado para usar los nuevos cmdlets del módulo Az de Azure, usan el alias de prefijo AzureRM.

La característica proporciona una opción de automatización descentralizada de bajo costo para los usuarios que quieran optimizar sus costos de máquinas virtuales. Puede usar la característica para:

- [Programar el inicio y la detención de las máquinas virtuales](automation-solution-vm-management-config.md#schedule).
- Programar el inicio y la detención de las máquinas virtuales en orden ascendente [mediante etiquetas de Azure](automation-solution-vm-management-config.md#tags). Esta actividad no se admite en máquinas virtuales del modelo clásico.
- Detener automáticamente las máquinas virtuales si se detecta un [bajo uso de la CPU](automation-solution-vm-management-config.md#cpuutil).

Las siguientes son limitaciones de la característica actual:

- Administra máquinas virtuales de cualquier región, pero solo se puede utilizar en la misma suscripción que la cuenta de Azure Automation.
- Está disponible en Azure y Azure Government para cualquier región que admita un área de trabajo de Log Analytics, una cuenta de Azure Automation y alertas. Las regiones de Azure Government no admiten la funcionalidad de correo electrónico en este momento.

## <a name="prerequisites"></a>Requisitos previos

- Los runbooks de la característica Start/Stop VMs during off hours funcionan con una [cuenta de ejecución de Azure](./automation-security-overview.md#run-as-accounts). La cuenta de ejecución es el método de autenticación preferido, ya que emplea la autenticación mediante certificado, en lugar de una contraseña que puede expirar o cambiar con frecuencia.

- Un [área de trabajo de Log Analytics de Azure Monitor](../azure-monitor/logs/design-logs-deployment.md) que almacene los registros de trabajo de los runbooks y los resultados de los flujos de trabajo en un área de trabajo para consultarlos y analizarlos. La cuenta de Automation y el área de trabajo de Log Analytics deben estar en la misma suscripción y la misma región admitida. El área de trabajo ya debe existir; no se puede crear una nueva área de trabajo durante la implementación de esta característica.

Se recomienda usar otra cuenta de Automation para trabajar con máquinas virtuales habilitadas para la característica Start/Stop VMs during off-hours. Las versiones del módulo de Azure se actualizan con frecuencia y puede que sus parámetros cambien. La característica no se actualiza al mismo ritmo y es posible que no funcione con versiones más recientes de los cmdlets que usa. Antes de importar los módulos actualizados en las cuentas de Automation de producción, se recomienda importarlos en una cuenta de prueba, para comprobar que no hay problemas de compatibilidad.

## <a name="permissions"></a>Permisos

Para habilitar la característica Start/Stop VMs during off-hours en las máquinas virtuales, debe tener determinados permisos. Los permisos son diferentes según si la característica usa una cuenta de Automation o un área de trabajo de Log Analytics creadas previamente o se crean unas nuevas.

Si es colaborador de la suscripción y administrador global del inquilino de Azure Active Directory, no tiene que configurar permisos. Si no tiene estos derechos o necesita configurar un rol personalizado, asegúrese de que dispone de los permisos que se describen a continuación.

### <a name="permissions-for-pre-existing-automation-account-and-log-analytics-workspace"></a>Permisos para la cuenta de Automation y área de trabajo de Log Analytics ya existentes

Para permitir la característica Start/Stop VMs during off-hours en las máquinas virtuales mediante una cuenta Automation y un área de trabajo de Log Analytics existentes, necesita los siguientes permisos en el ámbito del grupo de recursos. Para más información sobre los roles, consulte [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).

| Permiso | Ámbito|
| --- | --- |
| Microsoft.Automation/automationAccounts/read | Grupo de recursos |
| Microsoft.Automation/automationAccounts/variables/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/schedules/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/runbooks/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/connections/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/certificates/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/modules/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/modules/read | Grupo de recursos |
| Microsoft.automation/automationAccounts/jobSchedules/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/jobs/write | Grupo de recursos |
| Microsoft.Automation/automationAccounts/jobs/read | Grupo de recursos |
| Microsoft.OperationsManagement/solutions/write | Grupo de recursos |
| Microsoft.OperationalInsights/workspaces/* | Grupo de recursos |
| Microsoft.Insights/diagnosticSettings/write | Grupo de recursos |
| Microsoft.Insights/ActionGroups/Write | Grupo de recursos |
| Microsoft.Insights/ActionGroups/read | Grupo de recursos |
| Microsoft.Resources/subscriptions/resourceGroups/read | Grupo de recursos |
| Microsoft.Resources/deployments/* | Grupo de recursos |

### <a name="permissions-for-new-automation-account-and-new-log-analytics-workspace"></a>Permisos para una cuenta de Automation y un área de trabajo de Log Analytics nuevas

Puede habilitar la característica Start/Stop VMs during off-hours en las máquinas virtuales mediante una cuenta de Automation y un área de trabajo de Log Analytics nuevas. En este caso, necesita los permisos definidos en la sección anterior y los que se definen en esta. También necesita los siguientes roles:

- Coadministrador en la suscripción. Este rol es necesario para crear la cuenta de ejecución clásica si va a administrar máquinas virtuales clásicas. Las [cuentas de ejecución clásicas](automation-create-standalone-account.md#create-a-classic-run-as-account) ya no se crean de forma predeterminada.
- Pertenencia al rol de desarrollador de aplicaciones de [Azure AD](../active-directory/roles/permissions-reference.md). Para obtener más información sobre cómo configurar las cuentas de ejecución, consulte [Permisos para configurar cuentas de ejecución](automation-security-overview.md#permissions).
- Colaborador de la suscripción o los siguientes permisos.

| Permiso |Ámbito|
| --- | --- |
| Microsoft.Authorization/Operations/read | Suscripción|
| Microsoft.Authorization/permissions/read |Suscripción|
| Microsoft.Authorization/roleAssignments/read | Suscripción |
| Microsoft.Authorization/roleAssignments/write | Suscripción |
| Microsoft.Authorization/roleAssignments/delete | Suscripción |
| Microsoft.Automation/automationAccounts/connections/read | Grupo de recursos |
| Microsoft.Automation/automationAccounts/certificates/read | Grupo de recursos |
| Microsoft.Automation/automationAccounts/write | Grupo de recursos |
| Microsoft.OperationalInsights/workspaces/write | Grupo de recursos |

## <a name="components"></a>Componentes

La característica Start/Stop VMs during off-hours incluye runbooks, programaciones e integración preconfigurados con los registros de Azure Monitor. Puede usar estos elementos para personalizar el inicio y el apagado de las máquinas virtuales de modo que se adapten a sus necesidades empresariales.

### <a name="runbooks"></a>Runbooks

En la tabla siguiente se enumeran los runbooks que la característica implementa en la cuenta de Automation. NO realice cambios en el código del runbook. En su lugar, escriba un runbook que aporte una nueva funcionalidad.

> [!IMPORTANT]
> No ejecute directamente ningún runbook cuyo nombre tenga anexado **child**.

Todos los runbooks primarios incluyen el parámetro `WhatIf`. Cuando se establece en True, permite detallar el comportamiento exacto del runbook cuando se ejecuta sin el parámetro y comprueba que el destino sean las máquinas virtuales correctas. Los runbooks solo realizan sus acciones definidas cuando el parámetro `WhatIf` está establecido en False.

|Runbook | Parámetros | Descripción|
| --- | --- | ---|
|AutoStop_CreateAlert_Child | VMObject <br> AlertAction <br> WebHookURI | Se llama desde el runbook primario. Este runbook crea alertas según el recurso para el escenario de AutoStop.|
|AutoStop_CreateAlert_Parent | VMList<br> WhatIf: Verdadero o falso  | Crea o actualiza las reglas de alerta de Azure en las máquinas virtuales de los grupos de recursos o la suscripción de destino. <br> `VMList` es una lista separada por comas de VM (sin espacios en blanco), como, por ejemplo, `vm1,vm2,vm3`.<br> `WhatIf` habilita la validación de la lógica de runbook sin ejecutarse.|
|AutoStop_Disable | None | Deshabilita la programación predeterminada y las alertas de Auto-Stop.|
|AutoStop_VM_Child | WebHookData | Se llama desde el runbook primario. Las reglas de alertas llaman a este runbook para detener una máquina virtual clásica.|
|AutoStop_VM_Child_ARM | WebHookData |Se llama desde el runbook primario. Las reglas de alertas llaman a este runbook para detener una máquina virtual.  |
|ScheduledStartStop_Base_Classic | CloudServiceName<br> Acción: Iniciar o detener<br> VMList  | Realiza la acción de inicio o detención en el grupo de máquinas virtuales clásicas de Cloud Services. |
|ScheduledStartStop_Child | VMName <br> Acción: Iniciar o detener <br> ResourceGroupName | Se llama desde el runbook primario. Ejecuta una acción de inicio o detención para la detención programada.|
|ScheduledStartStop_Child_Classic | VMName<br> Acción: Iniciar o detener<br> ResourceGroupName | Se llama desde el runbook primario. Ejecuta una acción de inicio o detención para la detención programada de máquinas virtuales clásicas. |
|ScheduledStartStop_Parent | Acción: Iniciar o detener <br>VMList <br> WhatIf: Verdadero o falso | Inicia o detiene todas las máquinas virtuales de la suscripción. Edite las variables `External_Start_ResourceGroupNames` y `External_Stop_ResourceGroupNames` para que solo se ejecuten en estos grupos de recursos de destino. También puede excluir máquinas virtuales específicas si actualiza la variable `External_ExcludeVMNames`.|
|SequencedStartStop_Parent | Acción: Iniciar o detener <br> WhatIf: Verdadero o falso<br>VMList| Cree etiquetas denominadas **sequencestart** y **sequencestop** en todas las máquinas virtuales en las que quiera secuenciar la actividad de inicio y detención. Estos nombres de etiqueta distinguen entre mayúsculas y minúsculas. El valor de la etiqueta debe ser una lista de enteros positivos, como `1,2,3`, que se corresponda con el orden deseado para el inicio o la detención. <br>**Nota**: Las máquinas virtuales deben estar dentro de grupos de recursos definidos en las variables `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames` y `External_ExcludeVMNames`. Para que las acciones surtan efecto, deben tener las etiquetas correspondientes.|

### <a name="variables"></a>variables

En la tabla siguiente se enumeran las variables creadas en su cuenta de Automation. Modifique solo las variables cuyo prefijo sea `External`. La modificación de variables con el prefijo `Internal` produce efectos no deseados.

> [!NOTE]
> Las limitaciones en el nombre de la máquina virtual y el grupo de recursos son en gran medida el resultado del tamaño de la variable. Consulte [Recursos de variables en Azure Automation](./shared-resources/variables.md).

|Variable | Descripción|
|---------|------------|
|External_AutoStop_Condition | El operador condicional necesario para configurar la condición antes de desencadenar una alerta. Los valores aceptables son `GreaterThan`, `GreaterThanOrEqual`, `LessThan` y `LessThanOrEqual`.|
|External_AutoStop_Description | Alerta para detener la máquina virtual si el porcentaje de CPU supera el umbral.|
|External_AutoStop_Frequency | Frecuencia de evaluación de la regla. Este parámetro acepta la entrada en formato timespan. Los valores posibles son de 5 minutos a 6 horas. |
|External_AutoStop_MetricName | El nombre de la métrica de rendimiento para el que se configura la regla de alertas de Azure.|
|External_AutoStop_Severity | Gravedad de la alerta de métrica, que puede oscilar entre 0 y 4. |
|External_AutoStop_Threshold | El umbral de la regla de alerta de Azure especificada en la variable `External_AutoStop_MetricName`. Los valores de porcentaje oscilan entre 1 y 100.|
|External_AutoStop_TimeAggregationOperator | El operador de agregación de tiempo aplicado al tamaño de la ventana seleccionada para evaluar la condición. Los valores aceptables son `Average`, `Minimum`, `Maximum`, `Total` y `Last`.|
|External_AutoStop_TimeWindow | El tamaño de la ventana en la que Azure analiza la métrica seleccionada para desencadenar una alerta. Este parámetro acepta la entrada en formato timespan. Los valores posibles son de 5 minutos a 6 horas.|
|External_EnableClassicVMs| Valor que especifica si la característica tiene como destino las máquinas virtuales clásicas. El valor predeterminado es True. Establezca esta variable en False para las suscripciones del Proveedor de soluciones en la nube (CSP) de Azure. Las máquinas virtuales clásicas requieren una [cuenta de ejecución clásica](automation-create-standalone-account.md#create-a-classic-run-as-account).|
|External_ExcludeVMNames | Lista separada por comas de los nombres de máquina virtual que se van a excluir, limitado a 140. Si agrega más de 140 máquinas virtuales a la lista, las especificadas para exclusión pueden iniciarse o detenerse accidentalmente.|
|External_Start_ResourceGroupNames | Lista separada por comas de uno o varios grupos de recursos destinados a las acciones de inicio.|
|External_Stop_ResourceGroupNames | Lista separada por comas de uno o varios grupos de recursos destinados a las acciones de detención.|
|External_WaitTimeForVMRetrySeconds |Tiempo de espera en segundos para que las acciones se realicen en las máquinas virtuales del runbook **SequencedStartStop_Parent**. Esta variable permite que el runbook espere a las operaciones secundarias durante el número de segundos especificado antes de continuar con la siguiente acción. El tiempo de espera máximo es de 10 800 o tres horas. El valor predeterminado es 2 100 segundos.|
|Internal_AutomationAccountName | Especifica el nombre de la cuenta de Automation.|
|Internal_AutoSnooze_ARM_WebhookURI | Especifica el URI de Webhook llamado para el escenario de AutoStop de las máquinas virtuales.|
|Internal_AutoSnooze_WebhookUri | Especifica el URI de Webhook llamado para el escenario de AutoStop de las máquinas virtuales clásicas.|
|Internal_AzureSubscriptionId | El identificador de la suscripción de Azure.|
|Internal_ResourceGroupName | Especifica el nombre del grupo de recursos de la cuenta de Automation.|

>[!NOTE]
>En la variable `External_WaitTimeForVMRetryInSeconds`, el valor predeterminado se ha actualizado de 600 a 2100.

En todos los escenarios, las variables `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames` y `External_ExcludeVMNames` son necesarias para las máquinas virtuales de destino, excepto para las listas de máquinas virtuales separadas por comas de los runbooks **AutoStop_CreateAlert_Parent**, **SequencedStartStop_Parent** y **ScheduledStartStop_Parent**. Es decir, para que se realicen las acciones de inicio o detención, las máquinas virtuales deben pertenecer a los grupos de recursos de destino. La lógica funciona de forma similar a Azure Policy, en el sentido de que el destino puede ser la suscripción o el grupo de recursos, y las acciones las pueden heredar las máquinas virtuales recién creadas. Este enfoque evita tener que mantener una programación independiente para cada máquina virtual y administrar los inicios y detenciones en escala.

### <a name="schedules"></a>Programaciones

En la tabla siguiente se enumera cada una de las programaciones predeterminadas que se crean en su cuenta de Automation.  Puede modificarlas o crear sus propias programaciones personalizadas.  De forma predeterminada, todas las programaciones están deshabilitadas, excepto las de **Scheduled_StartVM** y **Scheduled_StopVM**.

No se deben habilitar todas las programaciones, ya que ello podría crear acciones de programación que se superpongan. Lo mejor es determinar las optimizaciones que se desean y modificarlas en consecuencia. Vea los escenarios de ejemplo en la sección de información general para obtener una explicación.

|Nombre de programación | Frecuencia | Descripción|
|--- | --- | ---|
|Schedule_AutoStop_CreateAlert_Parent | Cada ocho horas | Ejecuta el runbook **AutoStop_CreateAlert_Parent** cada 8 horas, que a su vez detiene los valores basados en la máquina virtual en las variables `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames` y `External_ExcludeVMNames`. Como alternativa, puede especificar una lista separada por comas de las máquinas virtuales mediante el parámetro `VMList`.|
|Scheduled_StopVM | Definido por el usuario, diariamente | Ejecuta el runbook **ScheduledStopStart_Parent**  con un parámetro de detención `Stop`cada día a la hora especificada.  Detiene automáticamente todas las máquinas virtuales que cumplen las reglas definidas por las variables de los recursos.  Habilite la programación relacionada, **Scheduled-StartVM**.|
|Scheduled_StartVM | Definido por el usuario, diariamente | Ejecuta el runbook **ScheduledStopStart_ParentStart** con un valor de parámetro de inicio `Start` cada día a la hora especificada. Inicia automáticamente todas las máquinas virtuales que cumplan las reglas definidas por los recursos de variables.  Habilite la programación relacionada, **Scheduled-StopVM**.|
|Sequenced-StopVM | 1:00 AM (UTC), todos los viernes | Ejecuta el runbook **Sequenced_StopStop_Parent** con el valor del parámetro `Stop` establecido en todos los viernes a la hora especificada.  Detiene secuencialmente (de forma ascendente) todas las máquinas virtuales que tengan la etiqueta **SequenceStop** definida por las variables adecuadas. Para más información sobre los valores de etiqueta y las variables de recurso, consulte [Runbooks](#runbooks).  Habilite la programación relacionada, **Sequenced-StartVM**.|
|Sequenced-StartVM | 1:00 PM (hora UTC), todos los lunes | Ejecuta el runbook **SequencedStopStart_Parent** con un valor de parámetro de detención `Start` cada lunes a la hora determinada. Inicia secuencialmente (de forma descendente) todas las máquinas virtuales que tengan la etiqueta **SequenceStart** definida por las variables adecuadas. Para más información sobre los valores de etiqueta y los recursos de variables, consulte [Runbooks](#runbooks). Habilite la programación relacionada, **Sequenced-StopVM**.|

## <a name="use-the-feature-with-classic-vms"></a>Uso de la característica con máquinas virtuales clásicas

Si usa la característica Start/Stop VMs during off-hours con máquinas virtuales clásicas, Automation procesa todas las máquinas virtuales en orden por cada servicio en la nube. Las máquinas virtuales se siguen procesando en paralelo entre diferentes servicios en la nube. 

Para usar la característica con máquinas virtuales clásicas, necesita una cuenta de ejecución clásica, que no se crea de forma predeterminada. Para obtener instrucciones sobre cómo crear una cuenta de ejecución clásica, consulte [Creación de una cuenta de ejecución clásica](automation-create-standalone-account.md#create-a-classic-run-as-account).

Si tiene más de 20 máquinas virtuales por cada servicio en la nube, estas son algunas recomendaciones:

* Cree varias programaciones con el runbook primario **ScheduledStartStop_Parent** y especifique 20 máquinas virtuales por programación.
* En las propiedades de la programación, use el parámetro `VMList` para especificar los nombres de las VM como una lista separada por comas (sin espacios en blanco).

De lo contrario, si el trabajo de Automation con esta característica se ejecuta durante más de tres horas, se descarga temporalmente o se detiene debido al límite de [distribución equilibrada](automation-runbook-execution.md#fair-share).

Las suscripciones de CSP de Azure solo admiten el modelo de Azure Resource Manager. Los servicios que no son de Azure Resource Manager no están disponibles en el programa. Cuando se ejecuta la característica Start/Stop VMs during off-hours, es posible que reciba errores, ya que contiene cmdlets para administrar los recursos clásicos. Para más información sobre CSP, consulte [Servicios disponibles en las suscripciones de CSP](/azure/cloud-solution-provider/overview/azure-csp-available-services). Si usa una suscripción de CSP, debe establecer la variable [External_EnableClassicVMs](#variables) en False después de la implementación.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="view-the-feature"></a>Visualización de la característica

Use uno de los siguientes mecanismos para acceder a la característica habilitada:

* En la cuenta de Automation, seleccione **Start/Stop VM** en **Recursos relacionados**. En la página Start/Stop VM, seleccione **Manage the solution** (Administrar la solución), en la sección **Manage Start/Stop VM Solutions** (Administrar soluciones Start/Stop VM).

* Vaya al área de trabajo de Log Analytics vinculada a su cuenta de Automation. Después de seleccionar el área de trabajo, elija **Soluciones** en el panel izquierdo. En la página Soluciones, seleccione **Start-Stop-VM[workspace]** en la lista.  

Al seleccionar la característica se muestra la página **Start-Stop-VM[workspace]** . En ella puede examinar detalles importantes, como el icono **StartStopVM**. Al igual que en el área de trabajo de Log Analytics, este icono muestra un número y una representación gráfica de los trabajos de runbook de la característica que se han iniciado y han terminado correctamente.

![Página Update Management de Automation](media/automation-solution-vm-management/azure-portal-vmupdate-solution-01.png)

Puede realizar más análisis de los registros de trabajos. Para ello, solo debe hacer clic en el icono del anillo. El panel muestra el historial de trabajos y las consultas de búsqueda de registros predefinidas. Cambie al portal avanzado de Log Analytics para realizar búsquedas basadas en sus consultas de búsqueda.

## <a name="update-the-feature"></a>Actualización de la característica

Si ha implementado una versión anterior de la característica Start/Stop VMs during off-hours, debe eliminarla de la cuenta antes de implementar una versión actualizada. Siga los pasos para [quitar la característica](automation-solution-vm-management-remove.md#delete-the-feature) y luego siga los pasos para [habilitarla](automation-solution-vm-management-enable.md).

## <a name="next-steps"></a>Pasos siguientes

Para habilitar la característica en las máquinas virtuales de su entorno, consulte [Habilitación de Start/Stop VMs during off-hours](automation-solution-vm-management-enable.md).
