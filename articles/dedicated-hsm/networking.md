---
title: 'Consideraciones de redes: Azure Dedicated HSM | Microsoft Docs'
description: Introducción a las consideraciones de redes aplicables a las implementaciones de Azure Dedicated HSM
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/25/2021
ms.author: keithp
ms.openlocfilehash: 14f7a88e756123b807852d78b6511939b81fd9db
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108126064"
---
# <a name="azure-dedicated-hsm-networking"></a>Redes de Azure Dedicated HSM

Azure Dedicated HSM requiere un entorno de red muy seguro. Esto es así tanto desde la nube de Azure al entorno de TI del cliente (local) con aplicaciones distribuidas como para escenarios de alta disponibilidad. Las redes de Azure proporcionan esto y existen cuatro áreas distintas que deben abordarse.

- Creación de dispositivos HSM dentro de la red virtual (VNet) en Azure
- Conexión local a recursos basados en la nube para la configuración y administración de dispositivos HSM
- Creación y conexión de redes virtuales para la interconexión de recursos de aplicación y dispositivos HSM
- Conexión de redes virtuales entre regiones para la intercomunicación y también para habilitar escenarios de alta disponibilidad

## <a name="virtual-network-for-your-dedicated-hsms"></a>Red virtual para los HSM dedicados

Los HSM dedicados se integran en una red virtual y se colocan en la propia red privada de los clientes en Azure. Esto permite acceder a los dispositivos desde máquinas virtuales o recursos de proceso en la red virtual.  
Para más información sobre la integración de servicios de Azure en la red virtual y las funcionalidades que ofrece, consulte la documentación sobre [Red virtual para servicios de Azure](../virtual-network/virtual-network-for-azure-services.md).

### <a name="virtual-networks"></a>Redes virtuales

Antes de aprovisionar un dispositivo HSM dedicado, los clientes primero deberán crear una red virtual en Azure o utilizar una que ya exista en la suscripción de los clientes. La red virtual define el perímetro de seguridad para el dispositivo HSM dedicado. Para más información acerca de cómo crear redes virtuales, consulte la [documentación de redes virtuales](../virtual-network/virtual-networks-overview.md).

### <a name="subnets"></a>Subredes

Las subredes segmentan la red virtual en espacios de direcciones independientes utilizables por los recursos de Azure que coloque en ellos. Los HMS dedicados se implementan en una subred que se encuentra en la red virtual. Cada dispositivo HSM dedicado que se implementa en la subred del cliente recibirá una dirección IP privada de esta subred. La subred en la que se implementa el dispositivo HSM debe delegarse explícitamente al servicio: Microsoft.HardwareSecurityModules/dedicatedHSMs. Esto concede determinados permisos al servicio HSM para la implementación en la subred. La delegación a HSM dedicados impone ciertas restricciones de directiva en la subred. Actualmente no se admiten grupos de seguridad de red (NSG) ni rutas definidas por el usuario (UDR) en las subredes delegadas. En consecuencia, una vez que una subred se delega a HSM dedicados, solo puede usarse para implementar recursos HSM. Al implementar cualquier otro recurso de cliente en la subred se producirá un error.


### <a name="expressroute-gateway"></a>Puerta de enlace de ExpressRoute

Un requisito de la arquitectura actual es la configuración de una puerta de enlace de ExpressRoute en la subred de los clientes en la que debe colocarse un dispositivo HSM para permitir la integración del dispositivo HSM en Azure. Esta puerta de enlace de ExpressRoute no se puede utilizar para conectar ubicaciones locales a los dispositivos HSM de los clientes en Azure.

## <a name="connecting-your-on-premises-it-to-azure"></a>Conexión del entorno de TI local a Azure

