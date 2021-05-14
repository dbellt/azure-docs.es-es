---
title: Administración de módulos en Azure Automation
description: En este artículo se explica cómo usar los módulos de PowerShell para habilitar cmdlets en runbooks y recursos de DSC en configuraciones de DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 04/28/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 48888f9ca840888310aebcc82d38d2af351a8611
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108277901"
---
# <a name="manage-modules-in-azure-automation"></a>Administración de módulos en Azure Automation

Azure Automation usa una serie de módulos de PowerShell para habilitar cmdlets en runbooks y recursos de DSC en configuraciones de DSC. Los módulos compatibles incluyen:

* [Azure PowerShell Az.Automation](/powershell/azure/new-azureps-module-az)
* [Azure PowerShell AzureRM.Automation](/powershell/module/azurerm.automation/)
* Otros módulos de PowerShell
* Módulo `Orchestrator.AssetManagement.Cmdlets` interno
* Módulos de Python 2
* Módulos personalizados creados por el usuario

Cuando se crea una cuenta de Automation, Azure Automation importa algunos módulos de forma predeterminada. Consulte [Módulos predeterminados](#default-modules).

## <a name="sandboxes"></a>Espacios aislados

Cuando Automation ejecuta trabajos de compilación de runbook y DSC, carga los módulos en espacios aislados en los que se pueden ejecutar los runbooks y se pueden compilar las configuraciones de DSC. Automation también coloca automáticamente los recursos de DSC en los módulos del servidor de extracción de DSC. Las máquinas pueden extraer los recursos cuando aplican las configuraciones de DSC.

>[!NOTE]
>Asegúrese de importar solo los módulos necesarios para los runbooks y las configuraciones de DSC. No es recomendable importar el módulo Az raíz, ya que incluye muchos otros módulos que es posible que no necesite, lo que puede causar problemas de rendimiento. En su lugar, importe módulos individuales, como Az.Compute.

El espacio aislado de nube admite 48 llamadas del sistema como máximo y, por motivos de seguridad, restringe todas las demás. Otras funcionalidades, como la administración de credenciales y algunas redes, no se admiten en el espacio aislado de nube.

Debido al número de módulos y cmdlets incluidos, es difícil saber de antemano cuál de los cmdlets realizará llamadas no admitidas. Por lo general, se han detectado problemas con cmdlets que requieren acceso con privilegios elevados, que requieren una credencial como parámetro o con cmdlets relacionados con redes. Los cmdlets que realizan operaciones de red de pila completa no se admiten en el espacio aislado, lo que incluye [Connect-AipService](/powershell/module/aipservice/connect-aipservice) del módulo AIPService de PowerShell y [Resolve-DnsName](/powershell/module/dnsclient/resolve-dnsname) del módulo DNSClient.

Estas son limitaciones conocidas del espacio aislado. La solución recomendada es implementar una instancia de [Hybrid Runbook Worker](../automation-hybrid-runbook-worker.md) o usar [Azure Functions](../../azure-functions/functions-overview.md).

## <a name="default-modules"></a>Módulos predeterminados

En la tabla siguiente se enumeran los módulos que importa Azure Automation de forma predeterminada al crear la cuenta de Automation. Automation puede importar versiones más recientes de estos módulos. Pero no se puede quitar la versión original de la cuenta de Automation aunque se elimine una versión más reciente. Tenga en cuenta que estos módulos predeterminados incluyen varios módulos de AzureRM.

Los módulos predeterminados también se conocen como módulos globales. En Azure Portal, la propiedad **Módulo global** será **true** al ver un módulo que se importó al crear la cuenta.

![Captura de pantalla de la propiedad Módulo global en Azure portal](../media/modules/automation-global-modules.png)

Automation no importa el módulo Az raíz automáticamente en ninguna cuenta de Automation nueva o existente. Para más información sobre cómo trabajar con estos módulos, consulte [Migración a los módulos Az](#migrate-to-az-modules).

> [!NOTE]
> No es recomendable alterar módulos y runbooks de cuentas de Automation utilizadas para la implementación de la característica [Start/Stop VMs during off-hours](../automation-solution-vm-management.md).

|Nombre del módulo|Versión|
|---|---|
| AuditPolicyDsc | 1.1.0.0 |
| Azure | 1.0.3 |
| Azure.Storage | 1.0.3 |
| AzureRM.Automation | 1.0.3 |
| AzureRM.Compute | 1.2.1 |
| AzureRM.Profile | 1.0.3 |
| AzureRM.Resources | 1.0.3 |
| AzureRM.Sql | 1.0.3 |
| AzureRM.Storage | 1.0.3 |
| ComputerManagementDsc | 5.0.0.0 |
| GPRegistryPolicyParser | 0,2 |
| Microsoft.PowerShell.Core | 0 |
| Microsoft.PowerShell.Diagnostics |  |
| Microsoft.PowerShell.Management |  |
| Microsoft.PowerShell.Security |  |
| Microsoft.PowerShell.Utility |  |
| Microsoft.WSMan.Management |  |
| Orchestrator.AssetManagement.Cmdlets | 1 |
| PSDscResources | 2.9.0.0 |
| SecurityPolicyDsc | 2.1.0.0 |
| StateConfigCompositeResources | 1 |
| xDSCDomainjoin | 1.1 |
| xPowerShellExecutionPolicy | 1.1.0.0 |
| xRemoteDesktopAdmin | 1.1.0.0 |

## <a name="az-modules"></a>Modules Az

En Az.Automation, la mayoría de los cmdlets tienen los mismos nombres utilizados para los módulos AzureRM, excepto que el prefijo `AzureRM` se ha cambiado por `Az`. Para obtener una lista de módulos Az que no siguen esta convención de nomenclatura, vea la [lista de excepciones](/powershell/azure/migrate-from-azurerm-to-az#update-cmdlets-modules-and-parameters).

## <a name="internal-cmdlets"></a>Cmdlets internos

Azure Automation admite cmdlets internos que solo están disponibles cuando se ejecutan runbooks en el entorno de espacio aislado de Azure o en una instancia de Hybrid Runbook Worker de Windows. El módulo interno `Orchestrator.AssetManagement.Cmdlets` se instala de forma predeterminada en la cuenta de Automation y cuando el rol Hybrid Runbook Worker de Windows está instalado en la máquina. 

En la tabla siguiente se definen los cmdlets internos. Estos cmdlets están diseñados para usarse en lugar de los cmdlets de Azure PowerShell para interactuar con recursos de la cuenta de Automation. Los cmdlets recuperan secretos a partir de variables cifradas, credenciales y conexiones cifradas.

|Nombre|Descripción|
|---|---|
|Get-AutomationCertificate|`Get-AutomationCertificate [-Name] <string> [<CommonParameters>]`|
|Get-AutomationConnection|`Get-AutomationConnection [-Name] <string> [-DoNotDecrypt] [<CommonParameters>]` |
|Get-AutomationPSCredential|`Get-AutomationPSCredential [-Name] <string> [<CommonParameters>]` |
|Get-AutomationVariable|`Get-AutomationVariable [-Name] <string> [-DoNotDecrypt] [<CommonParameters>]`|
|Set-AutomationVariable|`Set-AutomationVariable [-Name] <string> -Value <Object> [<CommonParameters>]` |
|Start-AutomationRunbook|`Start-AutomationRunbook [-Name] <string> [-Parameters <IDictionary>] [-RunOn <string>] [-JobId <guid>] [<CommonParameters>]`|
|Wait-AutomationJob|`Wait-AutomationJob -Id <guid[]> [-TimeoutInMinutes <int>] [-DelayInSeconds <int>] [-OutputJobsTransitionedToRunning] [<CommonParameters>]`|

Tenga en cuenta que los cmdlets internos difieren en la nomenclatura de los cmdlets de Az y AzureRM. Los nombres de los cmdlets internos no contienen palabras como `Azure` o `Az`, sino que usan la palabra `Automation`. Se recomienda su uso antes que el uso de los cmdlets Az o AzureRM durante la ejecución del runbook en un espacio aislado de Azure o en una instancia de Hybrid Runbook Worker de Windows porque requieren menos parámetros y se ejecutan en el contexto del trabajo durante la ejecución.

Use los cmdlets de Az o AzureRM para manipular recursos de Automation fuera del contexto de un runbook. 

## <a name="python-modules"></a>Módulos de Python

En Azure Automation se pueden crear runbooks de Python 2. Para obtener información sobre un módulo de Python, ver [Administrar paquetes de Python 2 en Azure Automation](../python-packages.md).

## <a name="custom-modules"></a>Módulos personalizados

Azure Automation admite módulos de PowerShell personalizados que se crean para usar con los runbooks y configuraciones de DSC. Un tipo de módulo personalizado es un módulo de integración que puede contener un archivo de metadatos para definir la funcionalidad personalizada para los cmdlets del módulo. Un ejemplo del uso de un módulo de integración se proporciona en [Incorporación de un tipo de conexión](../automation-connections.md#add-a-connection-type).

Azure Automation puede importar un módulo personalizado para que sus cmdlets estén disponibles. En segundo plano, almacena el módulo y lo usa en los espacios aislados de Azure, al igual que ocurre con otros módulos.

## <a name="migrate-to-az-modules"></a>Migración a módulos Az

En esta sección se explica cómo migrar a los módulos Az en Automation. Para más información, vea [Migración de Azure PowerShell de AzureRM a Az](/powershell/azure/migrate-from-azurerm-to-az).

No es recomendable ejecutar módulos Az y AzureRM en la misma cuenta de Automation. Cuando esté seguro de que quiere migrar de AzureRM a Az, es mejor confirmar completamente una migración entera. Automation suele reutilizar espacios aislados dentro de la cuenta de Automation para ahorrar en los tiempos de inicio. Si no realiza una migración completa del módulo, puede iniciar un trabajo que usa solo módulos AzureRM y luego iniciar otro trabajo que usa solo módulos Az. El espacio aislado pronto se bloquea, y recibe un error que indica que los módulos no son compatibles. Esta situación da lugar a bloqueos que se producen aleatoriamente de cualquier runbook o configuración específicos.

>[!NOTE]
>Cuando se crea una cuenta de Automation, incluso después de la migración a módulos Az, Automation instala los módulos AzureRM de forma predeterminada. Es posible actualizar los runbooks de tutorial con los cmdlets de AzureRM, aunque no debe ejecutar estos runbooks.

### <a name="test-your-runbooks-and-dsc-configurations-prior-to-module-migration"></a>Prueba de los runbooks y las configuraciones de DSC antes de la migración de módulos

Asegúrese de probar todos los runbooks y las configuraciones de DSC cuidadosamente en una cuenta de Automation independiente antes de migrar a los módulos Az. 

### <a name="stop-and-unschedule-all-runbooks-that-use-azurerm-modules"></a>Detención y anulación de la programación de todos los runbooks que usan módulos de AzureRM

Para asegurarse de no ejecutar ningún runbook ni configuración de DSC existente que use módulos AzureRM, debe detener y anular la programación de todos los runbooks y las configuraciones afectados. En primer lugar, asegúrese de revisar cada runbook o configuración de DSC y sus programaciones por separado para garantizar que puede volver a programar el elemento en el futuro en caso necesario.

Cuando esté listo para quitar las programaciones, puede usar Azure Portal o el cmdlet [Remove-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/remove-azurermautomationschedule). Consulte [Eliminación de una programación](schedules.md#remove-a-schedule).

### <a name="remove-azurerm-modules"></a>Eliminación de los módulos AzureRM

Es posible quitar los módulos AzureRM antes de importar los módulos Az. Sin embargo, si lo hace, puede interrumpir la sincronización del control de código fuente y provocar errores en los scripts que siguen programados. Si decide quitar los módulos, ver [Desinstalación de AzureRM](/powershell/azure/migrate-from-azurerm-to-az#uninstall-azurerm).

### <a name="import-az-modules"></a>Importación de módulos Az

Al importar un módulo Az en la cuenta de Automation, no se importa automáticamente ese módulo en la sesión de PowerShell que usan los runbooks. Los módulos se importan en la sesión de PowerShell en las situaciones siguientes:

* Cuando un runbook invoca un cmdlet desde un módulo.
* Cuando un runbook importa el módulo de forma explícita con el cmdlet [Import-Module](/powershell/module/microsoft.powershell.core/import-module).
* Cuando un runbook importa el módulo de forma explícita con la instrucción[using module](/powershell/module/microsoft.powershell.core/about/about_using#module-syntax). La instrucción using se admite a partir de Windows PowerShell 5.0 y es compatible con las clases y la importación de tipos de enumeración.
* Cuando un runbook importa otro módulo dependiente.

Puede importar los módulos Az en la cuenta de Automation desde Azure Portal. No olvide importar solo los módulos Az que necesite, no todos los módulos Az que estén disponibles. Dado que [Az.Accounts](https://www.powershellgallery.com/packages/Az.Accounts/1.1.0) es una dependencia para los otros módulos Az, asegúrese de importar este módulo antes que cualquier otro.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Cuentas de Automation**.
1. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.
1. En la cuenta de Automation, en **Recursos compartidos**, seleccione **Módulos**.
1. Seleccione **Explorar la galería**.  
1. En la barra de búsqueda, escriba el nombre del módulo, por ejemplo, `Az.Accounts`.
1. En la página Módulo de PowerShell, seleccione **Importar** para importar el módulo en la cuenta de Automation.

    ![Captura de pantalla de la importación de módulos a la cuenta de Automation](../media/modules/import-module.png)

También puede realizar esta importación con la [Galería de PowerShell](https://www.powershellgallery.com) mediante la búsqueda del módulo que se va a importar. Cuando encuentre el módulo, selecciónelo y elija la pestaña **Azure Automation**. Seleccione **Implementar en Azure Automation**.

![Captura de pantalla de la importación de módulos directamente desde la Galería de PowerShell](../media/modules/import-gallery.png)

### <a name="test-your-runbooks"></a>Prueba de los runbooks

Una vez que los módulos Az se han importado en la cuenta de Automation, puede empezar a editar los runbooks y las configuraciones de DSC para usar los nuevos módulos. Una manera de probar la modificación de un runbook para usar los nuevos cmdlets consiste en utilizar el comando `Enable-AzureRmAlias -Scope Process` al principio del runbook. Al incorporar este comando al runbook, el script se puede ejecutar sin cambios.

## <a name="author-modules"></a>Creación de módulos

Se recomienda que tenga en cuenta los aspectos de esta sección al crear un módulo de PowerShell personalizado para usarlo en Azure Automation. Para preparar el módulo para la importación, debe crear al menos un archivo **.dll** de módulo psd1, psm1 o PowerShell con el mismo nombre que la carpeta del módulo. Después, debe comprimir la carpeta del módulo para que Azure Automation pueda importarla como un único archivo. El paquete **.zip** debe tener el mismo nombre que la carpeta de módulo que contiene.

Para más información sobre la creación de un módulo de PowerShell, consulte [Cómo escribir un módulo de script de PowerShell](/powershell/scripting/developer/module/how-to-write-a-powershell-script-module).

### <a name="version-folder"></a>Carpeta de versiones

El control de versiones de módulos en paralelo de PowerShell le permite usar más de una versión de un módulo en PowerShell. Esto puede ser útil si tiene scripts anteriores que se han probado y solo funcionan con una versión determinada de un módulo de PowerShell, pero para otros necesita una versión más reciente del mismo módulo.

A fin de crear módulos de PowerShell para que contengan varias versiones, cree la carpeta del módulo y, después, cree una carpeta en su interior para cada versión del módulo que quiera usar. En el ejemplo siguiente, un módulo denominado *TestModule* proporciona dos versiones: 1.0.0 y 2.0.0.

```dos
TestModule
   1.0.0
   2.0.0
```

Dentro de cada una de las carpetas de versión, copie los archivos **.dll** de módulo de PowerShell o .psm1 y .psd1 que componen un módulo en la carpeta de la versión correspondiente. Comprima la carpeta del módulo para que Azure Automation pueda importarla como un único archivo .zip. Mientras que Automation solo muestra la versión más alta del módulo importado, si el paquete de módulo contiene versiones en paralelo, estarán disponibles para su uso en los runbooks o configuraciones de DSC.  

Aunque Automation admite módulos que contienen versiones en paralelo en el mismo paquete, no admite el uso de varias versiones de un módulo entre importaciones de paquetes de módulos. Por ejemplo, puede importar el **módulo A**, que contiene las versiones 1 y 2 a la cuenta de Automation. Después, actualice el **módulo A** para incluir las versiones 3 y 4; cuando lo importe en la cuenta de Automation, solo las versiones 3 y 4 se pueden usar en cualquier runbook o configuración de DSC. Si necesita que todas las versiones (1, 2, 3 y 4) estén disponibles, se deben incluir en el archivo .zip.

Si va a usar otras versiones del mismo módulo entre runbooks, siempre debe declarar la versión que quiera utilizar en el runbook con el cmdlet `Import-Module` e incluir el parámetro `-RequiredVersion <version>`. Incluso si la versión que quiere usar es la más reciente. Esto se debe a que los trabajos de runbook se pueden ejecutar en el mismo espacio aislado. Si el espacio aislado ya ha cargado de forma explícita un módulo de un número de versión concreto, porque un trabajo anterior de ese espacio aislado lo haya indicado, los trabajos futuros de ese espacio aislado no cargarán automáticamente la versión más reciente de ese módulo. Esto se debe a que alguna versión ya está cargada en el espacio aislado.

Para un recurso de DSC, use el comando siguiente para especificar una versión concreta:

```powershell
Import-DscResource -ModuleName <ModuleName> -ModuleVersion <version>
```

### <a name="help-information"></a>Información de ayuda

Incluya una sinopsis, una descripción y un identificador URI de ayuda para todos los cmdlets del módulo. En PowerShell, puede definir la información de ayuda de los cmdlets mediante el cmdlet `Get-Help`. En el ejemplo siguiente se muestra cómo definir una sinopsis y un identificador URI de ayuda en un archivo de módulo **.psm1**.

  ```powershell
  <#
       .SYNOPSIS
        Gets a Contoso User account
  #>
  function Get-ContosoUser {
  [CmdletBinding](DefaultParameterSetName='UseConnectionObject', `
  HelpUri='https://www.contoso.com/docs/information')]
  [OutputType([String])]
  param(
     [Parameter(ParameterSetName='UserAccount', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [string]
     $UserName,

     [Parameter(ParameterSetName='UserAccount', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [string]
     $Password,

     [Parameter(ParameterSetName='ConnectionObject', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [Hashtable]
     $Connection
  )

  switch ($PSCmdlet.ParameterSetName) {
     "UserAccount" {
        $cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $UserName, $Password
        Connect-Contoso -Credential $cred
     }
     "ConnectionObject" {
        Connect-Contoso -Connection $Connection
    }
  }
  }
  ```

  Proporcionar esta información permite mostrar el texto de ayuda mediante el cmdlet `Get-Help` en la consola de PowerShell. Este texto también se muestra en Azure Portal.

  ![Captura de pantalla de ayuda para los módulos de integración](../media/modules/module-activity-description.png)

### <a name="connection-type"></a>Tipo de conexión

Si el módulo se conecta a un servicio externo, defina un tipo de conexión mediante un [módulo de integración personalizado](#custom-modules). Cada cmdlet del módulo debería aceptar una instancia de ese tipo de conexión (objeto de conexión) como parámetro. Los usuarios asignan parámetros del recurso de conexión a los parámetros correspondientes del cmdlet cada vez que llaman a un cmdlet.

![Uso de una conexión personalizada en Azure Portal](../media/modules/connection-create-new.png)

El ejemplo de runbook siguiente usa un recurso de conexión de Contoso denominado `ContosoConnection` para acceder a los recursos de Contoso y devolver datos desde el servicio externo. En este ejemplo, los campos se asignan a las propiedades `UserName` y `Password` de un objeto `PSCredential` y, a continuación, se pasan al cmdlet.

  ```powershell
  $contosoConnection = Get-AutomationConnection -Name 'ContosoConnection'

  $cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $contosoConnection.UserName, $contosoConnection.Password
  Connect-Contoso -Credential $cred
  }
  ```

Existe otra forma más fácil y eficaz de enfocar este comportamiento y consiste en pasar directamente el objeto de conexión al cmdlet:

  ```powershell
  $contosoConnection = Get-AutomationConnection -Name 'ContosoConnection'

  Connect-Contoso -Connection $contosoConnection
  }
  ```

Para habilitar un comportamiento como este en sus cmdlets, permítales que acepten directamente un objeto de conexión como parámetro en lugar de usar solamente campos de conexión en los parámetros. Normalmente, necesitará un conjunto de parámetros para cada uno de ellos, de forma que los usuarios que no utilizan Automation puedan llamar a los cmdlets sin necesidad de crear una tabla hash que actúe como objeto de conexión. El conjunto de parámetros `UserAccount` se usa para pasar las propiedades de los campos de conexión. `ConnectionObject` permite pasar directamente la conexión.

### <a name="output-type"></a>Tipo de salida

Defina el tipo de salida de todos los cmdlets del módulo. Definir un tipo de salida en un cmdlet permite que IntelliSense le ayude a determinar, en tiempo de diseño, las propiedades de salida del cmdlet durante la creación. Este procedimiento recomendado resulta especialmente útil durante la creación gráfica de un runbook, para el que la información en tiempo de diseño resulta esencial para facilitar la experiencia del usuario con el módulo.

Agregue `[OutputType([<MyOutputType>])]`, donde `MyOutputType` es un tipo válido. Para más información sobre `OutputType`, consulte [About Functions OutputTypeAttribute](/powershell/module/microsoft.powershell.core/about/about_functions_outputtypeattribute) (Acerca de las funciones OutputTypeAttribute). El siguiente código es un ejemplo de cómo agregar `OutputType` a un cmdlet:

  ```powershell
  function Get-ContosoUser {
  [OutputType([String])]
  param(
     [string]
     $Parameter1
  )
  # <script location here>
  }
  ```

  ![Captura de pantalla del tipo de salida de runbooks gráficos](../media/modules/runbook-graphical-module-output-type.png)

  Este comportamiento es similar a la funcionalidad "type ahead" de la salida de los cmdlets del entorno del servicio de integración de PowerShell, sin necesidad de ejecutarla.

  ![Captura de pantalla de POSH IntelliSense](../media/modules/automation-posh-ise-intellisense.png)

### <a name="cmdlet-state"></a>Estado del cmdlet

Cree todos los cmdlets del módulo sin estado. Pueden ejecutarse varios trabajos de runbook simultáneamente en el mismo elemento `AppDomain` y el mismo proceso y espacio aislado. Si no hay ningún estado compartido en esos niveles, los trabajos pueden afectar entre sí. Este comportamiento puede provocar problemas intermitentes y difíciles de diagnosticar. A continuación, se muestra un ejemplo de lo que no hay que hacer:

  ```powershell
  $globalNum = 0
  function Set-GlobalNum {
     param(
         [int] $num
     )

     $globalNum = $num
  }
  function Get-GlobalNumTimesTwo {
     $output = $globalNum * 2

     $output
  }
  ```

### <a name="module-dependency"></a>Dependencia de los módulos

Asegúrese de que el módulo está totalmente contenido en un paquete que se puede copiar mediante XCopy. Los módulos de Automation se distribuyen en los espacios aislados de Automation cuando los runbooks se ejecutan. Los módulos deben funcionar de forma independiente con respecto al host que los ejecuta.

Debe ser capaz de comprimir y mover un paquete de módulos y conseguir que funcione con normalidad cuando se importe en el entorno de PowerShell de otro host. Para que esto ocurra, asegúrese de que el módulo no dependa de ningún archivo que esté fuera de la carpeta del módulo que se comprime cuando el módulo se importa en Automation.

El módulo no debe depender de ninguna configuración de registro única en un host. Un ejemplo es la configuración que se realiza cuando se instala un producto.

### <a name="module-file-paths"></a>Rutas de acceso a los archivos del módulo

Asegúrese de que todos los archivos del módulo tienen rutas de acceso con menos de 140 caracteres. Las rutas de acceso con más de 140 caracteres provocan problemas de importación de runbooks. Automation no puede importar ningún archivo con un tamaño de ruta de acceso superior a 140 caracteres en la sesión de PowerShell con `Import-Module`.

## <a name="import-modules"></a>Importación de módulos

En esta sección se definen varias formas de importar un módulo en la cuenta de Automation.

### <a name="import-modules-in-the-azure-portal"></a>Importación de módulos en Azure Portal

Para importar un módulo en Azure Portal:

1. En el portal, busque y seleccione **Cuentas de Automation**.
1. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.
1. En **Recursos compartidos**, seleccione **Módulos**.
1. Seleccione **Agregar un módulo**.
1. Seleccione el archivo **.zip** que contiene el módulo.
1. Seleccione **Aceptar** para iniciar el proceso de importación.

### <a name="import-modules-by-using-powershell"></a>Importación de módulos mediante PowerShell

Puede usar el cmdlet [New-AzAutomationModule](/powershell/module/az.automation/new-azautomationmodule) para importar un módulo en la cuenta de Automation. El cmdlet toma una dirección URL para un paquete .zip del módulo.

```azurepowershell-interactive
New-AzAutomationModule -Name <ModuleName> -ContentLinkUri <ModuleUri> -ResourceGroupName <ResourceGroupName> -AutomationAccountName <AutomationAccountName>
```

También puede usar el mismo cmdlet para importar un módulo directamente desde la Galería de PowerShell. Asegúrese de captar `ModuleName` y `ModuleVersion` de la [Galería de PowerShell](https://www.powershellgallery.com).

```azurepowershell-interactive
$moduleName = <ModuleName>
$moduleVersion = <ModuleVersion>
New-AzAutomationModule -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -Name $moduleName -ContentLinkUri "https://www.powershellgallery.com/api/v2/package/$moduleName/$moduleVersion"
```

### <a name="import-modules-from-the-powershell-gallery"></a>Importación de módulos desde la Galería de PowerShell

Puede importar módulos de la [Galería de PowerShell](https://www.powershellgallery.com) directamente desde la galería o desde la cuenta de Automation.

Para importar un módulo directamente desde la Galería de PowerShell:

1. Vaya a https://www.powershellgallery.com y busque el módulo que desea importar.
2. En **Opciones de instalación**, en la pestaña **Azure Automation**, seleccione **Implementar en Azure Automation**. Esta acción abre Azure Portal. 
3. En la página Importar, seleccione su cuenta de Automation y seleccione **Aceptar**.

![Captura de pantalla del módulo de importación de la Galería de PowerShell](../media/modules/powershell-gallery.png)

Para importar un módulo de la Galería de PowerShell directamente desde la cuenta de Automation:

1. En el portal, busque y seleccione **Cuentas de Automation**.
1. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.
1. En **Recursos compartidos**, seleccione **Módulos**. 
1. Seleccione **Examinar galería** y busque el módulo en la galería. 
1. Seleccione el módulo que quiere importar y, después, haga clic en **Importar**. 
1. Seleccione **Aceptar** para iniciar el proceso de importación.

![Captura de pantalla de la importación de un módulo de la Galería de PowerShell desde Azure Portal](../media/modules/gallery-azure-portal.png)

## <a name="delete-modules"></a>Eliminación de los módulos

Si tiene problemas con un módulo o necesita revertir a una versión anterior de un módulo, puede eliminarlo de la cuenta de Automation. No se pueden eliminar las versiones originales de los [módulos predeterminados](#default-modules) que se importan al crear una cuenta de Automation. Si el módulo que desea eliminar es una versión más reciente de uno de los [módulos predeterminados](#default-modules), se revertirá a la versión que se instaló con la cuenta de Automation. En caso contrario, se quitará cualquier módulo que se elimine de la cuenta de Automation.

### <a name="delete-modules-in-the-azure-portal"></a>Eliminación de módulos en Azure Portal

Para eliminar un módulo en Azure Portal:

1. En el portal, busque y seleccione **Cuentas de Automation**.
1. En la página **Cuentas de Automation**, seleccione su cuenta de Automation en la lista.
1. En **Recursos compartidos**, seleccione **Módulos**.
1. Seleccione el módulo que quiera quitar.
1. En la página Módulo, seleccione **Eliminar**. Si este módulo es uno de los [módulos predeterminados](#default-modules), se revertirá a la versión que existía cuando se creó la cuenta de Automation.

### <a name="delete-modules-by-using-powershell"></a>Eliminación de módulos mediante PowerShell

Para quitar un módulo a través de PowerShell, ejecute el siguiente comando:

```azurepowershell-interactive
Remove-AzAutomationModule -Name <moduleName> -AutomationAccountName <automationAccountName> -ResourceGroupName <resourceGroupName>
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre el uso de los módulos de Azure PowerShell, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps).

* Para más información sobre la creación de módulos de PowerShell, consulte [Escritura de un módulo de Windows PowerShell](/powershell/scripting/developer/module/writing-a-windows-powershell-module).
