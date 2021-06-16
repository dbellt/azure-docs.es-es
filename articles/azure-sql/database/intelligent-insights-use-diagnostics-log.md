---
title: Registro de diagnóstico del rendimiento de Intelligent Insights
description: Intelligent Insights proporciona un registro de diagnóstico de los problemas de rendimiento de Azure SQL Database e Instancia administrada de Azure SQL
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: AlainDormehlMSFT
ms.author: aldorme
ms.reviewer: mathoma, wiassaf
ms.date: 06/12/2020
ms.openlocfilehash: 551886010cb5dc6d1b4977a6687d11704cc7776e
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111590277"
---
# <a name="use-the-intelligent-insights-performance-diagnostics-log-of-azure-sql-database-and-azure-sql-managed-instance-performance-issues"></a>Registro de diagnóstico de rendimiento de Intelligent Insights con los problemas de rendimiento de Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Esta página proporciona información sobre cómo usar el registro de diagnóstico de rendimiento generado por [Intelligent Insights](intelligent-insights-overview.md) con los problemas de rendimiento de Azure SQL Database e Instancia administrada de Azure SQL, el formato y los datos que contiene según sus necesidades de desarrollo personalizadas. Este registro de diagnóstico se puede enviar a [registros de Azure Monitor](../../azure-monitor/insights/azure-sql.md), [Azure Event Hubs](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs), [Azure Storage](metrics-diagnostic-telemetry-logging-streaming-export-configure.md#stream-into-azure-storage) o a una solución de terceros para las funcionalidades personalizadas de informes y alertas de DevOps.

> [!NOTE]
> Intelligent Insights es una característica en versión preliminar que no está disponible en las siguientes regiones: Oeste de Europa, Norte de Europa, Oeste de EE. UU. 1 y Este de EE. UU. 1.

## <a name="log-header"></a>Encabezado de registro

El registro de diagnóstico usa el formato estándar de JSON para generar los resultados de Intelligent Insights. La propiedad de categoría exacta para acceder a un registro de Intelligent Insights es el valor fijo "SQLInsights".

El encabezado del registro es común y consta de la marca de tiempo (TimeGenerated) que muestra cuándo se creó una entrada. También incluye un identificador de recurso (ResourceId) que hace referencia a la base de datos concreta con la que se relaciona la entrada. La categoría (Category), el nivel (Level) y el nombre de la operación (OperationName) son propiedades fijas cuyos valores no cambian. Indican que la entrada del registro es informativa y que provienen de Intelligent Insights (SQLInsights).

```json
"TimeGenerated" : "2017-9-25 11:00:00", // time stamp of the log entry
"ResourceId" : "database identifier", // value points to a database resource
"Category": "SQLInsights", // fixed property
"Level" : "Informational", // fixed property
"OperationName" : "Insight", // fixed property
```

## <a name="issue-id-and-database-affected"></a>Identificador del problema y base de datos afectada

La propiedad de identificación del problema (issueId_d) proporciona un método exclusivo de seguimiento de los problemas de rendimiento hasta su resolución. En el registro que notifica el estado de un mismo problema, varios registros de eventos compartirán el mismo identificador de problema.

Junto con el identificador del problema, el registro de diagnóstico notifica las marcas de tiempo de inicio (intervalStartTime_t) y finalización (intervalEndTme_t) del evento concreto relacionado con un problema notificado en el registro de diagnóstico.

La propiedad de grupo elástico (elasticPoolName_s) indica a qué grupo elástico pertenece la base de datos con un problema. Si la base de datos no forma parte de ningún grupo elástico, esta propiedad no tiene ningún valor. La base de datos en la que se detecta un problema se revela en la propiedad de nombre de la base de datos (databaseName_s).

