---
title: Escenarios para usar una red virtual
description: Escenarios, recursos y limitaciones para implementar grupos de contenedores en una red virtual de Azure.
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 6de99c68c3f05e4734dd46a579d28a6f1a3b824e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763782"
---
# <a name="virtual-network-scenarios-and-resources"></a>Escenarios y recursos de red virtual

[Azure Virtual Network](../virtual-network/virtual-networks-overview.md) proporciona acceso de red seguro y privado a los recursos locales y de Azure. Al implementar grupos de contenedores en una red virtual de Azure, los contenedores pueden comunicarse de forma segura con otros recursos de la red virtual. 

Este artículo proporciona información general sobre escenarios, limitaciones y recursos de red virtual. Puede encontrar ejemplos de implementación mediante la CLI de Azure en [Implementación de instancias de contenedor en una red virtual de Azure](container-instances-vnet.md).

> [!IMPORTANT]
> La implementación del grupo de contenedores en una red virtual está disponible con carácter general para contenedores de Linux en la mayoría de las regiones donde Azure Container Instances está disponible. Para obtener más información, consulte [Regiones y disponibilidad de recursos](container-instances-region-availability.md). 

## <a name="scenarios"></a>Escenarios

Los grupos de contenedores implementados en una red virtual de Azure permiten escenarios como:

* Comunicación directa entre grupos de contenedores en la misma subred
* Enviar la salida de la carga de trabajo [basada en tareas](container-instances-restart-policy.md) procedente de las instancias de contenedor a una base de datos en la red virtual
* Recuperar contenido de las instancias de contenedor desde un [punto de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) en la red virtual
* Comunicación del contenedor con los recursos locales mediante una [puerta de enlace VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../expressroute/expressroute-introduction.md)
* Integración con [Azure Firewall](../firewall/overview.md) para identificar el tráfico saliente que se origina en el contenedor 
* Resolución de nombres mediante Azure DNS interno para la comunicación con los recursos de Azure de la red virtual, como las máquinas virtuales
* Uso de reglas de grupo de seguridad de red para controlar el acceso a las subredes o a otros recursos de red

## <a name="unsupported-networking-scenarios"></a>Escenarios de red avanzados no admitidos 

* **Azure Load Balancer**: no se admite la colocación de Azure Load Balancer al principio de las instancias de contenedor en un grupo de contenedores en red
* **Emparejamiento de redes virtuales globales**: no se admite el emparejamiento global (conexión de redes virtuales entre regiones de Azure).
* **Etiqueta de dirección IP o DNS pública**: los grupos de contenedores implementados en una red virtual no son compatibles actualmente con la exposición directa en Internet de contenedores con una dirección IP pública o un nombre de dominio completo.
* **NAT de Virtual Network**: los grupos de contenedores implementados en una red virtual no admiten actualmente el uso de un recurso de puerta de enlace NAT para la conectividad saliente de Internet.

## <a name="other-limitations"></a>Otras limitaciones

* Actualmente, solo se admiten contenedores de Linux en un grupo de contenedores implementado en una red virtual.
* Para implementar grupos de contenedores en una subred, la subred no puede contener otros tipos de recursos. Quite todos los recursos existentes de una subred existente antes de implementar grupos de contenedores en ella o crear una nueva subred.
* No puede usar una [identidad administrada](container-instances-managed-identity.md) en un grupo de contenedores implementado en una red virtual.
* No puede habilitar un [sondeo de ejecución](container-instances-liveness-probe.md) o un [sondeo de preparación](container-instances-readiness-probe.md) en un grupo de contenedores implementado en una red virtual.
* Debido a los recursos de red adicionales que intervienen, las implementaciones en una red virtual suelen ser más lentas que las de una instancia de contenedor estándar.
* Si va a conectar el grupo de contenedores a una cuenta de Azure Storage, debe agregar un [punto de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) a ese recurso.

[!INCLUDE [container-instances-restart-ip](../../includes/container-instances-restart-ip.md)]

## <a name="required-network-resources"></a>Recursos de red necesarios

Hay tres recursos de Azure Virtual Network necesarios para implementar grupos de contenedores en una red virtual: la propia [red virtual](#virtual-network), una [subred delegada](#subnet-delegated) dentro de la red virtual y un [perfil de red](#network-profile). 

### <a name="virtual-network"></a>Virtual network

Una red virtual define el espacio de direcciones en el que crear una o varias subredes. A continuación, implemente los recursos de Azure (por ejemplo, los grupos de contenedores) en las subredes de la red virtual.

### <a name="subnet-delegated"></a>Subred (delegada)

Las subredes segmentan la red virtual en espacios de direcciones independientes utilizables por los recursos de Azure que coloque en ellos. Crea una o varias subredes dentro de una red virtual.

La subred que usó para los grupos de contenedores puede contener solo grupos de contenedores. Cuando implementa por primera vez un grupo de contenedores en una subred, Azure delega esa subred para Azure Container Instances. Una vez delegada, la subred se puede usar solo para grupos de contenedores. Si intenta implementar recursos distintos de grupos de contenedores en una subred delegada, se produce un error en la operación.

### <a name="network-profile"></a>Perfil de red

Un perfil de red es una plantilla de configuración de red para los recursos de Azure. Especifica determinadas propiedades de red para el recurso, por ejemplo, la subred en la que debe implementarse. La primera vez que usa el comando [az container create][az-container-create] para implementar un grupo de contenedores en una subred (y, por tanto, en una red virtual), Azure crea un perfil de red para usted. A continuación, puede usar ese perfil de red para implementaciones futuras en la subred. 

Para usar una plantilla de Resource Manager, el archivo YAML o un método de programación para implementar un grupo de contenedores en una subred, deberá proporcionar el identificador de recurso de Resource Manager completo de un perfil de red. Puede usar un perfil que creó previamente mediante [az container create][az-container-create] o bien crear un perfil mediante una plantilla de Resource Manager (consulte el [ejemplo de plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet) y la [referencia](/azure/templates/microsoft.network/networkprofiles)). Para obtener el identificador de un perfil creado anteriormente, use el comando [az network profile list][az-network-profile-list]. 

En el diagrama siguiente, se han implementado varios grupos de contenedores en una subred delegada en Azure Container Instances. Una vez implementado un grupo de contenedores en una subred, puede implementar grupos de contenedores adicionales en él si especifica el mismo perfil de red.

![Grupos de contenedores dentro de una red virtual][aci-vnet-01]

## <a name="next-steps"></a>Pasos siguientes

* Puede encontrar ejemplos de implementación mediante la CLI de Azure en [Implementación de instancias de contenedor en una red virtual de Azure](container-instances-vnet.md).
* Para implementar una nueva red virtual, subred, perfil de red y grupo de contenedores mediante una plantilla de Resource Manager, consulte el artículo sobre la [creación de un grupo de contenedores de Azure con VNet](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
).
* Al utilizar el [Azure Portal](container-instances-quickstart-portal.md) para crear una instancia de contenedor, también puede incluir la configuración de una red virtual nueva o existente en la pestaña **Redes**.


<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-virtual-network-concepts/aci-vnet-01.png

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-network-profile-list]: /cli/azure/network/profile#az_network_profile_list
