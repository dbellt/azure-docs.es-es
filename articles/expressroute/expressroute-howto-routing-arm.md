---
title: 'Tutorial: Configuración del emparejamiento de un circuito ExpressRoute: Azure PowerShell'
description: En este tutorial se muestra cómo crear y administrar la configuración de enrutamiento de un circuito ExpressRoute en el modelo de implementación de Resource Manager mediante PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e65c021cf858a634fc7ee9466cd502257c613d52
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702478"
---
# <a name="tutorial-create-and-modify-peering-for-an-expressroute-circuit-using-powershell"></a>Tutorial: Creación y modificación del emparejamiento de un circuito ExpressRoute mediante PowerShell

En este tutorial se muestra cómo crear y administrar la configuración de enrutamiento de un circuito ExpressRoute en el modelo de implementación de Resource Manager mediante PowerShell. También puede comprobar el estado de los emparejamientos de un circuito ExpressRoute, así como el modo de actualizarlos o eliminarlos y desaprovisionarlos. Si quiere usar un método diferente para trabajar con el circuito, seleccione un artículo de la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [CLI de Azure](howto-routing-cli.md)
> * [Emparejamiento público](about-public-peering.md)
> * [Vídeo: pares privados](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Vídeo: emparejamiento de Microsoft](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-routing-classic.md)
> 

Estas instrucciones se aplican solo a los circuitos creados con proveedores de servicios que ofrecen servicios de conectividad de capa 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente, IPVPN, como MPLS), el mismo proveedor de conectividad configurará y administrará el enrutamiento.

> [!IMPORTANT]
> Actualmente no anunciamos los emparejamientos configurados por proveedores de servicios a través del Portal de administración de servicios. Se está trabajando para habilitar esta funcionalidad pronto. Antes de configurar los emparejamientos BGP, realice las comprobaciones pertinentes con su proveedor de servicios.
> 

Puede configurar el emparejamiento privado de Azure y de Microsoft para un circuito ExpressRoute (el emparejamiento público de Azure está en desuso para los nuevos circuitos). El emparejamiento se puede configurar en el orden que elija. Pero tiene que asegurarse de que completa cada configuración entre pares de una en una. Para más información sobre los emparejamientos y dominios de enrutamiento, vea [Dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md). Para saber más sobre el emparejamiento público, vea [Emparejamiento público de ExpressRoute](about-public-peering.md).

En este tutorial, obtendrá información sobre cómo:
> [!div class="checklist"]
> - Configurar, actualizar y eliminar el emparejamiento de Microsoft para un circuito
> - Configurar, actualizar y eliminar el emparejamiento privado de Azure para un circuito

## <a name="prerequisites"></a>Requisitos previos

* Asegúrese de revisar las páginas siguientes antes de comenzar la configuración:
    * [Requisitos previos](expressroute-prerequisites.md) 
    * [Requisitos de enrutamiento](expressroute-routing.md)
    * [Flujos de trabajo](expressroute-workflows.md)
* Tiene que tener un circuito ExpressRoute activo. Antes de proceder, siga las instrucciones para [Crear un circuito ExpressRoute](expressroute-howto-circuit-arm.md) y para que el proveedor de conectividad habilite el circuito. El circuito ExpressRoute debe estar en un estado habilitado y aprovisionado para que pueda ejecutar los cmdlets de este artículo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="microsoft-peering"></a><a name="msft"></a>Emparejamiento de Microsoft

Esta sección le ayuda a crear, obtener, actualizar y eliminar la configuración de emparejamiento de Microsoft para un circuito ExpressRoute.

> [!IMPORTANT]
> Se anunciarán todos los prefijos de servicio para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron antes del 1 de agosto de 2017, incluso si no se definen filtros de ruta. No se anunciará ningún prefijo para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron el 1 de agosto de 2017 o con posterioridad, hasta que se asocie un filtro de ruta al circuito. Para más información, consulte [Configuración de un filtro de ruta para el emparejamiento de Microsoft](how-to-routefilter-powershell.md).
> 

### <a name="to-create-microsoft-peering"></a>Creación del emparejamiento de Microsoft

