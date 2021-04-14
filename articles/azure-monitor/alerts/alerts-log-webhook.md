---
title: Acciones de webhook para alertas de registro en alertas de Azure
description: Describe cómo configurar una inserción de alertas de registro con la acción de webhook y las personalizaciones disponibles.
author: yanivlavi
ms.author: yalavi
services: monitoring
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 1834fb7478fbb9ed435dac4f1e4b43f83e5d2db1
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106383576"
---
# <a name="webhook-actions-for-log-alert-rules"></a>Acciones de webhook para reglas de alertas de registro

La [alerta de registro](alerts-log.md) admite la [configuración de grupos de acciones de webhook](./action-groups.md#webhook). En este artículo, describiremos qué propiedades están disponibles y cómo configurar un webhook JSON personalizado.

> [!NOTE]
> Actualmente, el webhook personalizado basado en JSON no se admite en la versión `2020-05-01-preview` de la API.

> [!NOTE]
> Se recomienda usar un [esquema de alertas común](../alerts/alerts-common-schema.md) para las integraciones de webhook. Este esquema ofrece la ventaja de tener una sola carga de alertas, extensible y unificada, para todos los servicios de alerta de Azure Monitor. En el caso de las reglas de alertas de registro que tienen definida una carga de JSON personalizada, habilitar el esquema de alertas común revierte el esquema de la carga al que se describe [aquí](../alerts/alerts-common-schema-definitions.md#log-alerts). Esto significa que, si quiere que se defina una carga de JSON personalizada, el webhook no puede usar el esquema de alertas común. Las alertas con el esquema común habilitado tienen un límite de tamaño superior de 256 KB por alerta y la alerta mayor no incluirá los resultados de la búsqueda. Si no se incluyen los resultados de la búsqueda, debe usar `LinkToFilteredSearchResultsAPI` o `LinkToSearchResultsAPI` para acceder a ellos con la API de Log Analytics.

## <a name="webhook-payload-properties"></a>Propiedades de la carga de webhook

Las acciones de webhook permiten invocar una única solicitud HTTP POST. El servicio al que se llama debe admitir webhooks y saber cómo usar la carga que reciba.

Propiedades de acción de webhook predeterminadas y sus nombres de parámetros JSON personalizados:

| Parámetro | Variable | Descripción |
|:--- |:--- |:--- |
| *AlertRuleName* |#alertrulename |Nombre de la regla de alerta. |
| *Gravedad* |#severity |Gravedad establecida en la alerta de registros activada. |
| *AlertThresholdOperator* |ThresholdOperator |Operador de umbral para la regla de alerta. |
| *AlertThresholdValue* |#thresholdvalue |Valor de umbral para la regla de alerta. |
| *LinkToSearchResults* |#linktosearchresults |Vínculo al portal de Analytics que devuelve los registros de la consulta que creó la alerta. |
| *LinkToSearchResultsAPI* |#linktosearchresultsapi |Vínculo a la API de Analytics que devuelve los registros de la consulta que creó la alerta. |
| *LinkToFilteredSearchResultsUI* |#linktofilteredsearchresultsui |Vínculo al portal de Analytics que devuelve los registros de la consulta filtrada por las combinaciones de valores de dimensiones que crearon la alerta. |
| *LinkToFilteredSearchResultsAPI* |#linktofilteredsearchresultsapi |Vínculo a la API de Analytics que devuelve los registros de la consulta filtrada por las combinaciones de valores de dimensiones que crearon la alerta. |
| *ResultCount* |#searchresultcount |Número de registros en los resultados de la búsqueda. |
| *SearchIntervalEndtimeUtc* |#searchintervalendtimeutc |Hora de finalización de la consulta en hora UTC, con formato mm/dd/aaaa HH:mm:ss AM/PM. |
| *SearchIntervalInSeconds* |#searchinterval |Período de tiempo para la regla de alertas, con formato HH:mm:ss. |
| *SearchIntervalStartTimeUtc* |#searchintervalstarttimeutc |Hora de inicio de la consulta en hora UTC, con formato mm/dd/aaaa HH:mm:ss AM/PM. 
| *SearchQuery* |#searchquery |Consulta de búsqueda de registros utilizada por la regla de alerta. |
| *SearchResults* |"IncludeSearchResults": true|Los registros devueltos por la consulta en forma de una tabla JSON, limitados a los primeros 1000 registros. Se agrega "IncludeSearchResults": true en una definición personalizada de webhook de JSON como una propiedad de nivel superior. |
| *Dimensiones* |"IncludeDimensions": true|Combinaciones de valores de dimensiones que desencadenaron esa alerta como una sección JSON. Se agrega "IncludeDimensions": true en una definición personalizada de webhook de JSON como una propiedad de nivel superior. |
| *AlertType*| #alerttype | El tipo de regla de alertas de registro configurada, bien como [Unidades métricas o como Número de resultados](./alerts-unified-log.md#measure).|
| *WorkspaceID* |#workspaceid |Identificador del área de trabajo de Log Analytics. |
| *Identificador de la aplicación* |#applicationid |Identificador de la aplicación Application Insights. |
| *Subscription ID* |#subscriptionid |Identificador de la suscripción a Azure usada. |

## <a name="custom-webhook-payload-definition"></a>Definición de carga de webhook personalizada

Puede usar la opción **Incluir la carga personalizada de JSON para webhook** para obtener una carga de JSON personalizada mediante los parámetros anteriores. También puede generar propiedades adicionales.
Por ejemplo, podría especificar la siguiente carga personalizada que incluye un único parámetro denominado *text*. El servicio al que llama este webhook espera este parámetro:

```json

    {
        "text":"#alertrulename fired with #searchresultcount over threshold of #thresholdvalue."
    }
```
Esta carga de ejemplo se resuelve en algo similar a lo siguiente al enviarse al webhook:

```json
    {
        "text":"My Alert Rule fired with 18 records over threshold of 10 ."
    }
```
Las variables de un webhook personalizado se deben especificar dentro de un contenedor JSON. Por ejemplo, si se hace referencia a "#searchresultcount" en el ejemplo de webhook anterior, se generará la salida en función de los resultados de la alerta.

Para incluir los resultados de la búsqueda, agregue **IncludeSearchResults** como propiedad de nivel superior en la carga JSON personalizada. Los resultados de la búsqueda se incluyen como una estructura JSON, por lo que no se puede hacer referencia a ellos en campos definidos personalizados. 

> [!NOTE]
> El botón **View webhook (Ver webhook)** situado junto a la opción **Incluir la carga personalizada de JSON para webhook** muestra la vista previa de lo que se proporcionó. No contiene datos reales, pero es representativo del esquema JSON que se usará. 

## <a name="sample-payloads"></a>Cargas de ejemplo
En esta sección se muestran cargas de ejemplo para los webhooks de alertas de registro. Estas cargas incluyen ejemplos en los que la carga es estándar y otros en los que es personalizada.

### <a name="log-alert-for-log-analytics"></a>Alerta de registro para Log Analytics
La carga de ejemplo siguiente corresponde a una acción de webhook estándar que se usa para las alertas basadas en Log Analytics:

> [!NOTE]
> El valor del campo "Gravedad" cambia si ha [cambiado a la versión scheduledQueryRules API actual](../alerts/alerts-log-api-switch.md) de [Alert API de Log Analytics heredada](./api-alerts.md).

```json
{
   "schemaId":"Microsoft.Insights/LogAlert",
   "data":{
      "SubscriptionId":"12345a-1234b-123c-123d-12345678e",
      "AlertRuleName":"AcmeRule",
      "SearchQuery":"Perf | where ObjectName == \"Processor\" and CounterName == \"% Processor Time\" | summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 5m), Computer",
      "SearchIntervalStartTimeUtc":"2018-03-26T08:10:40Z",
      "SearchIntervalEndtimeUtc":"2018-03-26T09:10:40Z",
      "AlertThresholdOperator":"Greater Than",
      "AlertThresholdValue":0,
      "ResultCount":2,
      "SearchIntervalInSeconds":3600,
      "LinkToSearchResults":"https://portal.azure.com/#Analyticsblade/search/index?_timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
      "LinkToFilteredSearchResultsUI":"https://portal.azure.com/#Analyticsblade/search/index?_timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
      "LinkToSearchResultsAPI":"https://api.loganalytics.io/v1/workspaces/workspaceID/query?query=Heartbeat&timespan=2020-05-07T18%3a11%3a51.0000000Z%2f2020-05-07T18%3a16%3a51.0000000Z",
      "LinkToFilteredSearchResultsAPI":"https://api.loganalytics.io/v1/workspaces/workspaceID/query?query=Heartbeat&timespan=2020-05-07T18%3a11%3a51.0000000Z%2f2020-05-07T18%3a16%3a51.0000000Z",
      "Description":"log alert rule",
      "Severity":"Warning",
      "AffectedConfigurationItems":[
         "INC-Gen2Alert"
      ],
      "Dimensions":[
         {
            "name":"Computer",
            "value":"INC-Gen2Alert"
         }
      ],
      "SearchResult":{
         "tables":[
            {
               "name":"PrimaryResult",
               "columns":[
                  {
                     "name":"$table",
                     "type":"string"
                  },
                  {
                     "name":"Computer",
                     "type":"string"
                  },
                  {
                     "name":"TimeGenerated",
                     "type":"datetime"
                  }
               ],
               "rows":[
                  [
                     "Fabrikam",
                     "33446677a",
                     "2018-02-02T15:03:12.18Z"
                  ],
                  [
                     "Contoso",
                     "33445566b",
                     "2018-02-02T15:16:53.932Z"
                  ]
               ]
            }
         ]
      },
      "WorkspaceId":"12345a-1234b-123c-123d-12345678e",
      "AlertType":"Metric measurement"
   }
}
```

### <a name="log-alert-for-application-insights"></a>Alerta de registro para Application Insights
La carga de ejemplo siguiente corresponde a un webhook estándar que se usa para alertas de registro basadas en recursos de Application Insights:
    
```json
{
    "schemaId": "Microsoft.Insights/LogAlert",
    "data": {
        "SubscriptionId": "12345a-1234b-123c-123d-12345678e",
        "AlertRuleName": "AcmeRule",
        "SearchQuery": "requests | where resultCode == \"500\" | summarize AggregatedValue = Count by bin(Timestamp, 5m), IP",
        "SearchIntervalStartTimeUtc": "2018-03-26T08:10:40Z",
        "SearchIntervalEndtimeUtc": "2018-03-26T09:10:40Z",
        "AlertThresholdOperator": "Greater Than",
        "AlertThresholdValue": 0,
        "ResultCount": 2,
        "SearchIntervalInSeconds": 3600,
        "LinkToSearchResults": "https://portal.azure.com/AnalyticsBlade/subscriptions/12345a-1234b-123c-123d-12345678e/?query=search+*+&timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
        "LinkToFilteredSearchResultsUI": "https://portal.azure.com/AnalyticsBlade/subscriptions/12345a-1234b-123c-123d-12345678e/?query=search+*+&timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
        "LinkToSearchResultsAPI": "https://api.applicationinsights.io/v1/apps/0MyAppId0/metrics/requests/count",
        "LinkToFilteredSearchResultsAPI": "https://api.applicationinsights.io/v1/apps/0MyAppId0/metrics/requests/count",
        "Description": null,
        "Severity": "3",
        "Dimensions": [
            {
                "name": "IP",
                "value": "1.1.1.1"
            }
        ],
        "SearchResult": {
            "tables": [
                {
                    "name": "PrimaryResult",
                    "columns": [
                        {
                            "name": "$table",
                            "type": "string"
                        },
                        {
                            "name": "Id",
                            "type": "string"
                        },
                        {
                            "name": "Timestamp",
                            "type": "datetime"
                        }
                    ],
                    "rows": [
                        [
                            "Fabrikam",
                            "33446677a",
                            "2018-02-02T15:03:12.18Z"
                        ],
                        [
                            "Contoso",
                            "33445566b",
                            "2018-02-02T15:16:53.932Z"
                        ]
                    ]
                }
            ]
        },
        "ApplicationId": "123123f0-01d3-12ab-123f-abc1ab01c0a1",
        "AlertType": "Metric measurement"
    }
}
```

### <a name="log-alert-for-other-resources-logs-from-api-version-2020-05-01-preview"></a>Alerta de registro para otros registros de recursos (a partir de la versión `2020-05-01-preview` de la API)

> [!NOTE]
> Actualmente no hay cargos adicionales por la versión `2020-05-01-preview` de la API y las alertas de registro orientadas a los recursos.  Los precios de las características en versión preliminar se anunciarán en el futuro y se avisará antes del inicio de la facturación. Si decide seguir usando la nueva versión de la API y las alertas de registro orientadas a los recursos después del período de aviso, se le facturará según la tarifa aplicable.

La carga de ejemplo siguiente corresponde a un webhook estándar cuando se usa para las alertas de registro basadas en otros registros de recursos (excepto las áreas de trabajo y Application Insights):

```json
{
    "schemaId": "azureMonitorCommonAlertSchema",
    "data": {
        "essentials": {
            "alertId": "/subscriptions/12345a-1234b-123c-123d-12345678e/providers/Microsoft.AlertsManagement/alerts/12345a-1234b-123c-123d-12345678e",
            "alertRule": "AcmeRule",
            "severity": "Sev4",
            "signalType": "Log",
            "monitorCondition": "Fired",
            "monitoringService": "Log Alerts V2",
            "alertTargetIDs": [
                "/subscriptions/12345a-1234b-123c-123d-12345678e/resourcegroups/ai-engineering/providers/microsoft.compute/virtualmachines/testvm"
            ],
            "originAlertId": "123c123d-1a23-1bf3-ba1d-dd1234ff5a67",
            "firedDateTime": "2020-07-09T14:04:49.99645Z",
            "description": "log alert rule V2",
            "essentialsVersion": "1.0",
            "alertContextVersion": "1.0"
        },
        "alertContext": {
            "properties": null,
            "conditionType": "LogQueryCriteria",
            "condition": {
                "windowSize": "PT10M",
                "allOf": [
                    {
                        "searchQuery": "Heartbeat",
                        "metricMeasure": null,
                        "targetResourceTypes": "['Microsoft.Compute/virtualMachines']",
                        "operator": "LowerThan",
                        "threshold": "1",
                        "timeAggregation": "Count",
                        "dimensions": [
                            {
                                "name": "ResourceId",
                                "value": "/subscriptions/12345a-1234b-123c-123d-12345678e/resourceGroups/TEST/providers/Microsoft.Compute/virtualMachines/testvm"
                            }
                        ],
                        "metricValue": 0.0,
                        "failingPeriods": {
                            "numberOfEvaluationPeriods": 1,
                            "minFailingPeriodsToAlert": 1
                        },
                        "linkToSearchResultsUI": "https://portal.azure.com#@12f345bf-12f3-12af-12ab-1d2cd345db67/blade/Microsoft_Azure_Monitoring_Logs/LogsBlade/source/Alerts.EmailLinks/scope/%7B%22resources%22%3A%5B%7B%22resourceId%22%3A%22%2Fsubscriptions%2F12345a-1234b-123c-123d-12345678e%2FresourceGroups%2FTEST%2Fproviders%2FMicrosoft.Compute%2FvirtualMachines%2Ftestvm%22%7D%5D%7D/q/eJzzSE0sKklKTSypUSjPSC1KVQjJzE11T81LLUosSU1RSEotKU9NzdNIAfJKgDIaRgZGBroG5roGliGGxlYmJlbGJnoGEKCpp4dDmSmKMk0A/prettify/1/timespan/2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z",
                        "linkToFilteredSearchResultsUI": "https://portal.azure.com#@12f345bf-12f3-12af-12ab-1d2cd345db67/blade/Microsoft_Azure_Monitoring_Logs/LogsBlade/source/Alerts.EmailLinks/scope/%7B%22resources%22%3A%5B%7B%22resourceId%22%3A%22%2Fsubscriptions%2F12345a-1234b-123c-123d-12345678e%2FresourceGroups%2FTEST%2Fproviders%2FMicrosoft.Compute%2FvirtualMachines%2Ftestvm%22%7D%5D%7D/q/eJzzSE0sKklKTSypUSjPSC1KVQjJzE11T81LLUosSU1RSEotKU9NzdNIAfJKgDIaRgZGBroG5roGliGGxlYmJlbGJnoGEKCpp4dDmSmKMk0A/prettify/1/timespan/2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z",
                        "linkToSearchResultsAPI": "https://api.loganalytics.io/v1/subscriptions/12345a-1234b-123c-123d-12345678e/resourceGroups/TEST/providers/Microsoft.Compute/virtualMachines/testvm/query?query=Heartbeat%7C%20where%20TimeGenerated%20between%28datetime%282020-07-09T13%3A44%3A34.0000000%29..datetime%282020-07-09T13%3A54%3A34.0000000%29%29&timespan=2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z",
                        "linkToFilteredSearchResultsAPI": "https://api.loganalytics.io/v1/subscriptions/12345a-1234b-123c-123d-12345678e/resourceGroups/TEST/providers/Microsoft.Compute/virtualMachines/testvm/query?query=Heartbeat%7C%20where%20TimeGenerated%20between%28datetime%282020-07-09T13%3A44%3A34.0000000%29..datetime%282020-07-09T13%3A54%3A34.0000000%29%29&timespan=2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z"
                    }
                ],
                "windowStartTime": "2020-07-07T13:54:34Z",
                "windowEndTime": "2020-07-09T13:54:34Z"
            }
        }
    }
}
```

### <a name="log-alert-with-a-custom-json-payload"></a>Alerta de registro con una carga de JSON personalizada
Por ejemplo, para crear una carga personalizada que solo incluya el nombre de la alerta y los resultados de la búsqueda, use esta configuración: 

```json
    {
       "alertname":"#alertrulename",
       "IncludeSearchResults":true
    }
```

A continuación se muestra una carga de ejemplo que corresponde a una acción de webhook personalizada para cualquier alerta de registro:
    
```json
    {
    "alertname":"AcmeRule","IncludeSearchResults":true,
    "SearchResults":
        {
        "tables":[
                    {"name":"PrimaryResult","columns":
                        [
                        {"name":"$table","type":"string"},
                        {"name":"Id","type":"string"},
                        {"name":"TimeGenerated","type":"datetime"}
                        ],
                    "rows":
                        [
                            ["Fabrikam","33446677a","2018-02-02T15:03:12.18Z"],
                            ["Contoso","33445566b","2018-02-02T15:16:53.932Z"]
                        ]
                    }
                ]
        }
    }
```

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre las [alertas de registro en las alertas de Azure](./alerts-unified-log.md).
- Aprenda a [administrar alertas de registro en Azure](alerts-log.md).
- Cree y administre [grupos de acciones en Azure](./action-groups.md).
- Más información sobre [Application Insights](../logs/log-query-overview.md).
- Obtenga más información sobre las [consultas de registro](../logs/log-query-overview.md).
