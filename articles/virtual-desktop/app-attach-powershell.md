---
title: 'PowerShell para la asociación de aplicaciones en formato MSIX en Azure Virtual Desktop: Azure'
description: Configuración de la asociación de aplicaciones en formato MSIX para Azure Virtual Desktop con PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 04/13/2021
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: femila
ms.openlocfilehash: 6a5dd53bee4e6ae99559d63441f1240d5905644e
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745428"
---
# <a name="set-up-msix-app-attach-using-powershell"></a>Configuración de la asociación de aplicaciones en formato MSIX (versión preliminar) con PowerShell

Además de Azure Portal, también puede configurar la conexión de aplicaciones MSIX manualmente con PowerShell. Este artículo lo guiará a través del uso de PowerShell para configurar la asociación de aplicaciones en formato MSIX.

## <a name="requirements"></a>Requisitos

Esto es lo que necesita para configurar la asociación de aplicaciones en formato MSIX:

- Una implementación de Azure Virtual Desktop en funcionamiento. Para aprender a implementar Azure Virtual Desktop (clásico), consulte [Creación de un inquilino en Azure Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md). Para aprender a implementar Azure Virtual Desktop con la integración de Azure Resource Manager, consulte [Creación de un grupo de hosts con Azure Portal](./create-host-pools-azure-marketplace.md).
- Un grupo de hosts de Azure Virtual Desktop con al menos un host de sesión activo.
- Un grupo de aplicaciones remotas de escritorio.
- La herramienta de empaquetado MSIX.
- Una aplicación empaquetada en formato MSIX expandida en una imagen MSIX que se carga en un recurso compartido de archivos.
- Un recurso compartido de archivos en la implementación de Azure Virtual Desktop donde se almacenará el paquete MSIX.
- El recurso compartido de archivos en el que cargó la imagen MSIX también debe ser accesible para todas las máquinas virtuales (VM) del grupo de hosts. Los usuarios necesitarán permisos de solo lectura para acceder a la imagen.
- Descargue e instale PowerShell Core.
- Descargue el módulo de Azure PowerShell de vista previa pública y expándalo a una carpeta local.
- Instale el módulo de Azure mediante la ejecución de este cmdlet:

    ```powershell
    Install-Module -Name Az -Force
    ```

## <a name="sign-in-to-azure-and-import-the-module"></a>Inicio de sesión en Azure e importación del módulo

Una vez que tenga listos todos los requisitos, abra PowerShell Core en un símbolo del sistema con privilegios elevados y ejecute este cmdlet:

```powershell
Connect-AzAccount
```

Después de la ejecución, use sus credenciales para autenticar la cuenta. En este caso, puede que se le pida un token o una dirección URL de dispositivo.

## <a name="import-the-azwindowsvirtualdesktop-module"></a>Importación del módulo Az.WindowsVirtualDesktop

Necesitará el módulo Az.DesktopVirtualization para seguir las instrucciones de este artículo.

>[!NOTE]
>En el caso de la versión preliminar pública, proporcionaremos el módulo como archivos ZIP independientes que debe importar de manera manual.

Antes de empezar, puede ejecutar el cmdlet siguiente para ver si el módulo Az.DesktopVirtualization ya está instalado en la sesión o en la VM:

```powershell
Get-Module | Where-Object { $_.Name -Like "desktopvirtualization" }
```

Si quiere desinstalar una copia existente del módulo y volver a empezar, ejecute este cmdlet:

```powershell
Uninstall-Module Az.DesktopVirtualization
```

Si el módulo está bloqueado en la VM, ejecute este cmdlet para desbloquearlo:

```powershell
Unblock-File "<path>\Az.DesktopVirtualization.psm1"
```

Una vez realizado este paso de limpieza, puede importar el módulo.

1. Ejecute el cmdlet siguiente y presione la tecla **R** cuando se le pregunte si quiere ejecutar el código personalizado.

   ```powershell
   Import-Module -Name "<path>\Az.DesktopVirtualization.psm1" -Verbose
   ```

2. Una vez que ejecute el cmdlet de importación, ejecute el cmdlet siguiente para ver si tiene los cmdlets para MSIX:

   ```powershell
   Get-Command -Module Az.DesktopVirtualization | Where-Object { $_.Name -match "MSIX" }
   ```

   Si los cmdlets están presentes, la salida debería ser similar a la siguiente:

   ```powershell
   CommandType     Name                                               Version    Source

   -----------     ----                                               -------    ------

   Function        Expand-AzWvdMsixImage                              0.0        Az.DesktopVirtualization

   Function        Get-AzWvdMsixPackage                               0.0        Az.DesktopVirtualization

   Function        New-AzWvdMsixPackage                               0.0        Az.DesktopVirtualization

   Function        Remove-AzWvdMsixPackage                            0.0        Az.DesktopVirtualization

   Function        Update-AzWvdMsixPackage                            0.0        Az.DesktopVirtualization
   ```

   Si no ve esta salida, cierre todas las sesiones de PowerShell y de PowerShell Core y vuelva a intentarlo.

## <a name="set-up-helper-variables"></a>Configuración de variables auxiliares

Cuando haya importado el módulo, deberá configurar las variables auxiliares. En los ejemplos siguientes se muestra cómo hacerlo con cada una.

Para obtener el id. de la suscripción:

```powershell
Get-AzContext -ListAvailable | fl
```

