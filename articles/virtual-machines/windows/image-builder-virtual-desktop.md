---
title: 'Image Builder: creación de una imagen de Windows Virtual Desktop'
description: Cree una imagen de máquina virtual de Azure de Windows Virtual Desktop con el servicio Image Builder de Azure en PowerShell.
author: danielsollondon
ms.author: danis
ms.reviewer: cynthn
ms.date: 01/27/2021
ms.topic: article
ms.service: virtual-machines
ms.collection: windows
ms.subservice: image-builder
ms.openlocfilehash: 45f4cbea20d6e223738adf3bcb9782a0c939946d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108744144"
---
# <a name="create-a-windows-virtual-desktop-image-using-azure-vm-image-builder-and-powershell"></a>Creación de una imagen de Windows Virtual Desktop mediante Azure VM Image Builder y PowerShell

En este artículo se muestra cómo crear una imagen de Windows Virtual Desktop con estas personalizaciones:

* Instalación de [FsLogix](https://github.com/DeanCefola/Azure-WVD/blob/master/PowerShell/FSLogixSetup.ps1).
* Ejecución de un [script de optimización de Windows Virtual Desktop](https://github.com/The-Virtual-Desktop-Team/Virtual-Desktop-Optimization-Tool) desde el repositorio de la comunidad.
* Instalación de [Microsoft Teams](../../virtual-desktop/teams-on-wvd.md).
* [Restart](../linux/image-builder-json.md?bc=%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#windows-restart-customizer) (Reiniciar)
* Ejecución de [Windows Update](../linux/image-builder-json.md?bc=%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#windows-update-customizer)

Le mostraremos cómo automatizar esto mediante Azure VM Image Builder y distribuir la imagen a una instancia de [Shared Image Gallery](../shared-image-galleries.md), donde puede replicarla en otras regiones, controlar su escala y compartirla dentro y fuera de las organizaciones.


Para simplificar la implementación de una configuración de Image Builder, en este ejemplo se usa una plantilla de Azure Resource Manager con la plantilla de Image Builder anidada dentro. Esto le ofrece otras ventajas, como variables y entradas de parámetros. También puede pasar parámetros desde la línea de comandos.

Este artículo pretende ser un ejercicio de copiar y pegar.

> [!NOTE]
> Los scripts para instalar las aplicaciones se encuentran en [GitHub](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/14_Building_Images_WVD). Son solo para fines de ejemplo y pruebas, y no para cargas de trabajo de producción. 

## <a name="tips-for-building-windows-images"></a>Sugerencias para compilar imágenes de Windows 

- Tamaño de máquina virtual: el tamaño de máquina virtual predeterminado es `Standard_D1_v2`, que no es adecuado para Windows. Use `Standard_D2_v2` o mayor.
- En este ejemplo se usan los [scripts del personalizador de PowerShell](../linux/image-builder-json.md). Debe usar esta configuración o la compilación dejará de responder.

    ```json
      "runElevated": true,
      "runAsSystem": true,
    ```

    Por ejemplo:

    ```json
      {
          "type": "PowerShell",
          "name": "installFsLogix",
          "runElevated": true,
          "runAsSystem": true,
          "scriptUri": "https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/14_Building_Images_WVD/0_installConfFsLogix.ps1"
    ```
- Comentario del código: el registro de compilación de AIB (Customization.log) es muy detallado. Si comenta los scripts con "write-host", estos se enviarán a los registros y facilitarán la solución de problemas.

    ```PowerShell
     write-host 'AIB Customization: Starting OS Optimizations script'
    ```

- Códigos de salida: AIB espera que todos los scripts devuelvan un código de salida de 0; cualquier código de salida distinto de cero producirá un error de AIB en la personalización y la detención de la compilación. Si tiene scripts complejos, agregue instrumentación y emita códigos de salida, que se mostrarán en el archivo customization.log.

    ```PowerShell
     Write-Host "Exit code: " $LASTEXITCODE
    ```
- Prueba: pruebe el código antes en una máquina virtual independiente, asegúrese de que no hay mensajes de usuario, verifique que está usando el privilegio adecuado, etc.

- Redes: `Set-NetAdapterAdvancedProperty`. Esto se establece en el script de optimización, pero produce un error en la compilación de AIB, ya que desconecta la red. Esto se marca como comentario.

## <a name="prerequisites"></a>Requisitos previos

Debe tener instalados los cmdlets de Azure PowerShell más recientes. consulte [aquí](/powershell/azure/overview) para obtener información detallada sobre la instalación.

```PowerShell
# Register for Azure Image Builder Feature
Register-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages

Get-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages

# wait until RegistrationState is set to 'Registered'

# check you are registered for the providers, ensure RegistrationState is set to 'Registered'.
Get-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
Get-AzResourceProvider -ProviderNamespace Microsoft.Storage 
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute
Get-AzResourceProvider -ProviderNamespace Microsoft.KeyVault

# If they do not saw registered, run the commented out code below.

## Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
## Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
## Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
## Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
```

## <a name="set-up-environment-and-variables"></a>Configuración del entorno y las variables

```azurepowershell-interactive
# Step 1: Import module
Import-Module Az.Accounts

# Step 2: get existing context
$currentAzContext = Get-AzContext

# destination image resource group
$imageResourceGroup="wvdImageDemoRg"

# location (see possible locations in main docs)
$location="westus2"

# your subscription, this will get your current subscription
$subscriptionID=$currentAzContext.Subscription.Id

# image template name
$imageTemplateName="wvd10ImageTemplate01"

# distribution properties object name (runOutput), i.e. this gives you the properties of the managed image on completion
$runOutputName="sigOutput"

# create resource group
New-AzResourceGroup -Name $imageResourceGroup -Location $location
```

## <a name="permissions-user-identity-and-role"></a>Permisos, identidad de usuario y rol 


 Cree una identidad de usuario.

```azurepowershell-interactive
# setup role def names, these need to be unique
$timeInt=$(get-date -UFormat "%s")
$imageRoleDefName="Azure Image Builder Image Def"+$timeInt
$idenityName="aibIdentity"+$timeInt

## Add AZ PS modules to support AzUserAssignedIdentity and Az AIB
'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}

# create identity
New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName

$idenityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName).Id
$idenityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName).PrincipalId

```

Asigne permisos a la identidad para distribuir imágenes. Este comando descargará y actualizará la plantilla con los parámetros especificados anteriormente.

```azurepowershell-interactive
$aibRoleImageCreationUrl="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json"
$aibRoleImageCreationPath = "aibRoleImageCreation.json"

# download config
Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing

((Get-Content -path $aibRoleImageCreationPath -Raw) -replace '<subscriptionID>',$subscriptionID) | Set-Content -Path $aibRoleImageCreationPath
((Get-Content -path $aibRoleImageCreationPath -Raw) -replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath
((Get-Content -path $aibRoleImageCreationPath -Raw) -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath

# create role definition
New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json

# grant role definition to image builder service principal
New-AzRoleAssignment -ObjectId $idenityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
```

> [!NOTE] 
> Si ve este error: "New-AzRoleDefinition: se ha superado el límite de definición de roles. No se pueden crear más definiciones de rol", consulte este artículo para resolverlo: https://docs.microsoft.com/azure/role-based-access-control/troubleshooting.



## <a name="create-the-shared-image-gallery"></a>Creación de la instancia de Shared Image Gallery 

Si aún no tiene una instancia de Shared Image Gallery, debe crearla.

```azurepowershell-interactive
$sigGalleryName= "myaibsig01"
$imageDefName ="win10wvd"

# create gallery
New-AzGallery -GalleryName $sigGalleryName -ResourceGroupName $imageResourceGroup  -Location $location

# create gallery definition
New-AzGalleryImageDefinition -GalleryName $sigGalleryName -ResourceGroupName $imageResourceGroup -Location $location -Name $imageDefName -OsState generalized -OsType Windows -Publisher 'myCo' -Offer 'Windows' -Sku '10wvd'

```

## <a name="configure-the-image-template"></a>Configuración de la plantilla de imagen

Para este ejemplo, tenemos una plantilla lista para que descargue y actualice la plantilla con los parámetros especificados anteriormente, instalará FsLogix, optimizaciones del sistema operativo y Microsoft Teams y ejecutará Windows Update al final.

Si abre la plantilla, puede ver en la propiedad source la imagen que se está utilizando, en este ejemplo se utiliza una imagen de Win 10 de sesión múltiple. 

### <a name="windows-10-images"></a>Imágenes de Windows 10
Dos tipos de clave que debe tener en cuenta: sesión múltiple y sesión única.

Las imágenes de sesión múltiple están pensadas para el uso agrupado. Este es un ejemplo de los detalles de imagen en Azure:

```json
"publisher": "MicrosoftWindowsDesktop",
"offer": "Windows-10",
"sku": "20h2-evd",
"version": "latest"
```

Las imágenes de sesión única están preparadas para su uso individual. Este es un ejemplo de los detalles de imagen en Azure:

```json
"publisher": "MicrosoftWindowsDesktop",
"offer": "Windows-10",
"sku": "19h2-ent",
"version": "latest"
```

También puede cambiar las imágenes de Win10 disponibles:

```azurepowershell-interactive
Get-AzVMImageSku -Location westus2 -PublisherName MicrosoftWindowsDesktop -Offer windows-10
```

## <a name="download-template-and-configure"></a>Descarga de la plantilla y configuración

Ahora, debe descargar la plantilla y configurarla para su uso.

```azurepowershell-interactive
$templateUrl="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/14_Building_Images_WVD/armTemplateWVD.json"
$templateFilePath = "armTemplateWVD.json"

Invoke-WebRequest -Uri $templateUrl -OutFile $templateFilePath -UseBasicParsing

((Get-Content -path $templateFilePath -Raw) -replace '<subscriptionID>',$subscriptionID) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<rgName>',$imageResourceGroup) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<region>',$location) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<runOutputName>',$runOutputName) | Set-Content -Path $templateFilePath

((Get-Content -path $templateFilePath -Raw) -replace '<imageDefName>',$imageDefName) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<sharedImageGalName>',$sigGalleryName) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<region1>',$location) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<imgBuilderId>',$idenityNameResourceId) | Set-Content -Path $templateFilePath

```

No dude en ver la [plantilla](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/14_Building_Images_WVD/armTemplateWVD.json); todo el código es visible.


## <a name="submit-the-template"></a>Envío de la plantilla

La plantilla debe enviarse al servicio, se descargarán todos los artefactos dependientes (como los scripts), se validarán, se comprobarán los permisos y se almacenarán en el grupo de recursos de almacenamiento provisional, con el prefijo *IT_* .

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName $imageResourceGroup -TemplateFile $templateFilePath -api-version "2020-02-14" -imageTemplateName $imageTemplateName -svclocation $location

# Optional - if you have any errors running the above, run:
$getStatus=$(Get-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName)
$getStatus.ProvisioningErrorCode 
$getStatus.ProvisioningErrorMessage
```
 
## <a name="build-the-image"></a>Compilación de la imagen
```azurepowershell-interactive
Start-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName -NoWait
```

> [!NOTE]
> El comando no esperará a que el servicio del generador de imágenes complete la compilación de la imagen, puede consultar el estado a continuación.

```azurepowershell-interactive
$getStatus=$(Get-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName)

# this shows all the properties
$getStatus | Format-List -Property *

# these show the status the build
$getStatus.LastRunStatusRunState 
$getStatus.LastRunStatusMessage
$getStatus.LastRunStatusRunSubState
```
## <a name="create-a-vm"></a>Crear una VM
Ahora que la compilación ha finalizado, puede crear una VM a partir de la imagen. Use para ello [estos](/powershell/module/az.compute/new-azvm#examples) ejemplos.

## <a name="clean-up"></a>Limpieza

Elimine primero la plantilla del grupo de recursos, no se limite a eliminar todo el grupo de recursos, ya que de lo contrario no se limpiará el grupo de recursos de almacenamiento provisional (*IT_* ) usado por AIB.

Quite la plantilla de imagen.

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name wvd10ImageTemplate
```

Elimine la asignación de roles.

```azurepowershell-interactive
Remove-AzRoleAssignment -ObjectId $idenityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"

## remove definitions
Remove-AzRoleDefinition -Name "$idenityNamePrincipalId" -Force -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"

## delete identity
Remove-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName -Force
```

Elimine el grupo de recursos.

```azurepowershell-interactive
Remove-AzResourceGroup $imageResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

Puede probar más ejemplos [en GitHub](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts).