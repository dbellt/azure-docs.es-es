---
title: Configurar la implementación continua de Azure Automation con Chocolatey
description: En este artículo se indica cómo configurar la implementación continua con State Configuration y el administrador de paquetes de Chocolatey.
services: automation
ms.subservice: dsc
ms.date: 08/08/2018
ms.topic: conceptual
ms.custom: references_regions, devx-track-azurepowershell
ms.openlocfilehash: 717561614a3e42995bbce6746839fd9b7cbca37e
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834865"
---
# <a name="set-up-continuous-deployment-with-chocolatey"></a>Configuración de la implementación continua con Chocolatey

En el mundo de DevOps, existen muchas herramientas que ayudan en diversos puntos de la canalización de integración continua. Azure Automation [State Configuration](automation-dsc-overview.md) es una opción nueva, y bienvenida, que puede ser de utilidad para los equipos de DevOps. 

Azure Automation es un servicio administrado de Microsoft Azure que le permite automatizar diversas tareas con runbooks, nodos y recursos compartidos, como credenciales, programaciones y variables globales. Azure Automation State Configuration amplía esta funcionalidad de automatización para incluir las herramientas Desired State Configuration (DSC) de PowerShell. Aquí hay una [introducción](automation-dsc-overview.md) muy buena.

En este artículo, se muestra cómo configurar la implementación continua (CD) para un equipo Windows. Puede ampliar fácilmente la técnica para incluir tantos equipos Windows como sean necesarios en el rol, por ejemplo, un sitio web y, a partir de ahí, a roles adicionales.

![Implementación continua para máquinas virtuales IaaS](./media/automation-dsc-cd-chocolatey/cdforiaasvm.png)

## <a name="at-a-high-level"></a>A nivel general

Aunque todo esto pueda parecer bastante complicado, afortunadamente puede dividirse en dos procesos principales:

- Escribir código y probarlo, y después crear y publicar paquetes de instalación para las versiones principales y secundarias del sistema;
- Crear y administrar las máquinas virtuales que instalen y ejecuten el código en los paquetes.  

Una vez que estos dos procesos principales se hayan puesto en práctica, es fácil actualizar automáticamente el paquete en las máquinas virtuales a medida que se crean e implementan las nuevas versiones.

## <a name="component-overview"></a>Información general sobre los componentes

