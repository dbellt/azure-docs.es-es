---
title: 'Conceptos: interconectividad de red'
description: Conozca los aspectos clave y los casos de uso de redes e interconectividad en Azure VMware Solution.
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: b3107ff85302e24e3c047ef0ea08f7a4b879b3be
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110089603"
---
# <a name="azure-vmware-solution-networking-and-interconnectivity-concepts"></a>Conceptos de interconectividad y redes de Azure VMware Solution

[!INCLUDE [avs-networking-description](includes/azure-vmware-solution-networking-description.md)]

Hay dos formas de interconectividad en la nube privada de Azure VMware Solution:

- La [**interconectividad básica solo de Azure**](#azure-virtual-network-interconnectivity) permite administrar y usar la nube privada con una sola red virtual en Azure. Esta implementación es más adecuada para las evaluaciones o implementaciones de Azure VMware Solution que no requieren acceso desde entornos locales.

- La [**interconectividad completa entre el entorno local y la nube privada**](#on-premises-interconnectivity) amplía la implementación básica solo de Azure para incluir la interconectividad entre el entorno local y las nubes privadas de Azure VMware Solution.
 
En este artículo se tratan los conceptos clave que establecen las redes y la interconectividad, incluidos los requisitos y las limitaciones. En este artículo se proporciona la información necesaria para configurar correctamente las redes a fin de que funcionen con Azure VMware Solution.

## <a name="azure-vmware-solution-private-cloud-use-cases"></a>Casos de uso de la nube privada de Azure VMware Solution

Los casos de uso de las nubes privadas de Azure VMware Solution incluyen los siguientes:
- nuevas cargas de trabajo de máquinas virtuales de VMware en la nube
- Ráfaga de cargas de trabajo de máquinas virtuales a la nube (solo del entorno local a Azure VMware Solution)
- Migración de cargas de trabajo de máquinas virtuales a la nube (solo del entorno local a Azure VMware Solution)
- Recuperación ante desastres (de Azure VMware Solution a Azure VMware Solution o del entorno local a Azure VMware Solution)
- Consumo de servicios de Azure

> [!TIP]
> En todos los casos de uso del servicio Azure VMware Solution se permite la conectividad del entorno local a la nube privada.

## <a name="azure-virtual-network-interconnectivity"></a>Conectividad de la red virtual de Azure

Puede interconectar su red virtual de Azure con la implementación de la nube privada de la Azure VMware Solution. Puede administrar su nube privada de Azure VMware Solution, consumir cargas de trabajo en la nube privada y acceder a otros servicios de Azure.

En el diagrama siguiente se muestra la interconectividad de red básica establecida al implementar una nube privada. Muestra las redes lógicas entre una red virtual de Azure y una nube privada. Esta conectividad se establece a través de una instancia de ExpressRoute de back-end que forma parte del servicio de Azure VMware Solution. La interconectividad cumple los siguientes casos de uso principales:

- Acceso de entrada al servidor vCenter y al administrador NSX-T, que es accesible desde las VM de la suscripción a Azure.
- Acceso de salida desde las máquinas virtuales de la nube privada a los servicios de Azure.
- Acceso de entrada de cargas de trabajo que se ejecutan en una nube privada.


:::image type="content" source="media/concepts/adjacency-overview-drawing-single.png" alt-text="Conectividad de red virtual básica a nube privada" border="false":::

## <a name="on-premises-interconnectivity"></a>Interconectividad local

En el escenario totalmente interconectado, puede acceder a Azure VMware Solution desde las redes virtuales de Azure y en el entorno local. Esta implementación es una extensión de la implementación básica descrita en la sección anterior. Se requiere un circuito de ExpressRoute para conectarse desde el entorno local a su nube privada de Azure VMware Solution en Azure.

En el diagrama siguiente se muestra la interconectividad local a nube privada, que permite los siguientes casos de uso:

- vCenter vMotion frecuente o en frío entre el entorno local y Azure VMware Solution.
- Acceso de administración desde el entorno local a la nube privada de Azure VMware Solution.

:::image type="content" source="media/concepts/adjacency-overview-drawing-double.png" alt-text="Red virtual y conectividad de nube privada local completa" border="false":::

Para obtener una interconectividad total a la nube privada, debe habilitar ExpressRoute Global Reach y solicitar una clave de autorización y un identificador de emparejamiento privado de Global Reach en Azure Portal. La clave de autorización y el id. de emparejamiento se usan para establecer Global Reach entre un circuito ExpressRoute de la suscripción y el circuito ExpressRoute de la nube privada. Una vez vinculados, los dos circuitos ExpressRoute enrutan el tráfico de red entre los entornos locales a la nube privada. Para obtener más información sobre los procedimientos necesarios, vea el [tutorial para crear un emparejamiento de ExpressRoute Global Reach con una nube privada](tutorial-expressroute-global-reach-private-cloud.md).

## <a name="limitations"></a>Limitaciones
[!INCLUDE [azure-vmware-solutions-limits](includes/azure-vmware-solutions-limits.md)]

## <a name="next-steps"></a>Pasos siguientes 

Ahora que ha tratado estos conceptos de red e interconectividad de Azure VMware Solution, puede que quiera obtener información sobre:

- [Conceptos de almacenamiento de Azure VMware Solution](concepts-storage.md).
- [Conceptos de identidad en Azure VMware Solution](concepts-identity.md)
- [Habilitación del recurso de Azure VMware Solution](deploy-azure-vmware-solution.md#step-1-register-the-microsoftavs-resource-provider)

<!-- LINKS - external -->
[enable Global Reach]: ../expressroute/expressroute-howto-set-global-reach.md

<!-- LINKS - internal -->
[concepts-upgrades]: ./concepts-private-clouds-clusters#host-maintenance-and-lifecycle-management
[concepts-storage]: ./concepts-storage.md
