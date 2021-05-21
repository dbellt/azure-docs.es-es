---
title: 'Copias de seguridad y restauración: instantáneas y redundancia geográfica'
description: Obtenga información acerca de cómo funcionan la copia de seguridad y la restauración en el grupo de SQL dedicado de Azure Synapse Analytics. Use copias de seguridad para restaurar el almacenamiento de datos a un punto de restauración en la región primaria. Use copias de seguridad con redundancia geográfica para restaurar en una región geográfica distinta.
services: synapse-analytics
author: joannapea
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/13/2020
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019"
ms.openlocfilehash: cb0cf3d84db507d3200f0285e95c2142e3288fbd
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784078"
---
# <a name="backup-and-restore-in-azure-synapse-dedicated-sql-pool"></a>Copia de seguridad y restauración en el grupo de SQL dedicado de Azure Synapse

Obtenga información sobre cómo usar la copia de seguridad y la restauración en el grupo de SQL dedicado de Azure Synapse. Use los puntos de restauración del grupo de SQL dedicado para devolver el almacenamiento de datos a un estado anterior o copiarlo en la región primaria. Utilice copias de seguridad con redundancia geográfica del almacenamiento de datos para restaurarlo en otra región geográfica.

## <a name="what-is-a-data-warehouse-snapshot"></a>Qué es una instantánea de almacenamiento de datos

Una *instantánea de almacenamiento de datos* crea un punto de restauración que se puede aprovechar para recuperar o copiar el almacenamiento de datos en un estado anterior.  Dado que el grupo de SQL dedicado es un sistema distribuido, una instantánea de almacenamiento de datos consta de muchos archivos que se almacenan en Azure Storage. Las instantáneas capturan los cambios incrementales de los datos almacenados en el almacenamiento de datos.

Una *restauración de almacenamiento de datos* es un nuevo almacenamiento de datos que se crea a partir de un punto de restauración de un almacenamiento de datos existente o eliminado. La restauración del almacenamiento de datos es una parte esencial de cualquier estrategia de recuperación ante desastres y continuidad empresarial, ya que vuelve a crear los datos tras daños o eliminaciones accidentales. El almacenamiento de datos es también un mecanismo eficaz para crear copias del almacenamiento de datos con fines de prueba o desarrollo. Las tasas de restauración del grupo de SQL dedicado pueden variar según el tamaño de la base de datos y la ubicación del almacenamiento de datos de origen y de destino.

## <a name="automatic-restore-points"></a>Puntos de restauración automáticos

Las instantáneas son una característica integrada que crea puntos de restauración. No es necesario habilitar esta funcionalidad. Sin embargo, el grupo de SQL dedicado debe estar en estado activo para la creación del punto de restauración. Si se pausa con frecuencia, es posible que no se creen puntos de restauración automáticos, por lo que debe asegurarse de crear un punto de restauración definido por el usuario antes de pausar el grupo de SQL dedicado. Actualmente, los usuarios no pueden eliminar los puntos de restauración automática mientras el servicio utiliza estos puntos de restauración para mantener los contratos de nivel de servicio para la recuperación.

Las instantáneas del almacenamiento de datos se toman a lo largo del día y crean puntos de restauración que están disponibles durante siete días. No se puede cambiar este período de retención. El grupo de SQL dedicado admite un objetivo de punto de recuperación (RPO) de ocho horas. Puede restaurar el almacenamiento de datos de la región primaria a partir de cualquiera de las instantáneas capturadas en los últimos siete días.

Para ver cuándo se inició la última instantánea, ejecute esta consulta en el grupo de SQL dedicado en línea.

```sql
select   top 1 *
from     sys.pdw_loader_backup_runs
order by run_id desc
;
```

## <a name="user-defined-restore-points"></a>Puntos de restauración definidos por el usuario

