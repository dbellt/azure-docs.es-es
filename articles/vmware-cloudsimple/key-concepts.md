---
title: Conceptos clave para administrar Azure VMware Solution by CloudSimple
titleSuffix: Azure VMware Solution by CloudSimple
description: En este artículo se describen conceptos clave para administrar Azure VMware Solution by CloudSimple
author: shortpatti
ms.author: v-patsho
ms.date: 04/24/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 9e97abdd46e1c83d7883a51ddbd73ea0aa75b9f2
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108184269"
---
# <a name="key-concepts-for-administration-of-azure-vmware-solutions-by-cloudsimple"></a>Conceptos clave para la administración de Azure VMware Solution by CloudSimple

La administración de Azure VMware Solution by CloudSimple exige entender los conceptos siguientes:

* El servicio CloudSimple, que se muestra como Azure VMware Solution by CloudSimple: servicio
* El nodo de CloudSimple, que se muestra como Azure VMware Solution by CloudSimple: nodo
* Nube privada de CloudSimple
* Redes de servicio
* La máquina virtual de CloudSimple, que se muestra como Azure VMware Solution by CloudSimple: máquina virtual

## <a name="cloudsimple-service"></a>Servicio CloudSimple

Con el servicio CloudSimple, puede crear y administrar todos los recursos asociados con soluciones de VMware de CloudSimple desde Azure Portal. Cree un recurso de servicio en todas las regiones donde piense usar el servicio.

Más información sobre el [servicio CloudSimple](cloudsimple-service.md).

## <a name="cloudsimple-node"></a>Nodo de CloudSimple

Un nodo de CloudSimple es un host de proceso y almacenamiento hiperconvergido, sin sistema operativo y dedicado en el que se implementa el hipervisor ESXi de VMware. Después, este nodo se incorpora a las plataformas VMware vSphere, vCenter, vSAN y NSX. También se habilitan los servicios de red de CloudSimple y los de red perimetral. Cada nodo actúa como una unidad de capacidad de proceso y almacenamiento que puede aprovisionar para crear [nubes privadas de CloudSimple](cloudsimple-private-cloud.md). Los nodos se aprovisionan o reservan en una región donde está disponible el servicio CloudSimple.

Más información sobre [los nodos de CloudSimple](cloudsimple-node.md).

## <a name="cloudsimple-private-cloud"></a>Nube privada de CloudSimple

Una nube privada de CloudSimple es un entorno de pila de VMware aislada administrado por un servidor vCenter en su propio dominio de administración. La pila VMware incluye hosts ESXi, vSphere, vCenter, vSAN y NSX. La pila se ejecuta en nodos dedicados (hardware sin sistema operativo dedicado y aislado) y los usuarios la consumen a través de herramientas nativas de VMware que incluyen vCenter y NSX Manager. Los nodos dedicados se implementan en ubicaciones de Azure y son administrados por Azure. Cada nube privada se puede segmentar y proteger mediante servicios de red como VLAN y subredes y tablas de firewall. Las conexiones al entorno local y la red de Azure se crean mediante VPN privadas y seguras, y conexiones de Azure ExpressRoute.

Más información sobre la [nube privada de CloudSimple](cloudsimple-private-cloud.md).

## <a name="service-networking"></a>Redes de servicio

El servicio CloudSimple proporciona una red por región donde se implementa el servicio CloudSimple. La red es un único espacio de direcciones TCP de nivel 3 con el enrutamiento habilitado de forma predeterminada. Todas las nubes privadas y las subredes creadas en esta región se comunican entre sí sin ninguna configuración adicional. En vCenter, los grupos de puertos distribuidos se crean mediante VLAN. Puede usar las características de red siguientes para configurar y proteger los recursos de carga de trabajo en la nube privada:

* [VLAN y subredes](cloudsimple-vlans-subnets.md)
* [Tablas de firewall](cloudsimple-firewall-tables.md)
* [Puertas de enlace de VPN](cloudsimple-vpn-gateways.md)
* [Dirección IP pública](cloudsimple-public-ip-address.md)
* [Conexión de red de Azure](cloudsimple-azure-network-connection.md)

## <a name="cloudsimple-virtual-machine"></a>Máquina virtual de CloudSimple

Con el servicio CloudSimple, puede administrar máquinas virtuales de VMware desde Azure Portal. Se pueden asignar uno o más clústeres o grupos de recursos del entorno de vSphere a la suscripción en la que se crea el servicio.

Más información sobre:

* [Máquinas virtuales de CloudSimple](cloudsimple-virtual-machines.md)
* [Asignación de suscripciones de Azure](./azure-subscription-mapping.md)
