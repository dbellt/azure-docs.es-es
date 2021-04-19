---
title: 'Reequilibrio de particiones en Hiperescala (Citus): Azure Database for PostgreSQL'
description: Distribuya las particiones de manera uniforme entre servidores para mejorar el rendimiento.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 04/09/2021
ms.openlocfilehash: 63322fac4c6ad5b705deedcd8a80466ddd803814
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305708"
---
# <a name="rebalance-shards-in-hyperscale-citus-server-group"></a>Reequilibrio de particiones en un grupo de servidores de Hiperescala (Citus)

Para aprovechar los nodos recién agregados, tiene que reequilibrar la tabla distribuida [particiones de base de datos](concepts-hyperscale-distributed-data.md#shards), lo que significa mover algunas particiones de los nodos existentes a las nuevas.

## <a name="determine-if-the-server-group-needs-a-rebalance"></a>Determinación de si el grupo de servidores necesita un reequilibrio

Azure Portal puede mostrar si los datos se distribuyen equitativamente entre los nodos de trabajo de un grupo de servidores. Para verlo, vaya a la página **Shard rebalancer** (Reequilibrador de particiones) en el menú **Server group management** (Administración de grupos de servidores). Si los datos se distorsionan entre trabajos, verá el mensaje **Rebalancing is recommended** (Se recomienda reequilibrar), junto con una lista del tamaño de cada nodo.

Si los datos ya están equilibrados, verá el mensaje **Rebalancing is not recommended at this time**(No se recomienda el reequilibrio en este momento).

## <a name="run-the-shard-rebalancer"></a>Ejecución del reequilibrador de particiones

Para iniciar el reequilibrador de particiones, debe conectarse al nodo de coordinación del grupo de servidores y ejecutar la función SQL [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards) en las tablas distribuidas. La función vuelve a equilibrar todas las tablas del grupo [coubicación](concepts-hyperscale-colocation.md) de la tabla nombrada en su argumento. Por lo tanto, no tiene que llamar a la función para cada tabla distribuida, simplemente llámela en una tabla representativa de cada grupo de ubicación.

```sql
SELECT rebalance_table_shards('distributed_table_name');
```

## <a name="monitor-rebalance-progress"></a>Supervisión del progreso del reequilibrio

Para inspeccionar el reequilibrador después de iniciarlo, vuelva a Azure Portal. Abra la página **Shard rebalancer** (Reequilibrador de particiones) en **Server group management** (Administración de grupos de servidores). Se mostrará el mensaje **Rebalancing is underway** (El reequilibrio está en curso) junto con dos tablas.

En la primera tabla se muestra el número de particiones que se mueven dentro o fuera de un nodo, por ejemplo, "6 de 24 movidos dentro". En la segunda tabla se muestra el progreso por tabla de base de datos: nombre, recuento de particiones afectadas, tamaño de datos afectados y estado de reequilibrio.

Seleccione el botón **Actualizar** para actualizar la página. Una vez completado el reequilibrio, se volverá a indicar **Rebalancing is not recommended at this time** (No se recomienda el reequilibrio en este momento).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el grupo de servidores en [opciones de rendimiento](concepts-hyperscale-configuration-options.md).
- [Escalado de un grupo de servidores](howto-hyperscale-scale-grow.md), vertical u horizontal.
- Consulte el material de referencia de [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards).