Esta característica permite desencadenar instantáneas manualmente para crear puntos de restauración del almacenamiento de datos antes y después de realizar grandes modificaciones. Esta funcionalidad garantiza que los puntos de restauración sean lógicamente coherentes, lo que proporciona una protección de datos adicional en caso de interrupciones de la carga de trabajo o de errores del usuario para un tiempo de recuperación rápido. Los puntos de restauración definidos por el usuario están disponibles durante siete días y se eliminan automáticamente. No se puede cambiar el período de retención de los puntos de restauración definidos por el usuario. Se garantizan **42 puntos de restauración definidos por el usuario** en un momento dado, por lo que deben [eliminarse](/powershell/module/azurerm.sql/remove-azurermsqldatabaserestorepoint) antes de crear otro punto de restauración. Puede desencadenar instantáneas para crear puntos de restauración definidos por el usuario mediante [PowerShell](/powershell/module/az.sql/new-azsqldatabaserestorepoint?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.jsont#examples) o Azure Portal.

> [!NOTE]
> Si necesita más de siete días de puntos de restauración, vote por esta funcionalidad [aquí](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/35114410-user-defined-retention-periods-for-restore-points). También puede crear un punto de restauración definido por el usuario y restaurar desde el punto de restauración recién creado en un nuevo almacenamiento de datos. Cuando haya realizado la restauración, tendrá el grupo de SQL dedicado en línea y podrá pausarlo indefinidamente para reducir los costos de proceso. La base de datos en pausa genera gastos de almacenamiento según la tarifa de almacenamiento de Azure Synapse. Si necesita una copia activa del almacenamiento de datos restaurado, puede reanudarlo, lo que sólo le llevará unos minutos.

### <a name="restore-point-retention"></a>Retención de punto de recuperación

A continuación se indican los detalles sobre los períodos de retención de punto de restauración:

1. El grupo de SQL dedicado elimina un punto de restauración cuando alcanza el período de retención de 7 días **y** cuando hay al menos 42 puntos de restauración totales (incluidos los definidos por el usuario y los automáticos).
2. Las instantáneas no se toman cuando un grupo de SQL dedicado está en pausa.
3. La antigüedad de un punto de restauración se mide por los días naturales absolutos a partir del momento en que se toma el punto de restauración, incluso cuando el grupo de SQL está en pausa.
4. En cualquier momento, se garantiza que un grupo de SQL dedicado podrá almacenar hasta 42 puntos de restauración definidos por el usuario o 42 puntos de restauración automáticos siempre y cuando estos puntos de restauración no hayan alcanzado el período de retención de 7 días.
5. Si se toma una instantánea, el grupo de SQL dedicado se pausa durante más de 7 días y luego se reanuda, el punto de restauración persistirá hasta que haya un total de 42 puntos de restauración (incluidos los definidos por el usuario y los automáticos).

### <a name="snapshot-retention-when-a-sql-pool-is-dropped"></a>Retención de instantáneas cuando se quita un grupo de SQL

Cuando se quita un grupo de SQL dedicado, se crea una instantánea final que se guarda durante siete días. Puede restaurar el grupo de SQL dedicado al punto de restauración final creado durante la eliminación. Si el grupo de SQL dedicado se quita en estado en pausa, no se toma ninguna instantánea. En ese escenario, asegúrese de crear un punto de restauración definido por el usuario antes de quitar el grupo de SQL dedicado.

> [!IMPORTANT]
> Si elimina el servidor o el espacio de trabajo que hospeda un grupo de SQL dedicado, todas las bases de datos que pertenecen al servidor o el espacio de trabajo también se eliminan y no se pueden recuperar. No puede restaurar un servidor eliminado.

## <a name="geo-backups-and-disaster-recovery"></a>Copias de seguridad geográficas y recuperación ante desastres

Se crea una copia de seguridad de replicación geográfica una vez al día en un [centro de datos emparejado](../../best-practices-availability-paired-regions.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). El RPO para una restauración geográfica es de 24 horas. Puede restaurar la copia de seguridad de replicación geográfica en un servidor de cualquier otra región donde se admita el grupo de SQL dedicado. Una copia de seguridad geográfica garantiza que pueda restaurar el almacenamiento de datos en caso de que no tenga acceso a los puntos de restauración de su región primaria.

Si no necesita copias de seguridad geográficas para su grupo de SQL dedicado, puede deshabilitarlas y ahorrar costos de almacenamiento de recuperación ante desastres. Para ello, consulte [Guía de procedimientos: Deshabilitación de copias de seguridad geográficas para un grupo de SQL dedicado (anteriormente SQL DW)](disable-geo-backup.md). Tenga en cuenta que si deshabilita las copias de seguridad geográficas, no podrá recuperar el grupo de SQL dedicado en la región de Azure emparejada si el centro de datos principal de Azure no está disponible. 

> [!NOTE]
> Si necesita un objetivo de punto de recuperación más reducido para copias de seguridad de replicación geográfica, vote por esta funcionalidad [aquí](https://feedback.azure.com/forums/307516-sql-data-warehouse). También puede crear un punto de restauración definido por el usuario y restaurar a partir del punto de restauración recién creado en un nuevo almacenamiento de datos. Cuando haya realizado la restauración, tendrá el almacenamiento de datos en línea y podrá pausarlo indefinidamente para ahorrar costos de proceso. La base de datos en pausa genera gastos de almacenamiento según la tarifa de Azure Premium Storage. Si necesita una copia activa del almacenamiento de datos, puede reanudarlo, lo que solo le llevará unos minutos.

## <a name="data-residency"></a>Residencia de datos 

Si el centro de datos emparejado se encuentra fuera de sus límites geográficos, puede asegurarse de que los datos permanecen dentro de sus límites geográficos deshabilitando el almacenamiento con redundancia geográfica. Esto puede hacerse al aprovisionar el grupo de SQL dedicado (anteriormente SQL DW) a través de la opción de almacenamiento con redundancia geográfica cuando se crea o restaura un grupo de SQL dedicado (anteriormente SQL DW). 

Para confirmar que el centro de datos emparejado se encuentra en un país diferente, consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="backup-and-restore-costs"></a>Costos de copia de seguridad y restauración

Observará que la factura de Azure tiene un elemento de línea para Storage y un elemento de línea para Storage de recuperación ante desastres. El cargo de almacenamiento corresponde al costo total del almacenamiento de sus datos en la región primaria con los cambios incrementales que capturan las instantáneas. Para obtener una explicación más detallada de cómo se cobran las instantáneas, consulte [Understanding how Snapshots Accrue Charges](/rest/api/storageservices/Understanding-How-Snapshots-Accrue-Charges?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) (Cómo se acumulan los cargos de instantáneas). El cargo con redundancia geográfica abarca el costo de almacenamiento de las copias de seguridad geográficas.  

El costo total del almacenamiento de datos principal y de los siete días de cambios de instantánea se redondea al TB más cercano. Por ejemplo, si el almacenamiento de datos es de 1,5 TB y las capturas de instantáneas 100 GB, se le facturan 2 TB de datos según las tarifas de Azure Premium Storage.

Si usa almacenamiento con redundancia geográfica, recibirá un cargo de almacenamiento por separado. El almacenamiento con redundancia geográfica se factura según la tarifa estándar de almacenamiento geográficamente redundante con acceso de lectura (RA-GRS).

Para más información sobre los precios de Azure Synapse, consulte [Precios de Azure Synapse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2/). La salida de datos no se cobra al restaurar entre regiones.

## <a name="restoring-from-restore-points"></a>Restauración a partir de puntos de restauración

Cada instantánea crea un punto de restauración que representa la hora de inicio de la instantánea. Para restaurar un almacenamiento de datos, elija un punto de restauración y emita un comando de restauración.  

Puede mantener el almacenamiento de datos restaurado y el actual, o eliminar uno de ellos. Si quiere reemplazar el almacenamiento de datos actual por el restaurado, puede cambiarle el nombre mediante [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) con la opción MODIFY NAME.

Para restaurar un almacenamiento de datos, consulte [Restauración de un grupo de SQL dedicado](sql-data-warehouse-restore-points.md#create-user-defined-restore-points-through-the-azure-portal).

Para restaurar un almacenamiento de datos eliminado o en pausa, puede [crear una incidencia de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="cross-subscription-restore"></a>Restauración entre suscripciones

Si necesita restaurar directamente entre suscripciones, vote por esta funcionalidad [aquí](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/36256231-enable-support-for-cross-subscription-restore). Realice la restauración en otro servidor y ["mueva"](../../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) el servidor entre suscripciones para realizar una restauración entre suscripciones.

## <a name="geo-redundant-restore"></a>Restauración con redundancia geográfica

Puede [restaurar el grupo de SQL dedicado](sql-data-warehouse-restore-from-geo-backup.md#restore-from-an-azure-geographical-region-through-powershell) en cualquier región que admita el grupo de SQL dedicado en el nivel de rendimiento elegido.

> [!NOTE]
> Para llevar a cabo una restauración con redundancia geográfica no puede haber anulado esta característica.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los puntos de restauración, consulte [Puntos de restauración definidos por el usuario](sql-data-warehouse-restore-points.md)
