---
title: Solución de problemas de alertas de registro en Azure Monitor | Microsoft Docs
description: Problemas comunes, errores y resoluciones para las reglas de alertas de registro en Azure.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 05f34d260e318e59381169dd63b654ec695a87cc
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108318250"
---
# <a name="troubleshoot-log-alerts-in-azure-monitor"></a>Solución de problemas de alertas de registro en Azure Monitor  

En este artículo se muestra cómo solucionar problemas habituales con las alertas de registro en Azure Monitor. Además, se proporcionan soluciones a los problemas comunes sobre la funcionalidad y la configuración de las alertas de registro.

Las alertas de registro permiten a los usuarios usar una consulta de [Log Analytics](../logs/log-analytics-tutorial.md) para evaluar los registros de los recursos según una frecuencia establecida y activar una alerta en función de los resultados. Las reglas pueden desencadenar una o varias acciones mediante [grupos de acciones](./action-groups.md). [Obtenga más información sobre la funcionalidad y la terminología de las alertas de registro](alerts-unified-log.md).

> [!NOTE]
> En este artículo no se tienen en cuenta los casos en que Azure Portal muestra una regla de alertas desencadenada y no se realiza una notificación a través de un grupo de acciones asociado. Para estos casos, consulte los detalles sobre la solución de problemas [aquí](./alerts-troubleshoot.md#action-or-notification-on-my-alert-did-not-work-as-expected).

## <a name="log-alert-didnt-fire"></a>No se activó la alerta de registro

### <a name="data-ingestion-time-for-logs"></a>Tiempo de ingesta de datos para registros

Azure Monitor procesa terabytes de registros de clientes de todo el mundo, lo que puede provocar [latencia en la ingesta de registros](../logs/data-ingestion-time.md).

Los registros son datos semiestructurados e intrínsecamente con mayor latencia que las métricas. Si advierte que hay más de cuatro minutos de retraso en las alertas activadas, debería usar [alertas de métricas](alerts-metric-overview.md). Puede enviar datos al almacén de métricas desde los registros mediante [alertas de métricas para registros](alerts-metric-logs.md).

El sistema vuelve a intentar la evaluación de la alerta varias veces para mitigar la latencia. Una vez que llegan los datos, se activa la alerta, que en la mayoría de los casos no coincide con la hora de entrada del registro.

### <a name="incorrect-query-time-range-configured"></a>Configuración incorrecta del intervalo de tiempo de consulta

El intervalo de tiempo de consulta se establece en la definición de la condición de la regla. Este campo se denomina **Período** en las áreas de trabajo y Application Insights, y **Reemplazar intervalo de tiempo de consulta** en los demás tipos de recursos. Al igual que en el análisis de registros, el intervalo de tiempo limita los datos de consulta al período especificado. Este intervalo se aplica incluso si se utiliza el comando **ago** en la consulta. 

Por ejemplo, una consulta examina 60 minutos si el intervalo de tiempo es de 60 minutos, aunque el texto contenga **ago(1d)** . El intervalo de tiempo y el filtrado de tiempo de la consulta deben coincidir. En el caso del ejemplo, cambiar el valor de **Período** / **Reemplazar intervalo de tiempo de consulta** a un día funcionaría según lo esperado.

![Período de tiempo](media/alerts-troubleshoot-log/LogAlertTimePeriod.png)

### <a name="actions-are-muted-in-the-alert-rule"></a>Acciones silenciadas en la regla de alertas

Las alertas de registro ofrecen una opción para silenciar las acciones de alerta activadas durante un período de tiempo determinado. Este campo se denomina **Desactivar alertas** en áreas de trabajo y Application Insights. En los demás tipos de recursos, se denomina **Silenciar acciones**. 

Un problema habitual es que crea que la alerta no ha desencadenado las acciones debido a un problema del servicio cuando, en realidad, se han silenciado en la configuración de la regla.

![Suprimir alertas](media/alerts-troubleshoot-log/LogAlertSuppress.png)

### <a name="metric-measurement-alert-rule-with-splitting-using-the-legacy-log-analytics-api"></a>Regla de alertas de unidades métricas con división mediante la API de Log Analytics heredada

[Unidades métricas](alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value) es un tipo de alerta de registro que se basa en resultados de series temporales resumidos. Estas reglas permiten agrupar por columnas para [dividir las alertas](alerts-unified-log.md#split-by-alert-dimensions). Si usa la API de Log Analytics heredada, la división no funcionará según lo previsto. En esta API heredada no se puede elegir la agrupación.

La versión actual, ScheduledQueryRules API, permite establecer la opción **Agregado en** en las reglas de tipo [Unidades métricas](alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value), que funcionará según lo previsto. [Obtenga más información sobre cómo cambiar a la versión ScheduledQueryRules API actual](../alerts/alerts-log-api-switch.md).

## <a name="log-alert-fired-unnecessarily"></a>Alerta de registro activada innecesariamente

Es posible que una [regla de alertas de registro configurada en Azure Monitor](./alerts-log.md) se desencadene de forma inesperada. En las secciones siguientes se describen algunos motivos comunes.

### <a name="alert-triggered-by-partial-data"></a>Alerta desencadenada por datos parciales

Azure Monitor procesa terabytes de registros de clientes de todo el mundo, lo que puede provocar [latencia en la ingesta de registros](../logs/data-ingestion-time.md).

Los registros son datos semiestructurados e intrínsecamente con mayor latencia que las métricas. Si advierte que hay errores de activación en las alertas, debería usar [alertas de métricas](alerts-metric-overview.md). Puede enviar datos al almacén de métricas desde los registros mediante [alertas de métricas para registros](alerts-metric-logs.md).

Las alertas de registro funcionan mejor cuando se quieren detectar datos en los registros. No funcionan tan bien cuando se quiere detectar la falta de datos en los registros. Por ejemplo, cuando se trata de alertas sobre el latido de máquinas virtuales. 

Aunque existen funcionalidades integradas para evitar alertas falsas, pueden seguir produciéndose en datos con mucha latencia (en torno a más de 30 minutos) y en datos con picos de latencia.

### <a name="query-optimization-issues"></a>Problemas de optimización de consultas

El servicio de alertas cambia la consulta para optimizarla y reducir la carga y la latencia de las alertas. El flujo de alerta se ha creado de forma que transforme los resultados que indican la emisión de una alerta. Por ejemplo, en un caso de una consulta como la siguiente:

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

La consulta optimizada es lo que ejecuta el servicio de alertas de registro. Puede ejecutar la consulta modificada en el [portal](../logs/log-query-overview.md) o en la [API](/rest/api/loganalytics/) de Log Analytics.

En el caso de las áreas de trabajo y Application Insights, se llama **Consulta que se va a ejecutar** en el panel de condición. En los demás tipos de recursos, seleccione **See final alert Query** (Ver consulta de alerta final) en la pestaña de condición.

![Consulta que se va a ejecutar](media/alerts-troubleshoot-log/LogAlertPreview.png)

## <a name="log-alert-was-disabled"></a>La alerta de registro se deshabilitó

En las siguientes secciones se enumeran algunos de los motivos por los que Azure Monitor podría deshabilitar la regla de alertas de registro. También se incluye un [ejemplo del registro de actividad que se envía cuando se deshabilita una regla](#activity-log-example-when-rule-is-disabled).

### <a name="alert-scope-no-longer-exists-or-was-moved"></a>El ámbito de alerta ya no existe o se ha movido

Cuando los recursos de ámbito de una regla de alertas ya no son válidos, se produce un error en la ejecución de la regla. En este caso, la facturación también se detiene.

Azure Monitor deshabilitará la alerta de registro después de una semana si se produce un error de forma continuada.

### <a name="query-used-in-a-log-alert-isnt-valid"></a>La consulta que se usa en una alerta de registro no es válida

Cuando se crea una regla de alertas de registro, se valida la consulta para comprobar si la sintaxis es correcta. En ocasiones, sin embargo, la consulta proporcionada en la regla de alertas de registro puede empezar a fallar. Estos son algunos de los motivos habituales:

- Las reglas se crearon a través de la API y el usuario omitió la validación.
- La consulta [se ejecuta en varios recursos](../logs/cross-workspace-query.md) y uno o varios de los recursos se eliminaron o se movieron.
- Se [produce un error en la consulta](https://dev.loganalytics.io/documentation/Using-the-API/Errors) por los siguientes motivos:
    - La solución de registro no se [implementó en el área de trabajo](../insights/solutions.md#install-a-monitoring-solution), por lo que no se crearon tablas.
    - Los datos dejaron de llegar a una tabla de la consulta durante más de 30 días.
    - Todavía no se han creado [tablas de registros personalizadas](../agents/data-sources-custom-logs.md), porque no se ha iniciado el flujo de datos.
- Los cambios en el [lenguaje de consulta](/azure/kusto/query/) incluyen un formato revisado para comandos y funciones. Por lo tanto, la consulta proporcionada anteriormente ya no es válida.

[Azure Advisor](../../advisor/advisor-overview.md) le advierte acerca de este comportamiento. Agrega una recomendación sobre la regla de alertas de registro afectada. La categoría usada es "Alta disponibilidad" con impacto medio y la descripción "Reparar la regla de alerta de registro para garantizar la supervisión".

## <a name="alert-rule-quota-was-reached"></a>Se alcanzó la cuota de la regla de alerta

El número de reglas de alertas de búsqueda de registros por suscripción y recurso está sujeto a los límites de cuota descritos [en este artículo](../service-limits.md).

### <a name="recommended-steps"></a>Pasos recomendados
    
Si ha alcanzado el límite de cuota, los siguientes pasos pueden ayudar a resolver el problema.

1. Intente eliminar o deshabilitar las reglas de alertas de búsqueda de registros que ya no se usan.
1. Intente usar la [división de alertas por dimensiones](alerts-unified-log.md#split-by-alert-dimensions) para reducir el número de reglas. Estas reglas pueden supervisar muchos recursos y casos de detección.
1. Si necesita aumentar el límite de cuota, abra una solicitud de soporte técnico y proporcione la siguiente información:

    - Los identificadores de suscripción y de recurso para los que tiene que aumentar los límites de cuota.
    - El motivo de aumento de la cuota.
    - Tipo de recurso para el que se va a aumentar la cuota: **Log Analytics**, **Application Insights**, etcétera.
    - El límite de cuota solicitado.


### <a name="to-check-the-current-usage-of-new-log-alert-rules"></a>Para comprobar el uso actual de las nuevas reglas de alertas de registro
    
#### <a name="from-the-azure-portal"></a>Desde Azure Portal

1. Abra la pantalla *Alertas* y seleccione *Administrar reglas de alerta*.
2. Filtre la suscripción correspondiente mediante el control desplegable *Suscripción*.
3. Asegúrese de NO filtrar por un grupo de recursos, tipo de recurso o recurso específico.
4. En el control desplegable *Tipo de señal*, seleccione "Búsqueda de registros".
5. Compruebe que el control desplegable *Estado* está establecido en "Habilitado".
6. El número total de reglas de alertas de búsqueda de registros se mostrará encima de la lista de reglas.

#### <a name="from-api"></a>Desde la API

- PowerShell: [Get-AzScheduledQueryRule](/powershell/module/az.monitor/get-azscheduledqueryrule)
- API REST: [Lista por suscripción](/rest/api/monitor/scheduledqueryrules/listbysubscription)

## <a name="activity-log-example-when-rule-is-disabled"></a>Ejemplo de registro de actividad cuando la regla está deshabilitada

Si se produce un error en la consulta durante siete días de forma continuada, Azure Monitor deshabilitará la alerta de registro y dejará de facturar la regla. Puede averiguar la hora exacta en que Azure Monitor deshabilitó la alerta de registro a través del [registro de actividad de Azure](../../azure-resource-manager/management/view-activity-logs.md). Consulte este ejemplo:

```json
{
    "caller": "Microsoft.Insights/ScheduledQueryRules",
    "channels": "Operation",
    "claims": {
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/ScheduledQueryRules"
    },
    "correlationId": "abcdefg-4d12-1234-4256-21233554aff",
    "description": "Alert: test-bad-alerts is disabled by the System due to : Alert has been failing consistently with the same exception for the past week",
    "eventDataId": "f123e07-bf45-1234-4565-123a123455b",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "Administrative",
        "localizedValue": "Administrative"
    },
    "eventTimestamp": "2019-03-22T04:18:22.8569543Z",
    "id": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
    "level": "Informational",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Insights/ScheduledQueryRules/disable/action",
        "localizedValue": "Microsoft.Insights/ScheduledQueryRules/disable/action"
    },
    "resourceGroupName": "<Resource Group>",
    "resourceProviderName": {
        "value": "MICROSOFT.INSIGHTS",
        "localizedValue": "Microsoft Insights"
    },
    "resourceType": {
        "value": "MICROSOFT.INSIGHTS/scheduledqueryrules",
        "localizedValue": "MICROSOFT.INSIGHTS/scheduledqueryrules"
    },
    "resourceId": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
    "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-03-22T04:18:22.8569543Z",
    "subscriptionId": "<SubscriptionId>",
    "properties": {
        "resourceId": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
        "subscriptionId": "<SubscriptionId>",
        "resourceGroup": "<ResourceGroup>",
        "eventDataId": "12e12345-12dd-1234-8e3e-12345b7a1234",
        "eventTimeStamp": "03/22/2019 04:18:22",
        "issueStartTime": "03/22/2019 04:18:22",
        "operationName": "Microsoft.Insights/ScheduledQueryRules/disable/action",
        "status": "Succeeded",
        "reason": "Alert has been failing consistently with the same exception for the past week"
    },
    "relatedEvents": []
}
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [alertas de registro en Azure](./alerts-unified-log.md).
- Más información sobre la [configuración de alertas de registro](../logs/log-query-overview.md).
- Obtenga más información sobre las [consultas de registro](../logs/log-query-overview.md).
