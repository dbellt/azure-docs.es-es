---
title: Supervisión de SYNAPSE Analytics mediante Azure Monitor
description: Obtenga información sobre cómo supervisar el área de trabajo de Synapse Analytics mediante métricas, alertas y registros de Azure Monitor
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: ba730adaa1e3b111586012ba1849c6dc84d6db00
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553998"
---
# <a name="use-azure-monitor-with-your-azure-synapse-analytics-workspace"></a>Uso de Azure Monitor con el área de trabajo de Azure Synapse Analytics

Las aplicaciones en la nube son complejas y tienen muchas partes en movimiento. Los supervisores proporcionan datos para garantizar que las aplicaciones permanecen en funcionamiento en un estado correcto. Los supervisores también ayudan a evitar posibles problemas y a solucionar los problemas anteriores. Puede usar datos de supervisión para extraer conclusiones detalladas sobre las aplicaciones. Este conocimiento le ayuda a mejorar el rendimiento y el mantenimiento de las aplicaciones. También le ayuda a automatizar acciones que, de otro modo, requieren intervención manual.

Azure Monitor ofrece métricas, alertas y registros de las infraestructuras a nivel básico para la mayoría de los servicios de Azure. Un recurso emite registros de diagnóstico de Azure que proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de ese recurso. Azure Synapse Analytics puede escribir registros de diagnóstico en Azure Monitor.

Para más información, consulte [Introducción a Azure Monitor](../../azure-monitor/overview.md).

## <a name="metrics"></a>Métricas

Con Monitor, puede obtener visibilidad sobre el rendimiento y el estado de las cargas de trabajo de Azure. El tipo más importante de datos de Monitor es la métrica, que también se denomina contador de rendimiento. La mayoría de los recursos de Azure emiten métricas. Monitor proporciona varias maneras de configurar y consumir estas métricas para supervisar y solucionar problemas.

Para acceder a estas métricas, complete las instrucciones que aparecen en la [plataforma de datos de Azure Monitor](../../azure-monitor/data-platform.md).

### <a name="workspace-level-metrics"></a>Métricas de nivel de área de trabajo

Estas son algunas de las métricas emitidas por las áreas de trabajo:

| **Métrica**                           | **Categoría de métrica, nombre para mostrar**                  | **Unidad** | **Tipos de agregación** | **Descripción**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| IntegrationActivityRunsEnded         | Integración, métrica de ejecuciones de actividad                     | Count    | Suma (valor predeterminado), Recuento                | El número total de ejecuciones de actividad que se produjeron/finalizaron en un período de 1 minuto. </br></br> Use la dimensión Resultado de esta métrica para filtrar por estado final Correcto, Error o Cancelado.|
| IntegrationPipelineRunsEnded         | Integración, métrica de ejecuciones de canalización                     | Count    | Suma (valor predeterminado), Recuento                | El número total de ejecuciones de canalización que se produjeron/finalizaron en un período de 1 minuto. </br></br> Use la dimensión Resultado de esta métrica para filtrar por estado final Correcto, Error o Cancelado. |
| IntegrationTriggerRunsEnded          | Integración, métrica de ejecuciones de desencadenador                      | Count    | Suma (valor predeterminado), Recuento                | El número total de ejecuciones de desencadenador que se produjeron/finalizaron en un período de 1 minuto. </br></br> Use la dimensión Resultado de esta métrica para filtrar por estado final Correcto, Error o Cancelado. |
| BuiltinSqlPoolDataProcessedBytes     | Grupo de SQL integrado, datos procesados (bytes) | Byte | Suma (valor predeterminado) | Cantidad de datos procesados por el grupo de SQL sin servidor integrado. |
| BuiltinSqlPoolLoginAttempts          | Grupo de SQL integrado, intentos de inicio de sesión | Count | Suma (valor predeterminado) | Número de intentos de inicio de sesión para el grupo de SQL sin servidor integrado. |
| BuiltinSqlPoolDataRequestsEnded      | Grupo de SQL integrado, solicitudes finalizadas (bytes) | Count | Suma (valor predeterminado) | Número de solicitudes de SQL finalizadas para el grupo de SQL sin servidor integrado. </br></br> Use la dimensión Resultado de esta métrica para filtrar por estado final. |

