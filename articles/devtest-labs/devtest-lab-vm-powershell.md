---
title: Creación de una máquina virtual en DevTest Labs con Azure PowerShell
description: Aprenda a usar Azure DevTest Labs para crear y administrar máquinas virtuales con Azure PowerShell.
ms.topic: article
ms.date: 06/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f5be9c6e11bb5037c3dbecfa256ba6a6b8058805
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110703716"
---
# <a name="create-a-virtual-machine-with-devtest-labs-using-azure-powershell"></a>Creación de una máquina virtual con DevTest Labs mediante Azure PowerShell
Este artículo muestra cómo crear una máquina virtual en Azure DevTest Labs mediante Azure PowerShell. Puede usar scripts de PowerShell para automatizar la creación de máquinas virtuales en un laboratorio en Azure DevTest Labs. 

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar:

- [Cree un laboratorio](devtest-lab-create-lab.md) si no desea usar un laboratorio existente para probar el script o los comandos de este artículo. 
- [Instale Azure PowerShell](/powershell/azure/install-az-ps) o use la instancia de Azure Cloud Shell que se integra en Azure Portal. 

## <a name="powershell-script"></a>Script de PowerShell
El script de ejemplo de esta sección utiliza el cmdlet [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction).  Este cmdlet toma el identificador de recurso del laboratorio, el nombre de la acción para realizar (`createEnvironment`) y los parámetros necesarios para realizar dicha acción. Los parámetros están en una tabla hash que contiene todas las propiedades de descripción de la máquina virtual. 

```powershell
[CmdletBinding()]

Param(
[Parameter(Mandatory = $false)]  $SubscriptionId,
[Parameter(Mandatory = $true)]   $LabResourceGroup,
[Parameter(Mandatory = $true)]   $LabName,
[Parameter(Mandatory = $true)]   $NewVmName,
[Parameter(Mandatory = $true)]   $UserName,
[Parameter(Mandatory = $true)]   $Password
)
 
pushd $PSScriptRoot

try {
    if ($SubscriptionId -eq $null) {
        $SubscriptionId = (Get-AzContext).Subscription.SubscriptionId
    }

    $API_VERSION = '2016-05-15'
    $lab = Get-AzResource -ResourceId "/subscriptions/$SubscriptionId/resourceGroups/$LabResourceGroup/providers/Microsoft.DevTestLab/labs/$LabName"

    if ($lab -eq $null) {
       throw "Unable to find lab $LabName resource group $LabResourceGroup in subscription $SubscriptionId."
    }

    #For this example, we are getting the first allowed subnet in the first virtual network
    #  for the lab.
    #If a specific virtual network is needed use | to find it. 
    #ie $virtualNetwork = @(Get-AzResource -ResourceType  'Microsoft.DevTestLab/labs/virtualnetworks' -ResourceName $LabName -ResourceGroupName $lab.ResourceGroupName -ApiVersion $API_VERSION) | Where-Object Name -EQ "SpecificVNetName"

    $virtualNetwork = @(Get-AzResource -ResourceType  'Microsoft.DevTestLab/labs/virtualnetworks' -ResourceName $LabName -ResourceGroupName $lab.ResourceGroupName -ApiVersion $API_VERSION)[0]

    $labSubnetName = $virtualNetwork.properties.allowedSubnets[0].labSubnetName

    #Prepare all the properties needed for the createEnvironment
    # call used to create the new VM.
    # The properties will be slightly different depending on the base of the vm
    # (a marketplace image, custom image or formula).
    # The setup of the virtual network to be used may also affect the properties.
    # This sample includes the properties to add an additional disk under dataDiskParameters
    
    $parameters = @{
       "name"      = $NewVmName;
       "location"  = $lab.Location;
       "properties" = @{
          "labVirtualNetworkId"     = $virtualNetwork.ResourceId;
          "labSubnetName"           = $labSubnetName;
          "notes"                   = "Windows Server 2016 Datacenter";
          "osType"                  = "windows"
          "expirationDate"          = "2019-12-01"
          "galleryImageReference"   = @{
             "offer"     = "WindowsServer";
             "publisher" = "MicrosoftWindowsServer";
             "sku"       = "2016-Datacenter";
             "osType"    = "Windows";
             "version"   = "latest"
          };
          "size"                    = "Standard_DS2_v2";
          "userName"                = $UserName;
          "password"                = $Password;
          "disallowPublicIpAddress" = $true;
          "dataDiskParameters" = @(@{
            "attachNewDataDiskOptions" = @{
                "diskName" = "adddatadisk"
                "diskSizeGiB" = "1023"
                "diskType" = "Standard"
                }
          "hostCaching" = "ReadWrite"
          })
       }
    }
    
    #The following line is the same as invoking
    # https://azure.github.io/projects/apis/#!/Labs/Labs_CreateEnvironment rest api

    Invoke-AzResourceAction -ResourceId $lab.ResourceId -Action 'createEnvironment' -Parameters $parameters -ApiVersion $API_VERSION -Force -Verbose
}
finally {
   popd
}
```

