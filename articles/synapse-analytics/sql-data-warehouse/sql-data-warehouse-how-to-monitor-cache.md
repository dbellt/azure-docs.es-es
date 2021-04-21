---
title: Optimización de la caché de Gen2
description: Obtenga información sobre cómo supervisar la caché de Gen2 mediante Azure Portal.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 9de795c54f55295fa69ed7fcb5dd894e2963385b
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107566637"
---
# <a name="how-to-monitor-the-adaptive-cache"></a>Supervisión de la memoria caché adaptable

En este artículo se describe cómo supervisar y solucionar problemas de rendimiento lento de las consultas determinando si la carga de trabajo aprovecha de forma óptima la memoria caché adaptable de los grupos de SQL dedicados.

La arquitectura de almacenamiento de grupos de SQL dedicados organiza de forma automática por niveles los segmentos de almacén de columnas consultados con más frecuencia en una memoria caché que reside en discos SSD basados en NVMe. El aumento de rendimiento se consigue cuando las consultas recuperan segmentos que se encuentran en la caché.
 
## <a name="troubleshoot-using-the-azure-portal"></a>Solución de problemas mediante Azure Portal

Puede usar a Azure Monitor para ver las métricas de caché para solucionar los problemas de rendimiento de las consultas. En primera lugar, vaya a Azure Portal y haga clic en **Supervisar**, **Métricas** y **+ Seleccionar un ámbito**:

![Captura de pantalla en la que se muestra la opción Seleccionar un ámbito seleccionada en Métricas en Azure Portal.](./media/sql-data-warehouse-how-to-monitor-cache/cache-0.png)

Use las barras desplegable y de búsqueda para buscar el grupo de SQL dedicado. Luego seleccione Aplicar.

![Captura de pantalla en la que se muestra el panel Seleccionar un ámbito donde puede seleccionar el almacenamiento de datos.](./media/sql-data-warehouse-how-to-monitor-cache/cache-1.png)

Las métricas clave para solucionar problemas de la caché son **Porcentaje de aciertos de caché** y **Porcentaje de caché usada**. Seleccione **Porcentaje de aciertos de caché** y luego use el botón **Agregar métrica** para agregar **Porcentaje de caché usada**. 

![Métricas de caché](./media/sql-data-warehouse-how-to-monitor-cache/cache-2.png)

## <a name="cache-hit-and-used-percentage"></a>Aciertos de caché y Porcentaje de caché usada

En la tabla siguiente se describen escenarios en función de los valores de las métricas de caché:

|                                | **Porcentaje de aciertos de caché alto** | **Porcentaje de aciertos de caché bajo** |
| :----------------------------: | :---------------------------: | :--------------------------: |
| **Porcentaje de caché usada alto** |          Escenario 1           |          Escenario 2          |
| **Porcentaje de caché usada bajo**  |          Escenario 3           |          Escenario 4          |

**Escenario 1:** la caché se está usando de forma óptima. [Solucione los problemas](sql-data-warehouse-manage-monitor.md) de otras áreas que puedan ralentizar las consultas.

**Escenario 2:** el conjunto de datos de trabajo actual no cabe en la caché, lo que provoca un porcentaje de aciertos de caché bajo debido a las lecturas físicas. Considere la posibilidad de escalar verticalmente el nivel de rendimiento y vuelva a ejecutar la carga de trabajo para rellenar la caché.

**Escenario 3:** es probable que la consulta se ejecute lentamente debido a razones no relacionadas con la caché. [Solucione los problemas](sql-data-warehouse-manage-monitor.md) de otras áreas que puedan ralentizar las consultas. También puede considerar la posibilidad de [reducir verticalmente la instancia](sql-data-warehouse-manage-monitor.md) para reducir el tamaño de la caché con el fin de ahorrar costos. 

**Escenario 4:** tenía una caché inactiva que podría ser el motivo de la lentitud de la consulta. Considere la posibilidad de volver a ejecutar la consulta, ya que ahora el conjunto de datos de trabajo debería estar almacenado en la caché. 

> [!IMPORTANT]
> Si el porcentaje de aciertos de caché o el porcentaje de caché usada no se actualiza después de volver a ejecutar la carga de trabajo, es posible que el conjunto de trabajo ya se encuentre en la memoria. En la caché solo se guardan las tablas de almacén de columnas en clúster.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre la optimización del rendimiento general de las consultas, vea [Supervisión de ejecuciones de consultas](sql-data-warehouse-manage-monitor.md#monitor-query-execution).
