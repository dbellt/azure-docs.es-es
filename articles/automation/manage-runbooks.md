---
title: Administración de runbooks en Azure Automation
description: En este artículo se explica cómo administrar runbooks en Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 04/22/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0226b864ae2f23c8cfaa1e69b4dd31f2800ce171
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164298"
---
# <a name="manage-runbooks-in-azure-automation"></a>Administración de runbooks en Azure Automation

Para agregar un runbook a Azure Automation, puede crear uno o importar uno existente desde un archivo o desde la [Galería de runbooks](automation-runbook-gallery.md). En este artículo se proporciona información para administrar un runbook importado desde un archivo. Puede encontrar todos los detalles sobre el acceso a los módulos y los runbooks de la comunidad en [Galerías de runbooks y módulos para Azure Automation](automation-runbook-gallery.md).

## <a name="create-a-runbook"></a>Crear un runbook

Cree un runbook nuevo en Azure Automation mediante Azure Portal o Windows PowerShell. Una vez creado el runbook, se puede editar siguiendo la información de las secciones siguientes:

* [Edición de runbooks de texto en Azure Automation](automation-edit-textual-runbook.md)
* [Aprendizaje de los conceptos clave del flujo de trabajo de Windows PowerShell para los runbooks de Automation](automation-powershell-workflow.md)
* [Administración de paquetes de Python 2 en Azure Automation](python-packages.md)
* [Administración de paquetes de Python 3 (versión preliminar) en Azure Automation](python-3-packages.md)

### <a name="create-a-runbook-in-the-azure-portal"></a>Creación de un runbook en Azure Portal

1. En Azure Portal, abra su cuenta de Automation.
2. Desde el centro de conectividad, seleccione **Runbooks** en **Automatización de procesos** para abrir la lista de runbooks.
3. Haga clic en **Crear un runbook**.
4. Escriba un nombre para el runbook y seleccione su [tipo](automation-runbook-types.md). El nombre del runbook debe empezar por una letra y puede tener letras, números, guiones bajos y guiones.
5. Haga clic en **Crear** para crear el runbook y abra el editor.

### <a name="create-a-runbook-with-powershell"></a>Creación de un runbook con PowerShell

Use el cmdlet [New-AzAutomationRunbook](/powershell/module/az.automation/new-azautomationrunbook) para crear un runbook vacío. Use el parámetro `Type` para especificar uno de los tipos de runbook definidos para `New-AzAutomationRunbook`.

En el ejemplo siguiente se muestra cómo crear un nuevo runbook vacío.

```azurepowershell-interactive
$params = @{
    AutomationAccountName = 'MyAutomationAccount'
    Name                  = 'NewRunbook'
    ResourceGroupName     = 'MyResourceGroup'
    Type                  = 'PowerShell'
}
New-AzAutomationRunbook @params
```

## <a name="import-a-runbook"></a>Importación de un runbook

Puede importar un script de PowerShell o de flujo de trabajo de PowerShell ( **.ps1**), un runbook gráfico ( **.graphrunbook**) o un script de Python 2 o Python 3 ( **.py**) para crear su propio runbook. Necesita especificar el [tipo de runbook](automation-runbook-types.md) que se creará durante la importación teniendo en cuenta las consideraciones siguientes.

