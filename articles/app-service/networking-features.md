---
title: Características de red
description: Obtenga información sobre las características de red de Azure App Service y aprenda qué características necesita para seguridad u otras funcionalidades.
author: ccompy
ms.assetid: 5c61eed1-1ad1-4191-9f71-906d610ee5b7
ms.topic: article
ms.date: 03/26/2021
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 7410793a429395b7f1b1e826c66cb2647eb77cd2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367716"
---
# <a name="app-service-networking-features"></a>Características de redes de App Service

Puede implementar aplicaciones de Azure App Service de varias maneras. De manera predeterminada, puede acceder a las aplicaciones hospedadas en App Service directamente a través de la Internet, y estas solo pueden acceder a los puntos de conexión hospedados en Internet. No obstante, en el caso de muchas aplicaciones, debe controlar el tráfico de red entrante y saliente. Hay varias características en App Service para ayudarle a satisfacer esas necesidades. El desafío consiste en saber qué característica usar para resolver un problema concreto. Este artículo le ayudará a determinar qué característica debe usar, en función de algunos casos de uso de ejemplo.

Hay dos tipos de implementación principales para Azure App Service: 
- El servicio público multiinquilino hospeda planes de App Service en las SKU de precios Gratis, Compartido, Básico, Estándar, Premium, PremiumV2 y PremiumV3. 
- App Service Environment (ASE) de un único inquilino hospeda planes de App Service de SKU aislada directamente en su red virtual de Azure. 

Las características que puede usar dependerán de si está en el servicio multiinquilino o en un entorno de ASE. 

> [!NOTE]
> Las características de redes no están disponibles para las [aplicaciones implementadas en Azure Arc](overview-arc-integration.md).

## <a name="multitenant-app-service-networking-features"></a>Características de red de App Service multiinquilino 

Azure App Service es un sistema distribuido. Los roles que controlan las solicitudes HTTP o HTTPS entrantes se denominan *servidores front-end*. Los roles que hospedan la carga de trabajo del cliente se denominan *roles de trabajo*. Todos los roles de una implementación de App Service existen en una red multiinquilino. Puesto que hay muchos clientes diferentes en la misma unidad de escalado de App Service, no puede conectar la red de App Service directamente a su red. 

En lugar de conectarse a redes, necesita características para administrar los diversos aspectos de la comunicación de aplicaciones. Las características que controlan las solicitudes *a* la aplicación no se pueden usar para solucionar problemas al realizar llamadas *desde* la aplicación. Del mismo modo, las características que resuelven problemas para las llamadas desde la aplicación no se pueden usar para solucionar problemas de llamadas a la aplicación.  

| Características de entrada | Características de salida |
|---------------------|-------------------|
| Dirección asignada a las aplicaciones | conexiones híbridas |
| Restricciones de acceso              | Integración con red virtual con requisito de puerta de enlace |
| Puntos de conexión del servicio | Integración con red virtual |
| Puntos de conexión privados ||

Además de las excepciones indicadas, puede usar todas estas características juntas. Puede combinar las características para solucionar los problemas.

## <a name="use-cases-and-features"></a>Casos de uso y características

Para un caso de uso determinado, podría haber varias maneras de solucionar el problema. Elegir la mejor característica a veces va más allá del caso de uso. Los siguientes casos de uso de entrada sugieren cómo usar las características de redes de App Service para solucionar problemas relacionados con el control del tráfico que va a la aplicación.
 
| Caso de uso de entrada | Característica |
|---------------------|-------------------|
| Compatibilidad con las necesidades de SSL basado en IP de la aplicación | Dirección asignada a las aplicaciones |
| Compatibilidad con la dirección entrante dedicada no compartida para la aplicación | Dirección asignada a las aplicaciones |
| Restricción del acceso a la aplicación desde un conjunto de direcciones bien definidas | Restricciones de acceso              |
| Restricción del acceso a la aplicación desde los recursos en una red virtual | Puntos de conexión del servicio </br> ASE de ILB </br> Puntos de conexión privados |
| Exposición de la aplicación en una dirección IP privada de la red virtual | ASE de ILB </br> Puntos de conexión privados </br> Dirección IP privada para el tráfico de entrada en una instancia de Application Gateway con puntos de conexión de servicio |
| Protección de la aplicación con un firewall de aplicaciones web (WAF) | Application Gateway y ASE de ILB </br> Application Gateway con puntos de conexión privados </br> Application Gateway con puntos de conexión de servicio </br> Azure Front Door con restricciones de acceso |
| Equilibrio de la carga del tráfico a las aplicaciones en diferentes regiones | Azure Front Door con restricciones de acceso | 
| Equilibrar la carga del tráfico en la misma región | [Application Gateway con puntos de conexión de servicio][appgwserviceendpoints] | 

