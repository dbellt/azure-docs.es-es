---
title: Supervisión de la actividad de copia de seguridad
titleSuffix: Azure SQL Managed Instance
description: Obtenga información sobre cómo supervisar la actividad de copia de seguridad de Azure SQL Managed Instance mediante eventos extendidos.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: Misliplavo
ms.author: mlazic
ms.reviewer: mathoma
ms.date: 12/14/2018
ms.openlocfilehash: 8c38b9d77baee05fe14c07dc5e6bc185bcb46538
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111897268"
---
# <a name="monitor-backup-activity-for-azure-sql-managed-instance"></a>Supervisión de la actividad de copia de seguridad para Azure SQL Managed Instance 
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo, se muestra cómo configurar sesiones de eventos extendidos (XEvent) para supervisar la actividad de copia de seguridad de [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md). 

## <a name="overview"></a>Información general

Azure SQL Managed Instance emite eventos (también conocidos como [eventos extendidos o XEvents](../database/xevent-db-diff-from-svr.md)) durante la actividad de copia de seguridad con el fin de crear informes. Configure una sesión de XEvent para realizar un seguimiento de información como el estado de la copia de seguridad, el tipo de copia de seguridad, el tamaño, la hora y la ubicación en la base de datos msdb. Esta información se puede integrar con el software de supervisión de copia de seguridad y también se puede usar para las auditorías de la empresa. 

Las auditorías de la empresa pueden requerir una evidencia de copias de seguridad correctas, la hora de copia de seguridad y la duración de la copia de seguridad.

## <a name="configure-xevent-session"></a>Configuración de la sesión de XEvent

Use el evento extendido `backup_restore_progress_trace` para registrar el progreso de la copia de seguridad de SQL Managed Instance. Modifique las sesiones de XEvent según sea necesario para realizar un seguimiento de la información que le interesa para su empresa. Estos fragmentos de código T-SQL almacenan las sesiones de XEvent en el búfer en anillo, pero también es posible escribir en [Azure Blob Storage](../database/xevent-code-event-file.md). Las sesiones de XEvent que almacenan datos en el búfer en anillo tienen un límite de aproximadamente 1000 mensajes, por lo que solo se deben usar para realizar un seguimiento de la actividad reciente. Además, los datos del búfer en anillo se pierden tras la conmutación por error. Por lo tanto, para un registro histórico de copias de seguridad, escriba en un archivo de eventos en su lugar. 

### <a name="simple-tracking"></a>Seguimiento simple

Configure una sesión de XEvent simple para capturar eventos simples sobre las copias de seguridad completas. Este script recopila el nombre de la base de datos, el número total de bytes procesados y la hora en la que se completó la copia de seguridad.

Use Transact-SQL (T-SQL) para configurar la sesión de XEvent simple: 


```sql
CREATE EVENT SESSION [Simple backup trace] ON SERVER
ADD EVENT sqlserver.backup_restore_progress_trace(
WHERE operation_type = 0
AND trace_message LIKE '%100 percent%')
ADD TARGET package0.ring_buffer
WITH(STARTUP_STATE=ON)
GO
ALTER EVENT SESSION [Simple backup trace] ON SERVER
STATE = start;
```



### <a name="verbose-tracking"></a>Seguimiento detallado

Configure una sesión de XEvent detallada para realizar un seguimiento más detallado de la actividad de copia de seguridad. Este script captura el inicio y el final de las copias de seguridad completas, diferenciales y de registros. Dado que este script es más detallado, rellena el búfer en anillo más rápido, por lo que las entradas se pueden reciclar más rápido que con el script simple. 

Use Transact-SQL (T-SQL) para configurar la sesión de XEvent detallada: 

```sql
CREATE EVENT SESSION [Verbose backup trace] ON SERVER 
ADD EVENT sqlserver.backup_restore_progress_trace(
    WHERE (
              [operation_type]=(0) AND (
              [trace_message] like '%100 percent%' OR 
              [trace_message] like '%BACKUP DATABASE%' OR [trace_message] like '%BACKUP LOG%'))
       )
ADD TARGET package0.ring_buffer
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,
       MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,
       TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)

ALTER EVENT SESSION [Verbose backup trace] ON SERVER
STATE = start;

```