* Puede importar un archivo **.ps1** que no contenga un flujo de trabajo, ya sea en un [runbook de PowerShell](automation-runbook-types.md#powershell-runbooks) o en un [runbook de flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks). Si lo importa en un runbook de flujo de trabajo de PowerShell, se convertirá en un flujo de trabajo. En este caso, se incluyen comentarios en el runbook para describir los cambios realizados.

* Solo puede importar un archivo **.ps1** que contiene un flujo de trabajo de PowerShell en un [runbook de flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks). Si el archivo contiene varios flujos de trabajo de PowerShell, se produce un error en la importación. Debe guardar cada flujo de trabajo en su propio archivo e importar cada uno por separado.

* No debe importar un archivo **.ps1** que contiene un flujo de trabajo de PowerShell en un [runbook de PowerShell](automation-runbook-types.md#powershell-runbooks), ya que el motor de scripts de PowerShell no lo reconoce.

* Importe solo un archivo **.graphrunbook** en un [runbook gráfico](automation-runbook-types.md#graphical-runbooks) nuevo.

### <a name="import-a-runbook-from-the-azure-portal"></a>Importación de un runbook desde Azure Portal

Puede usar el siguiente procedimiento para importar un archivo de script en Azure Automation.

> [!NOTE]
> Solo puede importar un archivo **.ps1** en un runbook de flujo de trabajo de PowerShell mediante el portal.

1. En Azure Portal, abra su cuenta de Automation.
2. En **Automatización de procesos**, haga clic en **Runbooks** para abrir la lista de runbooks.
3. Haga clic en **Importar un runbook**.
4. Haga clic en **Archivo de runbook** y seleccione el archivo que se va a importar.
5. Si el campo **Nombre** está habilitado, tiene la opción de cambiar el nombre del runbook. El nombre debe empezar por una letra y puede tener letras, números, guiones bajos y guiones.
6. El [tipo de runbook](automation-runbook-types.md) se seleccionará automáticamente, pero puede cambiarlo después de tomar en cuenta las restricciones aplicables.
7. Haga clic en **Crear**. El runbook nuevo aparece en la lista de runbooks de la cuenta de Automation.
8. Debe [publicar el runbook](#publish-a-runbook) para poder ejecutarlo.

> [!NOTE]
> Después de importar un runbook gráfico, puede convertirlo en otro tipo, pero no puede convertir un runbook gráfico en un runbook de texto.

### <a name="import-a-runbook-with-windows-powershell"></a>Importación de un runbook con Windows PowerShell

Use el cmdlet [Import-AzAutomationRunbook](/powershell/module/az.automation/import-azautomationrunbook) para importar un archivo de script como un runbook en borrador. Si ya existe el runbook, se producirá un error de importación, a menos que utilice el parámetro `Force` con el cmdlet.

En los ejemplos siguientes se muestra cómo importar un archivo de script en un runbook.

```azurepowershell-interactive
$params = @{
    AutomationAccountName = 'MyAutomationAccount'
    Name                  = 'Sample_TestRunbook'
    ResourceGroupName     = 'MyResourceGroup'
    Type                  = 'PowerShell'
    Path                  = 'C:\Runbooks\Sample_TestRunbook.ps1'
}
Import-AzAutomationRunbook @params
```

## <a name="handle-resources"></a>Control de recursos

Si el runbook crea un [recurso](automation-runbook-execution.md#resources), el script debe comprobar que el recurso ya exista antes de intentar crearlo. Este es un ejemplo básico.

```powershell
$vmName = 'WindowsVM1'
$rgName = 'MyResourceGroup'
$myCred = Get-AutomationPSCredential 'MyCredential'

$vmExists = Get-AzResource -Name $vmName -ResourceGroupName $rgName
if (-not $vmExists) {
    Write-Output "VM $vmName does not exist, creating"
    New-AzVM -Name $vmName -ResourceGroupName $rgName -Credential $myCred
} else {
    Write-Output "VM $vmName already exists, skipping"
}
```

## <a name="retrieve-details-from-activity-log"></a>Recuperación de detalles del registro de actividad

Puede recuperar detalles del runbook, como la persona o la cuenta que ha iniciado un runbook, del [registro de actividad](automation-runbook-execution.md#activity-logging) para la cuenta de Automation. En el siguiente ejemplo de PowerShell se proporciona el último usuario que ha ejecutado el runbook especificado.

```powershell-interactive
$SubID = '00000000-0000-0000-0000-000000000000'
$AutoRgName = 'MyResourceGroup'
$aaName = 'MyAutomationAccount'
$RunbookName = 'MyRunbook'
$StartTime = (Get-Date).AddDays(-1)

$params = @{
    ResourceGroupName = $AutoRgName
    StartTime         = $StartTime
}
$JobActivityLogs = (Get-AzLog @params).Where( { $_.Authorization.Action -eq 'Microsoft.Automation/automationAccounts/jobs/write' })

$JobInfo = @{}
foreach ($log in $JobActivityLogs) {
    # Get job resource
    $JobResource = Get-AzResource -ResourceId $log.ResourceId

    if ($null -eq $JobInfo[$log.SubmissionTimestamp] -and $JobResource.Properties.Runbook.Name -eq $RunbookName) {
        # Get runbook
        $jobParams = @{
            ResourceGroupName     = $AutoRgName
            AutomationAccountName = $aaName
            Id                    = $JobResource.Properties.JobId
        }
        $Runbook = Get-AzAutomationJob @jobParams | Where-Object RunbookName -EQ $RunbookName

        # Add job information to hashtable
        $JobInfo.Add($log.SubmissionTimestamp, @($Runbook.RunbookName,$Log.Caller, $JobResource.Properties.jobId))
    }
}
$JobInfo.GetEnumerator() | Sort-Object Key -Descending | Select-Object -First 1
```

## <a name="track-progress"></a>Seguimiento del progreso

Se recomienda crear los runbooks para que tengan una naturaleza modular, con una lógica que se pueda reutilizar y reiniciar fácilmente. El seguimiento del progreso en un runbook garantiza que la lógica de dicho runbook se ejecute correctamente si hay incidencias.

Puede realizar un seguimiento del progreso de un runbook mediante un origen externo, como una cuenta de almacenamiento, una base de datos o archivos compartidos. Cree lógica en el runbook para comprobar primero el estado de la última acción que se ha realizado. Luego, en función de los resultados de la comprobación, la lógica puede omitir o continuar tareas específicas del runbook.

## <a name="prevent-concurrent-jobs"></a>Evitación de trabajos simultáneos

Algunos runbooks se comportan de forma extraña si se ejecutan en varios trabajos al mismo tiempo. En este caso, es importante que un runbook implemente la lógica para establecer si ya hay un trabajo en ejecución. Este es un ejemplo básico.

```powershell
# Authenticate to Azure
$connection = Get-AutomationConnection -Name AzureRunAsConnection
$cnParams = @{
    ServicePrincipal      = $true
    Tenant                = $connection.TenantId
    ApplicationId         = $connection.ApplicationId
    CertificateThumbprint = $connection.CertificateThumbprint
}
Connect-AzAccount @cnParams
$AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID

# Check for already running or new runbooks
$runbookName = "<RunbookName>"
$rgName = "<ResourceGroupName>"
$aaName = "<AutomationAccountName>"
$jobs = Get-AzAutomationJob -ResourceGroupName $rgName -AutomationAccountName $aaName -RunbookName $runbookName -AzContext $AzureContext

# Check to see if it is already running
$runningCount = ($jobs.Where( { $_.Status -eq 'Running' })).count

if (($jobs.Status -contains 'Running' -and $runningCount -gt 1 ) -or ($jobs.Status -eq 'New')) {
    # Exit code
    Write-Output "Runbook [$runbookName] is already running"
    exit 1
} else {
    # Insert Your code here
}
```
También puede usar la característica de extensión de PowerShell para pasar la información de la conexión a `Connect-AzAccount`. En ese caso, las primeras líneas del ejemplo anterior tendrían este aspecto.

```powershell
# Authenticate to Azure
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount @connection
$AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID
```

Consulte [Acerca de la expansión](/powershell/module/microsoft.powershell.core/about/about_splatting) para obtener más información.

## <a name="handle-transient-errors-in-a-time-dependent-script"></a>Control de errores transitorios en un script dependiente del tiempo

Los runbooks deben ser sólidos y capaces de controlar los [errores](automation-runbook-execution.md#errors), incluidos los transitorios, que pueden provocar que se reinicien o fallen. Si se produce un error en un runbook, Azure Automation vuelve a intentarlo.

Si el runbook se ejecuta normalmente dentro de una restricción de tiempo, haga que el script implemente la lógica para comprobar el tiempo de ejecución. Esta comprobación garantiza la ejecución de operaciones, como el inicio, el apagado o el escalado horizontal, solo durante horas específicas.

> [!NOTE]
> La hora local en el proceso de espacio aislado de Azure se establece en UTC. Debe tenerse en cuenta este aspecto en los cálculos de fecha y hora de los runbooks.

## <a name="work-with-multiple-subscriptions"></a>Trabajo con varias suscripciones

El runbook debe poder trabajar con [suscripciones](automation-runbook-execution.md#subscriptions). Por ejemplo, para controlar varias suscripciones, el runbook debe usar el cmdlet [Disable-AzContextAutosave](/powershell/module/Az.Accounts/Disable-AzContextAutosave). Este cmdlet garantiza que no se recupere el contexto de autenticación desde otro runbook que se ejecute en el mismo espacio aislado. El runbook también usa el cmdlet `Get-AzContext` para recuperar el contexto de la sesión actual y asignarlo a la variable `$AzureContext`.

```powershell
Disable-AzContextAutosave -Scope Process

$connection = Get-AutomationConnection -Name AzureRunAsConnection
$cnParams = @{
    ServicePrincipal      = $true
    Tenant                = $connection.TenantId
    ApplicationId         = $connection.ApplicationId
    CertificateThumbprint = $connection.CertificateThumbprint
}
Connect-AzAccount @cnParams

$ChildRunbookName = 'ChildRunbookDemo'
$aaName = 'MyAutomationAccount'
$rgName = 'MyResourceGroup'

$startParams = @{
    ResourceGroupName     = $rgName
    AutomationAccountName = $aaName
    Name                  = $ChildRunbookName
    DefaultProfile        = $AzureContext
}
Start-AzAutomationRunbook @startParams
```

## <a name="work-with-a-custom-script"></a>Trabajo con un script personalizado

> [!NOTE]
> Normalmente no se pueden ejecutar scripts y runbooks personalizados en el host con un agente de Log Analytics instalado.

Para utilizar un script personalizado, haga lo siguiente:

1. Cree una cuenta de Automation y obtenga un [rol de colaborador](automation-role-based-access-control.md).
2. [Vincule la cuenta al área de trabajo de Azure](../security-center/security-center-enable-data-collection.md).
3. Habilite [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md), [Update Management](./update-management/overview.md) u otra característica de Automation. 
4. Si se trata de una máquina Linux, necesitará permisos elevados. Inicie sesión en [desactivar las comprobaciones de firmas](automation-linux-hrw-install.md#turn-off-signature-validation).

## <a name="test-a-runbook"></a>Prueba de un runbook

Cuando se prueba un runbook, se ejecuta la [versión de borrador](#publish-a-runbook) y se completan todas las acciones que realiza. No se crea ningún historial de trabajos, pero los flujos [salida](automation-runbook-output-and-messages.md#use-the-output-stream) y [advertencia y error](automation-runbook-output-and-messages.md#working-with-message-streams) se muestran en el panel de salida de la prueba. Los mensajes del [flujo detallado](automation-runbook-output-and-messages.md#write-output-to-verbose-stream) solo se muestran en el panel de salida si la variable [VerbosePreference](automation-runbook-output-and-messages.md#work-with-preference-variables) está establecida en `Continue`.

Aun cuando se ejecuta la versión de borrador, el runbook se ejecuta con normalidad y realiza las acciones correspondientes en los recursos del entorno. Por este motivo, solo debe probar runbooks en recursos no pertenecientes a entornos de producción.

El procedimiento para probar cada [tipo de runbook](automation-runbook-types.md) es el mismo. No hay diferencias entre realizar las pruebas en el editor de texto o en el editor gráfico de Azure Portal.

1. Abra la versión de borrador del runbook en el [editor de texto](automation-edit-textual-runbook.md) o el [editor gráfico](automation-graphical-authoring-intro.md).
1. Haga clic en **Prueba** para abrir la página de prueba.
1. Si el runbook tiene parámetros, se enumeran en el panel izquierdo, donde puede proporcionar los valores que se usarán para la prueba.
1. Si quiere ejecutar la prueba en una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md), cambie los **Parámetros de ejecución** a **Hybrid Worker** y seleccione el nombre del grupo de destino.  De lo contrario, mantenga el valor predeterminado **Azure** para ejecutar la prueba en la nube.
1. Haga clic en **Iniciar** para iniciar la prueba.
1. Puede usar los botones en el panel de salida para detener o suspender un runbook de [flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) o [gráfico](automation-runbook-types.md#graphical-runbooks) mientras se está probando. Cuando se suspende el runbook, este completa la actividad que está realizando en ese momento antes de suspenderse. Una vez suspendido, puede detener o reiniciar el runbook.
1. Inspeccione la salida del runbook en el panel de salida.

## <a name="publish-a-runbook"></a>Publicación de un runbook

Cuando cree o importe un runbook nuevo, debe publicarlo para poder ejecutarlo. Cada runbook de Azure Automation tiene una versión de borrador y una versión publicada. Solo es posible ejecutar la versión publicada y solo es posible editar la versión de borrador. Los cambios realizados en la versión de borrador no afectan la versión publicada. Cuando la versión de borrador deba estar lista para su disponibilidad, puede publicarla, lo que sobrescribirá la versión publicada actual con la versión de borrador.

### <a name="publish-a-runbook-in-the-azure-portal"></a>Publicación de un runbook en Azure Portal

1. Abra el runbook en Azure Portal.
2. Haga clic en **Editar**.
3. Haga clic en el botón **Publicar** y, a continuación, en **Sí** en el mensaje de comprobación.

### <a name="publish-a-runbook-using-powershell"></a>Publicación de un runbook mediante PowerShell

Utilice el cmdlet [Publish-AzAutomationRunbook](/powershell/module/Az.Automation/Publish-AzAutomationRunbook) para publicar el runbook. 

```azurepowershell-interactive
$aaName = "MyAutomationAccount"
$RunbookName = "Sample_TestRunbook"
$rgName = "MyResourceGroup"

$publishParams = @{
    AutomationAccountName = $aaName
    ResourceGroupName     = $rgName
    Name                  = $RunbookName
}
Publish-AzAutomationRunbook @publishParams
```

## <a name="schedule-a-runbook-in-the-azure-portal"></a>Programación de un runbook en Azure Portal

Una vez publicado el runbook, puede programarlo para que funcione:

1. Abra el runbook en Azure Portal.
2. Seleccione **Programas** en **Recursos**.
3. Seleccione **Agregar una programación**.
4. En el panel Programación de un runbook, seleccione **Vincular una programación a su runbook**.
5. En el panel Programación, seleccione **Crear una nueva programación**.
6. Escriba un nombre, una descripción y otros parámetros en el panel Nueva programación.
7. Una vez creada la programación, resáltela y haga clic en **Aceptar**. Ahora debería estar vinculada a su runbook.
8. Busque un correo electrónico en el buzón que le informa sobre el estado del runbook.

## <a name="obtain-job-statuses"></a>Obtención de estados del trabajo

### <a name="view-statuses-in-the-azure-portal"></a>Visualización de los estados en Azure Portal

En Azure Automation, los detalles del control de trabajos se proporcionan en [Trabajos](automation-runbook-execution.md#jobs). Cuando esté listo para ver los trabajos de runbook, use Azure Portal y acceda a su cuenta de Automation. A la derecha, se puede ver un resumen de todos los trabajos de runbook en **Estadísticas de trabajo**.

![Icono Estadísticas de trabajo](./media/manage-runbooks/automation-account-job-status-summary.png)

Este resumen muestra un contador y una representación gráfica del estado de trabajo de cada trabajo ejecutado.

Al hacer clic en el icono aparece la página Trabajos, que incluye una lista resumida de todos los trabajos ejecutados. En esta página se muestra el estado, el nombre del runbook, la hora de inicio y la hora de finalización de cada trabajo.

:::image type="content" source="./media/manage-runbooks/automation-account-jobs-status-blade.png" alt-text="Captura de pantalla de la página Trabajos.":::

Para filtrar la lista de trabajos, puede seleccionar **Filtrar trabajos**. Filtre por un runbook específico, estado de trabajo o una opción de la lista desplegable y proporcione el intervalo de tiempo para la búsqueda.

![Filtrado por estado del trabajo](./media/manage-runbooks/automation-account-jobs-filter.png)

Como alternativa, puede ver los detalles resumidos de los trabajos de un runbook concreto; para ello, selecciónelo en la página Runbooks de la cuenta de Automation y, después, seleccione **Trabajos**. Esta acción presenta la página Trabajos. Desde aquí, puede hacer clic en un registro del trabajo para ver sus detalles y resultados.

:::image type="content" source="./media/manage-runbooks/automation-runbook-job-summary-blade.png" alt-text="Captura de pantalla de la página Trabajos con el botón Errores resaltado.":::

### <a name="retrieve-job-statuses-using-powershell"></a>Recuperación de los estados del trabajo con PowerShell

Puede utilizar el cmdlet [Get-AzAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob) para recuperar los trabajos creados para un runbook y los detalles de un trabajo determinado. Si inicia un runbook mediante `Start-AzAutomationRunbook`, se devuelve el trabajo resultante. Utilice [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput) para recuperar la salida del trabajo.

En el ejemplo siguiente se recupera el último trabajo para un runbook de ejemplo y se muestra su estado, los valores proporcionados para los parámetros del runbook y la salida del trabajo.

```azurepowershell-interactive
$getJobParams = @{
    AutomationAccountName = 'MyAutomationAccount'
    ResourceGroupName     = 'MyResourceGroup'
    Runbookname           = 'Test-Runbook'
}
$job = (Get-AzAutomationJob @getJobParams | Sort-Object LastModifiedDate -Desc)[0]
$job | Select-Object JobId, Status, JobParameters

$getOutputParams = @{
    AutomationAccountName = 'MyAutomationAccount'
    ResourceGroupName     = 'MyResourceGroup'
    Id                    = $job.JobId
    Stream                = 'Output'
}
Get-AzAutomationJobOutput @getOutputParams
```

En el ejemplo siguiente se recupera la salida de un trabajo específico y se devuelve cada registro. Si hay una [excepción](automation-runbook-execution.md#exceptions) para uno de los registros, el script escribe la excepción en lugar del valor. Este comportamiento resulta útil, ya que las excepciones pueden proporcionar información adicional que podría no registrarse normalmente durante la salida.

```azurepowershell-interactive
$params = @{
    AutomationAccountName = 'MyAutomationAccount'
    ResourceGroupName     = 'MyResourceGroup'
    Stream                = 'Any'
}
$output = Get-AzAutomationJobOutput @params

foreach ($item in $output) {
    $jobOutParams = @{
        AutomationAccountName = 'MyAutomationAccount'
        ResourceGroupName     = 'MyResourceGroup'
        Id                    = $item.StreamRecordId
    }
    $fullRecord = Get-AzAutomationJobOutputRecord @jobOutParams

    if ($fullRecord.Type -eq 'Error') {
        $fullRecord.Value.Exception
    } else {
        $fullRecord.Value
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información detallada sobre la administración de un runbook, vea [Ejecución de un runbook en Azure Automation](automation-runbook-execution.md).
* Para preparar un runbook de PowerShell, vea [Edición de runbooks de texto en Azure Automation](automation-edit-textual-runbook.md).
* Para solucionar problemas con la ejecución de un runbook, vea [Solución de problemas relativos a errores con runbooks](troubleshoot/runbooks.md).
