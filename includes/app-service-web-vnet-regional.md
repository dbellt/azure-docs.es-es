---
author: ccompy
ms.service: app-service-web
ms.topic: include
ms.date: 10/21/2020
ms.author: ccompy
ms.openlocfilehash: 821746856cb37781c8f6a2e58659ce7db43e1479
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105609485"
---
Cuando se utiliza la versión regional de Integración con red virtual, la aplicación puede acceder a:

* Recursos en una VNet de la misma región que la aplicación.
* Recursos de las VNet emparejadas con la VNet con la que se integra la aplicación.
* Servicios protegidos mediante puntos de conexión de servicio.
* Recursos de diferentes conexiones de Azure ExpressRoute.
* Recursos de la VNet con la que está integrado.
* Recursos en conexiones emparejadas, lo que incluye conexiones de Azure ExpressRoute.
* Puntos de conexión privados 

Si Integración con red virtual se utiliza con redes virtuales de la misma región, se pueden usar las siguientes características de redes de Azure:

* **Grupos de seguridad de red (NSG)** : el tráfico saliente se puede bloquear con un grupo de seguridad de red que se encuentre en la subred de integración. Las reglas de entrada no se aplican, ya que Integración con red virtual no se puede usar para proporcionar acceso de entrada a la aplicación.
* **Tablas de enrutamiento (UDR)** : puede colocar una tabla de enrutamiento en la subred de integración para enviar el tráfico de salida donde quiera.

De forma predeterminada, su aplicación solo enruta el tráfico de RFC 1918 a su VNet. Si desea enrutar todo el tráfico de salida a la red virtual, use los pasos siguientes para agregar el valor `WEBSITE_VNET_ROUTE_ALL` en la aplicación: 

1. Vaya a la interfaz de usuario de **Configuración** en el portal de la aplicación. Seleccione **New application setting** (Nueva configuración de la aplicación).
1. Escriba `WEBSITE_VNET_ROUTE_ALL` en el cuadro **Nombre** y `1` en el cuadro **Valor**.

   ![Configuración del valor de la aplicación][4]

1. Seleccione **Aceptar**.
1. Seleccione **Guardar**.

> [!NOTE]
> Si enruta todo el tráfico de salida a la VNet, este estará sujeto a los grupos de seguridad de red y las rutas definidas por los usuarios que se apliquen a la subred de integración. Cuando `WEBSITE_VNET_ROUTE_ALL` se establece en `1`, el tráfico de salida sigue enviándose desde las direcciones que se muestran en las propiedades de la aplicación, a menos que proporcione rutas que dirijan el tráfico a otro lugar.
> 
> La integración de red virtual regional no puede usar el puerto 25.

Existen algunas limitaciones cuando se la característica Integración con red virtual se utiliza con redes virtuales que están en la misma región:

* No se puede acceder a los recursos en las conexiones de emparejamiento global.
* La característica está disponible en todas las unidades de escalado de App Service de nivel Premium V2 y Premium V3. También está disponible en Estándar, pero solo desde las unidades de escalado de App Service más recientes. Si está en una unidad de escalado anterior, solo puede usar la característica desde un plan de App Service Premium V2. Si quiere estar seguro de que puede usar la característica en un plan de App Service Estándar, cree la aplicación en un plan de App Service Premium V3. Estos planes solo se admiten en las unidades de escalado más recientes. Si quiere, después puede reducir verticalmente.  
* La subred de integración solo puede usarla un plan de App Service.
* Las aplicaciones del plan Aislado que estén en una instancia de App Service Environment no pueden usar la característica.
* La característica requiere una subred sin usar que sea /28 o mayor en una red virtual de Azure Resource Manager.
* La aplicación y la VNet deben estar en la misma región.
* No puede eliminar una VNet con una aplicación integrada. Quite la integración antes de eliminar la VNet.
* Solo se puede tener una característica Integración con red virtual regional por plan de App Service. Varias aplicaciones en el mismo plan de App Service pueden usar la misma red virtual.
* No se puede cambiar la suscripción de una aplicación o un plan mientras haya una aplicación que use Integración con red virtual regional.
* La aplicación no puede resolver direcciones en Azure DNS Private Zones sin que se realicen cambios en la configuración.

La integración con red virtual depende de una subred dedicada. Al aprovisionar una subred, la subred de Azure pierde cinco direcciones IP desde el inicio. Se usa una dirección de la subred de integración para cada instancia del plan. Si escala la aplicación a cuatro instancias, se usan cuatro direcciones. 

Al escalar o reducir verticalmente el tamaño, el espacio de direcciones necesario se duplica durante un breve período de tiempo. Esto afecta a las instancias admitidas reales y disponibles para un tamaño de subred determinado. En la tabla siguiente se muestran las direcciones máximas disponibles por bloque CIDR y el impacto que esto tiene en la escala horizontal:

| Tamaño de bloque CIDR | Número máximo de direcciones disponibles | Escala horizontal máxima (instancias)<sup>*</sup> |
|-----------------|-------------------------|---------------------------------|
| /28             | 11                      | 5                               |
| /27             | 27                      | 13                              |
| /26             | 59                      | 29                              |

<sup>*</sup>Se da por supuesto que tendrá que escalar o reducir verticalmente el tamaño o la SKU en algún momento. 

Puesto que el tamaño de la subred no se puede cambiar después de la asignación, use una subred lo suficientemente grande como para dar cabida a cualquier escala que pueda alcanzar la aplicación. Para evitar problemas con la capacidad de la subred, debe usar /26 con 64 direcciones.  

