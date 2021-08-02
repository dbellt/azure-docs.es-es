---
title: Conexión de máquinas virtuales a SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Obtenga información sobre cómo conectar máquinas virtuales a SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 6/1/2021
ms.author: madhukan
ms.custom: H1Hack27Feb2017, devx-track-azurepowershell
ms.openlocfilehash: e451f969c1518a920f7828d92fdfed65cd80f69c
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111412392"
---
# <a name="connect-a-virtual-network-to-hana-large-instances"></a>Conexión de una red virtual a HANA (instancias grandes)

Ha [creado una red virtual de Azure](hana-connect-azure-vm-large-instances.md). Ahora puede conectar esa red a SAP HANA (instancias grandes) (también conocidas como instancias de BareMetal Infrastructure). En este artículo, veremos los pasos que debe seguir.

## <a name="create-an-azure-expressroute-gateway-on-the-virtual-network"></a>Cree una puerta de enlace de Azure ExpressRoute en la red virtual.

En primer lugar, cree una puerta de enlace de ExpressRoute de Azure en la red virtual. Esta puerta de enlace le permite vincular la red virtual al circuito ExpressRoute que se conecta al inquilino en el sello de HANA (instancias grandes).

> [!NOTE] 
> Este paso puede tardar hasta 30 minutos en completarse. Cree la nueva puerta de enlace en la suscripción de Azure designada y, después, conéctela a la red virtual de Azure especificada.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