Los administradores de paquetes como [apt-get](https://en.wikipedia.org/wiki/Advanced_Packaging_Tool) son muy conocidos en el mundo de Linux, pero no tanto en el de Windows.
[Chocolatey](https://chocolatey.org/) es uno de ellos, y Scott Hanselman proporciona una estupenda introducción al tema en su [blog](https://www.hanselman.com/blog/IsTheWindowsUserReadyForAptget.aspx). En pocas palabras, Chocolatey permite usar la línea de comandos para instalar paquetes desde un repositorio central en un sistema operativo Windows. Puede crear y administrar su propio repositorio y Chocolatey puede instalar paquetes desde cualquier número de repositorios que designe.

[DSC de PowerShell](/powershell/scripting/dsc/overview/overview) es una herramienta de PowerShell que permite declarar la configuración que quiera para un equipo. Por ejemplo, si quiere que se instale Chocolatey, se instale IIS, se abra el puerto 80 y se instale la versión 1.0.0 de su sitio web, Local Configuration Manager (LCM) de DSC implementa esa configuración. Un servidor de extracción de DSC contiene un repositorio de configuraciones para los equipos. El LCM de cada equipo comprueba periódicamente para ver si su configuración coincide con la almacenada. Después, notifica el estado o intenta equiparar la configuración del equipo a la almacenada. Puede editar la configuración almacenada en el servidor de extracción para hacer que un equipo o un conjunto de equipos se adapten a la configuración modificada.

Un recurso de DSC es un módulo de código con capacidades específicas, como la administración de redes, Active Directory o SQL Server. El recurso de DSC de Chocolatey sabe cómo acceder a un servidor NuGet (entre otros), descargar paquetes, instalar paquetes, etc. Existen muchos otros recursos de DSC en la [Galería de PowerShell](https://www.powershellgallery.com/packages?q=dsc+resources&prerelease=&sortOrder=package-title). Estos módulos se instalan en el servidor de extracción de Azure Automation State Configuration para que las configuraciones puedan utilizarlas.

Las plantillas de Resource Manager proporcionan una forma declarativa de generar la infraestructura, por ejemplo, las redes, las subredes, la seguridad de red y el enrutamiento, los equilibradores de carga, las NIC, las máquinas virtuales, etc. En este [artículo](../azure-resource-manager/management/deployment-models.md), se compara el modelo de implementación de Resource Manager (declarativo) con el de Administración de servicios de Azure (ASM o clásico), que es imperativo. En este artículo se incluye una explicación de los proveedores de recursos principales: proceso, almacenamiento y red.

Una característica clave de una plantilla de Resource Manager es su capacidad para instalar una extensión de máquina virtual en la máquina virtual cuando se aprovisiona. Una extensión de máquina virtual posee capacidades específicas, como ejecutar un script personalizado, instalar software antivirus y ejecutar un script de configuración de DSC. Existen muchos otros tipos de extensiones de máquina virtual.

## <a name="quick-trip-around-the-diagram"></a>Vista rápida al diagrama

Empezando desde arriba, se escribe el código, se compila, se prueba y después se crea un paquete de instalación. Chocolatey admite diversos tipos de paquetes de instalación, como MSI, MSU o ZIP. Además, cuenta con toda la funcionalidad de PowerShell para realizar la instalación en sí si las capacidades nativas de Chocolatey no son suficientes. Coloque el paquete en un lugar accesible: un repositorio de paquetes. En este ejemplo de uso se emplea una carpeta pública en una cuenta de almacenamiento de blobs de Azure, pero vale cualquier lugar. Chocolatey funciona de forma nativa con los servidores NuGet y otros para administrar los metadatos de los paquetes. [este artículo](https://github.com/chocolatey/choco/wiki/How-To-Host-Feed) se describen las opciones. En este ejemplo de uso se utiliza NuGet. Nuspec son los metadatos de los paquetes. La información de Nuspec se compila en un NuPkg y se almacena en un servidor NuGet. Cuando la configuración solicita un paquete por su nombre y hace referencia a un servidor NuGet, el recurso de DSC de Chocolatey en la máquina virtual toma el paquete y lo instala. También se puede solicitar una versión específica de un paquete.

En la parte inferior izquierda de la imagen, se ve una plantilla de Azure Resource Manager. En este ejemplo de uso, la extensión de máquina virtual registra la máquina virtual con el servidor de extracción de Azure Automation State Configuration como nodo. La configuración se almacena en el servidor de extracción dos veces: una vez como texto sin formato y una vez compilada como un archivo MOF. En Azure Portal, MOF representa una configuración de nodo, en lugar de una configuración simple. Es el artefacto que se asocia con un nodo para que el nodo conozca su configuración. Los siguientes detalles muestran cómo asignar la configuración de nodo al nodo.

Crear el nuspec, compilarlo y almacenarlo en un servidor NuGet no es muy complicado y, además, ya está administrando máquinas virtuales. 

Para dar el siguiente paso en la implementación continua, se debe configurar el servidor de extracción una vez, registrar los nodos con él una vez y crear y almacenar la configuración inicial en el servidor. Como los paquetes se actualizan e implementan en el repositorio, solo debe actualizar la configuración y la configuración de nodo en el servidor de extracción si es necesario.

No comenzar con una plantilla de Resource Manager es perfectamente válido. Hay comandos de PowerShell para ayudar a registrar las máquinas virtuales con el servidor de extracción. Para más información, consulte [Incorporación de máquinas para administrarlas con State Configuration de Azure Automation](automation-dsc-onboarding.md).

## <a name="about-the-usage-example"></a>Acerca del ejemplo de uso

En este ejemplo de uso de este artículo se comienza con una máquina virtual de una imagen genérica de Windows Server 2012 R2 procedente de Azure Gallery. Puede iniciarla desde cualquier imagen almacenada y después ajustarla con la configuración de DSC.
Sin embargo, cambiar la configuración que se incluye en una imagen es mucho más difícil que actualizar dinámicamente la configuración con DSC.

No es necesario usar una plantilla de Resource Manager ni la extensión de máquina virtual para emplear esta técnica con sus máquinas virtuales. Además, sus máquinas virtuales tampoco tienen que estar en Azure para administrarlas mediante CD. Todo lo que se necesita es que Chocolatey esté instalado y LCM esté configurado en la máquina virtual para que conozca la ubicación del servidor de extracción.

Cuando se actualiza un paquete en una máquina virtual que se encuentra en producción, debe sacar esa máquina virtual de la rotación mientras se instala la actualización. Existen métodos muy diversos de hacerlo. Por ejemplo, con una máquina virtual tras un equilibrador de carga de Azure, puede agregar un sondeo personalizado. Al actualizar la máquina virtual, haga que el extremo de sondeo devuelva 400. El ajuste necesario para ocasionar este cambio puede estar dentro de la configuración, al igual que el ajuste para devolver de nuevo 200 una vez completada la actualización.

El código fuente completo de este ejemplo de uso se encuentra en [este proyecto de Visual Studio](https://github.com/sebastus/ARM/tree/master/CDIaaSVM) en GitHub.

## <a name="step-1-set-up-the-pull-server-and-automation-account"></a>Paso 1: Configurar la cuenta de Automation y el servidor de extracción

En una línea de comandos de PowerShell (`Connect-AzAccount`) autenticada: (puede tardar unos minutos mientras se configura el servidor de extracción)

```azurepowershell-interactive
New-AzResourceGroup -Name MY-AUTOMATION-RG -Location MY-RG-LOCATION-IN-QUOTES
New-AzAutomationAccount -ResourceGroupName MY-AUTOMATION-RG -Location MY-RG-LOCATION-IN-QUOTES -Name MY-AUTOMATION-ACCOUNT
```

Puede colocar su cuenta de Automation en cualquiera de las siguientes regiones (denominadas ubicaciones): Este de EE. UU. 2, Centro-sur de EE. UU., US Gov Virginia, Oeste de Europa, Sudeste de Asia, Este de Japón, Centro de la India y Sudeste de Australia, Centro de Canadá, Norte de Europa.

## <a name="step-2-make-vm-extension-tweaks-to-the-resource-manager-template"></a>Paso 2: Realizar ajustes de la extensión de máquina virtual en la plantilla de Resource Manager

Se proporcionan detalles sobre el registro de máquina virtual (con la extensión de máquina virtual de DSC de PowerShell) en esta [plantilla de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/dsc-extension-azure-automation-pullserver).
En este paso, se registra la nueva máquina virtual con el servidor de extracción en la lista de nodos de State Configuration. Parte de este registro es especificar la configuración del nodo que se aplica al nodo. Esta configuración de nodo no tiene que existir aún en el servidor de extracción, por lo que es correcto que sea el paso 4 en donde se realice esto por primera vez. Sin embargo, aquí en el paso 2 debe haber decidido el nombre del nodo y el nombre de la configuración. En este ejemplo de uso, el nodo es "isvbox" y la configuración es "ISVBoxConfig". Por tanto, el nombre de configuración del nodo (que se especificará en DeploymentTemplate.json) es "ISVBoxConfig.isvbox".

## <a name="step-3-add-required-dsc-resources-to-the-pull-server"></a>Paso 3: Agregar recursos de DSC necesarios al servidor de extracción

La Galería de PowerShell está equipada para instalar recursos de DSC en su cuenta de Azure Automation.
Navegue hasta el recurso que desee y haga clic en el botón "Deploy to Azure Automation" (Implementar en Azure Automation).

![Ejemplo de la Galería de PowerShell](./media/automation-dsc-cd-chocolatey/xNetworking.PNG)

Otra técnica agregada hace poco a Azure Portal permite extraer módulos nuevos o actualizar los existentes. Haga clic en el recurso Cuenta de Automation, el icono Activos y, por último, el icono Módulos. El icono de examinar la galería permite ver la lista de módulos de la galería, profundizar en los detalles y, en última instancia, importarlos a la cuenta de Automation. Se trata de un modo excelente de actualizar los módulos de vez en cuando. Además, la característica de importación comprueba las dependencias con otros módulos para asegurarse de que nada se queda sin sincronizarse.

También hay un enfoque manual, que se usa solo una vez por recurso, a menos que desee actualizarlo más adelante. Para más información sobre la creación de módulos de integración de PowerShell, consulte [Creación de módulos de integración para Azure Automation](https://azure.microsoft.com/blog/authoring-integration-modules-for-azure-automation/).

>[!NOTE]
>La estructura de carpetas de un módulo de integración de PowerShell para un equipo Windows es algo diferente de la estructura de carpetas que espera Azure Automation. 

1. Instale [Windows Management Framework v5](https://aka.ms/wmf5latest) (no es necesario para Windows 10).

2. Instale el módulo de integración.

    ```azurepowershell-interactive
    Install-Module -Name MODULE-NAME`    <—grabs the module from the PowerShell Gallery
    ```

3. Copie la carpeta del módulo desde **C:\Archivos de programa\WindowsPowerShell\Modules\MODULE-NAME** en una carpeta temporal.

4. Elimine los ejemplos y la documentación de la carpeta principal.

5. Comprima la carpeta principal y asigne al archivo comprimido el nombre de la carpeta.

6. Coloque el archivo comprimido en una ubicación HTTP accesible, como un almacenamiento de blobs en una cuenta de Azure Storage.

7. Ejecute el siguiente comando:

    ```azurepowershell-interactive
    New-AzAutomationModule `
      -ResourceGroupName MY-AUTOMATION-RG -AutomationAccountName MY-AUTOMATION-ACCOUNT `
      -Name MODULE-NAME -ContentLinkUri 'https://STORAGE-URI/CONTAINERNAME/MODULE-NAME.zip'
    ```

El ejemplo incluido implementa estos pasos para cChoco y xNetworking. 

## <a name="step-4-add-the-node-configuration-to-the-pull-server"></a>Paso 4: Agregar la configuración de nodo al servidor de extracción

No hay nada especial que mencionar acerca de la primera vez que se importa la configuración al servidor de extracción y se compila. Todas las importaciones o compilaciones posteriores de la misma configuración tendrán exactamente el mismo aspecto. Cada vez que actualice el paquete y deba enviarlo a producción, realice este paso después de asegurarse de que el archivo de configuración sea correcto, incluida la nueva versión del paquete. Este es el archivo de configuración **ISVBoxConfig.ps1**:

```powershell
Configuration ISVBoxConfig
{
    Import-DscResource -ModuleName cChoco
    Import-DscResource -ModuleName xNetworking

    Node 'isvbox' {

        cChocoInstaller installChoco
        {
            InstallDir = 'C:\choco'
        }

        WindowsFeature installIIS
        {
            Ensure = 'Present'
            Name   = 'Web-Server'
        }

        xFirewall WebFirewallRule
        {
            Direction    = 'Inbound'
            Name         = 'Web-Server-TCP-In'
            DisplayName  = 'Web Server (TCP-In)'
            Description  = 'IIS allow incoming web site traffic.'
            Enabled       = 'True'
            Action       = 'Allow'
            Protocol     = 'TCP'
            LocalPort    = '80'
            Ensure       = 'Present'
        }

        cChocoPackageInstaller trivialWeb
        {
            Name      = 'trivialweb'
            Version   = '1.0.0'
            Source    = 'MY-NUGET-V2-SERVER-ADDRESS'
            DependsOn = '[cChocoInstaller]installChoco','[WindowsFeature]installIIS'
        }
    }
}
```

Este es el script **New-ConfigurationScript.ps1** (modificado para usar el módulo Az):

```powershell
Import-AzAutomationDscConfiguration `
    -ResourceGroupName MY-AUTOMATION-RG -AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -SourcePath C:\temp\AzureAutomationDsc\ISVBoxConfig.ps1 `
    -Published -Force

$jobData = Start-AzAutomationDscCompilationJob `
    -ResourceGroupName MY-AUTOMATION-RG -AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -ConfigurationName ISVBoxConfig

$compilationJobId = $jobData.Id

Get-AzAutomationDscCompilationJob `
    -ResourceGroupName MY-AUTOMATION-RG -AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -Id $compilationJobId
```

Estos pasos dan como resultado que se coloque una nueva configuración de nodo denominada **ISVBoxConfig.isvbox** en el servidor de extracción. El nombre de configuración de nodo se genera como `configurationName.nodeName`.

## <a name="step-5-create-and-maintain-package-metadata"></a>Paso 5: Crear y mantener los metadatos de paquetes

Cada paquete que incluya en el repositorio de paquetes necesita un nuspec que lo describa. Se debe compilar y almacenar en el servidor NuGet. Este proceso se describe [aquí](https://docs.nuget.org/create/creating-and-publishing-a-package). 

Puede usar **MyGet.org** como servidor NuGet. Puede comprar este servicio, pero existe una SKU de inicio gratuita. En [NuGet](https://www.nuget.org/) encontrará instrucciones para instalar su propio servidor NuGet para los paquetes privados.

## <a name="step-6-tie-it-all-together"></a>Paso 6: Unirlo todo

Cada vez que una versión pasa un control de calidad y se aprueba para implementación, se crea el paquete, se actualizan nuspec y nupkg, y se implementan en el servidor NuGet. Se debe actualizar también la configuración (paso 4) para que coincida con el nuevo número de versión. A continuación, se debe enviar al servidor de extracción y se debe compilar.

Desde ese momento, corresponderá a las máquinas virtuales que dependen de esa configuración extraer la actualización e instalarla. Cada una de estas actualizaciones es sencilla: solo una línea o dos de PowerShell. En el caso de Azure DevOps, algunas de ellas se encapsulan en tareas de compilación que se pueden encadenar de manera conjunta en una compilación. En [este artículo](https://www.visualstudio.com/docs/alm-devops-feature-index#continuous-delivery) se ofrecen más detalles. En este [repositorio de GitHub](https://github.com/Microsoft/vso-agent-tasks) se detallan las tareas de compilación disponibles.

## <a name="related-articles"></a>Artículos relacionados
* [Información general de DSC de Azure Automation](automation-dsc-overview.md)
* [Incorporación de máquinas para administrarlas con DSC de Azure Automation](automation-dsc-onboarding.md)

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, vea [Información general de Azure Automation State Configuration](automation-dsc-overview.md).
- Para empezar a usar la característica, vea [Introducción a Azure Automation State Configuration](automation-dsc-getting-started.md).
- Para aprender a compilar configuraciones de DSC para poder asignarlas a los nodos de destino, consulte [Compilación de configuraciones de DSC en Azure Automation State Configuration](automation-dsc-compile.md).
- Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
- Para obtener información de precios, consulte [Precios de State Configuration de Azure Automation](https://azure.microsoft.com/pricing/details/automation/).
