---
title: 'Azure Front Door: Equilibrio de carga con el conjunto de entrega de aplicaciones de Azure | Microsoft Docs'
description: Este artículo le ayuda a saber cuándo Azure recomienda el equilibrio de carga con su conjunto de entrega de aplicaciones.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/06/2021
ms.author: duau
ms.openlocfilehash: 0a7e81c57552fdc24262522343a08fdabba71bfd
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552587"
---
# <a name="load-balancing-with-azures-application-delivery-suite"></a>Equilibrio de carga con el conjunto de entrega de aplicaciones de Azure

## <a name="introduction"></a>Introducción
Microsoft Azure proporciona diversos servicios globales y regionales para administrar cómo se distribuye el tráfico de red y se equilibra la carga: 

* Application Gateway
* Front Door 
* Load Balancer  
* Traffic Manager

El uso de estos servicios juntos, en combinación con la arquitectura zonal y de varias regiones de Azure, le permite crear aplicaciones sólidas, escalables y de alto rendimiento.

:::image type="content" source="./media/front-door-lb-with-azure-app-delivery-suite/application-delivery-figure1.png" alt-text="Conjunto de entrega de aplicaciones":::
 
Estos servicios se dividen en dos categorías:
1. **Servicios de equilibrio de carga global**, como Traffic Manager y Front Door, que distribuyen el tráfico de los usuarios finales entre los back-ends regionales, las nubes e incluso los servicios locales híbridos. El equilibrio de carga global enruta el tráfico al back-end de servicio más cercano y responde a los cambios en la confiabilidad del servicio para ofrecer disponibilidad AlwaysOn y alto rendimiento a los usuarios. 
1. **Servicios de equilibrio de carga regional**, como Load Balancer y Application Gateway, que proporcionan la capacidad de distribuir el tráfico a las máquinas virtuales de una red virtual o a los puntos de conexión de servicio de una región.

Al combinar estos servicios globales y regionales, la aplicación se beneficiará de un tráfico de un extremo a otro confiable y seguro que se envía de los usuarios finales a los servicios de IaaS, PaaS o locales. En la siguiente sección se describe cada uno de estos servicios.

## <a name="global-load-balancing"></a>Equilibrio de carga global
**Traffic Manager** proporciona equilibrio de carga de DNS global. Examina las solicitudes de DNS entrantes y responde con un back-end con estado correcto, de acuerdo con la directiva de enrutamiento que el cliente haya seleccionado. Las opciones de los métodos de enrutamiento son:
- El **enrutamiento de rendimiento**, que envía solicitudes al back-end más cercano con la menor latencia.
- El **enrutamiento de prioridad**, que dirige todo el tráfico a un back-end, con otros back-ends como respaldo.
- El **enrutamiento round robin ponderado**, que distribuye el tráfico según la ponderación asignada a cada back-end.
- El **enrutamiento geográfico**, que garantiza que las solicitudes que proceden de regiones geográficas específicas se controlan mediante los back-ends asignados a esas regiones. Por ejemplo, todas las solicitudes de España deben dirigirse a la región de Azure Centro de Francia.
- El **enrutamiento de subred**, que permite asignar intervalos de direcciones IP a los back-ends, de modo que las solicitudes entrantes de esas IP se envíen al back-end específico. Por ejemplo, los usuarios que se conectan desde el intervalo de direcciones IP de la oficina central deben obtener contenido web distinto de los usuarios generales.

El cliente se conecta directamente a ese back-end. Azure Traffic Manager detecta cuándo un back-end está en mal estado y redirige a los clientes a otra instancia en buen estado. Consulte la documentación de [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) para más información sobre el servicio.

**Azure Front Door** proporciona aceleración dinámica de sitios web (DSA), que incluye equilibrio de carga global HTTP.  Examina las rutas de las solicitudes HTTP entrantes a la región o el back-end de servicio más cercanos para comprobar que existen el nombre de host, la ruta de la dirección URL y las reglas configuradas que se han especificado.  
Front Door termina las solicitudes HTTP en el perímetro de la red de Microsoft y las sondea activamente para detectar los cambios en la aplicación, el mantenimiento de la infraestructura o la latencia.  Front Door siempre enruta el tráfico al back-end más rápido y disponible (con un buen estado). Consulte la [arquitectura de enrutamiento](front-door-routing-architecture.md) y los [métodos de enrutamiento del tráfico](front-door-routing-methods.md) de Front Door para conocer más sobre el servicio.

## <a name="regional-load-balancing"></a>Equilibrio de carga regional
Application Gateway proporciona un controlador de entrega de aplicaciones (ADC) como servicio, que ofrece diversas funcionalidades de equilibrio de carga de nivel 7 para la aplicación. Permite a los clientes optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación TLS con mayor actividad de CPU a Application Gateway. Otras funcionalidades adicionales de enrutamiento de nivel 7 son la distribución equilibrada del tráfico entrante, la afinidad de sesiones basada en cookies, el enrutamiento basado en ruta de acceso de URL y la posibilidad de hospedar varios sitios web en una única instancia de Application Gateway. Application Gateway puede configurarse como un punto de conexión accesible desde Internet, un punto de conexión solo para uso interno o una combinación de ambos. Este servicio está totalmente administrado por Azure, lo que proporciona escalabilidad y alta disponibilidad. Cuenta con un amplio conjunto de funcionalidades de diagnóstico y registro, lo que facilita su administración.

