---
title: Solución de problemas de la característica Start/Stop VMs during off-hours de Azure Automation
description: En este artículo se explica cómo solucionar y resolver los problemas que surgen durante el uso de la característica Start/Stop VMs during off-hours.
services: automation
ms.subservice: process-automation
ms.date: 04/04/2019
ms.topic: troubleshooting
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b03481b41e879acac4c3b193d72594454f650525
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831193"
---
# <a name="troubleshoot-startstop-vms-during-off-hours-issues"></a>Solución de problemas de la característica Start/Stop VMs during off-hours

En este artículo se proporciona información sobre la solución de los problemas que surgen al implementar la característica Start/Stop VMs during off-hours de Azure Automation en las máquinas virtuales. 

## <a name="scenario-startstop-vms-during-off-hours-fails-to-properly-deploy"></a><a name="deployment-failure"></a>Escenario: no se puede implementar correctamente Start/Stop VMs during off-hours

### <a name="issue"></a>Problema

Cuando se implementa [Start/Stop VMs during off-hours](../automation-solution-vm-management.md), se produce uno de los siguientes errores:

```error
Account already exists in another resourcegroup in a subscription. ResourceGroupName: [MyResourceGroup].
```

```error
Resource 'StartStop_VM_Notification' was disallowed by policy. Policy identifiers: '[{\\\"policyAssignment\\\":{\\\"name\\\":\\\"[MyPolicyName]".
```

```error
The subscription is not registered to use namespace 'Microsoft.OperationsManagement'.
```

```error
The subscription is not registered to use namespace 'Microsoft.Insights'.
```

```error
The scope '/subscriptions/000000000000-0000-0000-0000-00000000/resourcegroups/<ResourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<WorkspaceName>/views/StartStopVMView' cannot perform write operation because following scope(s) are locked: '/subscriptions/000000000000-0000-0000-0000-00000000/resourceGroups/<ResourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<WorkspaceName>/views/StartStopVMView'. Please remove the lock and try again
```

```error
A parameter cannot be found that matches parameter name 'TagName'
```

```error
Start-AzureRmVm : Run Login-AzureRmAccount to login
```

### <a name="cause"></a>Causa

Se pueden producir errores de implementación por uno de los siguientes motivos:

- Ya hay una cuenta de Automation con el mismo nombre en la región seleccionada.
- Una directiva no permite la implementación de Start/Stop VMs during off-hours.
- El tipo de recurso `Microsoft.OperationsManagement`, `Microsoft.Insights` o `Microsoft.Automation` no está registrado.
- El área de trabajo de Log Analytics está bloqueada.
- Tiene una versión desfasada de los módulos de AzureRM o la característica Start/Stop VMs during off-hours.

### <a name="resolution"></a>Solución

Revise las siguientes correcciones para conocer las posibles resoluciones:

* Las cuentas de Automation deben ser únicas en una misma región de Azure, incluso si se encuentran en distintos grupos de recursos. Compruebe sus cuentas de Automation existentes en la región de destino.
* Una directiva existente impide que se implemente un recurso necesario para Start/Stop VMs during off-hours. Vaya a las asignaciones de directivas en Azure Portal, y compruebe si tiene una asignación de directiva que no permite la implementación de este recurso. Para más información, consulte el [error RequestDisallowedByPolicy](../../azure-resource-manager/templates/error-policy-requestdisallowedbypolicy.md).
* Para implementar Start/Stop VMs during off-hours, la suscripción debe estar registrada en los siguientes espacios de nombres de recursos de Azure:

    * `Microsoft.OperationsManagement`
    * `Microsoft.Insights`
    * `Microsoft.Automation`

   Para más información acerca de los errores al registrar proveedores, consulte [Resolución de errores para el registro de proveedores de recursos](../../azure-resource-manager/templates/error-register-resource-provider.md).
