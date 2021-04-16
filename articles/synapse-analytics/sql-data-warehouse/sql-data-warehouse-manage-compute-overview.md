---
title: Administración de recursos de proceso para un grupo de SQL dedicado (anteriormente SQL DW)
description: Conozca las funcionalidades de escalabilidad horizontal del grupo de SQL dedicado (anteriormente SQL DW) de Azure Synapse Analytics. Puede escalar horizontalmente mediante el ajuste de las DWU o detener el grupo de SQL dedicado para reducir los costos (anteriormente SQL DW).
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/12/2019
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 8c44e0570b08bba852d57fbb25da05d778ea8a0d
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105960119"
---
# <a name="manage-compute-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Administración del proceso para un grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics

Obtenga información sobre cómo administrar los recursos de proceso para un grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics. Para reducir los costos, detenga el grupo de SQL dedicado o escale el grupo de SQL dedicado para satisfacer las demandas de rendimiento.

## <a name="what-is-compute-management"></a>¿Qué es la administración de proceso?

La arquitectura del grupo de SQL dedicado (anteriormente SQL DW) separa el proceso y el almacenamiento, lo que permite a cada uno escalar de manera independiente. Como resultado, puede escalar el proceso para satisfacer las demandas de rendimiento con independencia del almacenamiento de datos. También puede pausar y reanudar los recursos de proceso. Una consecuencia natural de esta arquitectura es que la [facturación](https://azure.microsoft.com/pricing/details/sql-data-warehouse/) por proceso y almacenamiento está separada. Si no es necesario usar el grupo de SQL dedicado (anteriormente SQL DW) durante un tiempo, puede detener el proceso para ahorrar costos de proceso.

## <a name="scaling-compute"></a>Escalado de proceso

Para escalar horizontalmente el proceso o reducir su escala, ajuste el valor de las [unidades de almacenamiento de datos](what-is-a-data-warehouse-unit-dwu-cdwu.md) del grupo de SQL dedicado (anteriormente SQL DW). La carga y el rendimiento de las consultas pueden aumentar linealmente a medida que agrega más unidades de almacenamiento de datos.

Para conocer los pasos de escalado horizontal, consulte las guías de inicio rápido de [Azure Portal](quickstart-scale-compute-portal.md), [PowerShell](quickstart-scale-compute-powershell.md) o [T-SQL](quickstart-scale-compute-tsql.md). También puede realizar operaciones de escalado horizontal con una [REST API](sql-data-warehouse-manage-compute-rest-api.md#scale-compute).

Para realizar una operación de escalado, el grupo de SQL dedicado (anteriormente SQL DW) termina primero con todas las consultas de entrada y, luego, revierte las transacciones para garantizar un estado coherente. El escalado solo se produce una vez finalizada la reversión de la transacción. En una operación de escalado, el sistema desasocia la capa de almacenamiento de los nodos de proceso, agrega nodos de proceso y, luego, vuelve a asociar la capa de almacenamiento a la capa de proceso. Cada grupo de SQL dedicado (anteriormente SQL DW) se almacena como 60 distribuciones, que se distribuyen de manera uniforme a los nodos de proceso. Al agregar más nodos de proceso, se agrega más capacidad de proceso. A medida que aumenta el número de nodos de proceso, se reduce el número de distribuciones por nodo de proceso, de forma que hay más capacidad de proceso para las consultas. De igual manera, al reducir las unidades de almacenamiento de datos se reduce el número de nodos de proceso, de forma que se reducen los recursos de proceso para las consultas.

En la tabla siguiente se muestra cómo cambia el número de distribuciones por nodo de proceso a medida que cambian las unidades de almacenamiento de datos.  DW30000c proporciona 60 nodos de proceso y consigue un rendimiento mucho mayor que DW100c.

| Unidades de almacenamiento de datos  | \# de nodos de proceso | \# de distribuciones por nodo |
| -------- | ---------------- | -------------------------- |
| DW100c   | 1                | 60                         |
| DW200c   | 1                | 60                         |
| DW300c   | 1                | 60                         |
| DW400c   | 1                | 60                         |
| DW500c   | 1                | 60                         |
| DW1000c  | 2                | 30                         |
| DW1500c  | 3                | 20                         |
| DW2000c  | 4                | 15                         |
| DW2500c  | 5                | 12                         |
| DW3000c  | 6                | 10                         |
| DW5000c  | 10               | 6                          |
| DW6000c  | 12               | 5                          |
| DW7500c  | 15               | 4                          |
| DW10000c | 20               | 3                          |
| DW15000c | 30               | 2                          |
| DW30000c | 60               | 1                          |

## <a name="finding-the-right-size-of-data-warehouse-units"></a>Búsqueda del tamaño adecuado de las unidades de almacenamiento de datos

Para ver las ventajas de rendimiento del escalado horizontal, en especial para unidades de almacenamiento de datos más grandes, querrá usar al menos un conjunto de datos de 1 TB. Para encontrar el mejor número de unidades de almacenamiento de datos para el grupo de SQL dedicado (anteriormente SQL DW), pruebe a escalarlo y reducirlo verticalmente. Ejecute algunas consultas con distintos números de unidades de almacenamiento de datos después de cargar los datos. Dado que el escalado se realiza rápidamente, puede probar varios niveles de rendimiento en una hora o menos.

Recomendaciones para encontrar el mejor número de unidades de almacenamiento de datos:

- Para un grupo de SQL dedicado (anteriormente SQL DW) en desarrollo, comience por seleccionar un número más pequeño de unidades de almacenamiento de datos.  Un buen punto de partida es DW400c o DW200c.
- Supervise el rendimiento de la aplicación y observe el número de unidades de almacenamiento de datos seleccionadas en comparación con el rendimiento observado.
- Suponga una escala lineal y determine cuánto debe aumentar o reducir las unidades de almacenamiento de datos.
- Continúe realizando ajustes hasta llegar a un nivel de rendimiento adecuado para sus requerimientos empresariales.

## <a name="when-to-scale-out"></a>Cuándo escalar horizontalmente

El escalado horizontal de unidades de almacenamiento de datos afecta a los siguientes aspectos de rendimiento:

- Mejora linealmente el rendimiento del sistema en lo referente a exámenes, agregaciones e instrucciones CTAS.
- Aumenta el número de lectores y escritores para cargar los datos.
- Número máximo de consultas simultáneas y ranuras de simultaneidad.

Recomendaciones para saber cuándo escalar horizontalmente las unidades de almacenamiento de datos:

- Para realizar una operación de transformación o carga de con muchos datos, realice el escalado horizontal para que los datos estén disponibles con mayor rapidez.
- Durante las horas punta comerciales, escale horizontalmente para dar cabida a un mayor número de consultas simultáneas.

## <a name="what-if-scaling-out-does-not-improve-performance"></a>¿Qué ocurre si escalado horizontal no mejora el rendimiento?

La adición de unidades de almacenamiento de datos aumenta el paralelismo. Si el trabajo se divide uniformemente entre los nodos de proceso, el paralelismo adicional mejora el rendimiento de las consultas. Si el escalado horizontal no cambia el rendimiento, hay algunos motivos por los que puede ocurrir. Puede que los datos se sesguen entre las distribuciones o que las consultas introduzcan un movimiento de gran cantidad de datos. Para investigar los problemas de rendimiento de consultas, consulte las [soluciones a los problemas de rendimiento](sql-data-warehouse-troubleshoot.md#performance).

## <a name="pausing-and-resuming-compute"></a>Pausa y reanudación del proceso

Al pausar el proceso, la capa de almacenamiento se desasocia de los nodos de proceso. Se liberan los recursos de proceso desde su cuenta. No se le cobra por el proceso mientras está en pausa. Al reanudarse el proceso, se vuelve a asociar el almacenamiento con los nodos de proceso y se reanudan los cargos correspondientes.
Al pausar un grupo de SQL dedicado (anteriormente SQL DW):

- Los recursos de memoria y proceso se devuelven al grupo de recursos disponibles en el centro de datos
- Los costos de unidad de almacenamiento de datos son cero lo que dura la pausa.
- El almacenamiento de datos no se ve afectado y sus datos permanecen intactos.
- Todas las operaciones en ejecución o en cola se cancelan.
- Se restablecen los contadores de DMV.

Al reanudar un grupo de SQL dedicado (anteriormente SQL DW):

- El grupo de SQL dedicado (anteriormente SQL DW) adquiere recursos de memoria y proceso equivalentes a su valor de unidades de almacenamiento de datos.
- Los cargos por la reanudación de las unidades de almacenamiento de datos se reanudan también.
- Los datos se vuelven disponibles.
- Una vez que el grupo de SQL dedicado (anteriormente SQL DW) está en línea, debe reiniciar las consultas de carga de trabajo.

Si quiere que el grupo de SQL dedicado (anteriormente SQL DW) esté siempre accesible, puede escalarlo verticalmente al tamaño más pequeño en lugar de pausarlo.

Para conocer los pasos de pausa y reanudación, consulte las guías de inicio rápido de [Azure Portal](pause-and-resume-compute-portal.md), o [PowerShell](pause-and-resume-compute-powershell.md). También puede usar la [REST API de pausa](sql-data-warehouse-manage-compute-rest-api.md#pause-compute) o la [REST API de reanudación](sql-data-warehouse-manage-compute-rest-api.md#resume-compute).

## <a name="drain-transactions-before-pausing-or-scaling"></a>Vaciado de las transacciones antes de aplicar la pausa o el escalado

Se recomienda permitir que finalicen las transacciones existentes antes de iniciar una operación de pausa o escalado.

Al pausar o escalar el grupo de SQL dedicado (anteriormente SQL DW), las consultas se cancelan en segundo plano al iniciar la solicitud de pausa o escalado. Una simple consulta de selección se cancela rápidamente y no afecta casi al tiempo que se tarda en aplicar la pausa o la escala a la instancia.  Sin embargo, las consultas sobre transacciones, que modifican los datos o la estructura de estos, no se puede detener rápidamente. **Por definición, las consultas sobre transacciones deben completarse íntegramente o deshacerse los cambios.** Deshacer el trabajo realizado con una consulta sobre una transacción puede tardar tanto (o más) que el tiempo necesario para aplicar el cambio original de esta. Por ejemplo, si se cancela una consulta que se eliminaba filas y ya lleva en ejecución una hora, el sistema puede tardar una hora para reinsertar las filas eliminadas. Si se ejecuta la pausa o la escala con transacciones en curso, puede parecer que tardan mucho en aplicarse, ya que deben esperar a que se deshagan todos los cambios para continuar.

Consulte también [Transacciones en SQL Data Warehouse](sql-data-warehouse-develop-transactions.md) y [Optimización de transacciones](sql-data-warehouse-develop-best-practices-transactions.md).

## <a name="automating-compute-management"></a>Automatización de la administración de proceso

Para automatizar las operaciones de administración de proceso, consulte [Administración de proceso con Azure Functions](manage-compute-with-azure-functions.md).

Cada una de las operaciones de escalado horizontal, pausar y reanudación puede tardar varios minutos en completarse. Si va a escalar, pausar o reanudar automáticamente, se recomienda implementar una lógica para garantizar que determinadas operaciones se hayan completado antes de continuar con otra acción. La comprobación del estado del grupo de SQL dedicado (anteriormente SQL DW) a través de varios puntos de conexión le permite implementar correctamente la automatización de tales operaciones.

Para comprobar el estado del grupo de SQL dedicado (anteriormente SQL DW), consulte el inicio rápido de [PowerShell](quickstart-scale-compute-powershell.md#check-data-warehouse-state) o [T-SQL](quickstart-scale-compute-tsql.md#check-dedicated-sql-pool-formerly-sql-dw-state). También puede comprobar el estado del grupo de SQL dedicado (anteriormente SQL DW) con una [API REST](sql-data-warehouse-manage-compute-rest-api.md#check-database-state).

## <a name="permissions"></a>Permisos

Para escalar el grupo de SQL dedicado (anteriormente SQL DW), se requieren los permisos descritos en [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Para pausar y reanudar, se requiere el permiso [Colaborador de base de datos SQL](../../role-based-access-control/built-in-roles.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#sql-db-contributor), en concreto Microsoft.Sql/servers/databases/action.

## <a name="next-steps"></a>Pasos siguientes

Consulte la guía de procedimientos de [administración del proceso](manage-compute-with-azure-functions.md) Otro aspecto de la administración de recursos de proceso es asignar distintos recursos a las consultas individuales. Para más información, consulte [Clases de recursos para la administración de cargas de trabajo](resource-classes-for-workload-management.md).