### <a name="dedicated-sql-pool-metrics"></a>Métricas de grupo de SQL dedicado

Estas son algunas de las métricas emitidas por los grupos de SQL dedicados:

| **Métrica**                           | **Nombre para mostrar**                  | **Unidad** | **Tipos de agregación** | **Descripción**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| DWULimit                            | Límite de DWU                       | Count   | Máx. (valor predeterminado), Mín., Promedio | Tamaño configurado del grupo de SQL |
| DWUUsed                             | DWU utilizada                        | Count   | Máx. (valor predeterminado), Mín., Promedio | Representa una representación de alto nivel del uso en el grupo de SQL. Medido por el límite de DWU * porcentaje de DWU |
| DWUUsedPercent                      | Porcentaje usado de DWU             | Percent | Máx. (valor predeterminado), Mín., Promedio | Representa una representación de alto nivel del uso en el grupo de SQL. Medido tomando el máximo entre el porcentaje de CPU y el porcentaje de E/S de datos |
| ConnectionsBlockedByFirewall        | Conexiones bloqueadas por el firewall | Count   | Suma (valor predeterminado)   | Recuento de conexiones bloqueadas por las reglas de firewall. Vuelva a visitar las directivas de control de acceso para el grupo de SQL y supervise estas conexiones si el recuento es alto. |
| AdaptiveCacheHitPercent             | Porcentaje de aciertos de caché adaptable   | Percent | Máx. (valor predeterminado), Mín., Promedio | Mide en qué grado las cargas de trabajo usan la caché adaptable. Use esta métrica con la métrica del porcentaje de aciertos de caché para determinar si se debe escalar para aumentar la capacidad o volver a ejecutar cargas de trabajo para hidratar la memoria caché. |
| AdaptiveCacheUsedPercent            | Porcentaje usado de caché adaptable  | Percent | Máx. (valor predeterminado), Mín., Promedio | Mide en qué grado las cargas de trabajo usan la caché adaptable. Use esta métrica con la métrica del porcentaje usado de caché para determinar si se debe escalar para aumentar la capacidad o volver a ejecutar cargas de trabajo para hidratar la memoria caché. |
| LocalTempDBUsedPercent              | Porcentaje usado de tempdb local    | Percent | Máx. (valor predeterminado), Mín., Promedio | El uso de tempdb local en todos los nodos de ejecución. Los valores se emiten cada cinco minutos. |
| MemoryUsedPercent                   | Porcentaje utilizado de memoria.          | Percent | Máx. (valor predeterminado), Mín., Promedio | Uso de memoria en todos los nodos del grupo de SQL |
| CPUPercent                          | Porcentaje usado de CPU             | Percent | Máx. (valor predeterminado), Mín., Promedio | Uso de CPU en todos los nodos del grupo de SQL |
| Conexiones                         | Conexiones                     | Count   | Suma (valor predeterminado)  | Recuento total de inicios de sesión en el grupo de SQL |
| ActiveQueries                      | Consultas activas                 | Count   | Suma (valor predeterminado)   | Las consultas activas. El uso de esta métrica sin filtrar y sin dividir muestra todas las consultas activas que se ejecutan en el sistema. |
| QueuedQueries                      | Consultas en cola                  | Count   | Suma (valor predeterminado)   | El recuento acumulado de solicitudes en cola una vez alcanzado el límite de simultaneidad máximo. |
| WLGActiveQueries                   | Consultas activas de grupo de cargas de trabajo   | Count   | Suma (valor predeterminado)   | Consultas activas dentro del grupo de cargas de trabajo. El uso de esta métrica sin filtrar y sin dividir muestra todas las consultas activas que se ejecutan en el sistema. |
| WLGActiveQueriesTimeouts           | Tiempos de espera de consultas de grupo de cargas de trabajo   | Count   | Suma (valor predeterminado)   | Consultas para el grupo de cargas de trabajo que han agotado el tiempo de espera. Los tiempos de espera de consulta que indica esta métrica son solo una vez que se ha iniciado la ejecución de la consulta (no incluye el tiempo de espera debido a las esperas de bloqueo o de recursos). |
| WLGQueuedQueries                   | Consultas en cola del grupo de cargas de trabajo   | Count   | Suma (valor predeterminado)   | El recuento acumulado de solicitudes en cola una vez alcanzado el límite de simultaneidad máximo. |
| WLGAllocationBySystemPercent        | Asignación de grupos de cargas de trabajo por porcentaje del sistema | Percent | Máx. (valor predeterminado), Mín., Promedio, Suma | El porcentaje de asignación de recursos en relación con todo el sistema. |
| WLGAllocationByEffectiveCapResourcePercent   | Asignación de grupos de cargas de trabajo por porcentaje máximo de recursos | Percent | Máx. (valor predeterminado), Mín., Promedio | Muestra el porcentaje de asignación de recursos en relación con la métrica Porcentaje máximo de recursos efectivo por grupo de cargas de trabajo. Esta métrica proporciona la utilización efectiva del grupo de cargas de trabajo. |
| WLGEffectiveCapResourcePercent      | Porcentaje máximo de recursos efectivo  | Percent | Máx. (valor predeterminado), Mín., Promedio | Porcentaje máximo de recursos efectivo para el grupo de cargas de trabajo. Si hay otros grupos de cargas de trabajo con min_percentage_resource > 0, effective_cap_percentage_resource se reduce proporcionalmente |
| WLGEffectiveMinResourcePercent      | Porcentaje mínimo de recursos efectivo  | Percent | Máx. (valor predeterminado), Mín., Promedio, Suma | La configuración de porcentaje de recursos mínimos efectivos permitida teniendo en cuenta el nivel de servicio y la configuración del grupo de cargas de trabajo. El valor min_percentage_resource efectivo se puede ajustar en niveles de servicio superiores o inferiores. |

