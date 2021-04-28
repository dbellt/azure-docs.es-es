---
title: Creación de runbooks modulares en Azure Automation
description: En este artículo se indica cómo crear un runbook al que llama otro runbook.
services: automation
ms.subservice: process-automation
ms.date: 01/17/2019
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 101ff9affe43dcc97de6cf5a535c82559aafeced
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834919"
---
# <a name="create-modular-runbooks"></a>Creación de runbooks modulares

Un procedimiento recomendado en Azure Automation es escribir runbooks reutilizables y modulares con una función discreta a la que llamen otros runbooks. Con frecuencia, un runbook primario llama a uno o varios runbooks secundarios para realizar la funcionalidad necesaria. 

Existen dos maneras de llamar a un runbook secundario y hay varias diferencias que debería conocer para poder determinar cuál de ellas es mejor para sus escenarios. En la siguiente tabla se resumen las diferencias entre las dos formas de llamar a un runbook desde otro.

|  | En línea | Cmdlet |
|:--- |:--- |:--- |
| **Trabajo** |Los runbooks secundarios se ejecutan en el mismo trabajo que el primario. |Se crea un trabajo independiente para el runbook secundario. |
| **Ejecución** |El runbook primario espera a que el runbook secundario se complete antes de continuar. |El runbook primario continúa inmediatamente después de que se inicie el runbook secundario *o* el runbook primario espera a que finalice el trabajo secundario. |
| **Salida** |El runbook primario puede obtener los resultados directamente del runbook secundario. |El runbook primario debe obtener los resultados directamente del trabajo de runbook secundario *o* el runbook primario puede obtener los resultados directamente del runbook secundario. |
| **Parámetros** |Los valores de los parámetros del runbook secundario se especifican por separado y pueden usar cualquier tipo de datos. |Los valores de los parámetros del runbook secundario deben combinarse en una sola tabla hash. Esta tabla hash solo puede incluir tipos de datos simples, de matriz y de objeto que usen la serialización JSON. |
| **Cuenta de Automation** |El runbook primario solo puede usar el runbook secundario en la misma cuenta de Automation. |Los runbooks primarios pueden usar un runbook secundario desde cualquier cuenta de Automation, desde la misma suscripción de Azure e incluso desde otra suscripción con la que se tenga conexión. |
| **Publicación** |El runbook secundario debe publicarse antes de publicar el runbook primario. |El runbook secundario se publica en cualquier momento antes de iniciar el runbook primario. |

## <a name="invoke-a-child-runbook-using-inline-execution"></a>Invocación de un runbook secundario mediante la ejecución insertada

Para invocar un runbook en línea desde otro runbook, use el nombre del runbook y proporcione valores para sus parámetros, igual a como usaría una actividad o un cmdlet. De esta manera, todos los runbooks de la misma cuenta de Automation estarán disponibles para que todos los demás los usen. El runbook primario espera a que el runbook secundario se complete antes de pasar a la siguiente línea y cualquier salida se devuelve directamente al elemento primario.

Al invocar un runbook en línea, este se ejecuta en el mismo trabajo que el runbook primario. No hay ninguna indicación en el historial de trabajos del runbook secundario. Todas las excepciones y salidas de flujo pertenecientes al runbook secundario se asocian al primario. Este comportamiento da lugar a que se creen menos trabajos y facilita el seguimiento y la solución de problemas.

Cuando se publica un runbook, cualquier runbook secundario al que llame deberá estar ya estar publicado. Esto se debe a que Azure Automation crea una asociación con los runbooks secundarios cuando compila un runbook. Si aún no se han publicado los runbooks secundarios, el runbook primario parece publicarse correctamente, pero genera una excepción cuando se inicia. Si esto sucediera, puede volver a publicar el runbook primario para que haga referencia correctamente a los runbooks secundarios. No es necesario que vuelva a publicar el runbook primario si cambia algún runbook secundario, porque ya se habrá creado la asociación.