Los siguientes casos de uso de salida sugieren cómo usar las características de redes de App Service para resolver las necesidades de acceso de salida de la aplicación:

| Caso de uso de salida | Característica |
|---------------------|-------------------|
| Acceso a los recursos de una red virtual de Azure en la misma región | Integración con red virtual </br> ASE |
| Acceso a los recursos de una red virtual de Azure en una región diferente | Integración con red virtual con requisito de puerta de enlace </br> ASE y emparejamiento de red virtual |
| Acceso a los recursos protegidos por puntos de conexión de servicio | Integración con red virtual </br> ASE |
| Acceso a los recursos de una red privada que no está conectada a Azure | conexiones híbridas |
| Acceso a los recursos mediante circuitos de Azure ExpressRoute | Integración con red virtual </br> ASE | 
| Protección del tráfico saliente de su aplicación web | Integración con red virtual y grupos de seguridad de red </br> ASE | 
| Enrutamiento del tráfico saliente de su aplicación web | Integración con red virtual y tablas de ruta </br> ASE | 


### <a name="default-networking-behavior"></a>Comportamiento predeterminado de la redes

Las unidades de escalado de Azure App Service admiten muchos clientes en cada implementación. Los planes de SKU Gratis y Compartido hospedan cargas de trabajo de clientes en roles de trabajo multiinquilino. El plan Básico y los planes superiores hospedan cargas de trabajo dedicadas a un único plan de App Service. Si tiene un plan de App Service Estándar, todas las aplicaciones de ese plan se ejecutarán en el mismo rol de trabajo. Si escala horizontalmente el rol de trabajo, todas las aplicaciones de ese plan de App Service se replicarán en un rol de trabajo nuevo para cada instancia del plan de App Service. 

#### <a name="outbound-addresses"></a>Direcciones de salida

Las máquinas virtuales de trabajo se desglosan en gran medida a través de los planes de App Service. Los planes Gratis, Compartido, Básico, Estándar y Premium usan el mismo tipo de máquina virtual de trabajo. El plan PremiumV2 usa otro tipo de máquina virtual. PremiumV3 también usa otro tipo de máquina virtual. Cuando se cambia la familia de máquinas virtuales, obtiene un conjunto diferente de direcciones de salida. Si realiza el escalado de Estándar a PremiumV2, las direcciones de salida cambian. Lo mismo sucede si escala de PremiumV2 a PremiumV3. En algunas unidades de escalado antiguas, las direcciones de entrada y salida se cambiarán al escalar de Estándar a PremiumV2. 

Hay varias direcciones que se usan para las llamadas salientes. Las direcciones de salida que usa la aplicación para realizar llamadas salientes se muestran en las propiedades de la aplicación. Todas las aplicaciones que se ejecutan en la misma familia de máquinas virtuales de trabajo de la implementación de App Service comparten estas direcciones. Si quiere ver todas las direcciones que puede usar la aplicación en una unidad de escalado, existe una propiedad denominada `possibleOutboundAddresses` que puede enumerarlas. 

![Captura de pantalla que muestra las propiedades de la aplicación.](media/networking-features/app-properties.png)

App Service tiene varios puntos de conexión que se usan para administrar el servicio.  Dichas direcciones se publican en un documento independiente y también se encuentran en la etiqueta de servicio de direcciones IP `AppServiceManagement`. La etiqueta `AppServiceManagement` se usa solo en los entornos de App Service Environment en los que es necesario permitir dicho tráfico. En la etiqueta de servicio IP de `AppService` se realiza un seguimiento de las direcciones de entrada de App Service. No hay ninguna etiqueta de servicio IP que contenga las direcciones de salida usadas por App Service. 