* Si tiene un bloqueo en el área de trabajo de Log Analytics, vaya al área de trabajo de Azure Portal y quite los bloqueos en el recurso.
* Si estas resoluciones no resuelven su problema, siga las instrucciones que aparecen en [Actualizar la característica](../automation-solution-vm-management.md#update-the-feature) para volver a implementar Start/Stop VMs during off-hours.

## <a name="scenario-all-vms-fail-to-start-or-stop"></a><a name="all-vms-fail-to-startstop"></a>Escenario: error al iniciar o detener todas las VM

### <a name="issue"></a>Problema

Ha configurado Start/Stop VMs during off-hours, pero esta no inicia o detiene todas las VM.

### <a name="cause"></a>Causa

Este error puede deberse a alguna de las siguientes razones:

- Una programación no está configurada correctamente.
- Es posible que la cuenta de ejecución no esté configurada correctamente.
- Es posible que un runbook presente errores.
- Es posible que las VM se hayan excluido.

### <a name="resolution"></a>Solución

Revise la siguiente lista de posibles resoluciones:

* Compruebe que ha configurado correctamente una programación para Start/Stop VMs during off-hours. Para más información sobre cómo configurar una programación, consulte [Programaciones](../shared-resources/schedules.md).

* Compruebe los [flujos de trabajo](../automation-runbook-execution.md#job-statuses) para buscar errores. Busque los trabajos de alguno de los siguientes runbooks:

  * **AutoStop_CreateAlert_Child**
  * **AutoStop_CreateAlert_Parent**
  * **AutoStop_Disable**
  * **AutoStop_VM_Child**
  * **ScheduledStartStop_Base_Classic**
  * **ScheduledStartStop_Child_Classic**
  * **ScheduledStartStop_Child**
  * **ScheduledStartStop_Parent**
  * **SequencedStartStop_Parent**

* Verifique que su [cuenta de ejecución](../automation-security-overview.md#run-as-accounts) tiene los permisos adecuados para las VM que intenta iniciar o detener. Para obtener información sobre cómo comprobar los permisos para un recurso, vea [Guía de inicio rápido: Visualización de los roles asignados a un usuario mediante Azure Portal](../../role-based-access-control/check-access.md). Deberá proporcionar el identificador de aplicación de la entidad de servicio usada por la cuenta de ejecución. Puede recuperar este valor si va a la cuenta de Automation en Azure Portal. Seleccione **Cuentas de ejecución** en **Configuración de la cuenta** y seleccione la cuenta de ejecución adecuada.

* Las VM no pueden iniciarse o detenerse en caso de que se hayan excluido de forma explícita. Las VM excluidas se definen en la variable `External_ExcludeVMNames` en la cuenta de Automation en la que está implementada la característica. En el ejemplo siguiente se muestra cómo consultar ese valor con PowerShell.

  ```powershell-interactive
  Get-AzAutomationVariable -Name External_ExcludeVMNames -AutomationAccountName <automationAccountName> -ResourceGroupName <resourceGroupName> | Select-Object Value
  ```

## <a name="scenario-some-of-my-vms-fail-to-start-or-stop"></a><a name="some-vms-fail-to-startstop"></a>Escenario: Algunas máquinas virtuales no se pueden iniciar o detener

### <a name="issue"></a>Problema

Ha configurado Start/Stop VMs during off-hours, pero esta no inicia o detiene algunas de las VM configuradas.

### <a name="cause"></a>Causa

Este error puede deberse a alguna de las siguientes razones:

- En el escenario en secuencia, es posible que una etiqueta falte o sea incorrecta.
- Es posible que la VM esté excluida.
- Es posible que la cuenta de ejecución no tenga permisos suficientes en la VM.
- Es posible que la VM tenga algún problema que haya impedido su inicio o detención.

### <a name="resolution"></a>Solución

Revise la siguiente lista de posibles resoluciones:

* Cuando usa el [escenario en secuencia](../automation-solution-vm-management.md) de Start/Stop VMs during off-hours, debe asegurarse de que cada VM que desea iniciar o detener tiene la etiqueta correcta. Asegúrese de que las máquinas virtuales que desea iniciar tienen la etiqueta `sequencestart` y que aquellas que desea detener tienen la etiqueta `sequencestop`. Ambas etiquetas requieren un valor entero positivo. Puede usar una consulta similar a la del ejemplo siguiente para buscar todas las máquinas virtuales con las etiquetas y sus valores.

  ```powershell-interactive
  Get-AzResource | ? {$_.Tags.Keys -contains "SequenceStart" -or $_.Tags.Keys -contains "SequenceStop"} | ft Name,Tags
  ```

* Las VM no pueden iniciarse o detenerse en caso de que se hayan excluido de forma explícita. Las VM excluidas se definen en la variable `External_ExcludeVMNames` en la cuenta de Automation en la que está implementada la característica. En el ejemplo siguiente se muestra cómo consultar ese valor con PowerShell.

  ```powershell-interactive
  Get-AzAutomationVariable -Name External_ExcludeVMNames -AutomationAccountName <automationAccountName> -ResourceGroupName <resourceGroupName> | Select-Object Value
  ```

* Para iniciar y detener las VM, la cuenta de ejecución de la cuenta de Automation debe tener los permisos apropiados para la VM. Para obtener información sobre cómo comprobar los permisos para un recurso, vea [Guía de inicio rápido: Visualización de los roles asignados a un usuario mediante Azure Portal](../../role-based-access-control/check-access.md). Deberá proporcionar el identificador de aplicación de la entidad de servicio usada por la cuenta de ejecución. Puede recuperar este valor si va a la cuenta de Automation en Azure Portal. Seleccione **Cuentas de ejecución** en **Configuración de la cuenta** y seleccione la cuenta de ejecución adecuada.
* Si la VM presenta algún problema para iniciarse o desasignarse, es posible que exista un problema en la propia VM. Entre los ejemplos se encuentra una actualización en curso mientras una VM intenta apagarse, el bloqueo de un servicio, etc. Vaya al recurso de máquina virtual y consulte los **registros de actividad** para ver si en ellos se ha identificado algún error. También puede tratar de iniciar sesión en la VM para ver si existe algún error en los registros de eventos. Para más información sobre la solución de problemas de la máquina virtual, consulte [Solución de problemas de máquinas virtuales de Azure](/troubleshoot/azure/virtual-machines/welcome-virtual-machines).
* Compruebe los [flujos de trabajo](../automation-runbook-execution.md#job-statuses) para buscar errores. En Azure Portal, vaya a la cuenta de Automation y seleccione **Trabajos** en **Automatización de procesos**.

## <a name="scenario-my-custom-runbook-fails-to-start-or-stop-my-vms"></a><a name="custom-runbook"></a>Escenario: Mi runbook personalizado no puede iniciar o detener mis máquinas virtuales

### <a name="issue"></a>Problema

Ha creado un runbook personalizado o ha descargado uno de la Galería de PowerShell, pero no funciona correctamente.

### <a name="cause"></a>Causa

El error puede deberse a muchas causas. Vaya a la cuenta de Automation en Azure Portal y seleccione **Trabajos** en **Automatización de procesos**. En la página **Trabajos**, busque los trabajos del runbook para ver cualquier error que pueda haber en ellos.

### <a name="resolution"></a>Solución

Se recomienda:

* Usar [Start/Stop VMs during off-hours](../automation-solution-vm-management.md) para iniciar y detener las VM en Azure Automation. 
* Tener en cuenta que Microsoft no admite runbooks personalizados. Puede encontrar una solución para su runbook personalizado en [Solución de incidencias de runbooks](runbooks.md). Compruebe los [flujos de trabajo](../automation-runbook-execution.md#job-statuses) para buscar errores. 

## <a name="scenario-vms-dont-start-or-stop-in-the-correct-sequence"></a><a name="dont-start-stop-in-sequence"></a>Escenario: Las máquinas virtuales no se inician o detienen en la secuencia correcta

### <a name="issue"></a>Problema

Las máquinas virtuales habilitadas para la característica no se inician o detienen en la secuencia correcta.

### <a name="cause"></a>Causa

Este problema se debe a un etiquetado incorrecto de las VM.

### <a name="resolution"></a>Solución

Aplique los pasos siguientes para asegurarse de que la característica está habilitada correctamente:

1. Asegúrese de que todas las máquinas virtuales que se van a iniciar o detener tienen una etiqueta `sequencestart` o `sequencestop`, en función de su situación. Estas etiquetas necesitan un número entero positivo como valor. Las máquinas virtuales se procesan en orden ascendente según este valor.
1. Asegúrese de que los grupos de recursos de las máquinas virtuales que se van a iniciar o detener están en las variables `External_Start_ResourceGroupNames` o `External_Stop_ResourceGroupNames`, en función de su situación.
1. Pruebe los cambios mediante la ejecución del runbook **SequencedStartStop_Parent** con el parámetro `WHATIF` establecido en True para obtener una vista previa de dichos cambios.

## <a name="scenario-startstop-vms-during-off-hours-job-fails-with-403-forbidden-error"></a><a name="403"></a>Escenario: Error 403 Prohibido en el trabajo Start/Stop VMs during off-hours

### <a name="issue"></a>Problema

Puede encontrar trabajos con el error `403 forbidden` para los runbooks de Start/Stop VMs during off-hours.

### <a name="cause"></a>Causa

Este problema puede deberse a una cuenta de ejecución mal configurada o expirada. La causa también puede ser que la cuenta de ejecución tenga permisos inadecuados para los recursos de VM.

### <a name="resolution"></a>Solución

Para comprobar si la cuenta de ejecución está configurada correctamente, vaya a la cuenta de Automation en Azure Portal y seleccione **Cuentas de ejecución** en **Configuración de la cuenta**. Si una cuenta de ejecución se ha configurado incorrectamente o ha expirado, el estado muestra la condición.

Si la cuenta de ejecución está mal configurada, elimínela y vuelva a crearla. Para más información, consulte [Cuentas de ejecución de Azure Automation](../automation-security-overview.md#run-as-accounts).

Si expiró el certificado de la cuenta de ejecución, siga los pasos de [Renovación de certificado autofirmado](../manage-runas-account.md#cert-renewal) para renovarlo.

Si faltan permisos, consulte [Inicio rápido: Visualización de los roles asignados a un usuario mediante Azure Portal](../../role-based-access-control/check-access.md). Debe proporcionar el identificador de aplicación de la entidad de servicio usada por la cuenta de ejecución. Puede recuperar este valor si va a la cuenta de Automation en Azure Portal. Seleccione **Cuentas de ejecución** en **Configuración de la cuenta** y seleccione la cuenta de ejecución adecuada.

## <a name="scenario-my-problem-isnt-listed-here"></a><a name="other"></a>Escenario: Mi problema no aparece en esta lista

### <a name="issue"></a>Problema

Experimenta un problema u obtiene un resultado inesperado cuando usa Start/Stop VMs during off-hours que no se indica en esta página.

### <a name="cause"></a>Causa

Muchas veces los errores pueden deberse a que se usa una versión anterior y obsoleta de la característica.

> [!NOTE]
> La característica Start/Stop VMs during off-hours se ha probado con los módulos de Azure importados en su cuenta de Automation al implementar la característica en las máquinas virtuales. La característica no funciona actualmente con las versiones más recientes del módulo de Azure. Esta restricción solo afecta a la cuenta de Automation que usa para ejecutar la característica Start/Stop VMs during off-hours. Puede seguir usando las versiones más recientes del módulo de Azure en sus otras cuentas de Automation, tal como se describe en [Actualización de módulos de Azure PowerShell](../automation-update-azure-modules.md).

### <a name="resolution"></a>Solución

Para resolver muchos errores, quite y [actualice Start/Stop VMs during off-hours](../automation-solution-vm-management.md#update-the-feature). También puede comprobar los [flujos de trabajo](../automation-runbook-execution.md#job-statuses) para buscar errores. 

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece aquí o no puede resolverlo, intente obtener ayuda adicional mediante uno de los siguientes canales:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. El Soporte técnico de Azure pone en contacto a la comunidad de Azure con respuestas, soporte técnico y expertos.
* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.
