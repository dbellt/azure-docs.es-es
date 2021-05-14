---
title: Preguntas más frecuentes sobre Azure Automation | Microsoft Docs
description: En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre Azure Automation.
services: automation
ms.subservice: ''
ms.topic: conceptual
ms.date: 12/17/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9ab4ce9b8691f27fe392d2e3099677d45900d2e3
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834235"
---
# <a name="azure-automation-frequently-asked-questions"></a>Preguntas más frecuentes sobre Azure Automation

En este artículo de preguntas más frecuentes de Microsoft, se presenta una lista con las preguntas frecuentes sobre Azure Automation. Si tiene alguna otra pregunta sobre sus funcionalidades, vaya al foro de discusión y publíquela. Si una pregunta es frecuente, se agrega a este artículo para que se encuentre de forma rápida y sencilla.

## <a name="update-management"></a>Administración de actualizaciones

### <a name="can-i-prevent-unexpected-os-level-upgrades"></a>¿Puedo evitar actualizaciones inesperadas de nivel de sistema operativo?

En algunas variantes de Linux, como Red Hat Enterprise Linux, las actualizaciones de nivel de sistema operativo pueden producirse mediante paquetes. Esto puede llevar a ejecuciones de Update Management en las que cambia el número de versión del sistema operativo. Como Update Management usa los mismos métodos para actualizar paquetes que un administrador utiliza localmente en la máquina Linux, este comportamiento es deliberado.

Para evitar la actualización de la versión del sistema operativo con las implementaciones de Update Management, use la característica **Exclusión**.

En Red Hat Enterprise Linux, el nombre del paquete para excluir es `redhat-release-server.x86_64`.

### <a name="why-arent-criticalsecurity-updates-applied"></a>¿Por qué no se aplican las actualizaciones de seguridad o críticas?

Al implementar actualizaciones en una máquina Linux, puede seleccionar clasificaciones de actualizaciones. Esta opción filtra las actualizaciones que cumplen los criterios especificados. Este filtro se aplica localmente en el equipo cuando se implementa la actualización.

Dado que Update Management realiza el enriquecimiento de actualizaciones en la nube, puede marcar algunas actualizaciones en Update Management como de impacto para la seguridad, aunque la máquina local no tenga esa información. Si aplica actualizaciones críticas a una máquina Linux, puede haber actualizaciones que no estén marcadas como de impacto para la seguridad en esa máquina y que, por lo tanto, no se apliquen. Sin embargo, es posible que Update Management todavía notifique que esa máquina no es compatible porque tiene información adicional acerca de la actualización pertinente.

La implementación de actualizaciones mediante la clasificación de actualizaciones no funciona en las versiones RTM de CentOS. Para implementar correctamente actualizaciones para CentOS, seleccione todas las clasificaciones para asegurarse de que se aplican las actualizaciones. Para SUSE, al seleccionar SOLO **Otras actualizaciones** como clasificación, puede que se instalen algunas actualizaciones de seguridad si están relacionadas con zypper (Administrador de paquetes) o se requieren sus dependencias en primer lugar. Este comportamiento es una limitación de zypper. En algunos casos, puede que se le pida volver a ejecutar la implementación de actualizaciones y, a continuación, comprobar la implementación a través del registro de actualizaciones.

### <a name="can-i-deploy-updates-across-azure-tenants"></a>¿Puedo implementar actualizaciones en los inquilinos de Azure?

Si tiene máquinas que necesitan revisión en otro inquilino de Azure que informan a Update Management, tiene que usar una de las soluciones alternativas siguientes para programarlas. Puede usar el cmdlet [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) con el parámetro `ForUpdateConfiguration` para crear una programación. Puede usar el cmdlet [New-AzAutomationSoftwareUpdateConfiguration](/powershell/module/Az.Automation/New-AzAutomationSoftwareUpdateConfiguration) y pasar las máquinas del otro inquilino al parámetro `NonAzureComputer`. El ejemplo siguiente muestra cómo hacerlo.

```azurepowershell-interactive
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$sched = New-AzAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdateConfiguration

New-AzAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName <automationAccountName> -Schedule $sched -Windows -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

## <a name="process-automation---python-runbooks"></a>Automatización de procesos: runbooks de Python

### <a name="which-python-3-version-is-supported-in-azure-automation"></a>¿Qué versión de Python 3 se admite en Azure Automation?

En el caso de los trabajos en la nube, se admite Python 3.8. Es posible que los scripts y paquetes de cualquier versión 3.x funcionen si el código es compatible con distintas versiones.

En el caso de los trabajos híbridos de Hybrid Runbook Worker para Windows, puede optar por instalar cualquier versión 3.x que desee usar. En el caso de los trabajos híbridos de Hybrid Runbook Worker para Linux, dependemos de la versión de Python 3 instalada en el equipo para ejecutar DSC OMSConfig y Hybrid Worker para Linux. Se recomienda instalar la versión 3.6; sin embargo, las distintas versiones también deben funcionar si no hay cambios importantes en los contratos o las firmas de método entre las versiones de Python 3.

### <a name="can-python-2-and-python-3-runbooks-run-in-same-automation-account"></a>¿Pueden los runbooks de Python 2 y Python 3 ejecutarse en la misma cuenta de automatización?

Sí, no hay ninguna limitación para usar runbooks de Python 2 y Python 3 en la misma cuenta de automatización.  

### <a name="what-is-the-plan-for-migrating-existing-python-2-runbooks-and-packages-to-python-3"></a>¿Cuál es el plan para migrar los paquetes y runbooks existentes de Python 2 a Python 3?

Azure Automation no pretende migrar los runbooks y paquetes de Python 2 a Python 3. Tendrá que realizar esta migración usted mismo. Los runbooks y paquetes nuevos y existentes de Python 2 seguirán funcionando.

### <a name="what-are-the-packages-supported-by-default-in-python-3-environment"></a>¿Cuáles son los paquetes admitidos de forma predeterminada en el entorno de Python 3?

Azure Package 4.0.0 está instalado de forma predeterminada en el entorno de Automation de Python 3. Puede importar manualmente una versión superior del paquete de Azure para invalidar la versión predeterminada.

### <a name="what-if-i-run-a-python-3-runbook-that-references-a-python-2-package-or-vice-versa"></a>¿Qué ocurre si ejecuto un runbook de Python 3 que haga referencia a un paquete de Python 2 o viceversa?

Python 2 y Python 3 tienen entornos de ejecución diferentes. Mientras se ejecuta un runbook de Python 2, solo se pueden importar paquetes de Python 2 y similares para Python 3.

### <a name="how-do-i-differentiate-between-python-2-and-python-3-runbooks-and-packages"></a>¿Cómo se diferencian los paquetes y runbooks de Python 2 y Python 3?

Python 3 es una nueva definición de runbook, que distingue entre los runbooks de Python 2 y Python 3. Del mismo modo, se introduce otro tipo de paquete para los paquetes de Python 3.

## <a name="next-steps"></a>Pasos siguientes

Si su pregunta no se ha respondido aquí, puede consultar las fuentes siguientes para preguntas y respuestas adicionales.

- [Azure Automation](/answers/topics/azure-automation.html)
- [Foro de comentarios](https://feedback.azure.com/forums/905242-update-management)