![Diagrama que muestra el tráfico entrante y saliente de App Service.](media/networking-features/default-behavior.png)

### <a name="app-assigned-address"></a>Dirección asignada a las aplicaciones 

La característica de direcciones asignadas a las aplicaciones es una rama de la funcionalidad SSL basada en IP. Puede acceder a ella mediante la configuración de SSL en la aplicación. Puede usar esta característica para las llamadas SSL basadas en IP. También puede usarla para asignar a la aplicación una dirección exclusiva. 

![Diagrama que muestra la dirección asignada a la aplicación.](media/networking-features/app-assigned-address.png)

Cuando se utiliza una dirección asignada a la aplicación, el tráfico sigue pasando por los mismos roles de front-end que controlan todo el tráfico entrante en la unidad de escalado de App Service. No obstante, la dirección que se asigna a la aplicación solo la utiliza la aplicación. Casos de uso para esta característica:

* Compatibilidad con las necesidades de SSL basado en IP de la aplicación.
* Configuración de una dirección dedicada para la aplicación que no sea compartida.

Para obtener información sobre cómo establecer una dirección en la aplicación, consulte [Adición de un certificado TLS/SSL en Azure App Service][appassignedaddress]. 

### <a name="access-restrictions"></a>Restricciones de acceso              

Las restricciones de acceso le permiten filtrar las solicitudes *entrantes*. La acción de filtrado tiene lugar en los roles de front-end, que están en un nivel superior al de los roles de trabajo en los que se ejecutan las aplicaciones. Ya que los roles de front-end están en un nivel superior al de los roles de trabajo, puede pensar en las restricciones de acceso como una protección de nivel de red para las aplicaciones. 

Esta característica le permite crear una lista de reglas permitidas y denegadas que se evalúan en orden de prioridad. Es similar a la característica Grupo de seguridad de red (NSG) de las redes de Azure. Puede usar esta característica en un entorno de ASE o en el servicio multiinquilino. Cuando la usa con un ASE de ILB o punto de conexión privado, puede restringir el acceso desde bloques de direcciones privadas.
> [!NOTE]
> Se pueden configurar hasta 512 reglas de restricción de acceso por cada aplicación. 

![Diagrama que muestra las restricciones de acceso.](media/networking-features/access-restrictions.png)

#### <a name="ip-based-access-restriction-rules"></a>Reglas de restricción de acceso basadas en IP

La característica Restricciones de acceso basadas en IP es de utilidad cuando quiere restringir las direcciones IP que se pueden usar para llegar a la aplicación. Se admiten tanto IPv4 como IPv6. Los siguientes son algunos casos de uso para esta característica:
* Restricción del acceso a la aplicación desde un conjunto de direcciones bien definidas. 
* Restrinja el acceso al tráfico que llega a través de un servicio de equilibrio de carga externo u otros dispositivos de red con direcciones IP de salida conocidas. 

Para obtener información sobre cómo habilitar esta característica, consulte [Configuración de restricciones de acceso][iprestrictions].

> [!NOTE]
> Las reglas de restricción de acceso basadas en IP solo controlan los intervalos de direcciones de red virtual cuando la aplicación se encuentra en una instancia de App Service Environment. Si su aplicación se encuentra en el servicio multiinquilino, debe usar [puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) para restringir el tráfico a determinadas subredes de la red virtual.

#### <a name="access-restriction-rules-based-on-service-endpoints"></a>Reglas de restricciones de acceso basadas en puntos de conexión de servicio 

Los puntos de conexión de servicio le permiten bloquear el acceso *entrante* a la aplicación, de modo que la dirección de origen debe proceder de un conjunto de subredes que haya seleccionado. Esta característica funciona en conjunto con las restricciones de acceso IP. Los puntos de conexión de servicio no son compatibles con la depuración remota. Si quiere usar la depuración remota con la aplicación, el cliente no puede estar en una subred que tenga los puntos de conexión de servicio habilitados. El proceso para establecer los puntos de conexión de servicio es similar al proceso para establecer las restricciones de acceso basadas en IP. Puede crear una lista de reglas de acceso tipo permitir/denegar que incluya las direcciones públicas y subredes de sus redes virtuales. 

