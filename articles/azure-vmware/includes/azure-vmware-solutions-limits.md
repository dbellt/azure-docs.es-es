---
title: Límites de Azure VMware Solution
description: Limitaciones de Azure VMware Solution.
ms.topic: include
ms.date: 03/24/2021
ms.openlocfilehash: 856aca934b7a80b8f5dc7e336b91821ab299b34c
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107951184"
---
<!-- Used in /azure/azure-resource-manager/management/azure-subscription-service-limits.md and concepts-networking.md -->

En la tabla siguiente se describen los límites máximos para Azure VMware Solution.

| **Recurso** | **Límite** |
| :-- | :-- |
| Clústeres por nube privada | 12 |
| Número mínimo de nodos por clúster | 3 |
| Número máximo de nodos por clúster | 16 |
| Nodos por nube privada | 96 |
| vCenter por nube privada | 1  |
| Emparejamientos con sitios de HCX | 3 con Advanced Edition, 10 con Enterprise Edition |
| Máx. de SDDC vinculados a AVS ExpressRoute | 4 |
| Velocidad de puerto de AVS ExpressRoute | 10 Gbps<br />La puerta de enlace de red virtual usada determina el ancho de banda real. Para más detalles, vea [Acerca de las puertas de enlace de red virtual de ExpressRoute](../../expressroute/expressroute-about-virtual-network-gateways.md). | 
| Direcciones IP públicas expuestas a través de vWAN | 100 |
| Límites de capacidad de vSAN | 75 % del total que se puede usar (el 25 % restante se mantiene disponible para el contrato de nivel de servicio)  |

En el caso de otros límites específicos de VMware, use la [herramienta de configuración máxima de VMware](https://configmax.vmware.com/).
