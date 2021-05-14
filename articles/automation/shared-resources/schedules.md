---
title: Administración de programaciones en Azure Automation
description: En este artículo se explica cómo crear una programación en Azure Automation y cómo trabajar con ella.
services: automation
ms.subservice: shared-capabilities
ms.date: 03/29/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f119c15cbbfd9586bdb06fdffad0b12c9a441eea
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832651"
---
# <a name="manage-schedules-in-azure-automation"></a>Administración de programaciones en Azure Automation

Para programar un runbook en Azure Automation para que se inicie en un momento determinado, vincúlelo a una o más programaciones. Es posible configurar una programación para que se ejecute una vez o según una programación periódica diaria o por hora para los runbooks en Azure Portal. También las puede programar para que se ejecuten una vez a la semana, una vez al mes, en días específicos de la semana o del mes, o bien en un día determinado del mes. Un runbook puede vincularse a varias programaciones y una programación puede tener varios runbooks vinculados a ella.

> [!NOTE]
> Azure Automation admite el horario de verano y lo programa de forma adecuada para las operaciones de automatización.

> [!NOTE]
> Las programaciones no están habilitadas actualmente para las configuraciones de DSC de Azure Automation.

## <a name="powershell-cmdlets-used-to-access-schedules"></a>Cmdlets de PowerShell usados para acceder a programaciones

