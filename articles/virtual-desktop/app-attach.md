---
title: 'Configuración de scripts de PowerShell para asociación de aplicaciones en formato MSIX para Windows Virtual Desktop: Azure'
description: Cree scripts de PowerShell para la asociación de aplicaciones en formato MSIX para Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 04/13/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 43a8cb00804927784982999db13ee193c34f55ca
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107835387"
---
# <a name="create-powershell-scripts-for-msix-app-attach"></a>Creación de scripts de PowerShell para la asociación de aplicaciones en formato MSIX (versión preliminar)

Este tema le guiará a través de la configuración de scripts de PowerShell para la asociación de aplicaciones en formato MSIX.

## <a name="install-certificates"></a>Instalación de certificados

Debe instalar certificados en todos los hosts de sesión del grupo de hosts que hospedará las aplicaciones de los paquetes de asociación de aplicaciones en formato MSIX.

Si la aplicación usa un certificado que no es de confianza pública o que fue autofirmado, puede instalarlo de esta manera:

1. Haga clic con el botón derecho en el paquete y seleccione **Propiedades**.
2. En la ventana que aparece, seleccione la pestaña **Firmas digitales**. Solo habrá un elemento en la lista en la pestaña. Seleccione ese elemento para resaltarlo y, luego, elija **Detalles**.
3. Cuando aparezca la ventana de detalles de firma digital, seleccione la pestaña **General**, luego, **Ver certificado** y, por último, **Instalar certificado**.
4. Cuando se abra el instalador, seleccione **máquina local** como su ubicación de almacenamiento y, luego, seleccione **Siguiente**.
5. Si el instalador le pregunta si quiere permitir que la aplicación haga cambios en el dispositivo, seleccione **Sí**.
6. Seleccione **Colocar todos los certificados en el siguiente almacén** y, después, **Examinar**.
7. Cuando aparezca la ventana del almacén de certificados, seleccione **Trusted people** (Personas de confianza) y, luego, **Aceptar**.
8. Seleccione **Siguiente** y **Finalizar**.

## <a name="enable-microsoft-hyper-v"></a>Habilitación de Microsoft Hyper-V

Microsoft Hyper-V debe estar habilitado porque el comando `Mount-VHD` es necesario para la copia intermedia y `Dismount-VHD` es necesario para eliminar la copia intermedia.

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

>[!NOTE]
>Este cambio requerirá que se reinicie la máquina virtual.

## <a name="prepare-powershell-scripts-for-msix-app-attach"></a>Preparación de scripts para PowerShell para la asociación de aplicaciones en formato MSIX

La asociación de aplicaciones en formato MSIX tiene cuatro fases distintas que se deben realizar en este orden:

1. Fase
2. Register
3. Eliminar registro
4. Quitar del "stage"

