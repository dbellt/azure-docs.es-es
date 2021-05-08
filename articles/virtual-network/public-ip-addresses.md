---
title: Direcciones IP públicas en Azure
titlesuffix: Azure Virtual Network
description: Obtenga más información sobre las direcciones IP públicas en Azure.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/28/2020
ms.author: allensu
ms.openlocfilehash: 121c22e3a25a95fa64f6f779ebc0827bb6c123c7
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108015958"
---
# <a name="public-ip-addresses"></a>Direcciones IP públicas

Las direcciones IP públicas permiten a los recursos de Internet la comunicación entrante a los recursos de Azure. Permiten que los recursos de Azure se comuniquen con los servicios de Azure orientados al público e Internet. Hasta que cancele la asignación, la dirección estará dedicada al recurso. Un recurso sin una dirección IP pública asignada puede realizar comunicaciones salientes. Azure asigna dinámicamente una dirección IP disponible que no está dedicada al recurso. Para más información acerca de las conexiones salientes en Azure, consulte [Comprender las conexiones salientes en Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

En el Administrador de recursos de Azure, una dirección [IP pública](virtual-network-public-ip-address.md) es un recurso que cuenta con propiedades específicas. Estos son algunos de los recursos con los que puede asociar un recurso de dirección IP pública:

* Interfaces de red de máquinas virtuales
* Equilibradores de carga accesibles desde Internet
* Puertas de enlace de VPN
* Puertas de enlace de aplicaciones
* Azure Firewall

## <a name="ip-address-version"></a>Versión de la dirección IP

Las direcciones IP públicas se crean con una dirección IPv4 o IPv6. 

## <a name="sku"></a>SKU

Para obtener información acerca de la actualización de SKU, consulte [Actualización de IP pública](../virtual-network/virtual-network-public-ip-address-upgrade.md).

Las direcciones IP públicas se crean con una de las SKU siguientes:

>[!IMPORTANT]
> En los recursos de IP pública y en el equilibrador de carga se requieren SKU coincidentes. No puede tener una combinación de recursos de SKU básica y recursos de SKU estándar. No se pueden asociar máquinas virtuales independientes, máquinas virtuales en un recurso de conjunto de disponibilidad o conjuntos de escalado de máquinas virtuales a ambas SKU simultáneamente.  Para los nuevos diseños, deberá considerar usar los recursos de SKU estándar.  Consulte [Load Balancer Estándar](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para obtener más información.

### <a name="standard"></a>Estándar

Las direcciones IP públicas de SKU estándar:

- Utilice siempre el método de asignación estática.
- Tiene un tiempo espera de inactividad del flujo originado de entrada ajustable de entre 4 y 30 minutos, y un valor predeterminado de 4 minutos, y el valor predeterminado del tiempo de espera del flujo originado es de 4 minutos.
- Son seguras de forma predeterminada y se cierran al tráfico de entrada. Permiten mostrar el tráfico de entrada con un [grupo de seguridad de red](./network-security-groups-overview.md#network-security-groups).
- Se asignan a interfaces de red, equilibradores de carga estándar públicos o puertas de enlace de aplicaciones. Para más información sobre Standard Load Balancer, consulte [¿Qué es Azure Load Balancer?](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Pueden tener redundancia de zona (haberse anunciado desde las tres zonas), ser zonales (garantizadas en una zona de disponibilidad preseleccionada específica) o no tener zona (no asociadas a ninguna zona de disponibilidad preseleccionada concreta). Para obtener más información acerca de las zonas de disponibilidad, consulte [Introducción a las zonas de disponibilidad](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Standard Load Balancer and Availability Zones](../load-balancer/load-balancer-standard-availability-zones.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Load Balancer Standard y zonas de disponibilidad). **Las direcciones IP con redundancia de zona solo se pueden crear en regiones de [en las que haya tres zonas de disponibilidad](../availability-zones/az-region.md) activas.** Las direcciones IP creadas antes de que las zonas estén activas no tendrán redundancia de zona.
- Se puede usar como dirección IP de front-end de difusión por proximidad (anycast) para [equilibradores de carga entre regiones](../load-balancer/cross-region-overview.md) (funcionalidad de versión preliminar).
 
> [!NOTE]
> Para evitar que se produzca un error en la comunicación de entrada con el recurso SKU estándar, debe crear un [grupo de seguridad de red](./network-security-groups-overview.md#network-security-groups), asociarlo y permitir explícitamente el tráfico de entrada deseado.

> [!NOTE]
> Cuando se usa el [servicio de metadatos de instancia IMDS](../virtual-machines/windows/instance-metadata-service.md), solo hay direcciones IP públicas con SKU básica disponibles. No se admiten las SKU estándar.

> [!NOTE]
> La configuración de diagnóstico no aparece en la hoja del recurso cuando se usa una dirección IP pública del SKU estándar. Para habilitar el registro en un recurso de dirección IP pública estándar, vaya a la configuración de diagnóstico en la hoja Azure Monitor y seleccione el recurso de dirección IP.

### <a name="basic"></a>Básica

Todas las direcciones IP públicas creadas antes de la introducción de SKU son direcciones IP públicas de SKU básica. 

Con la introducción de SKU, puede especificar qué SKU le gustaría que fuera la dirección IP pública. 

Las direcciones de SKU básica:

- Se asignan con el método de asignación estática o el de asignación dinámica.
- Tiene un tiempo espera de inactividad del flujo originado de entrada ajustable de entre 4 y 30 minutos, y un valor predeterminado de 4 minutos, y el valor predeterminado del tiempo de espera del flujo originado es de 4 minutos.
- Están abiertas de forma predeterminada.  Se recomienda el uso de grupos de seguridad de red, pero es opcional para restringir el tráfico de entrada o de salida.
- Se asignan a cualquier recurso de Azure al que se pueda asignar una dirección IP pública, como:
    * Interfaces de red
    * Puertas de enlace de VPN
    * Puertas de enlace de aplicaciones
    * Equilibradores de carga públicos
- No admiten escenarios de zona de disponibilidad. Use la dirección IP pública de SKU estándar para los escenarios de zona de disponibilidad. Para obtener más información acerca de las zonas de disponibilidad, consulte [Introducción a las zonas de disponibilidad](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Standard Load Balancer and Availability Zones](../load-balancer/load-balancer-standard-availability-zones.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Load Balancer Standard y zonas de disponibilidad).

## <a name="allocation-method"></a>Método de asignación

Las direcciones IP públicas básicas y estándar admiten la asignación **estática**.  Al recurso se le asigna una dirección IP cuando aquel se crea. La dirección IP se libera cuando el recurso se elimina.

Las direcciones IP públicas de SKU básica admiten una asignación **dinámica**. Este es el método de asignación predeterminado. La dirección IP **no** se asigna al recurso en el momento de crearse cuando se selecciona el método dinámico.

La dirección IP se asigna cuando se asocia el recurso de dirección IP pública con una:

* Máquina virtual 
* La primera máquina virtual está asociada con el grupo de back-end de un equilibrador de carga.

La dirección IP se libera cuando se detiene (o elimina) el recurso,  

Por ejemplo, un recurso de dirección IP pública se libera de un recurso denominado **Recurso A**. **Recurso A** recibe una dirección IP diferente en el inicio si el recurso de dirección IP pública se vuelve a asignar.

La dirección IP se libera cuando se cambia el método de asignación de **estático** a **dinámico**. Para asegurarse de que la dirección IP para el recurso asociado siga siendo la misma, establezca explícitamente el método de asignación en **estático**. Una dirección IP estática se asigna de inmediato.

> [!NOTE]
> Incluso cuando se establece el método de asignación en **estático**, no se puede especificar la dirección IP real asignada al recurso de dirección IP pública. Azure asigna la dirección IP desde un grupo de direcciones IP disponibles en la ubicación de Azure cuando se crea el recurso.
>

Las direcciones IP públicas se suelen usar en los escenarios siguientes:

* Cuando debe actualizar las reglas de firewall para comunicarse con los recursos de Azure.
* La resolución de nombres DNS, en la que un cambio de dirección IP requeriría actualizar los registros D.
* Los recursos de Azure se comunican con otras aplicaciones o servicios que utilizan un modelo de seguridad basado en dirección IP.
* Use certificados TLS/SSL vinculados a una dirección IP.

> [!NOTE]
> Azure asigna direcciones IP públicas a partir de un rango único para cada región en cada nube de Azure. Puede descargar la lista de intervalos (prefijos) para las nubes de Azure [Pública](https://www.microsoft.com/download/details.aspx?id=56519), [Gobierno de Estados Unidos](https://www.microsoft.com/download/details.aspx?id=57063), [China](https://www.microsoft.com/download/details.aspx?id=57062) y [Alemania](https://www.microsoft.com/download/details.aspx?id=57064).
>

## <a name="dns-hostname-resolution"></a>Resolución de nombres de host DNS

Seleccione la opción para especificar una etiqueta de nombre de dominio DNS para un recurso de dirección IP pública. 

Esta selección crea una asignación para **domainnamelabel**.**location**.cloudapp.azure.com a la dirección IP pública en el DNS administrado por Azure. 

Por ejemplo, la creación de una dirección IP pública con:

* **contoso** como **domainnamelabel**
* **Ubicación** de Azure **Oeste de EE. UU.**

El nombre de dominio completo (FQDN) **contoso.westus.cloudapp.azure.com** se resuelve en la dirección IP pública del recurso.

> [!IMPORTANT]
> Cada etiqueta de nombre de dominio que se cree debe ser única dentro de su ubicación de Azure.  
>

## <a name="dns-recommendations"></a>Recomendaciones de DNS

Si se necesita un traslado de región, no puede migrar el nombre de dominio completo de la dirección IP pública. Puede usar el nombre de dominio completo para crear un registro CNAME personalizado que apunte a la dirección IP pública. 

Si se requiere un traslado a una dirección IP pública diferente, actualice el registro CNAME en lugar de actualizar el FQDN.

Puede usar [Azure DNS](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address) o un proveedor de DNS externo para el registro DNS. 

## <a name="virtual-machines"></a>Máquinas virtuales

Puede asociar una dirección IP pública con una máquina virtual [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) mediante la asignación a su **interfaz de red**. 

Elija **dinámica** o **estática** para la dirección IP pública. Más información sobre [asignación de direcciones IP a interfaces de red](virtual-network-network-interface-addresses.md).

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="internet-facing-load-balancers"></a>Equilibradores de carga accesibles desde Internet

Puede asociar una dirección IP pública de [SKU](#sku) con una instancia de [Azure Load Balancer](../load-balancer/load-balancer-overview.md) asignándola a la configuración del **front-end** del equilibrador de carga. La dirección IP pública actúa como dirección IP de carga equilibrada. 

Puede asignar una dirección IP pública estática o dinámica al front-end de un equilibrador de carga. Puede asignar varias direcciones IP públicas a un front-end de equilibrador de carga. Esta configuración habilita escenarios de [varias VIP](../load-balancer/load-balancer-multivip-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json), como un entorno de varios inquilinos con sitios web basados en TLS. 

Para más información sobre las SKU de los equilibradores de carga de Azure, consulte [Azure load balancer standard SKU](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (SKU estándar de equilibrador de carga de Azure).

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="vpn-gateways"></a>Puertas de enlace de VPN

[Azure VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) conecta una red virtual de Azure a:

* Redes virtuales de Azure
* Redes locales. 

Se asigna una dirección IP pública a la instancia de VPN Gateway para habilitar la comunicación con la red remota. 

* Asigne una dirección IP pública básica **dinámica** a una configuración de front-end de SKU de VPNGw 1-5.
* Asigne una dirección IP pública estándar **estática** a una configuración de front-end de SKU de VPNGwAZ 1-5.

## <a name="application-gateways"></a>Puertas de enlace de aplicaciones

Puede asociar una dirección IP pública con una [puerta de enlace de aplicaciones](../application-gateway/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)de Azure asignándola a la configuración del **front-end** de la puerta de enlace. 

* Asigne una dirección IP pública **dinámica** de nivel básico a una configuración front-end V1 de la puerta de enlace de aplicaciones. 
* Asigne un dirección de SKU estándar **estática** a una configuración de front-end V2.

## <a name="azure-firewall"></a>Azure Firewall

[Azure Firewall](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) permite crear, aplicar y registrar directivas de conectividad de red y de aplicaciones, en suscripciones y redes virtuales.

Solo es posible asociar direcciones IP públicas estándar **estáticas** con un firewall. Esto permite que los firewalls externos identifiquen el tráfico que procede de su red virtual. 


## <a name="at-a-glance"></a>De un vistazo

En la siguiente tabla se muestra la propiedad a través de la cual una dirección IP pública se puede asociar a un recurso de nivel superior y los métodos de asignación posibles.

| Recurso de nivel superior | Asociación de dirección IP | Dinámica | estática |
| --- | --- | --- | --- |
| Máquina virtual |interfaz de red |Sí |Sí |
| Equilibrador de carga accesible desde Internet |Configuración de front-end |Sí |Sí |
| puerta de enlace de VPN |Configuración de dirección IP de puerta de enlace |Sí |Sí (solo VPNGwAZ) |
| puerta de enlace de aplicaciones |Configuración de front-end |Sí (solo en V1) |Sí (solo en V2) |
| Azure Firewall | Configuración de front-end | No | Sí|

## <a name="limits"></a>Límites

Los límites de una dirección IP se encuentran en el conjunto completo de [límites de red](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits) de Azure. 

Los límites son por región y suscripción. [Póngase en contacto con el servicio de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para aumentar los límites predeterminados hasta alcanzar los límites máximos, según las necesidades empresariales.

## <a name="pricing"></a>Precios

Las direcciones IP públicas pueden tener un precio simbólico. Para más información sobre los precios de las direcciones IP en Azure, revise la página [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="next-steps"></a>Pasos siguientes
* Obtener información sobre las [direcciones IP privadas en Azure](private-ip-addresses.md)
* [Implementar una máquina virtual con una dirección IP pública estática mediante el portal de Azure](virtual-network-deploy-static-pip-arm-portal.md)