Las propiedades de la máquina virtual en el script anterior nos permiten crear una máquina virtual con Windows Server 2016 DataCenter como sistema operativo. Para cada tipo de máquina virtual, estas propiedades serán ligeramente diferentes. La sección [Definición de la máquina virtual](#define-virtual-machine) muestra cómo determinar qué propiedades se van a usar en este script.

El siguiente comando proporciona un ejemplo de la ejecución del script guardado en un nombre de archivo: Create-LabVirtualMachine.ps1. 

```powershell
 PS> .\Create-LabVirtualMachine.ps1 -ResourceGroupName 'MyLabResourceGroup' -LabName 'MyLab' -userName 'AdminUser' -password 'Password1!' -VMName 'MyLabVM'
```

## <a name="define-virtual-machine"></a>Definición de la máquina virtual
En esta sección se muestra cómo obtener las propiedades que son específicas de un tipo de máquina virtual que desea crear. 

### <a name="use-azure-portal"></a>Usar Azure Portal
Puede generar una plantilla de Azure Resource Manager al crear una máquina virtual en Azure Portal. No es necesario completar el proceso de creación de la máquina virtual. Solo siga los pasos hasta que vea la plantilla. Esta es la mejor manera de obtener la descripción de JSON necesaria si aún no ha creado una máquina virtual para el laboratorio. 

1. Acceda a [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** en el menú de navegación izquierdo.
3. Seleccione **DevTest Labs** en la lista de servicios. 
4. En la página **DevTest Labs**, seleccione el laboratorio en la lista de laboratorios.
5. En la página de inicio de su laboratorio, seleccione **+ Agregar** en la barra de herramientas. 
6. Seleccione una **imagen base** para la máquina virtual. 
7. Seleccione **Opciones de automatización** en la parte inferior de la pantalla, encima del botón **Enviar**. 
8. Verá la **plantilla de Azure Resource Manager** para crear la máquina virtual. 
9. El segmento JSON de la sección de **recursos** tiene la definición para el tipo de imagen que seleccionó anteriormente. 

    ```json
    {
      "apiVersion": "2018-10-15-preview",
      "type": "Microsoft.DevTestLab/labs/virtualmachines",
      "name": "[variables('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "labVirtualNetworkId": "[variables('labVirtualNetworkId')]",
        "notes": "Windows Server 2019 Datacenter",
        "galleryImageReference": {
          "offer": "WindowsServer",
          "publisher": "MicrosoftWindowsServer",
          "sku": "2019-Datacenter",
          "osType": "Windows",
          "version": "latest"
        },
        "size": "[parameters('size')]",
        "userName": "[parameters('userName')]",
        "password": "[parameters('password')]",
        "isAuthenticationWithSshKey": false,
        "labSubnetName": "[variables('labSubnetName')]",
        "disallowPublicIpAddress": true,
        "storageType": "Standard",
        "allowClaim": false,
        "networkInterface": {
          "sharedPublicIpAddressConfiguration": {
            "inboundNatRules": [
              {
                "transportProtocol": "tcp",
                "backendPort": 3389
              }
            ]
          }
        }
      }
    }
    ```

En este ejemplo, ve cómo obtener una definición de una imagen de Azure Marketplace. Puede obtener una definición de una imagen personalizada, una fórmula o un entorno de la misma manera. Agregue los artefactos necesarios para la máquina virtual y establezca cualquier configuración avanzada que requiera. Después de proporcionar valores para los campos obligatorios, complete los campos opcionales antes de seleccionar el botón **Opciones de automatización**.

### <a name="use-azure-rest-api"></a>Uso de la API REST de Azure
En el procedimiento siguiente se proporcionan los pasos para obtener las propiedades de una imagen mediante la API de REST. Estos pasos solo funcionan en una máquina virtual existente en un laboratorio. 

1. Navegue hasta la página [Máquinas virtuales - Lista](/rest/api/dtl/virtualmachines/list) y seleccione el botón **Pruébelo**. 
2. Seleccione la **suscripción de Azure**.
3. Escriba el **grupo de recursos para el laboratorio**.
4. Escriba el **nombre del laboratorio**. 
5. Seleccione **Run** (Ejecutar).
6. Verá las **propiedades de la imagen** según la máquina virtual creada. 

## <a name="set-expiration-date"></a>Establecimiento de la fecha de expiración
En escenarios como aprendizaje, demostraciones y evaluaciones, es posible que desee crear máquinas virtuales y eliminarlas automáticamente después de una duración fija para no incurrir en costos innecesarios. Mediante PowerShell, puede establecer una fecha de expiración para una máquina virtual al crearla, tal como se muestra en la sección [Script de PowerShell](#powershell-script) de ejemplo.

Este es un script de PowerShell de ejemplo que establece la fecha de expiración de todas las máquinas virtuales existentes en un laboratorio:

```powershell
# Values to change
$subscriptionId = '<Enter the subscription Id that contains lab>'
$labResourceGroup = '<Enter the lab resource group>'
$labName = '<Enter the lab name>'
$VmName = '<Enter the VmName>'
$expirationDate = '<Enter the expiration date e.g. 2019-12-16>'

# Log into your Azure account
Login-AzureRmAccount

Select-AzureRmSubscription -SubscriptionId $subscriptionId
$VmResourceId = "subscriptions/$subscriptionId/resourcegroups/$labResourceGroup/providers/microsoft.devtestlab/labs/$labName/virtualmachines/$VmName"

$vm = Get-AzureRmResource -ResourceId $VmResourceId -ExpandProperties

# Get all the Vm properties
$VmProperties = $vm.Properties

# Set the expirationDate property
If ($VmProperties.expirationDate -eq $null) {
    $VmProperties | Add-Member -MemberType NoteProperty -Name expirationDate -Value $expirationDate
} Else {
    $VmProperties.expirationDate = $expirationDate
}

Set-AzureRmResource -ResourceId $VmResourceId -Properties $VmProperties -Force
```


## <a name="next-steps"></a>Pasos siguientes
Consulte el siguiente contenido: [Documentación de Azure PowerShell para Azure DevTest Labs](/powershell/module/az.devtestlabs/)
