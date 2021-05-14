---
title: Actualización a la última generación de un grupo de SQL dedicado (anteriormente SQL DW)
description: Actualice un grupo de SQL dedicado (anteriormente SQL DW) de Azure Synapse Analytics a la última generación de arquitectura de almacenamiento y hardware de Azure.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/19/2019
ms.author: martinle
ms.reviewer: jrasnick
ms.custom: seo-lt-2019
ms.openlocfilehash: 12ac49154fabbc0ce96390d5fffa5d02d7886324
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166008"
---
# <a name="optimize-performance-by-upgrading-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Optimización del rendimiento mediante la actualización del grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics

Actualice el grupo de SQL dedicado (anteriormente SQL DW) a la última generación de arquitectura de almacenamiento y hardware de Azure.

## <a name="why-upgrade"></a>¿Por qué actualizar?

Ahora puede actualizar sin problemas al grupo de SQL dedicado (anteriormente SQL DW) de nivel Gen2 optimizado para proceso en Azure Portal para las [regiones compatibles](gen2-migration-schedule.md#automated-schedule-and-region-availability-table). Si su región no admite la actualización automática, puede actualizar a una región admitida o esperar a que la actualización automática esté disponible en su región. Actualice ahora para beneficiarse de la última generación de hardware de Azure y de la arquitectura de almacenamiento mejorada, como un rendimiento más rápido, mayor escalabilidad y un almacenamiento en columnas ilimitado.

> [!VIDEO https://www.youtube.com/embed/9B2F0gLoyss]

> [!IMPORTANT]
> Esta actualización se aplica a los grupos de SQL dedicados (anteriormente SQL DW) de nivel Gen1 optimizados para proceso en [regiones compatibles](gen2-migration-schedule.md#automated-schedule-and-region-availability-table).

## <a name="before-you-begin"></a>Antes de empezar

1. Compruebe si su [región](gen2-migration-schedule.md#automated-schedule-and-region-availability-table) es compatible para la migración de GEN1 a GEN2. Tenga en cuenta las fechas de la migración automática. Para evitar conflictos con el proceso automatizado, planee la migración manual antes de la fecha de inicio del proceso automatizado.
2. Si está en una región que aún no es compatible, continúe con la comprobación de la región que se va a agregar o [actualice mediante restauración](#upgrade-from-an-azure-geographical-region-using-restore-through-the-azure-portal) a una región compatible.
3. Si la región es compatible, [realice la actualización en Azure Portal](#upgrade-in-a-supported-region-using-the-azure-portal).
4. **Seleccione el nivel de rendimiento sugerido** para el grupo de SQL dedicado (anteriormente SQL DW) en función del nivel de rendimiento actual en el nivel Gen1 optimizado para proceso mediante la asignación siguiente:

   | Nivel Gen1 optimizado para proceso | Nivel Gen2 optimizado para proceso |
   | :-------------------------: | :-------------------------: |
   |            DW100            |           DW100c            |
   |            DW200            |           DW200c            |
   |            DW300            |           DW300c            |
   |            DW400            |           DW400c            |
   |            DW500            |           DW500c            |
   |            DW600            |           DW500c            |
   |           DW1000            |           DW1000c           |
   |           DW1200            |           DW1000c           |
   |           DW1500            |           DW1500c           |
   |           DW2000            |           DW2000c           |
   |           DW3000            |           DW3000c           |
   |           DW6000            |           DW6000c           |

> [!NOTE]
> Los niveles de rendimiento sugeridos no son una conversión directa. Por ejemplo, se recomienda pasar de DW600 a DW500c.

## <a name="upgrade-in-a-supported-region-using-the-azure-portal"></a>Actualización en una región compatible mediante Azure Portal

- La migración de Gen1 a Gen2 mediante Azure Portal es permanente. No hay ningún proceso para volver a Gen1.
- El grupo de SQL dedicado (anteriormente SQL DW) debe estar en ejecución para migrar a Gen2.

### <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Inicie sesión en [Azure Portal](https://portal.azure.com/).
- Compruebe que se está ejecutando el grupo de SQL dedicado (anteriormente SQL DW) para poder migrar a Gen2.

### <a name="powershell-upgrade-commands"></a>Comandos de actualización de PowerShell

1. Si el grupo de SQL dedicado (anteriormente SQL DW) de nivel Gen1 optimizado para proceso que se va a actualizar se ha pausado, [reanude el grupo de SQL dedicado (anteriormente SQL DW)](pause-and-resume-compute-portal.md).

2. Prepárese para unos minutos de inactividad.

3. Identifique todas las referencias de código para los niveles de rendimiento Optimizado para Compute Gen1 y modifíquelos a su nivel de rendimiento equivalente Optimizado para Compute Gen2. A continuación, se muestran dos ejemplos de dónde debe actualizar las referencias de código antes de actualizar:

   Comando de PowerShell Gen1 original:

   ```powershell
   Set-AzSqlDatabase -ResourceGroupName "myResourceGroup" -DatabaseName "mySampleDataWarehouse" -ServerName "mynewserver-20171113" -RequestedServiceObjectiveName "DW300"
   ```

   Modificado a:

   ```powershell
   Set-AzSqlDatabase -ResourceGroupName "myResourceGroup" -DatabaseName "mySampleDataWarehouse" -ServerName "mynewserver-20171113" -RequestedServiceObjectiveName "DW300c"
   ```

   > [!NOTE]
   > -RequestedServiceObjectiveName "DW300" cambiado a RequestedServiceObjectiveName "DW300 **c**"
   >

   Comando de T-SQL Gen1 original:

   ```sql
   ALTER DATABASE mySampleDataWarehouse MODIFY (SERVICE_OBJECTIVE = 'DW300') ;
   ```

   Modificado a:

   ```sql
   ALTER DATABASE mySampleDataWarehouse MODIFY (SERVICE_OBJECTIVE = 'DW300c') ;
   ```

   > [!NOTE]
   > SERVICE_OBJECTIVE = 'DW300' cambiado a SERVICE_OBJECTIVE = 'DW300 **c**'

## <a name="start-the-upgrade"></a>Iniciar la actualización

1. Vaya al grupo de SQL dedicado (anteriormente SQL DW) de nivel Gen1 optimizado para proceso en Azure Portal. Si el grupo de SQL dedicado (anteriormente SQL DW) de nivel Gen1 optimizado para proceso que se va a actualizar se ha pausado, [reanude el grupo de SQL dedicado](pause-and-resume-compute-portal.md).
2. Seleccione la tarjeta **Actualizar a Gen2** en la pestaña Tareas: ![Upgrade_1](./media/upgrade-to-latest-generation/upgrade-to-gen2-1.png)

   > [!NOTE]
   > Si no aparece la tarjeta **Actualizar a Gen2** en la pestaña Tareas, su suscripción es de un tipo que tiene limitaciones en la región actual.
   > [Envíe una incidencia de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md) para que se apruebe la suscripción.

3. Asegúrese de que la carga de trabajo ha terminado de ejecutarse y está en modo inactivo antes de actualizar. Experimentará un tiempo de inactividad durante unos minutos antes de que el grupo de SQL dedicado (anteriormente SQL DW) vuelva a estar en línea como grupo de SQL dedicado (anteriormente SQL DW) de nivel Gen2 optimizado para proceso. **Seleccione Actualizar**:

   ![Upgrade_2](./media/upgrade-to-latest-generation/upgrade-to-gen2-2.png)

4. Para **supervisar la actualización**, compruebe el estado en Azure Portal:

   ![Upgrade3](./media/upgrade-to-latest-generation/upgrade-to-gen2-3.png)

   El primer paso del proceso de actualización recorre la operación de escalado ("Actualización: sin conexión") donde todas las sesiones se terminan y las conexiones se descartan.

   El segundo paso del proceso de actualización es la migración de datos ("Actualización: en línea"). La migración de datos es un proceso en segundo plano lento y en línea. Durante este proceso, los datos en columnas se mueven lentamente desde la arquitectura de almacenamiento anterior hasta la nueva arquitectura de almacenamiento mediante una caché de SSD local. Durante este tiempo, el grupo de SQL dedicado (anteriormente SQL DW) estará en línea para consulta y carga. Los datos estarán disponibles para consulta tanto si se han migrado como si no. La migración de datos se produce a una velocidad variable, según el tamaño de los datos, el nivel de rendimiento y el número de segmentos del almacén de columnas.

5. **Recomendación opcional:** Una vez completada la operación de escalado, puede acelerar el proceso en segundo plano de migración de datos. Puede forzar inmediatamente el movimiento de datos ejecutando [Alter Index rebuild](sql-data-warehouse-tables-index.md) en todas las tablas de almacén de columnas principales que consulte con una clase de recurso y un SLO de mayor tamaño. Esta operación es **sin conexión** en comparación con el proceso en segundo plano lento, que puede tardar horas en completarse según el número y tamaño de las tablas. Sin embargo, una vez que haya finalizado, la migración de datos será mucho más rápida debido a la nueva arquitectura de almacenamiento mejorada con grupos de filas de alta calidad.

> [!NOTE]
> Alter Index rebuild es una operación sin conexión y las tablas no estarán disponibles hasta que se complete la recompilación.

La consulta siguiente genera los comandos Alter Index Rebuild necesarios para acelerar la migración de datos:

```sql
SELECT 'ALTER INDEX [' + idx.NAME + '] ON ['
       + Schema_name(tbl.schema_id) + '].['
       + Object_name(idx.object_id) + '] REBUILD ' + ( CASE
                                                         WHEN (
                                                     (SELECT Count(*)
                                                      FROM   sys.partitions
                                                             part2
                                                      WHERE  part2.index_id
                                                             = idx.index_id
                                                             AND
                                                     idx.object_id =
                                                     part2.object_id)
                                                     > 1 ) THEN
              ' PARTITION = '
              + Cast(part.partition_number AS NVARCHAR(256))
              ELSE ''
                                                       END ) + '; SELECT ''[' +
              idx.NAME + '] ON [' + Schema_name(tbl.schema_id) + '].[' +
              Object_name(idx.object_id) + '] ' + (
              CASE
                WHEN ( (SELECT Count(*)
                        FROM   sys.partitions
                               part2
                        WHERE
                     part2.index_id =
                     idx.index_id
                     AND idx.object_id
                         = part2.object_id) > 1 ) THEN
              ' PARTITION = '
              + Cast(part.partition_number AS NVARCHAR(256))
              + ' completed'';'
              ELSE ' completed'';'
                                                    END )
FROM   sys.indexes idx
       INNER JOIN sys.tables tbl
               ON idx.object_id = tbl.object_id
       LEFT OUTER JOIN sys.partitions part
                    ON idx.index_id = part.index_id
                       AND idx.object_id = part.object_id
WHERE  idx.type_desc = 'CLUSTERED COLUMNSTORE';
```

## <a name="upgrade-from-an-azure-geographical-region-using-restore-through-the-azure-portal"></a>Actualización desde una región geográfica de Azure mediante restauración en Azure Portal

## <a name="create-a-user-defined-restore-point-using-the-azure-portal"></a>Uso de Azure Portal para crear un punto de restauración definido por el usuario

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Vaya al grupo de SQL dedicado (anteriormente SQL DW) para el que quiere crear un punto de restauración.

3. En la parte superior de la sección de información general, seleccione **+ Nuevo punto de restauración**.

    ![Nuevo punto de restauración](./media/upgrade-to-latest-generation/creating_restore_point_0.png)

4. Especifique un nombre para el punto de restauración.

    ![Nombre de punto de restauración](./media/upgrade-to-latest-generation/creating_restore_point_1.png)

## <a name="restore-an-active-or-paused-database-using-the-azure-portal"></a>Uso de Azure Portal para restaurar una base de datos activa o en pausa

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya al grupo de SQL dedicado (anteriormente SQL DW) desde el que desea restaurar.
3. En la parte superior de la sección de información general, seleccione **Restaurar**.

    ![ Introducción a la restauración](./media/upgrade-to-latest-generation/restoring_0.png)

4. Seleccione **Puntos de restauración automáticos** o **Puntos de restauración definidos por el usuario**. En el caso de los puntos de restauración definidos por el usuario, **seleccione un punto de restauración definido por el usuario** o **cree un punto de restauración definido por el usuario**. En el servidor, seleccione **Crear nuevo** y elija un servidor en una región geográfica compatible con Gen2.

    ![Puntos de restauración automáticos](./media/upgrade-to-latest-generation/restoring_1.png)

## <a name="restore-from-an-azure-geographical-region-using-powershell"></a>Uso de PowerShell para restaurar desde una región geográfica de Azure

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para recuperar una base de datos, use el cmdlet [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

> [!NOTE]
> Puede realizar una restauración geográfica en Gen2. Para ello, especifique Gen2 ServiceObjectiveName (p. ej., DW1000 **c**) como parámetro opcional.

1. Abra Windows PowerShell.
2. Conéctese a su cuenta de Azure y enumere todas las suscripciones asociadas a su cuenta.
3. Seleccione la suscripción que contiene la base de datos que se va a restaurar.
4. Obtenga la base de datos que desea recuperar.
5. Cree la solicitud de recuperación para la base de datos, especificando un atributo ServiceObjectiveName para Gen2.
6. Compruebe el estado de la base de datos restaurada geográficamente.

```powershell
Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName "<Subscription_name>"

# Get the database you want to recover
$GeoBackup = Get-AzSqlDatabaseGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

# Recover database
$GeoRestoredDatabase = Restore-AzSqlDatabase –FromGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourTargetServer>" -TargetDatabaseName "<NewDatabaseName>" –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName "<YourTargetServiceLevel>" -RequestedServiceObjectiveName "DW300c"

# Verify that the geo-restored database is online
$GeoRestoredDatabase.status
```

> [!NOTE]
> Para configurar la base de datos una vez finalizada la restauración, consulte [Configuración de la base de datos después de realizar la recuperación](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

La base de datos recuperada estará habilitada para TDE si la base de datos de origen está habilitada para TDE.

Si experimenta problemas con el grupo de SQL dedicado, cree una [solicitud de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md) e indique que la posible causa es la "actualización a Gen2".

## <a name="next-steps"></a>Pasos siguientes

El grupo de SQL dedicado (anteriormente SQL DW) actualizado está en línea. Para aprovechar las ventajas de la arquitectura mejorada, consulte [Clases de recursos para la administración de cargas de trabajo](resource-classes-for-workload-management.md).
