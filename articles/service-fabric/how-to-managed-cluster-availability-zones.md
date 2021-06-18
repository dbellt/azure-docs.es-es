---
title: Implementación de un clúster administrado de Service Fabric en zonas de disponibilidad
description: Aprenda a implementar un clúster administrado de Service Fabric en zonas de disponibilidad y a configurarlo en una plantilla de ARM.
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 5278ea170e0a60907813b9a79b151dde44ab4a12
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111956790"
---
# <a name="deploy-a-service-fabric-managed-cluster-across-availability-zones"></a>Implementación de un clúster administrado de Service Fabric en zonas de disponibilidad

Las zonas de disponibilidad de Azure son una función de alta disponibilidad que protege las aplicaciones y los datos frente a errores del centro de datos. Una zona de disponibilidad es una ubicación física única equipada con alimentación independiente, refrigeración y redes dentro de una región de Azure.

El clúster administrado de Service Fabric admite implementaciones que abarcan varias zonas de disponibilidad para proporcionar resistencia de zona. Esta configuración garantizará la alta disponibilidad de los servicios críticos del sistema y las aplicaciones para protegerse frente a puntos de error únicos. Azure Availability Zones solo está disponible en algunas regiones. Consulte [Azure Availability Zones Overview](../availability-zones/az-overview.md) (Información general de Azure Availability Zones).

>[!NOTE]
>La distribución de zonas de disponibilidad solo está disponible en clústeres de SKU Estándar.

Están disponibles plantillas de ejemplo: [Plantilla de Service Fabric en zona de disponibilidad](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="recommendations-for-zone-resilient-azure-service-fabric-managed-clusters"></a>Recomendaciones para clústeres administrados de Azure Service Fabric resistentes a zonas
Un clúster de Service Fabric distribuido en Availability Zones garantiza el estado de alta disponibilidad del clúster. 

La topología recomendada para el clúster administrado requiere los recursos que se indican a continuación:

* La SKU de clúster debe ser Estándar.
* El tipo de nodo principal debe tener al menos nueve nodos para obtener la mejor resistencia, aunque se admite un número mínimo de seis.
* Los tipos de nodos principales deben tener al menos seis nodos para obtener la mejor resistencia, aunque se admite un número mínimo de tres.

>[!NOTE]
>Solo se admiten tres implementaciones de zona de disponibilidad.

>[!NOTE]
> No es posible realizar un cambio local de clúster administrado en el que se pase de un clúster no distribuido a uno distribuido.

Diagrama que muestra la arquitectura de la zona de disponibilidad de Azure Service Fabric ![Arquitectura de la zona de disponibilidad de Azure Service Fabric][sf-multi-az-arch]

Lista de nodos de ejemplo que muestra los formatos FD/UD en las zonas de extensión de un conjunto de escalado de máquinas virtuales.

 ![Lista de nodos de ejemplo que muestra los formatos FD/UD en las zonas de extensión de un conjunto de escalado de máquinas virtuales.][sfmc-multi-az-nodes]

**Distribución de las réplicas de servicio entre zonas**: cuando se implementa un servicio en los elementos nodeTypes que son zonas distribuidas, las réplicas se colocan para asegurarse de que están en zonas independientes. Esta separación queda garantizada ya que los dominios de error de los nodos presentes en cada uno de estos nodeTypes se configuran con la información de zona (es decir, FD = fd:/zone1/1 etc.). Por ejemplo: para cinco réplicas o instancias de un servicio, la distribución será 2-2-1 y el entorno de ejecución intentará asegurarse de que la distribución sea la misma en las zonas de disponibilidad.

**Configuración de la réplica de servicio de usuario**: los servicios de usuario con estado implementados en la zona de disponibilidad cruzada nodeTypes deben configurarse con esta opción: recuento de réplicas con destino = 9, min = 5. Esta configuración le ayudará a garantizar que el servicio funcione incluso cuando una zona deje de funcionar, ya que las 6 réplicas seguirán activas en las otras dos zonas. Una actualización de la aplicación en este escenario también se realizará.

**Escenario de una zona que no funciona**: cuando una zona deja de funcionar, todos los nodos de esa zona aparecerán como inactivos. Las réplicas de servicio en estos nodos también estarán inactivas. Dado que hay réplicas en las otras zonas, el servicio sigue teniendo capacidad de respuesta con las réplicas principales que conmutan por error a las zonas que funcionan. Los servicios aparecerán en estado de advertencia, ya que el número de réplicas de destino no se ha alcanzado y el número de VM todavía es mayor que el tamaño mínimo de réplica de destino definido. Como resultado, el equilibrador de carga de Service Fabric abrirá réplicas en las zonas en funcionamiento para que coincidan con el número de réplicas de destino configurado. En este momento, los servicios aparecerán en buen estado. Cuando la zona que estaba inactiva vuelva a funcionar, el equilibrador de carga volverá a distribuir todas las réplicas de servicio uniformemente en todas las zonas.

## <a name="networking-configuration"></a>Configuración de redes
Para más información, consulte [Configuración de la red para clústeres administrados de Service Fabric](./how-to-managed-cluster-networking.md).

## <a name="enabling-a-zone-resilient-azure-service-fabric-managed-cluster"></a>Habilitación de un clúster administrado de Azure Service Fabric resistente a zona
Para habilitar un clúster administrado de Azure Service Fabric resistente a zona, debe incluir lo siguiente en la definición de recursos del clúster administrado.

* La propiedad **ZonalResiliency**, que especifica si el clúster es resistente a zona o no.

```json
{
    "apiVersion": "2021-05-01",
    "type": "Microsoft.ServiceFabric/managedclusters",
    "ZonalResiliency": "true"
    
}
```
[sf-architecture]: ./media/service-fabric-cross-availability-zones/sf-cross-az-topology.png
[sf-architecture]: ./media/service-fabric-cross-availability-zones/sf-cross-az-topology.png
[sf-multi-az-arch]: ./media/service-fabric-cross-availability-zones/sf-multi-az-topology.png
[sfmc-multi-az-nodes]: ./media/how-to-managed-cluster-availability-zones/sfmc-multi-az-nodes.png