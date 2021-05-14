---
title: Alertas de registro en Azure Monitor
description: Desencadene correos electrónicos, notificaciones, llamadas a direcciones URL de sitios web (webhooks) o automatización cuando se cumpla la condición de la consulta de registro que haya especificado.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 367dd261e9147c2dc14f1085af553222b621d91e
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108279791"
---
# <a name="log-alerts-in-azure-monitor"></a>Alertas de registro en Azure Monitor

## <a name="overview"></a>Introducción

Las alertas de registro son uno de los tipos de alerta que se admiten en [Alertas de Azure](./alerts-overview.md). Las alertas de registro permiten a los usuarios usar una consulta de [Log Analytics](../logs/log-analytics-tutorial.md) para evaluar los registros de los recursos según una frecuencia establecida y activar una alerta en función de los resultados. Las reglas pueden desencadenar una o varias acciones mediante [grupos de acciones](./action-groups.md).

> [!NOTE]
> Los datos de registro de un [área de trabajo de Log Analytics](../logs/log-analytics-tutorial.md) se pueden enviar al almacén de métricas de Azure Monitor. Las alertas de métricas tienen [diferentes comportamientos](alerts-metric-overview.md), lo que puede ser más conveniente en función de los datos con los que esté trabajando. Para saber más sobre cómo se pueden enrutar los registros a las métricas, vea [Alerta de métricas para los registros](alerts-metric-logs.md).

> [!NOTE]
> Actualmente no hay cargos adicionales por la versión `2020-05-01-preview` de la API y las alertas de registro orientadas a los recursos.  Los precios de las características en versión preliminar se anunciarán en el futuro y se avisará antes del inicio de la facturación. Si decide seguir usando la nueva versión de la API y las alertas de registro orientadas a los recursos después del período de aviso, se le facturará según la tarifa aplicable.

## <a name="prerequisites"></a>Requisitos previos

Las alertas de registro ejecutan consultas sobre los datos de Log Analytics. En primer lugar, debe comenzar a [recopilar datos de registro](../essentials/resource-logs.md) y consultar estos datos de registro para detectar problemas. Puede usar el [tema de ejemplos de consultas de alertas](../logs/example-queries.md) en Log Analytics para comprender qué puede detectar, o bien puede [empezar a escribir su propia consulta](../logs/log-analytics-tutorial.md).

El [colaborador de supervisión de Azure](../roles-permissions-security.md) es un rol común necesario para crear, modificar y actualizar las alertas de registro. También se necesitan derechos de acceso y de ejecución de consulta para los registros de recursos. El acceso parcial a los registros de recursos puede producir errores en las consultas o devolver resultados parciales. [Más información sobre la configuración de alertas de registro en Azure](./alerts-log.md).

> [!NOTE]
> Las alertas de registro de Log Analytics se administraban antes mediante la versión [Alert API heredada de Log Analytics](./api-alerts.md). [Obtenga más información sobre cómo cambiar a la versión ScheduledQueryRules API actual](../alerts/alerts-log-api-switch.md).

## <a name="query-evaluation-definition"></a>Definición de la evaluación de la consulta

La definición de la condición de las reglas de búsqueda de registros comienza con estos puntos:

- ¿Qué consulta debo ejecutar?
- ¿Cómo uso los resultados?

En las secciones siguientes se describen los distintos parámetros que se pueden usar para establecer la lógica anterior.

### <a name="log-query"></a>Consulta de registro
La consulta de [Log Analytics](../logs/log-analytics-tutorial.md) se utiliza para evaluar la regla. Los registros devueltos por esta consulta se usan para determinar si se desencadena una alerta. El ámbito de la consulta puede ser:

- Un recurso específico, como una máquina virtual.
- Un recurso a gran escala, como una suscripción o un grupo de recursos.
- Varios recursos, mediante el uso de una [consulta entre recursos](../logs/cross-workspace-query.md#querying-across-log-analytics-workspaces-and-from-application-insights). 
 
> [!IMPORTANT]
> Las consultas de alerta tienen restricciones para garantizar un rendimiento óptimo y la pertinencia de los resultados. [Obtenga más información aquí](./alerts-log-query.md).

> [!IMPORTANT]
> Las consultas orientadas a los recursos y las [consultas entre recursos](../logs/cross-workspace-query.md#querying-across-log-analytics-workspaces-and-from-application-insights) solo se admiten con la versión scheduledQueryRules API actual. Si usa la versión [Alert API de Log Analytics heredada](./api-alerts.md), tendrá que cambiar a la nueva. [Más información sobre el cambio](./alerts-log-api-switch.md)

#### <a name="query-time-range"></a>Intervalo de tiempo de consulta

El intervalo de tiempo se establece en la definición de la condición de la regla. En las áreas de trabajo y en Application Insights, se llama **Período**. En los demás tipos de recursos, se llama **Reemplazar intervalo de tiempo de consulta**.

Al igual que en el análisis de registros, el intervalo de tiempo limita los datos de consulta al intervalo especificado. Este intervalo se aplica incluso si se utiliza el comando **ago** en la consulta.

Por ejemplo, una consulta examina 60 minutos si el intervalo de tiempo es de 60 minutos, aunque el texto contenga **ago(1d)** . El intervalo de tiempo y el filtrado de tiempo de la consulta deben coincidir. En el caso del ejemplo, cambiar el valor de **Período** / **Reemplazar intervalo de tiempo de consulta** a un día funcionaría según lo esperado.

### <a name="measure"></a>Measure

Las alertas de registro convierten el registro en valores numéricos que se pueden evaluar. Se pueden medir dos elementos diferentes:

#### <a name="count-of-the-results-table-rows"></a>Recuento de las filas de la tabla de resultados

El recuento de resultados es la medida predeterminada. Es la medida perfecta para trabajar con eventos como registros de eventos de Windows, syslog o excepciones de aplicaciones. Se desencadena cuando se producen o no se producen entradas de registro en el período de tiempo evaluado.

Las alertas de registro funcionan mejor cuando se quieren detectar datos en el registro. No funcionan tan bien cuando se quiere detectar la falta de datos en los registros. Por ejemplo, cuando se trata de alertas sobre el latido de máquinas virtuales.

En el caso de las áreas de trabajo y Application Insights, se llama **Basada en** con la selección **Número de resultados**. En los demás tipos de recursos, se denomina **Medida** con la selección **Filas de la tabla**.

> [!NOTE]
> Dado que los registros son datos semiestructurados y tienen intrínsecamente mayor latencia que la métrica, puede que las alertas no se activen al intentar detectar la falta de datos en los registros; en este caso, puede usar [alertas de métricas](alerts-metric-overview.md). Puede enviar datos al almacén de métricas desde los registros mediante [alertas de métricas para registros](alerts-metric-logs.md).

##### <a name="example-of-results-table-rows-count-use-case"></a>Ejemplo de caso de uso de recuento de filas de la tabla de resultados

Desea saber cuándo respondió la aplicación con el código de error 500 (error interno del servidor). Crearía una regla de alertas con los detalles siguientes:

- **Consulta:** 

```Kusto
requests
| where resultCode == "500"
```

- **Período de tiempo / Granularidad de agregación**: 15 minutos
- **Frecuencia de la alerta**: 15 minutos
- **Valor del umbral:** mayor que 0

Con esta configuración, la regla de alertas supervisa si hay solicitudes que terminen con el código de error 500. La consulta se ejecuta cada 15 minutos y busca en los últimos 15 minutos. Si se encuentra aunque sea un registro, se activa la alerta y se desencadenan las acciones configuradas.

#### <a name="calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value"></a>Cálculo de medida basado en una columna numérica (como el valor del contador de CPU)

En el caso de las áreas de trabajo y Application Insights, se llama **Basada en** con la selección **Unidades métricas**. En los demás tipos de recursos, se denomina **Medida** con la selección de cualquier nombre de columna de número.

### <a name="aggregation-type"></a>Tipo de agregación

El cálculo se realiza en varios registros para agregarlos a un único valor numérico. Por ejemplo:
- **Recuento** devuelve el número de registros de la consulta.
- **Media** devuelve el promedio de la columna de la medida [**Granularidad de agregación**](#aggregation-granularity) definida.

En las áreas de trabajo y Application Insights, solo se admite en el tipo de medida **Unidades métricas**. El resultado de la consulta debe contener una columna denominada AggregatedValue que proporcione un valor numérico después de una agregación definida por el usuario. En los demás tipos de recursos, el valor de **Tipo de agregación** se selecciona a partir del campo con ese nombre.

### <a name="aggregation-granularity"></a>Granularidad de agregación

Determina el intervalo que se usa para agregar varios registros en un solo valor numérico. Por ejemplo, si ha especificado **5 minutos**, los registros se agruparán por intervalos de 5 minutos según el valor especificado en **Tipo de agregación**.

En las áreas de trabajo y Application Insights, solo se admite en el tipo de medida **Unidades métricas**. El resultado de la consulta debe contener [bin()](/azure/kusto/query/binfunction), que establece el intervalo de los resultados de la consulta. En los demás tipos de recursos, el campo que controla esta configuración se denomina **Granularidad de agregación**.

> [!NOTE]
> Como [bin()](/azure/kusto/query/binfunction) puede generar intervalos de tiempo distintos, el servicio de alerta convertirá automáticamente la función [bin()](/azure/kusto/query/binfunction) en la función [bin_at()](/azure/kusto/query/binatfunction) con el valor adecuado en tiempo de ejecución para garantizar resultados con un punto fijo.

### <a name="split-by-alert-dimensions"></a>División por dimensiones de alerta

Puede dividir las alertas por columnas de número o de cadena en alertas independientes mediante la agrupación en combinaciones únicas. Al crear alertas orientadas a recursos a gran escala (con un ámbito de suscripción o grupo de recursos), puede dividirlas por columna de identificador de recurso de Azure. La división según la columna de identificador de recurso de Azure cambiará el destino de la alerta al recurso especificado.

Se recomienda dividir por la columna de identificador de recursos de Azure cuando quiera supervisar la misma condición en varios recursos de Azure. Por ejemplo, supervisar un uso de la CPU superior al 80 % en todas las máquinas virtuales. También puede decidir no dividir cuando quiera supervisar una condición en varios recursos del ámbito, por ejemplo, supervisar que al menos cinco máquinas del ámbito del grupo de recursos tengan un uso de CPU superior al 80 %.

En las áreas de trabajo y Application Insights, solo se admite en el tipo de medida **Unidades métricas**. El campo se denomina **Agregado en**. Se limita a tres columnas. Si hay más de tres columnas de agrupación en la consulta, pueden producirse resultados inesperados. En los demás tipos de recursos, se configura en la sección **Split by dimensions** (Dividir por dimensiones) de la condición (se limita a seis divisiones).

#### <a name="example-of-splitting-by-alert-dimensions"></a>Ejemplo de división por dimensiones de alerta

Por ejemplo, supongamos que desea supervisar los errores de varias máquinas virtuales que ejecutan la aplicación o el sitio web en un grupo de recursos específico. Puede hacerlo mediante una regla de alertas de registro, tal como se indica a continuación:

- **Consulta:** 

    ```Kusto
    // Reported errors
    union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
    | where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
    ```

    Cuando se usan áreas de trabajo y Application Insights con la lógica de alerta **Unidades métricas**, es necesario agregar esta línea al texto de la consulta:

    ```Kusto
    | summarize AggregatedValue = count() by Computer, bin(TimeGenerated, 15m)
    ```

- **Resource ID Column (Columna de id. de recurso)** : _ResourceId (actualmente, la división por la columna de identificador de recurso en las reglas de alertas solo está disponible en las suscripciones y los grupos de recursos)
- **Dimensiones/Agregado en**:
  - Equipo = VM1, VM2 (actualmente, no está disponible el filtrado de valores en la definición de reglas de alertas para áreas de trabajo y Application Insights; debe filtrar en el texto de la consulta)
- **Período de tiempo / Granularidad de agregación**: 15 minutos
- **Frecuencia de la alerta**: 15 minutos
- **Valor del umbral:** mayor que 0

Esta regla supervisa si alguna máquina virtual presentó eventos de error en los últimos 15 minutos. Cada máquina virtual se supervisa por separado y desencadena acciones de forma individual.

> [!NOTE]
> La división por dimensiones de alerta solo está disponible en la versión scheduledQueryRules API actual. Si usa la versión [Alert API de Log Analytics heredada](./api-alerts.md), tendrá que cambiar a la nueva. [Más información sobre cómo cambiar](./alerts-log-api-switch.md). Las alertas orientadas a recursos a gran escala solo se admiten en la versión `2020-05-01-preview` y versiones posteriores de la API.

## <a name="alert-logic-definition"></a>Definición de la lógica de alerta

Después de definir la consulta para ejecutar y evaluar los resultados, deberá definir la lógica de alerta y cuándo activar las acciones. En las secciones siguientes se describen los distintos parámetros que se pueden usar:

### <a name="threshold-and-operator"></a>Umbral y operador

Los resultados de la consulta se transforman en un número que se compara con el umbral y operador.

### <a name="frequency"></a>Frecuencia

> [!NOTE]
> Actualmente no se cobran cargos adicionales por las alertas de registro con una frecuencia de 1 minuto. Los precios de las características en versión preliminar se anunciarán en el futuro y se avisará antes del inicio de la facturación. Si decide seguir usando las alertas de registro con una frecuencia de 1 minuto después del período de aviso, se le facturará según la tarifa aplicable.

El intervalo en el que se ejecuta la consulta. Se puede establecer entre 1 minuto y un día. Debe ser igual o menor que el [intervalo de tiempo de consulta ](#query-time-range) para no omitir entradas del registro.

Por ejemplo, supongamos que establece el período de tiempo en 30 minutos y la frecuencia en 1 hora.  Si la consulta se ejecuta a las 00:00, devuelve registros entre las 23:30 y las 00:00. La próxima vez que se ejecute la consulta será a la 01:00 y devolverá los registros comprendidos entre las 00:30 y la 01:00. Por tanto, nunca se evaluarán los registros creados entre las 00:00 y las 00:30.

Para usar alertas con una frecuencia de 1 minuto, tiene que establecer una propiedad a través de la API. Al crear o actualizar reglas de alertas de registro existentes en la versión de API: `2020-05-01-preview` en la sección `properties`, agregue `evaluationFrequency` con el valor `PT1M` de tipo `String`. Al crear o actualizar reglas de alertas de registro existentes en la versión de API: `2018-04-16` en la sección `schedule`, agregue `frequencyInMinutes` con el valor `1` de tipo `Int`. 

### <a name="number-of-violations-to-trigger-alert"></a>Número de infracciones que desencadenarán la alerta

Puede especificar el período de evaluación de la alerta y el número de errores necesarios para desencadenarla. Esto permite definir mejor el tiempo de impacto para desencadenar una alerta. 

Por ejemplo, si el valor de [**Granularidad de agregación**](#aggregation-granularity) de la regla se define como "5 minutos", solo se puede desencadenar una alerta si se producen tres errores (15 minutos) en la última hora. Esta configuración se define mediante la directiva empresarial de la aplicación.

## <a name="state-and-resolving-alerts"></a>Estado y resolución de alertas

Las alertas de registro pueden ser sin estado o con estado (actualmente en versión preliminar cuando se usa la API). 

Las alertas sin estado se activan cada vez que se cumple la condición, incluso si se han activado anteriormente. Puede [marcar la alerta como cerrada](../alerts/alerts-managing-alert-states.md) una vez que se resuelva la instancia de alerta. También puede silenciar acciones para evitar que se desencadenen durante un período después de que se active una regla de alertas. En áreas de trabajo de Log Analytics y Application Insights, esta opción se llama **Suprimir alertas**. En los demás tipos de recursos, se denomina **Silenciar acciones**. 

Consulte este ejemplo de evaluación de alertas:

| Time    | Evaluación de la condición de registro | Resultado 
| ------- | ----------| ----------| ------- 
| 00:05 | false | La alerta no se activa. No se llamó a ninguna acción.
| 00:10 | true  | La alerta se desencadena y se llama a los grupos de acciones. Estado de nueva alerta ACTIVA.
| 00:15 | true  | La alerta se desencadena y se llama a los grupos de acciones. Estado de nueva alerta ACTIVA.
| 00:20 | false | La alerta no se activa. No se llamó a ninguna acción. El estado de las alertas anteriores sigue como ACTIVA.

Las alertas con estado se activa una vez por incidente y se resuelven. Al crear o actualizar reglas de alertas de registro existentes, agregue la marca `autoMitigate` con el valor `true` de tipo `Boolean`, en la sección `properties`. Puede usar esta característica en estas versiones de API: `2018-04-16` y `2020-05-01-preview`.

## <a name="pricing-and-billing-of-log-alerts"></a>Precios y facturación de las alertas de registro

La información de precios se encuentra en la [página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). Las alertas de registro aparecen bajo el proveedor de recursos `microsoft.insights/scheduledqueryrules` con:

- Las alertas de registro en Application Insights se muestran con el nombre exacto del recurso junto con las propiedades del grupo de recursos y la alerta.
- Las alertas de registro de Log Analytics se muestran con el nombre exacto del recurso junto con las propiedades del grupo de recursos y la alerta, cuando se crean mediante [scheduledQueryRules API](/rest/api/monitor/scheduledqueryrules).
- Las alertas de registro creadas con la [API heredada de Log Analytics](./api-alerts.md) no son [recursos de Azure](../../azure-resource-manager/management/overview.md) con seguimiento y no tienen nombres de recurso únicos. Estas alertas se siguen creando en `microsoft.insights/scheduledqueryrules` como recursos ocultos, que tienen la estructura de nomenclatura de recursos `<WorkspaceName>|<savedSearchId>|<scheduleId>|<ActionId>`. Las alertas de registro de la API heredada se muestran con el nombre de recurso oculto anterior junto con las propiedades del grupo de recursos y la alerta.

> [!NOTE]
> Los caracteres de recursos no admitidos como `<, >, %, &, \, ?, /` se reemplazan por `_` en los nombres de recursos ocultos, lo que también se reflejará en la información de facturación.

> [!NOTE]
> Las alertas de registro de Log Analytics se solían administrar mediante [Alert API de Log Analytics](./api-alerts.md) y las plantillas heredadas de [búsquedas y alertas guardadas de Log Analytics](../insights/solutions.md). [Obtenga más información sobre cómo cambiar a la versión ScheduledQueryRules API actual](../alerts/alerts-log-api-switch.md). Deberá realizar cualquier administración de reglas de alertas mediante la [API heredada de Log Analytics](./api-alerts.md) hasta que decida cambiar y no pueda usar los recursos ocultos.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre la [creación de alertas de registro en Azure](./alerts-log.md).
* Información sobre [webhooks en alertas de registro en Azure](../alerts/alerts-log-webhook.md).
* Más información acerca de las [Alertas de Azure](./alerts-overview.md).
* Más información sobre [Log Analytics](../logs/log-query-overview.md).