Los siguientes son algunos casos de uso para esta característica:

* Configurar una puerta de enlace de aplicación con la aplicación para bloquear el tráfico entrante a la aplicación.
* Restringir el acceso a la aplicación a los recursos en la red virtual. Estos recursos pueden ser máquinas virtuales, entornos de ASE o incluso otras aplicaciones que usen la integración con red virtual. 

![Diagrama que muestra el uso de puntos de conexión de servicio con Application Gateway.](media/networking-features/service-endpoints-appgw.png)

Para más información sobre la configuración de puntos de conexión de servicio con la aplicación, consulte [Restricciones de acceso de Azure App Service][serviceendpoints].

#### <a name="access-restriction-rules-based-on-service-tags"></a>Reglas de restricciones de acceso basadas en etiquetas de servicio

Las [etiquetas de servicio de Azure][servicetags] son conjuntos bien definidos de direcciones IP de los servicios de Azure. Las etiquetas de servicio agrupan los intervalos IP que se usan en varios servicios de Azure y a menudo también se limitan a regiones específicas. Así, puede filtrar el tráfico *entrante* desde servicios específicos de Azure. 

Para obtener una lista completa de etiquetas y más información, haga clic en el vínculo de la etiqueta de servicio anterior. Para obtener información sobre cómo habilitar esta característica, consulte [Configuración de restricciones de acceso][iprestrictions].

#### <a name="http-header-filtering-for-access-restriction-rules"></a>Filtrado de encabezados HTTP para reglas de restricción de acceso

Para cada regla de restricción de acceso, puede agregar el filtrado adicional de encabezados HTTP. Esta acción le permite inspeccionar más a fondo la solicitud entrante y filtrar en función de los valores específicos de los encabezados HTTP. Cada encabezado puede tener hasta 8 valores por regla. Actualmente se admite la siguiente lista de encabezados HTTP: 
* X-Forwarded-For
* X-Forwarded-Host
* X-Azure-FDID
* X-FD-HealthProbe

Algunos casos de uso del filtrado de encabezados HTTP son:
* Restringir el acceso al tráfico procedente de los servidores proxy que reenvían el nombre de host.
* Restringir el acceso a una instancia específica de Azure Front Door con una regla de etiqueta de servicio y una restricción de encabezado X-Azure-FDID.

### <a name="private-endpoint"></a>Punto de conexión privado

Un punto de conexión privado es una interfaz de red que le conecta de manera privada y segura a la aplicación web con Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar la aplicación web de manera eficaz a su red virtual. Esta característica es solo para flujos *entrantes* en la aplicación web.
Para obtener más información, consulte [Uso de puntos de conexión privados para una aplicación web de Azure][privateendpoints].

Los siguientes son algunos casos de uso para esta característica:

* Restricción del acceso a la aplicación desde los recursos en una red virtual. 
* Exposición de la aplicación en una dirección IP privada de la red virtual. 
* Protección de la aplicación con un WAF.

Los puntos de conexión privados impiden la filtración de datos porque que lo único a lo que puede acceder a través del punto de conexión privado es a la aplicación con la que está configurado. 
 
### <a name="hybrid-connections"></a>conexiones híbridas

Conexiones híbridas de App Service permite que las aplicaciones realicen llamadas *salientes* a puntos de conexión TCP especificados. El punto de conexión puede ser local, estar en una red virtual o en cualquier lugar que permita el tráfico saliente a Azure en el puerto 443. Para usar esta característica, debe instalar un agente de retransmisión llamado Administrador de conexiones híbridas (HCM) en un host con Windows Server 2012 o posterior. El Administrador de conexiones híbridas debe ser capaz de conectarse a Azure Relay en el puerto 443. Puede descargar el Administrador de conexiones híbridas desde la interfaz de usuario de Conexiones híbridas de App Service en el portal. 

