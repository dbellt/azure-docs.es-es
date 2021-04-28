---
title: Información sobre el flujo de trabajo de PowerShell para Azure Automation
description: En este artículo se explican las diferencias entre el flujo de trabajo de PowerShell y PowerShell y los conceptos aplicables a los runbooks de Automation.
services: automation
ms.subservice: process-automation
ms.date: 12/14/2018
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3b3beb8b3eda4dfabf9240aa328a24d5f855689b
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833515"
---
# <a name="learn-powershell-workflow-for-azure-automation"></a>Información sobre el flujo de trabajo de PowerShell para Azure Automation

Los runbooks de Azure Automation se implementan como flujos de trabajo de Windows PowerShell, scripts de Windows PowerShell que usan Windows Workflow Foundation. Un flujo de trabajo es una secuencia de pasos conectados y programados que realizan tareas de larga duración o requieren de la coordinación de pasos múltiples a través de varios dispositivos o nodos administrados. 

Aunque el flujo de trabajo esté escrito con sintaxis de Windows PowerShell y se inicie mediante Windows PowerShell, se procesa mediante Windows Workflow Foundation. Las ventajas de un flujo de trabajo en un script normal incluyen el rendimiento simultáneo de una acción en varios dispositivos y la recuperación automática en caso de errores. 

