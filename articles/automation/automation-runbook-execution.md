---
title: Ejecución de un runbook en Azure Automation
description: En este artículo se proporciona información general del procesamiento de runbooks en Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 03/23/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0807b11adfc46b9c32a8f7bd36a2f7d4db519975
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107830527"
---
# <a name="runbook-execution-in-azure-automation"></a>Ejecución de un runbook en Azure Automation

La automatización de procesos en Azure Automation permite crear y administrar PowerShell, el flujo de trabajo de PowerShell y los runbooks gráficos. Para ver los detalles, consulte [Runbooks de Azure Automation](automation-runbook-types.md). 

Automation ejecuta los runbooks según la lógica definida dentro de ellos. Si se interrumpe un runbook, se reinicia desde el principio. Este comportamiento requiere que escriba runbooks que admitan el reinicio si se producen problemas transitorios.

Al iniciar un runbook en Azure Automation se crea un trabajo, que es una instancia de ejecución única del runbook. Cada trabajo accede a los recursos de Azure mediante una conexión a la suscripción a Azure. El rol de trabajo solo accede a los recursos de su centro de datos si se puede acceder a estos recursos desde la nube pública.

Azure Automation asigna un rol de trabajo para ejecutar cada trabajo durante la ejecución de un runbook. Aunque los trabajadores los comparten varias cuentas de Azure, los trabajos de diferentes cuentas de Automation están aislados entre sí. No puede controlar qué servicios de rol de trabajo solicita su trabajo.

Al ver la lista de runbooks en Azure Portal, se muestra el estado de cada trabajo iniciado para cada runbook. Azure Automation almacena los registros de trabajo durante un máximo de 30 días.

En el siguiente diagrama se muestra el ciclo de vida de un trabajo de runbook para [runbooks de PowerShell](automation-runbook-types.md#powershell-runbooks), [runbooks de flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) y [runbooks gráficos](automation-runbook-types.md#graphical-runbooks).

![Estados de trabajo: flujo de trabajo de PowerShell](./media/automation-runbook-execution/job-statuses.png)

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="runbook-execution-environment"></a>Entorno de ejecución de runbooks

Los runbooks de Azure Automation puede ejecutarse en un espacio aislado de Azure o una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md). 

Cuando los runbooks están diseñados para autenticarse y ejecutarse en recursos de Azure, se ejecutan en un espacio aislado de Azure, que es un entorno compartido que pueden usar varios trabajos. Los trabajos con el mismo espacio aislado están sujetos a las limitaciones de recursos de dicho espacio. El entorno de espacio aislado de Azure no admite operaciones interactivas. Impide el acceso a todos los servidores COM fuera de proceso y no admite la realización de [llamadas de WMI](/windows/win32/wmisdk/wmi-architecture) al proveedor Win32 en el runbook.  Estos escenarios solo se admiten si se ejecuta el runbook en una instancia de Hybrid Runbook Worker de Windows.


También puede usar una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md) para ejecutar runbooks directamente en el equipo que hospeda el rol y en los recursos locales del entorno. Azure Automation almacena y administra los runbooks y después los entrega a uno o más equipos asignados.

