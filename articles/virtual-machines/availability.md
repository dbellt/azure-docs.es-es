---
title: Opciones de disponibilidad para máquinas virtuales de Azure
description: Obtenga información sobre las opciones de disponibilidad para ejecutar máquinas virtuales en Azure.
author: mimckitt
ms.author: mimckitt
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 03/08/2021
ms.reviewer: cynthn
ms.openlocfilehash: 2acd6fe8450999b4518f19a353d0eab87ddedfaf
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108146426"
---
# <a name="availability-options-for-azure-virtual-machines"></a>Opciones de disponibilidad para máquinas virtuales de Azure
En este artículo se proporciona una visión general de las opciones de disponibilidad de las máquinas virtuales (VM) de Azure.

## <a name="availability-zones"></a>Zonas de disponibilidad

Las [zonas de disponibilidad](../availability-zones/az-overview.md?context=/azure/virtual-machines/context/context) expanden el nivel de control que tiene para mantener la disponibilidad de las aplicaciones y los datos en las máquinas virtuales. Una zona de disponibilidad es una zona separada físicamente dentro de una región de Azure. Hay tres zonas de disponibilidad por cada región de Azure admitida. 

Cada zona de disponibilidad tiene una fuente de alimentación, una red y un sistema de refrigeración distintos. Si diseña las soluciones para que usen máquinas virtuales replicadas en zonas, puede proteger sus datos y aplicaciones frente a la pérdida de un centro de datos. Aunque una zona esté en peligro, las aplicaciones y los datos replicados estarán disponibles instantáneamente en otra zona. 

## <a name="availability-sets"></a>Conjuntos de disponibilidad
Un [conjunto de disponibilidad](availability-set-overview.md) es una agrupación lógica de máquinas virtuales que permite a Azure conocer cómo se crea su aplicación para proporcionar redundancia y disponibilidad. Se recomienda la creación de dos, o más, máquinas virtuales en un conjunto de disponibilidad no solo para proporcionar una aplicación de alta disponibilidad sino también para cumplir el [99,95 % del Acuerdo de Nivel de Servicio de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). No hay ningún costo asociado con el conjunto de disponibilidad propiamente dicho, solo se paga por cada instancia de máquina virtual que cree.


## <a name="virtual-machines-scale-sets"></a>Conjuntos de escalado de máquinas virtuales 

Los [conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/overview.md?context=/azure/virtual-machines/context/context) permiten crear y administrar un grupo de máquinas virtuales con equilibrio de carga. El número de instancias de máquina virtual puede aumentar o disminuir automáticamente según la demanda, o de acuerdo a una programación definida. Los conjuntos de escalado proporcionan una alta disponibilidad a las aplicaciones y le permiten administrar, configurar y actualizar de forma centralizada muchas máquinas virtuales. Se recomienda la creación de dos o más máquinas virtuales en un conjunto de escalado para proporcionar una aplicación de alta disponibilidad y para cumplir el [99,95 % del Acuerdo de Nivel de Servicio de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). No hay ningún costo asociado con el conjunto de escalado propiamente dicho, solo se paga por cada instancia de máquina virtual que cree.

Las máquinas virtuales de un conjunto de escalado también se pueden implementar en una sola zona de disponibilidad o en regiones. Las opciones de implementación de zona de disponibilidad pueden diferir según el [modo de orquestación](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md?context=/azure/virtual-machines/context/context).

## <a name="load-balancer"></a>Equilibrador de carga
Combine [Azure Load Balancer](../load-balancer/load-balancer-overview.md) con un conjunto o zona de disponibilidad para aprovechar al máximo la resistencia de la aplicación. El equilibrador de carga de Azure distribuye el tráfico entre varias máquinas virtuales. El equilibrador de carga de Azure está incluido en nuestras máquinas virtuales de niveles estándar. No todos los niveles de las máquinas virtuales incluyen Azure Load Balancer. Para obtener más información sobre el equilibrio de carga en máquinas virtuales, consulte **Equilibrio de carga de máquinas virtuales** para [Linux](linux/tutorial-load-balancer.md) o [Windows](windows/tutorial-load-balancer.md).


## <a name="azure-storage-redundancy"></a>Redundancia de Azure Storage
Azure Storage siempre almacena varias copias de los datos, con el fin de protegerlos de eventos planeados y no planeados, como errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red y desastres naturales masivos. La redundancia garantiza que la cuenta de almacenamiento cumple sus objetivos de disponibilidad y durabilidad, aunque se produzcan errores.

A la hora de decidir qué opción de redundancia es la más adecuada para su escenario, intente buscar un equilibrio entre bajo costo y alta disponibilidad. Entre los factores que ayudan a determinar qué opción de redundancia debe elegir se incluye:
- Cómo se replican los datos en la región primaria.
- Si los datos se replicarán en una segunda ubicación que está alejada geográficamente de la región primaria, para protegerse frente a desastres regionales.
- Si la aplicación necesita acceso de lectura a los datos replicados en la región secundaria en caso de que la región primaria deje de estar disponible por cualquier motivo.

Para más información, consulte [Redundancia de Azure Storage](../storage/common/storage-redundancy.md).

## <a name="azure-site-recovery"></a>Azure Site Recovery
Como organización, necesita adoptar una estrategia de continuidad empresarial y de recuperación ante desastres (BCDR) que mantenga sus datos seguros, y sus aplicaciones y cargas de trabajo en línea cuando se produzcan interrupciones planeadas o imprevistas.

[Azure Site Recovery](../site-recovery/site-recovery-overview.md) ayuda a garantizar la continuidad empresarial manteniendo las aplicaciones y cargas de trabajo empresariales en funcionamiento durante las interrupciones. Site Recovery replica las cargas de trabajo que se ejecutan en máquinas físicas y virtuales desde un sitio principal a una ubicación secundaria. Cuando se produce una interrupción en el sitio principal, se conmuta por error a la ubicación secundaria y se accede desde allí a las aplicaciones. Cuando la ubicación principal vuelva a estar en ejecución, puede realizar la conmutación por recuperación en ella.

Site Recovery puede administrar la replicación de:
- Máquinas virtuales de Azure que se replican entre regiones de Azure.
- Máquinas virtuales locales, máquinas virtuales de Azure Stack y servidores físicos.

## <a name="next-steps"></a>Pasos siguientes
- [Creación de una máquina virtual en una zona de disponibilidad](./linux/create-cli-availability-zone.md)
- [Creación de una máquina virtual en un conjunto de disponibilidad](./linux/tutorial-availability-sets.md)
- [Creación de un conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/quick-create-portal.md)