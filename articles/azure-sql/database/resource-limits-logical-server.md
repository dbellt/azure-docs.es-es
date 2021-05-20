---
title: Límites de recursos para los servidores lógicos en Azure
description: En este artículo se proporciona información general sobre los límites de recursos para los servidores lógicos de Azure que usan Azure SQL Database y Azure Synapse Analytics. También se proporciona información acerca de lo qué ocurre cuando se alcanzan o superan dichos límites.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: ''
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: sashan,moslake,josack
ms.date: 03/25/2021
ms.openlocfilehash: 08b2d3ec205e22c85188a718a12b13aff04f311e
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108771710"
---
# <a name="resource-limits-for-azure-sql-database-and-azure-synapse-analytics-servers"></a>Límites de recursos para los servidores de Azure SQL Database y Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

En este artículo se proporciona información general sobre los límites de recursos para los [servidores lógicos](logical-servers.md) que usan Azure SQL Database y Azure Synapse Analytics. Proporciona información sobre lo que ocurre cuando esos límites de recursos se alcanzan o se superan y se describen los mecanismos de gobernanza de recursos usados para aplicar estos límites.

> [!NOTE]
> Para conocer los límites de Azure SQL Managed Instance, consulte [Límites de recursos para instancias administradas](../managed-instance/resource-limits.md).

## <a name="maximum-resource-limits"></a>Límites de recursos máximos

| Resource | Límite |
| :--- | :--- |
| Bases de datos por servidor lógico | 5000 |
| Número predeterminado de servidores lógicos por suscripción en una región | 20 |
| Número máximo de servidores lógicos por suscripción en una región | 200 |  
| Cuota de DTU o eDTU por servidor lógico | 54 000 |  
| Cuota de núcleo virtual por servidor lógico | 540 |
| Número máximo de grupos por servidor lógico | Limitado por el número de DTU o núcleos virtuales. Por ejemplo, si cada grupo tiene 1000 DTU, un servidor puede admitir 54 grupos.|
|||

> [!IMPORTANT]
> Cuando el número de bases de datos se aproxima al límite por servidor lógico, puede ocurrir lo siguiente:
>
> - Aumento de la latencia de las consultas en ejecución en la base de datos maestra.  Esto incluye las vistas de las estadísticas del uso de los recursos como `sys.resource_stats`.
> - Aumento de la latencia en las operaciones de administración y presentación de las perspectivas del portal que implican enumerar las bases de datos del servidor.

> [!NOTE]
> Para obtener más cuota de DTU o eDTU, cuota de núcleo virtual o más servidores lógicos que la cantidad predeterminada, envíe una nueva solicitud de soporte técnico en Azure Portal. Para más información, consulte [Solicitud de aumentos de cuota para Azure SQL Database](quota-increase-request.md).

### <a name="storage-size"></a>Tamaño de almacenamiento

Para información sobre los tamaños de almacenamiento de recursos de bases de datos únicas, consulte los [límites de recursos basados en DTU](resource-limits-dtu-single-databases.md) o los [límites de recursos basados en núcleo virtual](resource-limits-vcore-single-databases.md) para conocer los límites de tamaño de almacenamiento por plan de tarifa.

## <a name="what-happens-when-database-resource-limits-are-reached"></a>¿Qué ocurre cuando se alcanzan los límites de recursos de base de datos?

### <a name="compute-cpu"></a>CPU de proceso

Cuando la utilización de CPU de proceso de base de datos es elevada, la latencia de las consultas aumenta y puede agotar el tiempo de espera. En estas condiciones, el servicio puede poner en cola las consultas y se proporcionan recursos para la ejecución a medida que el recurso esté disponible.
Al encontrar un uso de proceso elevado, las opciones de mitigación incluyen:

- Aumentar el tamaño de proceso de la base de datos o del grupo elástico para proporcionar a la base de datos más recursos de proceso. Consulte [Scale single database resources](single-database-scale.md) (Escala de recursos de bases de datos únicas) y [Scale elastic pool resources](elastic-pool-scale.md) (Escala de recursos de grupos elásticos).
- Optimice las consultas para reducir el uso de recursos de CPU de cada consulta. Para más información, consulte [Optimización y sugerencias de consultas](performance-guidance.md#query-tuning-and-hinting).

### <a name="storage"></a>Storage

Cuando el espacio de la base de datos alcanza el límite de tamaño máximo, las inserciones y actualizaciones de la base de datos que aumentan el tamaño de los datos producen un error y los clientes reciben un [mensaje de error](troubleshoot-common-errors-issues.md). Las instrucciones SELECT y DELETE se siguen ejecutando correctamente.

Al encontrar un uso elevado de espacio, las opciones de mitigación incluyen:

- Aumentar el tamaño máximo de la base de datos o del grupo elástico, o agregar más almacenamiento. Consulte [Scale single database resources](single-database-scale.md) (Escala de recursos de bases de datos únicas) y [Scale elastic pool resources](elastic-pool-scale.md) (Escala de recursos de grupos elásticos).
- Si la base de datos está en un grupo elástico, se puede mover fuera del grupo para que su espacio de almacenamiento no se comparta con otras bases de datos.
- Reduzca una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).
- Compruebe si el uso elevado de espacio se debe a un pico en el tamaño del almacén de versiones persistentes (PVS). PVS es una parte de cada base de datos y se utiliza para implementar la [recuperación acelerada de la base de datos](../accelerated-database-recovery.md). Para determinar el tamaño actual del PVS, consulte [Solución de problemas de PVS](/sql/relational-databases/accelerated-database-recovery-management#troubleshooting). Un motivo común para un tamaño grande de PVS es una transacción abierta durante mucho tiempo (horas), lo que impide la limpieza de versiones anteriores en PVS.

### <a name="sessions-and-workers-requests"></a>Sesiones y trabajos (solicitudes)

El número máximo de sesiones y trabajos se determina por el nivel de servicio y el tamaño de proceso (DTU y eDTU o núcleos virtuales). Las nuevas solicitudes se rechazan cuando se alcanzan los límites de sesión o de trabajo y los clientes reciben un mensaje de error. Aunque el número de conexiones disponibles se pueden controlar por la aplicación, el número de trabajos simultáneos suele ser más difícil de calcular y controlar. Esto es especialmente cierto durante los períodos de carga máxima, cuando se alcanzan los límites de recursos de base de datos y los trabajos se apilan debido a consultas de larga duración, cadenas de bloqueo de gran tamaño o excesivo paralelismo de las consultas.

Al encontrar un uso elevado de sesión o de trabajo, las opciones de mitigación incluyen:

- Aumentar el nivel de servicio o el tamaño de proceso de la base de datos o del grupo elástico. Consulte [Scale single database resources](single-database-scale.md) (Escala de recursos de bases de datos únicas) y [Scale elastic pool resources](elastic-pool-scale.md) (Escala de recursos de grupos elásticos).
- Optimizar las consultas para reducir el uso de recursos de cada consulta si la causa del mayor uso de trabajo es debida a la contención de los recursos de proceso. Para más información, consulte [Optimización y sugerencias de consultas](performance-guidance.md#query-tuning-and-hinting).
- Reducir el valor de [MAXDOP](/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option#Guidelines) (grado máximo de paralelismo).
- Optimizar la carga de trabajo de consultas para reducir el número de repeticiones y la duración del bloqueo de consultas. Para obtener más información, consulte [Descripción y resolución de problemas de bloqueo en Azure SQL](understand-resolve-blocking.md).

### <a name="memory"></a>Memoria

A diferencia de otros recursos (CPU, nodos de trabajo, almacenamiento), el hecho de alcanzar el límite de memoria no afecta negativamente al rendimiento de las consultas y no provoca errores. Como se describe con detalle en [Guía de la arquitectura de administración de memoria](/sql/relational-databases/memory-management-architecture-guide), el motor de base de datos de SQL Server usa a menudo toda la memoria disponible, por diseño. La memoria se usa principalmente para el almacenamiento en caché de los datos, para evitar un acceso más costoso al almacenamiento. Por lo tanto, un mayor uso de memoria suele mejorar el rendimiento de las consultas debido a lecturas más rápidas de la memoria, en lugar de lecturas más lentas del almacenamiento.

Después del inicio del motor de base de datos, a medida que la carga de trabajo comienza a leer datos del almacenamiento, el motor de base de datos almacena en memoria caché los datos de forma agresiva. Después de este período inicial de crecimiento, es habitual y se espera ver que las columnas `avg_memory_usage_percent` y `avg_instance_memory_percent` de [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) estén cerca del 100 %, especialmente en el caso de las bases de datos que no están inactivas y no caben completas en la memoria.

Además de la memoria caché de datos, la memoria se utiliza en otros componentes del motor de base de datos. Cuando hay demanda de memoria y el almacenamiento en caché de los datos ha utilizado toda la memoria disponible, el motor de base de datos reduce dinámicamente el tamaño de la memoria caché de datos para poner la memoria a disposición de otros componentes y la memoria caché de datos crecerá dinámicamente cuando otros componentes liberen memoria.

En raras ocasiones, una carga de trabajo suficientemente exigente puede producir una condición de memoria insuficiente, lo que provoca errores de memoria insuficiente. Esto puede ocurrir en cualquier nivel de uso de memoria entre el 0 % y el 100 %. Es más probable que se produzca en tamaños de proceso más pequeños que tienen límites de memoria proporcionalmente menores o con cargas de trabajo que usan más memoria para el procesamiento de consultas, como en los [grupos elásticos densos](elastic-pool-resource-management.md).

Cuando aparecen errores de memoria insuficiente, las opciones de mitigación incluyen:
- Aumentar el nivel de servicio o el tamaño de proceso de la base de datos o del grupo elástico. Consulte [Scale single database resources](single-database-scale.md) (Escala de recursos de bases de datos únicas) y [Scale elastic pool resources](elastic-pool-scale.md) (Escala de recursos de grupos elásticos).
- Optimizar las consultas y la configuración para reducir el uso de memoria. Las soluciones más comunes se describen en la tabla siguiente.

|Solución|Descripción|
| :----- | :----- |
|Reducir el tamaño de las concesiones de memoria|Para más información sobre las concesiones de memoria, consulte la entrada de blog [Descripción de la concesión de memoria de SQL Server](https://techcommunity.microsoft.com/t5/sql-server/understanding-sql-server-memory-grant/ba-p/383595). Una solución común para evitar concesiones de memoria excesivamente grandes es mantener las [estadísticas](/sql/relational-databases/statistics/statistics) actualizadas. Esto da como resultado una estimación más precisa del consumo de memoria por parte del motor de consultas y se evitan concesiones de memoria innecesariamente grandes.</br></br>En las bases de datos que usan el nivel de compatibilidad 140 y posteriores, el motor de base de datos puede ajustar automáticamente el tamaño de la concesión de memoria mediante [Comentarios de concesión de memoria del modo de proceso por lotes](/sql/relational-databases/performance/intelligent-query-processing#batch-mode-memory-grant-feedback). En las bases de datos que usan el nivel de compatibilidad 150 y posteriores, el motor de base de datos utiliza de forma similar [Comentarios de concesión de memoria del modo de fila](/sql/relational-databases/performance/intelligent-query-processing#row-mode-memory-grant-feedback) para las consultas del modo de fila más comunes. Esta funcionalidad integrada ayuda a evitar errores de memoria insuficiente debidos a concesiones de memoria innecesariamente grandes.|
|Reducir el tamaño de la memoria caché del plan de consulta|El motor de base de datos almacena en memoria caché los planes de consulta para evitar la compilación de un plan de consulta para cada ejecución de la consulta. Para evitar la saturación de la memoria caché del plan de consulta causada por los planes de almacenamiento en caché que solo se usan una vez, habilite la [configuración con ámbito de base de datos](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) OPTIMIZE_FOR_AD_HOC_WORKLOADS.|
|Reducir el tamaño de la memoria de bloqueos|El motor de base de datos utiliza memoria para los [bloqueos](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#Lock_Engine). Cuando sea posible, evite transacciones grandes que puedan adquirir un gran número de bloqueos y provoquen un alto consumo de la memoria de bloqueos.|


## <a name="resource-consumption-by-user-workloads-and-internal-processes"></a>Consumo de recursos por cargas de trabajo de usuario y procesos internos

El consumo de CPU y memoria de las cargas de trabajo de usuario en cada base de datos se indica en las vistas [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) y [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database), en las columnas `avg_cpu_percent` y `avg_memory_usage_percent`. En el caso de los grupos elásticos, el consumo de recursos de nivel de grupo se indica en la vista [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database). El consumo de CPU de la carga de trabajo de usuario también se indica a través de la métrica de Azure Monitor `cpu_percent`, para [bases de datos únicas](../../azure-monitor/essentials/metrics-supported.md#microsoftsqlserversdatabases) y [grupos elásticos](../../azure-monitor/essentials/metrics-supported.md#microsoftsqlserverselasticpools) en el nivel de grupo.

Azure SQL Database requiere recursos de proceso para implementar características de servicio principales, como alta disponibilidad y recuperación ante desastres, copia de seguridad y restauración de bases de datos, supervisión, Almacén de consultas, ajuste automático, etc. El sistema reserva una parte limitada de los recursos generales para estos procesos internos mediante mecanismos de [Regulación de recursos](#resource-governance), lo que permite que el resto de los recursos estén disponibles para las cargas de trabajo de usuario. A veces, cuando los procesos internos no usan recursos de proceso, el sistema los pone a disposición de las cargas de trabajo de usuario.

El consumo total de CPU y memoria de las cargas de trabajo de usuario y de los procesos internos se indica en las vistas [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) y [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database), en las columnas `avg_instance_cpu_percent` y `avg_instance_memory_percent`. Estos datos también se indican a través de las métricas de Azure Monitor `sqlserver_process_core_percent` y `sqlserver_process_memory_percent` para [bases de datos únicas](../../azure-monitor/essentials/metrics-supported.md#microsoftsqlserversdatabases) y [grupos elásticos](../../azure-monitor/essentials/metrics-supported.md#microsoftsqlserverselasticpools) en el nivel de grupo.

En las vistas [sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database) y [sys.dm_resource_governor_workload_groups_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-history-ex-azure-sql-database), se muestra un desglose más detallado del consumo reciente de recursos por parte de cargas de trabajo de usuario y procesos internos. Para obtener más información sobre los grupos de recursos y los grupos de cargas de trabajo a los que se hace referencia en estas vistas, consulte [Regulación de recursos](#resource-governance). Estas vistas informan sobre el uso de recursos por parte de cargas de trabajo de usuario y procesos internos específicos en los grupos de recursos y grupos de cargas de trabajo asociados.

En el contexto de supervisión y solución de problemas de rendimiento, es importante tener en cuenta el **consumo de CPU del usuario** (`avg_cpu_percent`, `cpu_percent`) y el **consumo de CPU total** de las cargas de trabajo de usuario y los procesos internos (`avg_instance_cpu_percent`,`sqlserver_process_core_percent`).

El **consumo de CPU del usuario** se calcula como un porcentaje de los límites de carga de trabajo de usuario en cada objetivo de servicio. Un **uso de CPU del usuario** del 100 % indica que la carga de trabajo de usuario ha alcanzado el límite del objetivo del servicio. Sin embargo, cuando el **consumo de CPU total** alcanza el intervalo del 70-100 %, es posible ver una reducción del rendimiento de la carga de trabajo de usuario y un aumento de la latencia de las consultas, aunque se indique un **consumo de CPU del usuario** bastante por debajo del 100 %. Esto es más probable que suceda si se usan objetivos de servicio menores con una asignación moderada de recursos de proceso, pero cargas de trabajo de usuario relativamente intensas, como en [grupos elásticos densos](elastic-pool-resource-management.md). Esto también puede suceder con objetivos de servicio menores si los procesos internos requieren temporalmente recursos adicionales, como, por ejemplo, al crear una nueva réplica de la base de datos.

Si el **consumo de CPU total** es alto, las opciones de mitigación son las mismas que las indicadas anteriormente, e incluyen el aumento del objetivo de servicio y/o la optimización de la carga de trabajo de usuario.

## <a name="resource-governance"></a>Regulación de recursos

Para aplicar límites de recursos, Azure SQL Database usa una implementación de regulación de recursos basada en [Resource Governor](/sql/relational-databases/resource-governor/resource-governor) de SQL Server, modificada y extendida para ejecutarse en Azure SQL Database. En SQL Database, hay varios [grupos de recursos](/sql/relational-databases/resource-governor/resource-governor-resource-pool) y [grupos de cargas de trabajo](/sql/relational-databases/resource-governor/resource-governor-workload-group) con límites de recursos que se establecen en el nivel de grupo para proporcionar una [base de datos como servicio equilibrada](https://azure.microsoft.com/blog/resource-governance-in-azure-sql-database/). La carga de trabajo de usuario y las cargas de trabajo internas se clasifican en grupos de recursos y grupos de cargas de trabajo distintos. La carga de trabajo de usuario de las réplicas primarias y secundarias legibles, incluidas las réplicas geográficas, se clasifica en el grupo de recursos `SloSharedPool1` y en el grupo de cargas de trabajo `UserPrimaryGroup.DBId[N]`, donde `N` representa el valor del identificador de base de datos. Además, hay varios grupos de recursos y grupos de cargas de trabajo para varias cargas de trabajo internas.

Además de usar Resource Governor para regular los recursos dentro del proceso de SQL, Azure SQL Database también emplea [objetos de trabajo](/windows/win32/procthread/job-objects) de Windows para la regulación de los recursos de nivel de proceso, y el [Administrador de recursos del servidor de archivos (FSRM)](/windows-server/storage/fsrm/fsrm-overview) de Windows para la administración de la cuota de almacenamiento.

La regulación de recursos de Azure SQL Database es jerárquica por naturaleza. De arriba a abajo, los límites se aplican en el nivel de sistema operativo y en el nivel de volumen de almacenamiento mediante mecanismos de regulación de recursos del sistema operativo y Resource Governor; en el nivel de grupo de recursos mediante Resource Governor; y en el nivel de grupo de cargas de trabajo mediante Resource Governor. Los límites de regulación de recursos en vigor para la base de datos o el grupo elástico actuales se exponen en la vista [sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database).

### <a name="data-io-governance"></a>Regulación de E/S de los datos

La regulación de E/S de los datos es un proceso de Azure SQL Database que se usa para limitar la E/S física de lectura y escritura en los archivos de datos de una base de datos. Para cada nivel de servicio se establecen límites de IOPS con el fin de minimizar el efecto "vecino ruidoso", para proporcionar igualdad en la asignación de recursos en el servicio multiinquilino y para permanecer dentro de las capacidades del hardware y el almacenamiento subyacentes.

En el caso de bases de datos únicas, los límites del grupo de cargas de trabajo se aplican a toda la E/S de almacenamiento de la base de datos, mientras que los límites del grupo de recursos se aplican a toda la E/S de almacenamiento de todas las bases de datos del mismo grupo de recursos, incluida la base de datos `tempdb`. En el caso de grupos elásticos, los límites del grupo de cargas de trabajo se aplican a cada base de datos del grupo, mientras que el límite del grupos de recursos se aplica a todo el grupo elástico, incluida la base de datos `tempdb`, que se comparte entre todas las bases de datos del grupo. En general, es posible que la carga de trabajo no pueda lograr los límites del grupo de recursos de una base de datos (sola o agrupada), porque los límites del grupo de cargas de trabajo son inferiores a los límites del grupo de recursos y limitan antes el número de IOPS y el rendimiento. Aunque la carga de trabajo combinada puede alcanzar los límites del grupo en varias bases de datos del mismo grupo.

Por ejemplo, si una consulta genera 1000 IOPS sin gobernanza de recursos de E/S, pero el límite máximo de IOPS del grupo de cargas de trabajo se establece en 900 IOPS, la consulta no podrá generar más de 900 IOPS. No obstante, si el límite máximo de IOPS del grupo de recursos se establece en 1500 IOPS y la E/S total de todos los grupos de cargas de trabajo asociados con el grupo de recursos supera los 1500 IOPS, la E/S de la misma consulta puede reducirse por debajo del límite del grupo de trabajo de 900 IOPS.

El número de IOPS y los valores mínimo y máximo de rendimiento que devuelve la vista [sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database) funcionan como límites, no como garantías. Además, la gobernanza de recursos no garantiza ninguna latencia de almacenamiento específica. Los mejores logros en cuanto a latencia, IOPS y rendimiento para una carga de trabajo de usuario determinada no dependen solo de los límites de regulación de los recursos, sino también de la mezcla de tamaños de E/S usados y de las capacidades del almacenamiento subyacente. SQL Database usa E/S con un tamaño variable de entre 512 KB y 4 MB. Con el fin de aplicar los límites de IOPS, cada E/S se cuenta con independencia de su tamaño, a excepción de las bases de datos con archivos de datos de Azure Storage. En ese caso, las E/S de más de 256 KB se cuentan como varias E/S de 256 KB, para alinearse con el recuento de E/S de Azure Storage.

En las bases de datos Básica, Estándar y De uso general, que usan archivos de datos de Azure Storage, es posible que el valor de `primary_group_max_io` no sea factible si una base de datos no tiene suficientes archivos de datos para proporcionar de forma acumulativa este número de IOPS, o si los datos no se distribuyen uniformemente entre los archivos o si el nivel de rendimiento de los blobs subyacentes limita el número de IOPS o el rendimiento por debajo del límite de gobernanza de los recursos. Igualmente, con E/S de registros pequeños generadas por la confirmación frecuente de transacciones, puede que el valor `primary_max_log_rate` no sea factible para una carga de trabajo debido al límite de IOPS en el blob de Azure Storage subyacente. En el caso de las bases de datos que usan Azure Premium Storage, Azure SQL Database usa blobs de almacenamiento lo suficientemente grandes como para obtener el número de IOPS o el rendimiento necesarios, con independencia del tamaño de la base de datos. En el caso de bases de datos más grandes, se crean varios archivos de datos para aumentar la capacidad total de IOPS y rendimiento.

Los valores de uso de recursos, como `avg_data_io_percent` y `avg_log_write_percent`, notificados en las vistas [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database),  [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)y [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database), se calculan como porcentajes de los límites máximos de regulación de recursos. Por lo tanto, cuando otros factores que no son la gobernanza de recursos limitan el número de IOPS o el rendimiento, es posible que se reduzcan estos y que aumenten las latencias a medida que se incrementa la carga de trabajo, incluso aunque la utilización de los recursos notificada permanezca por debajo del 100 %.

Para ver el número de IOPS, el rendimiento y la latencia de lectura y escritura por archivo de base de datos, utilice la función [sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql). Esta función muestra toda la E/S en la base de datos, incluida la que tiene lugar en segundo plano que no se tiene en cuenta en `avg_data_io_percent`, pero usa el número de IOPS y el rendimiento del almacenamiento subyacente y puede afectar a la latencia de almacenamiento observada. La función también muestra una latencia adicional, en las columnas `io_stall_queued_read_ms` y `io_stall_queued_write_ms`, respectivamente, que podría deberse a la gobernanza de los recursos de E/S de lecturas y escrituras.

### <a name="transaction-log-rate-governance"></a>Regulación de la tasa de registro de transacciones

La gobernanza de las tasas de registros de transacciones es un proceso en Azure SQL Database que se usa para limitar las altas tasas de ingesta de cargas de trabajo como la inserción masiva, SELECT INTO y compilaciones de índice. Se realiza un seguimiento de estos límites y se aplican en el nivel de fracciones de segundo a la tasa de generación de registros, lo que limita el rendimiento sin importar cuántas E/S se pueden emitir respecto de los archivos de datos.  Las velocidades de generación de registros de transacciones se escalan linealmente hasta un punto que depende del hardware, con la tasa de registro máxima permitida de 96 MB/s con el modelo de compra de núcleo virtual.

> [!NOTE]
> No se controlan ni limitan las E/S físicas reales para los archivos de registro de transacciones.

Las tasas de registro se establecen de manera que se puedan alcanzar y sostener en diversos escenarios, mientras que el sistema global puede mantener su funcionalidad con un mínimo de impacto en la carga del usuario. La gobernanza de las tasas de registro garantiza que las copias de seguridad del registro de transacciones se mantendrán dentro de los contratos de nivel de servicio de la capacidad de recuperación publicados.  Esta gobernanza también evita un trabajo pendiente excesivo en las réplicas secundarias.

A medida que se generan los registros, se evalúa cada operación para ver si se debe retrasar a fin de mantener una tasa máxima de registro deseada (MB/s por segundo). No se agregan retrasos cuando los registros se vacían en el almacenamiento. En lugar de eso, la gobernanza de tasas de registro se aplica durante la propia generación de registros.

Las tasas reales de generación de registros impuestas en tiempo de ejecución también podrían verse influenciadas por los mecanismos de comentarios, lo que reducirá de manera temporal las tasas de registros permitidas para que el sistema se pueda estabilizar. La administración del espacio de los archivos de registro, para evitar la falta de espacio para registros, y los mecanismos de replicación del grupo de disponibilidad pueden disminuir temporalmente los límites del sistema global.

El modelado de tráfico del regulador de las tasas de registros se expone a través de los tipos de espera siguiente (expuesto en las vistas [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) y [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql)):

| Tipo de espera | Notas |
| :--- | :--- |
| LOG_RATE_GOVERNOR | Limitación de la base de datos |
| POOL_LOG_RATE_GOVERNOR | Limitación del grupo |
| INSTANCE_LOG_RATE_GOVERNOR | Limitación del nivel de instancia |  
| HADR_THROTTLE_LOG_RATE_SEND_RECV_QUEUE_SIZE | Control de comentarios, la replicación física del grupo de disponibilidad en el nivel de servicio Crítico para la empresa/Premium no está al día |  
| HADR_THROTTLE_LOG_RATE_LOG_SIZE | Control de comentarios, limitación de las tasas para evitar quedarse sin espacio para registros |
| HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO | Control de comentarios de replicación geográfica, limitación de la velocidad de registro para evitar una alta latencia de los datos y la no disponibilidad de replicaciones geográficas secundarias|
|||

Cuando encuentre un límite para las tasas de registros que dificulte la alcanzar la escalabilidad deseada, considere estas opciones:

- Escale verticalmente hasta un nivel de servicio superior para obtener la velocidad máxima de registro de 96 MB/s, o cambie a un nivel de servicio diferente. El nivel de servicio [Hiperescala](service-tier-hyperscale.md) proporciona una velocidad de registro de 100 MB/s, independientemente del nivel de servicio elegido.
- Si los datos que se cargan son transitorios, como los datos de ensayo de un proceso de ETL, se pueden cargar en tempdb (que genera un mínimo de registros).
- En los escenarios de análisis, cárguelos en una tabla cubierta de almacén de columnas en clúster. Esto disminuye la tasa de registros necesario debido a la compresión. Esta técnica sí aumenta la utilización de la CPU y solo se aplica a los conjuntos de datos que se benefician de los índices de almacén de columnas en clúster.

### <a name="storage-space-governance"></a>Gobernanza del espacio de almacenamiento

En los niveles de servicio Premium y Crítico para la empresa, los archivos de datos y de registro de transacciones se almacenan en el volumen de SSD local de la máquina que hospeda la base de datos o el grupo elástico. El resultado es un número elevado IOPS y un alto rendimiento, así como una baja latencia de E/S. El tamaño de este volumen local depende de las capacidades de hardware y es finito. En una máquina determinada, las bases de datos del cliente, como `tempdb`, el sistema operativo, el software de administración, los datos de supervisión, los registros, etc., consumen espacio de volumen local. A medida que se crean o eliminan las bases de datos y aumentan o reducen su uso del espacio, el consumo de espacio local en una máquina fluctúa con el tiempo. 

Si el sistema detecta que el espacio libre disponible en una máquina es bajo y una base de datos o un grupo elástico corren el riesgo de quedarse sin espacio, moverá la base de datos o el grupo elástico a otra máquina con suficiente espacio libre, lo que permite que crezcan hasta alcanzar los límites máximos del objetivo de servicio configurado. Este movimiento se produce en línea, de forma similar a una operación de escalado de bases de datos, y tiene un [efecto](single-database-scale.md#impact) parecido, por ejemplo, se produce una conmutación por error de corta duración (segundos) al final de la operación. Esta conmutación por error finaliza las conexiones abiertas y revierte las transacciones, lo que puede afectar a las aplicaciones que usan la base de datos en ese momento.

Dado que los datos se copian físicamente en una máquina diferente, mover bases de datos de mayor tamaño puede requerir una cantidad considerable de tiempo. Durante ese tiempo, si el consumo de espacio local por una base de datos de usuarios o un grupo elástico grandes, o por la base de datos `tempdb`, crece con mucha rapidez, el riesgo de quedarse sin espacio aumenta. El sistema inicia el movimiento de la base de datos de manera equilibrada para evitar errores de espacio insuficiente y conmutaciones por error innecesarias.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los límites generales de Azure, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Para más información sobre las DTU y eDTU, consulte [DTU y eDTU](purchasing-models.md#dtu-based-purchasing-model).
- Para más información sobre los límites de tamaño de tempdb, consulte [Base de datos tempdb en Azure SQL Database](/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database).
