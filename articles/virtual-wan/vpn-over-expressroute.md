---
title: 'Configuración del cifrado de ExpressRoute: IPsec sobre ExpressRoute para Azure Virtual WAN'
description: Aprenda a usar Azure Virtual WAN para crear una conexión VPN de sitio a sitio mediante el emparejamiento privado de ExpressRoute.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c74468620b61c0bff01acc0012958be71d5f3786
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704703"
---
# <a name="expressroute-encryption-ipsec-over-expressroute-for-virtual-wan"></a>Cifrado de ExpressRoute: IPsec sobre ExpressRoute para Virtual WAN

En este artículo se muestra cómo usar Azure Virtual WAN para establecer una conexión VPN de IPsec/IKE desde la red local a Azure a través del emparejamiento privado de un circuito Azure ExpressRoute. Esta técnica puede proporcionar un tránsito cifrado entre las redes locales y las redes virtuales de Azure a través de ExpressRoute sin necesidad de pasar por la red pública de Internet ni utilizar direcciones IP públicas.

## <a name="topology-and-routing"></a>Topología y enrutamiento

En el diagrama siguiente se muestra un ejemplo de conectividad de VPN a través del emparejamiento privado de ExpressRoute:

:::image type="content" source="./media/vpn-over-expressroute/vwan-vpn-over-er.png" alt-text="VPN a través de ExpressRoute":::

El diagrama muestra una red dentro de la red local conectada a la puerta de enlace de VPN del centro de conectividad de Azure a través del emparejamiento privado de ExpressRoute. El establecimiento de la conectividad es sencillo:

1. Establezca la conectividad de ExpressRoute con un circuito ExpressRoute y emparejamiento privado.
2. Establezca la conectividad VPN como se describe en este artículo.

Un aspecto importante de esta configuración es el enrutamiento entre las redes locales y Azure a través de las rutas de acceso de ExpressRoute y VPN.

### <a name="traffic-from-on-premises-networks-to-azure"></a>Tráfico desde redes locales a Azure

En el caso del tráfico entre las redes locales y Azure, los prefijos de Azure (lo que incluye el centro de conectividad virtual y todas las redes virtuales de radio conectadas a ese centro) se anuncian a través de los protocolos de puerta de enlace de borde del emparejamiento privado de ExpressRoute y de la red privada virtual. El resultado son dos rutas de red (rutas de acceso) hacia Azure desde las redes locales:

- Una sobre la ruta de acceso protegida mediante IPsec
- Otra directamente sobre ExpressRoute, *sin* protección de IPsec 

Para aplicar el cifrado a la comunicación, debe asegurarse de que para la red conectada a VPN del diagrama, se prefieren las rutas de Azure a través de la puerta de enlace de VPN local a la ruta de acceso directa de ExpressRoute.

### <a name="traffic-from-azure-to-on-premises-networks"></a>Tráfico desde Azure a las redes locales

El mismo requisito se aplica al tráfico de Azure a las redes locales. Para asegurarse de que la ruta de acceso de IPsec se elige antes que la ruta de acceso directa de ExpressRoute (sin IPsec), tiene dos opciones:

- Anuncie prefijos más específicos en la sesión BGP de VPN para la red conectada a VPN. Puede anunciar un rango mayor que abarque la red conectada a VPN a través del emparejamiento privado de ExpressRoute y, después, rangos más específicos en la sesión del protocolo de puerta de enlace de borde de VPN. Por ejemplo, anuncie 10.0.0.0/16 a través de ExpressRoute y 10.0.1.0/24 a través de VPN.

- Anuncie prefijos disjuntos para VPN y ExpressRoute. Si los rangos de redes conectadas a VPN no están unidos a otras redes conectadas de ExpressRoute, puede anunciar los prefijos en las sesiones de los protocolos de puerta de enlace de borde de ExpressRoute y VPN, respectivamente. Por ejemplo, anuncie 10.0.0.0/24 a través de ExpressRoute y 10.0.1.0/24 a través de VPN.

En ambos ejemplos, Azure enviará tráfico a 10.0.1.0/24 a través de la conexión VPN en lugar de hacerlo directamente a través de ExpressRoute sin protección VPN.

> [!WARNING]
> Si anuncia los *mismos* prefijos a través de las conexiones de ExpressRoute y VPN, Azure usará la ruta de acceso de ExpressRoute directamente sin protección de VPN.
>

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

## <a name="1-create-a-virtual-wan-and-hub-with-gateways"></a><a name="openvwan"></a>1. Creación de una red virtual WAN y un concentrador con puertas de enlace

Antes de continuar deben estar en vigor los siguientes recursos de Azure y las configuraciones locales correspondientes:

