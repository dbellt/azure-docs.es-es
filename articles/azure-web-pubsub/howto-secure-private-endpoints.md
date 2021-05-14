---
title: Protección del tráfico entre una red virtual y el servicio Azure Web PubSub mediante puntos de conexión privados de Azure
description: Información general de los puntos de conexión privados para el acceso seguro al servicio Azure Web PubSub desde redes virtuales.
author: yjin81
ms.author: yajin1
ms.service: azure-web-pubsub
ms.topic: how-to
ms.date: 03/23/2021
ms.openlocfilehash: 58268750b1189552a31e2dc0b455bdd3eff4f5f0
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108167105"
---
# <a name="use-private-endpoints-for-azure-web-pubsub-service"></a>Uso de puntos de conexión privados en el servicio Azure Web PubSub

Puede usar [puntos de conexión privados](../private-link/private-endpoint-overview.md) en el servicio Azure Web PubSub para que los clientes de una red virtual (VNet) puedan acceder de forma segura a los datos a través de una instancia de [Private Link](../private-link/private-link-overview.md). El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para el servicio Azure Web PubSub. El tráfico de red entre los clientes de la red virtual y el servicio Azure Web PubSub atraviesa un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet.

El uso de puntos de conexión privados para el servicio Azure Web PubSub le permite:

- Proteger el servicio Azure Web PubSub mediante el control de acceso a la red para bloquear todas las conexiones en el punto de conexión público del servicio Azure Web PubSub.
- Aumentar la seguridad de la red virtual (VNet), ya que permite bloquear la filtración de datos de la red virtual.
- Conectarse de forma segura al servicio Azure Web PubSub desde las redes locales que se conectan a la red virtual mediante [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) o instancias de [ExpressRoute](../expressroute/expressroute-locations.md) con emparejamiento privado.

## <a name="conceptual-overview"></a>Información general conceptual

:::image type="content" source="./media/howto-secure-private-endpoints/private-endpoint-overview.png" alt-text="Información general de los puntos de conexión privados en el servicio Azure Web PubSub.":::

Un punto de conexión privado es una interfaz de red especial para un servicio de Azure de una [red virtual](../virtual-network/virtual-networks-overview.md). Cuando se crea un punto de conexión privado para el servicio Azure Web PubSub, este proporciona conectividad segura entre los clientes de la red virtual y el servicio. Al punto de conexión privado se le asigna una dirección IP del intervalo de direcciones IP de la red virtual. La conexión entre el punto de conexión privado y Azure Web PubSub usa un vínculo privado seguro.

Las aplicaciones de la red virtual se pueden conectar al servicio Azure Web PubSub a través del punto de conexión privado sin problemas, **ya que se usan las mismas cadenas de conexión y mecanismos de autorización que en cualquier otro caso**. Los puntos de conexión privados se pueden usar con todos los protocolos que admita el servicio Azure Web PubSub, incluida la API REST.

Cuando se crea un punto de conexión privado para una instancia del servicio Azure Web PubSub, se envía una solicitud de consentimiento para su aprobación al propietario del servicio Azure Web PubSub. Si el usuario que solicita la creación del punto de conexión privado también es propietario del servicio Azure Web PubSub, esta solicitud de consentimiento se aprueba automáticamente.

