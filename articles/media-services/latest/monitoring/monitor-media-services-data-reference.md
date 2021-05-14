---
title: Supervisión de la referencia de datos de Media Services
description: Material de referencia importante necesario al supervisar Media Services
author: IngridAtMicrosoft
ms.author: inhenkel
manager: femila
ms.topic: reference
ms.service: media-services
ms.custom: subject-monitoring
ms.date: 04/21/2021
ms.openlocfilehash: a9bc17528e1314a033cf2e45fee4b112eb088cc3
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715581"
---
# <a name="monitoring-media-services-data-reference"></a>Supervisión de la referencia de datos de Media Services

En este artículo se tratan los datos que son útiles para la supervisión de Media Services. Para más información sobre todas las métricas de la plataforma que se admiten en Azure Monitor, consulte [Métricas compatibles con Azure Monitor](../../../azure-monitor/essentials/metrics-supported.md).

## <a name="metrics"></a>Métricas

Las métricas se recopilan a intervalos regulares independientemente de si cambia el valor o no. Son útiles para las alertas, dado que se pueden muestrear con frecuencia y se puede activar una alerta con rapidez con una lógica relativamente sencilla.


Media Services permite supervisar las métricas para los siguientes recursos:

|Tipo de métrica | Espacio de nombres de proveedor de recursos/tipo<br/> y vínculo a métricas individuales |
|-------|-----|
| General de Media Services | [General](../../../azure-monitor/essentials/metrics-supported.md#microsoftmediamediaservices) |
| Eventos en vivo | [Microsoft.Media/mediaservices/liveEvents](../../../azure-monitor/essentials/metrics-supported.md#microsoftmediamediaservicesliveevents) 
| Extremos de streaming | [Microsoft.Media/mediaservices/streamingEndpoints](../../../azure-monitor/essentials/metrics-supported.md#microsoftmediamediaservicesstreamingendpoints), que son pertinentes a la [API REST de puntos de conexión de streaming](/rest/api/media/streamingendpoints). 


También debe revisar las [cuotas y límites](../limits-quotas-constraints-reference.md) de la cuenta.


## <a name="metric-dimensions"></a>Dimensiones de métricas

Para obtener más información sobre las dimensiones de métricas, consulte [Métricas multidimensionales](../../../azure-monitor/essentials/data-platform-metrics.md#multi-dimensional-metrics).

Media Services tiene las siguientes dimensiones de métrica.  Se explican por sí solas en función de las métricas que admiten.  Para obtener más información, consulte los [vínculos de métricas](#metrics) anteriormente.   
- OutputFormat
- HttpStatusCode 
- ErrorCode 
- TrackName 

## <a name="resource-logs"></a>Registros del recurso

Los registros de recursos proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de un recurso de Azure. Para más información, aprenda a [recopilar y usar los datos de registro provenientes de los recursos de Azure](../../../azure-monitor/essentials/platform-logs-overview.md).

Media Services admite los siguientes registros de recursos: [Microsoft.Media/mediaservices](../../../azure-monitor/essentials/resource-logs-categories.md#microsoftmediamediaservices)

## <a name="schemas"></a>Esquemas

Para obtener una descripción detallada del esquema de registros de diagnóstico de nivel superior, consulte [Servicios, esquemas y categorías admitidos en los registros de diagnóstico de Azure](../../../azure-monitor/essentials/resource-logs-schema.md).

### <a name="key-delivery"></a>Entrega de claves 

Estas propiedades son específicas del esquema de registro de entrega de claves.

|Nombre|Descripción|
|---|---|
|keyId|Identificador de la clave solicitada.|
|keyType|Puede ser uno de los siguientes valores: "Clear" (sin cifrado), "FairPlay", "PlayReady" o "Widevine".|
|policyName|Nombre de Azure Resource Manager de la directiva.|
|tokenType|El tipo del token.|
|statusMessage|Mensaje de estado.|

### <a name="example"></a>Ejemplo

Propiedades del esquema de solicitudes de entrega de claves.

```json
{
    "time": "2019-01-11T17:59:10.4908614Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Succeeded",
    "resultSignature": "OK",
    "durationMs": 315,
    "identity": {
        "authorization": {
            "issuer": "http://testacs",
            "audience": "urn:test"
        },
        "claims": {
            "urn:microsoft:azure:mediaservices:contentkeyidentifier": "3321e646-78d0-4896-84ec-c7b98eddfca5",
            "iss": "http://testacs",
            "aud": "urn:test",
            "exp": "1547233138"
        }
    },
    "level": "Informational",
    "location": "uswestcentral",
    "properties": {
        "requestId": "b0243468-d8e5-4edf-a48b-d408e1661050",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "JWT",
        "statusMessage": "OK"
    }
} 
```

```json
 {
    "time": "2019-01-11T17:59:33.4676382Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Failed",
    "resultSignature": "Unauthorized",
    "durationMs": 2,
    "level": "Error",
    "location": "uswestcentral",
    "properties": {
        "requestId": "875af030-b77c-416b-b7e1-58f23ebec182",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "None",
        "statusMessage": "No token present in authorization header or URL."
    }
} 
```

>[!NOTE]
> Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [monitoring-next-steps](../includes/monitoring-next-steps.md)]