## <a name="monitor-backup-progress"></a>Supervisión del progreso de la copia de seguridad 

Una vez creada la sesión de XEvent, puede usar Transact-SQL (T-SQL) para consultar los resultados del búfer en anillo y supervisar el progreso de la copia de seguridad. Una vez que se inicia XEvent, recopila todos los eventos de copia de seguridad, por lo que se agregan entradas a la sesión aproximadamente cada 5-10 minutos.  

### <a name="simple-tracking"></a>Seguimiento simple

El siguiente código de Transact-SQL (T-SQL) consulta la sesión de XEvent simple y devuelve el nombre de la base de datos, el número total de bytes procesados y la hora en la que se completó la copia de seguridad: 

```sql 
WITH
a AS (SELECT xed = CAST(xet.target_data AS xml)
FROM sys.dm_xe_session_targets AS xet
JOIN sys.dm_xe_sessions AS xe
ON (xe.address = xet.event_session_address)
WHERE xe.name = 'Backup trace'),
b AS(SELECT
d.n.value('(@timestamp)[1]', 'datetime2') AS [timestamp],
ISNULL(db.name, d.n.value('(data[@name="database_name"]/value)[1]', 'varchar(200)')) AS database_name,
d.n.value('(data[@name="trace_message"]/value)[1]', 'varchar(4000)') AS trace_message
FROM a
CROSS APPLY  xed.nodes('/RingBufferTarget/event') d(n)
LEFT JOIN master.sys.databases db
ON db.physical_database_name = d.n.value('(data[@name="database_name"]/value)[1]', 'varchar(200)'))
SELECT * FROM b
```

La captura de pantalla siguiente muestra un ejemplo de la salida de la consulta anterior: 

![Captura de pantalla de la salida de xEvent](./media/backup-activity-monitor/present-xevents-output.png)

En este ejemplo, se ha realizado una copia de seguridad automática de cinco bases de datos en el transcurso de 2 horas y 30 minutos, y hay 130 entradas en la sesión de XEvent. 

### <a name="verbose-tracking"></a>Seguimiento detallado 

El siguiente código Transact-SQL (T-SQL) consulta la sesión de XEvent detallada y devuelve el nombre de la base de datos, así como el inicio y el final de las copias de seguridad completas, diferenciales y de registros. 


```sql
WITH
a AS (SELECT xed = CAST(xet.target_data AS xml)
FROM sys.dm_xe_session_targets AS xet
JOIN sys.dm_xe_sessions AS xe
ON (xe.address = xet.event_session_address)
WHERE xe.name = 'Verbose backup trace'),
b AS(SELECT
d.n.value('(@timestamp)[1]', 'datetime2') AS [timestamp],
ISNULL(db.name, d.n.value('(data[@name="database_name"]/value)[1]', 'varchar(200)')) AS database_name,
d.n.value('(data[@name="trace_message"]/value)[1]', 'varchar(4000)') AS trace_message
FROM a
CROSS APPLY  xed.nodes('/RingBufferTarget/event') d(n)
LEFT JOIN master.sys.databases db
ON db.physical_database_name = d.n.value('(data[@name="database_name"]/value)[1]', 'varchar(200)'))
SELECT * FROM b
```

En la captura de pantalla siguiente, se muestra un ejemplo de una copia de seguridad completa en la sesión de XEvent:

:::image type="content" source="media/backup-activity-monitor/output-with-full.png" alt-text="Salida de XEvent que muestra copias de seguridad completas":::

En la captura de pantalla siguiente, se muestra un ejemplo de una salida de una copia de seguridad diferencial en la sesión de XEvent:

:::image type="content" source="media/backup-activity-monitor/output-with-differential.png" alt-text="Salida de XEvent que muestra copias de seguridad diferenciales":::


## <a name="next-steps"></a>Pasos siguientes

Una vez completada la copia de seguridad, puede [restaurar a un momento dado](point-in-time-restore.md) o [configurar una directiva de retención a largo plazo](long-term-backup-retention-configure.md). 

Para más información, consulte [Copias de seguridad automatizadas - Azure SQL Database e Instancia administrada de SQL](../database/automated-backups-overview.md). 
