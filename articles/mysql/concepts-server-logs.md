---
title: 'Registros de consultas lentas: Azure Database for MySQL'
description: Describe los registros de consultas lentas disponibles en Azure Database for MySQL y los parámetros disponibles para habilitar diferentes niveles de registro.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 11/6/2020
ms.openlocfilehash: e3342b38f782b718cefd63295ef4d4d26b525058
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259054"
---
# <a name="slow-query-logs-in-azure-database-for-mysql"></a>Registros de consultas lentas en Azure Database for MySQL
En Azure Database for MySQL, el registro de consultas lentas está disponible para los usuarios. No se admite el acceso al registro de transacciones. El registro de consultas lentas puede utilizarse para identificar cuellos de botella que afectan al rendimiento a fin de solucionar el problema.

Para obtener más información sobre el registro de consultas lentas de MySQL, vea la [sección de registro de consultas lentas](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) del manual de referencia.

Cuando [Almacén de consultas](concepts-query-store.md) está habilitado en el servidor, puede ver que las consultas como "`CALL mysql.az_procedure_collect_wait_stats (900, 30);`" se han registrado en los registros de consultas lentas. Este comportamiento es el esperado, ya que la característica Almacén de consultas recopila estadísticas sobre las consultas. 

## <a name="configure-slow-query-logging"></a>Configuración del registro de consultas lentas 
De forma predeterminada, el registro de consultas lentas está deshabilitado. Para habilitarlo, cambie `slow_query_log` a Activado. Para ello, se puede usar Azure Portal o la CLI de Azure. 

Otros parámetros que se pueden ajustar son los siguientes:

- **long_query_time**: si una consulta tarda más que long_query_time (en segundos), esa consulta se registra. El valor predeterminado es 10 segundos.
- **log_slow_admin_statements**: si ON incluye instrucciones administrativas como ALTER_TABLE y ANALYZE_TABLE en las instrucciones escritas en slow_query_log.
- **log_queries_not_using_indexes**: determina si las consultas que no utilizan índices se registran en slow_query_log.
- **log_throttle_queries_not_using_indexes**: este parámetro limita el número de consultas que no son de índice que se pueden escribir en el registro de consultas lentas. Este parámetro surte efecto cuando log_queries_not_using_indexes está configurado en ON.
- **log_output**: si es "File", permite escribir el registro de consultas lento en el almacenamiento del servidor local y en los registros de diagnóstico de Azure Monitor. Si es "None", el registro de consultas lentas solo se escribirá en los registros de diagnóstico de Azure Monitor. 

> [!IMPORTANT]
> Si las tablas no están indexadas y se establecen loa parámetros `log_queries_not_using_indexes` y `log_throttle_queries_not_using_indexes` en ON (Activar) el rendimiento de MySQL puede resultar afectado, ya que todas las consultas que se ejecuten en estas tablas se escribirán en el registro de consulta lento.<br><br>
> Si planea registrar consultas lentas durante un periodo prolongado, se recomienda establecer `log_output` en "None" (No). Si se establece en "File" (Archivo), estos registros se escriben en el almacenamiento del servidor local, lo que puede afectar al rendimiento de MySQL. 

Consulte la [documentación rel registro de consultas lentas](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) de MySQL para obtener una descripción completa de los parámetros de registro de consultas lentas.

## <a name="access-slow-query-logs"></a>Acceso a registros de consultas lentas
Hay dos opciones para acceder a los registros de consultas lentas en Azure Database for MySQL: almacenamiento del servidor local o registros de diagnóstico de Azure Monitor. Esto se establece mediante el parámetro `log_output`.

Para el almacenamiento en el servidor local, puede enumerar y descargar los registros de consultas lentas mediante Azure Portal o la CLI de Azure. En Azure Portal, vaya al servidor en Azure Portal. En el encabezado **Supervisión**, seleccione la página **Registros de servidor**. Para más información sobre la CLI de Azure, consulte [Configuración y acceso a los registros de consultas lentas con la CLI de Azure](howto-configure-server-logs-in-cli.md). 

