---
title: Acerca de la red de base dual con Azure Route Server (versión preliminar)
description: Obtenga información sobre cómo funciona Azure Route Server (versión preliminar) en una red de base dual.
services: route-server
author: duongau
ms.service: route-server
ms.topic: conceptual
ms.date: 05/04/2021
ms.author: duau
ms.openlocfilehash: b786c58efd193eebbd43a4bd40f29201347c1309
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777059"
---
# <a name="about-dual-homed-network-with-azure-route-server-preview"></a>Acerca de la red de base dual con Azure Route Server (versión preliminar)

Azure Route Server admite la típica topología de red en estrella tipo hub-and-spoke. Esta configuración se debe a que Route Server y la aplicación virtual de red (NVA) están en la red virtual de centro de conectividad. Route Server también le permite configurar una topología diferente denominada red de base dual. Esta configuración se debe a que tiene una red virtual de radio emparejada con dos o más redes virtuales de centro de conectividad. Las máquinas virtuales de la red virtual de radio pueden comunicarse a través de la red virtual de centro de conectividad con el entorno local o Internet.

## <a name="how-to-set-it-up"></a>Configuración

Como se puede ver en el diagrama siguiente, debe:

* Implementar una NVA en cada red virtual de centro de conectividad y el servidor de rutas en la red virtual de radio.
* Habilitar el emparejamiento de VNet entre las redes virtuales de centro de conectividad y radio.
* Configurar el emparejamiento BGP entre Route Server y cada NVA implementada.

:::image type="content" source="./media/about-dual-homed-network/dual-homed-topology.png" alt-text="Diagrama de Route Server en una topología de base dual.":::

### <a name="how-does-it-work"></a>¿Cómo funciona?

En el plano de control, la NVA y Route Server intercambiarán rutas como si se implementaran en la misma red virtual. La NVA aprenderá sobre las direcciones de red virtual de radio de Route Server. Route Server aprenderá las rutas de cada una de las NVA. A continuación, Route Server programará todas las máquinas virtuales de la red virtual de radio con las rutas que aprendió. 

En el plano de datos, las máquinas virtuales de la red virtual de radio verán la NVA de seguridad o la NVA de VPN en el centro de conectividad como próximo salto. El tráfico destinado al tráfico enlazado a Internet o al tráfico híbrido entre entornos locales ahora se enruta a través de las NVA en la red virtual de centro de conectividad. Puede configurar ambos centros de conectividad para que sean activo/activo o activo/pasivo. En el caso de que se produzca un error en el centro de conectividad activo, el tráfico hacia y desde las máquinas virtuales conmutará por error al otro centro de conectividad. Estos errores incluyen, entre otros: errores de NVA o errores de conectividad de servicio. Esta configuración garantiza que la red está configurada para alta disponibilidad.

## <a name="integration-with-expressroute"></a>Integración con ExpressRoute

Puede crear una red de base dual que implique dos o más conexiones de ExpressRoute. Junto con los pasos descritos anteriormente, deberá:

* Crear otra instancia de Route Server en cada red virtual de centro de conectividad que tenga una puerta de enlace de ExpressRoute.
* Configurar el emparejamiento BGP entre la NVA y Route Server en la red virtual de centro de conectividad.
* [Habilitar el intercambio de rutas](quickstart-configure-route-server-portal.md#configure-route-exchange) entre la puerta de enlace de ExpressRoute y Route Server en la red virtual de centro de conectividad.
* Asegurarse de que la opción "Usar puerta de enlace remota o servidor de Route Server" está **deshabilitada** en la configuración de emparejamiento de VNET de red virtual de radio.

:::image type="content" source="./media/about-dual-homed-network/dual-homed-topology-expressroute.png" alt-text="Diagrama de Route Server en una topología de base dual con ExpressRoute.":::

### <a name="how-does-it-work"></a>¿Cómo funciona?

En el plano de control, la NVA de la red virtual de centro de conectividad aprenderá sobre las rutas locales desde la puerta de enlace de ExpressRoute a través del [intercambio de rutas](quickstart-configure-route-server-portal.md#configure-route-exchange) con Route Server en el centro de conectividad. A cambio, la NVA enviará las direcciones de red virtual de radio a la puerta de enlace de ExpressRoute con la misma instancia de Route Server. A continuación, la instancia de Route Server de la red virtual de radio y de centro de conectividad programará las direcciones de red local a las máquinas virtuales de su red virtual correspondiente.

> [!IMPORTANT]
> BGP evita un bucle comprobando el número de AS en la ruta de acceso de AS. Si el enrutador receptor ve su propio número de AS en la ruta de acceso de AS de un paquete BGP recibido, quitará el paquete. En este ejemplo, ambas instancias de Route Server tienen el mismo número de AS, 65515. Para evitar que cada instancia de Route Server quite las rutas de la otra instancia de Route Server, la NVA debe aplicar la directiva de BGP **como invalidación** al emparejarse con cada instancia de Route Server. 
>

En el plano de datos, las máquinas virtuales de la red virtual de radio enviarán primero todo el tráfico destinado a la red local a la NVA de la red virtual de centro de conectividad. A continuación, la NVA reenviará el tráfico a la red local a través de ExpressRoute. El tráfico del entorno local atravesará la misma ruta de acceso a datos en la dirección inversa. Observará que ninguna de las instancias de Route Server está en la ruta de acceso a datos.

## <a name="next-steps"></a>Pasos siguientes

* [Funcionamiento de Azure Route Server con ExpressRoute](expressroute-vpn-support.md)
* [Funcionamiento de Azure Route Server con una aplicación virtual de red](resource-manager-template-samples.md)