### <a name="apache-spark-pool-metrics"></a>Métricas de grupo de Apache Spark

Estas son algunas de las métricas emitidas por los grupos de Apache Spark:

| **Métrica**                           | **Categoría de métrica, nombre para mostrar**                  | **Unidad** | **Tipos de agregación** | **Descripción**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| BigDataPoolApplicationsEnded  | Aplicaciones de Apache Spark finalizadas  | Count | Suma (valor predeterminado) | Número de aplicaciones de grupo de Apache Spark finalizadas |
| BigDataPoolAllocatedCores     | Número de núcleos virtuales asignados al grupo de Apache Spark                 | Count | Máx. (valor predeterminado), Mín., Promedio | Núcleos virtuales asignados para un grupo de Apache Spark |
| BigDataPoolAllocatedMemory    | Cantidad de memoria (GB) asignada al grupo de Apache Spark            | Count | Máx. (valor predeterminado), Mín., Promedio | Memoria asignada para el grupo de Apache Spark (GB) |
| BigDataPoolApplicationsActive | Aplicaciones de Apache Spark activas | Count | Máx. (valor predeterminado), Mín., Promedio | Número de aplicaciones de grupo de Apache Spark activas |

## <a name="alerts"></a>Alertas

Inicie sesión en Azure Portal y haga clic en **Monitor** > **Alertas** para crear alertas.

### <a name="create-alerts"></a>Creación de alertas

1. Haga clic en **+Nueva regla de alertas** para crear una nueva alerta.

1. Defina la **condición de alerta** para especificar cuándo debe activarse la alerta.

    > [!NOTE]
    > Asegúrese de seleccionar **Todo** en **Filtrar por tipo de recurso**.

1. Defina los **detalles de alerta** para especificar más cómo debe configurare la alerta.

1. Defina el **grupo de acciones** para determinar quién debe recibir las alertas (y cómo).

## <a name="logs"></a>Registros

### <a name="workspace-level-logs"></a>Registros de nivel de área de trabajo

