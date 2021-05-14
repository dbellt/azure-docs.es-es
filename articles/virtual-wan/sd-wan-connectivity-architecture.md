---
title: Arquitecturas de conectividad de Virtual WAN y SD WAN
titleSuffix: Azure Virtual WAN
description: Obtenga información sobre cómo interconectar una SD-WAN privada con Azure Virtual WAN.
services: virtual-wan
author: skishen525
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: sukishen
ms.openlocfilehash: eb6ae05b0e3c296a1ca56d3ea6085b8cd4586c23
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165702"
---
# <a name="sd-wan-connectivity-architecture-with-azure-virtual-wan"></a>Arquitectura de conectividad SD WAN con Azure Virtual WAN

Azure Virtual WAN es un servicio de red que reúne muchos servicios de seguridad y conectividad en la nube con una única interfaz operativa. Estos servicios incluyen rama (a través de una VPN de sitio a sitio), usuario remoto (VPN de punto a sitio), conectividad privada (ExpressRoute), conectividad transitiva dentro de la nube para redes virtuales, interconectividad VPN y ExpressRoute, enrutamiento, Azure Firewall y cifrado para conectividad privada.

Aunque la propia Azure Virtual WAN es una WAN definida por software (SD-WAN), también se ha diseñado para permitir una interconexión perfecta con las tecnologías y los servicios SD-WAN basados en el entorno local. Muchos de estos servicios se ofrecen a través de nuestro ecosistema [Virtual WAN](virtual-wan-locations-partners.md) y los asociados de Azure Networking Managed Services [(MSP)](../networking/networking-partners-msp.md). Las empresas que transforman su WAN privada en SD-WAN tienen opciones cuando interconectan su SD-WAN privada con Azure Virtual WAN. Las empresas pueden elegir entre estas opciones:

* Modelo de interconexión directa
* Modelo de interconexión directa con NVA en centro de VWAN
* Modelo de interconexión indirecta
* Modelo WAN híbrido administrado con su proveedor de servicios administrados favorito [MSP](../networking/networking-partners-msp.md)

En todos estos casos, la interconexión de Virtual WAN con la SD-WAN es similar en relación a la conectividad, pero puede variar desde el punto de vista operativo y en la orquestación.

## <a name="direct-interconnect-model"></a><a name="direct"></a>Modelo de interconexión directa

:::image type="content" source="./media/sd-wan-connectivity-architecture/direct.png" alt-text="Modelo de interconexión directa":::

En este modelo de arquitectura, el equipo local (CPE) de la rama SD-WAN se conecta directamente a los centros de Virtual WAN a través de las conexiones IPsec. El CPE de la rama también puede estar conectado a otras ramas a través de la SD-WAN privada, o bien usar Virtual WAN para la conectividad de rama a rama. Las ramas que necesitan acceder a sus cargas de trabajo en Azure podrán acceder de forma directa y segura a Azure a través de los túneles IPsec que terminan en los centros Virtual WAN.

Los asociados CPE de SD-WAN pueden habilitar la automatización para automatizar la conectividad IPsec, normalmente tediosa y propensa a errores, desde sus dispositivos CPE respectivos. La automatización permite que el controlador de la SD-WAN se comunique con Azure a través de la API de Virtual WAN, con el fin de configurar los sitios de Virtual WAN y de insertar la configuración de túnel IPsec necesaria en los CPE de la rama. Consulte las [directrices de automatización](virtual-wan-configure-automation-providers.md) para obtener la descripción de la automatización de la interconexión de Virtual WAN por parte de varios asociados de SD-WAN.

El CPE de la SD-WAN se mantiene en el lugar donde se implementa y se aplica la optimización del tráfico y la selección de rutas de acceso. 

En este modelo, es posible que algunas optimizaciones de tráfico propietario del proveedor basadas en características de tráfico en tiempo real no sean compatibles, ya que la conectividad con Virtual WAN se realiza a través de IPsec y la VPN de IPsec se termina en la puerta de enlace VPN de Virtual WAN. Por ejemplo, la selección de rutas de acceso dinámicas en el CPE de la rama es factible debido a que el dispositivo de rama intercambia diversa información de paquetes de red con otro nodo de SD-WAN, por lo que se identifica el mejor vínculo que se debe usar para el tráfico priorizado dinámicamente en la rama. Esta característica puede ser útil en áreas en las que se necesite la optimización de última etapa (rama al punto de referencia de Microsoft más cercano).