Los parámetros de un runbook secundario al que se llama de modo insertado pueden ser de cualquier tipo de datos, incluidos objetos complejos. No hay ninguna [serialización JSON](start-runbooks.md#work-with-runbook-parameters), como sucede cuando se inicia el runbook mediante Azure Portal o con el cmdlet [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook).

### <a name="runbook-types"></a>Tipos de runbook

¿Qué tipos de runbook pueden llamarse entre sí?

* Un [runbook de PowerShell](automation-runbook-types.md#powershell-runbooks) y un [runbook gráfico](automation-runbook-types.md#graphical-runbooks) pueden llamarse entre sí de modo insertado, ya que ambos se basan en PowerShell.
* Un [runbook de Flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) y un runbook gráfico de Flujo de trabajo de PowerShell pueden llamarse entre sí de modo insertado, ya que ambos tipos se basan en Flujo de trabajo de PowerShell.
* Los tipos de PowerShell y los tipos de flujo de trabajo de PowerShell no pueden llamarse entre sí de modo insertado, y deben utilizar `Start-AzAutomationRunbook`.

¿Cuándo es importante el orden de publicación?

El orden de publicación de los runbooks solo es importante para los runbooks de Flujo de trabajo de PowerShell y los runbooks gráficos de Flujo de trabajo de PowerShell.

Cuando el runbook llama a un runbook secundario gráfico o de Flujo de trabajo de PowerShell con la ejecución insertada, usa el nombre del runbook. El nombre debe comenzar por `.\\` para especificar que el script se encuentra en el directorio local.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se inicia un runbook secundario de prueba que acepta un objeto complejo, un valor entero y un valor booleano. Los resultados del runbook secundario se asignan a una variable. En este caso, el runbook secundario es un runbook de flujo de trabajo de PowerShell.

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "LabRG" -Name "MyVM"
$output = PSWF-ChildRunbook -VM $vm -RepeatCount 2 -Restart $true
```

A continuación se muestra el mismo ejemplo con un runbook de PowerShell como elemento secundario.

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "LabRG" -Name "MyVM"
$output = .\PS-ChildRunbook.ps1 -VM $vm -RepeatCount 2 -Restart $true
```

## <a name="start-a-child-runbook-using-a-cmdlet"></a>Inicio de un runbook secundario mediante un cmdlet

> [!IMPORTANT]
> Si el runbook invoca un runbook secundario con el cmdlet `Start-AzAutomationRunbook` y el parámetro `Wait`, y el runbook secundario genera un resultado de objeto, es posible que se produzca un error en la operación. Para solucionar el error, consulte la información sobre [runbooks secundarios con salida de objetos](troubleshoot/runbooks.md#child-runbook-object), con el fin de aprender a implementar la lógica de sondeo en busca de resultados mediante el cmdlet [Get AzAutomationJobOutputRecord](/powershell/module/az.automation/get-azautomationjoboutputrecord).

Puede utilizar `Start-AzAutomationRunbook` para iniciar un runbook, tal como se describe en [Inicio de un runbook con PowerShell](start-runbooks.md#start-a-runbook-with-powershell). Existen dos modos de usar este cmdlet. En un modo, el cmdlet devuelve el identificador del trabajo cuando se crea el trabajo del runbook secundario. En el otro, que el script habilita mediante la especificación del parámetro *Wait*, el cmdlet espera a que finalice el trabajo secundario para devolver la salida del runbook secundario.

El trabajo de un runbook secundario que se inició con un cmdlet se ejecuta de forma independiente al trabajo del runbook primario. Como resultado de este comportamiento, habrá más trabajos que cuando se inicia el runbook insertado y será más difícil realizar un seguimiento de los trabajos. Asimismo, el primario puede iniciar varios runbooks secundarios de forma asincrónica sin tener que esperar a que se complete cada uno de ellos. Para esta ejecución en paralelo que llama a los runbooks secundarios en modo insertado, el runbook primario debe usar la [palabra clave parallel](automation-powershell-workflow.md#use-parallel-processing).

La salida del runbook secundario no se devuelve al runbook primario de forma confiable debido al tiempo. Además, las variables como `$VerbosePreference`, `$WarningPreference` y otras podrían no propagarse a los runbooks secundarios. Para evitar estos problemas, puede iniciar los runbooks secundarios como trabajos de Automation independientes mediante el cmdlet `Start-AzAutomationRunbook` con el parámetro `Wait`. Esta técnica bloquea el runbook primario hasta que se completa el runbook secundario.

Si no quiere bloquear el runbook primario durante la espera, puede iniciar el runbook secundario mediante el cmdlet `Start-AzAutomationRunbook` sin el parámetro `Wait`. En este caso, el runbook debe usar [Get-AzAutomationJob](/powershell/module/az.automation/get-azautomationjob) para esperar la finalización del trabajo. También debe usar [Get-AzAutomationJobOutput](/powershell/module/az.automation/get-azautomationjoboutput) y [Get-AzAutomationJobOutputRecord](/powershell/module/az.automation/get-azautomationjoboutputrecord) para recuperar los resultados.

Los parámetros de un runbook secundario iniciado con un cmdlet se proporcionan como una tabla hash, como se describe en [Parámetros de runbook](start-runbooks.md#work-with-runbook-parameters). Solo pueden usarse los tipos de datos simples. Si el runbook tiene un parámetro con un tipo de datos complejo, debe llamarse en línea.

Es posible que se pierda el contexto de suscripción cuando inicia runbooks secundarios como trabajos individuales. Para que el runbook secundario ejecute los cmdlets del módulo Az en una suscripción de Azure específica, el secundario debe autenticarse en esta suscripción de forma independiente al runbook primario.

Si los trabajos de una misma cuenta de Automation funcionan con varias suscripciones, la selección de una suscripción en uno de los trabajos puede cambiar el contexto de suscripción actualmente seleccionado para otros trabajos. Para evitar esta situación, use `Disable-AzContextAutosave -Scope Process` al principio de cada runbook. Esta acción solo guarda el contexto de esa ejecución de runbook.

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se inicia un runbook secundario con parámetros y se espera a que finalice mediante el cmdlet `Start-AzAutomationRunbook` con el parámetro `Wait`. Una vez completado, el ejemplo recopila la salida del cmdlet del runbook secundario. Para usar `Start-AzAutomationRunbook`, el script debe autenticarse en su suscripción de Azure.

```azurepowershell-interactive
# Ensure that the runbook does not inherit an AzContext
Disable-AzContextAutosave -Scope Process

# Connect to Azure with Run As account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Set-AzContext -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$params = @{"VMName"="MyVM";"RepeatCount"=2;"Restart"=$true}

Start-AzAutomationRunbook `
    -AutomationAccountName 'MyAutomationAccount' `
    -Name 'Test-ChildRunbook' `
    -ResourceGroupName 'LabRG' `
    -AzContext $AzureContext `
    -Parameters $params -Wait
```

## <a name="next-steps"></a>Pasos siguientes

* Para ejecutar el runbook, consulte [Inicio de un runbook en Azure Automation](start-runbooks.md).
* Para supervisar la operación del runbook, consulte [Salida y mensajes del runbook en Azure Automation](automation-runbook-output-and-messages.md).
