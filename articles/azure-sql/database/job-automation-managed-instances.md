---
title: Automatización de trabajos con trabajos del Agente SQL en Azure SQL Managed Instance
description: Opciones de automatización para ejecutar scripts de Transact-SQL (T-SQL) en Azure SQL Managed Instance
services: sql-database
ms.service: sql-db-mi
ms.subservice: deployment-configuration
ms.custom: sqldbrb=1
dev_langs:
- TSQL
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: ''
ms.date: 02/01/2021
ms.openlocfilehash: 326ac64d5ea8948b9f9990a76dbbcf76d9915d5a
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111415385"
---
# <a name="automate-management-tasks-using-sql-agent-jobs-in-azure-sql-managed-instance"></a>Automatización de las tareas de administración con trabajos del Agente SQL en Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Con el [Agente SQL Server](/sql/ssms/agent/sql-server-agent) en SQL Server y [SQL Managed Instance](../../azure-sql/managed-instance/sql-managed-instance-paas-overview.md), puede crear y programar trabajos que se pueden ejecutar periódicamente en una o varias bases de datos para realizar consultas Transact-SQL (T-SQL) y llevar a cabo tareas de mantenimiento. En este artículo, se presentó el Agente SQL para SQL Managed Instance.

> [!Note]
> El Agente SQL no está disponible en Azure SQL Database ni en Azure Synapse Analytics. En su lugar, se recomienda la [automatización de trabajos con trabajos elásticos](job-automation-overview.md).

### <a name="sql-agent-job-limitations-in-azure-sql-managed-instance"></a>Limitaciones de los trabajos del Agente SQL en Azure SQL Managed Instance

