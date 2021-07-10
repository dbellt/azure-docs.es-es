---
title: Creación de un punto de conexión privado mediante Azure PowerShell
description: Creación de un punto de conexión privado para Azure Attestation mediante Azure PowerShell
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 03/26/2021
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bafefd37647ee9d1f880c076348e4d20bead41c8
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111754266"
---
# <a name="quickstart-create-a-private-endpoint-using-azure-powershell"></a>Inicio rápido: Creación de un punto de conexión privado mediante Azure PowerShell

Comience a trabajar con Azure Private Link usando un punto de conexión privado para conectarse de forma segura a Azure Attestation.

En este inicio rápido, creará un punto de conexión privado para Azure Attestation e implementará una máquina virtual para probar la conexión privada.  

> [!NOTE]
> La implementación actual solo incluye la opción de aprobación automática. Se debe agregar la suscripción a una lista de suscripciones permitidas para poder continuar con la creación de un punto de conexión privado. Póngase en contacto con el equipo de servicio o envíe una solicitud de soporte técnico de Azure en la [Página de soporte técnico de Azure](https://azure.microsoft.com/support/options/) antes de continuar con los pasos siguientes.

## <a name="prerequisites"></a>Requisitos previos

* Obtenga información sobre [Azure Private Link](../private-link/private-link-overview.md)
* [Configuración de Azure Attestation con Azure PowerShell](./quickstart-powershell.md)

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
## Create to your Azure account subscription and create a resource group in a desired location. ##
Connect-AzAccount
Set-AzSubscription "mySubscription"
$rg = "CreateAttestationPrivateLinkTutorial-rg"
$loc= "eastus"
New-AzResourceGroup -Name $rg -Location $loc
```

## <a name="create-a-virtual-network-and-bastion-host"></a>Creación de una red virtual y un host bastión

En esta sección, creará una red virtual, una subred y un host bastión. 

El host bastión se utilizará para conectarse de forma segura a la máquina virtual a fin de probar el punto de conexión privado.

Creación de una red virtual y un host bastión con:

* [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)
* [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress)
* [New-AzBastion](/powershell/module/az.network/new-azbastion)

```azurepowershell-interactive
## Create backend subnet config. ##
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name myBackendSubnet -AddressPrefix 10.0.0.0/24

## Create Azure Bastion subnet. ##
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig -Name AzureBastionSubnet -AddressPrefix 10.0.1.0/24

## Create the virtual network. ##
$vnet = New-AzVirtualNetwork -Name "myAttestationTutorialVNet" -ResourceGroupName $rg -Location $loc -AddressPrefix "10.0.0.0/16" -Subnet $subnetConfig, $bastsubnetConfig

## Create public IP address for bastion host. ##
$publicip = New-AzPublicIpAddress -Name "myBastionIP" -ResourceGroupName $rg -Location $loc -Sku "Standard" -AllocationMethod "Static"

## Create bastion host ##
New-AzBastion -ResourceGroupName $rg -Name "myBastion" -PublicIpAddress $publicip -VirtualNetwork $vnet
```

El host de Azure Bastion tarda unos minutos en implementarse.

## <a name="create-test-virtual-machine"></a>Creación de una máquina virtual de prueba

En esta sección, creará una máquina virtual que se utilizará para probar el punto de conexión privado.

Cree la máquina virtual con:

  * [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential)
  * [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) 
  * [New-AzVM](/powershell/module/az.compute/new-azvm)
  * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
  * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
  * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
  * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)


```azurepowershell-interactive
## Set credentials for server admin and password. ##
$cred = Get-Credential

## Command to create network interface for VM ##
$nicVM = New-AzNetworkInterface -Name "myNicVM" -ResourceGroupName $rg -Location $loc -Subnet $vnet.Subnets[0] 

## Create a virtual machine configuration.##
$vmConfig = New-AzVMConfig -VMName "myVM" -VMSize "Standard_DS1_v2" | Set-AzVMOperatingSystem -Windows -ComputerName "myVM" -Credential $cred | Set-AzVMSourceImage -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2019-Datacenter" -Version "latest" | Add-AzVMNetworkInterface -Id $nicVM.Id 

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig
```

## <a name="create-an-attestation-provider"></a>Creación de un proveedor de atestación

```azurepowershell-interactive
## Create an attestation provider ##
$attestationProviderName = "myattestationprovider"
$attestationProvider = New-AzAttestation -Name $attestationProviderName -ResourceGroupName $rg -Location $loc
$attestationProviderId = $attestationProvider.Id
```
## <a name="access-the-attestation-provider-from-local-machine"></a>Acceso al proveedor de atestación desde la máquina local ##
Escriba `nslookup <provider-name>.attest.azure.net`. Reemplace **\<provider-name>** por el nombre de la instancia del proveedor de atestación que ha creado en los pasos anteriores. 
```azurepowershell-interactive
## Access the attestation provider from local machine ##
nslookup myattestationprovider.eus.attest.azure.net

<# You'll receive a message similar to what is displayed below:

Server:  cdns01.comcast.net
Address:  2001:558:feed::1

Non-authoritative answer:
Name:    eus.service.attest.azure.net
Address:  20.62.219.160
Aliases:  myattestationprovider.eus.attest.azure.net
    attesteusatm.trafficmanager.net

#>
```

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección creará el punto de conexión privado y la conexión con:

* [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/New-AzPrivateLinkServiceConnection)
* [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint)

```azurepowershell-interactive
## Create private endpoint connection. ##
$privateEndpointConnection = New-AzPrivateLinkServiceConnection -Name "myConnection" -PrivateLinkServiceId $attestationProviderId -GroupID "Standard"

## Disable private endpoint network policy ##
$vnet.Subnets[0].PrivateEndpointNetworkPolicies = "Disabled" 
$vnet | Set-AzVirtualNetwork

## Create private endpoint
New-AzPrivateEndpoint  -ResourceGroupName $rg -Name "myPrivateEndpoint" -Location $loc -Subnet $vnet.Subnets[0] -PrivateLinkServiceConnection $privateEndpointConnection
```
## <a name="configure-the-private-dns-zone"></a>Configuración de la zona DNS privada

En esta sección creará y configurará la zona DNS privada con:

* [New-AzPrivateDnsZone](/powershell/module/az.privatedns/new-azprivatednszone)
* [New-AzPrivateDnsVirtualNetworkLink](/powershell/module/az.privatedns/new-azprivatednsvirtualnetworklink)
* [New-AzPrivateDnsZoneConfig](/powershell/module/az.network/new-azprivatednszoneconfig)
* [New-AzPrivateDnsZoneGroup](/powershell/module/az.network/new-azprivatednszonegroup)

```azurepowershell-interactive
## Create private dns zone. ##
$zone = New-AzPrivateDnsZone -ResourceGroupName $rg -Name "privatelink.attest.azure.net"

## Create dns network link. ##
$link = New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $rg -ZoneName "privatelink.attest.azure.net" -Name "myLink" -VirtualNetworkId $vnet.Id

## Create DNS configuration ##
$config = New-AzPrivateDnsZoneConfig -Name "privatelink.attest.azure.net" -PrivateDnsZoneId $zone.ResourceId

## Create DNS zone group. ##
New-AzPrivateDnsZoneGroup -ResourceGroupName $rg -PrivateEndpointName "myPrivateEndpoint" -Name "myZoneGroup" -PrivateDnsZoneConfig $config
```

## <a name="test-connectivity-to-private-endpoint"></a>Prueba de la conectividad con el punto de conexión privado

En esta sección, utilizará la máquina virtual creada en el paso anterior para conectarse al servidor SQL mediante el punto de conexión privado.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com) 
 
2. En el panel de navegación de la izquierda, seleccione **Grupos de recursos**.

3. Seleccione **CreateAttestationPrivateLinkTutorial-rg**.

4. Seleccione **myVM**.

5. En la página de información general para **myVM**, seleccione **Conectar** y, luego, **Bastion**.

6. Seleccione el botón **Usar bastión** azul.

7. Especifique el nombre de usuario y la contraseña proporcionados durante la creación de la máquina virtual.

8. Abra Windows PowerShell en el servidor después de conectarse.

9. Escriba `nslookup <provider-name>.attest.azure.net`. Reemplace **\<provider-name>** por el nombre de la instancia del proveedor de atestación que ha creado en los pasos anteriores:

    ```azurepowershell-interactive
    ## Access the attestation provider from local machine ##
    nslookup myattestationprovider.eus.attest.azure.net
    
    <# You'll receive a message similar to what is displayed below:
    
    Server:  cdns01.comcast.net
    Address:  2001:558:feed::1

    cdns01.comcast.net can't find myattestationprovider.eus.attest.azure.net: Non-existent domain
    
    #>
    
    ## Access the attestation provider from the VM created in the same virtual network as the private endpoint.   ##
    nslookup myattestationprovider.eus.attest.azure.net
    
    <# You'll receive a message similar to what is displayed below:
    
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    myattestationprovider.eastus.test.attest.azure.net
    
    #>
    ```