1. Inicie sesión y seleccione su suscripción.

   Si ha instalado PowerShell localmente, inicie sesión. Puede omitir este paso si usa Azure Cloud Shell.

   ```azurepowershell
   Connect-AzAccount
   ```

   Seleccione la suscripción en la que desea crear un circuito ExpressRoute.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "<subscription ID>"
   ```
2. Crear un circuito ExpressRoute.

   Siga las instrucciones para crear un [circuito ExpressRoute](expressroute-howto-circuit-arm.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si el proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitar a su proveedor de conectividad que habilite para usted la configuración entre pares de Microsoft. No necesita seguir las instrucciones que aparecen en las secciones siguientes. Sin embargo, si el proveedor de conectividad no administra el enrutamiento automáticamente, después de crear el circuito, continúe con la configuración mediante los pasos siguientes. 

3. Compruebe el circuito ExpressRoute para asegurarse de que está aprovisionado y también habilitado. Utilice el ejemplo siguiente:

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   La respuesta es similar al siguiente ejemplo:

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"################################"
   ProvisioningState                : Succeeded
   Sku                              : {
                                       "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
   CircuitProvisioningState         : Enabled
   ServiceProviderProvisioningState : Provisioned
   ServiceProviderNotes             : 
   ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
4. Establezca la configuración del emparejamiento de Microsoft para el circuito. Asegúrese de que tiene la siguiente información antes de continuar.

   * Una subred /30 o /126 para el vínculo principal. El bloque de direcciones debe ser un prefijo IPv4 o IPv6 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Una subred /30 o /126 para el vínculo secundario. El bloque de direcciones debe ser un prefijo IPv4 o IPv6 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.
   * Prefijos anunciados: Proporcione una lista de todos los prefijos que planea anunciar en la sesión BGP. Se aceptan solo prefijos de direcciones IP públicas. Si tiene pensado enviar un conjunto de prefijos, puede enviar una lista separada por comas. Estos prefijos tienen que estar registrados a su nombre en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR). Las sesiones BGP de IPv4 requieren prefijos anunciados de IPv4 y las sesiones BGP de IPv6 BGP, prefijos anunciados de IPv6. 
   * Nombre del enrutamiento del Registro: puede especificar el RIR o TIR en el que están registrados el número AS y los prefijos.
   * Opcional:
     * Cliente ASN: Si anuncia prefijos que no están registrados en el número de AS de emparejamiento, puede especificar el número de AS en el que están registrados.
     * Un hash MD5, en caso de que haya decidido usarlo.

> [!IMPORTANT]
> Microsoft comprueba si los prefijos públicos anunciados y ASN del mismo nivel (o ASN de cliente) especificados se le han asignado en el registro de enrutamiento de Internet. Si obtiene los prefijos públicos de otra entidad y la asignación no se registra con el registro de enrutamiento, la validación automática no se completará y requerirá la validación manual. Si se produce un error en la validación automática, verá "AdvertisedPublicPrefixesState" como "Se necesita validación" en la salida del comando "Get-AzExpressRouteCircuitPeeringConfig" (vea "Obtención de detalles del emparejamiento de Microsoft" en la siguiente sección). 
> 
> Si ve el mensaje "Se necesita validación", recopile los documentos que muestren los prefijos públicos asignados a la organización por la entidad que aparece como propietaria de los prefijos en el registro de enrutamiento, y envíe estos documentos para su validación manual. Para hacerlo, abra una incidencia de soporte técnico. 
> 

   Use el ejemplo siguiente para configurar el emparejamiento de Microsoft para el circuito:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "123.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

   Add-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

### <a name="to-get-microsoft-peering-details"></a><a name="getmsft"></a>Obtención de detalles del emparejamiento de Microsoft

Puede obtener detalles sobre la configuración mediante el ejemplo siguiente:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

### <a name="to-update-microsoft-peering-configuration"></a><a name="updatemsft"></a>Actualización de la configuración de emparejamiento de Microsoft

Puede actualizar cualquier parte de la configuración mediante el ejemplo siguiente:

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "124.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="azure-private-peering"></a><a name="private"></a>Configuración entre pares privados de Azure

Esta sección le ayuda a crear, obtener, actualizar y eliminar la configuración de emparejamiento privado de Azure para un circuito ExpressRoute.

> [!IMPORTANT]
> La compatibilidad de IPv6 con el emparejamiento privado se encuentra actualmente en **versión preliminar pública**. 
> 
> 

### <a name="to-create-azure-private-peering"></a>Creación de un emparejamiento privado de Azure

1. Importe el módulo de PowerShell para ExpressRoute.

   Instale el instalador de PowerShell más reciente desde la [Galería de PowerShell](https://www.powershellgallery.com/). A continuación, importe los módulos de Azure Resource Manager en la sesión de PowerShell para poder usar los cmdlets de ExpressRoute. Deberá ejecutar PowerShell como administrador.

   ```azurepowershell-interactive
   Install-Module Az
   ```

   Importe todos los módulos de Az.\* dentro del intervalo de versiones semánticas conocidas.

   ```azurepowershell-interactive
   Import-Module Az
   ```

   También puede importar un módulo determinado en dicho intervalo.

   ```azurepowershell-interactive
   Import-Module Az.Network 
   ```

   Inicie sesión en su cuenta.

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

   Seleccione la suscripción en la que desea crear un circuito ExpressRoute.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "<subscription ID>"
   ```
