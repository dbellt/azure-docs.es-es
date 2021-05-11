---
title: 'Eliminación de una puerta de enlace de red virtual: Azure clásico'
description: Obtenga información sobre cómo eliminar una puerta de enlace de red virtual mediante PowerShell en el modelo de implementación clásica.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: e26c317295081531f295042efa812f091f06814a
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108291089"
---
# <a name="delete-a-virtual-network-gateway-using-powershell-classic"></a>Eliminación de una puerta de enlace de red virtual mediante PowerShell (clásico)

> [!div class="op_single_selector"]
> * [Resource Manager: Azure Portal](vpn-gateway-delete-vnet-gateway-portal.md)
> * [Resource Manager - PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [Clásico: PowerShell](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>

Este artículo le ayuda a eliminar una puerta de enlace de VPN en el modelo de implementación clásica mediante PowerShell. Una vez que elimina la puerta de enlace de red virtual, modifique el archivo de configuración de red para quitar los elementos que ya no usa.

## <a name="step-1-connect-to-azure"></a><a name="connect"></a>Paso 1: Conexión con Azure

### <a name="1-install-the-latest-powershell-cmdlets"></a>1. Instale los cmdlets más recientes de PowerShell.

[!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

### <a name="2-connect-to-your-azure-account"></a>2. Conéctese a su cuenta de Azure.

Abra la consola de PowerShell con privilegios elevados y conéctela a su cuenta. Use el siguiente ejemplo para conectarse:

1. Abra la consola de PowerShell con privilegios elevados.
2. Conéctese a su cuenta. Use el siguiente ejemplo para conectarse:

   ```powershell
   Add-AzureAccount
   ```

## <a name="step-2-export-and-view-the-network-configuration-file"></a><a name="export"></a>Paso 2: Exportación y visualización del archivo de configuración de red

Cree un directorio en el equipo y, a continuación, exporte el archivo de configuración de red al directorio. Use este archivo para ver la información de la configuración actual y también para modificar la configuración de red.

En este ejemplo, se exporta el archivo de configuración de red a C:\AzureNet.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

Abra el archivo con un editor de texto y consulte el nombre de la red virtual clásica. Cuando crea una red virtual en Azure Portal, el nombre completo que Azure usa no aparece en el portal. Por ejemplo, una red virtual que parece llamarse "ClassicVNet1" en Azure Portal puede que tenga un nombre mucho más largo en el archivo de configuración de la red. El nombre podría ser similar al siguiente: "Group ClassicRG1 ClassicVNet1". Los nombres de las redes virtuales aparecen como **"VirtualNetworkSite name ="** . Use los nombres que aparecen en el archivo de configuración de red cuando ejecute los cmdlets de PowerShell.

## <a name="step-3-delete-the-virtual-network-gateway"></a><a name="delete"></a>Paso 3: Eliminación de la puerta de enlace de red virtual

Cuando elimina una puerta de enlace de red virtual, se desconectan todas las conexiones a la red virtual a través de la puerta de enlace. Si tiene clientes de P2S conectados a la red virtual, se desconectarán sin advertencia.

Este ejemplo elimina la puerta de enlace de red virtual. Asegúrese de usar el nombre completo de la red virtual como aparece en el archivo de configuración de red.

```powershell
Remove-AzureVNetGateway -VNetName "Group ClassicRG1 ClassicVNet1"
```

Si es correcto, el resultado mostrará lo siguiente:

```
Status : Successful
```

## <a name="step-4-modify-the-network-configuration-file"></a><a name="modify"></a>Paso 4: Modificación del archivo de configuración de red

Cuando elimina una puerta de enlace de red virtual, el cmdlet no modifica el archivo de configuración de red. Debe modificar el archivo para quitar los elementos que ya no se usan. Las secciones siguientes lo ayudan a modificar el archivo de configuración de red que descargó.

### <a name="local-network-site-references"></a><a name="lnsref"></a>Referencias de sitio de red local

Cuando quite la información de referencia del sitio, haga cambios en la configuración en **ConnectionsToLocalNetwork/LocalNetworkSiteRef**. Quitar una referencia a sitio local hace que Azure elimine un túnel. Según la configuración que creó, puede que no aparezca **LocalNetworkSiteRef**.

```
<Gateway>
   <ConnectionsToLocalNetwork>
     <LocalNetworkSiteRef name="D1BFC9CB_Site2">
       <Connection type="IPsec" />
     </LocalNetworkSiteRef>
   </ConnectionsToLocalNetwork>
 </Gateway>
```

Ejemplo:

```
<Gateway>
   <ConnectionsToLocalNetwork>
   </ConnectionsToLocalNetwork>
 </Gateway>
```

### <a name="local-network-sites"></a><a name="lns"></a>Sitios de red local

Quite los sitios locales que ya no usa. Según la configuración que haya creado, es posible que no aparezca el elemento **LocalNetworkSite** en la lista.

```
<LocalNetworkSites>
  <LocalNetworkSite name="Site1">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
  </LocalNetworkSite>
  <LocalNetworkSite name="Site3">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>57.179.18.164</VPNGatewayAddress>
  </LocalNetworkSite>
 </LocalNetworkSites>
```

En este ejemplo, solo quitamos Site3.

```
<LocalNetworkSites>
  <LocalNetworkSite name="Site1">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
  </LocalNetworkSite>
 </LocalNetworkSites>
```

### <a name="client-addresspool"></a><a name="clientaddresss"></a>Grupo de direcciones de cliente

Si tuviera una conexión P2S a la red virtual, tendría un elemento **VPNClientAddressPool**. Quite los grupos de direcciones de cliente que correspondan a la puerta de enlace de red virtual que eliminó.

```
<Gateway>
    <VPNClientAddressPool>
      <AddressPrefix>10.1.0.0/24</AddressPrefix>
    </VPNClientAddressPool>
  <ConnectionsToLocalNetwork />
 </Gateway>
```

Ejemplo:

```
<Gateway>
  <ConnectionsToLocalNetwork />
 </Gateway>
```

### <a name="gatewaysubnet"></a><a name="gwsub"></a>GatewaySubnet

Elimine el elemento **GatewaySubnet** que corresponde a la red virtual.

```
<Subnets>
   <Subnet name="FrontEnd">
     <AddressPrefix>10.11.0.0/24</AddressPrefix>
   </Subnet>
   <Subnet name="GatewaySubnet">
     <AddressPrefix>10.11.1.0/29</AddressPrefix>
   </Subnet>
 </Subnets>
```

Ejemplo:

```
<Subnets>
   <Subnet name="FrontEnd">
     <AddressPrefix>10.11.0.0/24</AddressPrefix>
   </Subnet>
 </Subnets>
```

## <a name="step-5-upload-the-network-configuration-file"></a><a name="upload"></a>Paso 5: Carga del archivo de configuración de red

Guarde los cambios y cargue el archivo de configuración de red en Azure. Asegúrese de cambiar la ruta de acceso según sea necesario para su entorno.

```powershell
Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
```

Si lo hace correctamente, el resultado será similar a este ejemplo:

```
OperationDescription        OperationId                      OperationStatus                                                
--------------------        -----------                      ---------------                                           
Set-AzureVNetConfig         e0ee6e66-9167-cfa7-a746-7casb9   Succeeded
```
