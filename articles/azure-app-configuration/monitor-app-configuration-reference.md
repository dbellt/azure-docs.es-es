---
title: Referencia de datos para la supervisión de Azure App Configuration
description: Material de referencia importante necesario al supervisar App Configuration
services: azure-app-configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 05/05/2021
ms.openlocfilehash: ec6515403574af29c47b03b55eb3cbc42ecd4a7a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968337"
---
# <a name="monitoring-app-configuration-data-reference"></a>Referencia de datos para la supervisión de App Configuration

Este artículo es una referencia de los datos de supervisión recopilados por App Configuration. Consulte [Supervisión de App Configuration](monitor-app-configuration.md) para obtener un tutorial sobre la recopilación y el análisis de datos de supervisión para App Configuration.

## <a name="metrics"></a>Métricas 
Proveedor de recursos y tipo: [métricas de la plataforma de App Configuration](../azure-monitor/essentials/metrics-supported.md#microsoftappconfigurationconfigurationstores)

| Métrica | Unidad | Descripción |
|-------|-----| ----- |
| Número de solicitudes entrantes HTTP   | Count | Número total de solicitudes HTTP entrantes |
|Duración de las solicitudes entrantes HTTP | Milisegundos | Duración de una solicitud HTTP en el lado servidor |
| Número de solicitudes HTTP limitadas | Count |    Las solicitudes limitadas son solicitudes HTTP que devuelven un código de estado 429 (demasiadas solicitudes) |

Para obtener más información, vea una lista de [todas las métricas de la plataforma que se admiten en Azure Monitor](../azure-monitor/essentials/metrics-supported.md).


## <a name="metric-dimensions"></a>Dimensiones de métricas
App Configuration tiene las siguientes dimensiones asociadas a sus métricas.

| Nombre de la métrica | Descripción de la dimensión |
|-------|-----|
| Número de solicitudes entrantes HTTP | Número total de solicitudes HTTP. Las dimensiones admitidas son **HttpStatusCode** o **AuthenticationScheme** de cada solicitud. **AuthenticationScheme** se puede filtrar por autenticación de AAD o HMAC.   |
| Duración de las solicitudes entrantes HTTP | La duración de cada solicitud en el lado servidor. Las dimensiones admitidas son **HttpStatusCode** o **AuthenticationScheme** de cada solicitud. **AuthenticationScheme** se puede filtrar por autenticación de AAD o HMAC. |
| Número de solicitudes HTTP limitadas | Esta métrica no tiene ninguna dimensión. |

 Para obtener más información sobre las dimensiones de métricas, consulte [Métricas multidimensionales](../azure-monitor/essentials/data-platform-metrics.md#multi-dimensional-metrics).

## <a name="resource-logs"></a>Registros del recurso
En esta sección se enumeran los tipos de categoría del registro de recursos recopilados para App Configuration. 

| Tipo de registro de recursos | Más información|
|-------|-----|
| HttpRequest | [Información de la categoría del registro de recursos de App Configuration](../azure-monitor/essentials/resource-logs-categories.md) |

Para más información, consulte una lista de [todos los tipos de categorías de registros de recursos admitidos en Azure Monitor](../azure-monitor/essentials/resource-logs-schema.md).
 
## <a name="azure-monitor-logs-tables"></a>Tablas de registros de Azure Monitor

En esta sección se hace referencia a todas las tablas de Kusto de registros de Azure Monitor importantes para App Configuration y que están disponibles para la consulta mediante Log Analytics.

|Tipo de recurso | Notas |
|-------|-----|
| [AACHttpRequest](/azure/azure-monitor/reference/tables/aachttprequest) | Las entradas de cada solicitud HTTP enviada a un recurso seleccionado de App Configuration. |
| [AzureActivity](/azure/azure-monitor/reference/tables/azureactivity) | Entradas del registro de actividad de Azure que proporcionan información detallada sobre los eventos en el nivel de suscripción o de grupo de administración que se han producido en Azure. |

Puede encontrar una referencia de todas las tablas de registros de Azure Monitor o de Log Analytics en [Referencia de la tabla de registros de Azure Monitor](/azure/azure-monitor/reference/tables/tables-resourcetype).

### <a name="diagnostics-tables"></a>Tablas de diagnóstico

App Configuration usa la [tabla AACHttpRequest](/azure/azure-monitor/reference/tables/aachttprequest) para almacenar información del registro de recursos.

**Solicitudes HTTP**

|Propiedad | Tipo | Descripción |
|-------|-----| ----- |
|Category   |string |La categoría de registro del evento, siempre HttpRequest. 
|ClientIPAddress |  string| Dirección IP del cliente que envió la solicitud.
|ClientRequestId|   string| Identificador de la solicitud proporcionado por el cliente.
|CorrelationId| string| GUID de los registros correlacionados.
|DurationMs|    int |Duración de la operación en milisegundos.
|Cadena del método| HTTP| Método de solicitud HTTP (GET o POST)
|RequestId| string| Identificador de solicitud único generado por el servidor.
|RequestLength| int |Longitud en bytes de la solicitud HTTP.
|RequestURI|    string| El URI de la solicitud puede incluir el nombre de la clave y la etiqueta. 
|_ResourceId|   string| Identificador único del recurso al que está asociado el registro.
|ResponseLength|    int|    Longitud en bytes de la respuesta HTTP.
|SourceSystem| string|  
|StatusCode|    int |Código de estado HTTP de la solicitud.
|TenantId|  string  |WorkspaceId de la solicitud. 
|TimeGenerated| datetime|   Marca de tiempo (UTC) cuando se generó el registro porque se envió una solicitud
|Tipo   |string|    Nombre de la tabla.
|UserAgent| string| Agente de usuario proporcionado por el cliente.


## <a name="see-also"></a>Consulte también

* Consulte [Supervisión de Azure App Configuration](monitor-app-configuration.md) para obtener una descripción de la supervisión de Azure App Configuration.
* Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../azure-monitor/essentials/monitor-azure-resource.md).