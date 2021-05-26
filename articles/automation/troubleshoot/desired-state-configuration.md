---
title: Solución de problemas de State Configuration de Azure Automation
description: En este artículo se explica cómo solucionar problemas de State Configuration de Azure Automation.
services: automation
ms.subservice: ''
ms.date: 04/16/2019
ms.topic: troubleshooting
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5fe977851011bdfa4f7bbf2bde24e5e4b6fd480d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110457863"
---
# <a name="troubleshoot-azure-automation-state-configuration-issues"></a>Solución de problemas de State Configuration de Azure Automation

En este artículo se proporciona información sobre la solución de problemas que surgen al compilar o implementar configuraciones en State Configuration de Azure Automation. Para obtener información general sobre la característica State Configuration, consulte [Introducción a State Configuration](../automation-dsc-overview.md).

## <a name="diagnose-an-issue"></a>Diagnóstico de problemas

Si recibe un error de compilación o implementación de la configuración, los pasos siguientes le ayudarán a diagnosticar el problema.

### <a name="1-ensure-that-your-configuration-compiles-successfully-on-the-local-machine"></a>1. Asegúrese de que la configuración se compila correctamente en la máquina local

State Configuration de Azure Automation está integrado en Desired State Configuration (DSC) de PowerShell. Puede encontrar la documentación del lenguaje y la sintaxis de DSC en los [documentos de DSC de PowerShell](/powershell/scripting/overview).

Al compilar una configuración de DSC en la máquina local, puede detectar y resolver errores comunes, como los siguientes:

   - Módulos que faltan
   - Errores de sintaxis
   - Errores lógicos

### <a name="2-view-dsc-logs-on-your-node"></a>2. Visualización de los registros de DSC en el nodo

