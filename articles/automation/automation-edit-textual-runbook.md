---
title: Editar runbooks de texto en Azure Automation
description: En este artículo se indica cómo usar el editor de texto de Azure Automation para trabajar con los runbooks de PowerShell y de flujo de trabajo de PowerShell.
services: automation
ms.service: automation
ms.subservice: process-automation
author: mgoedtel
ms.author: magoedte
ms.date: 08/01/2018
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
manager: carmonm
ms.openlocfilehash: 296d45fae4d59553b54a1b68923c91be4168d3a5
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107829393"
---
# <a name="edit-textual-runbooks-in-azure-automation"></a>Editar runbooks de texto en Azure Automation

Puede utilizar el editor de texto de Azure Automation para editar [runbooks de PowerShell](automation-runbook-types.md#powershell-runbooks) y [runbooks del flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks). Este editor tiene las características típicas de otros editores de código, como IntelliSense. También utiliza codificación de colores con características especiales adicionales que le ayudarán a acceder a los recursos comunes para los runbooks. 

El editor de texto incluye una característica para insertar código para cmdlets, los recursos y los runbooks secundarios en un runbook. En lugar de escribir en el código personalmente, puede seleccionar entre una lista de recursos disponibles y el editor inserta el código adecuado en el runbook.

Cada runbook de Azure Automation tiene dos versiones: una de borrador y otra publicada. Edite la versión de borrador del runbook y después publíquela para que pueda ejecutarse. No se puede editar la versión publicada. Para obtener más información, vea [Publicar un runbook](manage-runbooks.md#publish-a-runbook).

En este artículo, se proporcionan pasos detallados para realizar diferentes funciones con este editor. No son aplicables a los [runbooks gráficos](automation-runbook-types.md#graphical-runbooks). Para trabajar con estos runbooks, consulte [Creación gráfica en Azure Automation](automation-graphical-authoring-intro.md).

## <a name="edit-a-runbook-with-the-azure-portal"></a>Edición de un runbook con Azure Portal

1. En Azure Portal, seleccione su cuenta de Automation.
2. En **AUTOMATIZACIÓN DE PROCESOS**, seleccione **Runbooks** para abrir la lista de runbooks.
3. Elija el runbook que desea editar y, a continuación, haga clic en **Editar**.
4. Edite el runbook.
5. Haga clic en **Guardar** cuando termine las modificaciones.
6. Haga clic en **Publicar** si desea publicar la última versión de borrador del runbook.

### <a name="insert-a-cmdlet-into-a-runbook"></a>Inserción de un cmdlet en un runbook

1. En el lienzo del editor de texto, coloque el cursor donde desee colocar el cmdlet.
2. Expanda el nodo **Cmdlets** en el control Biblioteca.
3. Expanda el módulo que contiene el cmdlet que desea usar.
4. Haga clic con el botón derecho en el nombre del cmdlet para insertarlo y seleccione **Agregar al lienzo**. Si el cmdlet tiene más de un conjunto de parámetros, el editor agrega el conjunto predeterminado. También puede expandir el cmdlet para seleccionar un conjunto de parámetros diferente.
5. Observe que el código del cmdlet se inserta con toda su lista de parámetros.
6. Proporcione un valor adecuado en lugar del valor entre corchetes (<>) para los parámetros necesarios. Quite todos los parámetros que no sean necesarios.

### <a name="insert-code-for-a-child-runbook-into-a-runbook"></a>Inserción del código de un runbook secundario en un runbook

1. En el lienzo del editor de texto, coloque el cursor donde desee colocar el código del [runbook secundario](automation-child-runbooks.md).
2. Expanda el nodo **Runbooks** en el control Biblioteca.
3. Haga clic con el botón derecho en el runbook para insertarlo y seleccione **Agregar al lienzo**.
4. Se inserta el código para el runbook secundario con los marcadores de posición de los parámetros de runbook.
5. Reemplace los marcadores de posición por valores adecuados para cada parámetro.

### <a name="insert-an-asset-into-a-runbook"></a>Inserción de un recurso en un runbook

1. En el control Lienzo del editor de texto, coloque el cursor donde desee colocar el código del runbook secundario.
2. Expanda el nodo **Recursos** en el control Biblioteca.
3. Expanda el nodo del tipo de recurso deseado.
4. Haga clic con el botón derecho en el nombre del recurso para insertarlo y seleccione **Agregar al lienzo**. Para [recursos variables](./shared-resources/variables.md), seleccione **Agregar "Obtener variable" a lienzo** o **Agregar "Establecer variable" a lienzo** en función de si desea obtener o establecer la variable.
5. Observe que el código del recurso se inserta en el runbook.

## <a name="edit-an-azure-automation-runbook-using-windows-powershell"></a>Edición de un runbook de Azure Automation mediante Windows PowerShell

Para editar un runbook con Windows PowerShell, use el editor de su elección y guarde el runbook en un archivo **.ps1**. Puede usar el cmdlet [Export-AzAutomationRunbook](/powershell/module/Az.Automation/Export-AzAutomationRunbook) para recuperar el contenido del runbook. Puede usar el cmdlet [Import-AzAutomationRunbook](/powershell/module/Az.Automation/import-azautomationrunbook) para reemplazar el runbook de borrador existente por el modificado.

### <a name="retrieve-the-contents-of-a-runbook-using-windows-powershell"></a>Recuperación del contenido de un runbook mediante Windows PowerShell

En los siguientes comandos de ejemplo, se muestra cómo se recupera el script para un runbook y se guarda en un archivo de script. En este ejemplo, se recupera la versión de borrador. También es posible recuperar la versión publicada del runbook, aunque esta no se puede cambiar.

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Export-AzAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -OutputFolder $scriptFolder -Slot Draft
```

### <a name="change-the-contents-of-a-runbook-using-windows-powershell"></a>Modificación del contenido de un runbook mediante Windows PowerShell

En los siguientes comandos de ejemplo, se muestra cómo reemplazar el contenido existente de un runbook por el de un archivo de script. 

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Import-AzAutomationRunbook -Path "$scriptfolder\Hello-World.ps1" -Name $runbookName -Type PowerShell -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -Force
Publish-AzAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

* [Administración de runbooks en Azure Automation](manage-runbooks.md).
* [Aprendizaje del flujo de trabajo de Windows PowerShell](automation-powershell-workflow.md).
* [Creación gráfica en Azure Automation](automation-graphical-authoring-intro.md).
* [Certificados](./shared-resources/certificates.md).
* [Conexiones](automation-connections.md).
* [Credenciales](./shared-resources/credentials.md).
* [Programaciones](./shared-resources/schedules.md).
* [Variables](./shared-resources/variables.md).
* [Referencia de cmdlets de PowerShell](/powershell/module/az.automation).
