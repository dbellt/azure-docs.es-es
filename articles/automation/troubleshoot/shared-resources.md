---
title: Solución de problemas de recursos compartidos de Azure Automation
description: En este artículo se describe cómo solucionar y resolver problemas con recursos compartidos de Azure Automation.
services: automation
ms.subservice: ''
ms.date: 01/27/2021
ms.topic: troubleshooting
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 36f8b9d8fc890eb486ec59b972cc2fdf52ae0c80
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166062"
---
# <a name="troubleshoot-shared-resource-issues"></a>Solución de problemas de recursos compartidos

En este artículo se describen problemas que pueden surgir al usar [recursos compartidos](../automation-intro.md#shared-resources) en Azure Automation.

## <a name="modules"></a>Módulos

### <a name="scenario-a-module-is-stuck-during-import"></a><a name="module-stuck-importing"></a>Escenario: Un módulo se bloquea durante la importación

#### <a name="issue"></a>Problema

Al importar o actualizar los módulos en Azure Automation, encuentra un módulo que está bloqueado en el estado *Importing* (Importando).

#### <a name="cause"></a>Causa

Dado que la importación de módulos de PowerShell es un proceso complejo de varios pasos, es posible que un módulo no se importe correctamente y se bloquee en un estado transitorio. Para más información sobre el proceso de importación, consulte [Importación de un módulo de PowerShell](/powershell/scripting/developer/module/importing-a-powershell-module#the-importing-process).

#### <a name="resolution"></a>Solución

Para resolver este problema, debe eliminar el módulo que está bloqueado mediante el cmdlet [Remove-AzAutomationModule](/powershell/module/Az.Automation/Remove-AzAutomationModule). A continuación, puede volver a intentar importar el módulo.

```azurepowershell-interactive
Remove-AzAutomationModule -Name ModuleName -ResourceGroupName ExampleResourceGroup -AutomationAccountName ExampleAutomationAccount -Force
```

### <a name="scenario-azurerm-modules-are-stuck-during-import-after-an-update-attempt"></a><a name="update-azure-modules-importing"></a>Escenario: Los módulos de AzureRM se bloquean durante la importación después de un intento de actualización

#### <a name="issue"></a>Problema

Después de intentar actualizar los módulos de AzureRM, en la cuenta permanece un banner con el siguiente mensaje:

```error
Azure modules are being updated
```

#### <a name="cause"></a>Causa

Hay un problema conocido con la actualización de los módulos AzureRM en una cuenta de Automation. En concreto, el problema se produce si los módulos se encuentran en un grupo de recursos con un nombre numérico que empieza por 0.

#### <a name="resolution"></a>Solución

Para actualizar los módulos de AzureRM en la cuenta de Automation, la cuenta debe estar en un grupo de recursos con un nombre alfanumérico. En este momento, los grupos de recursos con nombres numéricos que empiezan por 0 no pueden actualizar módulos de AzureRM.

### <a name="scenario-module-fails-to-import-or-cmdlets-cant-be-executed-after-importing"></a><a name="module-fails-to-import"></a>Escenario: No se puede importar el módulo o no se pueden ejecutar cmdlets después de la importación

#### <a name="issue"></a>Problema

No se puede importar un módulo, o bien se importa correctamente, pero no se extrae ningún cmdlet.

#### <a name="cause"></a>Causa

Algunas razones comunes por las que un módulo no se importa correctamente a Azure Automation son:

* La estructura no coincide con la que necesita Automation.
* El módulo depende de otro módulo que no se ha implementado en su cuenta de Automation.
* Al módulo le faltan sus dependencias en la carpeta.
* El cmdlet [New-AzAutomationModule](/powershell/module/Az.Automation/New-AzAutomationModule) se usa para cargar el módulo y no se ha proporcionado la ruta de acceso de almacenamiento completa o no se ha cargado el módulo mediante una dirección URL de acceso público.

#### <a name="resolution"></a>Solución

Use cualquiera de estas soluciones para corregir el problema:

* Asegúrese de que el módulo tiene el formato siguiente: nombreDeMódulo.zip -> nombreDeMódulo o númeroDeVersión -> (nombreDeMódulo.psm1, nombreDeMódulo.psd1).
* Abra el archivo **.psd1** y compruebe si el módulo tiene dependencias. Si es así, cargue estos módulos en la cuenta de Automation.
* Asegúrese de que todos los archivos **.dll** a los que se hace referencia están presentes en la carpeta del módulo.

### <a name="scenario-update-azuremoduleps1-suspends-when-updating-modules"></a><a name="all-modules-suspended"></a>Escenario: Update-AzureModule.ps1 se suspende durante la actualización de módulos

#### <a name="issue"></a>Problema

Al usar el runbook [Update-AzureModule.ps1](https://github.com/azureautomation/runbooks/blob/master/Utility/ARM/Update-AzureModule.ps1) para actualizar los módulos de Azure, se suspende el proceso de actualización del módulo.

#### <a name="cause"></a>Causa

En este runbook, el valor predeterminado para determinar cuántos módulos se actualizan simultáneamente es de 10. El proceso de actualización es propenso a errores cuando se actualizan demasiados módulos al mismo tiempo.

#### <a name="resolution"></a>Solución

No es habitual que se necesiten todos los módulos de AzureRM o Az en la misma cuenta de Automation. Se recomienda importar únicamente los módulos específicos que necesite.

> [!NOTE]
> Evite importar todo el módulo `Az.Automation` o `AzureRM.Automation`, que importa todos los módulos contenidos.

Si se suspende el proceso de actualización, agregue el parámetro `SimultaneousModuleImportJobCount` al script **Update-AzureModules.ps1** y proporcione un valor menor que el predeterminado de 10. Se recomienda que, si implementa esta lógica, comience por un valor de 3 o 5. `SimultaneousModuleImportJobCount` es un parámetro del runbook del sistema **Update-AutomationAzureModulesForAccount** que se usa para actualizar módulos de Azure. Si realiza este ajuste, el proceso de actualización se ejecuta durante más tiempo, pero tiene más probabilidades de completarse. En el ejemplo siguiente se muestra el parámetro y su ubicación en el runbook:

```powershell
$Body = @"
   {
      "properties":{
      "runbook":{
            "name":"Update-AutomationAzureModulesForAccount"
      },
      "parameters":{
            ...
            "SimultaneousModuleImportJobCount":"3",
            ... 
      }
      }
   }
"@
```

## <a name="run-as-accounts"></a>Cuentas de ejecución

### <a name="scenario-youre-unable-to-create-or-update-a-run-as-account"></a><a name="unable-create-update"></a>Escenario: No puede crear o actualizar una cuenta de ejecución

#### <a name="issue"></a>Problema

Al intentar crear o actualizar una cuenta de ejecución, recibe un error similar al siguiente:

```error
You do not have permissions to create…
```

#### <a name="cause"></a>Causa

No tiene los permisos que necesita para crear o actualizar la cuenta de ejecución o el recurso está bloqueado en un nivel de grupo de recursos.

#### <a name="resolution"></a>Solución

Para crear o actualizar una cuenta de ejecución, debe tener los [permisos](../automation-security-overview.md#permissions) adecuados para los distintos recursos que usa la cuenta de ejecución.

Si el problema se debe a un bloqueo, compruebe que el bloqueo se puede quitar. A continuación, vaya al recurso que está bloqueado en Azure Portal, haga clic con el botón derecho en el bloqueo y seleccione **Eliminar**.

### <a name="scenario-you-receive-the-error-unable-to-find-an-entry-point-named-getperadapterinfo-in-dll-iplpapidll-when-executing-a-runbook"></a><a name="iphelper"></a>Escenario: Aparece el error "No se puede encontrar el punto de entrada denominado 'GetPerAdapterInfo' en el archivo DLL 'iplpapi.dll'" al ejecutar un runbook.

#### <a name="issue"></a>Problema

Al ejecutar un runbook aparece la siguiente excepción:

```error
Unable to find an entry point named 'GetPerAdapterInfo' in DLL 'iplpapi.dll'
```

#### <a name="cause"></a>Causa

Este error suele deberse a una configuración incorrecta de la [cuenta de ejecución](../automation-security-overview.md).

#### <a name="resolution"></a>Solución

Asegúrese de que la cuenta de ejecución está configurada correctamente. A continuación, compruebe que tiene el código adecuado en el runbook para autenticarse con Azure. En el ejemplo siguiente se muestra un fragmento de código para la autenticación en Azure en un runbook mediante una cuenta de ejecución.

```powershell
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
-ApplicationID $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint
```

## <a name="next-steps"></a>Pasos siguientes

Si en este artículo no se resuelve la incidencia, pruebe uno de los canales siguientes para obtener ayuda adicional:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Conectar con [@AzureSupport](https://twitter.com/azuresupport). Se trata de la cuenta oficial de Microsoft Azure para conectar a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.
* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.
