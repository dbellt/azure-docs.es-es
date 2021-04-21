---
title: Supervisión del estado del área de trabajo de Log Analytics en Azure Monitor
description: Se describe cómo supervisar el estado del área de trabajo de Log Analytics con los datos de la tabla Operation (Operación).
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/20/2020
ms.openlocfilehash: 6f1a23170d84e39e5d531ae4e3a64b59d29bd677
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538855"
---
# <a name="monitor-health-of-log-analytics-workspace-in-azure-monitor"></a>Supervisión del estado del área de trabajo de Log Analytics en Azure Monitor
Para mantener el rendimiento y la disponibilidad del área de trabajo de Log Analytics en Azure Monitor, es necesario detectar con antelación los problemas que surjan. En este artículo se describe cómo supervisar el estado del área de trabajo de Log Analytics con los datos de la tabla [Operation](/azure/azure-monitor/reference/tables/operation) (Operación). Esta tabla se incluye en todas las áreas de trabajo de Log Analytics y contiene los errores y las advertencias que se producen en el área de trabajo. Revise periódicamente estos datos y cree alertas para recibir notificaciones de forma anticipada cuando haya incidentes importantes en el área de trabajo.

## <a name="_logoperation-function"></a>Función _LogOperation

Los registros de Azure Monitor envían detalles sobre cualquier problema a la tabla [Operation](/azure/azure-monitor/reference/tables/operation) (Operación) del área de trabajo donde se produjo el problema. La función **_LogsOperation** del sistema se basa en la tabla **Operation** (Operación) y proporciona un conjunto simplificado de información para el análisis y la generación de alertas.

## <a name="columns"></a>Columnas

La función **_LogsOperation** devuelve las columnas de la tabla siguiente.

| Columna | Descripción |
|:---|:---|
| TimeGenerated | Hora a la que se produjo el incidente en UTC. |
| Category  | Grupo de categorías de operaciones. Se puede usar para filtrar los tipos de operaciones y ayudar a crear alertas y auditorías del sistema más precisas. Consulte la sección siguiente para ver una lista de categorías. |
| Operación  | Descripción del tipo de operación. Puede indicar uno de los límites Log Analytics, el tipo de operación o parte de un proceso. |
| Nivel | Nivel de gravedad del problema:<br>- Info (Info.): no precisa atención específica.<br>- Warning (Advertencia): no se completó el proceso como se esperaba y precisa atención.<br>- Error: error en el proceso y precisa atención urgente. 
| Detalle | La descripción detallada de la operación incluye un mensaje de error específico, si existe. |
| _ResourceId | Identificador de recurso del recurso de Azure relacionado con la operación.  |
| Computer | Nombre del equipo si la operación está relacionada con un agente de Azure Monitor. |
| CorrelationId | Se utiliza para agrupar operaciones relacionadas consecutivas. |


## <a name="categories"></a>Categorías

En la tabla siguiente se describen las categorías de la función _LogOperation. 

| Category | Descripción |
|:---|:---|
| Ingesta de datos           | Operaciones que forman parte del proceso de ingesta de datos. Consulte a continuación para más información. |
| Agente               | Indica un problema con la instalación del agente. |
| datos, recopilación     | Operaciones relacionadas con los procesos de recopilaciones de datos. |
| Destino de la solución  | Se procesó la operación de tipo *ConfigurationScope*. |
| Solución de evaluación | Se ejecutó un proceso de evaluación. |


### <a name="ingestion"></a>Ingesta de datos
Las operaciones de ingesta son problemas que se produjeron durante la ingesta de datos, como la notificación de haberse alcanzado los límites del área de trabajo de Azure Log Analytics. Las condiciones de error de esta categoría pueden sugerir una pérdida de datos, por lo que es especialmente importante supervisarlas. En la tabla siguiente se proporcionan detalles sobre estas operaciones. Consulte [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) para conocer los límites de servicio de las áreas de trabajo de Log Analytics.


