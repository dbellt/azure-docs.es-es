---
title: Configuración de la exportación de streaming de métricas y registros de recursos
description: Aprenda a configurar la exportación de streaming de métricas y registros de recursos, como el análisis de diagnóstico inteligente, desde Azure SQL Database e Instancia administrada de Azure SQL al destino de su elección para almacenar información sobre el uso de recursos y estadísticas de ejecución de consultas.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: seoapril2019, devx-track-azurepowershell
ms.devlang: sqldbrb=2
ms.topic: how-to
author: AlainDormehlMSFT
ms.author: aldorme
ms.reviewer: mathoma, wiassaf
ms.date: 04/06/2020
ms.openlocfilehash: 0c719eb8cec894fda35dd81aa65668feedd72e65
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111590655"
---
# <a name="configure-streaming-export-of-azure-sql-database-and-sql-managed-instance-diagnostic-telemetry"></a>Configuración de la exportación de streaming de los datos de telemetría de diagnóstico de Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este artículo, obtendrá información sobre las métricas de rendimiento y los registros de recursos de Azure SQL Database que puede exportar a uno de varios destinos para su análisis. Aprenderá a configurar la exportación de streaming de estos datos de telemetría de diagnóstico mediante Azure Portal, PowerShell, la CLI de Azure, la API REST y las plantillas de Azure Resource Manager.

También obtendrá información sobre los destinos a los que puede transmitir esta telemetría de diagnóstico y cómo elegir entre las diversas opciones. Las opciones de destino incluyen:

- [Log Analytics y SQL Analytics](#stream-into-sql-analytics)
- [Event Hubs](#stream-into-event-hubs)
- [Almacenamiento de Azure](#stream-into-azure-storage)

## <a name="diagnostic-telemetry-for-export"></a>Datos de telemetría de diagnóstico para exportación

La telemetría de diagnóstico más importante que puede exportar es el registro de Intelligent Insights (SQLInsights). [Intelligent Insights](intelligent-insights-overview.md) usa inteligencia integrada para supervisar continuamente el uso de la base de datos mediante inteligencia artificial y detectar eventos potencialmente perjudiciales que provoquen un rendimiento bajo. Una vez detectados, se realiza un análisis detallado que genera un registro de Intelligent Insights con una evaluación inteligente del problema. Esta evaluación está formada por un análisis de la causa raíz del problema de rendimiento de la base de datos y, si es posible, recomendaciones para mejorar el rendimiento. Debe configurar la exportación de streaming de este registro para ver su contenido.

Además de hacer streaming de la exportación del registro de Intelligent Insights, también puede exportar varias métricas de rendimiento y registros de base de datos adicionales. En la tabla siguiente se describen las métricas de rendimiento y los registros de recursos que se pueden configurar para la exportación de streaming a uno de varios destinos. Esta telemetría de diagnóstico se puede configurar para bases de datos únicas, grupos de bases de datos elásticas y bases de datos agrupadas, así como para instancias administradas y bases de datos de instancia.

| Telemetría de diagnóstico para bases de datos | Compatibilidad con Azure SQL Database | Compatibilidad con la Instancia administrada de Azure SQL |
| :------------------- | ----- | ----- |
| [Métricas básicas](#basic-metrics): contiene el porcentaje de DTU/CPU, el límite de DTU/CPU, el porcentaje de lectura de datos físicos, el porcentaje de escritura en registro, las conexiones correctas, erróneas o bloqueadas por el firewall, el porcentaje de sesiones, el porcentaje de trabajo, el almacenamiento, el porcentaje de almacenamiento y el porcentaje de almacenamiento de XTP. | Sí | No |
| [Instancia y aplicación avanzadas](#advanced-metrics): contiene los datos de la base de datos y el tamaño de archivo de registro del sistema tempdb y el archivo de registro porcentual de tempdb usado. | Sí | No |
| [QueryStoreRuntimeStatistics](#query-store-runtime-statistics): contiene la información sobre las estadísticas de tiempo de ejecución de consulta, como el uso de CPU y la duración de la consulta. | Sí | Sí |
| [QueryStoreWaitStatistics](#query-store-wait-statistics): contiene la información sobre las estadísticas de espera de las consultas (el motivo de la espera de sus consultas), como CPU, LOG y LOCKING. | Sí | Sí |
| [Errores](#errors-dataset): contiene información sobre los errores de SQL en una base de datos. | Sí | Sí |
| [DatabaseWaitStatistics](#database-wait-statistics-dataset): contiene la información sobre cuánto tiempo ha dedicado una base de datos a esperar distintos tipos de espera. | Sí | No |
| [Tiempos de expiración](#time-outs-dataset): contiene información sobre los tiempos de expiración en una base de datos. | Sí | No |
| [Bloqueos](#blockings-dataset): contiene información sobre los eventos de bloqueo en una base de datos. | Sí | No |
| [Interbloqueos](#deadlocks-dataset): contiene información sobre los eventos de interbloqueo en una base de datos. | Sí | No |
| [AutomaticTuning](#automatic-tuning-dataset): contiene información sobre las recomendaciones de ajuste automático para una base de datos. | Sí | No |
| [SQLInsights](#intelligent-insights-dataset): contiene Intelligent Insights sobre el rendimiento de una base de datos. Para obtener más información, consulte [Intelligent Insights](intelligent-insights-overview.md). | Sí | Sí |

> [!NOTE]
> No se puede establecer la configuración de diagnóstico de las **bases de datos del sistema**, como las bases de datos maestra, msdb, modelo, de recursos y tempdb.

## <a name="streaming-export-destinations"></a>Destinos de la exportación de streaming

Esta telemetría de diagnóstico se puede exportar a uno de los siguientes recursos de Azure para su análisis.

- **[Área de trabajo de Log Analytics](#stream-into-sql-analytics)** :

  Los datos que se transmiten a un [área de trabajo de Log Analytics](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace) pueden ser consumidos por [SQL Analytics](../../azure-monitor/insights/azure-sql.md). SQL Analytics es una solución de supervisión en la nube que proporciona supervisión inteligente de las bases de datos, lo que incluye informes de rendimiento, alertas y recomendaciones de mitigación. Los datos que se transmiten a un área de trabajo de Log Analytics se pueden analizar con otros datos de supervisión recopilados y también permiten aprovechar otras características de Azure Monitor, como las alertas y las visualizaciones.
- **[Azure Event Hubs](#stream-into-event-hubs)** :

  Los datos que se transmiten a un [centro de eventos de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs) proporcionan las siguientes funcionalidades:

  - **Transmisión de registros a sistemas de registro y telemetría de terceros**: transmita todas sus métricas y todos sus registros de recursos a un centro de eventos único para canalizar datos de registro en una herramienta SIEM o de análisis de registros de terceros.
  - **Creación de una plataforma personalizada de registro y telemetría**: la naturaleza altamente escalable de publicación y suscripción de los centros de eventos otorga la flexibilidad necesaria para ingerir métricas y registros de recursos en una plataforma de telemetría personalizada. Consulte [Designing and Sizing a Global Scale Telemetry Platform on Azure Event Hubs](https://azure.microsoft.com/documentation/videos/build-2015-designing-and-sizing-a-global-scale-telemetry-platform-on-azure-event-Hubs/) (Diseño y cambio de tamaño de una plataforma de telemetría a escala global en Azure Event Hubs) para más información.
  - **Visualización del estado del servicio mediante la transmisión de datos a Power BI**: use Event Hubs, Stream Analytics y Power BI para transformar los datos de diagnóstico en información sobre los servicios de Azure prácticamente en tiempo real. Consulte [Stream Analytics y Power BI: panel de análisis en tiempo real de flujo de datos](../../stream-analytics/stream-analytics-power-bi-dashboard.md) para detalles sobre esta solución.
- **[Azure Storage](#stream-into-azure-storage)** :

  Los datos que se transmiten a [Azure Storage](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage) permiten archivar gran cantidad de información de telemetría de diagnóstico por una fracción del costo de las dos opciones anteriores de streaming.

Esta telemetría de diagnóstico transmitida a uno de estos destinos se puede usar para medir el uso de recursos y las estadísticas de ejecución de consultas con el fin de facilitar la supervisión del rendimiento.

![En el diagrama se muestran muchas bases de datos SQL y bases de datos en instancias administradas que envían datos de telemetría a Azure Diagnostics, que reenvía información a Azure SQL Analytics, Event Hub y Storage.](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/architecture.png)

## <a name="enable-and-configure-the-streaming-export-of-diagnostic-telemetry"></a>Habilitación y configuración de la exportación de streaming de los datos de telemetría de diagnóstico

Puede habilitar y administrar las métricas y los registros de datos de telemetría de diagnóstico usando uno de los métodos siguientes:

- Azure portal
- PowerShell
- Azure CLI
- API de REST de Azure Monitor
- Plantilla del Administrador de recursos de Azure

> [!NOTE]
> Para habilitar el streaming de registros de auditoría de la telemetría de seguridad, consulte los artículos en los que se explica la [configuración de la auditoría para una base de datos](./auditing-overview.md#setup-auditing) y los [registros de auditoría en los registros de Azure Monitor y Azure Event Hubs](https://techcommunity.microsoft.com/t5/Azure-SQL-Database/SQL-Audit-logs-in-Azure-Log-Analytics-and-Azure-Event-Hubs/ba-p/386242).

## <a name="configure-the-streaming-export-of-diagnostic-telemetry"></a>Configuración de la exportación de streaming de los datos de telemetría de diagnóstico

Puede usar el menú **Configuración de diagnóstico** de Azure Portal para habilitar y configurar el streaming de los datos de telemetría de diagnóstico. Además, puede usar PowerShell, la CLI de Azure, la [API REST](/rest/api/monitor/diagnosticsettings) y las [plantillas de Resource Manager](../../azure-monitor/essentials/resource-manager-diagnostic-settings.md) para configurar el streaming de los datos de telemetría de diagnóstico. Puede establecer los siguientes destinos para transmitir los datos de telemetría de diagnóstico: Azure Storage, Azure Event Hubs y registros de Azure Monitor.

> [!IMPORTANT]
> La exportación de streaming de los datos de telemetría de diagnóstico no está habilitada de forma predeterminada.

Seleccione una de las siguientes pestañas para obtener una guía paso a paso para configurar la exportación de streaming de los datos de telemetría de diagnóstico en Azure Portal y para obtener los scripts necesarios para realizar las mismas operaciones con PowerShell y la CLI de Azure.

# <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

### <a name="elastic-pools-in-azure-sql-database"></a>Grupos elásticos en Azure SQL Database

Puede configurar un recurso de grupos elásticos para recopilar los siguientes datos de telemetría de diagnóstico:

| Recurso | Telemetría de supervisión |
| :------------------- | ------------------- |
| **Grupo elástico** | [Métricas básicas](metrics-diagnostic-telemetry-logging-streaming-export-configure.md#basic-metrics) contiene el porcentaje de eDTU/CPU, el límite de eDTU/CPU, el porcentaje de lectura de datos físicos, el porcentaje de escritura en registro, el porcentaje de sesiones, el porcentaje de trabajos, el almacenamiento, el porcentaje de almacenamiento, el límite de almacenamiento y el porcentaje de almacenamiento de XTP. |

Para configurar el streaming de los datos de telemetría de diagnóstico de grupos elásticos y bases de datos agrupadas, es preciso que configure cada uno de estos elementos por separado:

- Habilite el streaming de datos de telemetría de diagnóstico para un grupo elástico.
- Habilite el streaming de datos de telemetría de diagnóstico para cada base de datos en un grupo elástico.

El contenedor del grupo elástico tiene su propia telemetría, que es independiente de la de cada base de datos agrupada individual.

Para habilitar el streaming de datos de telemetría de diagnóstico para un recurso de grupos elásticos, siga estos pasos:

1. Vaya al recurso del **grupo elástico** de Azure Portal.
2. Seleccione **Configuración de diagnóstico**.
3. Seleccione **Activar diagnóstico** si no existe ninguna configuración anterior o **Editar configuración** para editar una configuración anterior.

   ![Habilitación de diagnósticos en grupos elásticos](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-container-elasticpool-enable.png)

4. Escriba un nombre de configuración para su propia referencia.
5. Seleccione un recurso de destino para los datos de diagnóstico de streaming: **Archivar en una cuenta de almacenamiento**, **Transmitir en secuencias a un centro de eventos** o **Enviar a Log Analytics**.
6. En los análisis de registros, seleccione **Configurar** y cree una nueva área de trabajo en **+Crear nueva área de trabajo**, o bien seleccione un área de trabajo existente.
7. Active la casilla para la telemetría de diagnóstico de grupos elásticos: Métricas **básicas**.
   ![Configuración de diagnósticos en grupos elásticos](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-container-elasticpool-selection.png)

8. Seleccione **Guardar**.
9. Además, configure el streaming de datos de telemetría de diagnóstico para cada base de datos del grupo elástico que quiera supervisar siguiendo los pasos descritos en la sección siguiente.

> [!IMPORTANT]
> Aparte de configurar los datos de telemetría de diagnóstico de un grupo elástico, también es preciso que configure los datos de telemetría de diagnóstico de cada una de las bases de datos del grupo elástico.

### <a name="databases-in-azure-sql-database"></a>Bases de datos en Azure SQL Database

Puede configurar un recurso de base de datos para recopilar los siguientes datos de telemetría de diagnóstico:

| Recurso | Telemetría de supervisión |
| :------------------- | ------------------- |
| **Base de datos única o agrupada** | [Métrica básica](metrics-diagnostic-telemetry-logging-streaming-export-configure.md#basic-metrics) contiene el porcentaje de DTU; la DTU usada; el límite de DTU; el porcentaje de CPU; el porcentaje de lectura de datos físicos; el porcentaje de escritura en registro; las conexiones correctas, erróneas o bloqueadas por el firewall; el porcentaje de sesiones; el porcentaje de trabajos; el almacenamiento; el porcentaje de almacenamiento; el porcentaje de almacenamiento de XTP y los interbloqueos. |

Para habilitar el streaming de datos de telemetría de diagnóstico de una base de datos única o agrupada, siga estos pasos:

1. Vaya al recurso **Base de datos de Azure SQL**.
2. Seleccione **Configuración de diagnóstico**.
3. Seleccione **Activar diagnóstico** si no existe ninguna configuración anterior o **Editar configuración** para editar una configuración anterior. Puede crear hasta tres conexiones paralelas para transmitir los datos de la telemetría de diagnóstico.
4. Seleccione **Agregar configuración de diagnóstico** para configurar el streaming en paralelo de diagnóstico a varios recursos.

   ![Habilitar diagnósticos de bases de datos únicas y agrupadas](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-database-sql-enable.png)

5. Escriba un nombre de configuración para su propia referencia.
6. Seleccione un recurso de destino para los datos de diagnóstico de streaming: **Archivar en una cuenta de almacenamiento**, **Transmitir en secuencias a un centro de eventos** o **Enviar a Log Analytics**.
7. Para la experiencia de supervisión basada en eventos estándar, active las siguientes casillas para la telemetría de registro de diagnóstico de base de datos: **SQLInsights**, **AutomaticTuning**, **QueryStoreRuntimeStatistics**, **QueryStoreWaitStatistics**, **Errores**, **DatabaseWaitStatistics**, **Tiempos de expiración**, **Bloqueos** e **Interbloqueos**.
8. Para una experiencia avanzada de supervisión basada en un minuto, active la casilla de métricas **Básicas**.

   ![Configuración de diagnósticos en Azure SQL Database](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-database-sql-selection.png)
9. Seleccione **Guardar**.
10. Repita estos pasos para cada base de datos que quiera supervisar.

> [!TIP]
> Repita estos pasos para cada base de datos única o agrupada que quiera supervisar.

### <a name="instances-in-azure-sql-managed-instance"></a>Instancias en Instancia administrada de Azure SQL

Puede configurar un recurso de instancia administrada para recopilar los siguientes datos de telemetría de diagnóstico:

| Recurso | Telemetría de supervisión |
| :------------------- | ------------------- |
| **Instancia administrada** | [ResourceUsageStats](#resource-usage-stats-for-managed-instances): contiene el número de núcleos virtuales, el porcentaje medio de CPU, las solicitudes de E/S, los bytes leídos y escritos, el espacio de almacenamiento reservado y el espacio de almacenamiento utilizado. |

Para configurar el streaming de datos de telemetría de diagnóstico para bases de datos de instancia e instancia administrada, deberá configurar por separado cada una de ellas:

- Habilite el streaming de datos de telemetría de diagnóstico para una instancia administrada.
- Habilite el streaming de datos de telemetría de diagnóstico para cada base de datos de instancia.

El contenedor de instancia administrada tiene su propia telemetría, que es independiente de la de cada base de datos de instancia administrada.

Para habilitar el streaming de datos de telemetría de diagnóstico de un recurso de instancia administrada, siga estos pasos:

1. Vaya al recurso de **instancia administrada** en Azure Portal.
2. Seleccione **Configuración de diagnóstico**.
3. Seleccione **Activar diagnóstico** si no existe ninguna configuración anterior o **Editar configuración** para editar una configuración anterior.

   ![Habilitación de diagnósticos para una instancia administrada](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-container-mi-enable.png)

4. Escriba un nombre de configuración para su propia referencia.
5. Seleccione un recurso de destino para los datos de diagnóstico de streaming: **Archivar en una cuenta de almacenamiento**, **Transmitir en secuencias a un centro de eventos** o **Enviar a Log Analytics**.
6. En los análisis de registros, seleccione **Configurar** y cree una nueva área de trabajo en **+Crear nueva área de trabajo**, o bien use un área de trabajo existente.
7. Active la casilla para la telemetría de diagnóstico de instancias: **ResourceUsageStats**.

   ![Configuración de diagnósticos para una instancia administrada](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-container-mi-selection.png)

8. Seleccione **Guardar**.
9. Además, configure el streaming de datos de telemetría de diagnóstico para cada base de datos de instancia dentro de la instancia administrada que quiere supervisar siguiendo los pasos descritos en la sección siguiente.

> [!IMPORTANT]
> Además de configurar los datos de telemetría de diagnóstico para una instancia administrada, también deberá configurar los datos de telemetría de diagnóstico para cada base de datos de la instancia.

### <a name="databases-in-azure-sql-managed-instance"></a>Bases de datos en Instancia administrada de Azure SQL

Puede configurar un recurso de base de datos de instancia administrada para recopilar los siguientes datos de telemetría de diagnóstico:

| Recurso | Telemetría de supervisión |
| :------------------- | ------------------- |
| **Base de datos de instancia** | [ResourceUsageStats](#resource-usage-stats-for-managed-instances): contiene el número de núcleos virtuales, el porcentaje medio de CPU, las solicitudes de E/S, los bytes leídos y escritos, el espacio de almacenamiento reservado y el espacio de almacenamiento utilizado. |

Para habilitar el streaming de datos de telemetría de diagnóstico para una base de datos de instancia, siga estos pasos:

1. Vaya al recurso de **base de datos de instancia** dentro de la instancia administrada.
2. Seleccione **Configuración de diagnóstico**.
3. Seleccione **Activar diagnóstico** si no existe ninguna configuración anterior o **Editar configuración** para editar una configuración anterior.
   - Se pueden crear hasta tres (3) conexiones paralelas para transmitir datos de telemetría de diagnóstico.
   - Seleccione **+Agregar configuración de diagnóstico** para configurar el streaming en paralelo de diagnóstico a varios recursos.

   ![Habilitación de diagnóstico para bases de datos de instancia](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-database-mi-enable.png)

4. Escriba un nombre de configuración para su propia referencia.
5. Seleccione un recurso de destino para los datos de diagnóstico de streaming: **Archivar en una cuenta de almacenamiento**, **Transmitir en secuencias a un centro de eventos** o **Enviar a Log Analytics**.
6. Active las casillas para la telemetría de diagnóstico de bases de datos: **SQLInsights**, **QueryStoreRuntimeStatistics**, **QueryStoreWaitStatistics** y **Errores**.
   ![Configuración de diagnóstico para bases de datos de instancia](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/diagnostics-settings-database-mi-selection.png)
7. Seleccione **Guardar**.
8. Repita estos pasos para cada base de datos de instancia que quiera supervisar.

> [!TIP]
> Repita estos pasos para cada base de datos de instancia que quiera supervisar.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Puede habilitar las métricas y los registros de diagnóstico con PowerShell.

- Para habilitar el almacenamiento de métricas y registros de recursos en una cuenta de almacenamiento, use este comando:

  ```powershell
  Set-AzDiagnosticSetting -ResourceId [your resource id] -StorageAccountId [your storage account id] -Enabled $true
  ```

  El identificador de la cuenta de almacenamiento es el identificador de recurso para la cuenta de almacenamiento de destino.

- Para habilitar el streaming de métricas y registros de recursos en un centro de eventos, use este comando:

  ```powershell
  Set-AzDiagnosticSetting -ResourceId [your resource id] -ServiceBusRuleId [your service bus rule id] -Enabled $true
  ```

  El identificador de regla de Azure Service Bus es una cadena con este formato:

  ```powershell
  {service bus resource ID}/authorizationrules/{key name}
  ```

- Para habilitar el envío de métricas y registros de recursos a un área de trabajo de Log Analytics, use este comando:

  ```powershell
  Set-AzDiagnosticSetting -ResourceId [your resource id] -WorkspaceId [resource id of the log analytics workspace] -Enabled $true
  ```

- Puede obtener el identificador de recurso de su área de trabajo de Log Analytics con el comando siguiente:

  ```powershell
  (Get-AzOperationalInsightsWorkspace).ResourceId
  ```

Puede combinar estos parámetros para habilitar varias opciones de salida.

**Para configurar varios recursos de Azure**

Para admitir varias suscripciones, use el script de PowerShell de [Enable Azure resource metrics logging using PowerShell](/archive/blogs/msoms/enable-azure-resource-metrics-logging-using-powershell) (Habilitar registro de métricas de recursos de Azure mediante PowerShell).

Proporcione el identificador de recurso del área de trabajo \<$WSID\> como parámetro al ejecutar el script `Enable-AzureRMDiagnostics.ps1` para enviar los datos de diagnóstico de varios recursos a un área de trabajo.

- Para obtener el identificador de área de trabajo \<$WSID\> del destino para sus datos de diagnóstico, use el siguiente script:

  ```powershell
  $WSID = "/subscriptions/<subID>/resourcegroups/<RG_NAME>/providers/microsoft.operationalinsights/workspaces/<WS_NAME>"
  .\Enable-AzureRMDiagnostics.ps1 -WSID $WSID
  ```

  Reemplace \<subID\> por el identificador de suscripción, \<RG_NAME\> por el nombre del grupo de recursos y \<WS_NAME\> por el nombre del área de trabajo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede habilitar las métricas y los registros de diagnóstico con la CLI de Azure.

> [!IMPORTANT]
> Los scripts para habilitar el registro de diagnóstico son compatibles con CLI de Azure v1.0. La versión 2.0 de la CLI de Azure no es compatible en este momento.

- Para habilitar el almacenamiento de métricas y registros de recursos en una cuenta de almacenamiento, use este comando:

  ```azurecli-interactive
  azure insights diagnostic set --resourceId <resourceId> --storageId <storageAccountId> --enabled true
  ```

  El identificador de la cuenta de almacenamiento es el identificador de recurso para la cuenta de almacenamiento de destino.

- Para habilitar el streaming de métricas y registros de recursos en un centro de eventos, use este comando:

  ```azurecli-interactive
  azure insights diagnostic set --resourceId <resourceId> --serviceBusRuleId <serviceBusRuleId> --enabled true
  ```

  El identificador de regla de Service Bus es una cadena con este formato:

  ```azurecli-interactive
  {service bus resource ID}/authorizationrules/{key name}
  ```

- Para habilitar el envío de métricas y registros de recursos a un área de trabajo de Log Analytics, use este comando:

  ```azurecli-interactive
  azure insights diagnostic set --resourceId <resourceId> --workspaceId <resource id of the log analytics workspace> --enabled true
  ```

Puede combinar estos parámetros para habilitar varias opciones de salida.

---

## <a name="stream-into-sql-analytics"></a>Transmisión a SQL Analytics

Azure SQL Analytics puede consumir las métricas y los registros de recursos de Azure SQL Database e Instancia administrada de Azure SQL que se transmiten a un área de trabajo Log Analytics. Azure SQL Analytics es una solución en la nube que supervisa el rendimiento de bases de datos únicas, grupos elásticos y bases de datos agrupadas, así como instancias administradas y bases de datos de instancia a escala y entre varias suscripciones. Puede ayudarle a recopilar y visualizar métricas del rendimiento, y cuenta con inteligencia integrada para solucionar problemas de rendimiento.

![Información general de Azure SQL Analytics](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/azure-sql-sol-overview.png)

### <a name="installation-overview"></a>Introducción a la instalación

Puede supervisar una colección de bases de datos y colecciones de bases de datos con Azure SQL Analytics siguiendo los pasos a continuación:

1. Cree una solución de Azure SQL Analytics en Azure Marketplace.
2. Cree un área de trabajo de Log Analytics en la solución.
3. Configure bases de datos para transmitir datos de telemetría de diagnóstico al área de trabajo.

Puede configurar la exportación de streaming de esta telemetría de diagnóstico mediante la opción integrada **Enviar a Log Analytics** en la pestaña Configuración de diagnóstico de Azure Portal. También puede habilitar el streaming a un área de trabajo de Log Analytics mediante el uso de la configuración de diagnóstico a través de [cmdlets de PowerShell](metrics-diagnostic-telemetry-logging-streaming-export-configure.md?tabs=azure-powershell#configure-the-streaming-export-of-diagnostic-telemetry), la [CLI de Azure](metrics-diagnostic-telemetry-logging-streaming-export-configure.md?tabs=azure-cli#configure-the-streaming-export-of-diagnostic-telemetry), la [API REST de Azure Monitor](/rest/api/monitor/diagnosticsettings) o [plantillas de Resource Manager](../../azure-monitor/essentials/resource-manager-diagnostic-settings.md).

### <a name="create-an-azure-sql-analytics-resource"></a>Creación de un recurso de Azure SQL Analytics

1. Busque Azure SQL Analytics en Azure Marketplace y selecciónelo.

   ![Busque Azure SQL Analytics en el portal.](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/sql-analytics-in-marketplace.png)

2. Seleccione **Crear** en la pantalla de información general de la solución.

3. Rellene el formulario de Azure SQL Analytics con la información adicional necesaria: nombre del área de trabajo, suscripción, grupo de recursos, ubicación y plan de tarifa.

   ![Configuración de Azure SQL Analytics en el portal](./media/metrics-diagnostic-telemetry-logging-streaming-export-configure/sql-analytics-configuration-blade.png)

4. Seleccione **Aceptar** para confirmar y después **Crear**.

### <a name="configure-the-resource-to-record-metrics-and-resource-logs"></a>Configuración del recurso para registrar métricas y registros de recursos

Debe configurar por separado el streaming de datos de telemetría de diagnóstico para bases de datos únicas y agrupadas, grupos elásticos, instancias administradas y bases de datos de instancia. La manera más sencilla de configurar la ubicación en que un recurso registra las métricas es mediante Azure Portal. Para obtener más información, consulte la sección [Configuración de la exportación de streaming de los datos de telemetría de diagnóstico](metrics-diagnostic-telemetry-logging-streaming-export-configure.md?tabs=azure-portal#configure-the-streaming-export-of-diagnostic-telemetry).

### <a name="use-azure-sql-analytics-for-monitoring-and-alerting"></a>Uso de Azure SQL Analytics para la supervisión y las alertas

SQL Analytics se puede usar como panel jerárquico para ver los recursos de bases de datos.

- Para aprender a utilizar Azure SQL Analytics, consulte [Supervisión mediante SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Para aprender a configurar alertas en SQL Analytics, consulte [Creación de alertas para base de datos, grupos elásticos e instancias administradas](../../azure-monitor/insights/azure-sql.md#analyze-data-and-create-alerts).

## <a name="stream-into-event-hubs"></a>Transmisión a Event Hubs

Puede transmitir las métricas y los registros de recursos de Azure SQL Database e Instancia administrada de Azure SQL a Event Hubs mediante el uso de la opción **Transmitir a un centro de eventos** integrada en Azure Portal. También puede habilitar el identificador de regla de Service Bus mediante una configuración de diagnóstico a través de los cmdlets de PowerShell, la CLI de Azure o la API REST de Azure Monitor. Asegúrese de que el centro de eventos esté en la misma región que la base de datos y el servidor.

### <a name="what-to-do-with-metrics-and-resource-logs-in-event-hubs"></a>Utilidad de las métricas y los registros de recursos en Event Hubs

Una vez que los datos seleccionados se transmiten a los Event Hubs, está un paso más cerca de habilitar escenarios de supervisión avanzados. Los centros de Event Hubs actúan como la puerta de entrada de una canalización de eventos. Una vez que los datos se recopilan en un centro de eventos, se pueden transformar y almacenar con un proveedor de análisis en tiempo real o un adaptador de almacenamiento. Los centros de Event Hubs desacoplan la producción de un flujo de eventos del consumo de esos eventos. De esta manera, los consumidores de eventos pueden tener acceso a los eventos en su propia programación. Para obtener más información sobre Event Hubs, consulte:

- [¿Qué es Azure Event Hubs?](../../event-hubs/event-hubs-about.md)
- [Introducción a Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)

Puede usar las métricas transmitidas en Event Hubs para:

- **Visualización del estado del servicio mediante la transmisión de datos de ruta de acceso activa a Power BI**

  Con Event Hubs, Stream Analytics y Power BI, puede transformar fácilmente las métricas y los datos de diagnóstico en información prácticamente en tiempo real sobre los servicios de Azure. Para obtener información general sobre cómo configurar un centro de Event Hubs, procesar datos con Stream Analytics y usar Power BI como salida, vea [Stream Analytics y Power BI](../../stream-analytics/stream-analytics-power-bi-dashboard.md).

- **Transmisión de registros a registros de terceros y flujos de telemetría**

  Con la transmisión de Event Hubs puede enviar las métricas y los registros de recursos a distintas soluciones de supervisión y análisis de registros de terceros.

- **Creación de una plataforma personalizada de registro y telemetría**

  ¿Tiene ya una plataforma de telemetría personalizada o está pensando en crear una? La naturaleza altamente escalable de publicación y suscripción de Event Hubs le permite ingerir métricas y registros de recursos de forma flexible. Vea la [guía de Dan Rosanova para usar Event Hubs en una plataforma de telemetría de escala global](https://azure.microsoft.com/documentation/videos/build-2015-designing-and-sizing-a-global-scale-telemetry-platform-on-azure-event-Hubs/).

## <a name="stream-into-azure-storage"></a>Transmisión a Azure Storage

Puede almacenar las métricas y los registros de recursos en Azure Storage mediante la opción **Archivar en una cuenta de almacenamiento** integrada en Azure Portal. Storage también se puede habilitar mediante la configuración del diagnóstico a través de los cmdlets de PowerShell, la CLI de Azure o la API REST de Azure Monitor.

### <a name="schema-of-metrics-and-resource-logs-in-the-storage-account"></a>Esquema de métricas y registros de recursos en la cuenta de almacenamiento

Una vez que ha configurado la recopilación de métricas y los registros de recursos, se crea un contenedor de almacenamiento en la cuenta de almacenamiento seleccionada cuando las primeras filas de datos están disponibles. La estructura de los blobs es:

```powershell
insights-{metrics|logs}-{category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/ RESOURCEGROUPS/{resource group name}/PROVIDERS/Microsoft.SQL/servers/{resource_server}/ databases/{database_name}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

O, sencillamente:

```powershell
insights-{metrics|logs}-{category name}/resourceId=/{resource Id}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

Por ejemplo, un nombre de blob para las métricas básicas podría ser:

```powershell
insights-metrics-minute/resourceId=/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.SQL/ servers/Server1/databases/database1/y=2016/m=08/d=22/h=18/m=00/PT1H.json
```

Un nombre de blob para almacenar datos desde un grupo elástico tiene el aspecto siguiente:

```powershell
insights-{metrics|logs}-{category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/ RESOURCEGROUPS/{resource group name}/PROVIDERS/Microsoft.SQL/servers/{resource_server}/ elasticPools/{elastic_pool_name}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

## <a name="data-retention-policy-and-pricing"></a>Directiva de retención de datos y precios

Si selecciona Event Hubs o una cuenta de almacenamiento, puede especificar una directiva de retención. Esta directiva elimina los datos anteriores a un período de tiempo seleccionado. Si especifica Log Analytics, la directiva de retención depende del plan de tarifa seleccionado. En este caso, las unidades gratuitas de ingesta de datos especificadas pueden habilitar la supervisión gratuita de varias bases de datos al mes. Todo uso en exceso de la telemetría de diagnóstico de las unidades gratuitas puede suponer costos.

> [!IMPORTANT]
> Las bases de datos activas con cargas de trabajo más pesadas ingieren más datos que las bases de datos inactivas. Para más información, vea [Precios de análisis de registros](https://azure.microsoft.com/pricing/details/monitor/).

Si usa Azure SQL Analytics, puede supervisar el consumo de la ingesta de datos. Para ello, seleccione el **área de trabajo OMS** en el menú de navegación de Azure SQL Analytics y luego elija **Uso** y **Costos estimados**.

## <a name="metrics-and-logs-available"></a>Métricas y registros disponibles

La telemetría de supervisión disponible para las bases de datos únicas, las bases de datos agrupadas, los grupos elásticos, las instancias administradas y las bases de datos de instancia se documenta en esta sección del artículo. Los datos de telemetría de supervisión recopilados en SQL Analytics pueden usarse para realizar su propio análisis personalizado y desarrollo de aplicaciones mediante el lenguaje de [consultas de registro de Azure Monitor](../../azure-monitor/logs/get-started-queries.md).

### <a name="basic-metrics"></a>Métricas básicas

Vea las siguientes tablas para más detalles sobre todas las métricas básicas por recurso.

> [!NOTE]
> La opción Métricas básicas se conocía anteriormente como Todas las métricas. El cambio se realizó solo en el nombre y no se produjo ningún cambio en las métricas supervisadas. Este cambio se inició para permitir la introducción de las categorías de métricas adicionales en el futuro.

#### <a name="basic-metrics-for-elastic-pools"></a>Métricas básicas para grupos elásticos

|**Recurso**|**Métricas**|
|---|---|
|Grupo elástico|Porcentaje de eDTU; eDTU usada; límite de eDTU; porcentaje de CPU; porcentaje de lectura de datos físicos; porcentaje de escritura en registro; porcentaje de sesiones; porcentaje de trabajos; almacenamiento; porcentaje de almacenamiento; límite de almacenamiento y porcentaje de almacenamiento de XTP |

#### <a name="basic-metrics-for-single-and-pooled-databases"></a>Métrica básica para bases de datos únicas y agrupadas

|**Recurso**|**Métricas**|
|---|---|
|Base de datos única y agrupada|Porcentaje de DTU; DTU usada; límite de DTU; porcentaje de CPU; porcentaje de lectura de datos físicos; porcentaje de escritura en registro; conexiones correctas, erróneas o bloqueadas por el firewall; porcentaje de sesiones; porcentaje de trabajos; almacenamiento; porcentaje de almacenamiento; porcentaje de almacenamiento de XTP e interbloqueos |

### <a name="advanced-metrics"></a>Métricas avanzadas

Consulte la siguiente tabla para ver los detalles de las métricas avanzadas.

|**Métrica**|**Nombre de métrica para mostrar**|**Descripción**|
|---|---|---|
|sqlserver_process_core_percent<sup>1</sup>|Porcentaje de núcleos de proceso de SQL|Porcentaje de uso de la CPU para el proceso de SQL, medido por el sistema operativo.|
|sqlserver_process_memory_percent<sup>1</sup> |Porcentaje de memoria de proceso de SQL|Porcentaje de uso de la memoria para el proceso de SQL, medido por el sistema operativo.|
|tempdb_data_size<sup>2</sup>| Kilobytes de tamaño de archivo de datos Tempdb |Kilobytes de tamaño de archivo de datos Tempdb.|
|tempdb_log_size<sup>2</sup>| Kilobytes de tamaño de archivo de registro Tempdb |Kilobytes de tamaño de archivo de registro Tempdb.|
|tempdb_log_used_percent<sup>2</sup>| Porcentaje de registro de tempdb usado |Porcentaje de registro de tempdb usado.|

<sup>1</sup> Esta métrica está disponible para las bases de datos que usan el modelo de compra de núcleos virtuales con 2 núcleos virtuales, o más, o 200 DTU, o más, para los modelos de compra basados en DTU.

<sup>2</sup> Esta métrica está disponible para las bases de datos que usan el modelo de compra de núcleos virtuales con 2 núcleos virtuales, o más, o 200 DTU, o más, para los modelos de compra basados en DTU. Esta métrica no está disponible actualmente para bases de datos de hiperescala o almacenamientos de datos.

### <a name="basic-logs"></a>Registros básicos

Los detalles de los datos de telemetría disponibles para todos los registros se documentan en las tablas siguientes. Para obtener más información, consulte los [datos de telemetría de diagnóstico admitidos](#diagnostic-telemetry-for-export).

#### <a name="resource-usage-stats-for-managed-instances"></a>Estadísticas de uso de recursos para instancias administradas

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure|
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: ResourceUsageStats |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: MANAGEDINSTANCES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre de la instancia administrada |
|ResourceId|URI de recurso |
|SKU_s|SKU del producto de Instancia administrada de SQL |
|virtual_core_count_s|Número de núcleos virtuales disponibles |
|avg_cpu_percent_s|Porcentaje de CPU medio |
|reserved_storage_mb_s|Capacidad de almacenamiento reservado en la instancia administrada |
|storage_space_used_mb_s|Almacenamiento usado en la instancia administrada |
|io_requests_s|Recuento de IOPS |
|io_bytes_read_s|Bytes de IOPS leídos |
|io_bytes_written_s|Bytes de IOPS escritos |

#### <a name="query-store-runtime-statistics"></a>Estadísticas de tiempo de ejecución del Almacén de consultas

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: QueryStoreRuntimeStatistics |
|OperationName|Nombre de la operación. Siempre: QueryStoreRuntimeStatisticsEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|query_hash_s|Hash de consulta |
|query_plan_hash_s|Hash del plan de consulta |
|statement_sql_handle_s|Controlador de SQL de instrucción |
|interval_start_time_d|Inicio datetimeoffset del intervalo en número de tics de 1900-1-1 |
|interval_end_time_d|Fin datetimeoffset del intervalo en número de tics de 1900-1-1 |
|logical_io_writes_d|Número total de escrituras de E/S lógicas |
|max_logical_io_writes_d|Número máximo de escrituras de E/S lógicas por ejecución |
|physical_io_reads_d|Número total de lecturas de E/S físicas |
|max_physical_io_reads_d|Número máximo de lecturas de E/S lógicas por ejecución |
|logical_io_reads_d|Número total de lecturas de E/S lógicas |
|max_logical_io_reads_d|Número máximo de lecturas de E/S lógicas por ejecución |
|execution_type_d|Tipo de ejecución |
|count_executions_d|Número de ejecuciones de la consulta |
|cpu_time_d|Tiempo total de la CPU usado por la consulta en microsegundos |
|max_cpu_time_d|Consumidor de tiempo máximo de la CPU por una sola ejecución en microsegundos |
|dop_d|Suma de los grados de paralelismo |
|max_dop_d|Grado máximo de paralelismo que se usa para una sola ejecución |
|rowcount_d|Número total de filas devueltas |
|max_rowcount_d|Número máximo de filas devueltas en una sola ejecución |
|query_max_used_memory_d|Cantidad total de memoria usada en KB |
|max_query_max_used_memory_d|Cantidad máxima de memoria usada por una sola ejecución en KB |
|duration_d|Tiempo de ejecución total en milisegundos |
|max_duration_d|Tiempo de ejecución máximo de una sola ejecución |
|num_physical_io_reads_d|Número total de lecturas físicas |
|max_num_physical_io_reads_d|Número máximo de lecturas físicas por ejecución |
|log_bytes_used_d|Cantidad total de bytes de registro usados |
|max_log_bytes_used_d|Cantidad máxima de bytes de registro usados por ejecución |
|query_id_d|Identificador de la consulta en el Almacén de consultas |
|plan_id_d|Identificador del plan en el Almacén de consultas |

Obtenga más información sobre los [datos de estadísticas de tiempo de ejecución del Almacén de consultas](/sql/relational-databases/system-catalog-views/sys-query-store-runtime-stats-transact-sql).

#### <a name="query-store-wait-statistics"></a>Estadísticas de espera del Almacén de consultas

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: QueryStoreWaitStatistics |
|OperationName|Nombre de la operación. Siempre: QueryStoreWaitStatisticsEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|wait_category_s|Categoría de la espera |
|is_parameterizable_s|La consulta se puede parametrizar |
|statement_type_s|Tipo de la instrucción |
|statement_key_hash_s|Hash de clave de instrucción |
|exec_type_d|Tipo de ejecución |
|total_query_wait_time_ms_d|Tiempo total de espera de la consulta en la categoría de espera específica |
|max_query_wait_time_ms_d|Tiempo de espera máximo de la consulta en ejecución individual en la categoría de espera específica |
|query_param_type_d|0 |
|query_hash_s|Hash de consulta del Almacén de consultas |
|query_plan_hash_s|Hash de plan de consulta del Almacén de consultas |
|statement_sql_handle_s|Controlador de instrucción del Almacén de consultas |
|interval_start_time_d|Inicio datetimeoffset del intervalo en número de tics de 1900-1-1 |
|interval_end_time_d|Fin datetimeoffset del intervalo en número de tics de 1900-1-1 |
|count_executions_d|Número de ejecuciones de la consulta. |
|query_id_d|Identificador de la consulta en el Almacén de consultas |
|plan_id_d|Identificador del plan en el Almacén de consultas |

Obtenga más información sobre los [datos de estadísticas de espera del Almacén de consultas](/sql/relational-databases/system-catalog-views/sys-query-store-wait-stats-transact-sql).

#### <a name="errors-dataset"></a>Conjunto de datos de errores

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: Errors |
|OperationName|Nombre de la operación. Siempre: ErrorEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|Message|Mensaje de error en texto sin formato. |
|user_defined_b|Es el bit de error definido por el usuario. |
|error_number_d|Código de error |
|severity|Gravedad del error |
|state_d|Estado del error |
|query_hash_s|Hash de consulta de la consulta errónea, si está disponible |
|query_plan_hash_s|Hash de plan de consulta de la consulta errónea, si está disponible |

Obtenga más información sobre los [mensajes de error de SQL](/sql/relational-databases/errors-events/database-engine-events-and-errors).

#### <a name="database-wait-statistics-dataset"></a>Conjunto de datos de estadísticas de espera de base de datos

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: DatabaseWaitStatistics |
|OperationName|Nombre de la operación. Siempre: DatabaseWaitStatisticsEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|wait_type_s|Nombre del tipo de espera |
|start_utc_date_t [UTC]|Hora de inicio del período medido |
|end_utc_date_t [UTC]|Hora de finalización del período medido |
|delta_max_wait_time_ms_d|Tiempo máximo esperado por ejecución |
|delta_signal_wait_time_ms_d|Tiempo de espera de señales total |
|delta_wait_time_ms_d|Tiempo total de espera en el período |
|delta_waiting_tasks_count_d|Número de tareas en espera |

Obtenga más información sobre las [estadísticas de espera de la base de datos](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql).

#### <a name="time-outs-dataset"></a>Conjunto de datos de los tiempos de espera

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: Tiempos de expiración |
|OperationName|Nombre de la operación. Siempre: TimeoutEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|error_state_d|Código de estado de error |
|query_hash_s|Hash de consulta si está disponible |
|query_plan_hash_s|Hash de plan de consulta si está disponible |

#### <a name="blockings-dataset"></a>Conjunto de datos de bloqueos

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: Blocks |
|OperationName|Nombre de la operación. Siempre: BlockEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|lock_mode_s|Modo de bloqueo usado por la consulta |
|resource_owner_type_s|Propietario del bloqueo |
|blocked_process_filtered_s|Archivo XML de informe del proceso bloqueado |
|duration_d|Duración del bloqueo en microsegundos |

#### <a name="deadlocks-dataset"></a>Conjunto de datos de interbloqueos

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC] |Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: Interbloqueos |
|OperationName|Nombre de la operación. Siempre: DeadlockEvent |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|deadlock_xml_s|Informe XML de interbloqueo |

#### <a name="automatic-tuning-dataset"></a>Conjunto de datos de ajuste automático

|Propiedad|Descripción|
|---|---|
|TenantId|El identificador de inquilino |
|SourceSystem|Siempre: Azure |
|TimeGenerated [UTC]|Marca de tiempo de cuándo se registró el registro |
|Tipo|Siempre: AzureDiagnostics |
|ResourceProvider|Nombre del proveedor de recursos Siempre: MICROSOFT.SQL |
|Category|Nombre de la categoría Siempre: AutomaticTuning |
|Recurso|Nombre del recurso |
|ResourceType|Nombre del tipo de recurso Siempre: SERVERS/DATABASES |
|SubscriptionId|GUID de la suscripción para la base de datos |
|ResourceGroup|Nombre del grupo de recursos de la base de datos |
|LogicalServerName_s|Nombre del servidor de la base de datos |
|LogicalDatabaseName_s|Nombre de la base de datos |
|ElasticPoolName_s|Nombre del grupo elástico de la base de datos, si lo hay |
|DatabaseName_s|Nombre de la base de datos |
|ResourceId|URI de recurso |
|RecommendationHash_s|Código hash único de la recomendación de ajuste automático |
|OptionName_s|Operación de ajuste automático |
|Schema_s|Esquema de la base de datos |
|Table_s|Tabla afectada |
|IndexName_s|Nombre del índice |
|IndexColumns_s|Nombre de la columna |
|IncludedColumns_s|Columnas incluidas |
|EstimatedImpact_s|Impacto estimado del formato JSON de la recomendación de ajuste automático |
|Event_s|Tipo de evento de ajuste automático |
|Timestamp_t|Última marca de tiempo actualizada |

#### <a name="intelligent-insights-dataset"></a>Conjunto de datos de Intelligent Insights

Obtenga más información sobre el [formato de registro de Intelligent Insights](intelligent-insights-use-diagnostics-log.md).

## <a name="next-steps"></a>Pasos siguientes

Para aprender a habilitar el registro y comprender las métricas y las categorías de registro admitidas por los diferentes servicios de Azure, vea lo siguiente:

- [Información general sobre las métricas en Microsoft Azure](../../azure-monitor/data-platform.md)
- [Introducción a los registros de plataforma Azure](../../azure-monitor/essentials/platform-logs-overview.md)

Para obtener información sobre Event Hubs, lea lo siguiente:

- [¿Qué es Azure Event Hubs?](../../event-hubs/event-hubs-about.md)
- [Introducción a Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)

Para aprender a configurar alertas basadas en la telemetría de Log Analytics, consulte:

- [Creación de alertas para Azure SQL Database e Instancia administrada de Azure SQL](../../azure-monitor/insights/azure-sql.md#analyze-data-and-create-alerts)