- Si ya existe una puerta de enlace, compruebe si se trata de una puerta de enlace de ExpressRoute. Si no es una puerta de enlace de ExpressRoute, elimínela y vuelva a crearla como una puerta de enlace de ExpressRoute. Si ya hay una puerta de enlace de ExpressRoute establecida, pase a la sección siguiente de este artículo, [Vinculación de redes virtuales](#link-virtual-networks). 

- Use [Azure Portal](https://portal.azure.com/) o PowerShell para crear una puerta de enlace VPN de ExpressRoute conectada a la red virtual.
    - Si usa Azure Portal, agregue una nueva **puerta de enlace de red virtual** y seleccione **ExpressRoute** como tipo de puerta de enlace.
    - Si usa PowerShell, descargue y use la versión más reciente del [SDK de Azure PowerShell](https://azure.microsoft.com/downloads/). 
 
    Los siguientes comandos crean una puerta de enlace de ExpressRoute. Los textos precedidos por un signo _$_ son variables definidas por el usuario que se deben actualizar con la información específica.

    ```powershell
    # These Values should already exist, update to match your environment
    $myAzureRegion = "eastus"
    $myGroupName = "SAP-East-Coast"
    $myVNetName = "VNet01"
    
    # These values are used to create the gateway, update for how you wish the GW components to be named
    $myGWName = "VNet01GW"
    $myGWConfig = "VNet01GWConfig"
    $myGWPIPName = "VNet01GWPIP"
    $myGWSku = "UltraPerformance" # Supported values for HANA large instances are: UltraPerformance
    
    # These Commands create the Public IP and ExpressRoute Gateway
    $vnet = Get-AzVirtualNetwork -Name $myVNetName -ResourceGroupName $myGroupName
    $subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
    New-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName `
    -Location $myAzureRegion -AllocationMethod Dynamic
    $gwpip = Get-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName
    $gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name $myGWConfig -SubnetId $subnet.Id `
    -PublicIpAddressId $gwpip.Id
    
    New-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName -Location $myAzureRegion `
    -IpConfigurations $gwipconfig -GatewayType ExpressRoute `
    -GatewaySku $myGWSku -VpnType PolicyBased -EnableBgp $true
    ```

La única SKU de puerta de enlace compatible para SAP HANA en Azure (instancias grandes) es **UltraPerformance**.

## <a name="link-virtual-networks"></a>Vinculación de redes virtuales

La red virtual de Azure ahora tiene una puerta de enlace de ExpressRoute. Use la información de autorización que facilita Microsoft para conectar la puerta de enlace de ExpressRoute al circuito de ExpressRoute de SAP HANA (Instancias grandes). Puede usar Azure Portal o PowerShell para realizar la conexión. Las instrucciones de PowerShell son las siguientes. 

Ejecute los siguientes comandos para cada puerta de enlace de ExpressRoute con un AuthGUID diferente para cada conexión. Las dos primeras entradas mostradas en el siguiente script proceden de la información proporcionada por Microsoft. Además, el AuthGUID es específico de cada red virtual y su puerta de enlace. Si quiere agregar otra red virtual de Azure, tiene que obtener otro AuthID para el circuito ExpressRoute que conecta HANA (instancias grandes) con Azure desde Microsoft. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

> [!NOTE]
> El último parámetro del comando New-AzVirtualNetworkGatewayConnection, **ExpressRouteGatewayBypass**, es un nuevo parámetro que habilita ExpressRoute FastPath. Esta funcionalidad se agregó en mayo de 2019 y reduce la latencia de red entre las unidades de HANA (instancias grandes) y las máquinas virtuales de Azure. Para obtener más información, consulte [Arquitectura de red de SAP HANA (instancias grandes)](./hana-network-architecture.md). Antes de ejecutar los comandos, asegúrese de que está ejecutando la versión más reciente de los cmdlets de PowerShell.

Es posible que tenga que conectar la puerta de enlace a más de un circuito ExpressRoute asociado a su suscripción. En ese caso, deberá ejecutar este paso más de una vez. Por ejemplo, probablemente va a conectar la misma puerta de enlace de la red virtual al circuito de ExpressRoute que conecta la red virtual a la red local.

## <a name="applying-expressroute-fastpath-to-existing-hana-large-instance-expressroute-circuits"></a>Aplicación de ExpressRoute FastPath a circuitos de ExpressRoute de instancias grandes de HANA existentes
Ha visto cómo conectar un nuevo circuito ExpressRoute creado con una implementación de instancia grande de HANA a una puerta de enlace de Azure ExpressRoute en una de las redes virtuales de Azure. Pero, ¿qué ocurre si ya tiene los circuitos de ExpressRoute configurados y las redes virtuales ya están conectadas a HANA (instancias grandes)? 

El nuevo ExpressRoute FastPath reduce la latencia de red. Se recomienda aplicar el cambio para aprovechar esta latencia reducida. Los comandos para conectar un nuevo circuito de ExpressRoute son los mismos que para cambiar un circuito de ExpressRoute existente. Por tanto, es necesario ejecutar esta secuencia de comandos de PowerShell para cambiar un circuito existente. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

Para habilitar la funcionalidad ExpressRoute FastPath, es importante agregar el último parámetro como se mostró anteriormente.


## <a name="expressroute-global-reach"></a>Global Reach de ExpressRoute

Habilite Global Reach para cualquiera de los escenarios siguientes:

 - La replicación del sistema HANA sin firewalls ni proxies adicionales.
 - La copia de seguridad entre las unidades de HANA (instancias grandes) en dos regiones diferentes para hacer copias o actualizaciones del sistema.

Para habilitar Global Reach:

- Proporcione un intervalo de espacio de direcciones /29. Es posible que ese intervalo de direcciones no se superponga con ninguno de los otros intervalos de espacio de direcciones que usó hasta ahora para conectar HANA (instancias grandes) a Azure. El intervalo de direcciones tampoco debe superponerse con ninguno de los intervalos de direcciones IP que usó en otra parte de Azure o en el entorno local.
- Hay un límite en los números de sistema autónomo que se pueden usar para anunciar las rutas locales en HANA (instancias grandes). El entorno local no debe anunciar ninguna ruta con ASN privados que se encuentren dentro del intervalo 65000 - 65020 o 65515. 
- Cuando se conecta el acceso directo local a instancias grandes de HANA, debe calcular una tarifa para el circuito que le conecta a Azure. Para información sobre los precios, consulte el [complemento Global Reach](https://azure.microsoft.com/pricing/details/expressroute/).

Para aplicar uno o ambos escenarios a la implementación, abra un mensaje de soporte técnico con Azure como se describe en [Abrir una solicitud de soporte técnico para HANA (instancias grandes)](./hana-li-portal.md#open-a-support-request-for-hana-large-instances).

Los datos y las palabras clave que necesitará usar para Microsoft para enrutar y ejecutar la solicitud son los siguientes:

- Servicio: Instancia grande de SAP HANA
- Tipo de problema: Instalación y configuración
- Subtipo de problema: Mi problema no aparece aquí.
- Asunto "Modificar mi red: agregar Global Reach"
- Detalles: "Agregar Global Reach al inquilino de HANA (instancias grandes)". o "Agregar Global Reach al entorno local al inquilino de HANA (instancias grandes)".
- Detalles adicionales para el caso de instancia grande de HANA al inquilino de instancia grande de HANA: Debe definir las **dos regiones de Azure** donde se encuentran los dos inquilinos a los que quiere conectarse **Y** debe enviar el **intervalo de direcciones IP /29**.
- Detalles adicionales del caso de entorno local a inquilino de instancia grande de HANA: 
    - Defina la **región de Azure** donde se implementa el inquilino de HANA (instancias grandes) al que quiere conectarse directamente. 
    - Proporcione **AuthGUID** y el **identificador del mismo nivel de circuito** que recibió al establecer el circuito ExpressRoute entre el entorno local y Azure. 
    - Asigne un nombre al **ASN**. 
    - Proporcione un **intervalo de direcciones IP /29** para Global Reach de ExpressRoute.

> [!NOTE]
> Si quiere gestionar ambos casos, debe proporcionar dos intervalos de direcciones IP /29 diferentes que no se superpongan con ningún otro intervalo de direcciones IP usado hasta el momento. 

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre otros requisitos de red que puede tener para implementar SAP HANA (instancias grandes) en Azure.

> [!div class="nextstepaction"]
> [Requisitos de red adicionales para instancias grandes](hana-additional-network-requirements.md)