Si la configuración se compila correctamente, pero se produce un error cuando se aplica a un nodo, puede encontrar información detallada en los registros de DSC. Para obtener información sobre dónde encontrar estos registros, consulte [¿Dónde se encuentran los registros de eventos de DSC?](/powershell/scripting/dsc/troubleshooting/troubleshooting#where-are-dsc-event-logs)

El módulo [xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics) puede ayudarle a analizar la información detallada de los registros de DSC. Si se pone en contacto con el equipo de soporte técnico, necesitarán estos registros para diagnosticar el problema.

Puede instalar el módulo `xDscDiagnostics` en la máquina local con las instrucciones para la [instalación de la versión estable del módulo](https://github.com/PowerShell/xDscDiagnostics#install-the-stable-version-module).

Para instalar el módulo `xDscDiagnostics` en la máquina de Azure, use [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand). También puede usar la opción **Ejecutar comando** de Azure Portal siguiendo los pasos de [Ejecución de scripts de PowerShell en la máquina virtual Windows mediante Ejecutar comando](../../virtual-machines/windows/run-command.md).

Para obtener información sobre el uso de **xDscDiagnostics**, consulte [Uso de xDscDiagnostics para analizar los registros de DSC](/powershell/scripting/dsc/troubleshooting/troubleshooting#using-xdscdiagnostics-to-analyze-dsc-logs). Consulte también [Cmdlets de xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics#cmdlets).

### <a name="3-ensure-that-nodes-and-the-automation-workspace-have-required-modules"></a>3. Asegúrese de que los nodos y el área de trabajo de Automation tengan los módulos necesarios

DSC depende de módulos que hay instalados en el nodo. Al usar State Configuration de Azure Automation, importe los módulos necesarios en la cuenta de Automation mediante los pasos de [Importación de módulos](../shared-resources/modules.md#import-modules). Las configuraciones también pueden tener una dependencia en versiones específicas de los módulos. Para más información, consulte [Solución de problemas de módulos](shared-resources.md#modules).

## <a name="scenario-a-configuration-with-special-characters-cant-be-deleted-from-the-portal"></a><a name="unsupported-characters"></a>Escenario: no se puede eliminar una configuración con caracteres especiales del portal

### <a name="issue"></a>Incidencia

Al intentar eliminar una configuración de DSC del portal, verá el siguiente error:

```error
An error occurred while deleting the DSC configuration '<name>'.  Error-details: The argument configurationName with the value <name> is not valid.  Valid configuration names can contain only letters,  numbers, and underscores.  The name must start with a letter.  The length of the name must be between 1 and 64 characters.
```

### <a name="cause"></a>Causa

Este error es un problema temporal que está previsto que se resuelva.

### <a name="resolution"></a>Resolución

Use el cmdlet [Remove-AzAutomationDscConfiguration](/powershell/module/Az.Automation/Remove-AzAutomationDscConfiguration) para eliminar la configuración.

## <a name="scenario-failed-to-register-the-dsc-agent"></a><a name="failed-to-register-agent"></a>Escenario: no se pudo registrar el agente de DSC

### <a name="issue"></a>Incidencia

Si usa [Set-DscLocalConfigurationManager](/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager) u otro cmdlet de DSC, recibirá el siguiente error:

```error
Registration of the Dsc Agent with the server
https://<location>-agentservice-prod-1.azure-automation.net/accounts/00000000-0000-0000-0000-000000000000 failed. The
underlying error is: Failed to register Dsc Agent with AgentId 00000000-0000-0000-0000-000000000000 with the server htt
ps://<location>-agentservice-prod-1.azure-automation.net/accounts/00000000-0000-0000-0000-000000000000/Nodes(AgentId='00000000-0000-0000-0000-000000000000'). .
    + CategoryInfo          : InvalidResult: (root/Microsoft/...gurationManager:String) [], CimException
    + FullyQualifiedErrorId : RegisterDscAgentCommandFailed,Microsoft.PowerShell.DesiredStateConfiguration.Commands.Re
   gisterDscAgentCommand
    + PSComputerName        : <computerName>
```

### <a name="cause"></a>Causa

Este error se debe normalmente a un firewall, a la máquina que está detrás de un servidor proxy o a otros errores de red.

### <a name="resolution"></a>Solución

Compruebe que la máquina tenga acceso a los puntos de conexión adecuados para DSC y vuelva a intentarlo. Para ver una lista de puertos y direcciones necesarios, consulte el artículo sobre [planeamiento de red](../automation-dsc-overview.md#network-planning).

## <a name="scenario-status-reports-return-the-response-code-unauthorized"></a><a name="unauthorized"></a>Escenario: los informes de estado devuelven el código de respuesta No autorizado

### <a name="issue"></a>Incidencia

Al registrar un nodo con State Configuration de Azure Automation, recibirá uno de los mensajes de error siguientes:

```error
The attempt to send status report to the server https://{your Automation account URL}/accounts/xxxxxxxxxxxxxxxxxxxxxx/Nodes(AgentId='xxxxxxxxxxxxxxxxxxxxxxxxx')/SendReport returned unexpected response code Unauthorized.
```

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC / Registration of the Dsc Agent with the server failed.
```

### <a name="cause"></a>Causa

Este problema se debe a un certificado que no es válido o ha expirado. Consulte [Nuevo registro de un nodo](../automation-dsc-onboarding.md#re-register-a-node).

Este problema también puede deberse a una configuración de proxy que no permite el acceso a * **.azure-automation.net**. Para obtener más información, vea [Configuración de redes privadas](../automation-dsc-overview.md#network-planning). 

### <a name="resolution"></a>Solución

Siga estos pasos para volver a registrar el nodo de DSC con errores.

#### <a name="step-1-unregister-the-node"></a>Paso 1: Cancelación del registro del nodo

1. En Azure Portal, vaya a **Inicio** > **Cuentas de Automation** > (su cuenta de Automation) > **State Configuration (DSC)** .
1. Seleccione **Nodos** y, luego, el nodo con problemas.
1. Seleccione **Cancelar registro** para cancelar el registro del nodo.

#### <a name="step-2-uninstall-the-dsc-extension-from-the-node"></a>Paso 2: Desinstalación de la extensión DSC del nodo

1. En Azure Portal, vaya a **Inicio** > **Máquina virtual** > (nodo con errores) > **Extensiones**.
1. Seleccione **Microsoft.PowerShell.DSC**, la extensión DSC de PowerShell.
1. Haga clic en **Desinstalar** para desinstalar la extensión.

#### <a name="step-3-remove-all-bad-or-expired-certificates-from-the-node"></a>Paso 3: Eliminación de todos los certificados incorrectos o expirados del nodo

En un nodo con errores de un símbolo del sistema de PowerShell con privilegios elevados, ejecute estos comandos:

```powershell
$certs = @()
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC"}
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC-OaaS Client Authentication"}
$certs += dir cert:\localmachine\CA | ?{$_.subject -like "CN=AzureDSCExtension*"}
"";"== DSC Certificates found: " + $certs.Count
$certs | FL ThumbPrint,FriendlyName,Subject
If (($certs.Count) -gt 0)
{ 
    ForEach ($Cert in $certs) 
    {
        RD -LiteralPath ($Cert.Pspath) 
    }
}
```

#### <a name="step-4-reregister-the-failing-node"></a>Paso 4: repetición del registro del nodo con errores

1. En Azure Portal, vaya a **Inicio** > **Cuentas de Automation** > (su cuenta de Automation) > **State Configuration (DSC)** .
1. Seleccione **Nodos**.
1. Seleccione **Agregar**.
1. Seleccione el nodo con errores.
1. Seleccione **Conectar** y, luego, las opciones deseadas.

## <a name="scenario-node-is-in-failed-status-with-a-not-found-error"></a><a name="failed-not-found"></a>Escenario: el nodo se encuentra en estado de error con el error "No encontrado"

### <a name="issue"></a>Incidencia

El nodo tiene un informe con el estado Con errores y contiene el error:

```error
The attempt to get the action from server https://<url>//accounts/<account-id>/Nodes(AgentId=<agent-id>)/GetDscAction failed because a valid configuration <guid> cannot be found.
```

### <a name="cause"></a>Causa

Este error suele ocurrir cuando se asigna al nodo un nombre de configuración, como **ABC**, en lugar de un nombre de configuración de nodo (archivo MOF), como **ABC.WebServer**.

### <a name="resolution"></a>Solución

* Asegúrese de estar asignando al nodo un nombre de configuración de nodo y no el nombre de configuración.
* Puede asignar una configuración de nodo a un nodo mediante Azure Portal o con un cmdlet de PowerShell.

  * En Azure Portal, vaya a **Inicio** > **Cuentas de Automation** > (su cuenta de Automation) > **State Configuration (DSC)** . Seleccione un nodo y, luego, **Asignar configuración de nodo**.
  * Use el cmdlet [Set-AzAutomationDscNode](/powershell/module/Az.Automation/Set-AzAutomationDscNode).

## <a name="scenario-no-node-configurations-mof-files-were-produced-when-a-configuration-was-compiled"></a><a name="no-mof-files"></a>Escenario: no se produjeron configuraciones de nodo (archivos MOF) al compilar una configuración

### <a name="issue"></a>Incidencia

Su trabajo de compilación de DSC suspende con el error:

```error
Compilation completed successfully, but no node configuration **.mof** files were generated.
```

### <a name="cause"></a>Causa

Cuando la expresión que aparece junto a la palabra clave `Node` en la configuración de DSC se evalúa en `$null`, no se produce ninguna configuración de nodo.

### <a name="resolution"></a>Solución

Use cualquiera de las siguientes soluciones para corregir el problema.

* Asegúrese de que la expresión junto a la palabra clave `Node` en la definición de configuración no se está evaluando en Null.
* Si pasa [ConfigurationData](../automation-dsc-compile.md) al compilar la configuración, asegúrese de que pasa los valores esperados que la configuración necesita de los datos de configuración.

## <a name="scenario-the-dsc-node-report-becomes-stuck-in-the-in-progress-state"></a><a name="dsc-in-progress"></a>Escenario: el informe de nodo de DSC se queda bloqueado en el estado En curso

### <a name="issue"></a>Incidencia

El agente DSC genera la salida:

```error
No instance found with given property values
```

### <a name="cause"></a>Causa

Ha actualizado la versión de Windows Management Framework (WMF) y ha dañado la de Instrumental de administración de Windows (WMI).

### <a name="resolution"></a>Solución

Siga las instrucciones de [Problemas y limitaciones conocidos en DSC](/powershell/scripting/wmf/known-issues/known-issues-dsc).

## <a name="scenario-unable-to-use-a-credential-in-a-dsc-configuration"></a><a name="issue-using-credential"></a>Escenario: no se puede usar una credencial en una configuración de DSC

### <a name="issue"></a>Incidencia

El trabajo de compilación de DSC se ha suspendido con el error:

```error
System.InvalidOperationException error processing property 'Credential' of type <some resource name>: Converting and storing an encrypted password as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
```

### <a name="cause"></a>Causa

Ha usado una credencial en una configuración, pero no ha proporcionado el valor adecuado de `ConfigurationData` para establecer `PSDscAllowPlainTextPassword` en true para cada configuración de nodo.

### <a name="resolution"></a>Solución

Asegúrese de pasar el valor adecuado de `ConfigurationData` para establecer `PSDscAllowPlainTextPassword` en True para cada configuración de nodo mencionada en la configuración. Consulte [Compilación de configuraciones de DSC en Azure Automation State Configuration](../automation-dsc-compile.md).

## <a name="scenario-failure-processing-extension-error-when-enabling-a-machine-from-a-dsc-extension"></a><a name="failure-processing-extension"></a>Escenario: error de procesamiento de la extensión al habilitar una máquina desde una extensión DSC

### <a name="issue"></a>Incidencia

Al habilitar una máquina mediante una extensión DSC, se produce un error que contiene lo siguiente:

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'. Error message: \"DSC COnfiguration 'RegistrationMetaConfigV2' completed with error(s). Following are the first few: Registration of the Dsc Agent with the server <url> failed. The underlying error is: The attempt to register Dsc Agent with Agent Id <ID> with the server <url> return unexpected response code BadRequest. .\".
```

### <a name="cause"></a>Causa

Este error suele producirse cuando se asigna al nodo un nombre de configuración que no existe en el servicio.

### <a name="resolution"></a>Solución

* Asegúrese de que asigna al nodo un nombre que coincida exactamente con el nombre del servicio.
* Puede elegir no incluir el nombre de configuración del nodo, lo que da lugar a la habilitación del nodo, pero no a la asignación de una configuración del nodo.

## <a name="scenario-one-or-more-errors-occurred-error-when-registering-a-node-by-using-powershell"></a><a name="cross-subscription"></a>Escenario: error "Se produjeron uno o más errores" al registrar un nodo con PowerShell

### <a name="issue"></a>Incidencia

Al registrar un nodo mediante [Register-AzAutomationDSCNode](/powershell/module/az.automation/register-azautomationdscnode) o [Register-AzureRMAutomationDSCNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode), se recibirá el siguiente error:

```error
One or more errors occurred.
```

### <a name="cause"></a>Causa

Este error se produce cuando se intenta registrar un nodo en una suscripción independiente de la que usa la cuenta de Automation.

### <a name="resolution"></a>Solución

Trate el nodo entre suscripciones como si se hubiera definido para una nube independiente o local. Registre el nodo mediante una de estas opciones para habilitar las máquinas:

* Windows: [Máquinas físicas y virtuales con Windows locales o en una nube que no sea Azure/AWS](../automation-dsc-onboarding.md#enable-physicalvirtual-windows-machines).
* Linux: [Máquinas físicas y virtuales Linux locales o en una nube que no sea Azure](../automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines).

## <a name="scenario-provisioning-has-failed-error-message"></a><a name="agent-has-a-problem"></a>Escenario: mensaje de error: "Error en el aprovisionamiento"

### <a name="issue"></a>Incidencia

Al registrar un nodo, se muestra el error:

```error
Provisioning has failed
```

### <a name="cause"></a>Causa

Este mensaje se produce cuando hay un problema de conectividad entre el nodo y Azure.

### <a name="resolution"></a>Solución

Determine si el nodo está en una red virtual privada (VPN) o si tiene otros problemas para conectarse a Azure. Vea [Solución de incidencias en la implementación de características](onboarding.md).

## <a name="scenario-failure-with-a-general-error-when-applying-a-configuration-in-linux"></a><a name="failure-linux-temp-noexec"></a>Escenario: error general al aplicar una configuración en Linux

### <a name="issue"></a>Incidencia

Al aplicar una configuración en Linux, se produce un error que contiene lo siguiente:

```error
This event indicates that failure happens when LCM is processing the configuration. ErrorId is 1. ErrorDetail is The SendConfigurationApply function did not succeed.. ResourceId is [resource]name and SourceInfo is ::nnn::n::resource. ErrorMessage is A general error occurred, not covered by a more specific error code..
```

### <a name="cause"></a>Causa

Si la ubicación **/tmp** se establece en `noexec`, la versión actual de DSC no puede aplicar las configuraciones.

### <a name="resolution"></a>Solución

Quite la opción `noexec` de la ubicación **/tmp**.

## <a name="scenario-node-configuration-names-that-overlap-can-result-in-a-bad-release"></a><a name="compilation-node-name-overlap"></a>Escenario: los nombres de configuración de nodos que se superponen pueden provocar una versión incorrecta

### <a name="issue"></a>Incidencia

Si usa un único script de configuración para generar varias configuraciones de nodo y algunos nombres de configuración de nodos son subconjuntos de otros nombres, el servicio de compilación puede acabar asignando la configuración incorrecta. Este problema solo se produce al utilizar un único script para generar configuraciones con datos de configuración por nodo y solo cuando la superposición de los nombres tiene lugar al principio de la cadena. Un ejemplo es un script de configuración único para generar configuraciones basadas en los datos de nodo pasados como una tabla hash mediante cmdlets, y los datos del nodo incluyen servidores denominados **server** y **1server**.

### <a name="cause"></a>Causa

Este es un problema conocido con el servicio de compilación.

### <a name="resolution"></a>Solución

La mejor solución sería realizar la compilación localmente o en una canalización de CI/CD y cargar los archivos MOF de configuración de nodos directamente en el servicio. Si es necesario que la compilación tenga lugar en el servicio, la siguiente mejor solución alternativa es dividir los trabajos de compilación para que no haya nombres superpuestos.

## <a name="scenario-gateway-timeout-error-on-dsc-configuration-upload"></a><a name="gateway-timeout"></a>Escenario: Error de tiempo de espera de puerta de enlace en la carga de la configuración de DSC

#### <a name="issue"></a>Incidencia

Recibe un error `GatewayTimeout` al cargar una configuración de DSC. 

### <a name="cause"></a>Causa

Las configuraciones de DSC que tardan mucho tiempo en compilarse pueden producir este error.

### <a name="resolution"></a>Solución

Las configuraciones de DSC se pueden analizar más rápidamente si se incluye expresamente el parámetro `ModuleName` en cualquier llamada [Import-DSCResource](/powershell/scripting/dsc/configurations/import-dscresource).

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece aquí o no puede resolverlo, intente obtener ayuda adicional mediante uno de los siguientes canales:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. El Soporte técnico de Azure pone en contacto a la comunidad de Azure con respuestas, soporte técnico y expertos.
* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.
