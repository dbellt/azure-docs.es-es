---
title: Diagnóstico de rendimiento en Hiperescala
description: En este artículo se describe cómo solucionar los problemas de rendimiento de Hiperescala en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: seo-lt-2019 sqldbrb=1
ms.topic: troubleshooting
author: denzilribeiro
ms.author: denzilr
ms.reviewer: mathoma
ms.date: 10/18/2019
ms.openlocfilehash: 116212ff082a2d21e8609c41af82e8325c78156c
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704307"
---
# <a name="sql-hyperscale-performance-troubleshooting-diagnostics"></a>Diagnóstico de la solución de problemas de rendimiento de Hiperescala de SQL
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Para solucionar problemas de rendimiento en una base de datos de Hiperescala, las [metodologías generales de ajuste del rendimiento](monitor-tune-overview.md) en el nodo de proceso de Azure SQL Database es el punto de partida de una investigación del rendimiento. Sin embargo, dada la [arquitectura distribuida](service-tier-hyperscale.md#distributed-functions-architecture) de Hiperescala, se agregaron diagnósticos adicionales para ayudar. En este artículo se describen los datos de diagnóstico específicos de Hiperescala.

## <a name="log-rate-throttling-waits"></a>Esperas de limitación de la velocidad de registro

Cada nivel de servicio de Azure SQL Database tiene límites de velocidad de generación de registros que se aplican a través de la [gobernanza de las velocidades de registros](resource-limits-logical-server.md#transaction-log-rate-governance). En Hiperescala, el límite de generación de registros está establecido actualmente en 100 MB/s, independientemente del nivel de servicio. Sin embargo, hay ocasiones en las que la velocidad de generación de registros en la réplica de proceso principal debe limitarse para mantener los SLA de capacidad de recuperación. Esta limitación se produce cuando un [servidor de páginas u otra réplica de proceso](service-tier-hyperscale.md#distributed-functions-architecture) está muy por detrás de la aplicación de registros nuevos desde el servidor de registro.

Los tipos de espera siguientes (en [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql/)) describen los motivos por los que se puede limitar la velocidad de registros en la réplica de proceso principal:

|Tipo de espera    |Descripción                         |
|-------------          |------------------------------------|
|RBIO_RG_STORAGE        | Se produce cuando se limita la velocidad de generación de registros del nodo de proceso principal de una base de datos de Hiperescala debido al consumo de registro retrasado en los servidores de páginas.         |
|RBIO_RG_DESTAGE        | Se produce cuando se limita la velocidad de generación de registros del nodo de proceso de una base de datos de Hiperescala debido al consumo de registro retrasado por parte del almacenamiento de registros a largo plazo.         |
|RBIO_RG_REPLICA        | Se produce cuando se limita la velocidad de generación de registros del nodo de proceso de una base de datos de Hiperescala debido al consumo de registro retrasado por parte de las réplicas secundarias legibles.         |
|RBIO_RG_GEOREPLICA    | Se produce cuando se limita la velocidad de generación de registros del nodo de proceso de una base de datos de Hiperescala debido al consumo de registro retrasado por parte de la replicación geográfica secundaria.         |
|RBIO_RG_LOCALDESTAGE   | Se produce cuando se limita la velocidad de generación de registros del nodo de proceso de una base de datos de Hiperescala debido al consumo de registro retrasado por parte del servicio de registro.         |


## <a name="page-server-reads"></a>Lecturas del servidor de páginas

Las réplicas de proceso no almacenan en caché una copia completa de la base de datos de manera local. Los datos locales de la réplica de proceso se almacenan en el grupo de búferes (en memoria) y en la caché de la extensión del grupo de búferes resistentes (RBPEX) local, que es una caché parcial (sin cobertura) de páginas de datos. Esta caché de RBPEX local tiene un tamaño proporcional al tamaño de proceso y es 3 veces superior a la memoria del nivel de proceso. RBPEX es similar al grupo de búferes en que tiene los datos a los que se accede con mayor frecuencia. Por otro lado, cada servidor de páginas tiene una caché de RBPEX de cobertura para la parte de la base de datos que mantiene.

Cuando se emite una lectura en una réplica de proceso, si los datos no existen en el grupo de búferes o en la caché de RBPEX local, se emite una llamada de función getPage (pageId, LSN) y la página se captura desde el servidor de páginas correspondiente. Las lecturas de los servidores de páginas son lecturas remotas y, por tanto, son más lentas que las lecturas del RBPEX local. Al solucionar problemas de rendimiento relacionados con la E/S, es necesario poder saber cuántas E/S se realizaron a través de lecturas de servidor de páginas remotas relativamente más lentas.

Varias vistas administradas dinámicas (DMV) y eventos extendidos tienen columnas y campos que especifican el número de lecturas remotas de un servidor de páginas. Este número se puede comparar con el total de lecturas. El almacén de consultas también captura lecturas remotas como parte de las estadísticas de tiempo de ejecución de la consulta.

- Las columnas para informar sobre las lecturas del servidor de páginas están disponibles en las DMV de ejecución y las vistas de catálogo, como:

  - [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql/)
  - [sys.dm_exec_query_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql/)
  - [sys.dm_exec_procedure_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-procedure-stats-transact-sql/)
  - [sys.dm_exec_trigger_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-trigger-stats-transact-sql/)
  - [sys.query_store_runtime_stats](/sql/relational-databases/system-catalog-views/sys-query-store-runtime-stats-transact-sql/)
- Las lecturas del servidor de páginas se agregan a los siguientes eventos extendidos:
  - sql_statement_completed
  - sp_statement_completed
  - sql_batch_completed
  - rpc_completed
  - scan_stopped
  - query_store_begin_persist_runtime_stat
  - query-store_execution_runtime_info
- ActualPageServerReads/ActualPageServerReadAheads se agregan para consultar el archivo XML de plan para los planes reales. Por ejemplo:

`<RunTimeCountersPerThread Thread="8" ActualRows="90466461" ActualRowsRead="90466461" Batches="0" ActualEndOfScans="1" ActualExecutions="1" ActualExecutionMode="Row" ActualElapsedms="133645" ActualCPUms="85105" ActualScans="1" ActualLogicalReads="6032256" ActualPhysicalReads="0" ActualPageServerReads="0" ActualReadAheads="6027814" ActualPageServerReadAheads="5687297" ActualLobLogicalReads="0" ActualLobPhysicalReads="0" ActualLobPageServerReads="0" ActualLobReadAheads="0" ActualLobPageServerReadAheads="0" />`

> [!NOTE]
> Para ver estos atributos en la ventana de propiedades del plan de consulta, necesitará SSMS 18.3 o posterior.

## <a name="virtual-file-stats-and-io-accounting"></a>Estadísticas de archivos virtuales y cuentas de E/S

En Azure SQL Database, la DMF [sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql/) es la principal forma de supervisar la E/S de SQL Database. Las características de E/S en Hiperescala son diferentes debido a su [arquitectura distribuida](service-tier-hyperscale.md#distributed-functions-architecture). En esta sección, nos centramos en la E/S (lecturas y escrituras) en los archivos de datos que se muestran en esta DMF. En Hiperescala, cada archivo de datos visible en esta DMF corresponde a un servidor de páginas remoto. La caché de RBPEX mencionada aquí es una caché basada en SSD local que es una caché sin cobertura en la réplica de proceso.

### <a name="local-rbpex-cache-usage"></a>Uso de la caché de RBPEX local

La caché de RBPEX local existe en la réplica de proceso en el almacenamiento SSD local. Por lo tanto, la E/S de esta caché es más rápida que la E/S de los servidores de páginas remotos. Actualmente, [sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql/) en una base de datos de Hiperescala tiene una fila especial que informa de la E/S de la caché de RBPEX local en la réplica de proceso. Esta fila tiene el valor 0 para las columnas `database_id` y `file_id`. Por ejemplo, la consulta siguiente devuelve las estadísticas de uso de RBPEX desde el inicio de la base de datos.

`select * from sys.dm_io_virtual_file_stats(0,NULL);`

Una proporción de las lecturas realizadas en RBPEX a las lecturas agregadas realizadas en todos los demás archivos de datos indica la proporción de aciertos de caché de RBPEX. El contador `RBPEX cache hit ratio` también se expone en la vista de administración dinámica de los contadores de rendimiento `sys.dm_os_performance_counters`.                                                                        

### <a name="data-reads"></a>Lecturas de datos

- Cuando el motor de base de datos de SQL Server emite las lecturas en una réplica de proceso, pueden ser atendidas por la caché de RBPEX local, por servidores de páginas remotos o por una combinación de ambos, si se leen varias páginas.
- Cuando la réplica de proceso lee algunas páginas de un archivo concreto, por ejemplo, file_id 1, si estos datos residen únicamente en la memoria caché de RBPEX local, toda la E/S de esta lectura se contabiliza en file_id 0 (RBPEX). Si alguna parte de esos datos se encuentra en la memoria caché de RBPEX local y otra en un servidor de páginas remoto, la E/S se contabiliza en file_id 0 para la parte que se atiende desde RBPEX y la parte que se atiende desde el servidor de páginas remoto se contabiliza como file_id 1.
- Cuando una réplica de proceso solicita una página en un [LSN](/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide/) determinado desde un servidor de páginas, si el servidor de páginas no ha detectado la LSN solicitada, la lectura en la réplica de proceso esperará hasta que el servidor de páginas se ponga al día antes de que se devuelva la página a la réplica de proceso. Para cualquier lectura de un servidor de páginas en la réplica de proceso, verá el tipo de espera PAGEIOLATCH_* si está esperando en esa E/S. En Hiperescala, este tiempo de espera incluye el tiempo necesario para capturar la página solicitada en el servidor de páginas en el LSN requerido y el tiempo necesario para transferir la página del servidor de páginas a la réplica de proceso.
- Las lecturas de gran tamaño, como la lectura anticipada, se realizan a menudo mediante [ lecturas de "dispersión o recopilación"](/sql/relational-databases/reading-pages/). Estas lecturas permiten leer hasta 4 MB de páginas a la vez, lo que se considera una lectura única en el motor de base de datos de SQL Server. Sin embargo, cuando los datos que se leen se encuentran en RBPEX, estas lecturas se contabilizan como varias lecturas de 8 KB individuales, ya que el grupo de búferes y RBPEX siempre usan páginas de 8 KB. Como resultado, el número de E/S de lectura que se han encontrado en RBPEX puede ser mayor que el número real de E/S realizadas por el motor.

### <a name="data-writes"></a>Escrituras de datos

- La réplica de proceso principal no escribe directamente en los servidores de páginas. En cambio, las entradas de registro del servicio de registro se reproducen en los servidores de páginas correspondientes.
- Las escrituras que se producen en la réplica de proceso se escriben principalmente en el RBPEX local (file_id 0). En el caso de las escrituras en archivos lógicos mayores que 8 KB, es decir, las que se realizan con [recopilación y escritura](/sql/relational-databases/writing-pages/), cada operación de escritura se traduce en varias escrituras individuales de 8 KB en RBPEX, ya que el grupo de búferes y RBPEX siempre usan páginas de 8 KB. Como resultado, el número de E/S de escritura que se han encontrado en RBPEX puede ser mayor que el número real de E/S realizadas por el motor.
- Los archivos que no son de RBPEX o los archivos de datos distintos de file_id 0 que corresponden a los servidores de páginas también muestran escrituras. En el nivel de servicio de Hiperescala, estas escrituras se simulan, porque las réplicas de proceso nunca escriben directamente en los servidores de páginas. El rendimiento y las IOPS de escritura se contabilizan a medida que se producen en la réplica de proceso, pero la latencia de los archivos de datos distintos de file_id 0 no refleja la latencia real de las escrituras del servidor de página.

### <a name="log-writes"></a>Escrituras de registro

- En el proceso principal, una escritura de registro se contabiliza en file_id 2 de sys.dm_io_virtual_file_stats. Una escritura de registro en el proceso principal es una escritura en la zona de aterrizaje del registro.
- Las entradas de registro no se protegen en la réplica secundaria durante una confirmación ("commit"). En Hiperescala, el servicio de registro aplica el registro a las réplicas secundarias de forma asincrónica. Dado que las escrituras en el registro no suceden realmente en las réplicas secundarias, los recuentos de E/S de registro en estas solo se realizan con fines de seguimiento.

## <a name="data-io-in-resource-utilization-statistics"></a>E/S de datos en estadísticas de uso de recursos

En una base de datos que no sea de Hiperescala, las IOPS de lectura y escritura combinadas con los archivos de datos, en relación con el límite de IOPS de datos de [gobernanza de recursos](./resource-limits-logical-server.md#resource-governance), se muestran en las vistas [sys. dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) y [sys. resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database), en la columna `avg_data_io_percent`. El mismo valor se muestra en el portal como _Media de porcentaje de E/S de datos_.

En una base de datos de Hiperescala, esta columna informa del uso de IOPS de datos en relación con el límite para el almacenamiento local solo en la réplica de proceso, específicamente la E/S de RBPEX y `tempdb`. Un valor del 100% en esta columna indica que la gobernanza de recursos está limitando las IOPS del almacenamiento local. Si está correlacionado con un problema de rendimiento, ajuste la carga de trabajo para generar menos E/S, o bien aumente el objetivo del servicio de base de datos para aumentar el [límite](resource-limits-vcore-single-databases.md) _Max Data IOPS (IOPS de datos máximo)_ . Para la gobernanza de recursos de lecturas y escrituras de RBPEX, el sistema cuenta E/S individuales de 8 KB, en lugar de las E/S más grandes que puede indicar el motor de base de datos de SQL Server.

La E/S de datos en los servidores de páginas remotos no se notifica en las vistas de uso de recursos o en el portal, pero se notifica en la DMF [sys. dm_io_virtual_file_stats ()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql/), como se indicó anteriormente.

## <a name="additional-resources"></a>Recursos adicionales

- En el caso de los límites de recursos de núcleo virtual para una base de datos única de Hiperescala, consulte los [límites de núcleos virtuales del nivel de servicio de Hiperescala](resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5).
- Para supervisar las instancias de Azure SQL Database, habilite [Azure Monitor SQL Insights](../../azure-monitor/insights/sql-insights-overview.md).
- Para optimizar el rendimiento de Azure SQL Database, consulte [Ajuste manual del rendimiento de consultas en Azure SQL Database](performance-guidance.md).
- Para optimizar el rendimiento mediante el Almacén de consultas, consulte [Supervisión de rendimiento mediante el Almacén de consultas](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store/).
- Para los scripts de supervisión de DMV, consulte [Supervisión del rendimiento en Azure SQL Database con vistas de administración dinámica](monitoring-with-dmvs.md).