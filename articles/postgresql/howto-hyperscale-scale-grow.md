---
title: Escalado de un grupo de servidor de Hiperescala (Citus) en Azure Database for PostgreSQL
description: Ajuste la memoria del grupo de servidores, el disco y los recursos de CPU para tratar un aumento de la carga
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 04/07/2021
ms.openlocfilehash: 9746c6509673d3268a4afa15bcbeee9fa676d8c1
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111554415"
---
# <a name="scale-a-hyperscale-citus-server-group"></a>Escalado de un grupo de servidores Hiperescala (Citus)

Azure Database for PostgreSQL:Hiperescala (Citus) proporciona escalado de autoservicio para tratar el aumento de la carga. Azure Portal permite agregar nuevos nodos de trabajo y aumentar los núcleos virtuales de los nodos existentes. Agregar nodos no provoca ningún tiempo de inactividad e incluso mover particiones a los nuevos nodos (llamado [reequilibrio de particiones](howto-hyperscale-scale-rebalance.md)) se produce sin interrumpir las consultas.

## <a name="add-worker-nodes"></a>Incorporación de nodos de trabajo

Para agregar nodos, vaya a la pestaña **Compute + storage** (Proceso y almacenamiento) en el grupo de servidores de Hiperescala (Citus).  Arrastre el control deslizante de **Números de nodo de trabajo** para cambiar el valor.

> [!NOTE]
>
> Un grupo de servidores de Hiperescala (Citus) creado con el [nivel básico (versión preliminar)](concepts-hyperscale-tiers.md) no tiene ningún trabajo. Al aumentar el número de trabajos, el grupo de servidores se convierte automáticamente en el nivel estándar.
> Después de convertir un grupo de servidores al nivel estándar, no puede cambiarlo de nuevo al nivel básico.

:::image type="content" source="./media/howto-hyperscale-scaling/01-sliders-workers.png" alt-text="Controles deslizantes de recursos":::

Haga clic en el botón **Guardar** para que el valor cambiado surta efecto.

> [!NOTE]
> Una vez que se ha aumentado y guardado, el número de nodos de trabajo no se puede reducir con el control deslizante.

> [!NOTE]
> Para aprovechar los nodos recién agregados, tiene que [reequilibrar las particiones de la tabla distribuida](howto-hyperscale-scale-rebalance.md), lo que significa mover algunas [particiones](concepts-hyperscale-distributed-data.md#shards) de los nodos existentes a los nuevos.

## <a name="increase-or-decrease-vcores-on-nodes"></a>Aumentar o disminuir los núcleos virtuales de los nodos

Además de agregar nodos nuevos, puede aumentar las capacidades de los nodos existentes. Ajustar la capacidad de proceso hacia arriba y hacia abajo puede ser útil en los experimentos de rendimiento, así como en el caso de cambios a corto o largo plazo en las demandas de tráfico.

Para cambiar los núcleos virtuales de todos los nodos de trabajo, ajuste el control deslizante **Núcleos virtuales** en **Configuración (por nodo de trabajo)** . Los núcleos virtuales del nodo de coordinación se pueden ajustar de forma independiente. Ajuste el control deslizante **Núcleos virtuales** en **Configuración (nodo de coordinación)** .

## <a name="increase-storage-on-nodes"></a>Aumento del almacenamiento en los nodos

Además de agregar nuevos nodos, puede aumentar el espacio en disco de los nodos existentes. Aumentar el espacio en disco puede permitirle hacer más cosas con los nodos de trabajo existentes antes de necesitar agregar otros.

Para cambiar el almacenamiento de todos los nodos de trabajo, ajuste el control deslizante de **almacenamiento** en **Configuración (por nodo de trabajo)** . El almacenamiento del nodo de coordinación se puede ajustar de forma independiente. Ajuste el control deslizante de **almacenamiento** en **Configuración (nodo de coordinación)** .

> [!NOTE]
> Una vez que se ha aumentado y guardado, el almacenamiento por nodo no se puede reducir con el control deslizante.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el grupo de servidores en [opciones de rendimiento](concepts-hyperscale-configuration-options.md).
- [Reequilibre las particiones de tablas distribuidas](howto-hyperscale-scale-rebalance.md) de modo que todos los nodos de trabajo puedan participar en consultas paralelas.
