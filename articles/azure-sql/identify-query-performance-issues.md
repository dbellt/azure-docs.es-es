---
title: Tipos de problemas de rendimiento de las consultas en Azure SQL Database
description: En este artículo, obtendrá información sobre los distintos tipos de problemas de rendimiento de las consultas en Azure SQL Database y aprenderá a identificar y resolver consultas con estos problemas
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: troubleshooting
author: NikaKinska
ms.author: nnikolic
ms.reviewer: mathoma, wiassaf
ms.date: 1/14/2021
ms.openlocfilehash: e4120b151957d8bb7360d7313000269e1ef51b03
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111592347"
---
# <a name="detectable-types-of-query-performance-bottlenecks-in-azure-sql-database"></a>Tipos de cuellos de botella en el rendimiento de las consultas que se pueden detectar en Azure SQL Database
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Cuando intente resolver un cuello de botella de rendimiento, comience por determinar si el cuello de botella se produce cuando la consulta está en estado de ejecución o en estado de espera. Se aplican distintas resoluciones en función de esta determinación. El diagrama siguiente le ayuda a conocer los factores que pueden provocar un problema relacionado con la ejecución o un problema relacionado con la espera. En este artículo se describen los problemas y las soluciones en relación a cada tipo de problema.

Puede usar [Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#detectable-database-performance-patterns) de Azure SQL Database o [DMV](database/monitoring-with-dmvs.md) de SQL Server para detectar estos tipos de cuellos de botella de rendimiento.

![Estados de carga de trabajo](./media/identify-query-performance-issues/workload-states.png)

**Problemas relacionados con el rendimiento**: los problemas relacionados con la ejecución normalmente están relacionados con problemas de compilación, lo que produce un plan de consulta que no es óptimo o problemas de ejecución relacionados con un número insuficiente de recursos o una sobreutilización de estos.
**Problemas relacionados con la espera**: los problemas relacionados con la espera suelen estar relacionados con:

- Bloqueos
- E/S
- Contención relacionada con el uso de TempDB
- Esperas de concesión de memoria

## <a name="compilation-problems-resulting-in-a-suboptimal-query-plan"></a>Problemas de compilación que dan como resultado un plan de consulta no óptimo

Un plan no óptimo generado por SQL Query Optimizer puede ser la causa de un rendimiento lento de las consultas. SQL Query Optimizer puede producir un plan que no sea óptimo porque falta un índice, las estadísticas están obsoletas o se ha realizado una estimación incorrecta del número de filas que se van a procesar o de la memoria necesaria. Si se sabe que la consulta se ha ejecutado con más rapidez en el pasado o en otra instancia, compare los planes de ejecución reales para ver si son diferentes.

- Para identificar los índices que faltan, use uno de estos métodos:

  - Use [Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#missing-index).
  - [Database Advisor](database/database-advisor-implement-performance-recommendations.md) para bases de datos únicas y agrupadas.
  - DMV. En este ejemplo se muestra el impacto de un índice que falta, cómo detectar [índices que faltan](database/performance-guidance.md#identifying-and-adding-missing-indexes) mediante DMV y el impacto de implementar la recomendación del índice que falta.
- Intente aplicar [sugerencias de consulta](/sql/t-sql/queries/hints-transact-sql-query), [actualización de estadísticas](/sql/t-sql/statements/update-statistics-transact-sql) o [recompilación de índices](/sql/relational-databases/indexes/reorganize-and-rebuild-indexes) para obtener un plan mejor. Habilite la [corrección automática de los planes](../azure-sql/database/automatic-tuning-overview.md) en Azure SQL Database para mitigar estos problemas de forma automática.

  Este [ejemplo](database/performance-guidance.md#query-tuning-and-hinting) muestra el impacto de un plan de consulta no óptimo debido a una consulta con parámetros, cómo detectar esta condición y cómo usar una sugerencia de consulta para resolverla.

- Intente cambiar el nivel de compatibilidad de la base de datos e implementar el procesamiento de consultas inteligentes. SQL Query Optimizer puede generar un plan de consulta diferente en función del nivel de compatibilidad de la base de datos. Los niveles de compatibilidad más altos proporcionan más [funcionalidades de procesamiento de consultas inteligentes](/sql/relational-databases/performance/intelligent-query-processing).

  - Para más información sobre el procesamiento de consultas, consulte [Guía de arquitectura de procesamiento de consultas](/sql/relational-databases/query-processing-architecture-guide).
  - Para cambiar los niveles de compatibilidad de las bases de datos y obtener más información sobre las diferencias entre los distintos niveles de compatibilidad, consulte [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level).
  - Para más información sobre la estimación de la cardinalidad, consulte [Estimación de cardinalidad](/sql/relational-databases/performance/cardinality-estimation-sql-server)

## <a name="resolving-queries-with-suboptimal-query-execution-plans"></a>Resolución de consultas con planes de ejecución de consultas no óptimos

En las secciones siguientes se describe cómo resolver consultas con un plan de ejecución de consultas no óptimo.

### <a name="queries-that-have-parameter-sensitive-plan-psp-problems"></a><a name="ParamSniffing"></a> Consultas que tienen problemas con el plan de distinción de parámetros (PSP)

El problema del plan que distingue entre parámetros (PSP) sucede cuando el optimizador de consultas genera un plan de ejecución de consultas que es óptimo solo para un valor de parámetro específico (o un conjunto de valores) y el plan almacenado en caché no es óptimo para los valores de parámetros que se usan en ejecuciones consecutivas. Los planes que no son óptimos pueden provocar problemas de rendimiento de consultas y reducir el rendimiento general de la carga de trabajo.

Para más información sobre el procesamiento de consultas y el examen de parámetros, consulte la [Guía de arquitectura de procesamiento de consultas](/sql/relational-databases/query-processing-architecture-guide#ParamSniffing).

Varias soluciones alternativas pueden mitigar los problemas de PSP. Cada solución lleva asociada una serie de ventajas e inconvenientes:

- Use la sugerencia de consulta [RECOMPILE](/sql/t-sql/queries/hints-transact-sql-query) en cada ejecución de consulta. Esta solución alternativa tiene en cuenta el tiempo de compilación y el aumento del uso de la CPU para obtener mejor calidad en el plan. La opción `RECOMPILE` no es a menudo posible con cargas de trabajo que requieren un alto rendimiento.
- Use la sugerencia de consulta [OPTION (OPTIMIZE FOR…)](/sql/t-sql/queries/hints-transact-sql-query) para invalidar el valor del parámetro real con un valor de parámetro típico que cree un plan lo suficientemente bueno para la mayoría de las posibilidades que ofrece el valor de parámetro. Esta opción requiere tener una buena comprensión de los valores óptimos de los parámetros y de las características del plan asociado.
- Use la sugerencia de consulta [OPTION (OPTIMIZE FOR UNKNOWN)](/sql/t-sql/queries/hints-transact-sql-query) para invalidar el valor del parámetro real y así poder usar la media del vector de densidad. Otra forma de hacerlo es capturar los valores de los parámetros entrantes en variables locales y luego usar esas variables locales en los predicados en lugar de utilizar los parámetros en sí. En el caso de esta corrección, la densidad media debe ser *suficientemente buena*.
- Desactive el examen de parámetros por completo mediante la sugerencia de consulta [DISABLE_PARAMETER_SNIFFING](/sql/t-sql/queries/hints-transact-sql-query).
- Use la sugerencia de consulta [KEEPFIXEDPLAN](/sql/t-sql/queries/hints-transact-sql-query) para evitar recompilaciones en la caché. Esta solución asume que el plan común suficientemente bueno es el que ya está en la caché. También puede deshabilitar las actualizaciones automáticas de las estadísticas para reducir las posibilidades de que se expulse el plan bueno y se compile un nuevo plan malo.
- Para forzar el plan explícitamente, use la sugerencia de consulta [USE PLAN](/sql/t-sql/queries/hints-transact-sql-query), vuelva a escribir la consulta y agregue la sugerencia al texto de consulta. O bien, establezca un plan específico mediante el Almacén de consultas o la habilitación del [ajuste automático](../azure-sql/database/automatic-tuning-overview.md).
- Reemplace el procedimiento único con un conjunto anidado de procedimientos que se pueden usar según la lógica condicional y los valores de los parámetros asociados.
- Cree alternativas de ejecución de cadenas dinámicas en una definición de procedimiento estático.

Para más información sobre cómo resolver los problemas de PSP, consulte estas entradas de blog:

- [Huelo un parámetro](/archive/blogs/queryoptteam/i-smell-a-parameter)
- [Conor vs. dynamic SQL vs. procedures vs. plan quality for parameterized queries](/archive/blogs/conor_cunningham_msft/conor-vs-dynamic-sql-vs-procedures-vs-plan-quality-for-parameterized-queries) (Conor frente a SKL dinámico, procedimientos y calidad del plan de las consultas parametrizadas)
- [Técnicas de optimización de consultas de SQL en SQL Server: Examen de parámetros](https://www.sqlshack.com/query-optimization-techniques-in-sql-server-parameter-sniffing/)

### <a name="compile-activity-caused-by-improper-parameterization"></a>Actividad de compilación causada por una parametrización incorrecta

Cuando una consulta tiene literales, el motor de base de datos parametriza automáticamente la instrucción o un usuario la parametriza explícitamente para reducir el número de compilaciones. Un alto número de compilaciones de una consulta que usa el mismo patrón pero diferentes valores literales puede dar lugar a un uso elevado de la CPU. De manera similar, si solo parametriza parcialmente una consulta que sigue teniendo literales, el motor de base de datos no la parametriza más.

Este es un ejemplo de una consulta parcialmente parametrizada:

```sql
SELECT *
FROM t1 JOIN t2 ON t1.c1 = t2.c1
WHERE t1.c1 = @p1 AND t2.c2 = '961C3970-0E54-4E8E-82B6-5545BE897F8F';
```

En el ejemplo anterior, `t1.c1` toma `@p1`, pero `t2.c2` continúa tomando GUID como elemento literal. En este caso, si cambia el valor de `c2`, la consulta se trata como una consulta diferente y se produce una nueva compilación. Para reducir las compilaciones en este ejemplo, también parametrizaría el GUID.

La siguiente consulta muestra el número de consultas en función del hash de consulta para determinar si una consulta está correctamente parametrizada:

```sql
SELECT TOP 10
  q.query_hash
  , count (distinct p.query_id ) AS number_of_distinct_query_ids
  , min(qt.query_sql_text) AS sampled_query_text
FROM sys.query_store_query_text AS qt
  JOIN sys.query_store_query AS q
     ON qt.query_text_id = q.query_text_id
  JOIN sys.query_store_plan AS p
     ON q.query_id = p.query_id
  JOIN sys.query_store_runtime_stats AS rs
     ON rs.plan_id = p.plan_id
  JOIN sys.query_store_runtime_stats_interval AS rsi
     ON rsi.runtime_stats_interval_id = rs.runtime_stats_interval_id
WHERE
  rsi.start_time >= DATEADD(hour, -2, GETUTCDATE())
  AND query_parameterization_type_desc IN ('User', 'None')
GROUP BY q.query_hash
ORDER BY count (distinct p.query_id) DESC;
```

### <a name="factors-that-affect-query-plan-changes"></a>Factores que afectan a los cambios del plan de consulta

Una recompilación del plan de ejecución de consultas puede producir un plan de consulta generado que difiera del que inicialmente se almacenó en caché. Hay varias razones por las que un plan original existente se puede recompilar automáticamente:

- La consulta hace referencia a los cambios en el esquema
- La consulta hace referencia a los cambios de datos en las tablas
- Se han cambiado opciones del contexto de la consulta

Es posible que un plan compilado se expulse de la caché por diversos motivos, por ejemplo:

- La instancia se reinicia
- La configuración cuyo ámbito es la base de datos cambia
- Presión de memoria
- Solicitudes explícitas para borrar la caché

Si usa una sugerencia RECOMPILE, el plan no se almacenará en caché.

Una recompilación (o una compilación nueva tras la expulsión de la caché) aún puede provocar que se genere un plan de ejecución de consulta idéntico al original. Cuando el plan cambia del plan anterior u original, es probable que se produzcan estas explicaciones:

- **Diseño físico cambiado**: por ejemplo, los índices recién creados abarcan con mayor efectividad los requisitos de una consulta. Los nuevos índices se pueden usar en una nueva compilación si el optimizador de consultas decide que es más óptimo usar ese nuevo índice que usar la estructura de datos seleccionada originalmente para la primera versión de la ejecución de la consulta. Cualquier cambio físico en los objetos a los que se hace referencia puede dar lugar a una nueva opción de plan en tiempo de compilación.

- **Diferencias en los recursos del servidor**: cuando un plan de un sistema difiere del plan de otro sistema, la disponibilidad de recursos, como el número de procesadores disponibles, puede influir en el plan que se genera. Por ejemplo, si un sistema tiene más procesadores, podría elegirse un plan paralelo. Para obtener más información sobre el paralelismo en Azure SQL Database, vea [Configuración del grado máximo de paralelismo (MAXDOP) en Azure SQL Database](database/configure-max-degree-of-parallelism.md).

- **Estadísticas diferentes**: las estadísticas asociadas a los objetos a los que se hace referencia han cambiado o son significativamente diferentes de las estadísticas originales del sistema. Si las estadísticas cambian y se produce una recompilación, el optimizador de consultas usa las estadísticas a partir de cuando cambiaron. Las frecuencias y las distribuciones de datos de las estadísticas revisadas pueden diferir de las de la compilación original. Estos cambios se usan para crear estimaciones de cardinalidad. (Las *estimaciones de cardinalidad* son el número de filas que se espera que fluyan a través del árbol lógico de consultas). Los cambios en las estimaciones de cardinalidad pueden hacer que elijamos diferentes operadores físicos y órdenes asociadas de operaciones. Incluso los pequeños cambios en las estadísticas pueden dar lugar a un plan de ejecución de consultas modificado.

- **Versión del estimador de cardinalidad o nivel de compatibilidad de la base de datos modificados**: los cambios en el nivel de compatibilidad de la base de datos pueden permitir nuevas estrategias y características que podrían producir un plan de ejecución de consultas diferente. Más allá del nivel de compatibilidad de la base de datos, la deshabilitación o habilitación de la marca de seguimiento 4199 o el cambio del estado de la configuración con ámbito de la base de datos, QUERY_OPTIMIZER_HOTFIXES también puede influir en las opciones del plan de ejecución de consultas en tiempo de compilación. Las marcas de seguimiento 9481 (forzar CE heredada) y 2312 (forzar CE predeterminada) también afectan al plan.

## <a name="resource-limits-issues"></a>Problemas de límites de recursos

Por lo general, tanto el rendimiento lento de las consultas no relacionado con los planes de consulta no óptimos como los índices que faltan están relacionados con recursos insuficientes o sobreutilizados. Si el plan de consulta es óptimo, es posible que la consulta (y la base de datos) esté alcanzando los límites de recursos de la base de datos, el grupo elástico o la instancia administrada. Un ejemplo puede ser un rendimiento de escritura del registro excesivo para el nivel de servicio.

- Detección de problemas de los recursos mediante Azure Portal: Para ver si los límites de recursos son el problema, consulte [Supervisión de recursos de SQL Database](database/monitor-tune-overview.md#azure-sql-database-and-azure-sql-managed-instance-resource-monitoring). En el caso de las bases de datos únicas y los grupos elásticos, consulte [Recomendaciones de rendimiento de Database Advisor](database/database-advisor-implement-performance-recommendations.md) e [Información de rendimiento de consultas](database/query-performance-insight-use.md).
- Detección de límites de recursos mediante [Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#reaching-resource-limits)
- Detección de problemas de los recursos mediante [DMV](database/monitoring-with-dmvs.md):

  - La DMV [sys.dm_db_resource_stats](database/monitoring-with-dmvs.md#monitor-resource-use) devuelve el consumo de CPU, de E/S y de memoria de la base de datos. Hay una fila por cada intervalo de 15 segundos, incluso si no hay ninguna actividad en la base de datos. Los datos históricos se conservan durante una hora.
  - [sys.resource_stats](database/monitoring-with-dmvs.md#monitor-resource-use) devuelve datos de almacenamiento y uso de CPU para Azure SQL Database. Los datos se recopilan y agregan en intervalos de cinco minutos.
  - [Muchas consultas individuales que consumen mucha CPU de forma acumulativa](database/monitoring-with-dmvs.md#many-individual-queries-that-cumulatively-consume-high-cpu)

Si identifica que el problema es que el número de recursos es insuficiente, puede actualizar los recursos para aumentar la capacidad de la base de datos a fin de absorber los requisitos de la CPU. Para más información, consulte [Escalar recursos de base de datos única en Azure SQL Database](database/single-database-scale.md) y [Escalar recursos de grupos elásticos en Azure SQL Database](database/elastic-pool-scale.md). Para obtener información sobre cómo escalar una instancia administrada, consulte [Límites de recursos de nivel de servicio](managed-instance/resource-limits.md#service-tier-characteristics).

## <a name="performance-problems-caused-by-increased-workload-volume"></a>Problemas de rendimiento causados por un mayor volumen de carga de trabajo

Un aumento en el tráfico de la aplicación y en el volumen de la carga de trabajo puede incrementar el uso de la CPU. Sin embargo, debe tener cuidado para diagnosticar correctamente este problema. Cuando vea un problema de CPU elevada, responda a estas preguntas para determinar si el aumento se debe a cambios en el volumen de la carga de trabajo:

- ¿Son las consultas de la aplicación la causa del problema del uso elevado de la CPU?
- Para las [consultas que más CPU consumen que pueda identificar](database/monitoring-with-dmvs.md#the-cpu-issue-occurred-in-the-past):

  - ¿Había varios planes de ejecución asociados a la misma consulta? Si es así, ¿por qué?
  - En el caso de las consultas con el mismo plan de ejecución, ¿los tiempos de ejecución eran coherentes? ¿Aumentó el número de ejecuciones? En ese caso, es probable que el aumento de la carga de trabajo ocasione problemas de rendimiento.

Para resumir, si el plan de ejecución de consultas no se ejecutó de manera diferente pero el uso de la CPU aumentó junto con el número de ejecuciones, es probable que exista un problema de rendimiento relacionado con el aumento de la carga de trabajo.

No siempre es fácil identificar los cambios en el volumen de la carga de trabajo que está llevando a un problema con la CPU. Tenga en cuenta estos factores:

- **Cambio en el uso de los recursos**: por ejemplo, considere un escenario donde el uso de la CPU aumentó al 80 % durante un período prolongado de tiempo. El uso de CPU por sí solo no significa que el volumen de la carga de trabajo cambie. Las regresiones en el plan de ejecución de consultas y los cambios en la distribución de datos también pueden contribuir a un mayor uso de los recursos, incluso si la aplicación ejecuta la misma carga de trabajo.

- **La aparición de una nueva consulta**: una aplicación puede agregar un nuevo conjunto de consultas en diferentes momentos.

- **Aumento o disminución del número de solicitudes**: este escenario es la medida más obvia de la carga de trabajo. El número de consultas no siempre coincide con un mayor uso de los recursos. Sin embargo, esta métrica sigue siendo un indicio importante, suponiendo que otros factores no hayan cambiado.

Use Intelligent Insights para detectar los [aumentos de la carga de trabajo](database/intelligent-insights-troubleshoot-performance.md#workload-increase) y las [regresiones del plan](database/intelligent-insights-troubleshoot-performance.md#plan-regression).

- **Paralelismo**: un paralelismo excesivo puede empeorar el rendimiento de otras cargas de trabajo simultáneas al privar a otras consultas de recursos de CPU y subprocesos de trabajo. Para obtener más información sobre el paralelismo en Azure SQL Database, vea [Configuración del grado máximo de paralelismo (MAXDOP) en Azure SQL Database](database/configure-max-degree-of-parallelism.md).

## <a name="waiting-related-problems"></a>Problemas relacionados con la espera

Si ha eliminado un plan no óptimo y ha solucionado los problemas *relativos a la espera* que están relacionados con los problemas de ejecución, el problema de rendimiento se deberá probablemente a que las consultas están esperando a un recurso. Los problemas relacionados con la espera pueden tener estas causas:

- **Bloqueo**:

  es posible que una consulta mantenga el bloqueo en objetos de la base de datos mientras otras intentan acceder a los mismos objetos. Las consultas de bloqueo se pueden identificar mediante [DMV](database/monitoring-with-dmvs.md#monitoring-blocked-queries) o [Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#locking). Para obtener más información, consulte [Descripción y resolución de problemas de bloqueo en Azure SQL](database/understand-resolve-blocking.md).
- **Problemas de E/S**

  es posible que las consultas estén esperando a que se escriban las páginas en los archivos de datos o registro. En este caso, consulte las estadísticas de espera `INSTANCE_LOG_RATE_GOVERNOR`, `WRITE_LOG` o `PAGEIOLATCH_*` en la vista DMV. Consulte cómo usar DMV para [identificar problemas de rendimiento de E/S](database/monitoring-with-dmvs.md#identify-io-performance-issues).
- **Problemas de TempDB**

  si la carga de trabajo usa tablas temporales o se producen pérdidas de datos de TempDb en los planes, es posible que las consultas tengan un problema con el rendimiento de TempDb. Consulte cómo usar DMV para [identificar problemas de TempDB](database/monitoring-with-dmvs.md#identify-tempdb-performance-issues).
- **Problemas relacionados con la memoria**

  si la carga de trabajo no tiene suficiente memoria, la duración prevista de la página podría reducirse o las consultas podrían obtener menos memoria de la que necesitan. En algunos casos, la inteligencia integrada en el optimizador de consultas corregirá estos problemas. Consulte cómo usar DMV para [identificar problemas de concesión de memoria](database/monitoring-with-dmvs.md#identify-memory-grant-wait-performance-issues).

### <a name="methods-to-show-top-wait-categories"></a>Métodos para mostrar las categorías con mayor espera

Estos métodos se usan normalmente para mostrar las categorías principales de tipos de espera:

- Use Intelligent Insights para identificar consultas con degradación del rendimiento debido a [grandes esperas](database/intelligent-insights-troubleshoot-performance.md#increased-wait-statistic)
- Use el [Almacén de consultas](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) para buscar estadísticas de espera de cada consulta a lo largo del tiempo. En el Almacén de consultas, los tipos de espera se combinan en categorías de espera. Puede encontrar la asignación de categorías de espera a tipos de espera en [sys.query_store_wait_stats](/sql/relational-databases/system-catalog-views/sys-query-store-wait-stats-transact-sql#wait-categories-mapping-table).
- Use [sys.dm_db_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-wait-stats-azure-sql-database) para devolver información sobre todas las esperas que han encontrado los subprocesos que se ejecutaron durante una operación de consulta. Puede usar esta vista agregada para diagnosticar problemas de rendimiento con Azure SQL Database y también con consultas y lotes específicos. Las consultas pueden estar esperando recursos, en esperas de colas o esperas externas.
- Use [sys.dm_os_waiting_tasks](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql) para devolver información sobre la cola de tareas que están a la espera de recursos.

En escenarios de CPU elevada, puede ser que el Almacén de consultas y las estadísticas de espera no reflejen el uso de CPU en los siguientes casos:

- Las consultas que consumen mucha CPU todavía se están ejecutando.
- Las consultas que consumen mucha CPU se estaban ejecutando cuando se produjo una conmutación por error.

Las vistas DMV que realizan un seguimiento del Almacén de consultas y de las estadísticas de espera muestran los resultados únicamente de las consultas realizadas correctamente y de las que han agotado el tiempo de espera. No muestran los datos de las instrucciones que se ejecutan actualmente hasta que finaliza la instrucción. Use la vista de administración dinámica [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) para realizar un seguimiento de las consultas en ejecución y del tiempo de trabajo asociado.

> [!TIP]
> Herramientas adicionales:
>
> - [TigerToolbox waits and latches](https://github.com/Microsoft/tigertoolbox/tree/master/Waits-and-Latches) (TigerToolbox: esperas y bloqueos temporales)
> - [TigerToolbox: usp_whatsup](https://github.com/Microsoft/tigertoolbox/tree/master/usp_WhatsUp)

## <a name="see-also"></a>Consulte también

* [Configuración del grado máximo de paralelismo (MAXDOP) en Azure SQL Database](database/configure-max-degree-of-parallelism.md)
* [Descripción y resolución de problemas de bloqueo en Azure SQL Database](database/understand-resolve-blocking.md)

## <a name="next-steps"></a>Pasos siguientes

* [Introducción al ajuste y supervisión de SQL Database](database/monitor-tune-overview.md)