Los propietarios del servicio Azure Web PubSub pueden administrar las solicitudes de consentimiento y los puntos de conexión privados desde la pestaña "*Puntos de conexión privados*" del servicio Azure Web PubSub en [Azure Portal](https://portal.azure.com).

> [!TIP]
> Si desea restringir el acceso al servicio Azure Web PubSub de modo que solo se pueda realizar mediante el punto de conexión privado, [configure el control de acceso de red](howto-secure-network-access-control.md) para denegar o controlar todo acceso que se realice mediante el punto de conexión público.

### <a name="connecting-to-private-endpoints"></a>Conexión a puntos de conexión privados

Los clientes de una red virtual que usen el punto de conexión privado deben utilizar la misma cadena de conexión para el servicio Azure Web PubSub que aquellos clientes que se conectan mediante el punto de conexión público. Confiamos en la resolución del DNS para enrutar automáticamente las conexiones desde la red virtual al servicio Azure Web PubSub a través de un vínculo privado.

> [!IMPORTANT]
> Use la misma cadena de conexión para conectarse al servicio Azure Web PubSub mediante puntos de conexión privados, tal como haría en otras ocasiones. No se conecte al servicio Azure Web PubSub mediante su dirección URL de subdominio `privatelink`.

De forma predeterminada, se crea una [zona DNS privada](../dns/private-dns-overview.md) conectada a la red virtual con las actualizaciones necesarias para los puntos de conexión privados. Sin embargo, si usa su propio servidor DNS, puede que tenga que realizar otros cambios en la configuración de DNS. En la sección [Cambios de DNS](#dns-changes-for-private-endpoints) que aparece a continuación se describen las actualizaciones necesarias para los puntos de conexión privados.

## <a name="dns-changes-for-private-endpoints"></a>Cambios de DNS en puntos de conexión privados

Al crear un punto de conexión privado, el registro del recurso CNAME de DNS del servicio Azure Web PubSub se actualiza a un alias de un subdominio con el prefijo `privatelink`. De forma predeterminada, también se crea una [zona DNS privada](../dns/private-dns-overview.md), que se corresponde con el subdominio `privatelink`, con los registros de recursos D de DNS para los puntos de conexión privados.

Al resolver el nombre de dominio del servicio Azure Web PubSub desde fuera de la red virtual con el punto de conexión privado, se resuelve en el punto de conexión público del servicio Azure Web PubSub. Cuando se resuelve desde la red virtual que hospeda el punto de conexión privado, el nombre de dominio se resuelve en la dirección IP de este.

En el ejemplo anterior, los registros de recursos DNS del servicio Azure Web PubSub "foobar", cuando se resuelven desde fuera de la red virtual que hospeda el punto de conexión privado, serán:

| Nombre                                                  | Tipo  | Value                                                 |
| :---------------------------------------------------- | :---: | :---------------------------------------------------- |
| ``foobar.webpubsub.azure.com``                        | CNAME | ``foobar.privatelink.webpubsub.azure.com``            |
| ``foobar.privatelink.webpubsub.azure.com``            | A     | \<Azure Web PubSub service public IP address\>           |

Como ya se ha mencionado, puede denegar o controlar el acceso de los clientes de fuera de la red virtual con el punto de conexión público mediante el control de acceso de red.

Los registros de recursos DNS "foobar", cuando los resuelva un cliente en la red virtual que hospeda el punto de conexión privado, serán:

| Nombre                                                  | Tipo  | Value                                                 |
| :---------------------------------------------------- | :---: | :---------------------------------------------------- |
| ``foobar.webpubsub.azure.com``                        | CNAME | ``foobar.privatelink.webpubsub.azure.com``            |
| ``foobar.privatelink.webpubsub.azure.com``            | Un     | 10.1.1.5                                              |

Este enfoque permite el acceso al servicio Azure Web PubSub **mediante la misma cadena de conexión** para los clientes de la red virtual que hospeda los puntos de conexión privados y los clientes que están fuera de esta.

Si va a usar un servidor DNS personalizado en la red, los clientes deben ser capaces de resolver el FQDN del punto de conexión del servicio Azure Web PubSub en la dirección IP del punto de conexión privado. Debe configurar el servidor DNS para delegar el subdominio del vínculo privado en la zona DNS privada de la red virtual, o bien configurar los registros D para `foobar.privatelink.webpubsub.azure.com` con la dirección IP del punto de conexión privado.

> [!TIP]
> Cuando use un servidor DNS personalizado o local, debe configurarlo para resolver el nombre del servicio Azure Web PubSub en el subdominio `privatelink` en la dirección IP del punto de conexión privado. Para ello, puede delegar el subdominio `privatelink` en la zona DNS privada de la red virtual, o bien configurar la zona DNS en el servidor DNS y agregar los registros A de DNS.

El nombre de zona DNS recomendado para los puntos de conexión privados del servicio Azure Web PubSub es `privatelink.webpubsub.azure.com`.

Para más información sobre cómo configurar su propio servidor DNS para que admita puntos de conexión privados, consulte los artículos siguientes:

- [Resolución de nombres de recursos en redes virtuales de Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)
- [Configuración de DNS para puntos de conexión privados](../private-link/private-endpoint-overview.md#dns-configuration)

## <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado

### <a name="create-a-private-endpoint-along-with-a-new-azure-web-pubsub-service-in-the-azure-portal"></a>Creación de un punto de conexión privado junto con una instancia nueva del servicio Azure Web PubSub en Azure Portal

1. Al crear una instancia del servicio Azure Web PubSub, seleccione la pestaña **Redes**. Elija **Punto de conexión privado** como método de conectividad.

    :::image type="content" source="./media/howto-secure-private-endpoints/portal-create-blade-networking-tab.png" alt-text="Creación del servicio Azure Web PubSub: pestaña Redes.":::

1. Seleccione **Agregar**. Rellene la suscripción, el grupo de recursos, la ubicación y el nombre del nuevo punto de conexión privado. Elija una red virtual y una subred.

1. Seleccione **Revisar + crear**.

### <a name="create-a-private-endpoint-for-an-existing-azure-web-pubsub-service-in-the-azure-portal"></a>Creación de un punto de conexión privado para una instancia del servicio Azure Web PubSub existente en Azure Portal

1. Vaya al servicio Azure Web PubSub.

1. Seleccione el menú de configuración denominado **Conexiones de punto de conexión privado**.

1. Seleccione el botón **+ Punto de conexión privado** en la parte superior.

1. Complete la suscripción, el grupo de recursos, el nombre del recurso y la región para el punto de conexión privado nuevo.

1. Elija el recurso del servicio Azure Web PubSub de destino.

1. Elección de la red virtual de destino

1. Seleccione **Revisar + crear**.

## <a name="pricing"></a>Precios

Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link).

## <a name="known-issues"></a>Problemas conocidos

Tenga en cuenta los siguientes problemas conocidos relacionados con los puntos de conexión privados para el servicio Azure Web PubSub.

### <a name="free-tier"></a>Nivel gratis

La instancia de nivel gratis del servicio Azure Web PubSub no se puede integrar con el punto de conexión privado.

### <a name="access-constraints-for-clients-in-vnets-with-private-endpoints"></a>Restricciones de acceso para los clientes de redes virtuales con puntos de conexión privados

Los clientes de redes virtuales con puntos de conexión privados se enfrentan a ciertas restricciones al acceder a otras instancias del servicio Azure Web PubSub que tienen puntos de conexión privados. Por ejemplo, supongamos que una red virtual N1 tiene un punto de conexión privado para una instancia W1 del servicio Azure Web PubSub. Si el servicio Azure Web PubSub W2 tiene un punto de conexión privado en una red virtual N2, los clientes de la red virtual N1 también deben tener acceso al servicio Azure Web PubSub W2 con un punto de conexión privado. Si el servicio Azure Web PubSub W2 no tiene ningún punto de conexión privado, los clientes de la red virtual N1 pueden acceder al servicio Azure Web PubSub en esa cuenta sin un punto de conexión privado.

Esta restricción es el resultado de los cambios de DNS realizados cuando el servicio Azure Web PubSub W2 crea un punto de conexión privado.

### <a name="network-security-group-rules-for-subnets-with-private-endpoints"></a>Reglas de grupo de seguridad de red para subredes con puntos de conexión privados

Actualmente, no se pueden configurar reglas de [grupo de seguridad de red](../virtual-network/network-security-groups-overview.md) (NSG) ni rutas definidas por el usuario para puntos de conexión privados. Las reglas de NSG que se aplican a la subred que hospeda el punto de conexión privado se aplican al punto de conexión privado. Una solución alternativa limitada para este problema es implementar las reglas de acceso para los puntos de conexión privados en las subredes de origen, aunque este enfoque puede requerir mayor sobrecarga de administración.

