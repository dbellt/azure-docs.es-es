---
title: Información sobre los reinicios de máquinas virtuales
description: 'Información sobre los reinicios de máquinas virtuales: mantenimiento frente a tiempo de inactividad'
author: mimckitt
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 03/08/2021
ms.author: mimckitt
ms.reviewer: cynthn
ms.openlocfilehash: d9d53b4e6b7237c93c6203351463515bb2b34d26
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125308"
---
# <a name="understand-vm-reboots---maintenance-vs-downtime"></a>Información sobre los reinicios de máquinas virtuales: mantenimiento frente a tiempo de inactividad
Hay tres escenarios que pueden afectar a las máquinas virtuales de Azure: mantenimiento de hardware no planeado, tiempo de inactividad inesperado y mantenimiento planeado.

## <a name="unplanned-hardware-maintenance-event"></a>Evento de mantenimiento de hardware no planeado
El mantenimiento de hardware no planeado se produce cuando la plataforma de Azure predice que en el hardware o en cualquier componente de la plataforma asociado a una máquina física está a punto de producirse un error. Cuando la plataforma predice un error, se emitirá un evento de mantenimiento de hardware no planeado para reducir el efecto en las máquinas virtuales hospedadas en ese hardware. Azure usa la tecnología [Migración en vivo](./maintenance-and-updates.md) para migrar las máquinas virtuales del hardware con errores a un equipo físico en buen estado. La migración en vivo es una operación de conservación de máquinas virtuales que solo detiene la máquina virtual durante un breve período. Se mantienen la memoria, los archivos abiertos y las conexiones de red pero el rendimiento puede verse reducido antes o después del evento. En los casos en los que no se puede usar la migración en vivo, la máquina virtual experimentará tiempos de inactividad inesperados, tal y como se describe a continuación.


## <a name="unexpected-downtime"></a>Tiempo de inactividad inesperado
El tiempo de inactividad inesperado se produce cuando en el hardware o en la infraestructura física de la máquina virtual se produce un error de forma imprevista. Entre estos podemos encontrar errores de la red local, errores de los discos locales u otras errores a nivel de bastidor. Cuando se detecta, la plataforma de Azure migra (recupera) automáticamente la máquina virtual a una máquina física en estado correcto en el mismo centro de datos. Durante el procedimiento de recuperación, las máquinas virtuales experimentan tiempos de inactividad (reinicio) y, en algunos casos, pérdidas de la unidad temporal. El sistema operativo y los discos de datos asociados siempre se conservan.

Las máquinas virtuales también pueden experimentar tiempos de inactividad en el improbable caso de que una interrupción o desastre afecte a todo un centro de datos, o incluso a toda una región. Para estos casos, Azure proporciona opciones de protección que incluyen [zonas de disponibilidad](../availability-zones/az-overview.md) y [regiones emparejadas](regions.md#region-pairs).

## <a name="planned-maintenance-events"></a>Eventos de mantenimiento planeados
Eventos de mantenimiento planeado son actualizaciones periódicas realizadas por Microsoft en la plataforma Azure subyacente para mejorar en general la fiabilidad, el rendimiento y la seguridad de la infraestructura de la plataforma sobre las que se funcionan sus máquinas virtuales. La mayoría de estas actualizaciones se realizan sin que Virtual Machines ni Cloud Services resulten afectados (consulte [Mantenimiento que no requiere un reinicio](maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot)). Aunque la plataforma Azure intente utilizar el mantenimiento de conservación de máquinas virtuales en todas las ocasiones posibles, existen algunos casos poco frecuentes en los que estas actualizaciones requieren un reinicio de la máquina virtual para aplicar las actualizaciones necesarias a la infraestructura subyacente. En este caso, puede realizar un mantenimiento planeado de Azure con la operación de reimplementación de mantenimiento mediante el inicio del mantenimiento para sus máquinas virtuales en el período de tiempo adecuado. Para más información, consulte [Mantenimiento planeado de máquinas virtuales](maintenance-and-updates.md).

## <a name="reduce-downtime"></a>Reducción del tiempo de inactividad
Para reducir el impacto del tiempo de parada debido a uno o más de estos eventos, recomendamos las siguientes mejores prácticas de alta disponibilidad para las máquinas virtuales:

* Uso de [Availability Zones](../availability-zones/az-overview.md) para protegerse frente a errores del centro de datos
* Configuración de varias máquinas virtuales en un [conjunto de disponibilidad](availability-set-overview.md) para la redundancia
* Uso de [eventos programados para Linux](./linux/scheduled-events.md) o [eventos programados para Windows](./windows/scheduled-events.md) para responder proactivamente a los eventos que afectan a la máquina virtual
* Configure cada nivel de aplicación en conjuntos separados de disponibilidad
* Combinación de un [equilibrador de carga](../load-balancer/load-balancer-overview.md) con conjuntos o zonas de disponibilidad

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las opciones de disponibilidad en Azure, consulte [Opciones de disponibilidad para máquinas virtuales de Azure](availability.md).