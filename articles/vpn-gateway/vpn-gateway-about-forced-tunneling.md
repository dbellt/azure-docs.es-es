---
title: 'configuración de la tunelización forzada para conexiones de sitio a sitio: clásica'
titleSuffix: Azure VPN Gateway
description: Aprenda a configurar la tunelización forzada para las redes virtuales creadas mediante el modelo de implementación clásica.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 10/15/2020
ms.author: cherylmc
ms.openlocfilehash: 61f42ac26f5890ff8ab611e81406d3607b8d5005
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108292169"
---
# <a name="configure-forced-tunneling-using-the-classic-deployment-model"></a>Configuración de la tunelización forzada mediante el modelo de implementación clásica

La tunelización forzada permite redirigir o forzar todo el tráfico vinculado a Internet de vuelta a su ubicación local a través de un túnel VPN de sitio a sitio con fines de inspección y auditoría. Se trata de un requisito de seguridad crítico en la mayoría de las directivas de las empresas de TI. Sin la tunelización forzada, el tráfico vinculado a Internet desde las máquinas virtuales en Azure siempre irá desde la infraestructura de red de Azure directamente a Internet, sin la opción que permite inspeccionar o auditar el tráfico. Un acceso no autorizado a Internet puede provocar la divulgación de información u otros tipos de infracciones de seguridad.

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

Este artículo le guiará a través del proceso de configuración de la tunelización forzada para redes virtuales creadas mediante el modelo de implementación clásica. La tunelización forzada puede configurarse mediante el uso de PowerShell, no a través del portal. Si quiere configurar la tunelización forzada para el modelo de implementación de Resource Manager, seleccione el artículo de Resource Manager de la siguiente lista desplegable:

> [!div class="op_single_selector"]
> * [Clásico](vpn-gateway-about-forced-tunneling.md)
> * [Resource Manager](vpn-gateway-forced-tunneling-rm.md)
> 

## <a name="requirements-and-considerations"></a>Requisitos y consideraciones

La tunelización forzada en Azure se configura a través de rutas definidas por el usuario (UDR) de redes virtuales. La redirección del tráfico a un sitio local se expresa como una ruta predeterminada a la puerta de enlace de VPN de Azure. La sección siguiente muestra la limitación actual de la tabla de enrutamiento y las rutas de una instancia de Azure Virtual Network:

* Cada subred de la red virtual tiene una tabla de enrutamiento del sistema integrada. La tabla de enrutamiento del sistema tiene los siguientes tres grupos de rutas:

  * **Rutas de redes virtuales locales:** directamente a las máquinas virtuales de destino en la misma red virtual.
  * **Rutas locales:** a Azure VPN Gateway.
  * **Ruta predeterminada:** directamente a Internet. Los paquetes destinados a las direcciones IP privadas que no están cubiertos por las dos rutas anteriores se anularán.
* Con la liberación de las rutas definidas por el usuario, puede crear una tabla de enrutamiento para agregar una ruta predeterminada y, después, asociar la tabla de enrutamiento a las subredes de la red virtual para habilitar la tunelización forzada en esas subredes.
* Deberá establecer un "sitio predeterminado" entre los sitios locales entre entornos conectados a la red virtual.
* La tunelización forzada debe asociarse a una red virtual que tiene una puerta de enlace de VPN de enrutamiento dinámico (no una puerta de enlace estática).
* La tunelización forzada ExpressRoute no se configura mediante este mecanismo, sino que se habilita mediante el anuncio de una ruta predeterminada a través de las sesiones de emparejamiento BGP de ExpressRoute. Para más información, consulte [¿Qué es ExpressRoute?](../expressroute/expressroute-introduction.md)

## <a name="configuration-overview"></a>Información general sobre la configuración

En el ejemplo siguiente, la subred Frontend no usa la tunelización forzada. Las cargas de trabajo de la subred Frontend pueden continuar para aceptar y responder a las solicitudes de los clientes directamente desde Internet. Las subredes Mid-tier y Backend usan la tunelización forzada. Las conexiones salientes desde estas dos subredes a Internet se fuerzan o redirigen a un sitio local a través de uno de los túneles VPN de sitio a sitio.

Esto permite restringir e inspeccionar el acceso a Internet desde sus máquinas virtuales o servicios en la nube en Azure, al tiempo que continúa posibilitando la arquitectura de varios niveles de servicio necesaria. También tiene la opción de aplicar la tunelización forzada a redes virtuales completas si no hay ninguna carga de trabajo a través de Internet en las redes virtuales.

![Tunelización forzada](./media/vpn-gateway-about-forced-tunneling/forced-tunnel.png)

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar con la configuración, verifique que dispone de los elementos siguientes:

* Suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Una red virtual configurada. 
* [!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="configure-forced-tunneling"></a>Configuración de la tunelización forzada

El siguiente procedimiento lo ayudará a especificar la tunelización forzada en una red virtual. Los pasos de configuración corresponden al archivo de configuración de red virtual.  En este ejemplo, la red virtual "MultiTier-VNet" tiene tres subredes: las subredes "Frontend", "Midtier" y "Backend", con cuatro conexiones entre locales: "DefaultSiteHQ" y tres ramas.


```xml
<VirtualNetworkSite name="MultiTier-VNet" Location="North Europe">
     <AddressSpace>
      <AddressPrefix>10.1.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Frontend">
            <AddressPrefix>10.1.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Midtier">
            <AddressPrefix>10.1.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Backend">
            <AddressPrefix>10.1.2.0/23</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.1.200.0/28</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="DefaultSiteHQ">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch1">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch2">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch3">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSite>
```

Los siguientes pasos establecerán "DefaultSiteHQ" como la conexión de sitio predeterminada para la tunelización forzada y configurarán las subredes Midtier y Backend para que usen dicha tunelización.

1. Abra la consola de PowerShell con privilegios elevados. Conéctese a su cuenta mediante el ejemplo siguiente:

   ```powershell
   Add-AzureAccount
   ```

1. Cree una tabla de enrutamiento. Use el siguiente cmdlet para crear la tabla de enrutamiento.

   ```powershell
   New-AzureRouteTable –Name "MyRouteTable" –Label "Routing Table for Forced Tunneling" –Location "North Europe"
   ```

1. Agregue una ruta predeterminada a la tabla de enrutamiento. 

   El siguiente ejemplo agrega una ruta predeterminada a la tabla de enrutamiento que creó en el paso 1. Tenga en cuenta que la única ruta admitida es el prefijo de destino de 0.0.0.0/0 para el próximo salto VPNGateway.

   ```powershell
   Get-AzureRouteTable -Name "MyRouteTable" | Set-AzureRoute –RouteTable "MyRouteTable" –RouteName "DefaultRoute" –AddressPrefix "0.0.0.0/0" –NextHopType VPNGateway
   ```

1. Asocie la tabla de enrutamiento a las subredes. 

   Una vez creada una tabla de enrutamiento y agregada una ruta, use el ejemplo siguiente para agregar o asociar la tabla de enrutamiento a una subred de la red virtual. Los siguientes ejemplos agregan la tabla de enrutamiento MyRouteTable a las subredes Midtier y Backend de VNet MultiTier-VNet.

   ```powershell
   Set-AzureSubnetRouteTable -VirtualNetworkName "MultiTier-VNet" -SubnetName "Midtier" -RouteTableName "MyRouteTable"
   Set-AzureSubnetRouteTable -VirtualNetworkName "MultiTier-VNet" -SubnetName "Backend" -RouteTableName "MyRouteTable"
   ```

1. Asigne un sitio predeterminado para la tunelización forzada. 

   En el paso anterior, los scripts del cmdlet de ejemplo crean la tabla de enrutamiento y la tabla de enrutamiento asociada a dos de las subredes de la red virtual. El paso restante consiste en seleccionar un sitio local entre las conexiones de varios sitios de la red virtual como el sitio predeterminado o túnel.

   ```powershell
   $DefaultSite = @("DefaultSiteHQ")
   Set-AzureVNetGatewayDefaultSite –VNetName "MultiTier-VNet" –DefaultSite "DefaultSiteHQ"
   ```

## <a name="additional-powershell-cmdlets"></a>Cmdlets de PowerShell adicionales

### <a name="to-delete-a-route-table"></a>Para eliminar una tabla de enrutamiento

```powershell
Remove-AzureRouteTable -Name <routeTableName>
```
  
### <a name="to-list-a-route-table"></a>Para mostrar una tabla de enrutamiento

```powershell
Get-AzureRouteTable [-Name <routeTableName> [-DetailLevel <detailLevel>]]
```

### <a name="to-delete-a-route-from-a-route-table"></a>Para eliminar una ruta de una tabla de enrutamiento

```powershell
Remove-AzureRouteTable –Name <routeTableName>
```

### <a name="to-remove-a-route-from-a-subnet"></a>Para quitar una ruta de una subred

```powershell
Remove-AzureSubnetRouteTable –VirtualNetworkName <virtualNetworkName> -SubnetName <subnetName>
```

### <a name="to-list-the-route-table-associated-with-a-subnet"></a>Para mostrar la tabla de enrutamiento asociada a una subred

```powershell
Get-AzureSubnetRouteTable -VirtualNetworkName <virtualNetworkName> -SubnetName <subnetName>
```

### <a name="to-remove-a-default-site-from-a-vnet-vpn-gateway"></a>Para quitar un sitio predeterminado de una puerta de enlace de VPN de VNet

```powershell
Remove-AzureVnetGatewayDefaultSite -VNetName <virtualNetworkName>
```