Estos son los registros emitidos por las áreas de trabajo de Azure Synapse Analytics:

| Nombre de la tabla de Log Analytics       | Nombre de la categoría del registro              | Descripción          |
|--------------------------------|--------------------------------|----------------------|
| SynapseGatewayApiRequests      | GatewayApiRequests             | Solicitudes de API de puerta de enlace de Azure Synapse. |
| SynapseRbacOperations          | SynapseRbacOperations          | Operaciones de control de acceso basado en rol (SRBAC) de Azure Synapse. |
| SynapseBuiltinSqlReqsEnded     | BuiltinSqlReqsEnded            | Solicitudes finalizadas de grupo de SQL sin servidor integradas de Azure Synapse. |
| SynapseIntegrationPipelineRuns | IntegrationPipelineRuns        | Ejecuciones de canalización de integración de Azure Synapse. |
| SynapseIntegrationActivityRuns | IntegrationActivityRuns        | Ejecuciones de actividad de integración de Azure Synapse. |
| SynapseIntegrationTriggerRuns  | IntegrationTriggerRuns         | Ejecuciones de desencadenador de integración de Azure Synapse. |

### <a name="dedicated-sql-pool-logs"></a>Registros de grupo de SQL dedicado

Estos son los registros emitidos por los grupos de SQL dedicados:

| Nombre de la tabla de Log Analytics        | Nombre de la categoría del registro             | Descripción |
|----------------------|--------------------------------------|-------------|
| SynapseSqlPoolExecRequests  | ExecRequests | Información sobre las solicitudes y consultas de SQL de un grupo de SQL dedicado de Azure Synapse.
| SynapseSqlPoolDmsWorkers    | DmsWorkers   | Información sobre los trabajadores que completan los pasos de DMS en un grupo de SQL dedicado de Azure Synapse.
| SynapseSqlPoolRequestSteps  | RequestSteps | Información sobre los pasos de la solicitud que componen una solicitud o consulta de SQL determinada en un grupo de SQL dedicado de Azure Synapse.
| SynapseSqlPoolSqlRequests   | SqlRequests  | Información sobre las distribuciones de consultas de los pasos de las solicitudes y consultas de SQL en un grupo de SQL dedicado de Azure Synapse.
| SynapseSqlPoolWaits         | Esperas        | Información sobre los estados de espera encontrados durante la ejecución de una solicitud o consulta de SQL en un grupo de SQL dedicado de Azure Synapse, incluidos los bloqueos y las esperas en las colas de transmisión.

Para obtener más información sobre estos registros, consulte la siguiente información:
- [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?view=azure-sqldw-latest&preserve-view=true)

### <a name="apache-spark-pool-log"></a>Registro de grupo de Apache Spark

Este es el registro emitido por los grupos de Apache Spark:

| Nombre de la tabla de Log Analytics               | Nombre de la categoría del registro              | Descripción                 |
|-----------------------------|---------------------------------------|-----------------------------|
| SynapseBigDataPoolApplicationsEnded | BigDataPoolAppsEnded | Información sobre las aplicaciones de Apache Spark |

### <a name="diagnostic-settings"></a>Configuración de diagnóstico

Use la configuración de diagnóstico para configurar los registros de diagnóstico de los recursos que no son de proceso. La configuración de un control de recursos tiene las siguientes características:

* Especifican dónde se envían los registros de diagnóstico. Entre los ejemplos se incluyen una cuenta de almacenamiento de Azure, un centro de eventos de Azure o registros de Monitor.
* Especifican las categorías de registro que se envían.
* Especifican cuánto tiempo se debe conservar cada categoría de registro en una cuenta de almacenamiento.
* Una retención de cero días significa que los registros se conservan de forma indefinida. De lo contrario, el valor puede ser cualquier número de días comprendido entre 1 y 2 147 483 647.
* Si se establecen directivas de retención, pero el almacenamiento de registros en una cuenta de almacenamiento está deshabilitado, las directivas de retención no surten ningún efecto. Por ejemplo, esta condición puede ocurrir cuando solo se seleccionan las opciones de registros de Event Hubs o de Monitor.
* Las directivas de retención se aplican por día. El límite entre los días se produce a la medianoche de la hora universal coordinada (UTC). Al final del día, se eliminan los registros de los días que superan la directiva de retención. Por ejemplo, si tuviera una directiva de retención de un día, se eliminarían los registros de anteayer al principio del día de hoy.