Con Virtual WAN, los usuarios pueden obtener la selección de ruta de acceso de Azure, que es una selección de la ruta de acceso basada en directivas en varios vínculos de ISP desde la rama CPE a las puertas de enlace de VPN de Virtual WAN. Virtual WAN permite configurar varios vínculos (rutas de acceso) desde el mismo CPE de rama de SD-WAN; cada vínculo representa una conexión de túnel dual desde una dirección IP pública única del CPE de SD-WAN a dos instancias diferentes de la puerta de enlace de VPN de Azure Virtual WAN. Los proveedores de SD-WAN pueden implementar la ruta de acceso óptima a Azure, en función de las directivas de tráfico establecidas por el motor de directivas en los vínculos de CPE. En el extremo de Azure, todas las conexiones entrantes se tratan de igual manera.

## <a name="direct-interconnect-model-with-nva-in-vwan-hub"></a><a name="direct"></a>Modelo de interconexión directa con NVA en centro de VWAN

:::image type="content" source="./media/sd-wan-connectivity-architecture/direct-nva.png" alt-text="Modelo de interconexión directa con NVA en centro de VWAN":::

Este modelo de arquitectura admite la implementación de una [aplicación virtual de red (NVA) de un tercero directamente en el centro de conectividad virtual](./about-nva-hub.md). Esto permite a los clientes conectar el CPE de su rama a la NVA de la misma marca en el centro de conectividad virtual para poder sacar provecho de todas las funcionalidades propias de SD-WAN al conectarse a cargas de trabajo de Azure. 

Varios asociados de Virtual WAN se han esforzado para ofrecer una experiencia que configura la NVA de forma automática como parte del proceso de implementación. Una vez que la NVA se ha aprovisionado en el centro de conectividad virtual, cualquier otra configuración que necesite la NVA se debe realizar a través del portal de asociados o de la aplicación de administración de NVA. El acceso directo a la NVA no está disponible. Las NVA que están disponibles para implementarse directamente en el centro de Azure Virtual WAN se han diseñado específicamente para su uso en el centro de conectividad virtual. Puede conocer los partners que admiten NVA en un centro de VWAN y consultar su guía de implementación en el artículo [Partners de Virtual WAN](virtual-wan-locations-partners.md#partners-with-integrated-virtual-hub-offerings).

El CPE de la SD-WAN se mantiene en el lugar donde se implementa y se aplica la optimización del tráfico y la selección de rutas de acceso.
En este modelo, se admite la optimización del tráfico propio del proveedor en función de las características del tráfico en tiempo real porque la conectividad con Virtual WAN se realiza a través del NVA de SD-WAN en el centro.

## <a name="indirect-interconnect-model"></a><a name="indirect"></a>Modelo de interconexión indirecta

:::image type="content" source="./media/sd-wan-connectivity-architecture/indirect.png" alt-text="Modelo de interconexión indirecta":::

En este modelo de arquitectura, los CPE de rama SD-WAN se conectan indirectamente a los centros de Virtual WAN. Como se muestra en la ilustración, se implementa un CPE virtual de SD-WAN en una red virtual de empresa. Este CPE virtual, a su vez, se conecta a los centros de Virtual WAN mediante IPsec. El CPE virtual sirve como puerta de enlace de SD-WAN en Azure. Las ramas que necesitan acceder a sus cargas de trabajo en Azure podrán acceder a ellas a través de la puerta de enlace v-CPE.

Dado que la conectividad con Azure se realiza a través de la puerta de enlace v-CPE (NVA), todo el tráfico hacia y desde las redes virtuales de la carga de trabajo de Azure a otras ramas de SD-WAN se dirige a través de la NVA. En este modelo, el usuario es responsable de administrar y operar la NVA de la SD-WAN, como la alta disponibilidad, la escalabilidad y el enrutamiento.
  
## <a name="managed-hybrid-wan-model"></a><a name="hybrid"></a>Modelo WAN híbrido administrado

:::image type="content" source="./media/sd-wan-connectivity-architecture/hybrid.png" alt-text="Modelo WAN híbrido administrado":::

En este modelo de arquitectura, las empresas pueden aprovechar un servicio administrado de SD-WAN ofrecido por un asociado Proveedor de servicios administrados (MSP). Este modelo es similar a los modelos directo o indirecto descritos anteriormente. Sin embargo, en este modelo, el proveedor de la SD-WAN proporciona el diseño, la orquestación y las operaciones de la SD-WAN.

[Los asociados de Azure Networking MSP](../networking/networking-partners-msp.md) pueden usar [Azure Lighthouse](https://azure.microsoft.com/services/azure-lighthouse/) para implementar el servicio SD-WAN y Virtual WAN en la suscripción de Azure del cliente empresarial, así como para operar la WAN híbrida de un extremo a otro en nombre del cliente. Estos MSP también pueden implementar Azure ExpressRoute en la instancia de Virtual WAN y operar como un servicio administrado de un extremo a otro.

## <a name="additional-information"></a>Información adicional

* [Preguntas más frecuentes sobre Virtual WAN](virtual-wan-faq.md)
* [Solución de conectividad remota](work-remotely-support.md)