Al crear recursos basados en la nube, este suele ser un requisito típico para una conexión privada a los recursos de TI locales. En el caso de un HSM dedicado, esto servirá principalmente para que el software cliente de HSM configure los dispositivos HSM y también para actividades como copias de seguridad y extracción de registros de HSM para su análisis. Una decisión clave aquí es la naturaleza de la conexión, ya que hay diversas opciones.  La opción más flexible es VPN de sitio a sitio, ya que probablemente habrá varios recursos locales que requieran una comunicación segura con los recursos (incluidos los HSM) de la nube de Azure. Esto requerirá que la organización del cliente disponga de un dispositivo VPN para facilitar la conexión. Es posible usar una conexión VPN de punto a sitio si hay solo un único punto de conexión en el entorno local, como una única estación de trabajo de administración.
Para más información sobre las opciones de conectividad, consulte [Opciones de planificación de puerta de enlace de VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#planningtable).

> [!NOTE]
> En este momento, ExpressRoute no es una opción para la conexión a los recursos del entorno local. Asimismo, debe tenerse en cuenta que la puerta de enlace de ExpressRoute usada como se describe anteriormente no sirve para conexiones a la infraestructura local.

### <a name="point-to-site-vpn"></a>VPN de punto a sitio

Una red privada virtual de punto a sitio es la forma más sencilla de establecer una conexión segura a un único punto de conexión en el entorno local. Esta puede ser la solución adecuada si piensa tener solo una estación de trabajo de administración para HSM dedicados basados en Azure.

### <a name="site-to-site-vpn"></a>VPN de sitio a sitio

Una red privada virtual de sitio a sitio permite una comunicación segura entre los HSM dedicados basados en Azure y su entorno de TI local. Una razón para optar por esta solución es si dispone de un centro de copia de seguridad para los HSM en el entorno local y necesita una conexión entre ambos para ejecutar la copia de seguridad.

## <a name="connecting-virtual-networks"></a>Conexión de redes virtuales

Una arquitectura típica de implementación de HSM dedicados empezará con una única red virtual y la subred correspondiente en la que se crearon y aprovisionaron los dispositivos HSM. Dentro de esa misma región, podría haber redes virtuales adicionales y subredes para componentes de aplicación que podrían hacer uso de los HSM dedicados. Para habilitar la comunicación a través de estas redes, usamos el emparejamiento de redes virtuales.

### <a name="virtual-network-peering"></a>Emparejamiento de redes virtuales de Azure

Cuando hay varias redes virtuales dentro de una región que necesitan acceso a los recursos de las demás, es posible usar el emparejamiento de redes virtuales para crear los canales de comunicación seguros entre ellas.  El emparejamiento de redes virtuales no solo proporciona comunicaciones seguras, sino que también garantiza conexiones de baja latencia y alto ancho de banda entre los recursos de Azure.

![emparejamiento de redes](media/networking/peering.png)

## <a name="connecting-across-azure-regions"></a>Conexión a través de las regiones de Azure

Los dispositivos HSM tienen la capacidad, mediante bibliotecas de software, de redirigir el tráfico a un HSM alternativo. El redireccionamiento del tráfico es útil si se produce un error en los dispositivos o se pierde el acceso a un dispositivo. Los escenarios de errores a nivel regional se pueden mitigar mediante la implementación de HSM en otras regiones y la habilitación de la comunicación entre redes virtuales entre regiones.

### <a name="cross-region-ha-using-vpn-gateway"></a>Alta disponibilidad entre regiones mediante VPN Gateway

Para aplicaciones distribuidas globalmente o para escenarios de conmutación por error regional de alta disponibilidad, es necesario conectar redes virtuales entre regiones. Con Azure Dedicated HSM, se puede lograr alta disponibilidad mediante el uso de VPN Gateway, que proporciona un túnel seguro entre las dos redes virtuales. Para más información sobre las conexiones de red virtual a red virtual mediante VPN Gateway, consulte el artículo titulado [¿Qué es VPN Gateway?](../vpn-gateway/design.md#V2V)

> [!NOTE]
> El emparejamiento de red virtual global no está disponible en escenarios de conectividad entre regiones con HSM dedicados en este momento y, en su lugar, debe utilizarse VPN Gateway. 

![En el diagrama se muestran dos regiones conectadas por dos puertas de enlace V P N. Cada región contiene redes virtuales emparejadas.](media/networking/global-vnet.png)

## <a name="networking-restrictions"></a>Restricciones de redes
> [!NOTE]
> Una restricción del servicio Dedicated HSM mediante delegación de subredes impone restricciones que deben tenerse en cuenta a la hora de diseñar la arquitectura de red de destino para una implementación de HSM. El uso de la delegación de subredes significa que los NSG, UDR y el emparejamiento de VNet global no se admiten para Dedicated HSM. En las secciones siguientes encontrará técnicas alternativas para lograr el mismo resultado u otro similar para estas funcionalidades. 

La NIC de HSM que reside en la red virtual de Dedicated HSM no puede usar grupos de seguridad de red ni rutas definidas por el usuario. Esto significa que no es posible establecer directivas de denegación predeterminadas desde el punto de vista de la red virtual de Dedicated HSM, y que otros segmentos de red deben incluirse en la lista de permitidos para acceder al servicio Dedicated HSM. 

La adición de la solución de proxy de aplicaciones virtuales de red (NVA) también permite que un firewall de NVA en el concentrador de tránsito/DMZ se coloque lógicamente delante de la NIC de HSM, lo que proporciona la alternativa necesaria a los NSG y UDR.

### <a name="solution-architecture"></a>Arquitectura de la solución
Este diseño de redes requiere los elementos siguientes:
1.  Una red virtual de centro de conectividad de tránsito o DMZ con un nivel de proxy de NVA. Idealmente, hay dos NVA o más. 
2.  Un circuito ExpressRoute con un emparejamiento privado habilitado y una conexión a la red virtual del centro de tránsito.
3.  Un emparejamiento de VNet entre la red virtual del centro de tránsito y la red virtual de Dedicated HSM.
4.  Como opción, se puede implementar un firewall de NVA o Azure Firewall para ofrecer servicios de DMZ en el centro de conectividad. 
5.  Se pueden emparejar redes virtuales radiales de carga de trabajo adicionales a la red virtual del centro. El cliente de Gemalto puede acceder al servicio Dedicated HSM a través de la red virtual del centro.

![En el diagrama se muestra una red virtual de centro de conectividad DMZ con un nivel de proxy de NVA como solución para los NSG y UDR.](media/networking/network-architecture.png)

Dado que agregar la solución de proxy de NVA también permite que un firewall de NVA en el centro de tránsito/DMZ se coloque lógicamente delante de la NIC de HSM, se proporcionan las directivas de denegación predeterminadas necesarias. En nuestro ejemplo, usaremos Azure Firewall para este propósito y necesitaremos los siguientes elementos:
1. Una instancia de Azure Firewall implementada en la subred "AzureFirewallSubnet" en la red virtual del centro de DMZ.
2. Una tabla de enrutamiento con una UDR que dirija hacia Azure Firewall el tráfico que se envía al punto de conexión privado de Azure ILB. Esta tabla de enrutamiento se aplicará a GatewaySubnet donde reside la puerta de enlace virtual de ExpressRoute del cliente.
3. Reglas de seguridad de red dentro de Azure Firewall para permitir el reenvío entre un intervalo de orígenes de confianza y el punto de conexión privado de Azure IBL escuchando en el puerto TCP 1792. Esta lógica de seguridad agregará la directiva de "denegación predeterminada" necesaria en el servicio Dedicated HSM. Es decir, solo se permitirán los intervalos IP de origen de confianza en el servicio Dedicated HSM. Se quitarán todos los demás intervalos.  
4. Una tabla de enrutamiento con una UDR que dirija hacia Azure Firewall el tráfico que se envía al entorno local. Esta tabla de enrutamiento se aplicará a la subred de proxy de NVA. 
5. Un NSG aplicado a la subred de NVA del proxy para confiar solo en el intervalo de subredes de Azure Firewall como origen, y para permitir el reenvío solo a la dirección IP de la NIC de HSM a través del puerto TCP 1792. 

> [!NOTE]
> Dado que el nivel de proxy de NVA aplicará SNAT a la dirección IP del cliente a medida que se reenvía a la NIC de HSM, no se requiere ninguna UDR entre la red virtual de HSM y la red virtual del centro de conectividad de DMZ.  

### <a name="alternative-to-udrs"></a>Alternativa a las UDR
La solución al nivel de NVA mencionada anteriormente funciona como alternativa a las UDR. Hay algunos puntos importantes que se deben destacar.
1.  La traducción de direcciones de red debe configurarse en NVA para permitir que el tráfico de retorno se enrute correctamente.
2. Los clientes deben deshabilitar la configuración de Luna HSM de comprobación de IP de cliente para usar VNA para NAT. Los siguientes comandos sirven de ejemplo.
```
Disable:
[hsm01] lunash:>ntls ipcheck disable
NTLS client source IP validation disabled
Command Result : 0 (Success)

Show:
[hsm01] lunash:>ntls ipcheck show
NTLS client source IP validation : Disable
Command Result : 0 (Success)
```
3.  Implementación de UDR para el tráfico de entrada en el nivel de NVA. 
4. Por diseño, las subredes de HSM no iniciarán una solicitud de conexión saliente a nivel de la plataforma.

### <a name="alternative-to-using-global-vnet-peering"></a>Alternativa al uso del Emparejamiento de VNET global
Hay un par de arquitecturas que puede usar como alternativa al Emparejamiento de VNET global.
1.  Use la [conexión de VPN Gateway de red virtual a red virtual](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md). 
2.  Conecte la red virtual de HSM con otra red virtual con un circuito ER. Esto funciona mejor cuando se requiere una ruta de acceso local directa o una red virtual de VPN. 

#### <a name="hsm-with-direct-express-route-connectivity"></a>HSM con conectividad directa ExpressRoute
![Diagrama que muestra HSM con conectividad directa ExpressRoute](media/networking/expressroute-connectivity.png)

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas más frecuentes](faq.md)
- [Compatibilidad](supportability.md)
- [Alta disponibilidad](high-availability.md)
- [Seguridad física](physical-security.md)
- [Supervisión](monitoring.md)
- [Arquitectura de implementación](deployment-architecture.md)