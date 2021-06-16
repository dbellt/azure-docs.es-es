---
title: Automatización de la replicación de los cambios de esquema en SQL Data Sync
description: Obtenga información sobre cómo automatizar la replicación de los cambios de esquema en Azure SQL Data Sync.
services: sql-database
ms.service: sql-database
ms.subservice: sql-data-sync
ms.custom: data sync
ms.devlang: ''
ms.topic: how-to
author: MaraSteiu
ms.author: masteiu
ms.reviewer: mathoma
ms.date: 11/14/2018
ms.openlocfilehash: b165850106d674e001349711c34f3f7479ab831f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110707657"
---
# <a name="automate-the-replication-of-schema-changes-in-azure-sql-data-sync"></a>Automatización de la replicación de los cambios de esquema en Azure SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

SQL Data Sync permite que los usuarios sincronicen datos entre bases de datos de Azure SQL Database e instancias de SQL Server en una dirección o en ambas. Una de las limitaciones actuales de SQL Data Sync es la falta de compatibilidad con la replicación de cambios de esquema. Cada vez que cambia el esquema de tabla, debe aplicar los cambios de manera manual en todos los puntos de conexión, incluida la central y todos los miembros, y luego actualizar el esquema de sincronización.

En este artículo se presenta una solución para replicar de manera automática los cambios de esquema en todos los puntos de conexión de SQL Data Sync.

1. Esta solución usa un desencadenador DDL para realizar el seguimiento de cambios de esquema.
1. El desencadenador inserta los comandos de cambios de esquema en una tabla de seguimiento.
1. Esta tabla de seguimiento se sincroniza con todos los puntos de conexión mediante el servicio Data Sync.
1. Después de la inserción, los desencadenadores DML se usan para aplicar los cambios de esquema en los otros puntos de conexión.

En este artículo se usa ALTER TABLE como ejemplo de un cambio de esquema, pero esta solución también sirve para otros tipos de cambios de esquema.