Si quiere que las aplicaciones de otro plan lleguen a una VNet a la que ya están conectadas aplicaciones de otro plan, seleccione una subred distinta a la usada por la característica Integración con VNet ya existente.

La característica es totalmente compatible con aplicaciones para Windows y Linux, incluidos los [contenedores personalizados](../articles/app-service/quickstart-custom-container.md). Todos los comportamientos actúan del mismo modo entre aplicaciones para Windows y Linux.

### <a name="service-endpoints"></a>Puntos de conexión del servicio

La integración con red virtual regional le permite acceder a los servicios de Azure que están protegidos con puntos de conexión de servicio. Para acceder a un servicio protegido mediante puntos de conexión de servicio, debe hacer lo siguiente:

1. Configure la versión regional de integración con red virtual con su aplicación web para conectarse a una subred específica para la integración.
1. Vaya al servicio de destino y configure los puntos de conexión de servicio en la subred de integración.

### <a name="network-security-groups"></a>Grupos de seguridad de red

Puede usar grupos de seguridad de red para bloquear el tráfico de entrada y salida de los recursos de una VNet. Una aplicación que emplee la versión regional de Integración con red virtual puede usar un [grupo de seguridad de red][VNETnsg] para bloquear el tráfico de salida a los recursos de la VNet o Internet. Para bloquear el tráfico dirigido a direcciones públicas, debe tener el valor `WEBSITE_VNET_ROUTE_ALL` de la aplicación establecido en `1`. Las reglas de entrada de un grupo de seguridad de red no se aplican a la aplicación, ya que Integración con red virtual solo afecta al tráfico saliente de la aplicación.

Para controlar el trafico de entrada a la aplicación, use la característica Restricciones de acceso. Un grupo de seguridad de red que se aplique a la subred de integración está en vigor, con independencia de las rutas aplicadas a la subred de integración. Si `WEBSITE_VNET_ROUTE_ALL` está establecido en `1` y no tiene ninguna ruta que afecte al tráfico de direcciones públicas en la subred de integración, todo el tráfico de salida sigue estando sujeto a los grupos de seguridad de red asignados a la subred de integración. Si `WEBSITE_VNET_ROUTE_ALL` no se ha definido, los grupos de seguridad de red solo se aplican al tráfico de RFC 1918.

### <a name="routes"></a>Rutas

Las tablas de enrutamiento se pueden usar para enrutar el tráfico de salida de la aplicación al lugar que se desee. De forma predeterminada, las tablas de enrutamiento solo afectan al tráfico de destino de RFC 1918. Al establecer`WEBSITE_VNET_ROUTE_ALL` en `1`, todas sus llamadas salientes se ven afectadas. Las rutas que se establecen en la subred de integración no afectan a las respuestas a las solicitudes de entrada de la aplicación. Los destinos más habituales suelen ser puertas de enlace o dispositivos de firewall.

Si desea enrutar todo el tráfico de salida del entorno local, puede utilizar una tabla de rutas para enviar el tráfico de salida a la puerta de enlace de ExpressRoute. Si no enruta el tráfico a una puerta de enlace, asegúrese de establecer las rutas en la red externa para poder enviar de vuelta las respuestas.

Las rutas del Protocolo de puerta de enlace de borde (BGP) también afectan al tráfico de la aplicación. Si tiene rutas de BGP cuyo origen es algo similar a una puerta de enlace de ExpressRoute, el tráfico de salida de la aplicación se verá afectado. De forma predeterminada, las rutas de BGP solo afectan al tráfico de destino de RFC 1918. Si `WEBSITE_VNET_ROUTE_ALL` está establecido en `1`, todo el tráfico de salida puede verse afectado por las rutas de BGP.

### <a name="azure-dns-private-zones"></a>Zonas privadas de Azure DNS 

Una vez que la aplicación se integra con la red virtual, usa el mismo servidor DNS que el configurado para la red virtual. De forma predeterminada, la aplicación no funcionará con zonas privadas de Azure DNS. Para que lo haga es preciso agregar la siguiente configuración de la aplicación:

1. `WEBSITE_DNS_SERVER` con el valor `168.63.129.16`
1. `WEBSITE_VNET_ROUTE_ALL` con el valor `1`

Esta configuración envía todas las llamadas salientes desde la aplicación a la red virtual y permite que la aplicación tenga acceso a una zona privada de Azure DNS. Con esta configuración, su aplicación puede usar Azure DNS consultando la zona DNS privada en el nivel de trabajo.  

### <a name="private-endpoints"></a>Puntos de conexión privados

Si quiere realizar llamadas a [Puntos de conexión privados][privateendpoints], debe asegurarse de que las búsquedas de DNS se resuelvan en el punto de conexión privado. Puede aplicar este comportamiento de una de las siguientes formas: 

* Realizar la integración con zonas privadas de Azure DNS. Cuando la red virtual no tiene un servidor DNS personalizado, esto se hace automáticamente.
* Administrar el punto de conexión privado en el servidor DNS que usa la aplicación. Para ello, debe conocer la dirección del punto de conexión privado y, a continuación, apuntar el punto de conexión al que está intentando acceder a esa dirección con un registro A.
* Configurar su propio servidor DNS para reenviarlo a zonas privadas de Azure DNS.

<!--Image references-->
[4]: ../includes/media/web-sites-integrate-with-vnet/vnetint-appsetting.png

<!--Links-->
[VNETnsg]: /azure/virtual-network/security-overview/
[privateendpoints]: ../articles/app-service/networking/private-endpoint.md