![Diagrama que muestra el flujo de red de Conexiones híbridas.](media/networking-features/hybrid-connections.png)

Las Conexiones híbridas de App Service están basadas en la funcionalidad Conexiones híbridas de Azure Relay. App Service usa una forma especializada de la característica que solo admite la realización de llamadas salientes desde la aplicación en un host y puerto TCP. La resolución de este host y puerto solo es necesaria en el host en el que está instalado el Administrador de conexiones híbridas. 

Cuando la aplicación, en App Service, realiza una búsqueda DNS sobre el host y puerto definidos en la conexión híbrida, el tráfico se redirige automáticamente a través de la conexión híbrida y fuera del Administrador de conexiones híbridas. Para obtener más información, consulte [Hybrid Connections de Azure App Service][hybridconn].

Esta característica se utiliza normalmente para:

* Acceder a recursos en redes privadas que no están conectadas a Azure con una VPN o ExpressRoute.
* Admitir la migración de aplicaciones locales a App Service sin tener que mover también las bases de datos necesarias.  
* Proporcionar acceso con seguridad mejorada a un único host y puerto por cada conexión híbrida. La mayoría de las características de red abren el acceso a una red. Con las Conexiones híbridas, solo se puede comunicar con un único host y puerto.
* Tratar escenarios no cubiertos por otros métodos de conectividad de salida.
* Realizar desarrollos en App Service de manera que las aplicaciones puedan usar fácilmente los recursos locales. 

Dado que esta característica permite el acceso a los recursos locales sin un agujero en el firewall de entrada, es popular entre los desarrolladores. Las otras características de redes de App Service de salida están relacionadas con Azure Virtual Network. La característica Conexiones híbridas no necesita pasar por una red virtual. Se puede usar para una amplia variedad de requisitos de red. 

Tenga en cuenta que la característica Conexiones híbridas de App Service no detecta lo que está haciendo sobre ella. Por lo que puede usarla para acceder a una base de datos, un servicio web o un socket TCP arbitrario en un sistema central. La característica esencialmente tuneliza paquetes TCP. 

Aunque Conexiones híbridas es popular para el desarrollo, también se usa en aplicaciones de producción. Es muy útil para acceder a un servicio web o una base de datos, pero no es adecuada para situaciones que implican la creación de muchas conexiones. 

### <a name="gateway-required-vnet-integration"></a>Integración con red virtual con requisito de puerta de enlace 

La integración con red virtual de App Service requerida por la puerta de enlace permite a la aplicación realizar solicitudes *salientes* a una red virtual de Azure. La característica funciona mediante la conexión del host en el que se ejecuta la aplicación a una puerta de enlace de red virtual en la red virtual a través de una conexión VPN de punto a sitio. Al configurar la característica, la aplicación obtiene una de las direcciones de punto a sitio asignadas a cada instancia. Esta característica le permite acceder a recursos de redes virtuales clásicas o de Azure Resource Manager en cualquier región. 

![Diagrama que muestra la integración con red virtual requerida por la puerta de enlace.](media/networking-features/gw-vnet-integration.png)

Esta característica soluciona el problema de acceso a los recursos de otras redes virtuales. Incluso se puede usar para conectarse a través de una red virtual a otras redes virtuales o entornos locales. No funciona con redes virtuales conectadas mediante ExpressRoute, aunque funciona con ñas redes conectadas mediante VPN de sitio a sitio. Normalmente, no es adecuado usar esta característica desde una aplicación en un entorno de App Service Environment (ASE), porque la instancia de ASE ya está en su red virtual. Casos de uso para esta característica:

* Acceso a los recursos en direcciones IP privadas de sus redes virtuales de Azure. 
* Acceso a los recursos locales si hay una conexión VPN de sitio a sitio. 
* Acceso a recursos en redes virtuales emparejadas. 

Cuando esta característica está habilitada, la aplicación usará el servidor DNS configurado en la red virtual de destino. Para obtener más información sobre esta característica, consulte [Integración con red virtual de App Service][vnetintegrationp2s]. 

### <a name="vnet-integration"></a>Integración con red virtual

