---
title: Supervisión de las métricas del servidor de Azure Analysis Services | Microsoft Docs
description: Obtenga información sobre cómo Analysis Services usa el Explorador de métricas de Azure, una herramienta gratuita del portal, que permiten supervisar el rendimiento y el estado de los servidores.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 03/04/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 3991eb421f42ec6645e3321d3a624e226fd12c67
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107314259"
---
# <a name="monitor-server-metrics"></a>Supervisión de las métricas del servidor

Analysis Services proporciona métricas en el Explorador de métricas de Azure, una herramienta gratuita del portal, que permiten supervisar el rendimiento y el estado de los servidores. Por ejemplo, puede supervisar la memoria y el uso de CPU, el número de conexiones de cliente y el consumo de recursos de consulta. Analysis Services utiliza el mismo marco de supervisión que la mayoría de los demás servicios de Azure. Para más información, vea [Introducción al Explorador de métricas de Azure.](../azure-monitor/essentials/metrics-getting-started.md)

Para realizar diagnósticos más detallados, hacer un seguimiento del rendimiento e identificar tendencias en diferentes recursos de servicio de un grupo de recursos o de una suscripción, use [Azure Monitor](../azure-monitor/overview.md). Es posible que se apliquen cargos por el servicio Azure Monitor.


## <a name="to-monitor-metrics-for-an-analysis-services-server"></a>Para supervisar las métricas de un servidor de Analysis Services

1. En Azure Portal, seleccione **Métricas**.

    ![Monitor en Azure Portal](./media/analysis-services-monitor/aas-monitor-portal.png)

2. En **Métrica**, seleccione las métricas que desee incluir en el gráfico. 

    ![Gráfico de Monitor](./media/analysis-services-monitor/aas-monitor-chart.png)

<a id="#server-metrics"></a>

## <a name="server-metrics"></a>Métricas del servidor