- Una Azure Virtual WAN
- Un concentrador WAN virtual con una [puerta de enlace de ExpressRoute](virtual-wan-expressroute-portal.md) y una [puerta de enlace de VPN](virtual-wan-site-to-site-portal.md)

Para conocer los pasos que hay que seguir para crear una red de área extensa virtual de Azure y un centro de conectividad con una asociación de ExpressRoute, consulte [Creación de una asociación de ExpressRoute mediante Azure Virtual WAN](virtual-wan-expressroute-portal.md). Para conocer los pasos que deben seguirse para crear una puerta de enlace de VPN en la red de área extensa virtual, consulte [Creación de una conexión de sitio a sitio mediante Azure Virtual WAN](virtual-wan-site-to-site-portal.md).

## <a name="2-create-a-site-for-the-on-premises-network"></a><a name="site"></a>2. Creación de un sitio para la red local

El recurso del sitio es el mismo que el de los sitios VPN que no son de ExpressRoute para una red de área extensa virtual. La dirección IP del dispositivo VPN local ahora puede ser una dirección IP privada o una dirección IP pública de la red local a la que se puede acceder mediante el emparejamiento privado de ExpressRoute creado en el paso 1.

> [!NOTE]
> La dirección IP del dispositivo VPN local *debe* formar parte de los prefijos de dirección anunciados en el centro de conectividad de Virtual WAN a través del emparejamiento privado de ExpressRoute de Azure.
>

1. Vaya a Azure Portal en el explorador. 
1. Seleccione el centro que ha creado. En la página del centro de la WAN virtual, en **Conectividad**, seleccione **Sitios VPN**.
1. En la página **Sitios de VPN**, seleccione **+Crear sitio**.
1. En la página **Crear sitio** rellene los campos siguientes:
   * **Suscripción**: compruebe la suscripción.
   * **Grupo de recursos**: seleccione o cree el grupo de recursos que desea usar.
   * **Región**: escriba la región de Azure del recurso del sitio VPN.
   * **Name**: escriba el nombre que desee usar para hacer referencia a su sitio local.
   * **Proveedor del dispositivo**: escriba el proveedor del dispositivo VPN local.
   * **Protocolo de puerta de enlace de borde**: seleccione "Habilitar" si la red local usa el protocolo de puerta de enlace de borde.
   * **Espacio de direcciones privadas**: escriba el espacio de direcciones IP que se encuentra en el sitio local. El tráfico destinado a este espacio de direcciones se enruta a la red local a través de la puerta de enlace de VPN.
   * **Centros de conectividad**: seleccione uno o varios centros de conectividad para conectar este sitio VPN. Los concentradores seleccionados deben tener puertas de enlace de VPN ya creadas.
1. Seleccione **Siguiente: Vínculos >** para la configuración de vínculo de VPN:
   * **Nombre del vínculo**: el nombre que desee usar para hacer referencia a esta conexión.
   * **Nombre de proveedor**: el nombre del proveedor de acceso a Internet de este sitio. En el caso de la red local de ExpressRoute, es el nombre del proveedor de servicios de ExpressRoute.
   * **Velocidad**: la velocidad del vínculo de servicio de Internet o del circuito de ExpressRoute.
   * **Dirección IP**: la dirección IP pública del dispositivo VPN que reside en el sitio local. O bien, en el entorno local de ExpressRoute, es la dirección IP privada del dispositivo VPN a través de ExpressRoute.

   Si BGP está habilitado, se aplicará a todas las conexiones creadas para este sitio en Azure. La configuración del protocolo de puerta de enlace de borde en Virtual WAN es igual a su configuración en Azure VPN Gateway. 
   
   La dirección de su par BGP local *no debe* ser la misma que la dirección IP de la VPN al dispositivo o al espacio de direcciones de la red virtual del sitio VPN. Use otra dirección IP en el dispositivo VPN para la dirección IP del par BGP. Puede ser una dirección asignada a la interfaz de bucle invertido en el dispositivo. Sin embargo, *no puede* ser una dirección IP privada automática (169.254.*x*.*x*). Especifique esta dirección en el sitio VPN correspondiente que representa la ubicación. Para conocer los requisitos previos de BGP, consulte [Acerca de BGP con Azure VPN Gateway](../vpn-gateway/vpn-gateway-bgp-overview.md).

1. Seleccione **Siguiente: Revisar + Crear >** para comprobar los valores de configuración y crear el sitio VPN. Si seleccionó **Centros de conectividad** para conectarse, la conexión se establecerá entre la red local y la puerta de enlace de VPN del centro de conectividad.