Los registros de diagnóstico de Azure Monitor permiten canalizar los registros de consultas lentas a los registros de Azure Monitor (Log Analytics), Azure Storage o Event Hubs. Para más información, [siga leyendo](concepts-server-logs.md#diagnostic-logs).

## <a name="local-server-storage-log-retention"></a>Retención de registros de almacenamiento del servidor local
Al iniciar sesión en el almacenamiento local del servidor, los registros están disponibles hasta siete días desde su creación. Si el tamaño total de los registros disponibles supera los 7 GB, se eliminan los archivos más antiguos hasta que haya espacio disponible. El límite de almacenamiento de 7 GB para los registros del servidor está disponible sin costo y no se puede ampliar. 

Los registros se rotan cada 24 horas o 7 GB, lo que ocurra primero.

> [!Note]
> La retención de registros anterior no se aplica a los registros que se canalizan mediante los registros de diagnóstico de Azure Monitor. Puede cambiar el período de retención de los receptores de datos que se emiten a, por ejemplo, Azure Storage.

## <a name="diagnostic-logs"></a>Registros de diagnóstico
Azure Database for MySQL se integra con los registros de diagnóstico de Azure Monitor. Después de habilitar los registros de consultas lentas en el servidor MySQL, puede optar por hacer que se emitan a los registros de Azure Monitor, Event Hubs o Azure Storage. Para más información sobre cómo habilitar los registros de diagnóstico, consulte la sección de la [documentación de registros de diagnóstico](../azure-monitor/essentials/platform-logs-overview.md).

En la tabla siguiente se describe lo que contiene cada registro. En función del método de salida que elija, pueden variar los campos incluidos y el orden en el que aparecen.

| **Propiedad** | **Descripción** |
|---|---|
| `TenantId` | El identificador de inquilino |
| `SourceSystem` | `Azure` |
| `TimeGenerated` [UTC] | Marca de tiempo de cuando se grabó el registro en UTC |
| `Type` | Tipo del registro. Siempre `AzureDiagnostics` |
| `SubscriptionId` | GUID de la suscripción a la que pertenece el servidor |
| `ResourceGroup` | Nombre del grupo de recursos al que pertenece el servidor |
| `ResourceProvider` | Nombre del proveedor de recursos Siempre `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de recurso |
| `Resource` | Nombre del servidor |
| `Category` | `MySqlSlowLogs` |
| `OperationName` | `LogEvent` |
| `Logical_server_name_s` | Nombre del servidor |
| `start_time_t` [UTC] | Hora de inicio de la consulta |
| `query_time_s` | Tiempo total en segundos que tardó en ejecutarse la consulta |
| `lock_time_s` | Tiempo total en segundos durante el que se bloqueó la consulta |
| `user_host_s` | Nombre de usuario |
| `rows_sent_s` | Número de filas enviadas |
| `rows_examined_s` | Número de filas examinadas |
| `last_insert_id_s` | [last_insert_id](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_last-insert-id) |
| `insert_id_s` | Id. de inserción |
| `sql_text_s` | Consulta completa |
| `server_id_s` | ID del servidor |
| `thread_id_s` | Id. de subproceso |
| `\_ResourceId` | URI de recurso |

> [!Note]
> Para `sql_text`, el registro se truncará si supera los 2048 caracteres.

## <a name="analyze-logs-in-azure-monitor-logs"></a>Análisis de registros en los registros de Azure Monitor

Una vez que los registros de consultas lentos se canalizan a los registros de Azure Monitor a través de registros de diagnóstico, puede realizar un análisis en mayor profundidad de sus consultas lentas. A continuación encontrará algunas consultas de ejemplo que le ayudarán a ponerse en marcha. Asegúrese de que actualizar los datos siguientes con el nombre del servidor.

- Consultas que duran más de 10 segundos en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```

- Lista de cinco consultas más largas en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | order by query_time_d desc
    | take 5
    ```

- Resumir las consultas lentas por tiempo de consulta mínimo, máximo, medio y desviación estándar en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count(), min(query_time_d), max(query_time_d), avg(query_time_d), stdev(query_time_d), percentile(query_time_d, 95) by LogicalServerName_s
    ```

- Representar en un gráfico la distribución de consultas lentas en un servidor concreto

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart
    ```

- Mostrar las consultas durante más de 10 segundos en todos los servidores de MySQL con Registros de diagnóstico habilitado

    ```Kusto
    AzureDiagnostics
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```    
    
## <a name="next-steps"></a>Pasos siguientes
- [Configuración registros de consultas lentos en Azure Portal](howto-configure-server-logs-in-portal.md)
- [Configuración de registros de consultas lentos en la CLI de Azure](howto-configure-server-logs-in-cli.md)
