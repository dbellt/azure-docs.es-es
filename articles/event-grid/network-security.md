---
title: Seguridad de red para recursos de Azure Event Grid
description: En este artículo se describe cómo usar las etiquetas de servicio para la salida, las reglas de firewall de IP para la entrada y los puntos de conexión privados para la entrada con Azure Event Grid.
author: VidyaKukke
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: vkukke
ms.openlocfilehash: d58e8b5ce9fb444fa501f897cca722613d9c51fe
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105967581"
---
# <a name="network-security-for-azure-event-grid-resources"></a>Seguridad de red para recursos de Azure Event Grid
En este artículo se describe cómo usar las siguientes características de seguridad con Azure Event Grid: 

- Etiquetas de servicio para salida
- Reglas de firewall de IP para entrada
- Puntos de conexión privados para entrada


## <a name="service-tags"></a>Etiquetas de servicio
Una etiqueta de servicio representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian, lo que minimiza la complejidad de las actualizaciones frecuentes en las reglas de seguridad de red. Para más información sobre las etiquetas de servicio, consulte [Introducción a las etiquetas de servicio](../virtual-network/service-tags-overview.md).

Puede usar etiquetas de servicio para definir controles de acceso a la red en [grupos de seguridad de red](../virtual-network/network-security-groups-overview.md#security-rules) o [Azure Firewall](../firewall/service-tags.md). Utilice etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, **AzureEventGrid**) en el campo de *origen* o *destino* apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente.

| Etiqueta de servicio | Propósito | ¿Se puede usar para tráfico entrante o saliente? | ¿Puede ser regional? | ¿Se puede usar con Azure Firewall? |
| --- | -------- |:---:|:---:|:---:|
| AzureEventGrid | Azure Event Grid. | Ambos | No | No |


## <a name="ip-firewall"></a>Firewall de dirección IP 
Azure Event Grid admite controles de acceso basados en IP para la publicación en temas y dominios. Con los controles basados en IP, puede limitar los publicadores de un tema o dominio a solo un conjunto aprobado de máquinas y servicios en la nube. Esta característica complementa los [mecanismos de autenticación](security-authentication.md) compatibles con Event Grid.

De forma predeterminada, el tema y el dominio son accesibles desde Internet siempre y cuando la solicitud venga con una autenticación y una autorización válidas. Con el firewall de IP, puede restringirlo aún más a solo un conjunto de direcciones IP o intervalos de direcciones IP en la notación [CIDR (Enrutamiento de interdominios sin clases)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). Los publicadores que se originen desde cualquier otra dirección IP se rechazarán y recibirán una respuesta 403 (Prohibido).

A fin de obtener instrucciones paso a paso para configurar el firewall de IP para temas y dominios, consulte [configurar el firewall de IP](configure-firewall.md).

## <a name="private-endpoints"></a>Puntos de conexión privados
Puede usar [puntos de conexión privados](../private-link/private-endpoint-overview.md) para permitir la entrada de eventos directamente desde su red virtual a sus temas y dominios de forma segura a través de un [vínculo privado](../private-link/private-link-overview.md) sin tener que ir a la red pública de Internet. Un punto de conexión privado es una interfaz de red especial para un servicio de Azure de una red virtual. Cuando se crea un punto de conexión privado para un tema o dominio, este proporciona conectividad segura entre los clientes de la red virtual y el recurso de Event Grid. Al punto de conexión privado se le asigna una dirección IP del intervalo de direcciones IP de la red virtual. La conexión entre el punto de conexión privado y el servicio de Event Grid usa un vínculo privado seguro.

![Diagrama de la arquitectura](./media/network-security/architecture-diagram.png)

El uso de puntos de conexión privados en un recurso de Event Grid permite:

- Proteger el acceso a su tema o dominio desde una red virtual a través de la red troncal de Microsoft en oposición a la red pública de Internet.
- Conectarse de forma segura desde las redes locales que se conectan a la red virtual mediante VPN o instancias de ExpressRoute con emparejamiento privado.

Cuando se crea un punto de conexión privado para un tema o dominio en la red virtual, se envía una solicitud de consentimiento para su aprobación al propietario del recurso. Si el usuario que solicita la creación del punto de conexión privado también es propietario del recurso, esta solicitud de consentimiento se aprueba automáticamente. De lo contrario, la conexión está en estado **pendiente** hasta que se aprueba. Las aplicaciones de la red virtual se pueden conectar al servicio de Event Grid a través del punto de conexión privado sin problemas, ya que se usan las mismas cadenas de conexión y mecanismos de autorización que se usarían en cualquier otro caso. Los propietarios del recurso pueden administrar las solicitudes de consentimiento y los puntos de conexión privados desde la pestaña **Puntos de conexión privados** del recurso en Azure Portal.

### <a name="connect-to-private-endpoints"></a>Conexión a puntos de conexión privados
Los publicadores de una red virtual que usan el punto de conexión privado deben usar la misma cadena de conexión para el tema o dominio que aquellos clientes que se conectan mediante el punto de conexión público. La resolución DNS enruta automáticamente conexiones desde la red virtual al tema o dominio a través de un vínculo privado. De forma predeterminada, Event Grid crea una [zona DNS privada](../dns/private-dns-overview.md) conectada a la red virtual con la actualización necesaria para los puntos de conexión privados. Sin embargo, si usa su propio servidor DNS, puede que tenga que realizar cambios adicionales en la configuración de DNS.

### <a name="dns-changes-for-private-endpoints"></a>Cambios de DNS en puntos de conexión privados
Al crear un punto de conexión privado, el registro del recurso CNAME de DNS del recurso se actualiza a un alias de un subdominio con el prefijo `privatelink`. De forma predeterminada, se crea una zona DNS privada que corresponde al subdominio del vínculo privado. 

Cuando se resuelve la dirección URL del punto de conexión del tema o dominio desde fuera de la red virtual con el punto de conexión privado, se resuelve en el punto de conexión público del servicio. Los registros de recursos DNS de 'topicA', cuando se resuelven desde **fuera de la red virtual** que hospeda el punto de conexión privado, serán:

| Nombre                                          | Tipo      | Value                                         |
| --------------------------------------------- | ----------| --------------------------------------------- |  
| `topicA.westus.eventgrid.azure.net`             | CNAME     | `topicA.westus.privatelink.eventgrid.azure.net` |
| `topicA.westus.privatelink.eventgrid.azure.net` | CNAME     | \<Azure traffic manager profile\>

Puede denegar o controlar el acceso de un cliente de fuera de la red virtual a través del punto de conexión público mediante el [firewall de IP](#ip-firewall). 

Cuando se resuelve desde la red virtual que hospeda el punto de conexión privado, la dirección URL del punto de conexión del tema o dominio se resuelve en la dirección IP del punto de conexión privado. Los registros de recursos DNS del tema 'topicA', cuando se resuelven desde **dentro de la red virtual** que hospeda el punto de conexión privado, serán:

| Nombre                                          | Tipo      | Value                                         |
| --------------------------------------------- | ----------| --------------------------------------------- |  
| `topicA.westus.eventgrid.azure.net`             | CNAME     | `topicA.westus.privatelink.eventgrid.azure.net` |
| `topicA.westus.privatelink.eventgrid.azure.net` | A         | 10.0.0.5

Este enfoque permite el acceso al tema o dominio mediante la misma cadena de conexión para los clientes de la red virtual que hospeda los puntos de conexión privados y los clientes que están fuera de esta.

Si va a usar un servidor DNS personalizado en la red, los clientes pueden resolver el FQDN del punto de conexión del tema o dominio en la dirección IP del punto de conexión privado. Configure el servidor DNS para delegar el subdominio del vínculo privado en la zona DNS privada de la red virtual o configure los registros A para `topicOrDomainName.regionName.privatelink.eventgrid.azure.net` con la dirección IP del punto de conexión privado.

El nombre de zona DNS recomendado es `privatelink.eventgrid.azure.net`.

### <a name="private-endpoints-and-publishing"></a>Puntos de conexión privados y publicación

En la tabla siguiente se describen los distintos estados de la conexión del punto de conexión privado y los efectos en la publicación:

| Estado de conexión   |  Publicación correcta (Sí/No) |
| ------------------ | -------------------------------|
| Aprobado           | Sí                            |
| Rechazada           | No                             |
| Pending            | No                             |
| Escenario desconectado       | No                             |

Para que la publicación se realice correctamente, el estado de conexión del punto de conexión privado debe **aprobarse**. Si se rechaza una conexión, no se puede aprobar mediante Azure Portal. La única posibilidad es eliminar la conexión y crear una nueva en su lugar.

## <a name="pricing-and-quotas"></a>Precios y cuotas
Los **puntos de conexión privados** están disponible en los niveles Básico y Premium de Event Grid. Event Grid permite crear hasta 64 conexiones del punto de conexión privado por tema o dominio. 

La característica **Firewall de IP** está disponible en los niveles Básico y Premium de Event Grid. Permitimos crear hasta 16 reglas de firewall de IP por tema o dominio.

## <a name="next-steps"></a>Pasos siguientes
Puede configurar el firewall de IP para el recurso de Event Grid con el fin de restringir el acceso a través de Internet público desde únicamente un conjunto seleccionado de direcciones IP o intervalos de direcciones IP. Para obtener instrucciones paso a paso, consulte [Configuración del firewall de IP](configure-firewall.md).

Puede configurar puntos de conexión privados para restringir el acceso solo desde redes virtuales seleccionadas. Para obtener instrucciones detalladas, consulte [Configuración de puntos de conexión privados](configure-private-endpoints.md).

Para solucionar problemas de conectividad de red, consulte [Solucionar problemas de conectividad de red](troubleshoot-network-connectivity.md).