```json
"intervalStartTime_t": "2017-9-25 11:00", // start of the issue reported time stamp
"intervalEndTme_t":"2017-9-25 12:00", // end of the issue reported time stamp
"elasticPoolName_s" : "", // resource elastic pool (if applicable)
"databaseName_s" : "db_name", // database name
"issueId_d" : 1525, // unique ID of the issue detected
"status_s" : "Active" // status of the issue – possible values: "Active", "Verifying", and "Complete"
```

## <a name="detected-issues"></a>Problemas detectados

La sección siguiente del registro de rendimiento de Intelligent insights contiene problemas de rendimiento que se detectaron mediante la inteligencia artificial integrada. Las detecciones se revelan en las propiedades dentro del registro de diagnóstico JSON. Estas detecciones constan de la categoría de un problema, el impacto del problema, las consultas afectadas y las métricas. Las propiedades de las detecciones podrían contener varios problemas de rendimiento que se detectaron.

Los problemas de rendimiento detectados se notifican con la siguiente estructura de propiedad de las detecciones:

```json
"detections_s" : [{
"impact" : 1 to 3, // impact of the issue detected, possible values 1-3 (1 low, 2 moderate, 3 high impact)
"category" : "Detectable performance pattern", // performance issue detected, see the table
"details": <Details outputted> // details of an issue (see the table)
}]
```

Los patrones de rendimiento detectable y los detalles que se transmiten al registro de diagnóstico se proporcionan en la tabla siguiente.

### <a name="detection-category"></a>Categoría de detección

La propiedad de categoría (category) describe la categoría de los patrones de rendimiento detectable. Consulte la siguiente tabla para ver todas las categorías posibles de patrones de rendimiento detectables. Para más información, consulte [Troubleshoot database performance issues with Intelligent Insights](intelligent-insights-troubleshoot-performance.md) (Solución de problemas de rendimiento de la base de datos con Intelligent Insights).

En función del problema de rendimiento detectado, los detalles que incluye el archivo de registro de diagnóstico difieren en consecuencia.

| Patrones de rendimiento detectables | Detalles generados |
| :------------------- | ------------------- |
| Alcance de los límites de recursos | <li>Recursos afectados</li><li>Códigos hash de consulta</li><li>Porcentaje de consumo del recurso</li> |
| Aumento de la carga de trabajo | <li>Número de consultas cuya ejecución aumentó</li><li>Códigos hash de las consultas que contribuyen más al aumento de la carga de trabajo</li> |
| Presión de memoria | <li>Distribuidor de memoria</li> |
| Bloqueo | <li>Códigos hash de consulta afectados</li><li>Bloqueo de los códigos hash de consulta</li> |
| Aumento de MAXDOP | <li>Códigos hash de consulta</li><li>Tiempos de espera de CXP</li><li>Tiempos de espera</li> |
| Contención de PAGELATCH | <li>Códigos hash de las consultas que provocan la contención</li> |
| Carencia de un índice | <li>Códigos hash de consulta</li> |
| Nueva consulta | <li>Hash de consulta de las nuevas consultas</li> |
| Estadística de espera inusual | <li>Tipos de espera inusuales</li><li>Códigos hash de consulta</li><li>Tiempos de espera de la consulta</li> |
| Contención de TempDB | <li>Códigos hash de las consultas que provocan la contención</li><li>Atribución de consultas al tiempo de espera de contención de PAGELATCH de base de datos general [%]</li> |
| Escasez de DTU en el grupo elástico | <li>Grupo elástico</li><li>Base de datos que más consume DTU</li><li>Porcentaje de DTU de grupo que usa el mayor consumidor</li> |
| Regresión de un plan | <li>Códigos hash de consulta</li><li>Identificadores de planes correctos</li><li>Identificadores de planes inadecuados</li> |
| Cambio del valor de configuración de ámbito de base de datos | <li>Cambios en la configuración de ámbito de base de datos en comparación con los valores predeterminados</li> |
| Cliente lento | <li>Códigos hash de consulta</li><li>Tiempos de espera</li> |
| Degradación del plan de tarifa | <li>Notificación de texto</li> |