Utilice esta tabla para determinar qué métricas son más adecuadas para su escenario de supervisión. Las métricas deben utilizar la misma unidad para poder aparecer en el mismo gráfico.

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|
|---|---|---|---|---|
|CommandPoolJobQueueLength|Longitud de cola de trabajos de grupo de comandos|Count|Average|Número de trabajos en la cola del grupo de subprocesos de comandos.|
|CurrentConnections|Conexión: Conexiones actuales|Count|Average|Número actual de conexiones de cliente establecidas.|
|CurrentUserSessions|Sesiones de usuario actuales|Count|Average|Número actual de sesiones de usuario establecidas.|
|mashup_engine_memory_metric|Memoria del motor M|Bytes|Average|Uso de memoria por los procesos del motor de mashup|
|mashup_engine_qpu_metric|QPU de motor M|Count|Average|Uso de QPU por los procesos del motor de mashup|
|memory_metric|Memoria|Bytes|Average|Memoria. Intervalo de 0-25 GB para S1, 0-50 GB para S2 y 0-100 GB para S4|
|memory_thrashing_metric|Paginación excesiva de memoria|Percent|Average|Paginación excesiva media de memoria.|
|CleanerCurrentPrice|Memoria: precio actual de limpieza|Count|Average|Precio actual de memoria, USD/byte/tiempo, normalizado a 1000.|
|CleanerMemoryNonshrinkable|Memoria: memoria de limpieza no reducible|Bytes|Average|Cantidad de memoria, en bytes, que no purgará el limpiador en segundo plano.|
|CleanerMemoryShrinkable|Memoria: memoria de limpieza reducible|Bytes|Average|Cantidad de memoria, en bytes, que purgará el limpiador en segundo plano.|
|MemoryLimitHard|Memoria: límite de memoria física|Bytes|Average|Límite de memoria física del archivo de configuración.|
|MemoryLimitHigh|Memoria: límite alto de memoria|Bytes|Average|Límite alto de memoria del archivo de configuración.|
|MemoryLimitLow|Memoria: límite bajo de memoria|Bytes|Average|Límite bajo de memoria del archivo de configuración.|
|MemoryLimitVertiPaq|Memoria: VertiPaq de límite de memoria|Bytes|Average|Límite en memoria del archivo de configuración.|
|MemoryUsage|Memoria: Uso de la memoria|Bytes|Average|Uso de memoria del proceso de servidor tal como se usa para calcular el precio de la memoria del limpiador. Es igual que el contador Process\PrivateBytes más el tamaño de los datos asignados a la memoria; no se tiene en cuenta ninguna memoria asignada por el sistema de análisis en memoria (VertiPaq) que supere el límite de memoria del sistema.|
|private_bytes_metric|Bytes privados |Bytes|Average|La cantidad total de memoria que el proceso del motor de Analysis Services y los procesos de contenedor de Mashup han asignado, sin incluir la memoria compartida con otros procesos.|
|virtual_bytes_metric|Bytes virtuales |Bytes|Average|Tamaño actual del espacio de direcciones virtuales que están usando el proceso del motor de Analysis Services y los procesos de contenedor de Mashup.|
|mashup_engine_private_bytes_metric|Bytes privados del motor M |Bytes|Average|Cantidad total de procesos de contenedor de mashup de memoria asignados, sin incluir la memoria compartida con otros procesos.|
|mashup_engine_virtual_bytes_metric|Bytes virtuales del motor M |Bytes|Average|Tamaño actual del espacio de direcciones virtuales que están usando los proceso de contenedor de Mashup.|
|Quota|Memoria: Quota|Bytes|Average|Cuota de memoria actual, en bytes. La cuota de memoria también se denomina concesión de memoria o reserva de memoria.|
|QuotaBlocked|Memoria: cuota bloqueada|Count|Average|Número actual de solicitudes de cuota que están bloqueadas hasta que se liberen otras cuotas de memoria.|
|VertiPaqNonpaged|Memoria: VertiPaq no paginado|Bytes|Average|Bytes de memoria bloqueados en el espacio de trabajo para que los use el motor en memoria.|
|VertiPaqPaged|Memoria: VertiPaq paginado|Bytes|Average|Bytes de memoria paginada en uso para datos en memoria.|
|ProcessingPoolJobQueueLength|Longitud de cola de trabajos de grupo de procesamiento|Count|Average|Número de trabajos que no son de E/S en la cola del grupo de subprocesos de procesamiento.|
|RowsConvertedPerSec|Procesamiento: filas convertidas por segundo|CountPerSecond|Average|Velocidad de filas convertidas durante el procesamiento.|
|RowsReadPerSec|Procesamiento: filas leídas por segundo|CountPerSecond|Average|Velocidad de filas leídas de todas las bases de datos relacionales.|
|RowsWrittenPerSec|Procesamiento: filas escritas por segundo|CountPerSecond|Average|Velocidad de filas escritas durante el procesamiento.|
|qpu_metric|QPU|Count|Average|QPU. Intervalo de 0-100 para S1, 0-200 para S2 y 0-400 para S4|
|QueryPoolBusyThreads|Subprocesos ocupados de grupo de consultas|Count|Average|Número de subprocesos ocupados del grupo de subprocesos de consulta.|
|SuccessfullConnectionsPerSec|Conexiones correctas por segundo|CountPerSecond|Average|Tasa de finalizaciones de conexión correctas.|
|CommandPoolBusyThreads|Subprocesos: subprocesos ocupados del grupo de comandos|Count|Average|Número de subprocesos ocupados del grupo de subprocesos de comandos.|
|CommandPoolIdleThreads|Subprocesos: subprocesos inactivos del grupo de comandos|Count|Average|Número de subprocesos inactivos del grupo de subprocesos de comandos.|
|LongParsingBusyThreads|Subprocesos: subprocesos ocupados en análisis largos|Count|Average|Número de subprocesos ocupados del grupo de subprocesos de análisis largos.|
|LongParsingIdleThreads|Subprocesos: subprocesos inactivos en análisis largos|Count|Average|Número de subprocesos inactivos del grupo de subprocesos en análisis largos.|
|LongParsingJobQueueLength|Subprocesos: longitud de cola de trabajos en análisis largos|Count|Average|Número de trabajos en la cola del grupo de subprocesos en análisis largos.|
|ProcessingPoolIOJobQueueLength|Subprocesos: Longitud de cola de trabajos de E/S de grupo de procesamiento|Count|Average|Número de trabajos de E/S en la cola del grupo de subprocesos de procesamiento.|
|ProcessingPoolBusyIOJobThreads|Subprocesos: subprocesos de trabajo de E/S ocupados del grupo de procesamiento|Count|Average|Número de subprocesos que ejecutan trabajos de E/S en el grupo de subprocesos de procesamiento.|
|ProcessingPoolBusyNonIOThreads|Subprocesos: subprocesos de trabajo ocupados que no son de E/S del grupo de procesamiento|Count|Average|Número de subprocesos que ejecutan trabajos que no son de E/S del grupo de subprocesos de procesamiento.|
|ProcessingPoolIdleIOJobThreads|Subprocesos: subprocesos de trabajo de E/S inactivos del grupo de procesamiento|Count|Average|Número de subprocesos inactivos para trabajos de E/S en el grupo de subprocesos de procesamiento.|
|ProcessingPoolIdleNonIOThreads|Subprocesos: subprocesos de trabajo inactivos que no son de E/S del grupo de procesamiento|Count|Average|Número de subprocesos inactivos del grupo de subprocesos de procesamiento dedicado a trabajos que no son de E/S.|
|QueryPoolIdleThreads|Subprocesos: subprocesos inactivos del grupo de consultas|Count|Average|Número de subprocesos inactivos para trabajos de E/S en el grupo de subprocesos de procesamiento.|
|QueryPoolJobQueueLength|Subprocesos: longitud de cola de trabajos del grupo de consultas|Count|Average|Número de trabajos en la cola del grupo de subprocesos de consultas.|
|ShortParsingBusyThreads|Subprocesos: subprocesos ocupados en análisis cortos|Count|Average|Número de subprocesos ocupados del grupo de subprocesos de análisis cortos.|
|ShortParsingIdleThreads|Subprocesos: subprocesos inactivos en análisis cortos|Count|Average|Número de subprocesos inactivos del grupo de subprocesos de análisis cortos.|
|ShortParsingJobQueueLength|Subprocesos: longitud de cola de trabajos en análisis cortos|Count|Average|Número de trabajos en la cola del grupo de subprocesos en análisis cortos.|
|TotalConnectionFailures|Número total de errores de conexión|Count|Average|Número total de intentos de conexión con error.|
|TotalConnectionRequests|Número total de solicitudes de conexión|Count|Average|Número total de solicitudes de conexión. |

## <a name="next-steps"></a>Pasos siguientes
[Introducción a Azure Monitor](../azure-monitor/overview.md)      
[Introducción al Explorador de métricas de Azure](../azure-monitor/essentials/metrics-getting-started.md)      
[Métricas de la API de REST de Azure Monitor](/rest/api/monitor/metrics)