> [!IMPORTANT]
> Se recomienda leer este artículo con atención, especialmente las secciones [Solución de problemas](#troubleshoot) y [Otras consideraciones](#other), antes de empezar a implementar la replicación automatizada de cambios de esquema en el entorno de sincronización. También se recomienda leer [Sincronización de datos entre varias bases de datos locales y de la nube con SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md). Algunas operaciones de base de datos pueden interrumpir la solución que se describe en este artículo. Puede ser necesario tener conocimiento de dominio adicional de SQL Server y Transact-SQL para solucionar esos problemas.

![Automatización de la replicación de los cambios de esquema](./media/sql-data-sync-update-sync-schema/automate-schema-changes.png)

## <a name="set-up-automated-schema-change-replication"></a>Configuración de la replicación automatizada de cambios de esquema

### <a name="create-a-table-to-track-schema-changes"></a>Creación de una tabla para hacer seguimiento de los cambios de esquema

Cree una tabla para hacer seguimiento de los cambios de esquema en todas las bases de datos del grupo de sincronización:

```sql
CREATE TABLE SchemaChanges (
ID bigint IDENTITY(1,1) PRIMARY KEY,
SqlStmt nvarchar(max),
[Description] nvarchar(max)
)
```

Esta tabla tiene una columna de identidad para hacer seguimiento del orden de los cambios de esquema. Si es necesario, puede agregar más campos para registrar más información.

### <a name="create-a-table-to-track-the-history-of-schema-changes"></a>Creación de una tabla para hacer seguimiento del historial de los cambios de esquema

En todos los puntos de conexión, cree una tabla para hacer seguimiento del identificador del comando de cambios de esquema aplicado más recientemente.

```sql
CREATE TABLE SchemaChangeHistory (
LastAppliedId bigint PRIMARY KEY
)
GO

INSERT INTO SchemaChangeHistory VALUES (0)
```

### <a name="create-an-alter-table-ddl-trigger-in-the-database-where-schema-changes-are-made"></a>Creación de un desencadenador ALTER TABLE DDL en la base de datos cuando se realizan cambios de esquema

Cree un desencadenador DDL para las operaciones ALTER TABLE. Solo debe crear este desencadenador en la base de datos donde se hicieron los cambios de esquema. Para evitar conflictos, solo permita cambios de esquema en una base de datos en un grupo de sincronización.

```sql
CREATE TRIGGER AlterTableDDLTrigger
ON DATABASE
FOR ALTER_TABLE
AS

-- You can add your own logic to filter ALTER TABLE commands instead of replicating all of them.

IF NOT (EVENTDATA().value('(/EVENT_INSTANCE/SchemaName)[1]', 'nvarchar(512)') like 'DataSync')

INSERT INTO SchemaChanges (SqlStmt, Description)
    VALUES (EVENTDATA().value('(/EVENT_INSTANCE/TSQLCommand/CommandText)[1]', 'nvarchar(max)'), 'From DDL trigger')
```

El desencadenador inserta un registro en la tabla de seguimiento de cambios de esquema para cada comando ALTER TABLE. En este ejemplo se agrega un filtro para evitar replicar los cambios que se hicieron en el esquema **DataSync**, porque probablemente los hizo el servicio Data Sync. Agregue más filtros si solo quiere replicar ciertos tipos de cambios de esquema.

También puede agregar más desencadenadores para replicar otros tipos de cambios de esquema. Por ejemplo, cree los desencadenadores CREATE_PROCEDURE, ALTER_PROCEDURE y DROP_PROCEDURE para replicar los cambios de procedimientos almacenados.

### <a name="create-a-trigger-on-other-endpoints-to-apply-schema-changes-during-insertion"></a>Creación de un desencadenador en otros puntos de conexión para aplicar los cambios de esquema durante la inserción

Este desencadenador ejecuta el comando de cambios de esquema cuando se sincroniza con otros puntos de conexión. Debe crear este desencadenador en todos los puntos de conexión, excepto en el que se hacen los cambios de esquema (es decir, en la base de datos donde el desencadenador DDL `AlterTableDDLTrigger` se creó en el paso anterior).

```sql
CREATE TRIGGER SchemaChangesTrigger
ON SchemaChanges
AFTER INSERT
AS
DECLARE \@lastAppliedId bigint
DECLARE \@id bigint
DECLARE \@sqlStmt nvarchar(max)
SELECT TOP 1 \@lastAppliedId=LastAppliedId FROM SchemaChangeHistory
SELECT TOP 1 \@id = id, \@SqlStmt = SqlStmt FROM SchemaChanges WHERE id \> \@lastAppliedId ORDER BY id
IF (\@id = \@lastAppliedId + 1)
BEGIN
    EXEC sp_executesql \@SqlStmt
        UPDATE SchemaChangeHistory SET LastAppliedId = \@id
    WHILE (1 = 1)
    BEGIN
        SET \@id = \@id + 1
        IF exists (SELECT id FROM SchemaChanges WHERE ID = \@id)
            BEGIN
                SELECT \@sqlStmt = SqlStmt FROM SchemaChanges WHERE ID = \@id
                EXEC sp_executesql \@SqlStmt
                UPDATE SchemaChangeHistory SET LastAppliedId = \@id
            END
        ELSE
            BREAK;
    END
END
```

Este desencadenador se ejecuta después de la inserción y comprueba si el comando actual se debe ejecutar a continuación. La lógica de código garantiza que no se omita ninguna instrucción de cambios de esquema y que se apliquen todos los cambios incluso si la inserción no funciona.

### <a name="sync-the-schema-change-tracking-table-to-all-endpoints"></a>Sincronización de la tabla de seguimiento de cambios de esquema en todos los puntos de conexión

Puede sincronizar la tabla de seguimiento de cambios de esquema en todos los puntos de conexión mediante el grupo de sincronización existente o uno nuevo. Asegúrese de que todos los cambios de la tabla de seguimiento se pueden sincronizar en todos los puntos de conexión, sobre todo cuando usa la sincronización en una dirección.

No sincronice la tabla del historial de cambios de esquema, porque esa tabla conserva un valor distinto en los distintos puntos de conexión.

### <a name="apply-the-schema-changes-in-a-sync-group"></a>Aplicación de los cambios de esquema en un grupo de sincronización

Solo se replican los cambios de esquema que se hacen en la base de datos donde se creó el desencadenador DDL. No se replican los cambios de esquema hechos en otras bases de datos.

Una vez que los cambios de esquema se replican a todos los puntos de conexión, también debe realizar pasos adicionales para actualizar el esquema de sincronización a fin de empezar o detener la sincronización de las columnas nuevas.

#### <a name="add-new-columns"></a>Agregar columnas nuevas

1.  Realice el cambio de esquema.

1.  Evite cualquier cambio de datos en los que participen las columnas nuevas hasta que haya completado el paso que crea el desencadenador.

1.  Espere hasta que los cambios de esquema se apliquen a todos los puntos de conexión.

1.  Actualice el esquema de base de datos y agregue la columna nueva al esquema de sincronización.

1.  Los datos de la columna nueva se sincronizan durante la operación de sincronización siguiente.

#### <a name="remove-columns"></a>Quitar columnas

1.  Quite las columnas del esquema de sincronización. Data Sync detiene la sincronización de los datos de estas columnas.

1.  Realice el cambio de esquema.

1.  Actualice el esquema de la base de datos.

#### <a name="update-data-types"></a>Actualizar tipos de datos

1.  Realice el cambio de esquema.

1.  Espere hasta que los cambios de esquema se apliquen a todos los puntos de conexión.

1.  Actualice el esquema de la base de datos.

1.  Si los tipos de datos nuevos y anteriores no son totalmente compatibles (por ejemplo, si cambia de `int` a `bigint`), puede producirse un error de sincronización antes de completar los pasos que crean los desencadenadores. La sincronización se completa correctamente después de un reintento.

#### <a name="rename-columns-or-tables"></a>Cambiar el nombre de las columnas o tablas

Cambiar el nombre de las columnas o tablas hace que Data Sync deje de funcionar. Cree una tabla o columna nueva, reponga los datos y elimine la tabla o columna anterior en lugar de cambiar el nombre.

#### <a name="other-types-of-schema-changes"></a>Otros tipos de cambios de esquema

Para otros tipos de cambios de esquema (por ejemplo, crear procedimientos almacenados o colocar un índice), no es necesario actualizar el esquema de sincronización.

## <a name="troubleshoot-automated-schema-change-replication"></a><a name="troubleshoot"></a> Solución de problemas de la replicación automatizada de cambios de esquema

En este artículo se describe una lógica de replicación que deja de trabajar en ciertas situaciones, por ejemplo, si hace un cambio de esquema en una base de datos local no compatible con Azure SQL Database. En ese caso, no se puede sincronizar la tabla de seguimiento de cambios de esquema. Debe corregir manualmente este problema:

1.  Deshabilite el desencadenador DDL y evite cualquier cambio de esquema hasta que se corrija el problema.

1.  En la base de datos del punto de conexión donde se produce el problema, deshabilite el desencadenador AFTER INSERT en el punto de conexión donde no se puede hacer el cambio de esquema. Esta acción permite sincronizar el comando de cambios de esquema.

1.  Desencadene la sincronización para sincronizar la tabla de seguimiento de cambios de esquema.

1.  En la base de datos del punto de conexión donde se produce el problema, consulte la tabla del historial de cambios de esquema para obtener el identificador del último comando de cambios de esquema aplicado.

1.  Consulte la tabla de seguimiento de cambios de esquema para mostrar todos los comandos que tengan un identificador mayor que el valor del identificador que recuperó en el paso anterior.

    a.  Omita los comandos que no se pueden ejecutar en la base de datos del punto de conexión. Tiene que resolver la incoherencia del esquema. Revierta los cambios al esquema original si la incoherencia afecta la aplicación.

    b.  Aplique manualmente los comandos que se deben aplicar.

1.  Actualice la tabla del historial de cambios de esquema y establezca el último identificador aplicado en el valor correcto.

1.  Revise si el esquema está actualizado.

1.  Vuelva a habilitar el desencadenador AFTER INSERT que se deshabilitó en el segundo paso.

1.  Vuelva a habilitar el desencadenador DDL que se deshabilitó en el primer paso.

Si quiere limpiar los registros de la tabla de seguimiento de cambios de esquema, use DELETE en lugar de TRUNCATE. Nunca use DBCC CHECKIDENT para propagar la columna de identidad en la tabla de seguimiento de cambios de esquema. Si es necesario realizar una propagación, puede crear tablas de seguimiento de cambios de esquema nuevas y actualizar el nombre de la tabla en el desencadenador DDL.

## <a name="other-considerations"></a><a name="other"></a> Otras consideraciones

-   Los usuarios de la base de datos que configuran la central y las bases de datos miembro deben tener los permisos suficientes para ejecutar los comandos de cambios de esquema.

-   Puede agregar más filtros en el desencadenador DDL para replicar solo los cambios de esquema en operaciones o tablas seleccionadas.

-   Solo puede hacer cambios de esquema en la base de datos donde se creó el desencadenador DDL.

-   Si hace un cambio en una base de datos de SQL Server, asegúrese de que el cambio de esquema es compatible con Azure SQL Database.

-   Si se hacen cambios de esquema en bases de datos distintas de la base de datos donde se creó el desencadenador DDL, no se replican esos cambios. Para evitar este problema, puede crear desencadenadores DDL para bloquear los cambios en otros puntos de conexión.

-   Si tiene que cambiar el esquema de la tabla de seguimiento de cambios de esquema, deshabilite el desencadenador DDL antes de hacer el cambio y aplique manualmente el cambio en todos los puntos de conexión. No funciona actualizar el esquema de un desencadenador AFTER INSERT en la misma tabla.

-   No use DBCC CHECKIDENT para propagar la columna de identidad.

-   No use TRUNCATE para limpiar los datos en la tabla de seguimiento de cambios de esquema.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre SQL Data Sync, consulte:

-   Introducción: [Sincronización de datos entre varias bases de datos locales y en la nube con Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
-   Configuración de Data Sync
    - En el portal, [Tutorial: Configuración de SQL Data Sync para sincronizar datos entre Azure SQL Database y SQL Server](sql-data-sync-sql-server-configure.md)
    - Con PowerShell
        -  [Uso de PowerShell para sincronizar varias bases de datos de Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Uso de PowerShell para sincronizar una base de datos de Azure SQL Database y una base de datos de una instancia de SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Agente de sincronización de datos: [Agente de sincronización de datos para Azure SQL Data Sync](sql-data-sync-agent-overview.md)
-   Procedimientos recomendados: [Procedimientos recomendados para Azure SQL Data Sync](sql-data-sync-best-practices.md)
-   Supervisión: [Monitor SQL Data Sync with Azure Monitor logs](./monitor-tune-overview.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
-   Solución de problemas: [Solución de problemas de Azure SQL Data Sync]()
-   Actualización del esquema de sincronización
    -   Con PowerShell: [Usar PowerShell para actualizar el esquema de sincronización en un grupo de sincronización existente](scripts/update-sync-schema-in-sync-group.md)