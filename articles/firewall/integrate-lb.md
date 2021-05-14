---
title: Integración de Azure Firewall con Azure Standard Load Balancer
description: Puede integrar una instancia de Azure Firewall en una red virtual con una instancia de Azure Standard Load Balancer (pública o interna).
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 04/14/2021
ms.author: victorh
ms.openlocfilehash: e14a8afe27fc9dd9ca40730dd7e681c3093e0b50
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107505911"
---
# <a name="integrate-azure-firewall-with-azure-standard-load-balancer"></a>Integración de Azure Firewall con Azure Standard Load Balancer

Puede integrar una instancia de Azure Firewall en una red virtual con una instancia de Azure Standard Load Balancer (pública o interna). 

El diseño preferido consiste integrar un equilibrador de carga interno con el firewall de Azure, ya que este es un diseño mucho más sencillo. Puede usar un equilibrador de carga público si ya tiene uno implementado y desea mantenerlo en su lugar. Sin embargo, debe tener en cuenta un problema de ruta asimétrica que puede interrumpir la funcionalidad con el escenario del equilibrador de carga público.

Para más información sobre Azure Load Balancer, consulte [¿Qué e Azure Load Balancer?](../load-balancer/load-balancer-overview.md)

## <a name="public-load-balancer"></a>Equilibrador de carga público

Con un equilibrador de carga público, el equilibrador de carga se implementa con una dirección IP de front-end público.

### <a name="asymmetric-routing"></a>Ruta asimétrica

La ruta asimétrica se produce cuando un paquete toma una ruta al destino y toma otra ruta cuando vuelve al origen. Este problema se produce cuando una subred tiene una ruta predeterminada que va a la dirección IP privada del firewall, y usted usa un equilibrador de carga público. En este caso, el tráfico entrante del equilibrador de carga se recibe a través de su dirección IP pública, pero la ruta de vuelta pasa a través de la dirección IP privada del firewall. Dado que el firewall es con estado, quita el paquete de vuelta porque el firewall no tiene conocimiento de este tipo de sesión establecida.

### <a name="fix-the-routing-issue"></a>Corrección del problema de enrutamiento

Al implementar una instancia de Azure Firewall en una subred, un paso es crear una ruta predeterminada para la subred que dirige los paquetes a través de la dirección IP privada del firewall ubicado en AzureFirewallSubnet. Para más información, consulte el [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](tutorial-firewall-deploy-portal.md#create-a-default-route).

Al introducir el firewall en el escenario del equilibrador de carga, quiere que su tráfico de Internet llegue a través de la dirección IP pública del firewall. Desde allí, el firewall aplica sus reglas de firewall y traduce las direcciones de red de los paquetes a la dirección IP pública del equilibrador de carga. Allí es donde se produce el problema. Los paquetes llegan a la dirección IP pública del firewall, pero vuelven al firewall a través de la dirección IP privada (mediante la ruta predeterminada).
Para evitar este problema, cree una ruta de host adicional para la dirección IP pública del firewall. Los paquetes que van a la dirección IP pública del firewall se enrutan a través de Internet. Esto evita que tomen la ruta predeterminada a la dirección IP privada del firewall.

![Ruta asimétrica](media/integrate-lb/Firewall-LB-asymmetric.png)

### <a name="route-table-example"></a>Ejemplo de tabla de rutas

Por ejemplo, las rutas siguientes son para un firewall en la dirección IP pública 20.185.97.136 y la dirección IP privada 10.0.1.4.

> [!div class="mx-imgBorder"]
> ![Tabla de rutas](media/integrate-lb/route-table.png)

### <a name="nat-rule-example"></a>Ejemplo de regla NAT

En el ejemplo siguiente, una regla NAT traduce el tráfico RDP al firewall en 20.185.97.136 a través del equilibrador de carga en 20.42.98.220:

> [!div class="mx-imgBorder"]
> ![Regla de NAT](media/integrate-lb/nat-rule-02.png)

### <a name="health-probes"></a>Sondeos de estado

Recuerde que debe tener un servicio web que se ejecute en los hosts del grupo de equilibradores de carga si usa sondeos de estado TCP para el puerto 80 o sondeos HTTP/HTTPS.

## <a name="internal-load-balancer"></a>Equilibrador de carga interno

Con un equilibrador de carga interno, el equilibrador de carga se implementa con una dirección IP de front-end privado.

No hay ningún problema de rutas asimétricas con este escenario. Los paquetes entrantes llegan a la dirección IP pública del firewall, se traducen a la dirección IP privada del equilibrador de carga y, luego, devuelven a la dirección IP privada del firewall con la misma ruta de vuelta.

Por lo tanto, puede implementar este escenario similar para el escenario del equilibrador de carga público, pero sin necesidad de la ruta de host de direcciones IP públicas del firewall.

Las máquinas virtuales del grupo de back-end pueden tener conectividad a Internet de salida a través de Azure Firewall. Configure una ruta definida por el usuario en la subred de la máquina virtual, con el firewall como el próximo salto.


## <a name="additional-security"></a>Seguridad adicional

Para mejorar la seguridad de su escenario de equilibrio de carga, puede usar grupos de seguridad de red (NSG).

Por ejemplo, puede crear un NSG en la subred de back-end donde se encuentran las máquinas virtuales de carga equilibrada. Permita el tráfico entrante procedente de la dirección IP/puerto del firewall.

![Grupo de seguridad de red](media/integrate-lb/nsg-01.png)

Para más información sobre NSG, consulte [Grupos de seguridad](../virtual-network/network-security-groups-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Consulte el tutorial [Implementación y configuración de Azure Firewall](tutorial-firewall-deploy-portal.md).