>[!NOTE]
>Para ejecutarse en una instancia de Hybrid Runbook Worker en Linux, los scripts deben estar firmados y el trabajo configurado en consecuencia. Como alternativa, la [validación de la firma debe estar desactivada](automation-linux-hrw-install.md#turn-off-signature-validation).

En la tabla siguiente se enumeran algunas tareas de ejecución de runbooks con el entorno de ejecución recomendado que se muestra para cada una.

|Tarea|Recomendación|Notas|
|---|---|---|
|Integración con recursos de Azure|Espacio aislado de Azure|Hospedado en Azure, la autenticación es más sencilla. Si usa una instancia de Hybrid Runbook Worker en una VM de Azure, puede [usar la autenticación de runbooks con identidades administradas](automation-hrw-run-runbooks.md#runbook-auth-managed-identities).|
|Obtención del rendimiento óptimo para administrar recursos de Azure|Espacio aislado de Azure|El script se ejecuta en el mismo entorno, que tiene menos latencia.|
|Minimizar los costos operativos|Espacio aislado de Azure|No hay sobrecarga de proceso y no hay necesidad de una VM.|
|Ejecución de script de ejecución prolongada|Hybrid Runbook Worker|Los espacios aislados de Azure tienen [límites de recursos](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).|
|Interacción con servicios locales|Hybrid Runbook Worker|Acceda directamente a la máquina host, a los recursos de otros entornos en la nube o en el entorno local. |
|Requerimiento de software de terceros y archivos ejecutables|Hybrid Runbook Worker|Usted administra el sistema operativo y puede instalar el software.|
|Supervisar un archivo o carpeta con un runbook|Hybrid Runbook Worker|Use una [tarea del Monitor](automation-watchers-tutorial.md) en una instancia de Hybrid Runbook Worker.|
|Ejecución de un script con un uso intensivo de recursos|Hybrid Runbook Worker| Los espacios aislados de Azure tienen [límites de recursos](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).|
|Uso de módulos con requisitos específicos| Hybrid Runbook Worker|Ejemplos:</br> WinSCP: dependencia de winscp.exe </br> Administración de IIS: dependencia de la habilitación o administración de IIS|
|Instalación de un módulo con un instalador|Hybrid Runbook Worker|Los módulos para espacios aislados deben admitir copias.|
|Uso de runbooks o módulos que requieren .NET Framework con una versión distinta de 4.7.2|Hybrid Runbook Worker|Los espacios aislados de Azure son compatibles con .NET Framework 4.7.2 y no se admite la actualización a una versión diferente.|
|Ejecución de scripts que requieren elevación|Hybrid Runbook Worker|Los espacios aislados no permiten elevación. Con una instancia de Hybrid Runbook Worker, puede desactivar UAC y usar [Invoke-Command](/powershell/module/microsoft.powershell.core/invoke-command) al ejecutar el comando que requiere elevación.|
|Ejecución de scripts que requieren acceso a Instrumental de administración de Windows (WMI)|Hybrid Runbook Worker|Los trabajos que se ejecutan en espacios aislados en la nube no pueden acceder al proveedor de WMI. |

## <a name="temporary-storage-in-a-sandbox"></a>Almacenamiento temporal en un espacio aislado

Si necesita crear archivos temporales como parte de la lógica de runbook, puede usar la carpeta temporal (es decir, `$env:TEMP`) del espacio aislado de Azure para los runbooks que se ejecutan en Azure. La única limitación es que no se puede usar más de 1 GB de espacio en disco, que es la cuota de cada espacio aislado. Al trabajar con flujos de trabajo de PowerShell, este escenario puede generar un problema porque los flujos de trabajo de PowerShell usan puntos de control y el script se podría volver a intentar en otro espacio aislado.

Con el espacio aislado híbrido, puede usar `C:\temp` en función de la disponibilidad del almacenamiento en una instancia de Hybrid Runbook Worker. Sin embargo, según las recomendaciones de VM de Azure, no debe usar el [disco temporal](../virtual-machines/managed-disks-overview.md#temporary-disk) en Windows o Linux para los datos que se deben conservar.

## <a name="resources"></a>Recursos

Los runbooks deben incluir lógica para tratar con [recursos](/rest/api/resources/resources), por ejemplo, las máquinas virtuales, la red y los recursos de la red. Los recursos están vinculados a una suscripción de Azure y los runbooks requieren credenciales adecuadas para acceder a cualquier recurso. Para ver un ejemplo de cómo administrar los recursos de un runbook, consulte el artículo sobre [administración de recursos](manage-runbooks.md#handle-resources).

## <a name="security"></a>Seguridad

Azure Automation usa [Azure Security Center (ASC)](../security-center/security-center-introduction.md) para proporcionar seguridad a los recursos y detectar riesgos en los sistemas Linux. La seguridad se proporciona en todas las cargas de trabajo, independientemente de si los recursos se encuentran en Azure o no. Consulte [Introducción a la autenticación en Azure Automation](automation-security-overview.md).

ASC coloca restricciones en los usuarios que pueden ejecutar scripts, firmados o sin firmar, en una máquina virtual. Si es un usuario con acceso de raíz a una máquina virtual, debe configurar explícitamente la máquina con una firma digital o desactivarlo. De lo contrario, solo puede ejecutar un script para aplicar actualizaciones del sistema operativo después de crear una cuenta de Automation y habilitar la característica adecuada.

## <a name="subscriptions"></a>Suscripciones

Una [suscripción](/office365/enterprise/subscriptions-licenses-accounts-and-tenants-for-microsoft-cloud-offerings) a Azure es un contrato con Microsoft para usar uno o más servicios basados en la nube, por los que se le cobra. En el caso de Azure Automation, cada suscripción está vinculada a una cuenta de Azure Automation y puede [crear varias suscripciones](manage-runbooks.md#work-with-multiple-subscriptions) en la cuenta.

## <a name="credentials"></a>Credenciales

Un runbook requiere [credenciales](shared-resources/credentials.md) adecuadas para acceder a cualquier recurso, ya sea en sistemas de Azure o de terceros. Estas credenciales se almacenan en Azure Automation, Key Vault, etc.  

## <a name="azure-monitor"></a>Azure Monitor

Azure Automation usa [Azure Monitor](../azure-monitor/overview.md) para supervisar las operaciones de sus máquinas. Las operaciones requieren un área de trabajo de Log Analytics y un [agente de Log Analytics](../azure-monitor/agents/log-analytics-agent.md).

### <a name="log-analytics-agent-for-windows"></a>Agente de Log Analytics para Windows

El agente de [Log Analytics para Windows](../azure-monitor/agents/agent-windows.md) funciona con Azure Monitor para administrar equipos físicos y máquinas virtuales Windows. Los equipos pueden ejecutarse en un entorno de Azure o que no sea de Azure, como un centro de datos local.

>[!NOTE]
>El agente de Log Analytics para Windows se denominaba anteriormente Microsoft Monitoring Agent (MMA).

### <a name="log-analytics-agent-for-linux"></a>Agente de Log Analytics para Linux

El agente de [Log Analytics para Linux](../azure-monitor/agents/agent-linux.md) funciona de forma similar al agente para Windows, pero conecta equipos Linux a Azure Monitor. El agente se instala con una cuenta de usuario **nxautomation** que permite la ejecución de comandos que requieren permisos de raíz, por ejemplo, en una instancia de Hybrid Runbook Worker. La cuenta **nxautomation** es una cuenta del sistema que no requiere contraseña.

La cuenta **nxautomation** con los permisos sudo correspondientes debe estar presente durante la [instalación de la instancia de Hybrid Runbook Worker en Linux](automation-linux-hrw-install.md). Si intenta instalar el rol de trabajo y la cuenta no está presente o no tiene los permisos adecuados, no se realizará la instalación.

No debe cambiar los permisos de la carpeta `sudoers.d` ni su propiedad. Se requiere permiso sudo para la cuenta **nxautomation** y no se deben quitar los permisos. Su restricción a determinadas carpetas o comandos puede dar lugar a un cambio importante.

Los registros disponibles para el agente de Log Analytics y la cuenta **nxautomation** son:

* /var/opt/microsoft/omsagent/log/omsagent.log: registro de agente de Log Analytics
* /var/opt/microsoft/omsagent/run/automationworker/worker.log: registro de trabajo de Automation

>[!NOTE]
>El usuario **nxautomation** habilitado como parte de Update Management solo ejecuta runbooks firmados.

## <a name="runbook-permissions"></a>Permisos de runbooks

Un runbook necesita permisos para la autenticación en Azure a través de credenciales. Consulte [Introducción a la autenticación de Azure Automation](automation-security-overview.md).

## <a name="modules"></a>Módulos

Azure Automation admite varios módulos predeterminados, como algunos módulos AzureRM (AzureRM.Automation) y un módulo que contiene varios cmdlets internos. También se admiten módulos instalables, incluidos los módulos Az (Az.Automation), que actualmente se usan actualmente en lugar de los módulos AzureRM. Para más información sobre los módulos que están disponibles para sus runbooks y configuraciones de DSC, consulte [Administración de módulos en Azure Automation](shared-resources/modules.md).

## <a name="certificates"></a>Certificados

Azure Automation usa [certificados](shared-resources/certificates.md) para la autenticación en Azure o los agrega a recursos de Azure o de terceros. Los certificados se almacenan de forma segura para el acceso mediante runbooks y configuraciones de DSC.

Los runbooks pueden usar certificados autofirmados, que no están firmados por una entidad de certificación (CA). Consulte [Creación de un certificado nuevo](shared-resources/certificates.md#create-a-new-certificate).

## <a name="jobs"></a>Trabajos

Azure Automation admite un entorno para ejecutar trabajos de la misma cuenta de automatización. Un runbook individual puede tener muchos trabajos que se ejecutan al mismo tiempo. Cuantos más trabajos se ejecuten al mismo tiempo, más a menudo se podrán enviar al mismo espacio aislado. 

Los trabajos que se ejecutan en el mismo proceso de espacio aislado pueden afectarse entre sí. Un ejemplo es la ejecución del cmdlet [Disconnect-AzAccount](/powershell/module/az.accounts/disconnect-azaccount). La ejecución de este cmdlet desconecta cada trabajo de runbook en el proceso de espacio aislado compartido. Para ver un ejemplo de cómo trabajar en este escenario, consulte [Prevención de trabajos simultáneos](manage-runbooks.md#prevent-concurrent-jobs).

>[!NOTE]
>Es posible que los trabajos de PowerShell iniciados desde un runbook que se ejecuta en un espacio aislado de Azure no se puedan ejecutar en [modo de lenguaje de PowerShell](/powershell/module/microsoft.powershell.core/about/about_language_modes) completo.

### <a name="job-statuses"></a>Estados del trabajo

En la tabla siguiente se describen los estados posibles para un trabajo. Puede ver un resumen del estado de todos los trabajos del runbook o profundizar en los detalles de un trabajo específico del runbook en Azure Portal. También puede configurar la integración con el área de trabajo de Log Analytics para reenviar flujos de trabajos y el estado del trabajo del runbook. Para más información sobre la integración con los registros de Azure Monitor, consulte [Reenvío del estado del trabajo y flujos de trabajo de Automation a los registros de Azure Monitor](automation-manage-send-joblogs-log-analytics.md). Consulte también [Obtención de estados del trabajo](manage-runbooks.md#obtain-job-statuses) para ver un ejemplo de cómo trabajar con estados en un runbook.

| Status | Descripción |
|:--- |:--- |
| En activación |El trabajo se está activando. |
| Completed |El trabajo se completó correctamente. |
| Con error |Un runbook de flujo de trabajo de PowerShell o gráfico no se pudo compilar. Un runbook de PowerShell no se pudo iniciar o el trabajo generó una excepción. Consulte [Tipos de runbooks de Azure Automation](automation-runbook-types.md).|
| Error, esperando recursos |Se produjo un error en el trabajo porque ha alcanzado el límite de [distribución equilibrada](#fair-share) tres veces y se ha iniciado en el mismo punto de control o en el inicio del runbook cada vez. |
| En cola |El trabajo espera que los recursos en un trabajo de Automation estén disponibles para que se pueda iniciar. |
| Reanudando |El sistema está reanudando el trabajo después de que se suspendió. |
| En ejecución |El trabajo se está ejecutando. |
| En ejecución, esperando recursos |El trabajo se ha descargado porque ha alcanzado el límite de distribución equilibrada. Se reanudará en breve desde su último punto de control. |
| Iniciando |El trabajo se ha asignado a un trabajador y el sistema lo está iniciando. |
| Detenido |El trabajo lo detuvo el usuario antes de completarse. |
| Deteniéndose |El sistema está deteniendo el trabajo. |
| Suspended |Solo se aplica a [Runbooks de flujo de trabajo de PowerShell y gráficos](automation-runbook-types.md). El trabajo lo ha suspendido el usuario, el sistema o un comando en el runbook. Si un runbook no tiene un punto de control, se inicia desde el principio. Si lo tiene, puede volver a empezar y reanudarse desde su último punto de comprobación. El sistema solo suspende el runbook cuando se produce una excepción. De forma predeterminada, la variable `ErrorActionPreference` se establece en Continue, lo que indica que el trabajo sigue ejecutándose tras un error. Si la variable de preferencia se establece en Stop, se suspende el trabajo tras un error.  |
| Suspendiendo |Solo se aplica a [Runbooks de flujo de trabajo de PowerShell y gráficos](automation-runbook-types.md). El sistema intenta suspender el trabajo a petición del usuario. El runbook debe alcanzar su siguiente punto de control antes de que se pueda suspender. Si ya ha pasado su último punto de control, se completa antes de que se pueda suspender. |

## <a name="activity-logging"></a>Registro de actividad

La ejecución de runbooks en Azure Automation escribe los detalles en un registro de actividad de la cuenta de Automation. Para más información sobre el uso del registro, consulte [Recuperación de detalles del registro de actividad](manage-runbooks.md#retrieve-details-from-activity-log).

## <a name="exceptions"></a>Excepciones

En esta sección se describen algunas maneras de controlar excepciones o problemas intermitentes en los runbooks. Un ejemplo es una excepción de WebSocket. El control de excepciones correcto evita que los errores de red transitorios provoquen un error en los runbooks.

### <a name="erroractionpreference"></a>ErrorActionPreference

La variable [ErrorActionPreference](/powershell/module/microsoft.powershell.core/about/about_preference_variables#erroractionpreference) determina la manera en que PowerShell responde a un error de no terminación. Los errores de terminación siempre terminan y no se ven afectados por `ErrorActionPreference`.

Cuando el runbook usa `ErrorActionPreference`, un error normalmente de no terminación, como `PathNotFound` del cmdlet [Get-ChildItem](/powershell/module/microsoft.powershell.management/get-childitem), impide que el runbook se complete. En el siguiente ejemplo se muestra el uso de `ErrorActionPreference`. El comando final [Write-Output](/powershell/module/microsoft.powershell.utility/write-output) nunca se ejecuta, a medida que se detiene el script.

```powershell-interactive
$ErrorActionPreference = 'Stop'
Get-ChildItem -path nofile.txt
Write-Output "This message will not show"
```

### <a name="try-catch-finally"></a>Try Catch Finally

[Try Catch Finally](/powershell/module/microsoft.powershell.core/about/about_try_catch_finally) se usa en los scripts de PowerShell para controlar los errores de terminación. El script puede usar este mecanismo para detectar excepciones específicas o excepciones generales. La instrucción `catch` se debe usar para realizar el seguimiento de los errores o para intentar controlarlos. En el ejemplo siguiente se intenta descargar un archivo que no existe. Detecta la excepción `System.Net.WebException` y devuelve el último valor para cualquier otra excepción.

```powershell-interactive
try
{
   $wc = new-object System.Net.WebClient
   $wc.DownloadFile("http://www.contoso.com/MyDoc.doc")
}
catch [System.Net.WebException]
{
    "Unable to download MyDoc.doc from http://www.contoso.com."
}
catch
{
    "An error occurred that could not be resolved."
}
```

### <a name="throw"></a>Throw

[Throw](/powershell/module/microsoft.powershell.core/about/about_throw) se puede usar para generar un error de terminación. Este mecanismo puede ser útil al definir su propia lógica en un runbook. Si el script cumple un criterio que debe detenerlo, puede usar la instrucción `throw` para la detención. En el ejemplo siguiente se utiliza esta instrucción para mostrar un parámetro de función necesario.

```powershell-interactive
function Get-ContosoFiles
{
  param ($path = $(throw "The Path parameter is required."))
  Get-ChildItem -Path $path\*.txt -recurse
}
```

## <a name="errors"></a>Errors

Los runbooks deben controlar los errores. Azure Automation admite dos tipos de errores de PowerShell, de terminación y de no terminación. 

Los errores de terminación detienen la ejecución del runbook cuando se producen. El runbook se detiene con un estado de trabajo de Error.

Los errores de no terminación permiten que un script continúe después de que sucedan. Un ejemplo de error de no terminación es aquel donde un runbook usa el cmdlet `Get-ChildItem` con una ruta de acceso que no existe. PowerShell se ve que la ruta de acceso no existe, genera un error y pasa a la carpeta siguiente. En este caso, el error no establece el estado del trabajo del runbook en Con error y el trabajo podría incluso completarse. Para forzar a un runbook a detenerse ante un error de no terminación, puede usar `ErrorAction Stop` en el cmdlet.

## <a name="calling-processes"></a>Llamada a procesos

Los runbooks que se ejecutan en espacios aislados de Azure no admiten la llamada a procesos, por ejemplo, ejecutables (archivos **.exe**) ni subprocesos. El motivo es que un espacio aislado de Azure es un proceso compartido que se ejecuta en un contenedor que podría no poder acceder a todas las API subyacentes. En escenarios donde se requiere software de terceros o llamadas a subprocesos, debe ejecutar un runbook en una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).

## <a name="device-and-application-characteristics"></a>Características de dispositivos y aplicaciones

Los trabajos de runbook en espacios aislados de Azure no pueden acceder a las características del dispositivo o la aplicación. La API más común que se usa para consultar las métricas de rendimiento en Windows es WMI, donde algunas de las métricas comunes son la memoria y el uso de CPU. Sin embargo, no importa qué API se use, ya que los trabajos que se ejecutan en la nube no pueden acceder a la implementación de Microsoft de Web-Based Enterprise Management (WBEM). Esta plataforma se basa en el Modelo de información común (CIM), que proporciona los estándares del sector para definir las características de dispositivos y aplicaciones.

## <a name="webhooks"></a>webhooks

Los servicios externos, como Azure DevOps Services y GitHub, pueden iniciar un runbook en Azure Automation. Para realizar este tipo de inicio, el servicio usa un [webhook](automation-webhooks.md) a través de una única solicitud HTTP. El uso de un webhook permite iniciar runbooks sin necesidad de implementar una característica completa de Azure Automation.

## <a name="shared-resources"></a><a name="fair-share"></a>Recursos compartidos

Para compartir recursos entre todos los runbooks de la nube, Azure usa un concepto denominado distribución equilibrada. Con distribución equilibrada, Azure descarga temporalmente o detiene cualquier trabajo que se haya ejecutado durante más de tres horas. Los trabajos de [runbooks de PowerShell](automation-runbook-types.md#powershell-runbooks) y [runbooks de Python](automation-runbook-types.md#python-runbooks) se detienen y no se reinician, y el estado del trabajo cambia a Detenido.

En el caso de tareas de larga duración de Azure Automation, se recomienda usar una instancia de Hybrid Runbook Worker. Las instancias de Hybrid Runbook Worker no están limitadas por la distribución equilibrada y no tienen una limitación del tiempo que se puede ejecutar un runbook. Los demás [límites](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits) de trabajo se aplican a los espacios aislados de Azure y a Hybrid Runbook Workers. Aunque las instancias de Hybrid Runbook Worker no están restringidas por el límite de distribución equilibrada de tres horas, debe desarrollar los runbooks para que se ejecutan en los roles de trabajo que admitan reinicios si se producen problemas inesperados en la infraestructura local.

Otra opción es optimizar un runbook mediante runbooks secundarios. Por ejemplo, el runbook podría recorrer en bucle la misma función en varios recursos, como, por ejemplo, una operación de base de datos en varias bases de datos. Puede mover esta función a un [runbook secundario](automation-child-runbooks.md) y hace que el runbook lo llame mediante [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook). Los runbooks secundarios se ejecutan en paralelo en procesos independientes.

El uso de runbooks secundarios reduce la cantidad total de tiempo que tarda en completarse el runbook primario. El runbook puede usar el cmdlet [Get-AzAutomationJob](/powershell/module/az.automation/get-azautomationjob) para comprobar el estado del trabajo de un runbook secundario si aún hay más operaciones después de que se complete el secundario.

## <a name="next-steps"></a>Pasos siguientes

* Para empezar a trabajar con un runbook de PowerShell, consulte [Tutorial: Creación de un runbook de PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Para trabajar con runbooks, consulte [Administración de runbooks en Azure Automation](manage-runbooks.md).
* Para más información sobre PowerShell, consulte [PowerShell Docs](/powershell/scripting/overview).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation#automation).