La integración con red virtual requerida por la puerta de enlace es útil, pero no resuelve el problema de acceso a los recursos a través de ExpressRoute. Más allá de la necesidad de conexiones a través de ExpressRoute, las aplicaciones necesitan la capacidad de realizar llamadas a servicios protegidos por puntos de conexión de servicio. Otra funcionalidad de integración con red virtual puede satisfacer estas necesidades. 

La nueva característica Integración con red virtual le permite colocar el back-end de la aplicación en una subred de una red virtual de Resource Manager en la misma región que la aplicación. Esta característica no está disponible desde un entorno de App Service Environment, que ya se encuentra en una red virtual. Casos de uso para esta característica:

* Acceso a recursos en redes virtuales de Resource Manager de la misma región.
* Acceso a los recursos que están protegidos por puntos de conexión de servicio. 
* Acceso a recursos que son accesibles a través de conexiones ExpressRoute o VPN.
* Ayuda para proteger todo el tráfico saliente. 
* Tunelización forzada de todo el tráfico saliente. 

![Diagrama que muestra la integración con red virtual.](media/networking-features/vnet-integration.png)

Para obtener más información, consulte [Integración con red virtual de Azure App Service][vnetintegration].

### <a name="app-service-environment"></a>Entorno de App Service 

Un entorno de App Service Environment (ASE) es una implementación de inquilino único de Azure App Service que se ejecuta en su red virtual. Algunos casos de uso para esta característica:

* Acceso a recursos en la red virtual.
* Acceso a recursos a través de ExpressRoute.
* Exposición de las aplicaciones con una dirección privada en la red virtual. 
* Acceso a los recursos a través de puntos de conexión de servicio. 

Con un entorno de ASE, no es necesario utilizar características como la integración con red virtual o los puntos de conexión de servicio, porque la instancia de ASE ya está en la red virtual. Si quiere acceder a recursos como SQL o Azure Storage mediante puntos de conexión de servicio, habilite puntos de conexión de servicio en la subred de ASE. Si quiere acceder a los recursos de la red virtual, no es necesario realizar ninguna configuración adicional. Si quiere acceder a recursos a través de ExpressRoute, ya está en la red virtual y no es necesario configurar nada en el entorno de ASE ni en las aplicaciones en él. 

Dado que las aplicaciones de un ASE de ILB pueden exponerse en una dirección IP privada, puede agregar fácilmente dispositivos WAF para exponer solo las aplicaciones que quiera a Internet y ayudar a proteger el resto. Esta característica puede ayudar a facilitar el desarrollo de aplicaciones de varios niveles. 

Actualmente, algunos elementos no son posibles desde el servicio multiinquilino, aunque lo son desde un ASE. A continuación se muestran algunos ejemplos:

* Exponer las aplicaciones en una dirección IP privada.
* Ayudar a proteger todo el tráfico saliente con controles de red que no forman parte de la aplicación.
* Hospedar sus aplicaciones en un servicio de inquilino único. 
* Escalar verticalmente a muchas más instancias de las posibles en el servicio multiinquilino. 
* Cargar certificados de cliente de entidad de certificación privada para que las aplicaciones con puntos de conexión protegidos por entidad de certificación privada los usen.
* Uso forzado de TLS 1.1 en todas las aplicaciones hospedadas en el sistema sin ninguna posibilidad de deshabilitarlo en el nivel de aplicación. 
* Proporcionar una dirección de salida dedicada para todas las aplicaciones del entorno de ASE que no se comparten con los clientes. 

![Diagrama que muestra un entorno de ASE en una red Virtual.](media/networking-features/app-service-environment.png)