> [!NOTE]
> Un flujo de trabajo de PowerShell es muy similar a un script de Windows PowerShell, pero presenta algunas diferencias importantes que pueden resultar confusas para un nuevo usuario. Por lo tanto, se recomienda escribir los runbooks con el flujo de trabajo de PowerShell solo si hay que usar [puntos de control](#use-checkpoints-in-a-workflow). 

Para más información sobre los temas de este artículo, consulte [Introducción al flujo de trabajo de Windows PowerShell](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj134242(v=ws.11)).

## <a name="use-workflow-keyword"></a>Uso de la palabra clave workflow

El primer paso para convertir un script de PowerShell en un flujo de trabajo de PowerShell es delimitarlo con la palabra clave `Workflow`. Un flujo de trabajo comienza con la palabra clave `Workflow`, seguida del cuerpo del script entre llaves. El nombre del flujo de trabajo sigue a la palabra clave `Workflow`, tal y como se muestra en la sintaxis siguiente:

```powershell
Workflow Test-Workflow
{
    <Commands>
}
```

El nombre del flujo de trabajo debe coincidir con el nombre del runbook de Automation. Si se va a importar el runbook, el nombre de archivo debe coincidir con el nombre del flujo de trabajo y debe terminar en **.ps1**.

Para agregar parámetros al flujo de trabajo, use la palabra clave `Param` igual que lo haría en un script.

## <a name="learn-differences-between-powershell-workflow-code-and-powershell-script-code"></a>Información sobre las diferencias entre el código de flujo de trabajo de PowerShell y el código de script de PowerShell

El código de flujo de trabajo de PowerShell es casi idéntico al código de script de PowerShell salvo por algunos cambios importantes. En las secciones siguientes se describen los cambios que debe realizar en un script de PowerShell para que se ejecute en un flujo de trabajo.

### <a name="activities"></a>Actividades

Una actividad es una tarea específica en un flujo de trabajo que se realiza en una secuencia. El flujo de trabajo de Windows PowerShell convierte automáticamente muchos de los cmdlets de Windows PowerShell en actividades cuando se ejecuta un flujo de trabajo. Cuando se especifica uno de estos cmdlets en el runbook, Windows Workflow Foundation ejecuta la actividad correspondiente. 

Para los cmdlets sin actividad correspondiente, el flujo de trabajo de Windows PowerShell ejecuta automáticamente el cmdlet en una actividad [InlineScript](#use-inlinescript). Algunos cmdlets están excluidos y no se pueden usar en un flujo de trabajo a menos que se incluyan explícitamente en un bloque de InlineScript. Para más información, consulte [Uso de actividades en flujos de trabajo de script](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj574194(v=ws.11)).

Las actividades de flujo de trabajo comparten un conjunto de parámetros comunes para configurar su funcionamiento. Consulte [about_WorkflowCommonParameters](/powershell/module/psworkflow/about/about_workflowcommonparameters).

### <a name="positional-parameters"></a>Parámetros posicionales

No puede usar parámetros posicionales con actividades y cmdlets en un flujo de trabajo. Por lo tanto, debe usar nombres de parámetro. Considere el siguiente código que obtiene todos los servicios en ejecución:

```azurepowershell-interactive
Get-Service | Where-Object {$_.Status -eq "Running"}
```

Si intenta ejecutar este código en un flujo de trabajo, recibirá un mensaje similar a `Parameter set cannot be resolved using the specified named parameters.`. Para corregir este problema, proporcione el nombre del parámetro, como en el ejemplo siguiente:

```powershell
Workflow Get-RunningServices
{
    Get-Service | Where-Object -FilterScript {$_.Status -eq "Running"}
}
```

### <a name="deserialized-objects"></a>Objetos deserializados

Los objetos de los flujos de trabajo se deserializan, lo que significa que sus propiedades siguen estando disponibles, pero no sus métodos.  Por ejemplo, considere el siguiente código de PowerShell, que detiene un servicio con el método `Stop` del objeto `Service`.

```azurepowershell-interactive
$Service = Get-Service -Name MyService
$Service.Stop()
```

Si intenta ejecutarlo en un flujo de trabajo, recibirá un error que indica `Method invocation is not supported in a Windows PowerShell Workflow.`.

Una opción es ajustar estas dos líneas de código en un bloque de [InlineScript](#use-inlinescript). En este caso, `Service` representa un objeto de servicio dentro del bloque.

```powershell
Workflow Stop-Service
{
    InlineScript {
        $Service = Get-Service -Name MyService
        $Service.Stop()
    }
}
```

Otra opción es usar otro cmdlet que tenga la misma funcionalidad que el método, si hay uno disponible. En nuestro ejemplo, el cmdlet `Stop-Service` proporciona la misma funcionalidad que el método `Stop`, y se podría usar el siguiente código para un flujo de trabajo.

```powershell
Workflow Stop-MyService
{
    $Service = Get-Service -Name MyService
    Stop-Service -Name $Service.Name
}
```

## <a name="use-inlinescript"></a>Uso de InlineScript

La actividad `InlineScript` es útil cuando necesita ejecutar uno o más comandos como un script tradicional de PowerShell en lugar de como un flujo de trabajo de PowerShell.  Mientras que los comandos de un flujo de trabajo se envían a Windows Workflow Foundation para su procesamiento, los comandos de un bloque de InlineScript se procesan mediante Windows PowerShell.

InlineScript usa la sintaxis que se muestra a continuación.

```powershell
InlineScript
{
    <Script Block>
} <Common Parameters>
```

Puede devolver resultados de InlineScript asignando el resultado a una variable. El siguiente ejemplo detiene un servicio y, a continuación, envía el nombre de servicio.

```powershell
Workflow Stop-MyService
{
    $Output = InlineScript {
        $Service = Get-Service -Name MyService
        $Service.Stop()
        $Service
    }

    $Output.Name
}
```

Puede pasar valores en un bloque de InlineScript, pero debe usar el modificador de ámbito **$Using** .  El ejemplo siguiente es idéntico al ejemplo anterior salvo que se proporciona el nombre de servicio mediante una variable.

```powershell
Workflow Stop-MyService
{
    $ServiceName = "MyService"

    $Output = InlineScript {
        $Service = Get-Service -Name $Using:ServiceName
        $Service.Stop()
        $Service
    }

    $Output.Name
}
```

Aunque las actividades InlineScript puedan ser críticas en determinados flujos de trabajo, no admiten construcciones de flujo de trabajo. Solo debe usarlos cuando sea necesario por los siguientes motivos:

* No puede usar [puntos de control](#use-checkpoints-in-a-workflow) dentro de un bloque de InlineScript. Si se produce un error dentro del bloque, se debe reanudar desde el principio del bloque.
* No puede usar la [ejecución en paralelo](#use-parallel-processing) dentro de un bloque de InlineScript.
* InlineScript afecta a la escalabilidad del flujo de trabajo, ya que retiene la sesión de Windows PowerShell durante todo el bloque de InlineScript.

Para más información sobre el uso de InlineScript, vea [Ejecutar comandos de Windows PowerShell en un flujo de trabajo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj574197(v=ws.11)) y [about_InlineScript](/powershell/module/psworkflow/about/about_inlinescript).

## <a name="use-parallel-processing"></a>Uso del procesamiento paralelo

Una ventaja de los flujos de trabajo de Windows PowerShell es la capacidad para realizar un conjunto de comandos en paralelo en lugar de hacerlo secuencialmente como con un script típico.

Puede usar la palabra clave `Parallel` para crear un bloque de scripts con varios comandos que se ejecutan simultáneamente. Esto usa la siguiente sintaxis que se muestra a continuación. En este caso, Activity1 y Activity2 se inician al mismo tiempo. Activity3 se inicia después de que se hayan completado Activity1 y Activity2.

```powershell
Parallel
{
    <Activity1>
    <Activity2>
}
<Activity3>
```

Por ejemplo, considere los siguientes comandos de PowerShell que copian varios archivos a un destino de red. Estos comandos se ejecutan secuencialmente de modo que un archivo debe terminar de copiarse antes de que comience el siguiente.

```azurepowershell-interactive
Copy-Item -Path C:\LocalPath\File1.txt -Destination \\NetworkPath\File1.txt
Copy-Item -Path C:\LocalPath\File2.txt -Destination \\NetworkPath\File2.txt
Copy-Item -Path C:\LocalPath\File3.txt -Destination \\NetworkPath\File3.txt
```

El flujo de trabajo siguiente ejecuta estos mismos comandos en paralelo para que todos ellos empiezan a copiarse al mismo tiempo.  Una vez que todos se han copiado, se muestra el mensaje de finalización.

```powershell
Workflow Copy-Files
{
    Parallel
    {
        Copy-Item -Path "C:\LocalPath\File1.txt" -Destination "\\NetworkPath"
        Copy-Item -Path "C:\LocalPath\File2.txt" -Destination "\\NetworkPath"
        Copy-Item -Path "C:\LocalPath\File3.txt" -Destination "\\NetworkPath"
    }

    Write-Output "Files copied."
}
```

Puede utilizar la construcción `ForEach -Parallel` para procesar comandos para cada elemento de una colección simultáneamente. Los elementos de la colección se procesan en paralelo mientras que los comandos del bloque de scripts se ejecutan secuencialmente. Este proceso usa la siguiente sintaxis que se muestra a continuación. En este caso, Activity1 se inicia al mismo tiempo para todos los elementos de la colección. Para cada elemento, Activity2 se inicia una vez completado Activity1. Activity3 se inicia únicamente después de que se hayan completado Activity1 y Activity2 para todos los elementos. Se usa el parámetro `ThrottleLimit` para limitar el paralelismo. Un valor `ThrottleLimit` demasiado alto puede causar problemas. El valor ideal para el parámetro `ThrottleLimit` depende de muchos factores del entorno. Empiece con un valor bajo y pruebe distintos valores crecientes hasta que encuentre uno que funcione para su caso concreto.

```powershell
ForEach -Parallel -ThrottleLimit 10 ($<item> in $<collection>)
{
    <Activity1>
    <Activity2>
}
<Activity3>
```

El ejemplo siguiente es similar al ejemplo anterior, en el que se copian archivos en paralelo.  En este caso, se muestra un mensaje para cada archivo después de copiarse.  Una vez que todos se han copiado, se muestra el mensaje de finalización.

```powershell
Workflow Copy-Files
{
    $files = @("C:\LocalPath\File1.txt","C:\LocalPath\File2.txt","C:\LocalPath\File3.txt")

    ForEach -Parallel -ThrottleLimit 10 ($File in $Files)
    {
        Copy-Item -Path $File -Destination \\NetworkPath
        Write-Output "$File copied."
    }

    Write-Output "All files copied."
}
```

> [!NOTE]
> No se recomienda ejecutar runbooks secundarios en paralelo, ya que se ha demostrado que no tiene unos resultados confiables. A veces el resultado del runbook secundario no se muestra, y la configuración de un runbook secundario puede afectar a los demás runbooks secundarios paralelos. Variables tales como `VerbosePreference`, `WarningPreference` y otras podrían no propagarse a los runbooks secundarios. Y si el runbook secundario cambia estos valores, es posible que no se restauren correctamente después de su invocación.

## <a name="use-checkpoints-in-a-workflow"></a>Uso de puntos de control en un flujo de trabajo

Un punto de control es una instantánea del estado actual del flujo de trabajo que incluye el valor actual de las variables y cualquier salida generada en ese punto. Si un flujo de trabajo termina en error o se suspende, comienza desde su último punto de control la próxima vez que se ejecute, en lugar de comenzar desde el principio. 

Puede establecer un punto de control en un flujo de trabajo con la actividad `Checkpoint-Workflow`. Azure Automation tiene una característica denominada [distribución equilibrada](automation-runbook-execution.md#fair-share), por la que se descarga cualquier runbook que se ejecute durante tres horas para permitir la ejecución de otros runbooks. Finalmente, se recarga el runbook descargado. Cuando es así, se reanuda la ejecución desde el último punto de control tomado en el runbook.

Con el fin de garantizar que el runbook finalmente se complete, debe agregar puntos de control en intervalos que se ejecuten durante menos de tres horas. Si durante cada ejecución se agrega un nuevo punto de control, y si el runbook se expulsa después de tres horas debido a un error, el runbook se reanudará indefinidamente.

En el ejemplo siguiente, se produce una excepción después de Activity2, lo que provoca la finalización del flujo de trabajo. Cuando se vuelve a ejecutar el flujo de trabajo, se inicia ejecutando Activity2, ya que estaba inmediatamente después del último punto de control establecido.

```powershell
<Activity1>
Checkpoint-Workflow
<Activity2>
<Exception>
<Activity3>
```

Establezca puntos de control en un flujo de trabajo después de actividades que puedan ser propensas a la excepción y que no deben repetirse si se reanuda el flujo de trabajo. Por ejemplo, el flujo de trabajo puede crear una máquina virtual. Puede establecer un punto de control antes y después de los comandos para crear la máquina virtual. Si se produce un error en la creación, los comandos se repiten si el flujo de trabajo se vuelve a iniciar. Si el flujo de trabajo produce un error después de que la creación se realice correctamente, la máquina virtual no se vuelve a crear cuando se reanude el flujo de trabajo.

El siguiente ejemplo copia varios archivos en una ubicación de red y establece un punto de comprobación después de cada archivo.  Si la ubicación de red se pierde, el flujo de trabajo finaliza con error.  Cuando se vuelve a iniciar, se reanuda en el último punto de control. Solo se omiten los archivos que ya se han copiado.

```powershell
Workflow Copy-Files
{
    $files = @("C:\LocalPath\File1.txt","C:\LocalPath\File2.txt","C:\LocalPath\File3.txt")

    ForEach ($File in $Files)
    {
        Copy-Item -Path $File -Destination \\NetworkPath
        Write-Output "$File copied."
        Checkpoint-Workflow
    }

    Write-Output "All files copied."
}
```

Como las credenciales de nombre de usuario no se conservan después de llamar a la actividad [Suspend-Workflow](/powershell/module/psworkflow/about/about_suspend-workflow) o después del último punto de control, tiene que establecer las credenciales en NULL y, después, recuperarlas de nuevo desde el almacén de recursos tras llamar a `Suspend-Workflow` o al punto de control.  En caso contrario, es posible que reciba el siguiente mensaje de error: `The workflow job cannot be resumed, either because persistence data could not be saved completely, or saved persistence data has been corrupted. You must restart the workflow.`.

El mismo código de abajo muestra cómo controlar esta operación en los runbooks del flujo de trabajo de PowerShell.

```powershell
workflow CreateTestVms
{
    $Cred = Get-AzAutomationCredential -Name "MyCredential"
    $null = Connect-AzAccount -Credential $Cred

    $VmsToCreate = Get-AzAutomationVariable -Name "VmsToCreate"

    foreach ($VmName in $VmsToCreate)
        {
        # Do work first to create the VM (code not shown)

        # Now add the VM
        New-AzVM -VM $Vm -Location "WestUs" -ResourceGroupName "ResourceGroup01"

        # Checkpoint so that VM creation is not repeated if workflow suspends
        $Cred = $null
        Checkpoint-Workflow
        $Cred = Get-AzAutomationCredential -Name "MyCredential"
        $null = Connect-AzAccount -Credential $Cred
        }
}
```

> [!NOTE]
> En el caso de los runbooks de PowerShell no gráficos, `Add-AzAccount` y `Add-AzureRMAccount` son alias de [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Puede usar estos cmdlets o bien [actualizar los módulos](automation-update-azure-modules.md) de la cuenta de Automation a las versiones más recientes. Es posible que deba actualizar los módulos incluso si acaba de crear una nueva cuenta de Automation. El uso de estos cmdlets no es necesario si se autentica utilizando una cuenta de ejecución configurada con una entidad de servicio.

Para obtener más información acerca de los puntos de control, consulte [Adición de puntos de control a un flujo de trabajo de scripts](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj574114(v=ws.11)).

## <a name="next-steps"></a>Pasos siguientes

* Para mas información sobre los runbooks de flujo de trabajo de PowerShell, consulte [Tutorial: Creación de un runbook de flujo de trabajo de PowerShell](learn/automation-tutorial-runbook-textual.md).