Para seleccionar el contexto de un inquilino y una suscripción de Azure con un nombre:

```powershell
$obj = Select-AzContext -Name "<Name>"
```

Para establecer la variable de suscripción:

```powershell
$subId = $obj.Subscription.Id
```

Para establecer el nombre del área de trabajo:

```powershell
$ws = "<WorksSpaceName>"
```

Para establecer el nombre del grupo de hosts:

```powershell
$hp = "<HostPoolName>"
```

Para configurar el grupo de recursos en el que se configuran las VM del host de sesión:

```powershell
$rg = "<ResourceGroupName>"
```

Y, por último, para confirmar que estableció correctamente todas las variables:

```powershell
Get-AzWvdWorkspace -Name $ws -ResourceGroupName $rg -SubscriptionId $subID
```

## <a name="add-an-msix-package-to-a-host-pool"></a>Incorporación de un paquete MSIX en un grupo de hosts

Una vez que configuró todo, es momento de agregar el paquete MSIX a un grupo de hosts. Para ello, primero debe obtener una ruta de acceso UNC a la imagen MSIX.

Con la ruta de acceso UNC, ejecute este cmdlet para expandir la imagen MSIX:

```powershell
$obj = Expand-AzWvdMsixImage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subID -Uri <UNCPath>
```

Ejecute este cmdlet para agregar el paquete MSIX al grupo de hosts deseado:

```powershell
New-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId -PackageAlias $obj.PackageAlias -DisplayName <DisplayName> -ImagePath <UNCPath> -IsActive:$true
```

Cuando termine, confirme que el paquete se creó con este cmdlet:

```powershell
Get-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId | Where-Object {$_.PackageFamilyName -eq $obj.PackageFamilyName}
```

## <a name="remove-an-msix-package-from-a-host-pool"></a>Eliminación de un paquete MSIX desde un grupo de hosts

Para quitar un paquete de un grupo de hosts:

Obtenga una lista de todos los paquetes asociados a un grupo de hosts con este cmdlet y, a continuación, busque en la salida el nombre del paquete que quiere quitar:

```powershell
Get-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId 
```

Como alternativa, también puede obtener un paquete determinado en función de su nombre para mostrar con este cmdlet:

```powershell
Get-AzWvdMsixPackage -HostPoolName $hp -ResourceGroupName $rg -SubscriptionId $subId | Where-Object { $_.Name -like "Power" }
```

Para quitar el paquete, ejecute este cmdlet:

```powershell
Remove-AzWvdMsixPackage -FullName $obj.PackageFullName -HostPoolName $hp -ResourceGroupName $rg
```

## <a name="publish-msix-apps-to-an-app-group"></a>Publicación de aplicaciones en formato MSIX en un grupo de aplicaciones

Solo puede seguir las instrucciones de esta sesión si siguió todas las instrucciones de las secciones anteriores. Si tiene un grupo de hosts con un host de sesión activo, al menos un grupo de aplicaciones de escritorio, y agregó un paquete MSIX al grupo de hosts, ya está listo para empezar.

Para publicar una aplicación desde el paquete MSIX en un grupo de aplicaciones, deberá encontrar su nombre y, a continuación, usar ese nombre en el cmdlet de publicación.

Para publicar una aplicación:

Ejecute este cmdlet para enumerar todos los grupos de aplicaciones disponibles:

```powershell
Get-AzWvdApplicationGroup -ResourceGroupName $rg -SubscriptionId $subId
```

Cuando haya encontrado el nombre del grupo de aplicaciones en el que quiere publicar las aplicaciones, use su nombre en este cmdlet:

```powershell
$grName = "<AppGroupName>"
```

Por último, tendrá que publicar la aplicación.

- Para publicar la aplicación en formato MSIX en un grupo de aplicaciones de escritorio, ejecute este cmdlet:

   ```powershell
   New-AzWvdApplication -ResourceGroupName $rg -SubscriptionId $subId -Name PowerBi -ApplicationType MsixApplication -ApplicationGroupName $grName -MsixPackageFamilyName $obj.PackageFamilyName -CommandLineSetting 0
   ```

- En su lugar, para publicar la aplicación en un grupo de aplicaciones remotas, ejecute este cmdlet:

   ```powershell
   New-AzWvdApplication -ResourceGroupName $rg -SubscriptionId $subId -Name PowerBi -ApplicationType MsixApplication -ApplicationGroupName $grName -MsixPackageFamilyName $obj.PackageFamilyName -CommandLineSetting 0 -MsixPackageApplicationId $obj.PackageApplication.AppId
   ```

>[!NOTE]
>Si un usuario está asignado a un grupo de aplicaciones remotas y a un grupo de aplicaciones de escritorio en el mismo grupo de hosts, cuando el usuario se conecte a su escritorio remoto, verá las aplicaciones en formato MSIX de ambos grupos.

## <a name="next-steps"></a>Pasos siguientes

Pregunte a la comunidad sobre esta característica en el espacio de [TechCommunity sobre Azure Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

También puede dejar comentarios sobre Azure Virtual Desktop en el [Centro de opiniones de Azure Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).

Estos son algunos otros artículos que pueden resultarle útiles:

- [Glosario sobre la conexión de aplicaciones en formato MSIX](app-attach-glossary.md)
- [Preguntas frecuentes sobre la asociación de aplicaciones en formato MSIX](app-attach-faq.md)
