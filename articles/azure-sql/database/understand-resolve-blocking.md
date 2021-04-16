---
title: Descripción y resolución de problemas de bloqueo en Azure SQL
titleSuffix: Azure SQL Database
description: Información general sobre temas específicos de las bases de datos de Azure SQL relacionados con bloqueos y cómo solucionarlos.
services: sql-database
dev_langs:
- TSQL
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: ''
ms.date: 3/02/2021
ms.openlocfilehash: e176c0399b191c7a511ea1d26388219b2cef1df8
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106107153"
---
# <a name="understand-and-resolve-azure-sql-database-blocking-problems"></a>Descripción y resolución de problemas de bloqueo en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="objective"></a>Objetivo

En el artículo se describen los bloqueos de bases de datos de Azure SQL y se muestra cómo solucionar problemas y resolver el bloqueo. 

En este artículo, el término "conexión" hace referencia a una única sesión iniciada en la base de datos. Cada conexión se muestra como un id. de sesión (SPID) o session_id en muchas DMV. A menudo se hace referencia a cada uno de estos SPID como un proceso, aunque no se trate de un contexto de proceso independiente en el sentido habitual. En su lugar, cada SPID consta de los recursos de servidor y las estructuras de datos necesarias para atender las solicitudes de una única conexión de un cliente determinado. Una sola aplicación cliente puede tener una o varias conexiones. Desde la perspectiva de Azure SQL Database, no hay ninguna diferencia entre varias conexiones de una sola aplicación cliente en un único equipo cliente y varias conexiones de varias aplicaciones cliente o varios equipos cliente, ya que son atómicas. Una conexión puede bloquear otra conexión, independientemente del cliente de origen.

> [!NOTE]
> **Este contenido está orientado a Azure SQL Database.** Azure SQL Database se basa en la versión estable más reciente del motor de base de datos de Microsoft SQL Server, por lo que gran parte del contenido es similar aunque las herramientas y opciones de solución de problemas pueden diferir. Para obtener más información sobre los bloqueos en SQL Server, consulte [Descripción y resolución de problemas de bloqueo en SQL Server](/troubleshoot/sql/performance/understand-resolve-blocking).

## <a name="understand-blocking"></a>Descripción del bloqueo 
 
El bloqueo es una característica inevitable e inherente de cualquier sistema de administración de bases de datos relacionales (RDBMS) con simultaneidad basada en bloqueo. Como se mencionó anteriormente, en SQL Server el bloqueo se produce cuando una sesión mantiene bloqueado un recurso específico y un segundo SPID intenta adquirir un tipo de bloqueo en conflicto en el mismo recurso. Normalmente, el período de tiempo durante el que el primer SPID bloquea el recurso es pequeño. Cuando la sesión propietaria libera el bloqueo, la segunda conexión puede bloquear el recurso y continuar el procesamiento. Este es un comportamiento normal y puede suceder muchas veces durante el transcurso de un día sin ningún efecto perceptible en el rendimiento del sistema.

La duración y el contexto de transacción de una consulta determinan el tiempo que durarán sus bloqueos y, por lo tanto, el impacto en otras consultas. Si la consulta no se ejecuta dentro de una transacción (y no se usan sugerencias de bloqueo), los bloqueos de las instrucciones SELECT solo se conservarán en un recurso mientras se está leyendo, no durante la consulta. En el caso de las instrucciones INSERT, UPDATE y DELETE, los bloqueos se mantienen durante la consulta, tanto para la coherencia de los datos como para permitir que la consulta se revierta si es necesario.

En el caso de las consultas ejecutadas dentro de una transacción, la duración de los bloqueos se determina según el tipo de consulta, el nivel de aislamiento de la transacción y en función de si se usan sugerencias de bloqueo en la consulta. Para obtener una descripción de los bloqueos, las sugerencias de bloqueo y los niveles de aislamiento de transacción, consulte los siguientes artículos:

* [Bloqueo del motor de base de datos](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide)
* [Personalizar las versiones de fila y bloqueo](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#customizing-locking-and-row-versioning)
* [Modos de bloqueo](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#lock_modes)
* [Compatibilidad de bloqueos](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#lock_compatibility)
* [Transactions](/sql/t-sql/language-elements/transactions-transact-sql)

Cuando los bloqueos aumentan hasta el extremo de que hay un efecto perjudicial en el rendimiento del sistema, se debe a uno de los siguientes motivos:

* Un SPID mantiene bloqueado un conjunto de recursos durante un periodo de tiempo prolongado antes de liberarlos. Este tipo de bloqueo se resuelve con el tiempo, aunque puede provocar una degradación del rendimiento.

* Un SPID mantiene bloqueado un conjunto de recursos y nunca los libera. Este tipo de bloqueo no se resuelve automáticamente e impide indefinidamente el acceso a los recursos afectados.

En el primer escenario, la situación puede resultar muy fluida, ya que los diferentes SPID generan el bloqueo de diferentes recursos a lo largo del tiempo, lo que crea un destino móvil. Estas situaciones son difíciles de solucionar con [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) para delimitar el problema a las consultas individuales. En cambio, la segunda situación da lugar a un estado coherente que puede ser más fácil de diagnosticar.

## <a name="applications-and-blocking"></a>Aplicaciones y bloqueo

Puede haber una tendencia a centrarse en los problemas de la plataforma y la optimización del servidor al enfrentarse a un problema de bloqueo. No obstante, puede no encontrar la solución si solo se presta atención a la base de datos y esta puede acaparar el tiempo y la energía que sería mejor enfocar en examinar la aplicación cliente y las consultas que envía. Independientemente del nivel de visibilidad que ofrece la aplicación en relación con las llamadas realizadas a la base de datos, un problema de bloqueo con frecuencia requiere de la inspección de las instrucciones SQL exactas que envía aplicación, así como del comportamiento exacto de la aplicación con respecto a la cancelación de consultas, la administración de conexiones, la captura de todas las filas de resultados, etc. Si la herramienta de desarrollo no permite un control explícito sobre la administración de conexiones, la cancelación de consultas, el tiempo de espera de las consultas y la obtención de resultados, entre otros, es posible que los problemas de bloqueo no se puedan resolver. Este potencial debe examinarse atentamente antes de seleccionar una herramienta de desarrollo de aplicaciones para Azure SQL Database, en especial para los entornos OLTP sensibles al rendimiento. 

Preste atención al rendimiento de la base de datos durante la fase de diseño y construcción de la base de datos y la aplicación. En concreto, se debe evaluar el consumo de recursos, el nivel de aislamiento y la longitud de la ruta de acceso de la transacción para cada consulta. Cada consulta y transacción deben ser lo más ligeras posible. Se debe ejercer una buena disciplina de administración de conexiones, ya que sin ella, es posible que la aplicación parezca tener un rendimiento aceptable con un número reducido de usuarios, pero el rendimiento podría degradarse significativamente a medida que el número de usuarios escale. 

Con un correcto diseño de la aplicación y la consulta, Azure SQL Database es capaz de admitir muchos miles de usuarios simultáneos en un único servidor, con pocos bloqueos.

> [!Note]
> Para más instrucciones sobre el desarrollo de aplicaciones, consulte [Solución de problemas de conectividad y otros errores con Azure SQL Database y Azure SQL Managed Instance](troubleshoot-common-errors-issues.md) y [Control de errores transitorios](/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/transient-fault-handling).

## <a name="troubleshoot-blocking"></a>Solución de problemas de bloqueo

Independientemente de la situación de bloqueo en la que se encuentre, la metodología para solucionar los problemas de bloqueo es la misma. Estas separaciones lógicas son lo que determinará el resto de la estructura de este artículo. El concepto consiste en encontrar el bloqueador de encabezado e identificar lo que está haciendo la consulta y por qué está bloqueada. Una vez que se identifica la consulta problemática (es decir, qué mantiene los bloqueos durante un periodo prolongado), el siguiente paso es analizar y determinar por qué se está produciendo el bloqueo. Después de comprender el motivo, podemos realizar cambios al rediseñar la consulta y la transacción.

Pasos de solución de problemas:

1. Identificar la sesión de bloqueo principal (bloqueador de encabezado)

2. Buscar la consulta y la transacción que están causando el bloqueo (lo que mantiene los bloqueos durante un período prolongado)

3. Analizar/comprender por qué se produce el bloqueo prolongado

4. Resolver el problema de bloqueo al rediseñar la consulta y la transacción

Ahora expliquemos detalladamente cómo localizar la sesión de bloqueo principal con una captura de datos adecuada.

## <a name="gather-blocking-information"></a>Recopilación de información de bloqueo

Para contrarrestar la dificultad de solucionar problemas de bloqueo, un administrador de bases de datos puede usar scripts de SQL que supervisen constantemente el estado de bloqueo en la base de datos de Azure SQL. Para recopilar estos datos, en esencia existen dos métodos. 

El primero consiste en consultar objetos de administración dinámica (DMO) y almacenar los resultados para las comparaciones a lo largo del tiempo. Algunos de los objetos a los que se hace referencia en este artículo son vistas de administración dinámica (DMV) y otros son funciones de administración dinámica (DMF). El segundo método consiste en usar XEvents para capturar lo que se está ejecutando. 


## <a name="gather-information-from-dmvs"></a>Recopilación de información a partir de DMV

Se hace referencia a DMV para solucionar los problemas de bloqueo con el objetivo de identificar el SPID (id. de sesión) en el encabezado de la cadena de bloqueo y la instrucción SQL. Busque los SPID objetivo que se van a bloquear. Si algún SPID está bloqueando por otro SPID, investigue el SPID que es propietario del recurso (el SPID que realiza el bloqueo). ¿El SPID propietario también está bloqueado? Puede recorrer la cadena para buscar el bloqueador de encabezado y, a continuación, investigar por qué está manteniendo el bloqueo.

Recuerde ejecutar cada uno de estos scripts en la base de datos de Azure SQL de destino.

* Los comandos sp_who y sp_who2 son comandos más antiguos para mostrar todas las sesiones actuales. La DMV sys.dm_exec_sessions devuelve más datos en un conjunto de resultados que resulta más fácil de consultar y filtrar. Encontrará sys.dm_exec_sessions en el centro de otras consultas. 

* Si ya tiene una sesión específica identificada, puede usar `DBCC INPUTBUFFER(<session_id>)` para buscar la última instrucción enviada por una sesión. Se pueden devolver resultados similares con la función de administración dinámica (DMF) sys.dm_exec_input_buffer en un conjunto de resultados que sea más fácil de consultar y filtrar, si se proporciona el session_id y el request_id. Por ejemplo, para devolver la consulta más reciente enviada por ession_id 66 y request_id 0, haga lo siguiente:

```sql
SELECT * FROM sys.dm_exec_input_buffer (66,0);
```

* Consulte sys.dm_exec_requests y haga referencia a la columna blocking_session_id. Cuando blocking_session_id = 0, una sesión no está bloqueada. Mientras sys.dm_exec_requests solo muestra las solicitudes que se están ejecutando actualmente, todas las conexiones (activas o no) se mostrarán en sys.dm_exec_sessions. Use como base esta combinación común entre sys.dm_exec_requests y sys.dm_exec_sessions en la consulta siguiente.

* Ejecute esta consulta de ejemplo para buscar las consultas que están ejecutándose activamente y su texto de proceso por lotes de SQL o texto de búfer de entrada actual mediante las DMV [sys.dm_exec_sql_text](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sql-text-transact-sql) o [sys.dm_exec_input_buffer](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-input-buffer-transact-sql). Si los datos devueltos por el campo `text` de sys.dm_exec_sql_text es NULL, la consulta no se está ejecutando actualmente. En ese caso, el campo `event_info` de sys.dm_exec_input_buffer contendrá la última cadena de comando pasada al motor de SQL. Esta consulta también se puede usar para identificar las sesiones que bloquean otras sesiones, incluida una lista de identificadores de sesión bloqueados por identificador de sesión. 

```sql
WITH cteBL (session_id, blocking_these) AS 
(SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                FROM sys.dm_exec_requests as er
                WHERE er.blocking_session_id = isnull(s.session_id ,0)
                AND er.blocking_session_id <> 0
                FOR XML PATH('') ) AS x (blocking_these)
)
SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
, batch_text = t.text, input_buffer = ib.event_info, * 
FROM sys.dm_exec_sessions s 
LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
INNER JOIN cteBL as bl on s.session_id = bl.session_id
OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
WHERE blocking_these is not null or r.blocking_session_id > 0
ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
```

* Ejecute esta consulta de ejemplo más elaborada, que proporciona el servicio de soporte técnico de Microsoft, para identificar el encabezado de una cadena de bloqueo de varias sesiones, como el texto de consulta de las sesiones que intervienen en una cadena de bloqueo.

```sql
WITH cteHead ( session_id,request_id,wait_type,wait_resource,last_wait_type,is_user_process,request_cpu_time
,request_logical_reads,request_reads,request_writes,wait_time,blocking_session_id,memory_usage
,session_cpu_time,session_reads,session_writes,session_logical_reads
,percent_complete,est_completion_time,request_start_time,request_status,command
,plan_handle,sql_handle,statement_start_offset,statement_end_offset,most_recent_sql_handle
,session_status,group_id,query_hash,query_plan_hash) 
AS ( SELECT sess.session_id, req.request_id, LEFT (ISNULL (req.wait_type, ''), 50) AS 'wait_type'
    , LEFT (ISNULL (req.wait_resource, ''), 40) AS 'wait_resource', LEFT (req.last_wait_type, 50) AS 'last_wait_type'
    , sess.is_user_process, req.cpu_time AS 'request_cpu_time', req.logical_reads AS 'request_logical_reads'
    , req.reads AS 'request_reads', req.writes AS 'request_writes', req.wait_time, req.blocking_session_id,sess.memory_usage
    , sess.cpu_time AS 'session_cpu_time', sess.reads AS 'session_reads', sess.writes AS 'session_writes', sess.logical_reads AS 'session_logical_reads'
    , CONVERT (decimal(5,2), req.percent_complete) AS 'percent_complete', req.estimated_completion_time AS 'est_completion_time'
    , req.start_time AS 'request_start_time', LEFT (req.status, 15) AS 'request_status', req.command
    , req.plan_handle, req.[sql_handle], req.statement_start_offset, req.statement_end_offset, conn.most_recent_sql_handle
    , LEFT (sess.status, 15) AS 'session_status', sess.group_id, req.query_hash, req.query_plan_hash
    FROM sys.dm_exec_sessions AS sess
    LEFT OUTER JOIN sys.dm_exec_requests AS req ON sess.session_id = req.session_id
    LEFT OUTER JOIN sys.dm_exec_connections AS conn on conn.session_id = sess.session_id 
    )
, cteBlockingHierarchy (head_blocker_session_id, session_id, blocking_session_id, wait_type, wait_duration_ms,
wait_resource, statement_start_offset, statement_end_offset, plan_handle, sql_handle, most_recent_sql_handle, [Level])
AS ( SELECT head.session_id AS head_blocker_session_id, head.session_id AS session_id, head.blocking_session_id
    , head.wait_type, head.wait_time, head.wait_resource, head.statement_start_offset, head.statement_end_offset
    , head.plan_handle, head.sql_handle, head.most_recent_sql_handle, 0 AS [Level]
    FROM cteHead AS head
    WHERE (head.blocking_session_id IS NULL OR head.blocking_session_id = 0)
    AND head.session_id IN (SELECT DISTINCT blocking_session_id FROM cteHead WHERE blocking_session_id != 0)
    UNION ALL
    SELECT h.head_blocker_session_id, blocked.session_id, blocked.blocking_session_id, blocked.wait_type,
    blocked.wait_time, blocked.wait_resource, h.statement_start_offset, h.statement_end_offset,
    h.plan_handle, h.sql_handle, h.most_recent_sql_handle, [Level] + 1
    FROM cteHead AS blocked
    INNER JOIN cteBlockingHierarchy AS h ON h.session_id = blocked.blocking_session_id and h.session_id!=blocked.session_id --avoid infinite recursion for latch type of blocking
    WHERE h.wait_type COLLATE Latin1_General_BIN NOT IN ('EXCHANGE', 'CXPACKET') or h.wait_type is null
    )
SELECT bh.*, txt.text AS blocker_query_or_most_recent_query 
FROM cteBlockingHierarchy AS bh 
OUTER APPLY sys.dm_exec_sql_text (ISNULL ([sql_handle], most_recent_sql_handle)) AS txt;
```

* Para detectar transacciones de larga duración o no confirmadas, use otro conjunto de DMV para ver las transacciones abiertas actuales, como [sys.dm_tran_database_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-database-transactions-transact-sql), [sys.dm_tran_session_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-session-transactions-transact-sql), [sys.dm_exec_connections](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-connections-transact-sql) y sys.dm_exec_sql_text. Hay varias DMV asociadas a las transacciones de seguimiento, consulte más [DMV sobre transacciones](/sql/relational-databases/system-dynamic-management-views/transaction-related-dynamic-management-views-and-functions-transact-sql) aquí. 

```sql
SELECT [s_tst].[session_id],
[database_name] = DB_NAME (s_tdt.database_id),
[s_tdt].[database_transaction_begin_time], 
[sql_text] = [s_est].[text] 
FROM sys.dm_tran_database_transactions [s_tdt]
INNER JOIN sys.dm_tran_session_transactions [s_tst] ON [s_tst].[transaction_id] = [s_tdt].[transaction_id]
INNER JOIN sys.dm_exec_connections [s_ec] ON [s_ec].[session_id] = [s_tst].[session_id]
CROSS APPLY sys.dm_exec_sql_text ([s_ec].[most_recent_sql_handle]) AS [s_est];
```

* Consulte [sys.dm_os_waiting_tasks](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql), que se encuentra en el nivel de subproceso o tarea de SQL. Esta devuelve información sobre el tipo de espera de SQL que está experimentando actualmente la solicitud. Al igual que con sys.dm_exec_requests, sys.dm_os_waiting_tasks devuelve solo las solicitudes activas. 

> [!Note]
> Para más información sobre los tipos de espera, incluidas las estadísticas de espera agregadas a lo largo del tiempo, consulte la [DMV sys.dm_db_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-wait-stats-azure-sql-database). Esta DMV devuelve estadísticas de espera agregadas solo para la base de datos actual.

* Use la DMV [sys.dm_tran_locks](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql) para obtener información más detallada sobre los bloqueos que han realizado las consultas. Esta DMV puede devolver grandes cantidades de datos en una instancia de SQL Server de producción y resulta útil para diagnosticar los bloqueos actuales. 

Debido a la instrucción INNER JOIN de sys.dm_os_waiting_tasks, la siguiente consulta restringe la salida de sys.dm_tran_locks solo a las solicitudes bloqueadas actualmente, su estado de espera y sus bloqueos:

```sql
SELECT table_name = schema_name(o.schema_id) + '.' + o.name
, wt.wait_duration_ms, wt.wait_type, wt.blocking_session_id, wt.resource_description
, tm.resource_type, tm.request_status, tm.request_mode, tm.request_session_id
FROM sys.dm_tran_locks AS tm
INNER JOIN sys.dm_os_waiting_tasks as wt ON tm.lock_owner_address = wt.resource_address
LEFT OUTER JOIN sys.partitions AS p on p.hobt_id = tm.resource_associated_entity_id
LEFT OUTER JOIN sys.objects o on o.object_id = p.object_id or tm.resource_associated_entity_id = o.object_id
WHERE resource_database_id = DB_ID()
AND object_name(p.object_id) = '<table_name>';
```

* Con las DMV, el almacenamiento de los resultados de la consulta con el tiempo proporcionará puntos de datos que le permitirán revisar el bloqueo durante un intervalo de tiempo especificado para identificar las tendencias o bloqueos persistentes. 

## <a name="gather-information-from-extended-events"></a>Recopilación de información sobre eventos extendidos

Además de la información anterior, a menudo es necesario capturar el seguimiento de las actividades en el servidor para investigar exhaustivamente un problema de bloqueo en Azure SQL Database. Por ejemplo, si una sesión ejecuta varias instrucciones dentro de una transacción, solo se representará la última instrucción que se haya enviado. Sin embargo, una de las instrucciones anteriores puede ser el motivo por el que los bloqueos continúan. Un seguimiento le permitirá ver todos los comandos ejecutados por una sesión durante la transacción actual.

Hay dos maneras de capturar los seguimientos en SQL Server; eventos extendidos (XEvents) y los de Profiler. Sin embargo, [SQL Server Profiler](/sql/tools/sql-server-profiler/sql-server-profiler) es una tecnología de seguimiento en desuso que no es compatible con Azure SQL Database. Los [eventos extendidos](/sql/relational-databases/extended-events/extended-events) son la nueva tecnología de seguimiento que ofrece mayor versatilidad y menor impacto en el sistema observado y su interfaz está integrada en SQL Server Management Studio (SSMS). 

Consulte el documento en el que se explica cómo usar el [Asistente para nueva sesión de eventos extendidos](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server) en SSMS. No obstante, en el caso de las bases de datos de Azure SQL, SSMS proporciona una subcarpeta de eventos extendidos en cada base de datos del Explorador de objetos. Use un asistente para sesión de eventos extendidos para capturar estos eventos útiles: 

-   Errores de categoría:
    -   Atención
    -   Error_reported  
    -   Execution_warning

-   Advertencias de categoría: 
    -   Missing_join_predicate

-   Ejecución de categoría:
    -   Rpc_completed
    -   Rpc_starting 
    -   Sql_batch_completed
    -   Sql_batch_starting

-   Bloquear
    -   Lock_deadlock

-   Sesión
    -   Existing_connection
    -   Iniciar sesión
    -   Logout

## <a name="identify-and-resolve-common-blocking-scenarios"></a>Identificación y resolución de escenarios de bloqueo comunes

Mediante el examen de la información anterior, puede determinar la causa de la mayoría de los problemas de bloqueo. En el resto de este artículo se explica cómo usar esta información para identificar y resolver algunos escenarios de bloqueo comunes. En este tema se da por sentado que ha usado los scripts de bloqueo (a los que se hace referencia anteriormente) para capturar información sobre los SPID de bloqueo y que ha capturado la actividad de la aplicación mediante una sesión de XEvent.

## <a name="analyze-blocking-data"></a>Análisis de datos de bloqueo 

* Examine la salida de las DMV sys.dm_exec_requests y sys.dm_exec_sessions para determinar el principio de las cadenas de bloqueo mediante blocking_these y session_id. Esto identificará con mayor claridad qué solicitudes están bloqueadas y cuáles están realizando el bloqueo. Examine más exhaustivamente las sesiones que están bloqueadas y que realizan el bloqueo. ¿Hay una raíz o elemento común para la cadena de bloqueo? Probablemente comparten una tabla común y una o varias de las sesiones implicadas en una cadena de bloqueo están realizando una operación de escritura. 

* Examine la salida de las DMV sys.dm_exec_requests y sys.dm_exec_sessions para obtener información sobre los SPID en el principio de la cadena de bloqueo. Busque los siguientes campos: 

    -    `sys.dm_exec_requests.status`  
    Esta columna muestra el estado de una solicitud determinada. Normalmente, un estado En espera indica que el SPID ha finalizado la ejecución y está esperando a que la aplicación envíe otra consulta o lote. Un estado ejecutable o en ejecución indica que el SPID está procesando una consulta actualmente. En la tabla siguiente se proporcionan explicaciones breves sobre los distintos valores de estado.

    | Status | Significado |
    |:-|:-|
    | Información previa | El SPID está ejecutando una tarea en segundo plano, como una detección de interbloqueos, un escritor de registros o un punto de control. |
    | En espera | El SPID no se está ejecutando actualmente. Normalmente, esto indica que el SPID está esperando un comando de la aplicación. |
    | En ejecución | El SPID está ejecutándose actualmente en un programador. |
    | Ejecutable | El SPID se encuentra en la cola de ejecutables de un programador y está en espera para obtener una hora del programador. |
    | Suspended | El SPID está esperando un recurso, como un bloqueo o un bloqueo temporal. | 
                       
    -    `sys.dm_exec_sessions.open_transaction_count`  
    Este campo indica el número de transacciones abiertas en esta sesión. Si este valor es mayor que 0, el SPID está dentro de una transacción abierta y puede estar manteniendo los bloqueos adquiridos por cualquier instrucción de la transacción.

    -    `sys.dm_exec_requests.open_transaction_count`  
    De igual modo, este campo indica el número de transacciones abiertas en esta solicitud. Si este valor es mayor que 0, el SPID está dentro de una transacción abierta y puede estar manteniendo los bloqueos adquiridos por cualquier instrucción de la transacción.

    -   `sys.dm_exec_requests.wait_type`, `wait_time` y `last_wait_type`  
    Si  `sys.dm_exec_requests.wait_type`  es NULL, la solicitud no está esperando nada y el valor de  `last_wait_type`  indica el último elemento  `wait_type`  que la solicitud encontró. Para obtener más información sobre  `sys.dm_os_wait_stats` y una descripción de los tipos de espera más comunes, consulte [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql). Se puede usar el valor de  `wait_time`  para determinar si la solicitud está avanzando. Cuando una consulta en la tabla sys.dm_exec_requests devuelve un valor en la columna  `wait_time`  que es menor que el valor de   `wait_time` de una consulta anterior de sys.dm_exec_requests, esto indica que el bloqueo anterior se adquirió, se liberó y ahora está en espera de un nuevo bloqueo (suponiendo que `wait_time` es distinto de cero). Esto se puede comprobar al comparar el elemento `wait_resource`  entre la salidas de sys.dm_exec_requests, que muestran el recurso al que está esperando la solicitud.

    -   `sys.dm_exec_requests.wait_resource` Este campo indica el recurso al que está esperando una solicitud bloqueada. En la tabla siguiente se enumeran los formatos comunes de  `wait_resource`  y su significado:

    | Resource | Formato | Ejemplo | Explicación | 
    |:-|:-|:-|:-|
    | Tabla | DatabaseID:ObjectID:IndexID | TAB: 5:261575970:1 | En este caso, el id. de base de datos 5 es la base de datos del ejemplo pubs, el id. de objeto 261575970 es la tabla titles y 1 es el índice agrupado. |
    | Página | DatabaseID:FileID:PageID | PAGE: 5:1:104 | En este caso, el id. de base de datos 5 es pubs, el id. de archivo 1 es el archivo de datos principal y la página 104 es una página que pertenece a la tabla titles. Para identificar el elemento object_id al que pertenece la página, use la función de administración dinámica [sys.dm_db_page_info](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-page-info-transact-sql) y pase los valores de DatabaseID, FileId, PageId de `wait_resource`. | 
    | Clave | DatabaseID:Hobt_id (valor hash de la clave de índice) | KEY: 5:72057594044284928 (3300a4f361aa) | En este caso, el id. de base de datos 5 es Pubs y Hobt_ID 72057594044284928 le corresponde a index_id 2 para object_id 261575970 (tabla titles). Use la vista de catálogo sys.partitions para asociar hobt_id a un index_id y object_id determinados. No hay ninguna manera de deshacer el hash de la clave de índice correspondiente a un valor de clave concreto. |
    | Row | DatabaseID:FileID:PageID:Slot(fila) | RID: 5:1:104:3 | En este caso, el id. de base de datos 5 es pubs, el id. de archivo 1 es el archivo de datos principal, la página 104 es una página que pertenece a la tabla titles y la ranura 3 indica la posición de la fila dentro de la página. |
    | Compile  | DatabaseID:FileID:PageID:Slot(fila) | RID: 5:1:104:3 | En este caso, el id. de base de datos 5 es pubs, el id. de archivo 1 es el archivo de datos principal, la página 104 es una página que pertenece a la tabla titles y la ranura 3 indica la posición de la fila dentro de la página. |

    -    `sys.dm_tran_active_transactions` La DMV [sys.dm_tran_active_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-active-transactions-transact-sql) contiene datos sobre transacciones abiertas que se pueden combinar con otras DMV para obtener un panorama completo de las transacciones que esperan una confirmación o reversión. Use la siguiente consulta para devolver información sobre transacciones abiertas combinadas con otras DMV, como [sys.dm_tran_session_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-session-transactions-transact-sql). Considere el estado actual de una transacción, el elemento `transaction_begin_time` y otros datos de la situación para evaluar si puede ser el origen de un bloqueo.

    ```sql
    SELECT tst.session_id, [database_name] = db_name(s.database_id)
    , tat.transaction_begin_time
    , transaction_duration_s = datediff(s, tat.transaction_begin_time, sysdatetime()) 
    , transaction_type = CASE tat.transaction_type  WHEN 1 THEN 'Read/write transaction'
                                                    WHEN 2 THEN 'Read-only transaction'
                                                    WHEN 3 THEN 'System transaction'
                                                    WHEN 4 THEN 'Distributed transaction' END
    , input_buffer = ib.event_info, tat.transaction_uow     
    , transaction_state  = CASE tat.transaction_state    
                WHEN 0 THEN 'The transaction has not been completely initialized yet.'
                WHEN 1 THEN 'The transaction has been initialized but has not started.'
                WHEN 2 THEN 'The transaction is active - has not been committed or rolled back.'
                WHEN 3 THEN 'The transaction has ended. This is used for read-only transactions.'
                WHEN 4 THEN 'The commit process has been initiated on the distributed transaction.'
                WHEN 5 THEN 'The transaction is in a prepared state and waiting resolution.'
                WHEN 6 THEN 'The transaction has been committed.'
                WHEN 7 THEN 'The transaction is being rolled back.'
                WHEN 8 THEN 'The transaction has been rolled back.' END 
    , transaction_name = tat.name, request_status = r.status
    , azure_dtc_state = CASE tat.dtc_state 
                        WHEN 1 THEN 'ACTIVE'
                        WHEN 2 THEN 'PREPARED'
                        WHEN 3 THEN 'COMMITTED'
                        WHEN 4 THEN 'ABORTED'
                        WHEN 5 THEN 'RECOVERED' END
    , tst.is_user_transaction, tst.is_local
    , session_open_transaction_count = tst.open_transaction_count  
    , s.host_name, s.program_name, s.client_interface_name, s.login_name, s.is_user_process
    FROM sys.dm_tran_active_transactions tat 
    INNER JOIN sys.dm_tran_session_transactions tst  on tat.transaction_id = tst.transaction_id
    INNER JOIN Sys.dm_exec_sessions s on s.session_id = tst.session_id 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    CROSS APPLY sys.dm_exec_input_buffer(s.session_id, null) AS ib;
    ```

    -   Otras columnas

        Las demás columnas de [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql) y [sys.dm_exec_request](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) también pueden ofrecer información sobre la causa de un problema. Su utilidad varía en función de las circunstancias del problema. Por ejemplo, puede determinar si el problema solo se produce desde determinados clientes (nombre de host), en determinadas bibliotecas de red (net_library), cuándo un SPID envió el último lote mediante `last_request_start_time` en sys.dm_exec_sessions, durante cuánto tiempo se ha estado ejecutando una solicitud mediante `start_time` en sys.dm_exec_requests, etc.


## <a name="common-blocking-scenarios"></a>Escenarios de bloqueo comunes

En la tabla siguiente se asignan los síntomas comunes con sus posibles causas.  

Las columnas `wait_type`, `open_transaction_count` y `status` hacen referencia a la información que devuelve [sys.dm_exec_request](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql), mientras que [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql)puede devolver otras columnas. La columna "¿Se resuelve?" indica si el bloqueo se resolverá por sí mismo o si se debe terminar la sesión mediante el comando `KILL`. Para más información, consulte [KILL (Transact-SQL)](/sql/t-sql/language-elements/kill-transact-sql).

| Escenario | Waittype | Open_Tran | Status | ¿Se resuelve? | Otros síntomas |  
|:-|:-|:-|:-|:-|:-|--|
| 1 | NOT NULL | >= 0 | ejecutable | Sí, cuando finaliza la consulta. | En sys.dm_exec_sessions, las columnas **reads**, **cpu_time** o **memory_usage** aumentarán con el tiempo. La duración de la consulta será elevada cuando se complete. |
| 2 | NULL | \>0 | en espera | No, pero se puede terminar el SPID. | Puede aparecer una señal de atención en la sesión de eventos extendidos para este SPID, que indica que se agotó el tiempo de espera de una consulta o que se ha cancelado. |
| 3 | NULL | \>= 0 | ejecutable | No. No se resolverá hasta que el cliente recupere todas las filas o se cierre la conexión. Se puede terminar el SPID, pero puede tardar hasta 30 segundos. | Si open_transaction_count = 0 y el SPID mantiene los bloqueos mientras el nivel de aislamiento de transacción es el valor predeterminado (READ COMMMITTED), se trata de una causa probable. |  
| 4 | Varía | \>= 0 | ejecutable | No. No se resolverá hasta que el cliente cancele las consultas o cierre las conexiones. Se pueden terminar los SPID, pero pueden tardar hasta 30 segundos. | La columna **hostname** de sys.dm_exec_sessions para el SPID en el principio de una cadena de bloqueo será la misma que la del SPID que está realizando el bloqueo. |  
| 5 | NULL | \>0 | revertir | Sí. | Puede aparecer una señal de atención en la sesión de eventos extendidos para este SPID, que indica que se agotó el tiempo de espera de una consulta o que se ha cancelado, o simplemente que se emitió una instrucción de reversión. |  
| 6 | NULL | \>0 | en espera | Con el tiempo. Cuando Windows NT determina que la sesión ya no está activa, se interrumpe la conexión a Azure SQL Database. | El valor de `last_request_start_time` en sys.dm_exec_sessions es muy anterior a la hora actual. |

## <a name="detailed-blocking-scenarios"></a>Escenarios de bloqueo detallados

1.  Bloqueo causado por una consulta en ejecución normal con un tiempo de ejecución largo

    **Solución:** la solución a este tipo de problema de bloqueo consiste en buscar maneras de optimizar la consulta. En realidad, esta clase de problema de bloqueo puede ser un simple problema de rendimiento y requerir que lo trate como tal. Para obtener información sobre cómo solucionar problemas de una consulta específica que se ejecuta con lentitud, consulte [Solución de problemas en consultas que se ejecutan con lentitud en SQL Server](/troubleshoot/sql/performance/troubleshoot-slow-running-queries). Para obtener más información, vea [Supervisión y optimización del rendimiento](/sql/relational-databases/performance/monitor-and-tune-for-performance). 

    Los informes del [Almacén de consultas](/sql/relational-databases/performance/best-practice-with-the-query-store) en SSMS también son una herramienta muy recomendable y valiosa para identificar las consultas más costosas y los planes de ejecución poco óptimos. Revise también la sección [Rendimiento inteligente](intelligent-insights-overview.md) de Azure Portal para la base de datos de Azure SQL, incluida la [Información de rendimiento de consultas](query-performance-insight-use.md).

    Si tiene una consulta de larga duración que está bloqueando a otros usuarios y no se puede optimizar, considere la posibilidad de moverla de un entorno OLTP a un sistema de informes dedicado, una [réplica sincrónica de solo lectura de la base de datos](read-scale-out.md).

1.  Bloqueo causado por un SPID en espera que tiene una transacción pendiente de confirmación

    A menudo, este tipo de bloqueo puede identificarse mediante un SPID que está en espera o esperando un comando, pero cuyo nivel de anidamiento de la transacción (`@@TRANCOUNT`, `open_transaction_count` de sys.dm_exec_requests) es mayor que cero. Esto puede suceder si la aplicación experimenta el agotamiento del tiempo de espera de una consulta o si emite una operación de cancelación sin emitir también el número necesario de instrucciones ROLLBACK o COMMIT. Cuando un SPID recibe un agotamiento de tiempo de espera de consulta o una operación de cancelación, finalizará la consulta y lote actuales, pero no revertirá automáticamente ni confirmará la transacción. La aplicación se encarga de esto, ya que Azure SQL Database no puede suponer que se debe revertir una transacción completa debido a la cancelación de una sola consulta. El agotamiento del tiempo de espera de una consulta o la operación de cancelación aparecerá como un evento de señal ATTENTION para el SPID en la sesión de eventos extendidos.

    Para mostrar una transacción explícita pendiente de confirmación, emita la siguiente consulta:

    ```sql
    CREATE TABLE #test (col1 INT);
    INSERT INTO #test SELECT 1;
    BEGIN TRAN
    UPDATE #test SET col1 = 2 where col1 = 1;
    ```

    A continuación, ejecute esta consulta en la misma ventana:
    ```sql
    SELECT @@TRANCOUNT;
    ROLLBACK TRAN
    DROP TABLE #test;
    ```

    La salida de la segunda consulta indica que el nivel de anidamiento de la transacción es uno. Todos los bloqueos adquiridos en la transacción se conservarán hasta que la transacción se confirme o se revierta. Si las aplicaciones abren y confirman transacciones explícitamente, un error de comunicación u otro tipo podría dejar la sesión y su transacción en estado abierto. 

    Use el script descrito anteriormente en este artículo basado en sys.dm_tran_active_transactions para identificar las transacciones actualmente no confirmadas en la instancia.

    **Soluciones**:

     -   además , esta clase de problema de bloqueo también puede ser un problema de rendimiento y requerir que lo trate como tal. Si se puede reducir el tiempo de ejecución de la consulta, no se producirá el agotamiento del tiempo de espera de la consulta o su cancelación. Es importante que la aplicación pueda controlar los escenarios de agotamiento del tiempo de espera o cancelación que surjan, aunque también puede beneficiarse del examen del rendimiento de la consulta. 
     
     -    Las aplicaciones deben administrar correctamente los niveles de anidamiento de las transacciones o podrían provocar un problema de bloqueo después de la cancelación de la consulta con este método. Tenga en cuenta lo siguiente.  

            *    En el controlador de errores de la aplicación cliente, ejecute `IF @@TRANCOUNT > 0 ROLLBACK TRAN` después de cualquier error, incluso si la aplicación cliente no considera que una transacción esté abierta. Debe comprobar las transacciones abiertas, ya que un procedimiento almacenado llamado durante el lote podría haber iniciado una transacción sin el conocimiento de la aplicación cliente. Ciertas condiciones, como cancelar la consulta, impiden que el procedimiento se ejecute después de la instrucción actual, de modo que incluso si el procedimiento tiene lógica para comprobar si `IF @@ERROR <> 0` y anular la transacción, este código de reversión no se ejecutará en estos casos.  
            *    Si se usa la agrupación de conexiones en una aplicación que abre la conexión y ejecuta un número pequeño de consultas antes de devolver la conexión al grupo, como una aplicación basada en web, deshabilitar temporalmente la agrupación de conexiones puede ayudar a mitigar el problema hasta que se modifique la aplicación cliente para controlar los errores de manera adecuada. Al deshabilitar la agrupación de conexiones, la liberación de la conexión provocará una desconexión física de Azure SQL Database, lo dará lugar a que el servidor revierta las transacciones abiertas.  
            *    Use `SET XACT_ABORT ON` para la conexión, o en cualquier procedimiento almacenado que inicie transacciones y que no realicen una limpieza después de un error. En caso de que se produzca un error en tiempo de ejecución, este valor anulará cualquier transacción abierta y devolverá el control al cliente. Para obtener más información, consulte [SET XACT_ABORT (Transact-SQL)](/sql/t-sql/statements/set-xact-abort-transact-sql).

    > [!NOTE]
    > La conexión no se restablece hasta que se vuelve a usar desde el grupo de conexiones, por lo que es posible que un usuario pueda abrir una transacción y, después, liberar la conexión en el grupo de conexiones, pero puede que no se vuelva a usar durante varios segundos, durante los que la transacción permanecerá abierta. Si no se reutiliza la conexión, la transacción se anulará cuando se agote el tiempo de espera de la conexión y se quite del grupo de conexiones. Por lo tanto, resulta un método óptimo para que la aplicación cliente anule las transacciones en el controlador de errores o use `SET XACT_ABORT ON` para evitar este posible retraso.

    > [!CAUTION]
    > De acuerdo con `SET XACT_ABORT ON`, no se ejecutarán las instrucciones T-SQL después de una instrucción que produce un error. Esto podría afectar al flujo previsto del código existente.

1.  Bloqueo causado por un SPID cuya aplicación cliente correspondiente no recuperó todas las filas de resultados hasta finalizar

    Después de enviar una consulta al servidor, todas las aplicaciones deben recuperar inmediatamente todas las filas de resultados hasta su finalización. Si una aplicación no recupera todas las filas de resultados, se pueden quedar bloqueos en las tablas que impiden el acceso a otros usuarios. Si usa una aplicación que envía de manera transparente instrucciones SQL al servidor, la aplicación debe capturar todas las filas de resultados. De lo contrario (y si no se puede configurar para ello), es posible que no pueda resolver el problema de bloqueo. Para evitar este problema, puede restringir las aplicaciones con un comportamiento deficiente a una base de datos de informes o de ayuda para la toma de decisiones, independiente de la base de datos OLTP.
    
    > [!NOTE]
    > Consulte la [guía de lógica de reintento](./troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) para las aplicaciones que se conectan a Azure SQL Database. 
    
    **Solución:** se debe volver a escribir la aplicación para que recupere todas las filas de resultados hasta su finalización. Esto no descarta el uso de [OFFSET y FETCH en la cláusula ORDER BY](/sql/t-sql/queries/select-order-by-clause-transact-sql#using-offset-and-fetch-to-limit-the-rows-returned) de una consulta para realizar la paginación del servidor.

1.  Bloqueo causado por una sesión que está en estado de reversión

    Se revertirá una consulta de modificación de datos que se haya terminado con una instrucción KILL o que se haya cancelado fuera de una transacción definida por el usuario. Esto también puede producirse como efecto secundario de la desconexión de una sesión de red del cliente, o cuando se selecciona una solicitud como elemento afectado del interbloqueo. A menudo, esto se puede identificar al observar la salida de sys.dm_exec_requests, que puede indicar el **comando** ROLLBACK y la **columna percent_complete** puede mostrar el progreso. 

    Gracias a la [característica Recuperación acelerada de la base de datos](../accelerated-database-recovery.md) que se introdujo en 2019, las reversiones de larga duración deben ser poco frecuentes.
    
    **Solución:** espere a que el SPID termine de revertir los cambios realizados. 

    Para evitar esta situación, no realice operaciones de escritura por lotes de gran tamaño, operaciones de creación de índices u operaciones de mantenimiento durante las horas punto de los sistemas OLTP. Si es posible, realice estas operaciones durante los períodos de baja actividad.

1.  Bloqueo causado por una conexión huérfana

    Si la aplicación cliente intercepta errores o se reinicia la estación de trabajo cliente, es posible que la sesión de red en el servidor no se cancele inmediatamente en algunos casos. Desde la perspectiva de Azure SQL Database, el cliente todavía parece estar presente y es posible que conserven los bloqueos adquiridos. Para obtener más información, consulte [Solución de problemas de conexiones huérfanas en SQL Server](/sql/t-sql/language-elements/kill-transact-sql#remarks). 

    **Solución:** si la aplicación cliente se desconectó sin limpiar correctamente los recursos, puede finalizar el SPID mediante el comando `KILL`. El comando `KILL` toma como entrada el valor del SPID. Por ejemplo, para terminar el SPID 99, use el siguiente comando:

    ```sql
    KILL 99
    ```

## <a name="see-also"></a>Consulte también

* [Supervisión y ajuste del rendimiento en Azure SQL Database e Instancia administrada de Azure SQL](./monitor-tune-overview.md)
* [Supervisión del rendimiento mediante el Almacén de consultas](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store)
* [Guía de versiones de fila y bloqueo de transacciones](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide)
* [SET TRANSACTION ISOLATION LEVEL](/sql/t-sql/statements/set-transaction-isolation-level-transact-sql)
* [Inicio rápido: Eventos extendidos en SQL Server](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server)
* [Intelligent Insights con inteligencia artificial para supervisar y solucionar problemas de rendimiento de base de datos](intelligent-insights-overview.md)

## <a name="learn-more"></a>Más información

* [Azure SQL Database: Mejorar el ajuste del rendimiento con el ajuste automático](https://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Database-Improving-Performance-Tuning-with-Automatic-Tuning)
* [Mejorar el rendimiento de Azure SQL Database con el ajuste automático](https://channel9.msdn.com/Shows/Azure-Friday/Improve-Azure-SQL-Database-Performance-with-Automatic-Tuning)
* [Entrega de rendimiento coherente con Azure SQL](/learn/modules/azure-sql-performance/)
* [Solución de problemas de conectividad y otros errores con Azure SQL Database y Azure SQL Managed Instance](troubleshoot-common-errors-issues.md)
* [Transient Fault Handling](/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/transient-fault-handling)
* [Configuración del grado máximo de paralelismo (MAXDOP) en Azure SQL Database](configure-max-degree-of-parallelism.md)