El entorno de ASE proporciona la mejor opción para el hospedaje de una aplicación aislada y dedicada, aunque implica algunos desafíos de administración. Algunos aspectos a tener en cuenta antes de usar una instancia de ASE operativa:
 
 * Un entorno de ASE se ejecuta dentro de la red virtual, pero tiene dependencias fuera de esta. Se deben permitir esas dependencias. Para obtener más información, consulte [Consideraciones de red para un entorno de App Service Environment][networkinfo].
 * Un entorno de ASE no se escala inmediatamente a la par del servicio multiinquilino. Es necesario anticipar las necesidades de escalado en lugar de escalar de forma reactiva. 
 * Una instancia de ASE tiene un mayor costo por adelantado. Para sacar el máximo partido de su entorno de ASE, debe planear poner muchas cargas de trabajo en una instancia de ASE, en lugar de usarlo para iniciativas pequeñas.
 * Las aplicaciones de un entorno de ASE no pueden restringir el acceso de manera selectiva a algunas aplicaciones del ASE y no a otras.
 * Un entorno de ASE está en una subred y las reglas de red se aplican a todo el tráfico hacia y desde dicho ASE. Si quiere asignar reglas de tráfico entrante para una sola aplicación, use las restricciones de acceso. 

## <a name="combining-features"></a>Combinación de características 

Las características indicadas para el servicio multiinquilino se pueden usar en conjunto para resolver casos de uso más elaborados. Aquí se describen dos de los casos de uso más comunes, pero son solo ejemplos. Al comprender lo que hacen las distintas características, puede satisfacer casi todas las necesidades de arquitectura del sistema.

### <a name="place-an-app-into-a-virtual-network"></a>Colocación de una aplicación en una red virtual

Quizá se pregunte cómo poner una aplicación en una red virtual. Si coloca la aplicación en una red virtual, los puntos de conexión de entrada y salida de la aplicación se ubicarán dentro de la red virtual. Un entorno de ASE es la mejor manera de resolver este problema. No obstante, puede satisfacer la mayoría de sus necesidades en el servicio multiinquilino mediante la combinación de características. Por ejemplo, puede hospedar aplicaciones solo de intranet con direcciones privadas de entrada y de salida de las siguientes maneras:

* Crear una puerta de enlace de aplicación con direcciones de entrada y de salida privadas.
* Proteger el tráfico entrante a la aplicación con puntos de conexión de servicio. 
* Usar la nueva característica de integración con red virtual para que el back-end de la aplicación esté en la red virtual. 

Este estilo de implementación no le proporcionará una dirección dedicada para el tráfico saliente a Internet, ni la posibilidad de bloquear todo el tráfico saliente de la aplicación. Le aportará una buena parte de lo que, de otro modo, solo obtendría con un entorno de ASE. 

### <a name="create-multitier-applications"></a>Creación de aplicaciones de varios niveles

Una aplicación de varios niveles es una aplicación en la que solo se puede acceder a las aplicaciones de back-end de la API desde el nivel de front-end. Hay dos maneras de crear una aplicación de varios niveles. Ambas comienzan por el uso de la integración con red virtual para conectar la aplicación web de front-end a una subred de una red virtual. De este modo, la aplicación web podrá realizar llamadas a la red virtual. Una vez que la aplicación de front-end esté conectada a la red virtual, deberá decidir cómo bloquear el acceso a la aplicación de API. Puede:

* Hospedar la aplicación de front-end y de API en el mismo entorno de ASE de ILB y exponer la aplicación de front-end a Internet mediante una puerta de enlace de aplicación.
* Hospedar el front-end en el servicio multiinquilino y el back-end en un ASE de ILB.
* Hospedar la aplicación de front-end y de API en el servicio multiinquilino.

Si hospeda la aplicación de front-end y de API para una aplicación de varios niveles, puede:

- Exponer la aplicación de API mediante puntos de conexión privados en la red virtual:

  ![Diagrama que muestra el uso de puntos de conexión privados en una aplicación de dos niveles.](media/networking-features/multi-tier-app-private-endpoint.png)

- Usar puntos de conexión de servicio para garantizar que el tráfico entrante a la aplicación de API solo proceda de la subred usada por la aplicación web de front-end:

  ![Diagrama que muestra el uso de puntos de conexión de servicio para proteger una aplicación.](media/networking-features/multi-tier-app.png)

Estas son algunas consideraciones que le ayudarán a decidir qué método debe usar:

* Cuando use puntos de conexión de servicio, solo tendrá que proteger el tráfico que va de la aplicación de API a la subred de integración. De este modo, se protege la aplicación de API, pero aún es posible una filtración de datos de la aplicación de front-end a otras aplicaciones en el servicio de la aplicación.
* Cuando se usan puntos de conexión privados, hay dos subredes en juego, lo que aumenta la complejidad. Además, el punto de conexión privado es un recurso de nivel superior y supone una sobrecarga administrativa. La ventaja de usar puntos de conexión privados es que no existe la posibilidad de una filtración de datos. 

Cualquiera de los métodos funcionará con varias aplicaciones de front-end. A pequeña escala, los puntos de conexión de servicio son más fáciles de usar porque solo hay que habilitarlos para la aplicación de API en la subred de integración de front-end. A medida que agregue más aplicaciones de front-end, tendrá que ajustar cada aplicación de API para incluir puntos de conexión de servicio con la subred de integración. Cuando usa puntos de conexión privados, la complejidad es mayor, pero no tiene que cambiar nada en las aplicaciones de API después de establecer un punto de conexión privado. 

### <a name="line-of-business-applications"></a>Aplicaciones de línea de negocio

Las aplicaciones de línea de negocio (LOB) son aplicaciones internas que normalmente no están expuestas para el acceso desde Internet. Estas aplicaciones se invocan desde redes corporativas internas donde el acceso puede estar controlado de forma estricta. Si usa un ASE de ILB, es fácil hospedar las aplicaciones de línea de negocio. Si usa el servicio multiinquilino, puede usar puntos de conexión privados o puntos de conexión de servicio combinados con una puerta de enlace de aplicación. Hay dos razones para usar una puerta de enlace de aplicación con puntos de conexión de servicio, en lugar de puntos de conexión privados:
* Necesita la protección de WAF en sus aplicaciones de línea de negocio.
* Quiere equilibrar la carga entre varias instancias de las aplicaciones de línea de negocio.

Si ninguna de estas necesidades es aplicable, es mejor usar puntos de conexión privados. Con los puntos de conexión privados disponibles en App Service, puede exponer las aplicaciones de direcciones privadas de la red virtual. Se puede comunicar con el punto de conexión privado colocado en la red virtual a través de conexiones de ExpressRoute y VPN. 

La configuración de puntos de conexión privados expondrá las aplicaciones de una dirección privada, pero tendrá que configurar DNS para comunicarse con esa dirección desde el entorno local. Para que esta configuración funcione, deberá reenviar la zona privada de Azure DNS que contiene los puntos de conexión privados a los servidores DNS locales. Las zonas privadas de Azure DNS no admiten el reenvío de zona, pero puede admitirlo mediante el uso de un servidor DNS para ese fin. La plantilla [Reenviador de DNS](https://azure.microsoft.com/resources/templates/301-dns-forwarder/) facilita el reenvío de la zona privada de Azure DNS a los servidores DNS locales.

## <a name="app-service-ports"></a>Puertos de App Service

Si examina App Service, encontrará varios puertos que se exponen para las conexiones entrantes. No hay ninguna manera de bloquear o controlar el acceso a estos puertos en el servicio multiinquilino. La siguiente es la lista de puertos expuestos:

| Uso | Puerto o puertos |
|----------|-------------|
|  HTTP/HTTPS  | 80, 443 |
|  Administración | 454, 455 |
|  FTP/FTPS    | 21, 990, 10001-10020 |
|  Depuración remota en Visual Studio  |  4020, 4022, 4024 |
|  Servicio Web Deploy | 8172 |
|  Uso de la infraestructura | 7654, 1221 |

<!--Links-->
[appassignedaddress]: ./configure-ssl-certificate.md
[iprestrictions]: ./app-service-ip-restrictions.md
[serviceendpoints]: ./app-service-ip-restrictions.md
[hybridconn]: ./app-service-hybrid-connections.md
[vnetintegrationp2s]: ./web-sites-integrate-with-vnet.md
[vnetintegration]: ./web-sites-integrate-with-vnet.md
[networkinfo]: ./environment/network-info.md
[appgwserviceendpoints]: ./networking/app-gateway-with-service-endpoints.md
[privateendpoints]: ./networking/private-endpoint.md
[servicetags]: ../virtual-network/service-tags-overview.md