## <a name="3-update-the-vpn-connection-setting-to-use-expressroute"></a><a name="hub"></a>3. Actualización de la configuración de conexión VPN para usar ExpressRoute

Después de crear el sitio VPN y conectarlo al centro de conectividad, use estos pasos para configurar la conexión para que use el emparejamiento privado de ExpressRoute:

1. Vuelva a la página de recursos de la red de área extensa virtual y seleccione el recurso del centro de conectividad. O bien, vaya desde el sitio VPN hasta el concentrador conectado.

   :::image type="content" source="./media/vpn-over-expressroute/hub-selection.png" alt-text="Selección de un centro":::
1. En **Conectividad**, seleccione **VPN (de sitio a sitio)**.

   :::image type="content" source="./media/vpn-over-expressroute/vpn-select.png" alt-text="Seleccionar VPN (sitio a sitio)":::
1. Seleccione los puntos suspensivos (**...**) en el sitio VPN sobre ExpressRoute y seleccione **Editar la conexión de VPN a este concentrador**.

   :::image type="content" source="./media/vpn-over-expressroute/config-menu.png" alt-text="Menú Especificar configuración":::
1. En **Usar dirección IP privada de Azure**, seleccione **Sí**. El ajuste configura la puerta de enlace de VPN de concentrador para que use direcciones IP privadas dentro del intervalo de direcciones del concentrador en la puerta de enlace para esta conexión, en lugar de las direcciones IP públicas. Esto garantizará que el tráfico de la red local atraviesa las rutas de acceso del emparejamiento privado de ExpressRoute, en lugar de usar la red pública de Internet para esta conexión VPN. En la captura de pantalla siguiente se muestra la configuración:

   :::image type="content" source="./media/vpn-over-expressroute/vpn-link-configuration.png" alt-text="Configuración para el uso de una dirección IP privada para la conexión VPN" border="false":::
1. Seleccione **Guardar**.

Después de guardar los cambios, la puerta de enlace de VPN del centro de conectividad usará las direcciones IP privadas en la puerta de enlace de VPN para establecer las conexiones IPsec/IKE con el dispositivo VPN local a través de ExpressRoute.

## <a name="4-get-the-private-ip-addresses-for-the-hub-vpn-gateway"></a><a name="associate"></a>4. Obtención de las direcciones IP privadas de puerta de enlace de VPN del centro de conectividad

Descargue la configuración del dispositivo VPN para obtener las direcciones IP privadas de la puerta de enlace de VPN del centro de conectividad. Estas direcciones son necesarias para configurar el dispositivo VPN local.

1. En la página del centro de conectividad, seleccione **VPN (sitio a sitio)** en **Conectividad**.
1. En la parte superior de la página de **información general**, seleccione **Descargar la configuración de VPN**. 

   Azure crea una cuenta de almacenamiento en el grupo de recursos "microsoft - network-[location]", donde *location* es la ubicación de la red de área extensa. Después de aplicar la configuración a los dispositivos VPN, puede eliminar esta cuenta de almacenamiento.
1. Después de que se cree el archivo, seleccione el vínculo para descargarlo.
1. Aplique la configuración al dispositivo VPN.

### <a name="vpn-device-configuration-file"></a>Archivo de configuración del dispositivo VPN

El archivo de configuración del dispositivo contiene la configuración que se debe usar al configurar el dispositivo VPN local. Cuando visualice este archivo, tenga en cuenta la siguiente información:

* **vpnSiteConfiguration**: en esta sección se indica la configuración de los detalles del dispositivo como un sitio que se conecta a la WAN virtual. Incluye el nombre y la dirección IP pública del dispositivo de rama.
* **vpnSiteConnections**: en esta sección se proporciona información sobre la siguiente configuración:

    * Espacio de direcciones de la red virtual del centro de conectividad virtual.<br/>Ejemplo:
           ```
           "AddressSpace":"10.51.230.0/24"
           ```
    * Espacio de direcciones de las redes virtuales que están conectadas al centro de conectividad.<br>Ejemplo:
           ```
           "ConnectedSubnets":["10.51.231.0/24"]
            ```
    * Direcciones IP de la puerta de enlace de VPN del centro de conectividad virtual. Dado que cada conexión de la puerta de enlace de VPN consta de dos túneles en una configuración activo-activo, verá ambas direcciones IP en este archivo. En este ejemplo, verá `Instance0` y `Instance1` para cada sitio, y son direcciones IP privadas, en lugar de direcciones IP públicas.<br>Ejemplo:
           ``` 
           "Instance0":"10.51.230.4"
           "Instance1":"10.51.230.5"
           ```
    * Detalles de la configuración de la conexión de la puerta de enlace de VPN, como BGP y clave precompartida. La clave precompartida se genera automáticamente. La conexión se puede modificar en cualquier momento en la página **Información general** de una clave precompartida personalizada.
  