| Operación | Nivel | Detalle | Artículo relacionado |
|:---|:---|:---|:---|
| Registro personalizado | Error   | Se alcanzó el límite de columnas de campos personalizados. | [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Registro personalizado | Error   | Error de ingesta de registros personalizados. | |
| Metadatos. | Error | Se detectó un error de configuración. | |
| datos, recopilación | Error   | Se descartaron los datos porque la solicitud se creó con anterioridad al número de días establecido. | [Administrar el uso y los costos con los registros de Azure Monitor](./manage-cost-storage.md#alert-when-daily-cap-reached)
| datos, recopilación | Información    | Se detectó la configuración de la máquina de recopilación.| |
| datos, recopilación | Información    | Recopilación de datos iniciada debido a un nuevo día. | [Administrar el uso y los costos con los registros de Azure Monitor](./manage-cost-storage.md#alert-when-daily-cap-reached) |
| datos, recopilación | Advertencia | Recopilación de datos detenida debido a que se alcanzó el límite diario.| [Administrar el uso y los costos con los registros de Azure Monitor](./manage-cost-storage.md#alert-when-daily-cap-reached) |
| Procesamiento de datos | Error   | Formato JSON no válido. | [Envío de datos de registro a Azure Monitor con HTTP Data Collector API (versión preliminar pública)](../logs/data-collector-api.md#request-body) | 
| Procesamiento de datos | Advertencia | El valor se ha recortado al tamaño máximo permitido. | [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Procesamiento de datos | Advertencia | Se ha recortado el valor de campo cuando se alcanzó el tamaño límite. | [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) | 
| Tasa de ingesta | Información | Límite de tasa de ingesta aproximándose al 70 %. | [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Tasa de ingesta | Advertencia | Límite de tasa de ingesta aproximándose al límite. | [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Tasa de ingesta | Error   | Límite de tasa alcanzado. | [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) |
| Storage | Error   | No se puede acceder a la cuenta de almacenamiento, ya que las credenciales usadas no son válidas.  |



   

## <a name="alert-rules"></a>Las reglas de alertas
Use [alertas de consulta de registro](../alerts/alerts-log-query.md) de Azure Monitor para que se le notifique de forma anticipada cuando se detecte un problema en el área de trabajo de Log Analytics. Use una estrategia que le permita responder de forma oportuna a los problemas y, al mismo tiempo, reducir los costos. La suscripción se cobra por cada regla de alerta, con un costo que depende de la frecuencia con que se evalúe.

Una estrategia recomendada es comenzar con dos reglas de alerta en función del nivel del problema. Use una frecuencia corta, cada 5 minutos para errores, por ejemplo, y una frecuencia más larga, 24 horas para advertencias, por ejemplo. Dado que los errores indican una posible pérdida de datos, querrá responder a ellos rápidamente para minimizar cualquier pérdida. Las advertencias suelen indicar un problema que no requiere atención inmediata, por lo que puede revisarlas diariamente.

Use el proceso que se describe en [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../alerts/alerts-log.md) para crear las reglas de alerta de registro. En las secciones siguientes se describen los detalles de cada regla.


| Consultar | Valor del umbral | Período | Frecuencia |
|:---|:---|:---|:---|
| `_LogOperation | where Level == "Error"`   | 0 | 5 | 5 |
| `_LogOperation | where Level == "Warning"` | 0 | 1440 | 1440 |

Estas reglas de alerta responderán igual a todas las operaciones con error o advertencia. A medida que se familiarice con las operaciones que generan alertas, puede que quiera responder de forma diferente a operaciones concretas. Por ejemplo, puede que quiera enviar notificaciones a distintas personas con determinadas operaciones. 

Para crear una regla de alerta para una operación específica, use una consulta que incluya las columnas **Category** (Categoría) y **Operation** (Operación). 

En el ejemplo siguiente se crea una alerta de advertencia cuando la tasa de volumen de ingesta alcanza el 80 % del límite.

- Destino: Selección del área de trabajo de Log Analytics
- Criterios:
  - Nombre de señal: Búsqueda de registros personalizada
  - Consulta de búsqueda: `_LogOperation | where Category == "Ingestion" | where Operation == "Ingestion rate" | where Level == "Warning"`
  - Basado en: Número de resultados
  - Condición: Mayor que
  - Umbral: 0
  - Período: 5 (minutos)
  - Frecuencia: 5 (minutos)
- Nombre de regla de alertas: Límite de datos diario alcanzado
- Gravedad: advertencia (gravedad 1)


En el ejemplo siguiente se crea una alerta de advertencia cuando la recopilación de datos ha alcanzado el límite diario. 

- Destino: Selección del área de trabajo de Log Analytics
- Criterios:
  - Nombre de señal: Búsqueda de registros personalizada
  - Consulta de búsqueda: `_LogOperation | where Category == "Ingestion" | where Operation == "Data collection Status" | where Level == "Warning"`
  - Basado en: Número de resultados
  - Condición: Mayor que
  - Umbral: 0
  - Período: 5 (minutos)
  - Frecuencia: 5 (minutos)
- Nombre de regla de alertas: Límite de datos diario alcanzado
- Gravedad: advertencia (gravedad 1)



## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [alertas de registro](../alerts/alerts-log.md).
- [Recopile datos de auditoría de consultas](./query-audit.md) para el área de trabajo.
