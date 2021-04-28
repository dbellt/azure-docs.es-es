---
title: Habilitar Azure Automation State Configuration
description: En este artículo se explica cómo configurar las máquinas para realizar la administración con Azure Automation State Configuration.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.topic: conceptual
ms.date: 12/10/2019
ms.custom: devx-track-azurepowershell
manager: carmonm
ms.openlocfilehash: d338c5f34d49663345582198ff53ba50a2919d7e
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107829429"
---
# <a name="enable-azure-automation-state-configuration"></a>Habilitar Azure Automation State Configuration

En este tema se describe cómo puede configurar las máquinas para realizar la administración con Azure Automation State Configuration. Para obtener más información sobre este servicio, vea [Información general de Azure Automation State Configuration](automation-dsc-overview.md).

## <a name="enable-azure-vms"></a>Habilitar máquinas virtuales de Azure

Con Azure Automation State Configuration, se pueden habilitar fácilmente máquinas virtuales de Azure para la administración de configuraciones usando Azure Portal, plantillas de Azure Resource Manager o PowerShell. Internamente, y sin que un administrador tenga que acceder de forma remota a la máquina virtual, la extensión Desired State Configuration de máquina virtual de Azure registra la máquina virtual con State Configuration de Azure Automation. Como la extensión de Azure se ejecuta de forma asincrónica, se proporcionan pasos para realizar un seguimiento de su progreso en [Comprobar el estado de la instalación de la máquina virtual](#check-status-of-vm-setup).

> [!NOTE]
>En la implementación de DSC en un nodo de Linux se usa la carpeta **/tmp**. Ciertos módulos, como `nxautomation`, se descargan temporalmente para su comprobación antes de instalarlos en las ubicaciones adecuadas correspondientes. Para garantizar que los módulos se instalan correctamente, el agente de Log Analytics para Linux necesita permiso de lectura y escritura para la carpeta **/tmp**.<br><br>
>El agente de Log Analytics para Linux se ejecuta como el usuario `omsagent`. Para conceder permiso de escritura al usuario `omsagent`, ejecute el comando `setfacl -m u:omsagent:rwx /tmp`.

### <a name="enable-a-vm-using-azure-portal"></a>Habilitar una máquina virtual mediante Azure Portal

Para habilitar una máquina virtual de Azure a State Configuration a través de [Azure Portal](https://portal.azure.com/):

1. Vaya a la cuenta de Azure Automation en la que vaya a habilitar las máquinas virtuales. 

2. En la página de State Configuration, seleccione la pestaña **Nodos** y haga clic en **Agregar**.

3. Elija la máquina virtual que quiera habilitar.

4. Si la máquina no tiene instalada la extensión de estado de PowerShell deseada y el estado de energía es "en ejecución", haga clic en **Conectar**.

5. En **Registro**, escriba los [valores del Administrador de configuración local de DSC de PowerShell](/powershell/scripting/dsc/managing-nodes/metaConfig) necesarios para su caso de uso. Opcionalmente, se puede especificar una configuración de nodo para asignarla a la máquina virtual.

![Habilitación de máquina virtual](./media/automation-dsc-onboarding/DSC_Onboarding_6.png)

### <a name="enable-a-vm-using-azure-resource-manager-templates"></a>Habilitación de una máquina virtual con plantillas de Azure Resource Manager

Puede instalar y habilitar una máquina virtual para State Configuration con plantillas de Azure Resource Manager. Vea [Server managed by Desired State Configuration service](https://azure.microsoft.com/resources/templates/101-automation-configuration/) (Servidor administrado por el servicio Desired State Configuration) para obtener una plantilla de ejemplo que habilita una máquina virtual existente de State Configuration. Si administra un conjunto de escalado de máquinas virtuales, consulte la plantilla de ejemplo de [Configuración de un conjunto de escalado de máquinas virtuales administrado por Azure Automation](https://azure.microsoft.com/resources/templates/201-vmss-automation-dsc/).

### <a name="enable-machines-using-powershell"></a>Habilitación de máquinas con PowerShell

Puede usar el cmdlet [Register-AzAutomationDscNode](/powershell/module/az.automation/register-azautomationdscnode) de PowerShell para habilitar las máquinas virtuales de State Configuration. 

> [!NOTE]
>Actualmente, el cmdlet `Register-AzAutomationDscNode` se implementa solo para equipos que ejecutan Windows, ya que únicamente desencadena la extensión de Windows.

### <a name="register-vms-across-azure-subscriptions"></a>Registro de máquinas virtuales en suscripciones de Azure

La mejor manera de registrar máquinas virtuales desde otras suscripciones de Azure consiste en usar la extensión DSC en una plantilla de implementación de Azure Resource Manager. Los ejemplos se proporcionan en [Extensión Desired State Configuration con plantillas de Azure Resource Manager](../virtual-machines/extensions/dsc-template.md).

Para buscar la clave y la dirección URL de registro que se van a usar como parámetros en esta plantilla, vea [Habilitar máquinas de manera segura con un protocolo de registro](#enable-machines-securely-using-registration).

## <a name="enable-physicalvirtual-windows-machines"></a>Habilitar máquinas físicas o virtuales con Windows

Los servidores de Windows que se ejecutan en el entorno local o en otros entornos de nube (incluidas las instancias de AWS EC2) se pueden habilitar en Azure Automation State Configuration. Los servidores deben tener [acceso de salida a Azure](automation-dsc-overview.md#network-planning).

1. Asegúrese de que tiene instalada la versión más reciente de [WMF 5](https://aka.ms/wmf5latest) en las máquinas que vaya a habilitar State Configuration. Además, WMF 5 debe estar instalado en el equipo que esté usando para habilitar las máquinas.
1. Siga las instrucciones de [Generación de metaconfiguraciones de DSC](#generate-dsc-metaconfigurations) para crear una carpeta que contenga las metaconfiguraciones de DSC necesarias. 
1. Use el siguiente cmdlet para aplicar las metaconfiguraciones de DSC de PowerShell de forma remota a las máquinas que se van a habilitar. 

   ```powershell
   Set-DscLocalConfigurationManager -Path C:\Users\joe\Desktop\DscMetaConfigs -ComputerName MyServer1, MyServer2
   ```

1. Si no puede aplicar las metaconfiguraciones de DSC de PowerShell de forma remota, copie la carpeta de **metaconfiguraciones** del paso 2 en las máquinas que vaya a habilitar. A continuación, agregue código para llamar a [Set-DscLocalConfigurationManager](/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager) localmente en las máquinas.
1. Mediante Azure Portal o con cmdlets, compruebe que las máquinas que se van a habilitar aparecen ahora como nodos de State Configuration registrados en la cuenta de Azure Automation.

## <a name="enable-physicalvirtual-linux-machines"></a>Habilitación de máquinas físicas o virtuales con Linux

Puede habilitar servidores Linux que se ejecutan de forma local o en otros entornos de nube a State Configuration. Los servidores deben tener [acceso de salida a Azure](automation-dsc-overview.md#network-planning).

1. Asegúrese de que tiene instalada la versión más reciente d [PowerShell Desired State Configuration para Linux](https://github.com/Microsoft/PowerShell-DSC-for-Linux) en las máquinas que quiera habilitar State Configuration.
2. Si los [Valores predeterminados del Configuration Manager local de DSC de PowerShell](/powershell/scripting/dsc/managing-nodes/metaConfig4) coinciden con su caso de uso, y quiere habilitar máquinas de modo que extraigan información de State Configuration y también envíen informes allí:

   - En cada máquina Linux que se va a habilitar, use `Register.py` para habilitar la máquina con los valores predeterminados locales de PowerShell DSC Configuration Manager.

     `/opt/microsoft/dsc/Scripts/Register.py <Automation account registration key> <Automation account registration URL>`

   - Para buscar la clave y la dirección URL de registro de su cuenta de Automation, vea [Habilitar máquinas de manera segura con un protocolo de registro](#enable-machines-securely-using-registration).

3. Si los valores predeterminados del Configuration Manager local de DSC de PowerShell no coinciden con el caso de uso, o si quiere habilitar máquinas para que solo notifiquen a Azure Automation State Configuration, siga los pasos 4-7. De lo contrario, vaya directamente al paso 7.

4. Siga las instrucciones de la sección [Generación de metaconfiguraciones de DSC](#generate-dsc-metaconfigurations) para generar una carpeta que contenga las metaconfiguraciones de DSC necesarias.

5. Asegúrese de que tiene instalada la versión más reciente de [WMF 5](https://aka.ms/wmf5latest) en el equipo que se usa para habilitar las máquinas para State Configuration.

6. Agregue el siguiente código para aplicar las metaconfiguraciones de DSC de PowerShell de forma remota a las máquinas que quiera habilitar.

    ```powershell
    $SecurePass = ConvertTo-SecureString -String '<root password>' -AsPlainText -Force
    $Cred = New-Object System.Management.Automation.PSCredential 'root', $SecurePass
    $Opt = New-CimSessionOption -UseSsl -SkipCACheck -SkipCNCheck -SkipRevocationCheck

    # need a CimSession for each Linux machine to onboard
    $Session = New-CimSession -Credential $Cred -ComputerName <your Linux machine> -Port 5986 -Authentication basic -SessionOption $Opt

    Set-DscLocalConfigurationManager -CimSession $Session -Path C:\Users\joe\Desktop\DscMetaConfigs
    ```

7. Si no puede aplicar las metaconfiguraciones de DSC de PowerShell de forma remota, copie la metaconfiguración correspondiente a las máquinas remotas desde la carpeta en el paso 4 en las máquinas Linux.

8. Agregue código para llamar a `Set-DscLocalConfigurationManager.py` localmente en cada máquina Linux que vaya a habilitar State Configuration.

   `/opt/microsoft/dsc/Scripts/SetDscLocalConfigurationManager.py -configurationmof <path to metaconfiguration file>`

9. Usando Azure Portal o cmdlets, compruebe que las máquinas que va a habilitar ahora aparecen como nodos de DSC registrados en su cuenta de Azure Automation.

## <a name="generate-dsc-metaconfigurations"></a>Generación de metaconfiguraciones de DSC

Para habilitar cualquier máquina a State Configuration, puede generar una [metaconfiguración de DSC](/powershell/scripting/dsc/managing-nodes/metaConfig). Esta configuración indica al agente de DSC que extraiga y/o informe a State Configuration de Azure Automation. Una metaconfiguración de State Configuration de Azure Automation se puede generar usando una configuración de DSC de PowerShell o con cmdlets de PowerShell de Azure Automation.

> [!NOTE]
> Las metaconfiguraciones de DSC contienen los secretos necesarios para habilitar una máquina en una cuenta de Automation para su administración. Asegúrese de proteger adecuadamente cualquier metaconfiguración de DSC que cree o elimínelos tras su uso.

La compatibilidad de proxy de las metaconfiguraciones se controla a través del [Configuration Manager local](/powershell/scripting/dsc/managing-nodes/metaconfig), que es el motor de DSC de Windows PowerShell. LCM se ejecuta en todos los nodos de destino, y es el responsable de llamar a los recursos de configuración que se incluyen en un script de metaconfiguración de DSC. Para incluir compatibilidad de proxy en una metaconfiguración, incluya definiciones de `ProxyURL` y propiedades `ProxyCredential` según sea necesario en los bloques `ConfigurationRepositoryWeb`,`ResourceRepositoryWeb` y`ReportServerWeb`. Un ejemplo de la configuración de la dirección URL es `ProxyURL = "http://172.16.3.6:3128";`. La propiedad `ProxyCredential` se establece en un objeto `PSCredential`, tal y como se describe en [Administración de credenciales en Azure Automation](shared-resources/credentials.md). 

### <a name="generate-dsc-metaconfigurations-using-a-dsc-configuration"></a>Generación de metaconfiguraciones de DSC con una configuración de DSC

1. Abra VSCode (o el editor que prefiera) como administrador en una máquina del entorno local. La máquina debe tener instalada la versión más reciente de [WMF 5](https://aka.ms/wmf5latest) .
1. Copie el siguiente script localmente. Este script contiene una configuración de DSC de PowerShell para crear metaconfiguraciones y un comando para iniciar la creación de la metaconfiguración.

    > [!NOTE]
    > Los nombres de configuración del nodo de State Configuration distinguen mayúsculas de minúsculas en Azure Portal. Si estas no coinciden, el nodo no se mostrará en la pestaña **Nodos**.

   ```powershell
   # The DSC configuration that will generate metaconfigurations
   [DscLocalConfigurationManager()]
   Configuration DscMetaConfigs
   {
        param
        (
            [Parameter(Mandatory=$True)]
            [String]$RegistrationUrl,

            [Parameter(Mandatory=$True)]
            [String]$RegistrationKey,

            [Parameter(Mandatory=$True)]
            [String[]]$ComputerName,

            [Int]$RefreshFrequencyMins = 30,

            [Int]$ConfigurationModeFrequencyMins = 15,

            [String]$ConfigurationMode = 'ApplyAndMonitor',

            [String]$NodeConfigurationName,

            [Boolean]$RebootNodeIfNeeded= $False,

            [String]$ActionAfterReboot = 'ContinueConfiguration',

            [Boolean]$AllowModuleOverwrite = $False,

            [Boolean]$ReportOnly
        )

        if(!$NodeConfigurationName -or $NodeConfigurationName -eq '')
        {
            $ConfigurationNames = $null
        }
        else
        {
            $ConfigurationNames = @($NodeConfigurationName)
        }

        if($ReportOnly)
        {
            $RefreshMode = 'PUSH'
        }
        else
        {
            $RefreshMode = 'PULL'
        }

        Node $ComputerName
        {
            Settings
            {
                RefreshFrequencyMins           = $RefreshFrequencyMins
                RefreshMode                    = $RefreshMode
                ConfigurationMode              = $ConfigurationMode
                AllowModuleOverwrite           = $AllowModuleOverwrite
                RebootNodeIfNeeded             = $RebootNodeIfNeeded
                ActionAfterReboot              = $ActionAfterReboot
                ConfigurationModeFrequencyMins = $ConfigurationModeFrequencyMins
            }

            if(!$ReportOnly)
            {
            ConfigurationRepositoryWeb AzureAutomationStateConfiguration
                {
                    ServerUrl          = $RegistrationUrl
                    RegistrationKey    = $RegistrationKey
                    ConfigurationNames = $ConfigurationNames
                }

                ResourceRepositoryWeb AzureAutomationStateConfiguration
                {
                    ServerUrl       = $RegistrationUrl
                    RegistrationKey = $RegistrationKey
                }
            }

            ReportServerWeb AzureAutomationStateConfiguration
            {
                ServerUrl       = $RegistrationUrl
                RegistrationKey = $RegistrationKey
            }
        }
   }

    # Create the metaconfigurations
    # NOTE: DSC Node Configuration names are case sensitive in the portal.
    # TODO: edit the below as needed for your use case
   $Params = @{
        RegistrationUrl = '<fill me in>';
        RegistrationKey = '<fill me in>';
        ComputerName = @('<some VM to onboard>', '<some other VM to onboard>');
        NodeConfigurationName = 'SimpleConfig.webserver';
        RefreshFrequencyMins = 30;
        ConfigurationModeFrequencyMins = 15;
        RebootNodeIfNeeded = $False;
        AllowModuleOverwrite = $False;
        ConfigurationMode = 'ApplyAndMonitor';
        ActionAfterReboot = 'ContinueConfiguration';
        ReportOnly = $False;  # Set to $True to have machines only report to AA DSC but not pull from it
   }

   # Use PowerShell splatting to pass parameters to the DSC configuration being invoked
   # For more info about splatting, run: Get-Help -Name about_Splatting
   DscMetaConfigs @Params
   ```

1. Rellene la clave del Registro y la dirección URL para su cuenta de Automation, así como los nombres de las máquinas que quiere habilitar. Todos los demás parámetros son opcionales. Para buscar la clave y la dirección URL de registro de su cuenta de Automation, vea [Habilitar máquinas de manera segura con un protocolo de registro](#enable-machines-securely-using-registration).

1. Si quiere que las máquinas notifiquen la información de estado de DSC a State Configuration de Azure Automation, pero que no extraigan la configuración o módulos de PowerShell, establezca el parámetro `ReportOnly` en verdadero.

1. Si el parámetro `ReportOnly` no se establece, las máquinas notificarán la información de estado de DSC a State Configuration de Azure Automation y extraerán la configuración o módulos de PowerShell. Establezca los parámetros según corresponda en los bloques `ConfigurationRepositoryWeb`, `ResourceRepositoryWeb` y `ReportServerWeb`.

1. Ejecute el script. Ahora, debería tener una carpeta directorio llamada **DscMetaConfigs** en funcionamiento en el directorio de trabajo que contenga las metaconfiguraciones de DSC de PowerShell de las máquinas que quiera habilitar (como administrador).

    ```powershell
    Set-DscLocalConfigurationManager -Path ./DscMetaConfigs
    ```

### <a name="generate-dsc-metaconfigurations-using-azure-automation-cmdlets"></a>Generación de metaconfiguraciones de DSC con cmdlets de Azure Automation

Si los valores predeterminados de LCM de DSC de PowerShell coinciden con su caso de uso, y quiere habilitar máquinas de modo que extraigan de Azure Automation State Configuration y también notifiquen allí, puede usar cmdlets de Azure Automation para generar las metaconfiguraciones de DSC necesarias de forma más sencilla.

1. Abra la consola de PowerShell o VSCode como administrador en una máquina de su entorno local.
2. Conéctese a Azure Resource Manager mediante [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount).
3. Descargue las metaconfiguraciones de DSC de PowerShell de las máquinas que quiera habilitar desde la cuenta de Automation en la que esté configurando los nodos.

   ```powershell
   # Define the parameters for Get-AzAutomationDscOnboardingMetaconfig using PowerShell Splatting
   $Params = @{
       ResourceGroupName = 'ContosoResources'; # The name of the Resource Group that contains your Azure Automation account
       AutomationAccountName = 'ContosoAutomation'; # The name of the Azure Automation account where you want a node on-boarded to
       ComputerName = @('web01', 'web02', 'sql01'); # The names of the computers that the metaconfiguration will be generated for
       OutputFolder = "$env:UserProfile\Desktop\";
   }
   # Use PowerShell splatting to pass parameters to the Azure Automation cmdlet being invoked
   # For more info about splatting, run: Get-Help -Name about_Splatting
   Get-AzAutomationDscOnboardingMetaconfig @Params
   ```

1. Ahora, debería tener la carpeta **DscMetaConfigs** que contiene las metaconfiguraciones de DSC de PowerShell de las máquinas que quiera habilitar (como administrador).

    ```powershell
    Set-DscLocalConfigurationManager -Path $env:UserProfile\Desktop\DscMetaConfigs
    ```

## <a name="enable-machines-securely-using-registration"></a>Habilitar máquinas de manera segura con un protocolo de registro

Puede habilitar máquinas de forma segura para una cuenta de Azure Automation a través del Protocolo de registro WMF 5 DSC. Este protocolo permite que un nodo de DSC se autentique en un servidor de informes o de extracción de DSC de PowerShell, incluido State Configuration de Azure Automation. El nodo se registra en el servidor en una dirección URL de registro y se autentica usando una clave de registro. Durante el registro, el nodo de DSC y el servidor de informes/extracción de DSC negocian un certificado único que este nodo usará para autenticarse en el servidor después del registro. Este proceso impide que los nodos habilitados se suplanten entre sí, por ejemplo, en caso de que un nodo se vea comprometido y se comporte de forma malintencionada. Después del registro, la clave de registro no se vuelve a usar en operaciones de autenticación y se elimina del nodo.

Puede obtener la información necesaria para el protocolo de registro de State Configuration en **Claves** dentro de **Configuración de la cuenta** en Azure Portal. 

![Claves y dirección URL de Azure Automation](./media/automation-dsc-onboarding/DSC_Onboarding_4.png)

- La dirección URL de registro es el campo de dirección URL en la página Claves.
- La clave de registro es el valor del campo **Clave de acceso primaria** o del campo **Clave de acceso secundaria** de la página Claves. Se puede usar cualquiera de las dos.

Para una mayor seguridad, las claves de acceso primaria y secundaria de una cuenta de Automation se pueden volver a generar en cualquier momento en la página Claves. Volver a generar las claves evita que se produzcan futuros registros de nodo en los que se usen las claves anteriores.

## <a name="re-register-a-node"></a>Nuevo registro de un nodo

Después de registrar un equipo como un nodo de DSC en State Configuration de Azure Automation, existen diversos motivos por los que puede que sea necesario volver a registrar ese nodo en el futuro.

- **Renovación de certificados.** En las versiones de Windows Server anteriores a Windows Server 2019, cada nodo negocia automáticamente un certificado único para la autenticación que expira después de un año. Si un certificado expira sin renovarse, el nodo no puede comunicarse con Azure Automation y se marca como `Unresponsive`. En la actualidad, el protocolo de registro de DSC de PowerShell no puede renovar automáticamente los certificados que vayan a expirar, por lo que tendrá que volver a registrar los nodos transcurrido un año. Antes de volver a registrarlos, asegúrese de que cada nodo ejecuta WMF 5 RTM. 

    Si dicha actualización se realiza en un plazo de 90 días o menos desde la fecha de expiración del certificado o en cualquier momento después de dicha fecha, se generará y usará un certificado nuevo. En Windows Server 2019 y versiones posteriores se incluye una solución a este problema.

- **Cambios en los valores de LCM de DSC.** Puede que haya que cambiar los [valores de LCM de DSC de PowerShell](/powershell/scripting/dsc/managing-nodes/metaConfig4) establecidos durante el registro inicial del nodo, por ejemplo, `ConfigurationMode`. Actualmente, estos valores de agente DSC solo se pueden cambiar realizando un nuevo registro. La única excepción es el valor Configuración del nodo que el nodo tiene asignado. Esto se puede cambiar directamente en DSC de Azure Automation.

Un nodo se puede volver a registrar del mismo modo en que se registró inicialmente, con cualquiera de los métodos descritos en este documento. No es necesario anular el registro de un nodo desde Azure Automation State Configuration antes de volver a registrarlo.

## <a name="check-status-of-vm-setup"></a>Comprobación del estado de la instalación de máquina virtual

State Configuration permite habilitar fácilmente máquinas virtuales de Azure con Windows para la administración de configuraciones. Internamente, la extensión Configuración de estado deseado de la máquina virtual de Azure se usa para registrar la máquina virtual con Azure Automation State Configuration. Puesto que la extensión Desired State Configuration de máquina virtual de Azure se ejecuta de forma asincrónica, puede ser importante realizar el seguimiento de su progreso y solucionar los problemas de ejecución.

> [!NOTE]
> Cualquier método para habilitar máquinas virtuales Windows de Azure para State Configuration que usa la extensión de Desired State Configuration de la máquina virtual de Azure puede tardar hasta una hora en Azure Automation para mostrar las máquinas virtuales como registradas. Este retraso se debe a que la extensión Desired State Configuration de máquina virtual de Azure debe instalar WMF 5 en la máquina virtual como parte necesaria para habilitar las máquinas virtuales para State Configuration.

Para ver el estado de la extensión de Desired State Configuration de la máquina virtual de Azure:

1. En Azure Portal, vaya a la máquina virtual que quiera habilitar.
2. Haga clic en **Extensiones** en **Configuración**. 
3. Ahora, seleccione **DSC** o **DSCForLinux**, según el sistema operativo. 
4. Para obtener más detalles, puede hacer clic en **Ver estado detallado**.

## <a name="next-steps"></a>Pasos siguientes

- Para dar los primeros pasos, consulte [Introducción a Azure Automation State Configuration](automation-dsc-getting-started.md).
- Para aprender a compilar configuraciones de DSC para poder asignarlas a los nodos de destino, consulte [Compilación de configuraciones de DSC en Azure Automation State Configuration](automation-dsc-compile.md).
- Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
- Para obtener información de precios, consulte [Precios de State Configuration de Azure Automation](https://azure.microsoft.com/pricing/details/automation/).
- Para ver un ejemplo del uso de Azure Automation State Configuration en una canalización de implementación continua, consulte [Configuración de la implementación continua con Chocolatey](automation-dsc-cd-chocolatey.md).
- Para obtener información sobre la solución de problemas, vea [Solucionar problemas de Azure Automation State Configuration](./troubleshoot/desired-state-configuration.md).
