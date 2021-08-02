---
title: Configuración de la infraestructura de Azure Application Gateway
description: En este artículo se describe cómo configurar la infraestructura de Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 06/14/2021
ms.author: surmb
ms.openlocfilehash: 785741c029fa3b44fffca5140906689f478fb247
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081234"
---
# <a name="application-gateway-infrastructure-configuration"></a>Configuración de la infraestructura de Azure Application Gateway

La infraestructura de Application Gateway incluye la red virtual, las subredes, los grupos de seguridad de red y las rutas definidas por el usuario.

## <a name="virtual-network-and-dedicated-subnet"></a>Red virtual y subred dedicada

Una puerta de enlace de aplicaciones es una implementación dedicada en su red virtual. Dentro de la red virtual, es necesaria una subred dedicada para la puerta de enlace de aplicaciones. Puede tener varias instancias de una implementación de puerta de enlace de aplicaciones determinada en una subred. También se pueden implementar otras puertas de enlace de aplicaciones en la subred. Pero no se puede implementar ningún otro recurso en la subred de la puerta de enlace de aplicaciones. No se pueden combinar Application Gateway Standard y Standard_v2 en la misma subred.

> [!NOTE]
> Actualmente, no se admiten [directivas de punto de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoint-policies-overview.md) en una subred de Application Gateway.

### <a name="size-of-the-subnet"></a>Tamaño de la subred

Application Gateway usa una dirección IP privada por instancia, más otra dirección IP privada si se configura una dirección IP de front-end privada.

Además, Azure se reserva cinco direcciones IP en cada subred para uso interno: las cuatro primeras direcciones IP y la última. Por ejemplo, suponga que hay 15 instancias de Application Gateway sin ninguna dirección IP de front-end privada. Necesita al menos 20 direcciones IP para esta subred: cinco para uso interno y 15 para las instancias de Application Gateway.

Ahora imagine una subred que tiene 27 instancias de Application Gateway y una dirección IP de front-end privada. En este caso, necesita 33 direcciones IP: 27 para las instancias de Application Gateway, una para el front-end privado y cinco para uso interno.

La SKU de Application Gateway (Estándar o WAF) puede admitir hasta 32 instancias (32 instancias de direcciones IP + 1 dirección IP de front-end privada + 5 instancias reservadas de Azure), por lo que se recomienda un tamaño de subred mínimo de /26.

Application Gateway (SKU Standard_v2 o WAF_v2) puede admitir hasta 125 instancias (125 instancias de direcciones IP + 1 dirección IP de front-end privada + 5 instancias reservadas de Azure). Se recomienda un tamaño mínimo de subred de /24.

