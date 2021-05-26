---
title: 'Azure Event Grid en Kubernetes: características'
description: En este artículo se comparan las características de Event Grid en Kubernetes con Event Grid en Azure.
author: jfggdl
ms.subservice: kubernetes
ms.author: jafernan
ms.date: 05/25/2021
ms.topic: conceptual
ms.openlocfilehash: af18458e0370846a6c34a1728dfd7c98ca1276a5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388736"
---
# <a name="event-grid-on-kubernetes-with-azure-arc-features"></a>Características de Event Grid en Kubernetes con Azure Arc
Event Grid en Kubernetes ofrece un amplio conjunto de características que le ayudan a integrar las cargas de trabajo de Kubernetes y a llevar a cabo arquitecturas híbridas. Comparte la misma [API REST](/rest/api/eventgrid/version2020-10-15-preview/topics) (a partir de la versión 2020-10-15-preview), la [CLI de Event Grid](/cli/azure/eventgrid), la experiencia de Azure Portal, los [SDK de administración](../sdk-overview.md#management-sdks) y los [SDK del plano de datos](../sdk-overview.md#data-plane-sdks) con Azure Event Grid, la otra edición del mismo servicio. Cuando esté listo para publicar eventos, puede usar los [ejemplos del SDK del plano de datos proporcionados en distintos lenguajes](https://devblogs.microsoft.com/azure-sdk/event-grid-ga/) que funcionan para ambas ediciones de Event Grid.

Aunque Event Grid en Kubernetes y Azure Event Grid comparten muchas características y el objetivo es proporcionar la misma experiencia de usuario, hay algunas diferencias dadas las necesidades únicas que buscan cumplir y la fase en la que están en su ciclo de vida de software. Por ejemplo, el único tipo de tema disponible en Event Grid en Kubernetes son los temas de Event Grid, que a veces también se conocen como temas personalizados. Otros tipos de temas (consulte a continuación) no son aplicables o la compatibilidad con ellos aún no está disponible. En la tabla siguiente se presentan las principales diferencias entre las dos ediciones de Event Grid.

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]


## <a name="event-grid-on-kubernetes-vs-event-grid-on-azure"></a>Event Grid en Kubernetes frente a Event Grid en Azure

| Característica | Event Grid en Kubernetes | Azure Event Grid |
|:--|:-:|:-:|
| [Temas de Event Grid](/rest/api/eventgrid/version2020-10-15-preview/topics) | ✔ | ✔ |
| [Esquema de eventos en la nube de CNCF](https://github.com/cloudevents/spec/blob/master/spec.md) | ✔ | ✔ |
| Event Grid y esquemas personalizados | ✘* | ✔ |
| Entrega confiable | ✔ | ✔ |
| Métricas  | ✔** | ✔ |
| Azure Monitor  | ✘ | ✔ |
| [Ubicación de los mensajes con problemas de entrega](../manage-event-delivery.md#set-dead-letter-location) | ✘ | ✔ |
| [Reenvío de eventos a otro tema de Event Grid](event-handlers.md#azure-event-grid) | ✔ | ✘ |
| [Temas del sistema](../system-topics.md) | ✘ | ✔ |
| [Temas de dominio](../event-domains.md) | ✘ | ✔ |
| [Eventos de asociado](../partner-events-overview.md) | ✘ | ✔ |
| [Validación del punto de conexión de destino](../webhook-event-delivery.md#endpoint-validation-with-event-grid-events) | ✘ | ✔ |
| [Desencadenador de Azure Event Grid para Azure Functions](../../azure-functions/functions-bindings-event-grid-trigger.md) | ✘ | ✔ |
| Conexiones híbridas de Azure Relay como destino | ✘ | ✔ |
| [Filtrado avanzado](filter-events.md) | ✔*** | ✔ |
| [AuthN y AuthZ de webhooks con AAD](../secure-webhook-delivery.md) | ✘ | ✔ |
| [Entrega de eventos con una identidad de recurso](/rest/api/eventgrid/version2020-10-15-preview/eventsubscriptions/createorupdate#deliverywithresourceidentity) | ✘ | ✔ |
| El mismo conjunto de SDK del plano de datos | ✔ | ✔ |
| El mismo conjunto de SDK de administración | ✔ | ✔ |
| La misma CLI de Event Grid | ✔ | ✔ |

\* El esquema de Cloud Events 1.0 proporciona un mecanismo de extensibilidad y es un estándar abierto. Esas cualidades o características no las proporcionan los esquemas de Event Grid ni los personalizados. El esquema de Cloud Events 1.0 es una evolución del esquema de Event Grid.

\** Las métricas de los temas y las suscripciones a eventos se proporcionan con el [formato de exposición Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/). Las métricas y otras características de supervisión de Azure Portal no están disponibles actualmente en la versión preliminar.

\*** Event Grid en Kubernetes admite el filtrado avanzado de eventos en función de los valores de los datos de eventos como lo hace Event Grid en Azure, pero hay algunas características y operadores que Event Grid en Kubernetes no admite. Para obtener más información, consulte [Filtrado avanzado](filter-events.md#filter-by-values-in-event-data).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre Event Grid en Kubernetes, consulte [Event Grid en Kubernetes con Azure Arc (versión preliminar): información general](overview.md).