Los servicios Load Balancer son una parte integral de la pila de Azure SDN; ofrecen servicios de equilibrio de carga de nivel 4 de alto rendimiento y baja latencia para todos los protocolos TCP y UDP. Puede configurar puntos de conexión con equilibrio de carga públicos o internos mediante la definición de reglas que asignan conexiones entrantes a los grupos de back-end. Gracias a la supervisión de sondeo de estado mediante TCP o HTTPS, puede ayudarle a administrar la disponibilidad del servicio.

## <a name="choosing-a-global-load-balancer"></a>Selección de un equilibrador de carga global
Al elegir un equilibrador de carga global entre Traffic Manager y Azure Front Door para el enrutamiento global, debe considerar las similitudes y las diferencias en los dos servicios.   Lo que proporcionan ambos servicios
- **Redundancia geográfica múltiple:** Si una región queda fuera de servicio, el tráfico se dirige sin problemas a la siguiente región más cercana, sin intervención del propietario de la aplicación.
- **Enrutamiento a la región más cercana:** El tráfico se enruta automáticamente a la región más cercana.

</br>En la tabla siguiente se describen las diferencias entre Traffic Manager y Azure Front Door:</br>

| Traffic Manager | Azure Front Door |
| --------------- | ------------------------ |
|**Cualquier protocolo:** Traffic Manager funciona en el nivel DNS, por lo que puede enrutar cualquier tipo de tráfico de red, como HTTP, TCP, UDP, etc. | **Aceleración de HTTP:** Front Door hace que el tráfico se redirija mediante proxy en el perímetro de la red de Microsoft. Las solicitudes HTTP/S verán mejoras de latencia y rendimiento, lo que reduce la latencia en la negociación de TLS.|
|**Enrutamiento local:** Con el enrutamiento en una capa de DNS, el tráfico siempre va de punto a punto.  El enrutamiento desde una sucursal al centro de datos local puede tomar una ruta directa, incluso en su propia red con Traffic Manager. | **Escalabilidad independiente:** Front Door funciona con las solicitudes HTTP, por lo que las solicitudes a rutas de dirección URL diferentes se pueden enrutar a diferentes grupos de servicios back-end y regionales (microservicios) en función de las reglas y el estado de cada microservicio de aplicación.|
|**Formato de facturación:** La facturación basada en DNS se escala con el número de usuarios y para servicios con más usuarios, con el fin de reducir el costo de un mayor uso. |**Seguridad alineada:** Front Door permite reglas como limitación de tasa y creación de listas ACL de IP para que pueda proteger los servidores back-end antes de que el tráfico llegue a la aplicación. 

</br>Se recomienda a los clientes que usen Front Door con sus cargas de trabajo HTTP debido a las ventajas de rendimiento, funcionamiento y seguridad que aporta. Traffic Manager y Front Door se pueden usar en paralelo para suministrar todo el tráfico de la aplicación. 

## <a name="building-with-azures-application-delivery-suite"></a>Creación con el conjunto de entrega de aplicaciones de Azure 
Se recomienda que todos los sitios web, las API y los servicios sean geográficamente redundantes, de modo que puedan entregar el tráfico a sus usuarios desde la ubicación más cercana siempre que sea posible.  La combinación de varios servicios de equilibrio de carga permite crear redundancia geográfica y de zona para maximizar la confiabilidad y el rendimiento.

En el diagrama siguiente se describe una arquitectura de ejemplo que usa una combinación de todos estos servicios para crear un servicio web global. El arquitecto usó Traffic Manager para enrutar el tráfico a los back-ends globales para la entrega de archivos y objetos. A la vez, usó Front Door para enrutar de forma global las rutas de dirección URL que coincidan con el patrón /store/* al servicio que han migrado a App Service. Por último, enrutó todas las demás solicitudes a instancias regionales de Application Gateway.

En cada región del servicio IaaS, el desarrollador de aplicaciones decidió que las direcciones URL que coincidan con el patrón /images/* se atiendan desde un grupo dedicado de máquinas virtuales. Este grupo de máquinas virtuales es diferente del resto de la granja de servidores web.

Además, el grupo de máquinas virtuales predeterminado que sirve el contenido dinámico debe comunicarse con una base de datos de back-end hospedada en un clúster de alta disponibilidad. Toda la implementación se configuró mediante Azure Resource Manager.

En el siguiente diagrama se muestra la arquitectura de este escenario:

:::image type="content" source="./media/front-door-lb-with-azure-app-delivery-suite/application-delivery-figure2.png" alt-text="Arquitectura detallada del conjunto de entrega de aplicaciones":::

> [!NOTE]
> Este ejemplo es solo una de las muchas configuraciones posibles de los servicios de equilibrio de carga que ofrece Azure. Traffic Manager, Front Door, Application Gateway y Load Balancer se pueden combinar de la forma que mejor se adapte a sus necesidades de equilibrio de carga. Por ejemplo, si la descarga de TLS/SSL o el procesamiento de Capa 7 no son necesarios, se puede usar Load Balancer en lugar de Application Gateway.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
