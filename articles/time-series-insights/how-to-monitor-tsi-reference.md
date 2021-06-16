---
title: Supervisión de la referencia de datos de Azure Time Series Insights | Microsoft Docs
description: Documentación de referencia para supervisar Azure Time Series Insights.
author: esung22
ms.author: elsung
manager: cnovak
ms.reviewer: orspodek
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 12/10/2020
ms.openlocfilehash: 52f2ea8b88bb7f83a3a897d101030d134749fc1a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110792055"
---
# <a name="monitoring-azure-time-series-insights-data-reference"></a>Supervisión de la referencia de datos de Azure Time Series Insights

Obtenga información sobre los datos y los recursos recopilados por Azure Monitor de su entorno de Azure Time Series Insights. Consulte [Supervisión de Azure Time Series Insights]( ./how-to-monitor-tsi.md) para más información sobre la recopilación y el análisis de datos de supervisión.

## <a name="metrics"></a>Métricas

En esta sección se indican todas las métricas de la plataforma recopiladas automáticamente de Azure Time Series Insights. Para ver una lista de todas las métricas compatibles con Azure Monitor (incluido Azure Time Series Insights), consulte [Métricas compatibles con Azure Monitor](../azure-monitor/essentials/metrics-supported.md).
El proveedor de recursos de estas métricas es [Microsoft.TimeSeriesInsights/environments/eventsources](../azure-monitor/essentials/metrics-supported.md#microsofttimeseriesinsightsenvironmentseventsources) y [Microsoft.TimeSeriesInsights/environments](../azure-monitor/essentials/metrics-supported.md#microsofttimeseriesinsightsenvironments).

### <a name="ingress"></a>Entrada

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|
|---|---|---|---|---|
|IngressReceivedBytes|Bytes de entrada recibidos|Bytes|Total|Número de bytes leídos desde el origen del evento|
|IngressReceivedInvalidMessages|Mensajes no válidos de entrada recibidos|Count|Total|Número de mensajes leídos no válidos desde el origen del evento|
|IngressReceivedMessages|Mensajes de entrada recibidos|Count|Total|Número de mensajes leídos desde el origen del evento|
|IngressReceivedMessagesCountLag|Intervalo de recuento de los mensajes de entrada recibidos|Count|Average|Diferencia entre el número de secuencia del último mensaje en cola en la partición del origen del evento y el del mensaje que se está procesando en la entrada.|
|IngressReceivedMessagesTimeLag|Tiempo de retardo de los mensajes de entrada recibidos|Segundos|Máxima|Diferencia entre el momento en el que el mensaje se pone en la cola en el origen del evento y el momento en que se procesa en la entrada|
|IngressStoredBytes|Bytes de entrada almacenados|Bytes|Total|Tamaño total de los eventos almacenados correctamente y disponibles para su consulta|
|IngressStoredEvents|Eventos de entrada almacenados|Count|Total|Número de eventos acoplados que se procesaron correctamente y que están disponibles para su consulta|

### <a name="storage"></a>Storage

|Métrica|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|
|---|---|---|---|---|
|WarmStorageMaxProperties|Propiedades máximas del almacenamiento intermedio|Count|Máxima|Número máximo de propiedades utilizadas que permite el entorno para la SKU S1/S2 y número máximo de propiedades admitido por el almacenamiento intermedio para la SKU de pago por uso|
|WarmStorageUsedProperties|Propiedades utilizadas de almacenamiento intermedio |Count|Máxima|Número de propiedades utilizadas por el entorno para la SKU S1/S2 y número de propiedades que utiliza el almacenamiento intermedio para la SKU de pago por uso|

## <a name="resource-logs"></a>Registros del recurso

En esta sección se enumeran los tipos de registros de recursos que se pueden recopilar para un entorno de Azure Time Series Insights.

| Category | Display Name (Nombre para mostrar) | Descripción |
|----- |----- |----- |
| Entrada | TSIIngress | La categoría Entrada realiza un seguimiento de los errores que se producen en la canalización de entrada. Esta categoría incluye los errores que se producen al recibir eventos (como errores de conexión a un origen de eventos) y al procesamiento eventos (como errores al analizar una carga de eventos). |

## <a name="schemas"></a>Esquemas

Azure Time Series Insights usa los siguientes esquemas

### <a name="tsiingress-table"></a>Tabla TSIIngress

| Propiedad. | Descripción |
|----- |----- |
| TimeGenerated | Hora (UTC) a la que se genera este evento. |
| Location | Ubicación del recurso. |
| Category | Categoría del evento de registro. |
| OperationName | Nombre de operación del evento. |
| CorrelationId | El Id. de correlación de la solicitud. |
| Nivel | Nivel de gravedad del evento. |
| ResultDescription | Descripción del resultado de la operación, como "Error de prohibido recibido". |
| Message | El mensaje asociado al error. Incluye detalles sobre lo que ha ido mal y cómo mitigar el error. |
| ErrorCode | El código asociado al error |
| EventSourceType | Tipo de origen del evento. Puede ser centro de eventos o centro de IoT. |
| EventSourceProperties | Colección de propiedades específicas del origen de eventos. Contiene detalles como el grupo de consumidores y el nombre de la clave de acceso. |
