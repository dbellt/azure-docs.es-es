---
title: Protección del tráfico entre Application Gateway y los grupos de back-end
titleSuffix: Azure Virtual WAN
description: Obtenga información sobre los escenarios de enrutamiento de Virtual WAN para el tráfico seguro que viaja a través de una puerta de enlace de aplicación. La puerta de enlace de aplicación se implementa en una red virtual de radio que se conecta a un centro protegido de Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: cherylmc
ms.openlocfilehash: 4231ca2330fd49bcfae28c0aa844088bdbd7f50e
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110575345"
---
# <a name="scenario-secure-traffic-between-application-gateway-and-backend-pools"></a>Escenario: protección del tráfico entre Application Gateway y los grupos de back-end.

Al trabajar con el enrutamiento de centros virtuales de Virtual WAN, hay bastantes escenarios disponibles. En este escenario, el tráfico de un usuario entra en Azure a través de una puerta de enlace de aplicaciones implementada en una red virtual radial que está conectada a un centro de Virtual WAN protegido (un centro de Virtual WAN con Azure Firewall). El objetivo es usar Azure Firewall en el centro virtual protegido para inspeccionar el tráfico entre la puerta de enlace de aplicaciones y los grupos de back-end.

Hay dos patrones de diseño específicos en este escenario, dependiendo de si la puerta de enlace de aplicaciones y los grupos de back-end están en la misma red virtual o en redes virtuales diferentes.

* **Escenario 1:** la puerta de enlace de aplicaciones y los grupos de back-end están en la misma red virtual emparejada con un centro de Virtual WAN (subredes independientes).
* **Escenario 2:** la puerta de enlace de aplicaciones y los grupos de back-end están redes virtuales diferentes que están emparejadas con un centro de Virtual WAN diferente.

## <a name="scenario-1---same-vnet"></a><a name="scenario-1"></a>Escenario 1: misma red virtual

En este escenario, la puerta de enlace de aplicaciones y los grupos de back-end están en la misma red virtual emparejada con un centro de Virtual WAN (subredes independientes).

:::image type="content" source="./media/routing-scenarios/secured-application-gateway/same-vnet.png" alt-text="Diagrama del escenario 1." lightbox="./media/routing-scenarios/secured-application-gateway/same-vnet.png":::

### <a name="workflow"></a>Flujo de trabajo

Actualmente, las rutas que se anuncian desde la tabla de rutas de Virtual WAN a las redes virtuales radiales, se aplican a toda la red virtual y no a las subredes de la red virtual radial. Como resultado, las rutas definidas por el usuario son necesarias para habilitar este escenario. Para obtener información sobre las rutas definidas por el usuario (UDR), consulte las [rutas personalizadas de Virtual Network](../virtual-network/virtual-networks-udr-overview.md#user-defined).


1. En la red virtual radial de Azure Firewall Manager que contiene la puerta de enlace de aplicaciones y los grupos de back-end, seleccione **Habilitar tráfico seguro de Internet** y **Habilitar tráfico privado seguro**.
1. Configure las rutas definidas por el usuario (UDR) en la subred de la puerta de enlace de la aplicación.

   * Para asegurarse de que la puerta de enlace de aplicaciones puede enviar tráfico directamente a Internet, especifique el siguiente UDR:

     * **Prefijo de dirección**: 0.0.0.0.0/0
     * **Próximo salto**: Internet

   * Para asegurarse de que la puerta de enlace de aplicaciones puede enviar tráfico al grupo de back-end a través de Azure Firewall en el centro de Virtual WAN, especifique la siguiente UDR:

      * **Prefijo de dirección:** subred del grupo de back-end (10.2.0.0/24)
      * **Próximo salto**: IP privada de Azure Firewall

1. Configure una ruta definida por el usuario (UDR) en la subred del grupo de back-end.

   * **Prefijo de dirección**: subred de Application Gateway
   * **Próximo salto**: IP privada de Azure Firewall

## <a name="scenario-2---different-vnets"></a><a name="scenario-2"></a>Escenario 2: redes virtuales diferentes

En este escenario, la puerta de enlace de aplicaciones y los grupos de back-end están redes virtuales diferentes que están emparejadas con un centro de Virtual WAN diferente.

:::image type="content" source="./media/routing-scenarios/secured-application-gateway/different-vnets.png" alt-text="Diagrama del escenario 2." lightbox="./media/routing-scenarios/secured-application-gateway/different-vnets.png":::

### <a name="workflow"></a>Flujo de trabajo

Actualmente, las rutas que se anuncian desde la tabla de rutas de Virtual WAN a las redes virtuales radiales, se aplican a toda la red virtual y no a las subredes de la red virtual radial. Como resultado, las rutas definidas por el usuario son necesarias para habilitar este escenario. Para obtener información sobre las rutas definidas por el usuario (UDR), consulte las [rutas personalizadas de Virtual Network](../virtual-network/virtual-networks-udr-overview.md#user-defined).

1. En **Azure Firewall Manager**, seleccione **Habilitar tráfico seguro de Internet** y **Habilitar tráfico privado seguro** en ambas redes virtuales radiales.

1. Configure las rutas definidas por el usuario (UDR) en la subred de la puerta de enlace de la aplicación. Para asegurarse de que la puerta de enlace de aplicaciones puede enviar tráfico directamente a Internet, especifique el siguiente UDR:

   * **Prefijo de dirección**: 0.0.0.0.0/0
   * **Próximo salto**: Internet

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
* Para obtener información sobre las rutas definidas por el usuario (UDR), consulte las [Rutas personalizadas de Virtual Network](../virtual-network/virtual-networks-udr-overview.md#user-defined).
* Para obtener información sobre los centros virtuales protegidos de Virtual WAN, consulte [Centros virtuales protegidos.](../firewall-manager/secured-virtual-hub.md)