### <a name="example-device-configuration-file"></a>Archivo de configuración de dispositivo de ejemplo

```
[{
      "configurationVersion":{
        "LastUpdatedTime":"2019-10-11T05:57:35.1803187Z",
        "Version":"5b096293-edc3-42f1-8f73-68c14a7c4db3"
      },
      "vpnSiteConfiguration":{
        "Name":"VPN-over-ER-site",
        "IPAddress":"172.24.127.211",
        "LinkName":"VPN-over-ER"
      },
      "vpnSiteConnections":[{
        "hubConfiguration":{
          "AddressSpace":"10.51.230.0/24",
          "Region":"West US 2",
          "ConnectedSubnets":["10.51.231.0/24"]
        },
        "gatewayConfiguration":{
          "IpAddresses":{
            "Instance0":"10.51.230.4",
            "Instance1":"10.51.230.5"
          }
        },
        "connectionConfiguration":{
          "IsBgpEnabled":false,
          "PSK":"abc123",
          "IPsecParameters":{"SADataSizeInKilobytes":102400000,"SALifeTimeInSeconds":3600}
        }
      }]
    },
    {
      "configurationVersion":{
        "LastUpdatedTime":"2019-10-11T05:57:35.1803187Z",
        "Version":"fbdb34ea-45f8-425b-9bc2-4751c2c4fee0"
      },
      "vpnSiteConfiguration":{
        "Name":"VPN-over-INet-site",
        "IPAddress":"13.75.195.234",
        "LinkName":"VPN-over-INet"
      },
      "vpnSiteConnections":[{
        "hubConfiguration":{
          "AddressSpace":"10.51.230.0/24",
          "Region":"West US 2",
          "ConnectedSubnets":["10.51.231.0/24"]
        },
        "gatewayConfiguration":{
          "IpAddresses":{
            "Instance0":"51.143.63.104",
            "Instance1":"52.137.90.89"
          }
        },
        "connectionConfiguration":{
          "IsBgpEnabled":false,
          "PSK":"abc123",
          "IPsecParameters":{"SADataSizeInKilobytes":102400000,"SALifeTimeInSeconds":3600}
        }
      }]
}]
```

### <a name="configuring-your-vpn-device"></a>Configuración del dispositivo VPN

Si necesita instrucciones para configurar el dispositivo, puede utilizar las que se proporcionan en la [página de scripts de configuración de dispositivo VPN](~/articles/vpn-gateway/vpn-gateway-about-vpn-devices.md#configscripts) con las siguientes advertencias:

* Las instrucciones de la página del dispositivo VPN no están escritas para una red de área extensa virtual. Sin embargo, puede usar los valores de la red de área extensa virtual del archivo de configuración para configurar manualmente el dispositivo VPN. 
* Los scripts de configuración de dispositivo descargables que sean para la puerta de enlace de VPN no funcionan para la red de área extensa virtual, ya que la configuración es diferente.
* Una red de área extensa virtual nueva puede admitir IKEv1 e IKEv2.
* Una red de área extensa virtual solo puede usar dispositivos VPN basados en rutas e instrucciones de los dispositivos.

## <a name="5-view-your-virtual-wan"></a><a name="viewwan"></a>5. Visualizar la instancia de Virtual WAN

1. Vaya a la red de área extensa virtual.
1. En la página **Información general**, cada punto del mapa representa un centro de conectividad.
1. En la sección **Centros y conexiones**, puede ver el centro de conectividad, sitio, región y el estado de la conexión VPN. También puede ver los bytes que entran y salen.

## <a name="6-monitor-a-connection"></a><a name="connectmon"></a>6. Supervisar una conexión

Cree una conexión para supervisar la comunicación entre una máquina virtual de Azure y un sitio remoto. Para información acerca de cómo configurar una supervisión de conexión, consulte [Supervisar la comunicación de red](~/articles/network-watcher/connection-monitor.md). El campo de origen es la dirección IP de la máquina virtual en Azure y la IP de destino es la dirección IP del sitio.

## <a name="7-clean-up-resources"></a><a name="cleanup"></a>7. Limpieza de recursos

Cuando ya no necesite estos recursos, puede usar [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene. Ejecute el siguiente comando de PowerShell y reemplace `myResourceGroup` con el nombre del grupo de recursos:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

Este artículo le ayuda a crear una conexión VPN a través del emparejamiento privado de ExpressRoute mediante Virtual WAN. Para más información sobre Virtual WAN y otras características relacionadas, consulte la página [Acerca de Azure Virtual WAN](virtual-wan-about.md).
