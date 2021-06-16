---
title: Lectura de consultas en réplicas
description: Azure SQL permite usar la capacidad de las réplicas de solo lectura para cargas de trabajo de lectura. Esto se denomina escalado horizontal de lectura.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: conceptual
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma
ms.date: 01/20/2021
ms.openlocfilehash: a6f9debd9b4daec96fdb2f00f81bcbeb5073e4c7
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110691723"
---
# <a name="use-read-only-replicas-to-offload-read-only-query-workloads"></a>Uso de réplicas de solo lectura para descargar cargas de trabajo de consulta de solo lectura
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Como parte de la [arquitectura de alta disponibilidad](high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability), cada base de datos única, base de datos de grupos elásticos e instancia administrada del nivel de servicio Premium y Crítico para la empresa se aprovisiona automáticamente con una réplica principal de lectura y escritura y varias réplicas secundarias de solo lectura. Las réplicas secundarias se aprovisionan con el mismo tamaño de proceso que la réplica principal. La característica *Escalado horizontal de lectura* le permite descargar las cargas de trabajo de solo lectura usando la capacidad de proceso de una de las réplicas de solo lectura, en lugar de ejecutarlas en la réplica de lectura y escritura. De este modo, algunas cargas de trabajo de solo lectura se pueden aislar de las cargas de trabajo principales de lectura y escritura sin que su rendimiento se vea afectado. La característica está diseñada para las aplicaciones que contienen cargas de solo lectura separadas de forma lógica; por ejemplo, análisis. En los niveles de servicio Premium y Crítico para la empresa, las aplicaciones podrían obtener ventajas de rendimiento gracias a esta capacidad sin costo adicional.

La característica *Escalado horizontal de lectura* también está disponible en el nivel de servicio Hiperescala cuando se crea al menos una réplica secundaria. Se pueden usar varias réplicas secundarias para el equilibrio de carga de las cargas de trabajo de solo lectura que requieren más recursos de los disponibles en una réplica secundaria.

La arquitectura de alta disponibilidad de los niveles de servicio Básico, Estándar y De uso general no incluye réplicas. La característica *Escalado horizontal de lectura* no está disponible en estos niveles de servicio.

En el siguiente diagrama se ilustra esta característica.

![Réplicas de solo lectura](./media/read-scale-out/business-critical-service-tier-read-scale-out.png)

La característica *Escalado horizontal de lectura* está habilitada de forma predeterminada en las nuevas bases de datos de nivel Premium, Crítico para la empresa e Hiperescala. En el caso de Hiperescala, se crea una réplica secundaria de forma predeterminada para las bases de datos nuevas. 

> [!NOTE]
> El escalado horizontal de lectura siempre está habilitado en el nivel de servicio Crítico para la empresa de la Instancia administrada.

