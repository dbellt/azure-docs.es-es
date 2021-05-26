---
title: Optimización de consultas de alerta de registro | Microsoft Docs
description: Recomendaciones para escribir consultas de alerta eficientes
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: e15b3f81cf5f29efa739939fe29df25f338187cc
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475264"
---
# <a name="optimizing-log-alert-queries"></a>Optimización de consultas de alerta de registro
En este artículo se describe cómo escribir y convertir las consultas de [alerta de registro](./alerts-unified-log.md) para lograr un rendimiento óptimo. Las consultas optimizadas reducen la latencia y la carga de alertas, que se ejecutan con frecuencia.

## <a name="how-to-start-writing-an-alert-log-query"></a>Cómo empezar a escribir una consulta de alerta de registro

Las consultas de alerta comienzan a partir de la [consulta de los datos de registro en Log Analytics](alerts-log.md#create-a-log-alert-rule-with-the-azure-portal) que indican el problema. Puede usar el [tema de ejemplos de consultas de alertas](../logs/queries.md) para comprender qué puede detectar. También puede [empezar a escribir su propia consulta](../logs/log-analytics-tutorial.md). 

### <a name="queries-that-indicate-the-issue-and-not-the-alert"></a>Consultas que indican el problema y no la alerta

El flujo de alerta se ha creado de forma que transforme los resultados que indican la emisión de una alerta. Por ejemplo, en un caso de una consulta como la siguiente:

``` Kusto
SecurityEvent
| where EventID == 4624
```

Si la intención del usuario es enviar una alerta, cuando se produce este tipo de evento la lógica de alerta anexa `count` a la consulta. La consulta que se ejecutará será la siguiente:

``` Kusto
SecurityEvent
| where EventID == 4624
| count
```

No es necesario agregar lógica de alerta a la consulta y hacerlo puede incluso provocar problemas. En el ejemplo anterior, si incluye `count` en la consulta, siempre dará como resultado el valor 1, ya que el servicio de alertas ejecutará el comando `count` sobre `count`.

### <a name="avoid-limit-and-take-operators"></a>Evitar los operadores `limit` y `take`

El uso de `limit` y `take` en las consultas puede aumentar la latencia y la carga de alertas, ya que los resultados no son coherentes a lo largo del tiempo. Es preferible usarlos solo si es necesario.

## <a name="log-query-constraints"></a>Restricciones de la consulta de registro
Las [consultas de registro de Azure Monitor](../logs/log-query-overview.md) comienzan con una tabla o con un operador [`search`](/azure/kusto/query/searchoperator) o [`union`](/azure/kusto/query/unionoperator).

Las consultas para las reglas de alertas de registro siempre deben empezar con una tabla que defina un ámbito claro, que mejore el rendimiento de las consultas y la pertinencia de los resultados. Las consultas de las reglas de alertas se ejecutan con frecuencia, por lo que el uso de `search` y `union` puede producir una sobrecarga excesiva que suma latencia a la alerta, ya que requiere el examen de varias tablas. Estos operadores también reducen la capacidad del servicio de alertas para optimizar la consulta.

No se admite la creación o modificación de reglas de alertas de registro que utilicen operadores `search` o `union`, excepto en consultas entre recursos.

Por ejemplo, la siguiente consulta de alertas tiene como ámbito la tabla _SecurityEvent_ y busca un identificador de evento específico. Es la única tabla que debe procesar la consulta.

``` Kusto
SecurityEvent
| where EventID == 4624
```

A las reglas de alertas del registro que usan [consultas entre recursos](../logs/cross-workspace-query.md) no les afecta este cambio, ya que usan un tipo de operador `union` que limita el ámbito de la consulta a recursos específicos. El ejemplo siguiente sería una consulta de alerta de registro válida:

```Kusto
union
app('Contoso-app1').requests,
app('Contoso-app2').requests,
workspace('Contoso-workspace1').Perf 
```

>[!NOTE]
> Las [consultas entre recursos](../logs/cross-workspace-query.md) se admiten en la nueva versión [scheduledQueryRules API](/rest/api/monitor/scheduledqueryrules). Si sigue usando la versión [Alert API de Log Analytics heredada](./api-alerts.md) para crear alertas de registro, puede obtener información sobre el cambio [aquí](../alerts/alerts-log-api-switch.md).

## <a name="examples"></a>Ejemplos
Los siguientes ejemplos incluyen consultas de registro que usan `search` y `union`, y proporcionan los pasos que puede seguir para modificar estas consultas para su uso en las reglas de alertas.

### <a name="example-1"></a>Ejemplo 1
Desea crear una regla de alertas de registro mediante la siguiente consulta que recupera información de rendimiento mediante `search`: 

``` Kusto
search *
| where Type == 'Perf' and CounterName == '% Free Space'
| where CounterValue < 30
```

Para modificar esta consulta, empiece por usar la consulta siguiente para identificar la tabla a la que pertenecen las propiedades:

``` Kusto
search *
| where CounterName == '% Free Space'
| summarize by $table
```

El resultado de esta consulta muestra que la propiedad _CounterName_ procede de la tabla _Perf_.

Puede utilizar este resultado para crear la siguiente consulta, que podría usar para la regla de alertas:

``` Kusto
Perf
| where CounterName == '% Free Space'
| where CounterValue < 30
```

### <a name="example-2"></a>Ejemplo 2
Desea crear una regla de alertas de registro mediante la siguiente consulta que recupera información de rendimiento mediante `search`: 

``` Kusto
search ObjectName =="Memory" and CounterName=="% Committed Bytes In Use"
| summarize Avg_Memory_Usage =avg(CounterValue) by Computer
| where Avg_Memory_Usage between(90 .. 95)  
```

Para modificar esta consulta, empiece por usar la consulta siguiente para identificar la tabla a la que pertenecen las propiedades:

``` Kusto
search ObjectName=="Memory" and CounterName=="% Committed Bytes In Use"
| summarize by $table
```

El resultado de esta consulta muestra que la propiedad _ObjectName_ y _CounterName_ procede de la tabla _Perf_.

Puede utilizar este resultado para crear la siguiente consulta, que podría usar para la regla de alertas:

``` Kusto
Perf
| where ObjectName =="Memory" and CounterName=="% Committed Bytes In Use"
| summarize Avg_Memory_Usage=avg(CounterValue) by Computer
| where Avg_Memory_Usage between(90 .. 95)
```

### <a name="example-3"></a>Ejemplo 3

Desea crear una regla de alertas de registro mediante la siguiente consulta que usa `search` y `union` para recuperar información de rendimiento: 

``` Kusto
search (ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total")
| where Computer !in (
    union *
    | where CounterName == "% Processor Utility"
    | summarize by Computer)
| summarize Avg_Idle_Time = avg(CounterValue) by Computer
```

Para modificar esta consulta, empiece por usar la consulta siguiente para identificar la tabla a la que pertenecen las propiedades de la primera parte de la consulta: 

``` Kusto
search (ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total")
| summarize by $table
```

El resultado de esta consulta muestra que todas estas propiedades proceden de la tabla _Perf_.

Ahora use `union` con el comando `withsource` para identificar qué tabla de origen ha contribuido a cada fila.

``` Kusto
union withsource=table *
| where CounterName == "% Processor Utility"
| summarize by table
```

El resultado de esta consulta muestra que estas propiedades también proceden de la tabla _Perf_.

Puede utilizar estos resultados para crear la siguiente consulta, que podría usar para la regla de alertas: 

``` Kusto
Perf
| where ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total"
| where Computer !in (
    (Perf
    | where CounterName == "% Processor Utility"
    | summarize by Computer))
| summarize Avg_Idle_Time = avg(CounterValue) by Computer
``` 

### <a name="example-4"></a>Ejemplo 4
Desea crear una regla de alertas de registro mediante la siguiente consulta que combina los resultados de dos consultas `search`:

```Kusto
search Type == 'SecurityEvent' and EventID == '4625'
| summarize by Computer, Hour = bin(TimeGenerated, 1h)
| join kind = leftouter (
    search in (Heartbeat) OSType == 'Windows'
    | summarize arg_max(TimeGenerated, Computer) by Computer , Hour = bin(TimeGenerated, 1h)
    | project Hour , Computer
) on Hour
```

Para modificar la consulta, empiece por usar la consulta siguiente para identificar la tabla que contiene las propiedades del lado izquierdo de la combinación: 

``` Kusto
search Type == 'SecurityEvent' and EventID == '4625'
| summarize by $table
```

El resultado indica que las propiedades del lado izquierdo de la combinación pertenecen a la tabla _SecurityEvent_. 

Ahora, use la consulta siguiente para identificar la tabla que contiene las propiedades del lado derecho de la combinación: 
 
``` Kusto
search in (Heartbeat) OSType == 'Windows'
| summarize by $table
```
 
El resultado indica que las propiedades del lado derecho de la combinación pertenecen a la tabla _Heartbeat_.

Puede utilizar estos resultados para crear la siguiente consulta, que podría usar para la regla de alertas: 

``` Kusto
SecurityEvent
| where EventID == '4625'
| summarize by Computer, Hour = bin(TimeGenerated, 1h)
| join kind = leftouter (
    Heartbeat
    | where OSType == 'Windows'
    | summarize arg_max(TimeGenerated, Computer) by Computer , Hour = bin(TimeGenerated, 1h)
    | project Hour , Computer
) on Hour
```

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre las [alertas de registro](alerts-log.md) de Azure Monitor.
- Más información acerca de las [consultas de registro](../logs/log-query-overview.md).