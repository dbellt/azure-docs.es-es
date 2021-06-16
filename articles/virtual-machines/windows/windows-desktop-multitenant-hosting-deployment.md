---
title: Implementación de Windows 10 en Azure con derechos de hospedaje multiinquilino
description: Obtenga información sobre cómo maximizar las ventajas de Software Assurance de Windows para incorporar licencias locales en Azure con derechos de hospedaje multiinquilino.
author: mimckitt
ms.service: virtual-machines
ms.collection: windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 2/2/2021
ms.author: mimckitt
ms.custom: rybaker, chmimckitt, devx-track-azurepowershell
ms.openlocfilehash: 9a2b93a951b9aa2056ee547d8d74c90eb7d96f71
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983841"
---
# <a name="how-to-deploy-windows-10-on-azure-with-multitenant-hosting-rights"></a>Implementación de Windows 10 en Azure con derechos de hospedaje multiinquilino 
Para los clientes con Windows 10 Enterprise E3/E5 por usuario o con acceso a escritorios virtuales de Windows por usuario (licencias de suscripción de usuarios o licencias de suscripción de usuario de complemento), los derechos de hospedaje multiinquilino de Windows 10 le permiten llevar sus licencias de Windows 10 a la nube y ejecutar máquinas virtuales de Windows 10 en Azure sin pagar por otra licencia. Los derechos de hospedaje multiinquilino solo están disponibles para Windows 10 (versión 1703 o posterior).

Para más información, consulte [Hospedaje multiinquilino para Windows 10](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx).

> [!NOTE]
> - Para usar imágenes de Windows 7, 8.1 y 10 en desarrollo o pruebas, consulte [Cliente de Windows en Azure para escenarios de desarrollo y pruebas](client-images.md).
> - Para revisar las ventajas de licencias de Windows Server, consulte [Ventaja para uso híbrido de Azure para Windows Server](hybrid-use-benefit-licensing.md).

## <a name="subscription-licenses-that-qualify-for-multitenant-hosting-rights"></a>Licencias de suscripción que cumplen los requisitos de los derechos de hospedaje multiinquilino

Con el [centro de administración de Microsoft](/microsoft-365/admin/admin-overview/about-the-admin-center) puede confirmar si un usuario tiene asignada una licencia admitida de Windows 10.

