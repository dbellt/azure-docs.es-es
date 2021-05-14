---
title: Análisis de la telemetría de Azure Functions en Application Insights
description: Obtenga información sobre cómo ver y consultar los datos de telemetría de Azure Functions que Azure Application Insights recopila y almacena.
ms.topic: how-to
ms.date: 10/14/2020
ms.custom: contperf-fy21q2
ms.openlocfilehash: 47d5bbf8d8a1f85c4f9441076ef29ffd94710947
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108064132"
---
# <a name="analyze-azure-functions-telemetry-in-application-insights"></a>Análisis de la telemetría de Azure Functions en Application Insights 

Azure Functions se integra con Application Insights para permitirle supervisar mejor las aplicaciones de funciones. Application Insights recopila los datos de telemetría generados por la aplicación de funciones, incluida la información que la aplicación escribe en los registros. La integración de Application Insights se habilita normalmente al crear la aplicación de funciones. Si la aplicación de funciones no tiene establecida la clave de instrumentación, primero debe [habilitar la integración de Application Insights](configure-monitoring.md#enable-application-insights-integration). 

De forma predeterminada, los datos recopilados de la aplicación de funciones se almacenan en Application Insights. En [Azure Portal](https://portal.azure.com), Application Insights proporciona un amplio conjunto de visualizaciones de los datos de telemetría. Puede profundizar en los registros de errores y en los eventos de consulta y las métricas. En este artículo se proporcionan ejemplos básicos de cómo ver y consultar los datos recopilados. Para más información sobre cómo explorar los datos de la aplicación de funciones en Azure Application Insights, consulte [¿Qué es Application Insights?](../azure-monitor/app/app-insights-overview.md). 

Para más información sobre la retención de datos y los posibles costos de almacenamiento, consulte [Recopilación, retención y almacenamiento de datos en Application Insights](../azure-monitor/app/data-retention-privacy.md).   

## <a name="viewing-telemetry-in-monitor-tab"></a>Visualización de la telemetría en la pestaña Supervisar

Con la [integración de Application Insights habilitada](configure-monitoring.md#enable-application-insights-integration), puede ver los datos de telemetría en la pestaña **Supervisar**.

1. En la página de la aplicación de función, seleccione una función que se haya ejecutado por lo menos una vez después de que se configurase Application Insights. A continuación, seleccione **Supervisar** en el panel izquierdo. Seleccione **Actualizar** periódicamente hasta que aparezca la lista de las invocaciones de funciones.

   ![Lista de invocaciones](media/functions-monitoring/monitor-tab-ai-invocations.png)

    > [!NOTE]
    > La lista puede tardar hasta cinco minutos en aparecer, mientras el cliente de telemetría agrupa los datos en lotes para su transmisión al servidor. Este retraso no se aplica a [Live Metrics Stream](../azure-monitor/app/live-stream.md). Este servicio se conecta al host de Functions cuando se carga la página, por lo que los registros se transmiten directamente a la página.

1. Para ver los registros de una invocación de función determinada, seleccione el vínculo de la columna **Fecha (UTC)** para esa invocación. La salida del registro para esa invocación aparece en una nueva página.

   ![Detalles de la invocación](media/functions-monitoring/invocation-details-ai.png)

1. Elija **Ejecutar en Application Insights** para ver el origen de la consulta que recupera los datos de registro de Azure Monitor en el registro de Azure. Si esta es la primera vez que usa Azure Log Analytics en la suscripción, se le pedirá que lo habilite.

1. Después de habilitar Log Analytics, se muestra la siguiente consulta. Puede ver que los resultados de la consulta se limitan a los últimos 30 días (`where timestamp > ago(30d)`) y en los resultados no se muestran más de 20 filas (`take 20`). Por el contrario, la lista de detalles de invocación de la función es para los últimos 30 días sin límite.

   ![Lista de invocación del análisis de Application Insights](media/functions-monitoring/ai-analytics-invocation-list.png)

Para más información, consulte la sección [Consultar datos de telemetría](#query-telemetry-data) que se encuentra más adelante en este artículo.

## <a name="view-telemetry-in-application-insights"></a>Visualización de la telemetría en Application Insights

Para abrir Application Insights desde una aplicación de funciones en [Azure Portal](https://portal.azure.com):

1. Examine la aplicación de funciones en el portal.

1. Seleccione **Application Insights** en **Configuración**  en la página de la izquierda. 

1. Si esta es la primera vez que usa Application Insights con su suscripción, se le pedirá que lo habilite. Para ello, seleccione **Activar Application Insights** y, a continuación, seleccione **Aplicar** en la página siguiente.

![Abra Application Insights desde la página Información general de la aplicación de función](media/functions-monitoring/ai-link.png)

Para obtener más información acerca de Application Insights, consulte [Documentación de Application Insights](/azure/application-insights/). En esta sección se muestran algunos ejemplos de cómo ver los datos de Application Insights. Si ya está familiarizado con Application Insights, puede ir directamente a [las secciones sobre cómo configurar y personalizar los datos de telemetría](configure-monitoring.md#configure-log-levels).

![Pestaña Información general de Application Insights](media/functions-monitoring/metrics-explorer.png)

Las siguientes áreas de Application Insights pueden ser útiles al evaluar el comportamiento, el rendimiento y los errores de las funciones:

| Investigación | Descripción |
| ---- | ----------- |
| **[Errores](../azure-monitor/app/asp-net-exceptions.md)** |  Cree alertas y gráficos basados en errores de funciones y excepciones del servidor. **Nombre de la operación** es el nombre de la función. Los errores en las dependencias no se muestran, a no ser que se implemente telemetría personalizada para las dependencias. |
| **[Rendimiento](../azure-monitor/app/performance-counters.md)** | Analice los problemas de rendimiento mediante la visualización del uso de recursos y el rendimiento por **instancias de rol de nube**. Estos datos de rendimiento pueden ser útiles para la depuración de escenarios en los que las funciones atascan los recursos subyacentes. |
| **[Métricas](../azure-monitor/essentials/metrics-charts.md)** | Cree gráficos y alertas basados en métricas. Las métricas incluyen el número de invocaciones de función, el tiempo de ejecución y las tasas de éxito. |
| **[Live Metrics](../azure-monitor/app/live-stream.md)** | Vea los datos de métricas a medida que se crean casi en tiempo real. |

## <a name="query-telemetry-data"></a>Consultar datos de telemetría

El [Análisis de Application Insights](../azure-monitor/logs/log-query-overview.md) proporciona acceso a todos los datos de telemetría en forma de tabla en una base de datos. Analytics ofrece un lenguaje de consulta para extraer, manipular y visualizar los datos. 

Elija **Registros** para explorar o consultar eventos registrados.

![Ejemplo de Analytics](media/functions-monitoring/analytics-traces.png)

Este es un ejemplo de consulta que muestra la distribución de solicitudes por trabajador durante los últimos 30 minutos.

```kusto
requests
| where timestamp > ago(30m) 
| summarize count() by cloud_RoleInstance, bin(timestamp, 1m)
| render timechart
```

Las tablas que están disponibles se muestran en la pestaña **Esquema** de la izquierda. Encontrará los datos generados por las invocaciones de función en las tablas siguientes:

| Tabla | Descripción |
| ----- | ----------- |
| **traces** | Registros que crea el runtime, el controlador de escala y seguimientos del código de función. |
| **requests** | Una solicitud por cada invocación de función. |
| **exceptions** | Cualquier excepción que produzca el tiempo de ejecución. |
| **customMetrics** | Recuento de invocaciones correctas y erróneas, tasa de éxito y duración. |
| **customEvents** | Eventos registrados por el tiempo de ejecución, por ejemplo: solicitudes HTTP que desencadenan una función. |
| **performanceCounters** | Información acerca del rendimiento de los servidores en los que se ejecutan las funciones. |

El resto de las tablas son para pruebas de disponibilidad, y telemetría de explorador y de cliente. Puede implementar telemetría personalizada para agregarles datos.

Dentro de cada tabla, algunos de los datos específicos de Functions están en un campo `customDimensions`.  Por ejemplo, la consulta siguiente recupera todos los seguimientos que tienen el nivel de registro `Error`.

```kusto
traces 
| where customDimensions.LogLevel == "Error"
```

El tiempo de ejecución proporciona los campos `customDimensions.LogLevel` y `customDimensions.Category`. Puede proporcionar campos adicionales en los registros que escriba en el código de función. Para obtener un ejemplo en C# , vea [Registro estructurado](functions-dotnet-class-library.md#structured-logging) en la guía del desarrollador de la biblioteca de clases .Net.

## <a name="query-scale-controller-logs"></a>Consulta de registros del controlador de escala

_Esta característica se encuentra en su versión preliminar._

Después de habilitar tanto el [registro del controlador de escala](configure-monitoring.md#configure-scale-controller-logs) como la [integración de Application Insights](configure-monitoring.md#enable-application-insights-integration), puede usar la búsqueda de registros de Application Insights para consultar los registros del controlador de escala emitidos. Los registros del controlador de escala se guardan en la colección `traces` de la categoría **ScaleControllerLogs**.

La siguiente consulta se puede usar para buscar en todos los registros del controlador de escala la aplicación de funciones actual dentro del período especificado:

```kusto
traces 
| extend CustomDimensions = todynamic(tostring(customDimensions))
| where CustomDimensions.Category == "ScaleControllerLogs"
```

La siguiente consulta se expande en la consulta anterior para mostrar cómo obtener únicamente aquellos registros que indican un cambio en la escala:

```kusto
traces 
| extend CustomDimensions = todynamic(tostring(customDimensions))
| where CustomDimensions.Category == "ScaleControllerLogs"
| where message == "Instance count changed"
| extend Reason = CustomDimensions.Reason
| extend PreviousInstanceCount = CustomDimensions.PreviousInstanceCount
| extend NewInstanceCount = CustomDimensions.CurrentInstanceCount
```

## <a name="consumption-plan-specific-metrics"></a>Métricas específicas del plan de consumo

Cuando se ejecuta un [Plan de consumo](consumption-plan.md), el *costo* de la ejecución de una única función se mide en *GB-segundos*. El costo de ejecución se calcula mediante la combinación del uso de memoria con el tiempo de ejecución. Para más información, consulte [Estimación de los costos según el plan de consumo](functions-consumption-costs.md).

Las siguientes consultas de telemetría son específicas de las métricas que afectan al costo de la ejecución de funciones en el plan de consumo.

[!INCLUDE [functions-consumption-metrics-queries](../../includes/functions-consumption-metrics-queries.md)]

## <a name="azure-monitor-metrics"></a>Métricas de Azure Monitor

Además de los datos de telemetría recopilados por Application Insights, también puede obtener datos sobre cómo se ejecuta la aplicación de funciones a partir de las [métricas de Azure Monitor](../azure-monitor/essentials/data-platform-metrics.md). Junto con las [métricas disponibles para aplicaciones de App Service](../app-service/web-sites-monitor.md#understand-metrics) de manera habitual, hay dos métricas específicas de Functions que le pueden interesar:

| Métrica | Descripción |
| ---- | ---- |
| **FunctionExecutionCount** | El número de ejecuciones de función indica el número de veces que se ha ejecutado la aplicación de funciones. Este valor se correlaciona con el número de veces que se ejecuta una función en la aplicación. |
| **FunctionExecutionUnits** | Las unidades de ejecución de función son una combinación del tiempo de ejecución y el uso de memoria.  Los datos de memoria no son una métrica disponible actualmente en Azure Monitor. Sin embargo, si desea optimizar el uso de memoria de la aplicación, puede utilizar los datos del contador de rendimiento recopilados por Application Insights. Esta métrica no se admite actualmente en los planes Prémium y Dedicado (App Service) que se ejecutan en Linux.|

Para más información sobre el cálculo de los costos de un plan de consumo con datos de Application Insights, consulte [Estimación de los costos según el plan de consumo](functions-consumption-costs.md). Para más información sobre el uso del explorador de supervisión para ver las métricas, consulte [Introducción al Explorador de métricas de Azure](../azure-monitor/essentials/metrics-getting-started.md).


## <a name="next-steps"></a>Pasos siguientes

Más información acerca de la supervisión de Azure Functions:

+ [Monitor Azure Functions](functions-monitoring.md)
+ [Configuración de la supervisión para Azure Functions](configure-monitoring.md)
