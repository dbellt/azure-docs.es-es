---
author: alexeyo26
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 03/19/2021
ms.author: alexeyo
ms.openlocfilehash: db8ae4bba6717cb1c0f27befd0897b60c2760ed2
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "110059214"
---
## <a name="private-endpoints-and-vnet-service-endpoints"></a>Puntos de conexión privados y puntos de conexión de servicio de red virtual

Azure ofrece puntos de conexión privados y puntos de conexión de servicio de red virtual para la tunelización de tráfico mediante la [red troncal de Azure privada](https://azure.microsoft.com/global-infrastructure/global-network/). Estos tipos de puntos de conexión son similares en lo que respecta a su propósito y las tecnologías en las que se basan. Sin embargo, existen diferencias entre las dos tecnologías y se recomienda obtener más información sobre las ventajas y desventajas de cada una antes de diseñar la red.

Estos son algunos aspectos que debe tener en cuenta al hacer su elección:
- Ambas tecnologías garantizan que el tráfico entre la red virtual y el recurso de voz *no* pase por la red pública de Internet.
- Un punto de conexión privado proporciona una dirección IP privada dedicada para el recurso de voz. Esta dirección IP solo es accesible dentro de una red virtual y una subred específicas. Tiene control total sobre el acceso a esta dirección IP dentro de su infraestructura de red.
- Los puntos de conexión de servicio de red virtual *no* proporcionan una dirección IP privada dedicada para el recurso de voz, sino que encapsulan todos los paquetes enviados al recurso de voz y los entregan directamente a través de la red troncal de Azure.
- Ambas tecnologías admiten escenarios locales. De forma predeterminada, cuando se usan puntos de conexión de servicio de red virtual, los recursos de servicio de Azure protegidos en redes virtuales no son accesibles desde redes locales, pero esto [se puede configurar](../../../virtual-network/virtual-network-service-endpoints-overview.md#secure-azure-service-access-from-on-premises).
- Los puntos de servicio de red virtual se usan a menudo cuando se desea restringir el acceso del recurso de voz basándose en las redes virtuales desde las que se origina el tráfico.
- En el caso de Cognitive Services, la habilitación del punto de conexión de servicio de red virtual fuerza el tráfico de **todos** los recursos de Cognitive Services a través de la red troncal privada. Esto requiere una configuración explícita de acceso a la red (consulte [aquí](../speech-service-vnet-service-endpoint.md#configure-vnets-and-the-speech-resource-networking-settings) los detalles). Los puntos de conexión privados no tienen esta limitación y proporcionan más flexibilidad para la configuración de red: puede acceder a un recurso a través de la red troncal privada y a otro a través de la red pública de Internet mediante la misma subred de la misma red virtual.
- Los puntos de conexión privados acarrean [costos adicionales](https://azure.microsoft.com/pricing/details/private-link). Los puntos de conexión de servicio de red virtual son gratuitos.
- Los puntos de conexión privados requieren una [configuración de DNS adicional](../speech-services-private-link.md#turn-on-private-endpoints).
- Un recurso de voz puede funcionar simultáneamente con puntos de conexión privados y puntos de conexión de servicio de red virtual.

Se recomienda probar ambos tipos de puntos de conexión antes de decidir el diseño de producción. 

- [Documentación de Private Link y puntos de conexión privados](../../../private-link/private-link-overview.md)
- [Documentación de puntos de conexión de servicio de red virtual](../../../virtual-network/virtual-network-service-endpoints-overview.md)
