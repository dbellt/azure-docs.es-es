---
title: 'Análisis sobre la base de conocimiento: QnA Maker'
titleSuffix: Azure Cognitive Services
description: QnA Maker almacena todos los registros de chat y otra telemetría, si ha habilitado Application Insights durante la creación del servicio QnA Maker. Ejecute las consultas de ejemplo para obtener los registros de chat de Application Insights.
services: cognitive-services
manager: nitinme
displayName: chat history, history, chat logs, logs
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 41477b99c6bb07ab836f6c4ef75a06b2f642f3a2
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375431"
---
# <a name="get-analytics-on-your-knowledge-base"></a>Obtener análisis en la base de conocimiento

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

QnA Maker almacena todos los registros de chat y otra telemetría, si ha habilitado Application Insights durante la [creación del servicio QnA Maker](./set-up-qnamaker-service-azure.md). Ejecute las consultas de ejemplo para obtener los registros de chat de Application Insights.

1. Vaya al recurso de Application Insights.

    ![Selección del recurso de Application Insights](../media/qnamaker-how-to-analytics-kb/resources-created.png)

2. Selecciona **Log (Analytics)** (Registro [análisis]). Se abre una nueva ventana en la que puede consultar la telemetría de QnA Maker.

3. Pegue la consulta siguiente y ejecútela.

    ```kusto
    requests
    | where url endswith "generateAnswer"
    | project timestamp, id, url, resultCode, duration, performanceBucket
    | parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
    | join kind= inner (
    traces | extend id = operation_ParentId
    ) on id
    | extend question = tostring(customDimensions['Question'])
    | extend answer = tostring(customDimensions['Answer'])
    | extend score = tostring(customDimensions['Score'])
    | project timestamp, resultCode, duration, id, question, answer, score, performanceBucket,KbId
    ```

    Seleccione **Ejecutar** para ejecutar la consulta.

    [![Ejecute la consulta para determinar las preguntas, respuestas y puntuaciones de los usuarios](../media/qnamaker-how-to-analytics-kb/run-query.png)](../media/qnamaker-how-to-analytics-kb/run-query.png#lightbox)

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

Respuesta a preguntas personalizada (versión preliminar) usa el registro de diagnóstico de Azure para almacenar los datos de telemetría y los registros de chat. Siga los pasos que se indican a continuación para ejecutar consultas de ejemplo para obtener análisis sobre el uso de la knowledge base de QnA Maker.

1. [Habilite el registro de diagnóstico](../../diagnostic-logging.md) del servicio Text Analytics con la característica de respuesta a preguntas personalizada (versión preliminar).

2. En el paso anterior, seleccione **Trace** (Seguimiento), además de **Audit, RequestResponse y AllMetrics** para el registro

    ![Habilitación del registro de seguimiento en QnA Maker administrado (versión preliminar)](../media/qnamaker-how-to-analytics-kb/qnamaker-v2-enable-trace-logging.png)

---

## <a name="run-queries-for-other-analytics-on-your-qna-maker-knowledge-base"></a>Ejecución de consultas de otros análisis en la base de conocimiento de QnA Maker

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

### <a name="total-90-day-traffic"></a>Tráfico total de 90 días

```kusto
//Total Traffic
requests
| where url endswith "generateAnswer" and name startswith "POST"
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| summarize ChatCount=count() by bin(timestamp, 1d), KbId
```

### <a name="total-question-traffic-in-a-given-time-period"></a>Tráfico total de preguntas en un período de tiempo determinado

```kusto
//Total Question Traffic in a given time period
let startDate = todatetime('2019-01-01');
let endDate = todatetime('2020-12-31');
requests
| where timestamp <= endDate and timestamp >=startDate
| where url endswith "generateAnswer" and name startswith "POST"
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| summarize ChatCount=count() by KbId
```

### <a name="user-traffic"></a>Tráfico de usuario

```kusto
//User Traffic
requests
| where url endswith "generateAnswer"
| project timestamp, id, url, resultCode, duration
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| join kind= inner (
traces | extend id = operation_ParentId
) on id
| extend UserId = tostring(customDimensions['UserId'])
| summarize ChatCount=count() by bin(timestamp, 1d), UserId, KbId
```

### <a name="latency-distribution-of-questions"></a>Distribución de latencia de preguntas

```kusto
//Latency distribution of questions
requests
| where url endswith "generateAnswer" and name startswith "POST"
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| project timestamp, id, name, resultCode, performanceBucket, KbId
| summarize count() by performanceBucket, KbId
```

### <a name="unanswered-questions"></a>Preguntas sin responder

```kusto
// Unanswered questions
requests
| where url endswith "generateAnswer"
| project timestamp, id, url
| parse kind = regex url with *"(?i)knowledgebases/"KbId"/generateAnswer"
| join kind= inner (
traces | extend id = operation_ParentId
) on id
| extend question = tostring(customDimensions['Question'])
| extend answer = tostring(customDimensions['Answer'])
| extend score = tostring(customDimensions['Score'])
| where  score  == "0" and message == "QnAMaker GenerateAnswer"
| project timestamp, KbId, question, answer, score
| order  by timestamp  desc
```

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

### <a name="all-qna-chat-log"></a>Todos los registros de chat de QnA

```kusto
// All QnA Traffic
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where OperationName=="QnAMaker GenerateAnswer"
| extend answer_ = tostring(parse_json(properties_s).answer)
| extend question_ = tostring(parse_json(properties_s).question)
| extend score_ = tostring(parse_json(properties_s).score)
| extend kbId_ = tostring(parse_json(properties_s).kbId)
| project question_, answer_, score_, kbId_
```

### <a name="traffic-count-per-knowledge-base-and-user-in-a-time-period"></a>Recuento de tráfico por knowledge base y usuario en un período

```kusto
// Traffic count per KB and user in a time period
let startDate = todatetime('2019-01-01');
let endDate = todatetime('2020-12-31');
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where OperationName=="QnAMaker GenerateAnswer"
| where TimeGenerated <= endDate and TimeGenerated >=startDate
| extend kbId_ = tostring(parse_json(properties_s).kbId)
| extend userId_ = tostring(parse_json(properties_s).userId)
| summarize ChatCount=count() by bin(TimeGenerated, 1d), kbId_, userId_
```

### <a name="latency-of-generateanswer-api"></a>Latencia de GenerateAnswer API

```kusto
// Latency of GenerateAnswer
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where OperationName=="Generate Answer"
| project TimeGenerated, DurationMs
| render timechart
```

### <a name="average-latency-of-all-operations"></a>Latencia media de todas las operaciones

```kusto
// Average Latency of all operations
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| project DurationMs, OperationName
| summarize count(), avg(DurationMs) by OperationName
| render barchart
```

### <a name="unanswered-questions"></a>Preguntas sin responder

```kusto
// All unanswered questions
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
| where OperationName=="QnAMaker GenerateAnswer"
| extend answer_ = tostring(parse_json(properties_s).answer)
| extend question_ = tostring(parse_json(properties_s).question)
| extend score_ = tostring(parse_json(properties_s).score)
| extend kbId_ = tostring(parse_json(properties_s).kbId)
| where score_ == 0
| project question_, answer_, score_, kbId_
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Selección de la capacidad](./improve-knowledge-base.md)