Es importante destacar las diferencias entre el Agente SQL disponible en SQL Server y como parte de SQL Managed Instance. Para más información sobre las diferencias de las características compatibles entre SQL Server y SQL Managed Instance, consulte [Diferencias entre T-SQL de Azure SQL Managed Instance y SQL Server](../../azure-sql/managed-instance/transact-sql-tsql-differences-sql-server.md#sql-server-agent). 

Algunas de las características del Agente SQL que están disponibles en SQL Server no son compatibles con SQL Managed Instance:

- La configuración del Agente SQL es de solo lectura. 
    - El procedimiento almacenado del sistema `sp_set_agent_properties` no es compatible con SQL Managed Instance.
- La habilitación o deshabilitación del Agente SQL no se admite actualmente en SQL Managed Instance. El Agente SQL se ejecuta de forma continua.
- Las notificaciones se admiten parcialmente:
  - No se admite el paginador.
  - No se admite NetSend.
  - No se admiten las alertas.
- No se admiten servidores proxy.
- No se admite Eventlog.
- No se admite el desencadenador de programación de trabajos basado en una CPU inactiva.

## <a name="when-to-use-sql-agent-jobs"></a>Cuándo usa los trabajos del Agente SQL 

Son varios los escenarios en los que podría usar los trabajos del Agente SQL:

- Automatización de las tareas de administración y, luego, su programación para ejecutarlas todos los días laborables, después de horas, etc.
  - Implementación de cambios de esquema, administración de credenciales, recopilación de datos de rendimiento o recopilación de datos de telemetría de inquilinos (clientes).
  - Actualización de datos de referencia (información común entre todas las bases de datos), carga de datos desde Azure Blob Storage.
  - Tareas de mantenimiento comunes que incluyen DBCC CHECKDB para garantizar la integridad de datos o el mantenimiento de índices a fin de mejorar el rendimiento de la consulta. Configuración de trabajos para su ejecución en una colección de bases de datos periódicamente como, por ejemplo, fuera de horas pico.
  - Recopile los resultados de consulta de un conjunto de bases de datos en una tabla central de forma continua. Las consultas de rendimiento pueden ejecutarse continuamente y configurarse para que desencadenen la ejecución de otras tareas.
- Recopilación de datos para informes
  - Agregue datos de una colección de bases de datos a una tabla de destino única.
  - Ejecute consultas de procesamiento de datos de ejecución más larga en un conjunto grande de bases de datos; por ejemplo, la recopilación de telemetría de cliente. Los resultados se recopilan en una sola tabla de destino para su posterior análisis.
- Movimientos de datos
  - Cree trabajos que repliquen los cambios realizados en las bases de datos a otras bases de datos o que recopilen las actualizaciones realizadas en las bases de datos remotas y apliquen los cambios en la base de datos.
  - Cree trabajos que carguen los datos desde o hacia sus bases de datos mediante SQL Server Integration Services (SSIS).

## <a name="sql-agent-jobs-in-azure-sql-managed-instance"></a>Trabajos del Agente SQL en Azure SQL Managed Instance

Los trabajos del Agente SQL los ejecuta el servicio Agente SQL que se sigue usando para la automatización de tareas en SQL Server y SQL Managed Instance. 

Los trabajos del Agente SQL son una serie especificada de scripts de T-SQL que se ejecutan en la base de datos. Use trabajos para definir una tarea administrativa que se puede ejecutar una o varias veces y supervisar para detectar si lo hace correctamente o con errores. 

Un trabajo se puede ejecutar en un servidor local o en varios servidores remotos. Los trabajos del Agente SQL son un componente interno del Motor de base de datos que se ejecuta dentro del servicio SQL Managed Instance.

En los trabajos del Agente SQL, hay varios conceptos clave que conviene describir:

- **Pasos del trabajo**: conjunto de uno o varios pasos que se deben ejecutar dentro del trabajo. Para cada paso de trabajo se puede definir la estrategia de reintento y la acción que tendrá lugar si el paso del trabajo se realiza correctamente o con errores.
- **Programaciones**: definen cuándo se debe ejecutar el trabajo.
- **Notificaciones**: le permiten definir las reglas que se usarán para notificar a los operadores la finalización del trabajo mediante un mensaje de correo electrónico.

### <a name="sql-agent-job-steps"></a>Pasos del trabajo del Agente SQL

Los pasos del trabajo del Agente SQL son secuencias de acciones que debe ejecutar el Agente SQL. Cada paso tiene el siguiente paso que se debe ejecutar si el paso se realiza correctamente o con errores y el número de reintentos en caso de error.

El Agente SQL le permite crear distintos tipos de pasos de trabajo, como pasos de trabajo de Transact-SQL que ejecutan un único lote de Transact-SQL en la base de datos, pasos de PowerShell o de comandos del sistema operativo que pueden ejecutar scripts personalizados del sistema operativo, [pasos de trabajo de SSIS](../../data-factory/how-to-invoke-ssis-package-managed-instance-agent.md) que le permiten cargar datos con el runtime de SSIS, o pasos de [replicación](../managed-instance/replication-transactional-overview.md) que pueden publicar los cambios de la base de datos en otras bases de datos.

> [!Note]
> Para más información sobre cómo aprovechar Azure-SSIS Integration Runtime con SSISDB hospedado por Azure SQL Managed Instance, consulte [Uso de Azure SQL Managed Instance con SQL Server Integration Services (SSIS) en Azure Data Factory](../../data-factory/how-to-use-sql-managed-instance-with-ir.md).

La [replicación transaccional](../managed-instance/replication-transactional-overview.md) puede replicar los cambios de las tablas en otras bases de datos de Azure SQL Managed Instance, Azure SQL Database o SQL Server. Para más información, vea [Configuración de la replicación en una Instancia administrada de Azure SQL](../../azure-sql/managed-instance/replication-between-two-instances-configure-tutorial.md). 

Actualmente, no se admiten otros tipos de pasos de trabajo en SQL Managed Instance, entre los que se incluyen los siguientes:

- No se admite el paso de trabajo de replicación de mezcla.
- Aún no se admite el lector de colas.
- No se admite Analysis Services.
 
### <a name="sql-agent-job-schedules"></a>Programaciones de trabajos del Agente SQL

Una programación especifica cuándo se ejecuta un trabajo. Se puede ejecutar más de un trabajo en la misma programación y se puede aplicar más de una programación al mismo trabajo.

Una programación puede definir las condiciones siguientes para el momento en que se ejecuta un trabajo:

- Cada vez que se inicia el Agente SQL Server. El trabajo se activa después de cada conmutación por error.
- Una vez, en una fecha y a una hora específicas, que sea útil para la ejecución retrasada de algún trabajo.
- Según una programación periódica.

> [!Note]
> SQL Managed Instance actualmente no permite iniciar un trabajo cuando la CPU está inactiva.

### <a name="sql-agent-job-notifications"></a>Notificaciones de un trabajo del Agente SQL

Los trabajos del Agente SQL le permiten recibir notificaciones cuando el trabajo finaliza correctamente o con errores. Puede recibir notificaciones por correo electrónico.

Si todavía no está habilitada, primero deberá configurar [la característica Correo electrónico de base de datos](/sql/relational-databases/database-mail/database-mail) en Azure SQL Managed Instance:

```sql
GO
EXEC sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
EXEC sp_configure 'Database Mail XPs', 1;
GO
RECONFIGURE
```

Como un ejercicio de ejemplo, configure la cuenta de correo electrónico que se utilizará para enviar las notificaciones por correo electrónico. Asigne la cuenta al perfil de correo electrónico denominado `AzureManagedInstance_dbmail_profile`. Para enviar correos electrónicos mediante los trabajos del Agente SQL en SQL Managed Instance, tiene que haber un perfil que se debe denominar `AzureManagedInstance_dbmail_profile`. De lo contrario, SQL Managed Instance no podrá enviar correos electrónicos a través del Agente SQL. Consulte el ejemplo siguiente:

```sql
-- Create a Database Mail account
EXECUTE msdb.dbo.sysmail_add_account_sp
    @account_name = 'SQL Agent Account',
    @description = 'Mail account for Azure SQL Managed Instance SQL Agent system.',
    @email_address = '$(loginEmail)',
    @display_name = 'SQL Agent Account',
    @mailserver_name = '$(mailserver)' ,
    @username = '$(loginEmail)' ,
    @password = '$(password)';

-- Create a Database Mail profile
EXECUTE msdb.dbo.sysmail_add_profile_sp
    @profile_name = 'AzureManagedInstance_dbmail_profile',
    @description = 'E-mail profile used for messages sent by Managed Instance SQL Agent.';

-- Add the account to the profile
EXECUTE msdb.dbo.sysmail_add_profileaccount_sp
    @profile_name = 'AzureManagedInstance_dbmail_profile',
    @account_name = 'SQL Agent Account',
    @sequence_number = 1;
```

Pruebe la configuración de Correo electrónico de base de datos con T-SQL mediante el procedimiento almacenado del sistema [sp_send_db_mail](/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql):

```sql
DECLARE @body VARCHAR(4000) = 'The email is sent from ' + @@SERVERNAME;
EXEC msdb.dbo.sp_send_dbmail  
    @profile_name = 'AzureManagedInstance_dbmail_profile',  
    @recipients = 'ADD YOUR EMAIL HERE',  
    @body = 'Add some text',  
    @subject = 'Azure SQL Instance - test email';  
```

Puede notificar al operador que algo ha sucedido con los trabajos del Agente SQL. Un operador define la información de contacto de los responsables del mantenimiento de una o varias instancias de Instancia administrada de SQL. En ocasiones, las responsabilidades del operador se asignan a una sola persona.

En sistemas con varias instancias de Instancia administrada de SQL o SQL Server, muchas personas pueden compartir las responsabilidades del operador. Un operador no contiene información de seguridad y no define una entidad de seguridad. Idealmente, un operador no es un individuo cuyas responsabilidades pueden cambiar, sino un grupo de distribución de correo electrónico.   

Puede [crear operadores](/sql/relational-databases/system-stored-procedures/sp-add-operator-transact-sql) con SQL Server Management Studio (SSMS) o el script de Transact-SQL que se muestra en el ejemplo siguiente:

```sql
EXEC msdb.dbo.sp_add_operator
    @name=N'AzureSQLTeam',
    @enabled=1,
    @email_address=N'AzureSQLTeamn@contoso.com';
```

Confirme si el correo electrónico se envió correctamente o no con el [registro de Correo electrónico de base de datos](/sql/relational-databases/database-mail/database-mail-log-and-audits) en SSMS.

Luego puede [modificar cualquier trabajo del Agente SQL](/sql/relational-databases/system-stored-procedures/sp-update-job-transact-sql) y asignar los operadores a los que se les notificará por correo electrónico si el trabajo se completa, produce errores o se realiza correctamente mediante SSMS o el script de Transact-SQL siguiente:

```sql
EXEC msdb.dbo.sp_update_job @job_name=N'Load data using SSIS',
    @notify_level_email=3, -- Options are: 1 on succeed, 2 on failure, 3 on complete
    @notify_email_operator_name=N'AzureSQLTeam';
```

### <a name="sql-agent-job-history"></a>Historial de trabajos del Agente SQL

Actualmente, Azure SQL Managed Instance no permite cambiar ninguna propiedad del Agente SQL, porque se almacenan en los valores del registro subyacente. Esto sgnifica que las opciones para ajustar la directiva de retención del Agente para los registros del historial de trabajos se fijan en el valor predeterminado de 1000 registros totales y un máximo de 100 registros históricos por trabajo.

### <a name="sql-agent-fixed-database-role-membership"></a>Pertenencia al rol fijo de base de datos del Agente SQL

Si los usuarios vinculados a inicios de sesión que no son sysadmin se agregan a cualquiera de los tres roles fijos de base de datos del Agente SQL en la base de datos del sistema msdb, existe un problema en el que los permisos EXECUTE explícitos se deben conceder a los procedimientos almacenados maestros para que estos inicios de sesión funcionen. Si se encuentra este problema, aparece el mensaje de error "Se denegó el permiso EXECUTE en el objeto <nombre_objeto> (Microsoft SQL Server, error: 229)". 

Una vez que agrega usuarios a un rol fijo de base de datos del Agente SQL (SQLAgentUserRole, SQLAgentReaderRole o SQLAgentOperatorRole) en msdb, para cada inicio de sesión del usuario agregado a estos roles, ejecute el script T-SQL siguiente para conceder explícitamente permisos EXECUTE a los procedimientos almacenados del sistema que se muestran. En este ejemplo se da por supuesto que el nombre de usuario y el nombre de inicio de sesión son los mismos. 

```sql
USE [master]
GO
CREATE USER [login_name] FOR LOGIN [login_name];
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_enum_jobs TO [login_name];
GRANT EXECUTE ON master.dbo.xp_sqlagent_is_starting TO [login_name];
GRANT EXECUTE ON master.dbo.xp_sqlagent_notify TO [login_name];
```

## <a name="learn-more"></a>Más información

- [¿Qué es Instancia administrada de Azure SQL?](../managed-instance/sql-managed-instance-paas-overview.md)
- [Novedades de Azure SQL Database y SQL Managed Instance](../../azure-sql/database/doc-changes-updates-release-notes.md?tabs=managed-instance)
- [Diferencias entre T-SQL de Azure SQL Managed Instance y SQL Server](../../azure-sql/managed-instance/transact-sql-tsql-differences-sql-server.md#sql-server-agent)
- [Comparación de características: Azure SQL Database e Instancia administrada de Azure SQL](../../azure-sql/database/features-comparison.md)
