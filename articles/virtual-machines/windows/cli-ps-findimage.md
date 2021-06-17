---
title: Búsqueda y uso de la información del plan de compra de Marketplace mediante PowerShell
description: Use Azure PowerShell para buscar URN de imagen y parámetros de plan de compra, como el editor, la oferta, la SKU y la versión, para las imágenes de máquina virtual de Marketplace.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 03/17/2021
ms.author: cynthn
ms.custom: contperf-fy21q3, devx-track-azurepowershell
ms.openlocfilehash: e3f49268b11d57ccabccd7c14ccc244103d1db86
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110670243"
---
# <a name="find-and-use-azure-marketplace-vm-images-with-azure-powershell"></a>Búsqueda y uso de imágenes de máquina virtual de Azure Marketplace con Azure PowerShell

En este artículo se describe cómo usar Azure PowerShell para buscar imágenes de VM en Azure Marketplace. Después, puede especificar información sobre una imagen o un plan de Marketplace al crear una máquina virtual.

Puede examinar también las imágenes y ofertas disponibles mediante[Azure Marketplace](https://azuremarketplace.microsoft.com/)o la [CLI de Azure](../linux/cli-ps-findimage.md). 

## <a name="terminology"></a>Terminología

Una imagen de Marketplace de Azure tiene los atributos siguientes:

* **Publicador**: organización que ha creado la imagen. Ejemplos: Canonical, MicrosoftWindowsServer
* **Oferta**: nombre de un grupo de imágenes relacionadas creado por un publicador. Ejemplos: Ubuntu Server, WindowsServer
* **SKU**: instancia de una oferta, por ejemplo, una versión principal de una distribución. Ejemplos: 18.04-LTS, 2019-Datacenter
* **Versión**: número de versión de una SKU de imagen. 

Estos valores se pueden pasar individualmente o como un *URN* de imagen, combinando los valores separados por dos puntos (:). Por ejemplo: *Publicador*:*Oferta*:*SKU*:*Versión*. Puede reemplazar el número de versión del URN por `latest` para usar la versión más reciente de la imagen. 

Si el editor de imágenes proporciona una licencia adicional y términos de compra, debe aceptarlos para poder usar la imagen.  Para obtener más información, consulte [Aceptación de los términos del plan de compra](#accept-purchase-plan-terms).

## <a name="list-images"></a>Lista de imágenes

Puede usar PowerShell para reducir la lista de imágenes. Reemplace los valores de las variables para que se adapten a sus necesidades.

1. Muestre una lista de los editores de imágenes mediante [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher).
    
    ```powershell
    $locName="<location>"
    Get-AzVMImagePublisher -Location $locName | Select PublisherName
    ```
1. Puede mostrar una lista de las ofertas de un editor determinado mediante [Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer).
    
    ```powershell
    $pubName="<publisher>"
    Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
    ```
1. Puede mostrar una lista de las SKU disponibles de un editor y una oferta determinados mediante [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku).
    
    ```powershell
    $offerName="<offer>"
    Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
    ```
1. Puede mostrar las versiones de la imagen de una SKU mediante [Get-AzVMImage](/powershell/module/az.compute/get-azvmimage).

    ```powershell
    $skuName="<SKU>"
    Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
    ```
    También puede usar `latest` si quiere usar la imagen más reciente y no una versión anterior específica.


Ahora puede combinar el publicador, la oferta, la SKU y la versión que se seleccionó en un URN (valores separados por :). Pase este URN con el parámetro `-Image` al crear una VM con el cmdlet [New-AzVM](/powershell/module/az.compute/new-azvm). También puede reemplazar el número de versión del URN por `latest` para obtener la versión más reciente de la imagen.

Si implementa una máquina virtual con una plantilla de Resource Manager, establecerá los parámetros de imagen individualmente en las propiedades `imageReference`. Consulte la [referencia de plantilla](/azure/templates/microsoft.compute/virtualmachines).


## <a name="view-purchase-plan-properties"></a>Visualización de las propiedades del plan de compra

Algunas imágenes de máquina virtual en Azure Marketplace tienen términos adicionales de licencia y compra que debe aceptar antes de poder implementarlas mediante programación. Tendrá que aceptar los términos de la imagen una vez para cada suscripción.

Para ver la información del plan de compra de una imagen, ejecute el cmdlet `Get-AzVMImage`. Si la propiedad `PurchasePlan` de la salida no es `null`, la imagen tienen términos que debe aceptar antes de la implementación mediante programación.  

Por ejemplo, la imagen *Windows Server 2016 Datacenter* no tiene términos adicionales, porque la información de `PurchasePlan` es `null`:

```powershell
$version = "2016.127.20170406"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName -Version $version
```

La salida tendrá una apariencia parecida a la siguiente:

```output
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2019.0.20190115
Location         : westus
PublisherName    : MicrosoftWindowsServer
Offer            : WindowsServer
Skus             : 2019-Datacenter
Version          : 2019.0.20190115
FilterExpression :
Name             : 2019.0.20190115
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : null
DataDiskImages   : []

```

En el ejemplo siguiente se muestra un comando similar para la imagen *Data Science Virtual Machine - Windows 2016* que tiene las propiedades `PurchasePlan` siguientes: `name`, `product` y `publisher`. Algunas imágenes también tienen una propiedad `promotion code`. Para implementar esta imagen, consulte las secciones siguientes para aceptar los términos y habilitar la implementación mediante programación.

```powershell
Get-AzVMImage -Location "westus" -PublisherName "microsoft-ads" -Offer "windows-data-science-vm" -Skus "windows2016" -Version "0.2.02"
```

La salida tendrá una apariencia parecida a la siguiente:

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/microsoft-ads/ArtifactTypes/VMImage/Offers/windows-data-science-vm/Skus/windows2016/Versions/19.01.14
Location         : westus
PublisherName    : microsoft-ads
Offer            : windows-data-science-vm
Skus             : windows2016
Version          : 19.01.14
FilterExpression :
Name             : 19.01.14
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : {
                     "publisher": "microsoft-ads",
                     "name": "windows2016",
                     "product": "windows-data-science-vm"
                   }
DataDiskImages   : []

```

Para ver los términos de licencia, use el cmdlet [Get-AzMarketplaceterms](/powershell/module/az.marketplaceordering/get-azmarketplaceterms) y pase los parámetros del plan de compra. La salida proporciona un vínculo a los términos de la imagen de Marketplace y muestra si anteriormente aceptó los términos. Asegúrese de que todas las letras sean minúsculas en los valores de parámetros.

```powershell
Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"
```

La salida tendrá una apariencia parecida a la siguiente:

```output
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DVM%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : 2UMWH6PHSAIM4U22HXPXW25AL2NHUJ7Y7GRV27EBL6SUIDURGMYG6IIDO3P47FFIBBDFHZHSQTR7PNK6VIIRYJRQ3WXSE6BTNUNENXA
Accepted          : False
Signdate          : 1/25/2019 7:43:00 PM
```

## <a name="accept-purchase-plan-terms"></a>Aceptación de los términos del plan de compra

Use el cmdlet [Set-AzMarketplaceterms](/powershell/module/az.marketplaceordering/set-azmarketplaceterms) para aceptar o rechazar los términos. Solo debe aceptar los términos una vez por suscripción para la imagen. Asegúrese de que todas las letras sean minúsculas en los valores de parámetros. 

```powershell
$agreementTerms=Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"

Set-AzMarketplaceTerms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016" -Terms $agreementTerms -Accept
```



```output
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DV
                    M%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : XXXXXXK3MNJ5SROEG2BYDA2YGECU33GXTD3UFPLPC4BAVKAUL3PDYL3KBKBLG4ZCDJZVNSA7KJWTGMDSYDD6KRLV3LV274DLBXXXXXX
Accepted          : True
Signdate          : 2/23/2018 7:49:31 PM
```


## <a name="create-a-new-vm-from-a-marketplace-image"></a>Creación de una máquina virtual con una imagen de Marketplace

Si ya tiene información sobre la imagen que quiere usar, puede pasarla al cmdlet [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage) para agregar datos de la imagen a la configuración de la máquina virtual. Consulte las secciones siguientes para buscar y enumerar las imágenes disponibles en Marketplace.

Algunas imágenes de pago también requieren que proporcione información del plan de compra mediante el cmdlet [Set-AzVMPlan](/powershell/module/az.compute/set-azvmplan). 

```powershell
...

$vmConfig = New-AzVMConfig -VMName "myVM" -VMSize Standard_D1

# Set the Marketplace image
$offerName = "windows-data-science-vm"
$skuName = "windows2016"
$version = "19.01.14"
$vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName $publisherName -Offer $offerName -Skus $skuName -Version $version

# Set the Marketplace plan information, if needed
$publisherName = "microsoft-ads"
$productName = "windows-data-science-vm"
$planName = "windows2016"
$vmConfig = Set-AzVMPlan -VM $vmConfig -Publisher $publisherName -Product $productName -Name $planName

...
```

Luego, la configuración de la máquina virtual se pasará junto con los demás objetos de configuración al cmdlet `New-AzVM`. Para ver un ejemplo detallado de uso de una configuración de máquina virtual con PowerShell, consulte este [script](https://github.com/Azure/azure-docs-powershell-samples/blob/master/virtual-machine/create-vm-detailed/create-windows-vm-detailed.ps1).

Si aparece un mensaje sobre la aceptación de los términos de la imagen, consulte la sección [Aceptación de los términos del plan de compra](#accept-purchase-plan-terms) más arriba.

## <a name="create-a-new-vm-from-a-vhd-with-purchase-plan-information"></a>Creación de una nueva VM a partir de un VHD con la información del plan de compra

Si tiene un disco duro virtual que se creó mediante una imagen de Azure Marketplace, puede que tenga que proporcionar la información del plan de compra al crear una máquina virtual con él.

Si todavía tiene la máquina virtual original, u otra máquina virtual creada con la misma imagen, puede obtener información sobre el nombre del plan, el editor y el producto mediante Get-AzVM. En este ejemplo se obtiene una máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup* y, después, se muestra la información del plan de compra.

```azurepowershell-interactive
$vm = Get-azvm `
   -ResourceGroupName myResourceGroup `
   -Name myVM
$vm.Plan
```

Si no recibió la información del plan antes de que se eliminara la máquina virtual original, puede presentar una [solicitud de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support). Deberá proporcionar el nombre de la máquina virtual, el identificador de la suscripción y la marca de tiempo de la operación de eliminación.

Para crear una máquina virtual con un disco duro virtual, consulte el artículo [Creación de una máquina virtual a partir de un VHD especializado](create-vm-specialized.md) e incorpore una línea para agregar la información del plan a la configuración de máquina virtual mediante [Set-AzVMPlan](/powershell/module/az.compute/set-azvmplan), como en el ejemplo siguiente:

```azurepowershell-interactive
$vmConfig = Set-AzVMPlan `
   -VM $vmConfig `
   -Publisher "publisherName" `
   -Product "productName" `
   -Name "planName"
```


## <a name="next-steps"></a>Pasos siguientes

Para crear una máquina virtual rápidamente con el cmdlet `New-AzVM` mediante información de imagen básica, consulte [Creación de una máquina virtual Windows con PowerShell](quick-create-powershell.md).

Para más información sobre el uso de imágenes de Azure Marketplace para crear imágenes personalizadas en una galería de imágenes compartidas, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](../marketplace-images.md).