Si la cadena de conexión SQL está configurada con `ApplicationIntent=ReadOnly`, la aplicación se redirigirá a una réplica de solo lectura de esa base de datos o instancia administrada. Para más información sobre la propiedad `ApplicationIntent`, consulte [Especificar el intento de la aplicación](/sql/relational-databases/native-client/features/sql-server-native-client-support-for-high-availability-disaster-recovery#specifying-application-intent).

Si desea asegurarse de que la aplicación se conecta a la réplica principal sin tener en cuenta el valor `ApplicationIntent` de la cadena de conexión SQL, debe deshabilitar explícitamente el escalado horizontal de lectura cuando cree la base de datos o modifique su configuración. Por ejemplo, si actualiza una base de datos de nivel Estándar o De uso General al nivel Premium, Crítico para la empresa o Hiperescala y desea asegurarse de que todas las conexiones siguen estableciéndose con la réplica principal, deshabilite el escalado horizontal de lectura. Para más información acerca de cómo deshabilitarlo, consulte [Habilitar y deshabilitar el escalado horizontal de lectura](#enable-and-disable-read-scale-out).

> [!NOTE]
> Las características Almacén de consultas y SQL Profiler no son compatibles con las réplicas de solo lectura. 

## <a name="data-consistency"></a>Coherencia de datos

Una de las ventajas de las réplicas es que siempre se encuentran en estado de coherencia transaccional, pero es posible que en diferentes momentos pueda haber una pequeña latencia entre las distintas réplicas. El escalado horizontal de lectura admite la coherencia en el nivel de sesión. Esto significa que, si la sesión de solo lectura vuelve a conectarse después de un error de conexión provocado por la falta de disponibilidad de las réplicas, podría redirigirse a una réplica que no tuviera toda la información de la réplica de lectura y escritura. Del mismo modo, si una aplicación escribe datos en una sesión de lectura y escritura y los lee inmediatamente en una sesión de solo lectura, es posible que las actualizaciones más recientes no puedan verse inmediatamente en la réplica. La latencia se produce por una operación asincrónica que rehace el registro de transacciones.

> [!NOTE]
> Las latencias de replicación en la región son bajas y esta situación es poco frecuente. Para supervisar la latencia de replicación, consulte [Supervisión y solución de problemas de la réplica de solo lectura](#monitoring-and-troubleshooting-read-only-replicas).

## <a name="connect-to-a-read-only-replica"></a>Conexión a una réplica de solo lectura

Cuando se habilita el escalado horizontal de lectura en una base de datos, la opción `ApplicationIntent` de la cadena de conexión proporcionada por el cliente dictamina si la conexión se enruta a la réplica de escritura o a una réplica de solo lectura. Específicamente, si el valor de `ApplicationIntent` es `ReadWrite` (valor predeterminado), la conexión se dirigirá a la réplica de lectura y escritura. Es idéntico al comportamiento cuando `ApplicationIntent` no está incluido en la cadena de conexión. Si el valor de `ApplicationIntent` es `ReadOnly`, la conexión se enruta a una réplica de solo lectura.

Por ejemplo, la siguiente cadena de conexión conecta el cliente a una réplica de solo lectura (los elementos entre corchetes angulares se sustituyen por los valores correctos para su entorno y se quitan dichos corchetes):

```sql
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

Cualquiera de las siguientes cadenas de conexión conecta el cliente a una réplica de lectura-escritura (los elementos entre corchetes angulares se sustituyen por los valores correctos para su entorno y se quitan dichos corchetes):

```sql
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadWrite;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;

Server=tcp:<server>.database.windows.net;Database=<mydatabase>;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

## <a name="verify-that-a-connection-is-to-a-read-only-replica"></a>Verificación de que se trata de una conexión a una réplica de solo lectura

Puede comprobar si está conectado a una réplica de solo lectura mediante la ejecución de la consulta siguiente en el contexto de la base de datos. Si está conectado a una réplica de solo lectura, se devolverá READ_ONLY.

```sql
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability');
```

> [!NOTE]
> En los niveles de servicio Premium y Crítico para la empresa, solo se puede acceder a una de las réplicas de solo lectura en un momento dado. La Hiperescala admite varias réplicas de solo lectura.

## <a name="monitoring-and-troubleshooting-read-only-replicas"></a>Supervisión y solución de problemas con las réplicas de solo lectura

Cuando se conecta a una réplica de solo lectura, las vistas de administración dinámica (DMV) reflejan el estado de la réplica y se pueden consultar con fines de supervisión y solución de problemas. El motor de base de datos proporciona varias vistas para exponer una amplia variedad de datos de supervisión. 

Las vistas que se usan con frecuencia son:

| Nombre | Propósito |
|:---|:---|
|[sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Proporciona métricas de uso de recursos durante la última hora, incluida la CPU, la E/S de datos y el uso de la escritura de registros en relación con los límites de los objetivos de servicio.|
|[sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql)| Proporciona estadísticas de espera agregadas para la instancia del motor de base de datos. |
|[sys.dm_database_replica_states](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-replica-states-azure-sql-database)| Proporciona estadísticas de sincronización y estado de mantenimiento de la réplica. La tasa y el tamaño de la cola de la fase de puesta al día sirven como indicadores de latencia de datos en la réplica de solo lectura. |
|[sys.dm_os_performance_counters](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-performance-counters-transact-sql)| Proporciona contadores de rendimiento del motor de base de datos.|
|[sys.dm_exec_query_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql)| Proporciona estadísticas de ejecución por consulta, como el número de ejecuciones, el tiempo de CPU usado, etc.|
|[sys.dm_exec_query_plan()](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-transact-sql)| Proporciona planes de consulta almacenados en caché. |
|[sys.dm_exec_sql_text()](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sql-text-transact-sql)| Proporciona texto de consulta para un plan de consulta almacenado en caché.|
|[sys.dm_exec_query_profiles](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql)| Proporciona el progreso de la consulta en tiempo real mientras las consultas están en ejecución.|
|[sys.dm_exec_query_plan_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql)| Proporciona el último plan de ejecución real conocido, incluidas las estadísticas de tiempo de ejecución de una consulta.|
|[sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql)| Proporciona las IOPS de almacenamiento, el rendimiento y las estadísticas de latencia para todos los archivos de base de datos. |

> [!NOTE]
> Las DMV `sys.resource_stats` y `sys.elastic_pool_resource_stats` de la base de datos lógica maestra devuelven información sobre uso de los recursos en la réplica principal.

### <a name="monitoring-read-only-replicas-with-extended-events"></a>Supervisar réplicas de solo lectura con eventos extendidos

No se puede crear una sesión de eventos extendidos si está conectado a una réplica de solo lectura. Sin embargo, en Azure SQL Database, las definiciones de las sesiones de [eventos extendidos](xevent-db-diff-from-svr.md) creadas y alteradas en la réplica principal se replican en réplicas de solo lectura, incluidas las réplicas geográficas, y capturan eventos en réplicas de solo lectura.

Una sesión de eventos extendidos en una réplica de solo lectura que se basa en una definición de sesión de la réplica principal se puede iniciar y detener independientemente de la réplica principal. Cuando se quita una sesión de eventos extendidos en la réplica principal, también se quita en todas las réplicas de solo lectura.

### <a name="transaction-isolation-level-on-read-only-replicas"></a>Nivel de aislamiento de transacción en réplicas de solo lectura

Las consultas que se ejecutan en réplicas de solo lectura siempre se asignan al nivel de aislamiento de transacción de [instantáneas](/dotnet/framework/data/adonet/sql/snapshot-isolation-in-sql-server). El aislamiento de instantáneas usa las versiones de fila para evitar situaciones de bloqueo en que los lectores bloquean a los escritores.

En raras ocasiones, si una transacción de aislamiento de instantánea accede a los metadatos de objeto que se han modificad en otra transacción simultánea, puede recibir el error [3961](/sql/relational-databases/errors-events/mssqlserver-3961-database-engine-error): "Error de la transacción de aislamiento de instantánea en la base de datos "%.*ls" porque el objeto al que tuvo acceso la instrucción se modificó mediante una instrucción DDL de otra transacción simultánea desde el inicio de esta transacción. Está deshabilitada porque los metadatos no tienen versiones. Una actualización simultánea de los metadatos puede dar lugar a incoherencias si se combina con el aislamiento de la instantánea."

### <a name="long-running-queries-on-read-only-replicas"></a>Consultas de ejecución prolongada en réplicas de solo lectura

Las consultas que se ejecutan en réplicas de solo lectura necesitan tener acceso a los metadatos para los objetos a los que se hace referencia en la consulta (tablas, índices, estadísticas, etc.). En raras ocasiones, si se modifica un objeto de metadatos en la réplica principal mientras una consulta mantiene un bloqueo en el mismo objeto en la réplica de solo lectura, la consulta puede [bloquear](/sql/database-engine/availability-groups/windows/troubleshoot-primary-changes-not-reflected-on-secondary#BKMK_REDOBLOCK) el proceso que aplica los cambios de la réplica principal a la de solo lectura. Si este tipo de consulta se ejecutara durante mucho tiempo, haría que la réplica de solo lectura no estuviera sincronizada con la réplica principal.

Si una consulta de ejecución prolongada en una réplica de solo lectura produce este tipo de bloqueo, se terminará automáticamente. La sesión recibirá el error 1219, "Su sesión se ha desconectado debido a una operación DDL de elevada prioridad" o el error 3947, "Se ha anulado la transacción porque el proceso secundario no pudo ponerse al día. Vuelva a intentar la transacción".

> [!NOTE]
> Si recibe el error 3961, 1219 o 3947 al ejecutar consultas en una réplica de solo lectura, vuelva a intentar la consulta.

> [!TIP]
> En los niveles de servicio Premium y Crítico para la empresa, cuando se conecta a una réplica de solo lectura, se pueden usar las columnas `redo_queue_size` y `redo_rate` de las DMV [sys.dm_database_replica_states](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-replica-states-azure-sql-database) para supervisar el proceso de sincronización de datos, sirviendo como indicadores de latencia de datos en la réplica de solo lectura.
> 

## <a name="enable-and-disable-read-scale-out"></a>Habilitación y deshabilitación del escalado horizontal de lectura

La característica Escalado horizontal de lectura está habilitada de forma predeterminada en los niveles de servicio Premium, Crítico para la empresa e Hiperescala. No se puede habilitar en los niveles de servicio Básico, Estándar o De uso general. Se deshabilita automáticamente en las bases de datos del nivel Hiperescala que no están configuradas con ninguna réplica.

Puede deshabilitar y volver a habilitar la característica Escalado horizontal de lectura en las bases de datos únicas y las de grupos elásticos de los niveles de servicio Premium o Crítico para la empresa con los métodos siguientes.

> [!NOTE]
> Para bases de datos únicas y de grupos elásticos, la posibilidad de deshabilitar la característica Escalado horizontal de lectura se proporciona por motivos de compatibilidad con versiones anteriores. No se puede deshabilitar el escalado horizontal de lectura en las instancias administradas de tipo Crítico para la empresa.

### <a name="azure-portal"></a>Azure portal

Puede administrar la configuración de escalado horizontal de lectura en la hoja **Configurar** de la base de datos.

### <a name="powershell"></a>PowerShell

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de Azure Resource Manager continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos de Azure Resource Manager son esencialmente idénticos. Para más información sobre la compatibilidad, consulte [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

Para administrar el escalado horizontal de lectura en Azure PowerShell se requiere la versión de Azure PowerShell de diciembre de 2016 o posterior. Para saber dónde encontrar la versión más reciente de PowerShell, consulte [Azure PowerShell](/powershell/azure/install-az-ps).

Puede habilitar o deshabilitar la característica Escalado horizontal de lectura en Azure PowerShell invocando el cmdlet [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) y pasando el valor deseado, `Enabled` o `Disabled`, al parámetro `-ReadScale`.

Para deshabilitar el escalado horizontal de lectura en una base de datos existente (los elementos entre corchetes angulares deben sustituirse por los valores correctos en función del entorno y sin incluir dichos corchetes):

```powershell
Set-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> -ReadScale Disabled
```

Para deshabilitar el escalado horizontal de lectura en una nueva base de datos (los elementos entre corchetes angulares deben sustituirse por los valores correctos en función del entorno y sin incluir dichos corchetes):

```powershell
New-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> -ReadScale Disabled -Edition Premium
```

Para volver a habilitar el escalado horizontal de lectura en una base de datos existente (los elementos entre corchetes angulares deben sustituirse por los valores correctos en función del entorno y sin incluir dichos corchetes):

```powershell
Set-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> -ReadScale Enabled
```

### <a name="rest-api"></a>API DE REST

Para crear una base de datos con el escalado horizontal de lectura deshabilitado o para cambiar la configuración de una base de datos existente, utilice el método siguiente con la propiedad `readScale` establecida en `Enabled` o `Disabled` como en la siguiente solicitud de ejemplo.

```rest
Method: PUT
URL: https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{GroupName}/providers/Microsoft.Sql/servers/{ServerName}/databases/{DatabaseName}?api-version= 2014-04-01-preview
Body: {
   "properties": {
      "readScale":"Disabled"
   }
}
```

Para más información, consulte [Bases de datos: crear o actualizar](/rest/api/sql/databases/createorupdate).

## <a name="using-the-tempdb-database-on-a-read-only-replica"></a>Uso de la base de datos `tempdb` en una réplica de solo lectura

La base de datos `tempdb` de la réplica principal no se replica a las de solo lectura. Cada réplica tiene su propia base de datos `tempdb`, que se crea a la vez que lo hace la réplica. Esto garantiza que `tempdb` se puede actualizar y modificar durante la ejecución de consultas. Si la carga de trabajo de solo lectura depende del uso de objetos de `tempdb`, debe crear estos objetos como parte del script de la consulta.

## <a name="using-read-scale-out-with-geo-replicated-databases"></a>Uso del escalado horizontal de lectura con las bases de datos con replicación geográfica

Las bases de datos secundarias con replicación geográfica tienen la misma arquitectura de alta disponibilidad que las principales. Si se conecta a la base de datos secundaria con replicación geográfica con el escalado horizontal de lectura habilitado, las sesiones con `ApplicationIntent=ReadOnly` se redirigirán a una de las réplicas de alta disponibilidad del mismo modo que se redirigen en la base de datos de escritura principal. Las sesiones sin `ApplicationIntent=ReadOnly` se enrutarán a la réplica principal de la secundaria con replicación geográfica, que también es de solo lectura. 

De este modo, la creación de una réplica geográfica proporciona dos réplicas más de solo lectura para una base de datos principal de lectura y escritura y un total de tres réplicas de solo lectura. Cada réplica geográfica adicional proporciona otro par de réplicas de solo lectura. Las réplicas geográficas se pueden crear en cualquier región de Azure, incluida la región de la base de datos principal.

> [!NOTE]
> No se permite aplicar el método round-robin ni ningún otro enrutamiento de carga equilibrada entre las réplicas de una base de datos secundaria con replicación geográfica.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la oferta de SQL Database de nivel Hiperescala, consulte el artículo sobre el [nivel de servicio Hiperescala](service-tier-hyperscale.md).
