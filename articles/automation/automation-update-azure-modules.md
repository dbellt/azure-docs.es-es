---
title: Actualización de módulos de Azure PowerShell en Azure Automation
description: En este artículo se describe cómo actualizar módulos comunes de Azure PowerShell proporcionados de forma predeterminada en Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 06/14/2019
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c1632da35864fc6822b385adac06d7f124aea061
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107830419"
---
# <a name="update-azure-powershell-modules"></a>Actualización de módulos de Azure PowerShell

Los módulos de PowerShell más comunes se proporcionan de forma predeterminada en cada cuenta de Automation. Consulte [Módulos predeterminados](shared-resources/modules.md#default-modules). Dado que el equipo de Azure actualiza los módulos de Azure con regularidad, pueden producirse cambios en los cmdlets incluidos. Estos cambios, como el cambio de nombre de un parámetro o dejar en desuso un cmdlet por completo, pueden afectar negativamente a los runbooks. 

> [!NOTE]
> No se pueden eliminar módulos globales, que son los módulos que Automation proporciona de forma integrada.

## <a name="set-up-an-automation-account"></a>Configuración de una cuenta de Automation

Para evitar que incidan en los runbooks y los procesos que automatizan, asegúrese de probarlos y validarlos cuando realice actualizaciones. Si no tiene una cuenta de Automation dedicada pensada para este propósito, considere la posibilidad de crear una para que pueda probar muchos escenarios diferentes durante el desarrollo de los runbooks. Estas pruebas deben incluir cambios iterativos, por ejemplo, la actualización de los módulos de PowerShell.

Asegúrese de que la cuenta de Automation tiene una [cuenta de ejecución de Azure](automation-security-overview.md#run-as-accounts) creada.

Si desarrolla sus scripts de forma local, se recomienda tener localmente las mismas versiones del módulo que tiene en la cuenta de Automation al realizar pruebas, a fin de asegurarse de que obtendrá los mismos resultados. Una vez validados los resultados y aplicados los cambios necesarios, puede mover los cambios a producción.

> [!NOTE]
> Es posible que una cuenta de Automation nueva no contenga los módulos más recientes.

## <a name="obtain-a-runbook-to-use-for-updates"></a>Obtención de un runbook para su uso en las actualizaciones

Para actualizar los módulos de Azure en la cuenta de Automation, debe usar el runbook [Update-AutomationAzureModulesForAccount](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update), que está disponible como código abierto. Para empezar a usar este runbook para actualizar los módulos de Azure, descárguelo desde el repositorio de GitHub. A continuación, puede importarlo en la cuenta de Automation o ejecutarlo como un script. Para aprender a importar un runbook en la cuenta de Automation, consulte [Importación de un runbook](manage-runbooks.md#import-a-runbook).

El runbook **Update-AutomationAzureModulesForAccount** admite de forma predeterminada la actualización de los módulos Azure, AzureRM y AZ. Revise el archivo [LÉAME del runbook para actualizar módulos de Azure](https://github.com/microsoft/AzureAutomation-Account-Modules-Update/blob/master/README.md) para más información sobre cómo actualizar los módulos de Az.Automation con este runbook. Hay otros factores importantes que se deben tener en cuenta al usar los módulos de Az en la cuenta de Automation. Consulte [Administración de módulos en Azure Automation](shared-resources/modules.md) para más información.

## <a name="use-update-runbook-code-as-a-regular-powershell-script"></a>Uso del código de runbook de actualización como script de PowerShell normal

Puede usar el código de runbook como script de PowerShell normal en lugar de un runbook. Para ello, inicie sesión en Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) primero y, después, pase `-Login $false` al script.

## <a name="use-the-update-runbook-on-sovereign-clouds"></a>Uso del runbook de actualización en nubes soberanas

Para usar este runbook en nubes soberanas, use el parámetro `AzEnvironment` para pasar el entorno correcto al runbook. Los valores permitidos son AzureCloud (nube pública de Azure), AzureChinaCloud, AzureGermanCloud y AzureUSGovernment. Estos valores se pueden recuperar con `Get-AzEnvironment | select Name`. Si no pasa un valor a este cmdlet, el runbook toma como valor predeterminado AzureCloud.

## <a name="use-the-update-runbook-to-update-a-specific-module-version"></a>Uso del runbook de actualización para actualizar una versión específica del módulo

Si desea utilizar una versión específica del módulo de Azure PowerShell en lugar del último módulo disponible en la Galería de PowerShell, pase estas versiones al parámetro opcional `ModuleVersionOverrides` del runbook **Update-AutomationAzureModulesForAccount**. Para obtener ejemplos, consulte el runbook [Update-AutomationAzureModulesForAccount.ps1](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update/blob/master/Update-AutomationAzureModulesForAccount.ps1). Los módulos de Azure PowerShell que no se mencionan en el parámetro `ModuleVersionOverrides` se actualizan con las últimas versiones de los módulos en la Galería de PowerShell. Si no se pasa nada al parámetro `ModuleVersionOverrides`, todos los módulos se actualizan con las versiones de módulo más recientes de la Galería de PowerShell. Se trata del mismo comportamiento que tiene el botón **Actualizar módulos de Azure** de Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre el uso de módulos, consulte [Administración de módulos en Azure Automation](shared-resources/modules.md).
* Para más información sobre el runbook de actualización, consulte el [runbook Actualizar módulos de Azure](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update).