Con la configuración de diagnóstico de Azure Monitor, puede enrutar los registros de diagnóstico para su análisis a varios destinos diferentes.

* **Cuenta de almacenamiento**: Guarde los registro de diagnóstico en una cuenta de almacenamiento para auditarlos o para inspeccionarlos manualmente. Puede usar la configuración de diagnóstico para especificar el tiempo de retención en días.
* **Centro de eventos**: Transmita los registros a Azure Event Hubs. Los registros se convierten en la entrada de un servicio del asociado o una solución de análisis personalizada como Power BI.
* **Área de trabajo de Log Analytics**: Analice los registros con Log Analytics. La integración de Azure Synapse con Log Analytics es útil en los escenarios siguientes:
  * Quiere escribir consultas complejas en un amplio conjunto de métricas publicadas por Azure Synapse para Log Analytics. Puede crear alertas personalizadas sobre estas consultas a través de Azure Monitor.
  * Quiere realizar la supervisión entre áreas de trabajo. Puede enrutar datos desde varias áreas de trabajo a una sola área de trabajo de Log Analytics.

Puede usar también una cuenta de almacenamiento o un espacio de nombres del Centro de eventos que no esté en la misma suscripción que el recurso que emite los registros. El usuario que configura los ajustes debe tener el control de acceso basado en roles de Azure (Azure RBAC) adecuado a ambas suscripciones.

#### <a name="configure-diagnostic-settings"></a>Configuración de valores de diagnóstico

Cree o agregue una configuración de diagnóstico para el área de trabajo, el grupo de SQL dedicado o el grupo de Apache Spark.

1. En el portal, vaya a Monitor. Seleccione **Configuración** > **Configuración de diagnóstico**.

1. Seleccione el área de trabajo de Synapse, el grupo de SQL dedicado o el grupo de Apache Spark para el que desea crear una configuración de diagnóstico.

1. Si no existe ninguna configuración de diagnóstico en el área de trabajo seleccionada, se le pedirá que cree una. Seleccione **Activar diagnósticos**.

   Si hay una configuración de diagnóstico actual en el área de trabajo, verá una lista de opciones ya configuradas en el recurso. Seleccione **Agregar configuración de diagnóstico**.

1. Asigne un nombre de la configuración, seleccione **Enviar a Log Analytics** y, después, seleccione un área de trabajo en **Área de trabajo de Log Analytics**.

    > [!NOTE]
    > Como una tabla de registro de Azure no puede tener más de 500 columnas, se **recomienda encarecidamente** seleccionar el _modo específico de recursos_. Para obtener más información, consulte la [referencia de registros de AzureDiagnostics](/azure/azure-monitor/reference/tables/azurediagnostics).

1. Seleccione **Guardar**.

Transcurridos unos instantes, la nueva configuración aparecerá en la lista de opciones de configuración para el área de trabajo, el grupo de SQL dedicado o el grupo de Apache Spark. Los registros de diagnóstico se transmiten en esa área de trabajo en cuanto se generan nuevos datos de evento. Pueden pasar hasta 15 minutos entre el momento en que se emite un evento y cuando aparece en Log Analytics.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la supervisión de ejecuciones de canalización, consulte el artículo [Supervisión de ejecuciones de canalización en Synapse Studio](how-to-monitor-pipeline-runs.md). 

Para más información sobre la supervisión de las aplicaciones de Apache Spark, consulte el artículo sobre la [supervisión de aplicaciones de Apache Spark en Synapse Studio](apache-spark-applications.md).

Para obtener más información sobre la supervisión de solicitudes de SQL, consulte el artículo sobre la [supervisión de solicitudes de SQL en Synapse Studio](how-to-monitor-sql-requests.md).