2. Crear un circuito ExpressRoute.

   Siga las instrucciones para crear un [circuito ExpressRoute](expressroute-howto-circuit-arm.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si su proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitarle que habilite la configuración entre pares privados de Azure. No necesita seguir las instrucciones que aparecen en las secciones siguientes. Sin embargo, si el proveedor de conectividad no administra el enrutamiento automáticamente, después de crear el circuito, continúe con la configuración mediante los pasos siguientes.

3. Compruebe el circuito ExpressRoute para asegurarse de que está aprovisionado y también habilitado. Utilice el ejemplo siguiente:

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   La respuesta es similar al siguiente ejemplo:

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"################################"
   ProvisioningState                : Succeeded
   Sku                              : {
                                       "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
   CircuitProvisioningState         : Enabled
   ServiceProviderProvisioningState : Provisioned
   ServiceProviderNotes             : 
   ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
4. Establecimiento de la configuración entre pares privados de Azure para el circuito. Asegúrese de que tiene los elementos siguientes antes de continuar con los siguientes pasos:

   * Un par de subredes que no forman parte de ningún espacio de direcciones reservado para redes virtuales. Una subred se usará para el vínculo principal, mientras que la otra se usará para el vínculo secundario. Desde cada una de estas subredes, asignará la primera dirección IP utilizable para el enrutador, ya que Microsoft usa la segunda dirección IP utilizable para su enrutador. Tiene tres opciones para este par de subredes:
       * IPv4: dos subredes /30.
       * IPv6: dos subredes /126.
       * Las dos: dos subredes /30 y dos subredes /126.
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS. Puede usar un número AS privado para esta configuración entre pares. Asegúrese de que no está usando 65515.
   * Opcional:
     * Un hash MD5, en caso de que haya decidido usarlo.

   Use el ejemplo siguiente para configurar el emparejamiento privado de Azure para su circuito:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 200 -PeerAddressType IPv6

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   Si decide usar un hash MD5, use el ejemplo siguiente:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200  -SharedKey "A1B2C3D4"

   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 200 -PeerAddressType IPv6 -SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Asegúrese de especificar su número AS como ASN de configuración entre pares, no como cliente ASN.
   > 
   >

### <a name="to-get-azure-private-peering-details"></a><a name="getprivate"></a>Obtención de los detalles del emparejamiento privado

Puede obtener detalles sobre la configuración mediante el ejemplo siguiente:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

### <a name="to-update-azure-private-peering-configuration"></a><a name="updateprivate"></a>Actualización del establecimiento de configuración del emparejamiento privado de Azure

Puede actualizar cualquier parte de la configuración mediante el ejemplo siguiente. En este ejemplo, se va a actualizar el identificador de VLAN del circuito de 200 a 500.

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 500

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="to-delete-microsoft-peering"></a><a name="deletemsft"></a>Eliminación del emparejamiento de Microsoft

Puede quitar la configuración de emparejamiento ejecutando el siguiente cmdlet:

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-azure-private-peering"></a><a name="deleteprivate"></a>Eliminación del emparejamiento privado de Azure

Puede quitar la configuración de emparejamiento mediante la ejecución del ejemplo siguiente:

> [!WARNING]
> Debe asegurarse de que todas las conexiones de redes virtuales y de ExpressRoute Global Reach se eliminan antes de ejecutar este ejemplo. 
> 

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado el emparejamiento privado de Azure, puede crear una puerta de enlace de ExpressRoute para vincular una red virtual al circuito. 

> [!div class="nextstepaction"]
> [Configuración de una puerta de enlace de red virtual para ExpressRoute](expressroute-howto-add-gateway-resource-manager.md)
