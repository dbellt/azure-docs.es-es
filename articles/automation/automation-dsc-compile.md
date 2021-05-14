---
title: Compilación de configuraciones de DSC en Azure Automation State Configuration
description: En este artículo se indica cómo compilar configuraciones de Desired State Configuration (DSC) para Azure Automation.
services: automation
ms.subservice: dsc
ms.date: 04/06/2020
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4e0a8bc3057bc098855eb7ff65a8feeb83b7c746
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834829"
---
# <a name="compile-dsc-configurations-in-azure-automation-state-configuration"></a>Compilación de configuraciones de DSC en Azure Automation State Configuration

Azure Automation State Configuration permite compilar las configuraciones de Desired State Configuration (DSC) de las siguientes formas:

- Servicio de compilación de Azure State Configuration
  - Método para principiantes, con una interfaz de usuario interactiva
   - Seguimiento sencillo del estado del trabajo

- Windows PowerShell
  - Llamada desde Windows PowerShell en una estación de trabajo local o un servicio de compilación
  - Integración con la canalización de pruebas de desarrollo
  - Disponibilidad de valores de parámetro complejos
  - Trabajo con datos de nodo y datos que no son de nodo a escala
  - Mejora significativa del rendimiento

También puede usar plantillas de Azure Resource Manager con la extensión de Azure Desired State Configuration (DSC) para insertar configuraciones en las máquinas virtuales de Azure. La extensión DSC de Azure usa el marco del agente de máquina virtual de Azure para entregar y aplicar las configuraciones de DSC que se ejecutan en las máquinas virtuales de Azure, así como informar sobre estas. Para obtener información detallada sobre la compilación mediante plantillas de Azure Resource Manager, consulte [Extensión Desired State Configuration con plantillas de Azure Resource Manager](../virtual-machines/extensions/dsc-template.md#details). 

## <a name="compile-a-dsc-configuration-in-azure-state-configuration"></a>Compilar una configuración de DSC en Azure State Configuration

### <a name="portal"></a>Portal

1. En su cuenta de Automation, haga clic en **State Configuration (DSC)** .
1. Haga clic en la pestaña **Configuraciones** y, a continuación, haga clic en el nombre de la configuración que se va a compilar.
1. Haga clic en **Compilar**.
1. Si la configuración no tiene parámetros, se le pedirá que confirme que desea compilarla. Si la configuración tiene parámetros, se abrirá la hoja **Compilar configuración** para que pueda proporcionar valores de parámetros.
1. Se abre la página Trabajo de compilación para que pueda realizar el seguimiento del estado del trabajo de compilación. También puede usar esta página para realizar el seguimiento de las configuraciones de nodo (documentos de configuración MOF) colocadas en el servidor de extracción de State Configuration de Azure Automation.

### <a name="azure-powershell"></a>Azure PowerShell

Puede usar [Start-AzAutomationDscCompilationJob](/powershell/module/az.automation/start-azautomationdsccompilationjob) para empezar a compilar con Windows PowerShell. El código de ejemplo siguiente inicia la compilación de una configuración de DSC llamada **SampleConfig**.

```powershell
Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'
```

`Start-AzAutomationDscCompilationJob` devuelve un objeto de trabajo de compilación que puede usar para realizar el seguimiento del estado del trabajo. Después, puede utilizar este objeto de trabajo de compilación con [Get-AzAutomationDscCompilationJob](/powershell/module/az.automation/get-azautomationdsccompilationjob) para determinar el estado del trabajo de compilación y [Get-AzAutomationDscCompilationJobOutput](/powershell/module/az.automation/get-azautomationdscconfiguration) para ver sus flujos (salida). El ejemplo siguiente inicia la compilación de la configuración SampleConfig, espera hasta que finalice y, a continuación, muestra sus flujos.

```powershell
$CompilationJob = Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'

while($null -eq $CompilationJob.EndTime -and $null -eq $CompilationJob.Exception)
{
    $CompilationJob = $CompilationJob | Get-AzAutomationDscCompilationJob
    Start-Sleep -Seconds 3
}

$CompilationJob | Get-AzAutomationDscCompilationJobOutput –Stream Any
```

### <a name="declare-basic-parameters"></a>Declaración de los parámetros básicos

La declaración de parámetro en configuraciones DSC, incluidos los tipos de parámetro y las propiedades, funciona igual que en los runbooks de Azure Automation. Vea [Inicio de un runbook en Azure Automation](./start-runbooks.md) para obtener más información acerca de los parámetros del runbook.

En el ejemplo siguiente se usan los parámetros `FeatureName` y `IsPresent` para determinar los valores de las propiedades de la configuración del nodo **ParametersExample.sample**, que se han generado durante la compilación.

```powershell
Configuration ParametersExample
{
    param(
        [Parameter(Mandatory=$true)]
        [string] $FeatureName,

        [Parameter(Mandatory=$true)]
        [boolean] $IsPresent
    )

    $EnsureString = 'Present'
    if($IsPresent -eq $false)
    {
        $EnsureString = 'Absent'
    }

    Node 'sample'
    {
        WindowsFeature ($FeatureName + 'Feature')
        {
            Ensure = $EnsureString
            Name   = $FeatureName
        }
    }
}
```

Puede compilar configuraciones DSC que usan parámetros básicos en el portal de Azure Automation State Configuration o Azure PowerShell.

#### <a name="portal"></a>Portal

En el portal, puede especificar valores de parámetro después de hacer clic **Compilar**.

![Parámetros compilación de configuración](./media/automation-dsc-compile/DSC_compiling_1.png)

#### <a name="azure-powershell"></a>Azure PowerShell

PowerShell requiere parámetros de un [hashtable](/powershell/module/microsoft.powershell.core/about/about_hash_tables) donde la clave coincide con el nombre del parámetro y el valor es igual al valor de parámetro.

```powershell
$Parameters = @{
    'FeatureName' = 'Web-Server'
    'IsPresent' = $False
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ParametersExample' -Parameters $Parameters
```

Para obtener información acerca de cómo pasar objetos `PSCredential` como parámetros, consulte [Recursos de credenciales](#credential-assets).

### <a name="compile-configurations-containing-composite-resources-in-azure-automation"></a>Compilación de configuraciones que contienen recursos compuestos en Azure Automation

La característica **Recursos compuestos** le permite utilizar las configuraciones de DSC como recursos anidados dentro de una configuración. Esta característica permite la aplicación de varias configuraciones a un solo recurso. Consulte [Recursos compuestos: uso de una configuración de DSC como un recurso](/powershell/scripting/dsc/resources/authoringresourcecomposite) para más información acerca de los recursos compuestos.

> [!NOTE]
> Para que las configuraciones que contienen recursos compuestos se compilen correctamente, antes debe importar en Azure Automation los recursos de DSC en los que se basan los compuestos. La incorporación de un recurso compuesto de DSC no es diferente de la incorporación de cualquier módulo de PowerShell a Azure Automation. El proceso se documenta en [Administración de módulos en Azure Automation](./shared-resources/modules.md).

### <a name="manage-configurationdata-when-compiling-configurations-in-azure-automation"></a>Administración de ConfigurationData al compilar configuraciones en Azure Automation

`ConfigurationData` es un parámetro de DSC integrado que le permite separar la configuración estructural de cualquier configuración específica del entorno al usar DSC de PowerShell. Para obtener más información, consulte [Diferenciación de "Qué" y "Dónde" en DSC de PowerShell](https://devblogs.microsoft.com/powershell/separating-what-from-where-in-powershell-dsc/).

> [!NOTE]
> Al compilar en Azure Automation State Configuration, puede usar `ConfigurationData` en Azure PowerShell, pero no en Azure Portal.

En el siguiente ejemplo de configuración de DSC se usa `ConfigurationData` mediante las palabras clave `$ConfigurationData` y `$AllNodes`. En este ejemplo, también necesitará el [módulo xWebAdministration](https://www.powershellgallery.com/packages/xWebAdministration/).

```powershell
Configuration ConfigurationDataSample
{
    Import-DscResource -ModuleName xWebAdministration -Name MSFT_xWebsite

    Write-Verbose $ConfigurationData.NonNodeData.SomeMessage

    Node $AllNodes.Where{$_.Role -eq 'WebServer'}.NodeName
    {
        xWebsite Site
        {
            Name         = $Node.SiteName
            PhysicalPath = $Node.SiteContents
            Ensure       = 'Present'
        }
    }
}
```

Puede compilar la configuración de DSC anterior con Windows PowerShell. El siguiente script agrega dos configuraciones de nodo al servicio de extracción de State Configuration de Azure Automation: **ConfigurationDataSample.MyVM1** y **ConfigurationDataSample.MyVM3**.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = 'MyVM1'
            Role = 'WebServer'
        },
        @{
            NodeName = 'MyVM2'
            Role = 'SQLServer'
        },
        @{
            NodeName = 'MyVM3'
            Role = 'WebServer'
        }
    )

    NonNodeData = @{
        SomeMessage = 'I love Azure Automation State Configuration and DSC!'
    }
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ConfigurationDataSample' -ConfigurationData $ConfigData
```

### <a name="work-with-assets-in-azure-automation-during-compilation"></a>Trabajo con recursos en Azure Automation durante la compilación

Las referencias de recursos son las mismas en State Configuration de Azure Automation y en los runbooks. Para obtener más información, vea lo siguiente:

- [Certificados](./shared-resources/certificates.md)
- [Conexiones](automation-connections.md)
- [Credenciales](./shared-resources/credentials.md)
- [Variables](./shared-resources/variables.md)

#### <a name="credential-assets"></a>Recursos de credenciales

Las configuraciones de DSC en Azure Automation pueden hacer referencia a los activos de credenciales de Automation mediante el uso del cmdlet `Get-AutomationPSCredential`. Si una configuración tiene un parámetro que especifica un objeto `PSCredential`, use `Get-AutomationPSCredential` pasando el nombre de cadena de un recurso de credencial de Azure Automation al cmdlet para recuperar la credencial. Luego use ese objeto para el parámetro que requiere el objeto `PSCredential`. En segundo plano, se recuperará el recurso de credenciales de Azure Automation con ese nombre y se pasará a la configuración. El ejemplo siguiente muestra este escenario en acción.

Mantener las credenciales seguras en configuraciones de nodo (documentos de configuración MOF) exige el cifrado de las credenciales en el archivo MOF de configuración de nodo. Actualmente, debe proporcionar a DSC de PowerShell permiso para mostrar en la salida las credenciales en texto sin formato durante la generación del archivo MOF de configuración de nodo. DSC de PowerShell no sabe que Azure Automation cifra todo el archivo MOF después de su generación mediante un trabajo de compilación.

Puede indicar a DSC de PowerShell que es correcto que las credenciales tengan salida como texto sin formato en los MOF de configuración de nodo generado mediante Datos de configuración. Debe pasar `PSDscAllowPlainTextPassword = $true` a través de `ConfigurationData` para el nombre de cada bloque de nodo que aparezca en la configuración de DSC y use las credenciales.

En el ejemplo siguiente se muestra una configuración de DSC que usa un recurso de la credencial de Automation.

```powershell
Configuration CredentialSample
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    $Cred = Get-AutomationPSCredential 'SomeCredentialAsset'

    Node $AllNodes.NodeName
    {
        File ExampleFile
        {
            SourcePath      = '\\Server\share\path\file.ext'
            DestinationPath = 'C:\destinationPath'
            Credential      = $Cred
        }
    }
}
```

Puede compilar la configuración de DSC anterior con PowerShell. El siguiente código de PowerShell agrega las configuraciones de dos nodos al servidor de extracción de State Configuration de Azure Automation: **CredentialSample.MyVM1** y **CredentialSample.MyVM2**.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = '*'
            PSDscAllowPlainTextPassword = $True
        },
        @{
            NodeName = 'MyVM1'
        },
        @{
            NodeName = 'MyVM2'
        }
    )
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'CredentialSample' -ConfigurationData $ConfigData
```

> [!NOTE]
> Una vez completada la compilación, puede recibir el mensaje de error `The 'Microsoft.PowerShell.Management' module was not imported because the 'Microsoft.PowerShell.Management' snap-in was already imported.`. Puede omitir este mensaje con seguridad.

## <a name="compile-your-dsc-configuration-in-windows-powershell"></a>Compilar la configuración de DSC en Windows PowerShell

El proceso de compilación de las configuraciones de DSC en Windows PowerShell se incluye en la documentación de DSC de PowerShell, en [Escritura, compilación y aplicación de una configuración](/powershell/scripting/dsc/configurations/write-compile-apply-configuration#compile-the-configuration).
Puede ejecutar este proceso desde una estación de trabajo de desarrollador o dentro de un servicio de compilación como [Azure DevOps](https://dev.azure.com). A continuación, puede importar los archivos MOF generados al compilar la configuración en el servicio Azure State Configuration.

La compilación en Windows PowerShell también ofrece la opción de firmar el contenido de la configuración. El agente de DSC comprueba una configuración de nodo firmada localmente en un nodo administrado. La comprobación garantiza que la configuración que se aplica al nodo proviene de un origen autorizado.

También puede importar las configuraciones de nodo (archivos MOF) que se hayan compilado fuera de Azure. La importación incluye la compilación desde una estación de trabajo de desarrollador o en un servicio como [Azure DevOps](https://dev.azure.com). Este enfoque tiene múltiples ventajas, como el rendimiento y la confiabilidad.

> [!NOTE]
> Un archivo de configuración de nodo no debe ser superior a 1 MB para que Azure Automation pueda importarlo.

Para más información sobre la firma de configuraciones de nodo, consulte [Mejoras en WMF 5.1: firma de la configuración y del módulo](/powershell/scripting/wmf/whats-new/dsc-improvements#dsc-module-and-configuration-signing-validations).

### <a name="import-a-node-configuration-in-the-azure-portal"></a>Importación de una configuración de nodo en Azure Portal

1. En la cuenta de Automation, haga clic en **State Configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), haga clic en la pestaña **Configuraciones** y, después, haga clic en **Agregar**.
1. En la página Importar, haga clic en el icono de carpeta situado junto al cuadro de texto **Archivo de configuración de nodo** para buscar un archivo MOF de configuración de nodo en el equipo local.

   ![Buscar archivo local](./media/automation-dsc-compile/import-browse.png)

1. Escriba un nombre en el campo **Nombre de la configuración**. Este nombre debe coincidir con el de la configuración desde la que se compiló la configuración del nodo.
1. Haga clic en **OK**.

### <a name="import-a-node-configuration-with-azure-powershell"></a>Importación de una configuración de nodo con Azure PowerShell

Puede usar el cmdlet [Import-AzAutomationDscNodeConfiguration](/powershell/module/az.automation/import-azautomationdscnodeconfiguration) para importar una configuración de nodo en la cuenta de Automation.

```powershell
Import-AzAutomationDscNodeConfiguration -AutomationAccountName 'MyAutomationAccount' -ResourceGroupName 'MyResourceGroup' -ConfigurationName 'MyNodeConfiguration' -Path 'C:\MyConfigurations\TestVM1.mof'
```

## <a name="next-steps"></a>Pasos siguientes

- Para dar los primeros pasos, consulte [Introducción a Azure Automation State Configuration](automation-dsc-getting-started.md).
- Para aprender a compilar configuraciones de DSC para poder asignarlas a los nodos de destino, consulte [Compilación de configuraciones de DSC en Azure Automation State Configuration](automation-dsc-compile.md).
- Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
- Para obtener información de precios, consulte [Precios de State Configuration de Azure Automation](https://azure.microsoft.com/pricing/details/automation/).
- Para ver un ejemplo del uso de State Configuration en una canalización de implementación continua, consulte [Configuración de la implementación continua con Chocolatey](automation-dsc-cd-chocolatey.md).