En cada fase se crea un script de PowerShell. Los scripts de ejemplo para fase están disponibles [aquí](https://github.com/Azure/RDS-Templates/tree/master/msix-app-attach).

### <a name="stage-powershell-script"></a>Script de PowerShell de fase

Antes de actualizar los scripts de PowerShell, asegúrese de que tiene el GUID de volumen del volumen en el disco duro virtual. Para obtener el GUID del volumen:

1.  Abra el recurso compartido de red donde se encuentra el disco duro virtual dentro de la máquina virtual donde se ejecutará el script.

2.  Haga clic con el botón derecho y seleccione **Montar**. De este modo, el disco duro virtual se montará en una letra de unidad.

3.  Después de montar el disco duro virtual, se abrirá la ventana **Explorador de archivos**. Capture la carpeta principal y actualice la variable **$parentFolder**.

    >[!NOTE]
    >Si no ve una carpeta principal, significa que MSIX no se expandió de manera correcta. Rehaga la sección anterior y vuelva a intentarlo.

4.  Abra la carpeta principal. Si se expandió correctamente, verá una carpeta con el mismo nombre del paquete. Actualice la variable **$packageName** para que coincida con el nombre de esta carpeta.

    Por ejemplo, `VSCodeUserSetup-x64-1.38.1_1.38.1.0_x64__8wekyb3d8bbwe`.

5.  Abra un símbolo del sistema y escriba **mountvol**. Este comando mostrará una lista de volúmenes y sus GUID. Copie el GUID del volumen en el que la letra de unidad coincide con la unidad que montó en el disco duro virtual del paso 2.

    Por ejemplo, en esta salida de ejemplo del comando mountvol, si montó el disco duro virtual en la unidad C, querrá copiar el valor anterior `C:\`:

    ```cmd
    Possible values for VolumeName along with current mount points are:

    \\?\Volume{a12b3456-0000-0000-0000-10000000000}\
    *** NO MOUNT POINTS ***

    \\?\Volume{c78d9012-0000-0000-0000-20000000000}\
        E:\

    \\?\Volume{d34e5678-0000-0000-0000-30000000000}\
        C:\

    ```


6.  Actualice la variable **$volumeGuid** con el GUID del volumen que acaba de copiar.

7. Abra un símbolo del sistema de administrador de PowerShell y actualice el script de PowerShell siguiente con las variables que se aplican al entorno.

    ```powershell
    #MSIX app attach staging sample

    #region variables
    $vhdSrc="<path to vhd>"
    $packageName = "<package name>"
    $parentFolder = "<package parent folder>"
    $parentFolder = "\" + $parentFolder + "\"
    $volumeGuid = "<vol guid>"
    $msixJunction = "C:\temp\AppAttach\"
    #endregion

    #region mountvhd
    try
    {
          Mount-Diskimage -ImagePath $vhdSrc -NoDriveLetter -Access ReadOnly
          Write-Host ("Mounting of " + $vhdSrc + " was completed!") -BackgroundColor Green
    }
    catch
    {
          Write-Host ("Mounting of " + $vhdSrc + " has failed!") -BackgroundColor Red
    }
    #endregion

    #region makelink
    $msixDest = "\\?\Volume{" + $volumeGuid + "}\"
    if (!(Test-Path $msixJunction))
    {
         md $msixJunction
    }

    $msixJunction = $msixJunction + $packageName
    cmd.exe /c mklink /j $msixJunction $msixDest
    #endregion

    #region stage
    [Windows.Management.Deployment.PackageManager,Windows.Management.Deployment,ContentType=WindowsRuntime] | Out-Null
    Add-Type -AssemblyName System.Runtime.WindowsRuntime
    $asTask = ([System.WindowsRuntimeSystemExtensions].GetMethods() | Where { $_.ToString() -eq 'System.Threading.Tasks.Task`1[TResult] AsTask[TResult,TProgress](Windows.Foundation.IAsyncOperationWithProgress`2[TResult,TProgress])'})[0]
    $asTaskAsyncOperation = $asTask.MakeGenericMethod([Windows.Management.Deployment.DeploymentResult], [Windows.Management.Deployment.DeploymentProgress])
    $packageManager = [Windows.Management.Deployment.PackageManager]::new()
    $path = $msixJunction + $parentFolder + $packageName # needed if we do the pbisigned.vhd
    $path = ([System.Uri]$path).AbsoluteUri
    $asyncOperation = $packageManager.StagePackageAsync($path, $null, "StageInPlace")
    $task = $asTaskAsyncOperation.Invoke($null, @($asyncOperation))
    $task
    #endregion
    ```

### <a name="register-powershell-script"></a>Registrar el script de PowerShell

Para ejecutar el script de registro, ejecute estos cmdlets de PowerShell con los valores de marcador de posición reemplazados por los valores que se aplican al entorno.

```powershell
#MSIX app attach registration sample

#region variables
$packageName = "<package name>"
$path = "C:\Program Files\WindowsApps\" + $packageName + "\AppxManifest.xml"
#endregion

#region register
Add-AppxPackage -Path $path -DisableDevelopmentMode -Register
#endregion
```

### <a name="deregister-powershell-script"></a>Eliminar el registro del script de PowerShell

Para este script, reemplace el marcador de posición de **$packageName** por el nombre del paquete que está probando.

```powershell
#MSIX app attach deregistration sample

#region variables
$packageName = "<package name>"
#endregion

#region deregister
Remove-AppxPackage -PreserveRoamableApplicationData $packageName
#endregion
```

### <a name="destage-powershell-script"></a>Quitar el script de PowerShell del "stage"

Para este script, reemplace el marcador de posición de **$packageName** por el nombre del paquete que está probando. En una implementación de producción sería mejor ejecutar esta opción en el modo apagado.

```powershell
#MSIX app attach de staging sample

$vhdSrc="<path to vhd>"

#region variables
$packageName = "<package name>"
$msixJunction = "C:\temp\AppAttach"
#endregion

#region deregister
Remove-AppxPackage -AllUsers -Package $packageName
Remove-Item "$msixJunction\$packageName" -Recurse -Force -Verbose
#endregion

#region Detach VHD
Dismount-DiskImage -ImagePath $vhdSrc -Confirm:$false
#endregion
```

## <a name="set-up-simulation-scripts-for-the-msix-app-attach-agent"></a>Configuración de scripts de simulación para el agente de asociación de aplicaciones en formato MSIX

Después de crear los scripts, los usuarios pueden ejecutarlos manualmente o configurarlos para que se ejecuten de manera automática en como scripts de inicio, inicio de sesión, cierre de sesión y apagado. Para más información sobre estos tipos de scripts, consulte el artículo sobre cómo [usar los scripts de inicio, apagado, inicio de sesión y cierre de sesión en una directiva de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn789196(v=ws.11)/).

Cada uno de estos scripts automáticos ejecuta una fase de los scripts de asociación de aplicaciones:

- El script de inicio ejecuta el script para agregar al "stage".
- El script de inicio de sesión ejecuta el script para registrar.
- El script de cierre de sesión ejecuta el script para eliminar del registro.
- El script de apagado ejecuta el script para quitar del "stage".

## <a name="use-packages-offline"></a>Uso de paquetes sin conexión

Si usa paquetes de [Microsoft Store para Empresas](https://businessstore.microsoft.com/) o de [Microsoft Store para Educación](https://educationstore.microsoft.com/) dentro de la red o en dispositivos que no está conectados a Internet, debe obtener las licencias de paquete en Microsoft Store e instalarlas en el dispositivo para ejecutar correctamente la aplicación. Si el dispositivo está en línea y puede conectarse a Microsoft Store para Empresas, las licencias necesarias se deben descargar automáticamente pero, si está sin conexión, deberá configurar las licencias de manera manual.

Para instalar los archivos de licencia, deberá usar un script de PowerShell que llama a la clase MDM_EnterpriseModernAppManagement_StoreLicenses02_01 en el proveedor de puentes de WMI.

Aquí se muestra cómo configurar las licencias para usarlas sin conexión:

1. Descargue el paquete de la aplicación, las licencias y los marcos de trabajo requeridos de Microsoft Store para Empresas. Necesita los archivos de licencias codificados y sin codificar. [Aquí](/microsoft-store/distribute-offline-apps#download-an-offline-licensed-app) puede encontrar instrucciones detalladas sobre la descarga.
2. Actualice estas variables en el script del paso 3:
      1. `$contentID` es el valor ContentID del archivo de licencia no codificado (.xml). Puede abrir el archivo de licencia en el editor de texto que prefiera.
      2. `$licenseBlob` es la cadena entera para el blob de licencia en el archivo de licencia codificado (.bin). Puede abrir el archivo de licencia codificado en el editor de texto que prefiera.
3. Ejecute el script siguiente desde un símbolo del sistema de administrador de PowerShell. Un buen lugar para instalar la licencia es al final del [script para agregar al "stage"](#stage-powershell-script) que también se tiene que ejecutar desde un símbolo del sistema de administrador.

```powershell
$namespaceName = "root\cimv2\mdm\dmmap"
$className = "MDM_EnterpriseModernAppManagement_StoreLicenses02_01"
$methodName = "AddLicenseMethod"
$parentID = "./Vendor/MSFT/EnterpriseModernAppManagement/AppLicenses/StoreLicenses"

#TODO - Update $contentID with the ContentID value from the unencoded license file (.xml)
$contentID = "{'ContentID'_in_unencoded_license_file}"

#TODO - Update $licenseBlob with the entire String in the encoded license file (.bin)
$licenseBlob = "{Entire_String_in_encoded_license_file}"

$session = New-CimSession

#The final string passed into the AddLicenseMethod should be of the form <License Content="encoded license blob" />
$licenseString = '<License Content='+ '"' + $licenseBlob +'"' + ' />'

$params = New-Object Microsoft.Management.Infrastructure.CimMethodParametersCollection
$param = [Microsoft.Management.Infrastructure.CimMethodParameter]::Create("param",$licenseString ,"String", "In")
$params.Add($param)


try
{
   $instance = New-CimInstance -Namespace $namespaceName -ClassName $className -Property @{ParentID=$parentID;InstanceID=$contentID}
   $session.InvokeMethod($namespaceName, $instance, $methodName, $params)

}
catch [Exception]
{
     write-host $_ | out-string
}
```

## <a name="next-steps"></a>Pasos siguientes

Esta característica no se admite actualmente, pero puede preguntarle a la comunidad en [TechCommunity sobre Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

También puede dejar comentarios sobre Windows Virtual Desktop en el [Concentrador de comentarios de Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).
