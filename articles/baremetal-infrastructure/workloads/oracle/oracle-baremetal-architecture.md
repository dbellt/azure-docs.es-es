---
title: Arquitectura de BareMetal Infrastructure para Oracle
description: Obtenga información acerca de la arquitectura de varias configuraciones de BareMetal Infrastructure para Oracle.
ms.topic: reference
ms.subservice: baremetal-oracle
ms.date: 04/14/2021
ms.openlocfilehash: 1e4a00f53018647ca3b3528a9881ec36af067e28
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578643"
---
# <a name="architecture-of-baremetal-infrastructure-for-oracle"></a>Arquitectura de BareMetal Infrastructure para Oracle

En este artículo, se explicarán las opciones de arquitectura de BareMetal Infrastructure para Oracle y las características que admite cada una.

## <a name="single-instance"></a>Instancia única

Esta topología admite una única instancia de Oracle Database con Oracle Data Guard para migrar a BareMetal Infrastructure. Admite el uso de nodos en espera para los trabajos de alta disponibilidad y mantenimiento.

[![Diagrama que muestra la arquitectura de una única instancia de Oracle Database con Oracle Data Guard.](media/oracle-baremetal-architecture/single-instance-architecture.png)](media/oracle-baremetal-architecture/single-instance-architecture.png#lightbox)

## <a name="oracle-real-application-clusters-rac-one-node"></a>Oracle Real Application Clusters (RAC) One Node

Esta topología admite una configuración de RAC con almacenamiento compartido y clústeres de Grid. Las instancias de base de datos solo se ejecutan en un nodo (configuración activo-pasivo).

Características incluidas:

- Configuración activo-pasivo con Oracle RAC One Node

    - Conmutación por error automática

    - Reinicio rápido en el segundo nodo

- Conmutación por error y escalabilidad en tiempo real con Oracle RAC

- Mantenimiento gradual sin tiempo de inactividad

[![Diagrama que muestra la arquitectura de una configuración activo-pasivo de Oracle RAC One Node.](media/oracle-baremetal-architecture/one-node-rac-architecture.png)](media/oracle-baremetal-architecture/one-node-rac-architecture.png#lightbox)

## <a name="rac"></a>RAC

Esta topología admite una configuración de Oracle RAC con almacenamiento compartido y un clúster de Grid, mientras se ejecutan simultáneamente varias instancias por base de datos (configuración activo-activo).

- El rendimiento se puede escalar fácilmente mediante el aprovisionamiento en línea de servidores agregados. 
-  Los usuarios están activos en todos los servidores, y todos los servidores comparten acceso a la misma instancia de Oracle Database. 
-  Todos los tipos de mantenimiento de bases de datos se pueden realizar en línea o de forma gradual para lograr un tiempo de inactividad mínimo o inexistente. 
- Los sistemas en espera de Oracle Active Data Guard (ADG) pueden cumplir fácilmente un doble propósito como sistemas de prueba. 

Esta configuración permite probar todos los cambios en una copia exacta de la base de datos de producción antes de que se apliquen al entorno de producción.

> [!NOTE]
> Si piensa usar Active Data Guard Far Sync (modo sincrónico), deberá tener en cuenta las zonas regionales en las que se admite esta característica. Solo en el caso de las regiones distribuidas geográficamente, se recomienda usar Data Guard con el modo asincrónico.

[![Diagrama que muestra la arquitectura de una configuración activo-activo de Oracle RAC.](media/oracle-baremetal-architecture/rac-architecture.png)](media/oracle-baremetal-architecture/rac-architecture.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca del aprovisionamiento de las instancias de BareMetal para las cargas de trabajo de Oracle.

> [!div class="nextstepaction"]
> [Aprovisionamiento de BareMetal Infrastructure para Oracle](oracle-baremetal-provision.md)