> [!IMPORTANT]
> Para poder usar imágenes de Windows 10 en Azure, los usuarios deben tener una de las siguientes licencias de suscripción. Si no la tienen, se pueden adquirir a través de su [asociado de servicios en la nube](https://azure.microsoft.com/overview/choosing-a-cloud-service-provider/) o directamente a través de [Microsoft](https://www.microsoft.com/microsoft-365?rtc=1).

**Licencias de suscripción aptas:**

-   Microsoft 365 E3/E5 
-   Microsoft 365 F3 
-   Microsoft 365 A3/A5 
-   Windows 10 Enterprise E3/E5
-   Windows 10 Education A3/A5 
-   Windows VDA E3/E5


## <a name="deploying-windows-10-image-from-azure-marketplace"></a>Implementación de la imagen de Windows 10 desde Azure Marketplace 
En el caso de las implementaciones de plantillas de PowerShell, CLI y Azure Resource Manager, se pueden encontrar imágenes de Windows 10 mediante `PublisherName: MicrosoftWindowsDesktop` y `Offer: Windows-10`. La versión Creators Update (1809) o posterior de Windows 10 es compatible con los derechos de hospedaje multiinquilino. 

```powershell
Get-AzVmImageSku -Location '$location' -PublisherName 'MicrosoftWindowsDesktop' -Offer 'Windows-10'

Skus                        Offer      PublisherName           Location 
----                        -----      -------------           -------- 
rs4-pro                     Windows-10 MicrosoftWindowsDesktop eastus   
rs4-pron                    Windows-10 MicrosoftWindowsDesktop eastus   
rs5-enterprise              Windows-10 MicrosoftWindowsDesktop eastus   
rs5-enterprisen             Windows-10 MicrosoftWindowsDesktop eastus   
rs5-pron                    Windows-10 MicrosoftWindowsDesktop eastus  
```

Para más información sobre las imágenes disponibles, consulte [Búsqueda y uso de imágenes de máquina virtual de Azure Marketplace con Azure PowerShell](./cli-ps-findimage.md).

## <a name="uploading-windows-10-vhd-to-azure"></a>Carga del disco duro virtual de Windows 10 en Azure
Si va a cargar un disco duro virtual generalizado de Windows 10, tenga en cuenta que Windows 10 no tiene una cuenta de administrador integrada habilitada de forma predeterminada. Para habilitar la cuenta de administrador integrada, incluya el siguiente comando como parte de la extensión de script personalizada.

```powershell
Net user <username> /active:yes
```

El siguiente fragmento de código de PowerShell sirve para marcar todas las cuentas de administrador como activas, incluida la del administrador integrado. Este ejemplo es útil si desconoce el nombre de usuario del administrador integrado.
```powershell
$adminAccount = Get-WmiObject Win32_UserAccount -filter "LocalAccount=True" | ? {$_.SID -Like "S-1-5-21-*-500"}
if($adminAccount.Disabled)
{
    $adminAccount.Disabled = $false
    $adminAccount.Put()
}
```
Para obtener más información: 
* [Carga del disco duro virtual en Azure](upload-generalized-managed.md)
* [Preparación de un disco duro virtual de Windows para cargarlo en Azure](prepare-for-upload-vhd-image.md)


## <a name="deploying-windows-10-with-multitenant-hosting-rights"></a>Implementación de Windows 10 con derechos de hospedaje multiinquilino
Asegúrese de que tiene [la versión de Azure PowerShell más reciente instalada y configurada](/powershell/azure/). Cuando haya preparado el disco duro virtual, cárguelo en su cuenta de Azure Storage mediante el cmdlet `Add-AzVhd` de la siguiente forma:

```powershell
Add-AzVhd -ResourceGroupName "myResourceGroup" -LocalFilePath "C:\Path\To\myvhd.vhd" `
    -Destination "https://mystorageaccount.blob.core.windows.net/vhds/myvhd.vhd"
```


**Implementación mediante la plantilla de Azure Resource Manager**: dentro de las plantillas de Resource Manager, se puede especificar un parámetro adicional para `licenseType`. En [Creación de plantillas de Azure Resource Manager](../../azure-resource-manager/templates/syntax.md), puede encontrar más información al respecto. Una vez que haya cargado el VHD en Azure, edite la plantilla de Resource Manager para incluir el tipo de licencia como parte del proveedor de procesos e impleméntela de la forma habitual:
```json
"properties": {
    "licenseType": "Windows_Client",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
```

**Implementación mediante PowerShell**: cuando se implementa una máquina virtual de Windows Server mediante PowerShell, se dispone de un parámetro adicional para `-LicenseType`. Cuando el disco duro virtual esté cargado en Azure, cree una máquina virtual mediante `New-AzVM` y especifique el tipo de concesión de licencias de la siguiente forma:
```powershell
New-AzVM -ResourceGroupName "myResourceGroup" -Location "West US" -VM $vm -LicenseType "Windows_Client"
```

## <a name="verify-your-vm-is-utilizing-the-licensing-benefit"></a>Comprobación de que la máquina virtual está utilizando la ventaja de licencia
Cuando haya implementado la máquina virtual mediante el método de implementación de Resource Manager o PowerShell, compruebe el tipo de licencia con `Get-AzVM` , como se indica a continuación:
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

La salida es similar a la del ejemplo siguiente para Windows 10 con el tipo correcto de licencia:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Client
```

Esta salida contrasta con la siguiente máquina virtual implementada sin la licencia de ventaja de uso híbrido de Azure, como una implementada directamente desde la galería de Azure:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

## <a name="additional-information-about-joining-azure-ad"></a>Información adicional sobre la unión con Azure AD
Azure aprovisiona todas las máquinas virtuales Windows con la cuenta de administrador integrada, que no se puede usar para unirse con AAD. Por ejemplo, *Configuración > Cuenta > Obtener acceso a trabajo o escuela > +Conectar* no funcionará. Debe crear e iniciar sesión como una segunda cuenta de administrador para unirse de forma manual a Azure AD. También puede configurar Azure AD mediante un paquete de aprovisionamiento. Use el vínculo de la sección *Pasos siguientes* para obtener más información.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre la [configuración de VDA para Windows 10](/windows/deployment/vda-subscription-activation).
- Obtenga más información sobre el [hospedaje multiinquilino para Windows 10](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx)