Los cmdlets de la tabla siguiente permiten crear y administrar programaciones de Automation con PowerShell. Se suministran como un componente de los [módulos Az](modules.md#az-modules).

| Cmdlets | Descripción |
|:--- |:--- |
| [Get-AzAutomationSchedule](/powershell/module/Az.Automation/Get-AzAutomationSchedule) |Recupera una programación. |
| [Get-AzAutomationScheduledRunbook](/powershell/module/az.automation/get-azautomationscheduledrunbook) |Recupera runbooks programados. |
| [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) |Crea una nueva programación. |
| [Register-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Register-AzAutomationScheduledRunbook) |Asocia un runbook con una programación. |
| [Remove-AzAutomationSchedule](/powershell/module/Az.Automation/Remove-AzAutomationSchedule) |Quita una programación. |
| [Set-AzAutomationSchedule](/powershell/module/Az.Automation/Set-AzAutomationSchedule) |Establece las propiedades de una programación existente. |
| [Unregister-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Unregister-AzAutomationScheduledRunbook) |Anula la asociación de un runbook con una programación. |

## <a name="create-a-schedule"></a>Crear una programación

Puede crear una nueva programación para los runbooks desde Azure Portal, con PowerShell o mediante una plantilla Azure Resource Manager (ARM). Para evitar que afecte a los runbooks y a los procesos que automatizan, primero debe probar los runbooks que tengan programaciones vinculadas con una cuenta de Automation dedicada para realizar pruebas. Una prueba valida que los runbooks programados sigan funcionando correctamente. Si ve un problema, puede solucionarlo y aplicar los cambios necesarios antes de migrar la versión de runbook actualizada a producción.

> [!NOTE]
> La cuenta de Automation no obtiene automáticamente las nuevas versiones de los módulos, a menos que se hayan actualizado manualmente seleccionando la opción [Actualizar módulos de Azure](../automation-update-azure-modules.md) en **Módulos**. Azure Automation utiliza los módulos más recientes en su cuenta de Automation al ejecutar un nuevo trabajo programado. 

### <a name="create-a-new-schedule-in-the-azure-portal"></a>Creación de una programación nueva en Azure Portal

1. En la cuenta de Automation, en el panel izquierdo, seleccione **Programas** en **Recursos compartidos**.
2. En la página **Programas**, seleccione **Agregar una programación**.
3. En la página **Nueva programación**, escriba un nombre y, opcionalmente, una descripción para la nueva programación.

    >[!NOTE]
    >Las programaciones de Automation no admiten actualmente el uso de caracteres especiales en el nombre de la programación.
    >

4. Elija si la programación se ejecuta una vez o de forma periódica seleccionando **Una vez** o **Periódico**. Si elige **Una vez**, especifique una hora de inicio y seleccione **Crear**. Si selecciona **Periódica**, especifique una hora de inicio. Para **Repetir cada**, seleccione la frecuencia con la que desea que se repita el runbook. Seleccione por hora, día, semana o mes.

    * Si selecciona **Semana**, se presentan los días de la semana entre los que puede elegir. Seleccione tantos días como quiera. La primera ejecución de la programación se realizará en el primer día seleccionado después de la hora de inicio. Por ejemplo, para elegir una programación durante el fin de semana, elija sábado y domingo.

    ![Establecimiento de la programación recurrente durante el fin de semana](../media/schedules/week-end-weekly-recurrence.png)

    * Si selecciona **Mes**, tendrá diferentes opciones. Para la opción **Repeticiones mensuales**, seleccione **Días del mes** o **Días de la semana**. Si selecciona **Días del mes**, aparece un calendario para que pueda elegir tantos días como desee. Si elige una fecha como el día 31 que no existe en el mes actual, no se ejecutará la programación. Si quiere que la programación se ejecute en el último día, seleccione **Sí** en **Ejecutar en el último día del mes**. Si selecciona **Días de la semana**, aparecerá la opción **Repetir cada**. Elija **Primero**, **Segundo**, **Tercero**, **Cuarto** o **Último**. Por último, elija un día en el que realizar la repetición.

    ![Programación mensual el primer día, el decimoquinto día y el último día del mes](../media/schedules/monthly-first-fifteenth-last.png)

5. Cuando haya finalizado, seleccione **Crear**.

### <a name="create-a-new-schedule-with-powershell"></a>Creación de una nueva programación con PowerShell

Use el cmdlet [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) para crear programaciones. Especifique la hora de inicio de la programación y la frecuencia con que se debe ejecutar. En los ejemplos siguientes se muestra cómo crear muchos escenarios de programación distintos.

>[!NOTE]
>Las programaciones de Automation no admiten actualmente el uso de caracteres especiales en el nombre de la programación.
>

#### <a name="create-a-one-time-schedule"></a>Creación de una programación única

En el siguiente ejemplo se crea una programación única.

```azurepowershell-interactive
$TimeZone = ([System.TimeZoneInfo]::Local).Id
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule01" -StartTime "23:00" -OneTime -ResourceGroupName "ResourceGroup01" -TimeZone $TimeZone
```

#### <a name="create-a-recurring-schedule"></a>Creación de una programación recurrente

En el ejemplo siguiente se muestra cómo crear una programación periódica que se ejecuta cada día a la 1:00 p.m. durante un año.

```azurepowershell-interactive
$StartTime = Get-Date "13:00:00"
$EndTime = $StartTime.AddYears(1)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule02" -StartTime $StartTime -ExpiryTime $EndTime -DayInterval 1 -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-weekly-recurring-schedule"></a>Creación de una programación semanal recurrente

En el ejemplo siguiente se muestra cómo crear una programación semanal que se ejecute solo en días laborables.

```azurepowershell-interactive
$StartTime = (Get-Date "13:00:00").AddDays(1)
[System.DayOfWeek[]]$WeekDays = @([System.DayOfWeek]::Monday..[System.DayOfWeek]::Friday)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule03" -StartTime $StartTime -WeekInterval 1 -DaysOfWeek $WeekDays -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-weekly-recurring-schedule-for-weekends"></a>Creación de una programación semanal recurrente para los fines de semana

En el ejemplo siguiente se muestra cómo crear una programación semanal que se ejecute solo los fines de semana.

```azurepowershell-interactive
$StartTime = (Get-Date "18:00:00").AddDays(1)
[System.DayOfWeek[]]$WeekendDays = @([System.DayOfWeek]::Saturday,[System.DayOfWeek]::Sunday)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Weekends 6PM" -StartTime $StartTime -WeekInterval 1 -DaysOfWeek $WeekendDays -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-recurring-schedule-for-the-first-fifteenth-and-last-days-of-the-month"></a>Creación de una programación periódica para el primer día, el decimoquinto día y el último día del mes

En el ejemplo siguiente se muestra cómo crear una programación periódica que se ejecuta el primer día, el decimoquinto día y el último día de un mes.

```azurepowershell-interactive
$StartTime = (Get-Date "18:00:00").AddDays(1)
New-AzAutomationSchedule -AutomationAccountName "TestAzureAuto" -Name "1st, 15th and Last" -StartTime $StartTime -DaysOfMonth @("One", "Fifteenth", "Last") -ResourceGroupName "TestAzureAuto" -MonthInterval 1
```

## <a name="create-a-schedule-with-a-resource-manager-template"></a>Creación de una programación con una plantilla de Resource Manager

En este ejemplo, se usa una plantilla de Resource Manager de Automation (ARM) que crea una nueva programación de trabajo. Para obtener información general sobre esta plantilla para administrar las programaciones de trabajos de Automation, consulte la [referencia de plantillas de Microsoft.Automation automationAccounts/jobSchedules](/azure/templates/microsoft.automation/2015-10-31/automationaccounts/jobschedules#quickstart-templates).

Copie este archivo de plantilla en un editor de texto:

```json
{
  "name": "5d5f3a05-111d-4892-8dcc-9064fa591b96",
  "type": "Microsoft.Automation/automationAccounts/jobSchedules",
  "apiVersion": "2015-10-31",
  "properties": {
    "schedule": {
      "name": "scheduleName"
    },
    "runbook": {
      "name": "runbookName"
    },
    "runOn": "hybridWorkerGroup",
    "parameters": {}
  }
}
```

Edite los siguientes valores de parámetro y guarde la plantilla como un archivo JSON:

* Nombre del objeto de programación de trabajo: se usa un GUID (identificador único global) como nombre del objeto de programación de trabajo.

   >[!IMPORTANT]
   > Para cada programación de trabajo implementada con una plantilla de ARM, el GUID debe ser único. Incluso si va a volver a programar una programación existente, deberá cambiar el GUID. Esto se aplica incluso si ha eliminado previamente una programación de trabajo existente creada con la misma plantilla. Si se reutiliza el mismo GUID, se producirá un error en la implementación.</br></br>
   > Hay servicios en línea que pueden generar un nuevo GUID automáticamente, como este [generador de GUID en línea gratuito](https://guidgenerator.com/).

* Nombre de programación: representa el nombre de la programación de trabajos de automatización que se vinculará al runbook especificado.
* Nombre del runbook: representa el nombre del runbook de Automation con el que se asociará la programación del trabajo.

Una vez guardado el archivo, puede crear la programación del trabajo del runbook con el siguiente comando de PowerShell. El comando usa el parámetro `TemplateFile` para especificar la ruta de acceso y el nombre de archivo de la plantilla.

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "<path>\RunbookJobSchedule.json"
```

## <a name="link-a-schedule-to-a-runbook"></a>Vinculación de una programación a un runbook

Un runbook puede vincularse a varias programaciones y una programación puede tener varios runbooks vinculados a ella. Si un runbook tiene parámetros, puede proporcionar valores para ellos. Debe proporcionar valores para los parámetros obligatorios y también puede proporcionar valores para los parámetros opcionales. Estos valores se usan cada vez que esta programación inicia el runbook. Puede adjuntar el mismo runbook a otra programación y especificar distintos valores de parámetros.

### <a name="link-a-schedule-to-a-runbook-with-the-azure-portal"></a>Vinculación de una programación a un runbook con Azure Portal

1. En Azure Portal, en la cuenta de Automation, seleccione **Runbooks** en **Automatización de procesos**.
1. Seleccione el nombre del runbook que se va a programar.
1. Si el runbook no está vinculado actualmente a una programación, se le ofrecerá la opción de crear una o de vincularlo a una existente.
1. Si el runbook tiene parámetros, puede seleccionar la opción **Modificar la configuración de ejecución (Predeterminada: Azure)** y aparece el panel **Parámetros**. Aquí puede escribir la información de los parámetros.

### <a name="link-a-schedule-to-a-runbook-with-powershell"></a>Vinculación de una programación a un runbook con PowerShell

Use el cmdlet [Register-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Register-AzAutomationScheduledRunbook) para vincular una programación. Puede especificar valores para los parámetros del runbook con el parámetro Parameters. Para más información sobre cómo especificar valores de parámetro, consulte [Inicio de un runbook en Azure Automation](../start-runbooks.md).
En el ejemplo siguiente se muestra cómo vincular una programación a un runbook mediante un cmdlet de Azure Resource Manager con parámetros.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$runbookName = "Test-Runbook"
$scheduleName = "Sample-DailySchedule"
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Register-AzAutomationScheduledRunbook -AutomationAccountName $automationAccountName `
-Name $runbookName -ScheduleName $scheduleName -Parameters $params `
-ResourceGroupName "ResourceGroup01"
```

## <a name="schedule-runbooks-to-run-more-frequently"></a>Programación de runbooks para que se ejecuten con más frecuencia

El intervalo de mayor frecuencia para el que se puede configurar una programación en Azure Automation es una hora. Si necesita programaciones que se ejecuten con mayor frecuencia, tiene dos opciones:

* Crear un [webhook](../automation-webhooks.md) para el runbook y usar [Azure Logic Apps](../../logic-apps/logic-apps-overview.md) para llamar al webhook. Azure Logic Apps ofrece una granularidad más específica para definir una programación.

* Crear cuatro programaciones que comiencen todas con una diferencia de 15 minutos entre sí y que se ejecuten una vez cada hora. Este escenario permite que el runbook se ejecute cada 15 minutos con las distintas programaciones.

## <a name="disable-a-schedule"></a>Deshabilitación de una programación

Cuando se deshabilita una programación, los runbooks vinculados a ella ya no se ejecutan en ella. Puede deshabilitar una programación manualmente o establecer una fecha de expiración para las programaciones con frecuencia al crearlas. Cuando se alcanza la fecha de expiración, la programación se deshabilita.

### <a name="disable-a-schedule-from-the-azure-portal"></a>Deshabilitación de una programación desde Azure Portal

1. En la cuenta de Automation, en el panel izquierdo, seleccione **Programas** en **Recursos compartidos**.
1. Seleccione el nombre de una programación para abrir el panel de detalles.
1. Cambie **Habilitado** a **No**.

> [!NOTE]
> Si quiere deshabilitar una programación con una hora de inicio en el pasado, debe cambiar la fecha de inicio a un momento en el futuro antes de guardarla.

### <a name="disable-a-schedule-with-powershell"></a>Deshabilitación de una programación con PowerShell

Use el cmdlet [Set-AzAutomationSchedule](/powershell/module/Az.Automation/Set-AzAutomationSchedule) para cambiar las propiedades de una programación existente. Para deshabilitar la programación, especifique False para el parámetro `IsEnabled`.

En el ejemplo siguiente se muestra cómo deshabilitar una programación para un runbook mediante un cmdlet de Azure Resource Manager.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Set-AzAutomationSchedule -AutomationAccountName $automationAccountName `
-Name $scheduleName -IsEnabled $false -ResourceGroupName "ResourceGroup01"
```

## <a name="remove-a-schedule"></a>Eliminación de una programación

Cuando esté preparado para quitar programaciones, puede usar Azure Portal o PowerShell. Recuerde que solo puede quitar una programación que se haya deshabilitado, tal como se describe en la sección anterior.

### <a name="remove-a-schedule-using-the-azure-portal"></a>Eliminación de una programación con Azure Portal

1. En la cuenta de Automation, en el panel izquierdo, seleccione **Programas** en **Recursos compartidos**.
2. Seleccione el nombre de una programación para abrir el panel de detalles.
3. Haga clic en **Eliminar**.

### <a name="remove-a-schedule-with-powershell"></a>Eliminación de una programación con PowerShell

Puede usar el cmdlet `Remove-AzAutomationSchedule` como se muestra a continuación para eliminar una programación existente.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Remove-AzAutomationSchedule -AutomationAccountName $automationAccountName `
-Name $scheduleName -ResourceGroupName "ResourceGroup01"
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre los cmdlets que se usan para acceder a las programaciones, consulte [Administración de módulos en Azure Automation](modules.md).
* Para obtener información general sobre los runbooks, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).