> [!IMPORTANT]
> Aunque no se requiere una subred /24 por cada implementación de SKU v2 de Application Gateway, se recomienda encarecidamente. Esto es así para asegurarse de que Application Gateway v2 tenga suficiente espacio para la expansión del escalado automático y las actualizaciones de mantenimiento. Debe asegurarse de que la subred Application Gateway v2 tenga suficiente espacio de direcciones para dar cabida al número de instancias necesarias para atender el tráfico máximo esperado. Si especifica el recuento de instancias máximo, la subred debe tener capacidad cómo mínimo para ese número de direcciones. Para planear la capacidad según el recuento de instancias, consulte [detalles del recuento de instancias](understanding-pricing.md#instance-count).

> [!TIP]
> Es posible cambiar la subred de una instancia de Application Gateway dentro de la misma red virtual. Puede hacerlo con Azure PowerShell o la CLI de Azure. Para más información, consulte las [preguntas más frecuentes sobre Application Gateway](application-gateway-faq.yml#can-i-change-the-virtual-network-or-subnet-for-an-existing-application-gateway).

## <a name="network-security-groups"></a>Grupos de seguridad de red

Los grupos de seguridad se admiten en Application Gateway. Pero hay algunas restricciones:

- Debe permitir el tráfico entrante de Internet en los puertos TCP 65503-65534 para la SKU de Application Gateway v1, y los puertos TCP 65200-65535 para la SKU de V2 con la subred de destino como **cualquiera** y origen como la etiqueta de servicio de **GatewayManager**. Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure. Estos puertos están protegidos (bloqueados) mediante certificados de Azure. Las entidades externas, incluidos los clientes de esas puertas de enlace, no pueden comunicarse en estos puntos de conexión.

- No puede bloquearse la conectividad saliente de Internet. Las reglas de salida predeterminadas del grupo de seguridad de red permiten la conectividad a Internet. Se recomienda que:

  - No elimine las reglas de salida predeterminadas.
  - No cree otras reglas de salida que denieguen toda la conectividad saliente.

- Se debe permitir el tráfico de la etiqueta **AzureLoadBalancer** con la subred de destino como **Cualquiera**.

### <a name="allow-access-to-a-few-source-ips"></a>Permiso de acceso a algunas IP de origen

En este escenario, puede usar grupos de seguridad de red en la subred de Application Gateway. Ponga las restricciones siguientes en la subred en este orden de prioridad:

1. Permita el tráfico entrante desde una dirección IP de origen o intervalo de direcciones IP con el destino como el intervalo de direcciones de subred Application Gateway completo y el puerto de destino como puerto de acceso de entrada, por ejemplo, el puerto 80 para el acceso HTTP.
2. Permita solicitudes entrantes desde el origen como **GatewayManager** etiqueta de servicio y destino como **Cualquiera** y puertos de destino como 65503-65534 para el SKU de Application Gateway v1, y puertos 65200-65535 para SKU v2 para [ comunicación de estado de salud de back-end](./application-gateway-diagnostics.md). Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure. Estos puertos están protegidos (bloqueados) mediante certificados de Azure. Sin los certificados adecuados en vigor, las entidades externas no podrán iniciar cambios en esos puntos de conexión.
3. Permitir sondeos entrantes de Azure Load Balancer (con la etiqueta *AzureLoadBalancer*) y el tráfico de red virtual entrante (con la etiqueta *VirtualNetwork*) en el [grupo de seguridad de red](../virtual-network/network-security-groups-overview.md).
4. Bloquear todo el tráfico entrante restante mediante una regla Denegar todo.
5. Permitir el tráfico saliente a Internet para todos los destinos.

## <a name="supported-user-defined-routes"></a>Rutas definidas por el usuario admitidas 

> [!IMPORTANT]
> El uso de UDR en la subred de Application Gateway podría hacer que el estado de mantenimiento en la [vista de mantenimiento de back-end](./application-gateway-diagnostics.md#back-end-health) se muestre como **Desconocido**. También podría provocar un error en la generación de registros y métricas de Application Gateway. Se recomienda que no use rutas definidas por el usuario en la subred de Application Gateway para que pueda ver el estado, los registros y las métricas del back-end.

- **v1**

   Para la SKU v1, las rutas definidas por el usuario (UDR) se admiten en la subred de la puerta de enlace de aplicaciones, siempre que no alteren la comunicación de solicitud y respuesta de un extremo a otro. Por ejemplo, puede configurar una ruta definida por el usuario en la subred de Application Gateway para que apunte a un dispositivo de firewall para la inspección de paquetes. Debe asegurarse de que el paquete puede llegar a su destino previsto después de la inspección. El no hacerlo podría resultar en un sondeo del estado o en un comportamiento de enrutamiento de tráfico incorrectos. Esto incluye las rutas aprendidas o las rutas 0.0.0.0/0 predeterminadas que se propagan por las puertas de enlace de VPN o de Azure ExpressRoute en la red virtual. La versión 1 no admite los escenarios en los que se debe redirigir 0.0.0.0/0 en el entorno local (tunelización forzada).

- **v2**

   En el caso de la SKU v2, existen escenarios admitidos y no admitidos:

   **Escenarios admitidos de v2**
   > [!WARNING]
   > Una configuración incorrecta de la tabla de rutas podría dar lugar a un enrutamiento asimétrico en Application Gateway v2. Asegúrese de que todo el tráfico de plano de control o administración se envía directamente a Internet y no mediante una aplicación virtual. El registro y las métricas también podrían verse afectados.


  **Escenario 1**: UDR para deshabilitar la propagación de rutas del Protocolo de puerta de enlace de borde (BGP) a la subred de Application Gateway

   A veces, la ruta de puerta de enlace predeterminada (0.0.0.0/0) se anuncia mediante las puertas de enlace de VPN o ExpressRoute asociadas con la red virtual de Application Gateway. Esto interrumpe el tráfico del plano de administración, lo que requiere una ruta de acceso directa a Internet. En estos escenarios, se puede usar una UDR para deshabilitar la propagación de la ruta BGP. 

   Para deshabilitar la propagación de la ruta BGP, siga estos pasos:

   1. Cree un recurso de tabla de rutas en Azure.
   2. Deshabilite el parámetro **Propagación de rutas de puerta de enlace de red virtual**. 
   3. Asocie la tabla de rutas a la subred adecuada. 

   La habilitación de UDR para este escenario no debe interrumpir las configuraciones existentes.

  **Escenario 2**: UDR para dirigir 0.0.0.0/0 a Internet

   Puede crear una UDR para enviar el tráfico 0.0.0.0/0 directamente a Internet. 

  **Escenario 3**: UDR para Azure Kubernetes Service con kubenet

  Si usa kubenet con Azure Kubernetes Service (AKS) y el Controlador de entrada de Application Gateway (AGIC), necesitará una tabla de rutas para permitir que el tráfico enviado a los pods desde Application Gateway se redirija al nodo correcto. Esto no será necesario si usa Azure CNI. 

  Para usar la tabla de rutas para que kubenet funcione, siga estos pasos:

  1. Vaya al grupo de recursos creado por AKS (el nombre del grupo de recursos debe comenzar por "MC_").
  2. Busque la tabla de rutas creada por AKS en ese grupo de recursos. La tabla de rutas se debe rellenar con la siguiente información:
     - El prefijo de dirección debe ser el intervalo IP de los pods a los que quiere acceder en AKS. 
     - El siguiente tipo de salto debe ser Aplicación virtual. 
     - La dirección del próximo salto debe ser la dirección IP del nodo que hospeda los pods.
  3. Asocie esta tabla de rutas a la subred de Application Gateway. 
    
  **Escenarios incompatibles de v2**

  **Escenario 1**: UDR para aplicaciones virtuales

  Cualquier escenario en que se necesite redirigir 0.0.0.0/0 a través de cualquier aplicación virtual, una red virtual radial o un entorno local (tunelización forzada) no se admite para la versión V2.

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre la configuración de la dirección IP de front-end](configuration-front-end-ip.md).