### <a name="impact"></a>Impacto

La propiedad de impacto (Impact) describe cuánto ha contribuido un comportamiento detectado al problema que tiene una base de datos. Los impactos van de 1 a 3, siendo 3 la contribución más alta, 2 la moderada y 1 la más baja. El valor de impacto podría usarse como entrada para la automatización personalizada de las alertas, en función de sus necesidades específicas. La propiedad de consultas afectadas (QueryHashes) proporciona una lista de los códigos hash de consulta que se vieron afectados por una detección particular.

### <a name="impacted-queries"></a>Consultas afectadas

En la sección siguiente del registro de Intelligent Insights se proporciona información sobre consultas concretas que resultaron afectadas por los problemas de rendimiento detectados. Esta información se divulga como una matriz de objetos insertados en la propiedad impact_s. La propiedad de impacto consta de entidades y métricas. Las entidades hacen referencia a una consulta determinada (tipo: Consulta). El código hash de consulta única se revela en la propiedad de valor (Value). Además, cada una de las consultas reveladas va seguida de una métrica y un valor, que indican que se detectó un problema de rendimiento.

En el ejemplo de registro siguiente, se detectó que la consulta con el código hash 0x9102EXZ4 tenía una mayor duración de ejecución (Métrica: DurationIncreaseSeconds). El valor de 110 segundos indica que esta consulta en particular tardó 110 segundos más en ejecutarse. Dado que se pueden detectar varias consultas, esta sección del registro en particular podría incluir varias entradas de consulta.

```json
"impact" : [{
"entity" : {
"Type" : "Query", // type of entity - query
"Value" : "query hash value", // for example "0x9102EXZ4" query hash value },
"Metric" : "DurationIncreaseSeconds", // measured metric and the measurement unit (in this case seconds)
"Value" : 110 // value of the measured metric (in this case seconds)
}]
```

### <a name="metrics"></a>Métricas

La unidad de medida de cada métrica notificada se proporciona en virtud de la propiedad de métrica (Metric) con los valores posibles: segundos, número y porcentaje. El valor de una métrica medida se notifica en la propiedad de valor (Value).

La propiedad DurationIncreaseSeconds proporciona la unidad de medida en segundos. La unidad de medida CriticalErrorCount es un número que representa un recuento de errores.

```json
"metric" : "DurationIncreaseSeconds", // issue metric type – possible values: DurationIncreaseSeconds, CriticalErrorCount, WaitingSeconds
"value" : 102 // value of the measured metric (in this case seconds)
```

## <a name="root-cause-analysis-and-improvement-recommendations"></a>Recomendaciones de mejoras y análisis de la causa principal

La última parte del registro de rendimiento de Intelligent Insights pertenece al análisis automatizado de la causa principal del problema de degradación de rendimiento identificado. La información aparece en un vocabulario comprensible en la propiedad de análisis de la causa principal (rootCauseAnalysis_s). Siempre que sea posible, se incluyen recomendaciones de mejora en el registro.

```json
// example of reported root cause analysis of the detected performance issue, in a human-readable format

"rootCauseAnalysis_s" : "High data IO caused performance to degrade. It seems that this database is missing some indexes that could help."
```

El registro de rendimiento de Intelligent Insights se puede usar con [registros de Azure Monitor](/azure/log-analytics/log-analytics-azure-sql) o una solución de terceros para las funcionalidades personalizadas de informes y alertas de DevOps.

## <a name="next-steps"></a>Pasos siguientes

- Conozca los conceptos de [Intelligent Insights](intelligent-insights-overview.md).
- Aprenda a [solucionar problemas de rendimiento con Intelligent Insights](intelligent-insights-troubleshoot-performance.md).
- Aprenda a [supervisar problemas de rendimiento mediante Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Aprenda a [recopilar y usar los datos de registro provenientes de los recursos de Azure](../../azure-monitor/essentials/platform-logs-overview.md).