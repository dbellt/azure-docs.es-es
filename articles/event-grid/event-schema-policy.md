---
title: Azure Policy como origen de Event Grid
description: En este artículo se describe cómo utilizar Azure Policy como origen de eventos de Event Grid. Proporciona el esquema y los vínculos a los artículos de procedimientos y tutorial.
ms.topic: conceptual
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/29/2021
ms.openlocfilehash: 7723b618910f52d58204711468b482db85ab502c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105734915"
---
# <a name="azure-policy-as-an-event-grid-source"></a>Azure Policy como origen de Event Grid

En este artículo se proporcionan las propiedades y los esquemas de los eventos de [Azure Policy](../governance/policy/index.yml).  Para una introducción a los esquemas de eventos, consulte [Esquema de eventos de Azure Event Grid](./event-schema.md). También proporciona una lista de inicios rápidos y tutoriales para usar Azure Policy como un origen de eventos.

## <a name="available-event-types"></a>Tipos de eventos disponibles

Azure Policy emite los siguientes tipos de eventos:

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.PolicyInsights.PolicyStateCreated | Se genera cuando se crea un estado de cumplimiento de directiva. |
| Microsoft.PolicyInsights.PolicyStateChanged | Se genera cuando se cambia un estado de cumplimiento de directiva. |
| Microsoft.PolicyInsights.PolicyStateDeleted | Se genera cuando se elimina un estado de cumplimiento de directiva. |

## <a name="example-event"></a>Evento de ejemplo

# <a name="event-grid-event-schema"></a>[Esquema de eventos de Event Grid](#tab/event-grid-event-schema)
En el ejemplo siguiente, se muestra el esquema de un evento de creación de estado de directiva: 

```json
[{
    "id": "5829794FCB5075FCF585476619577B5A5A30E52C84842CBD4E2AD73996714C4C",
    "topic": "/subscriptions/<SubscriptionID>",
    "subject": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>",
    "data": {
        "timestamp": "2021-03-27T18:37:42.4496956Z",
        "policyAssignmentId": "<policy-assignment-scope>/providers/microsoft.authorization/policyassignments/<policy-assignment-name>",
        "policyDefinitionId": "<policy-definition-scope>/providers/microsoft.authorization/policydefinitions/<policy-definition-name>",
        "policyDefinitionReferenceId": "",
        "complianceState": "NonCompliant",
        "subscriptionId": "<subscription-id>",
        "complianceReasonCode": ""
    },
    "eventType": "Microsoft.PolicyInsights.PolicyStateCreated",
    "eventTime": "2021-03-27T18:37:42.5241536Z",
    "dataVersion": "1",
    "metadataVersion": "1"
}]
```

El esquema de un evento de cambio de estado de directiva es similar: 

```json
[{
    "id": "5829794FCB5075FCF585476619577B5A5A30E52C84842CBD4E2AD73996714C4C",
    "topic": "/subscriptions/<SubscriptionID>",
    "subject": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>",
    "data": {
        "timestamp": "2021-03-27T18:37:42.4496956Z",
        "policyAssignmentId": "<policy-assignment-scope>/providers/microsoft.authorization/policyassignments/<policy-assignment-name>",
        "policyDefinitionId": "<policy-definition-scope>/providers/microsoft.authorization/policydefinitions/<policy-definition-name>",
        "policyDefinitionReferenceId": "",
        "complianceState": "NonCompliant",
        "subscriptionId": "<subscription-id>",
        "complianceReasonCode": ""
    },
    "eventType": "Microsoft.PolicyInsights.PolicyStateChanged",
    "eventTime": "2021-03-27T18:37:42.5241536Z",
    "dataVersion": "1",
    "metadataVersion": "1"
}]
```
# <a name="cloud-event-schema"></a>[Esquema de eventos en la nube](#tab/cloud-event-schema)

En el ejemplo siguiente, se muestra el esquema de un evento de creación de estado de directiva: 

```json
[{
    "id": "5829794FCB5075FCF585476619577B5A5A30E52C84842CBD4E2AD73996714C4C",
    "source": "/subscriptions/<SubscriptionID>",
    "subject": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>",
    "data": {
        "timestamp": "2021-03-27T18:37:42.4496956Z",
        "policyAssignmentId": "<policy-assignment-scope>/providers/microsoft.authorization/policyassignments/<policy-assignment-name>",
        "policyDefinitionId": "<policy-definition-scope>/providers/microsoft.authorization/policydefinitions/<policy-definition-name>",
        "policyDefinitionReferenceId": "",
        "complianceState": "NonCompliant",
        "subscriptionId": "<subscription-id>",
        "complianceReasonCode": ""
    },
    "type": "Microsoft.PolicyInsights.PolicyStateCreated",
    "time": "2021-03-27T18:37:42.5241536Z",
    "specversion": "1.0"
}]
```

El esquema de un evento de cambio de estado de directiva es similar: 

```json
[{
    "id": "5829794FCB5075FCF585476619577B5A5A30E52C84842CBD4E2AD73996714C4C",
    "source": "/subscriptions/<SubscriptionID>",
    "subject": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>",
    "data": {
        "timestamp": "2021-03-27T18:37:42.4496956Z",
        "policyAssignmentId": "<policy-assignment-scope>/providers/microsoft.authorization/policyassignments/<policy-assignment-name>",
        "policyDefinitionId": "<policy-definition-scope>/providers/microsoft.authorization/policydefinitions/<policy-definition-name>",
        "policyDefinitionReferenceId": "",
        "complianceState": "NonCompliant",
        "subscriptionId": "<subscription-id>",
        "complianceReasonCode": ""
    },
    "type": "Microsoft.PolicyInsights.PolicyStateChanged",
    "time": "2021-03-27T18:37:42.5241536Z",
    "specversion": "1.0"
}]
```

---

## <a name="event-properties"></a>Propiedades de evento

# <a name="event-grid-event-schema"></a>[Esquema de eventos de Event Grid](#tab/event-grid-event-schema)

Un evento tiene los siguientes datos de nivel superior:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| `topic` | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| `subject` | string | Id. completo del recurso para el que se cambia el estado de cumplimiento, incluido el nombre del recurso y el tipo de recurso. Usa el formato `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>` |
| `eventType` | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| `eventTime` | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| `id` | string | Identificador único para el evento |
| `data` | object | Datos de evento de Azure Policy. |
| `dataVersion` | string | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| `metadataVersion` | string | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |

# <a name="cloud-event-schema"></a>[Esquema de eventos en la nube](#tab/cloud-event-schema)

Un evento tiene los siguientes datos de nivel superior:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| `source` | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| `subject` | string | Id. completo del recurso para el que se cambia el estado de cumplimiento, incluido el nombre del recurso y el tipo de recurso. Usa el formato `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>` |
| `type` | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| `time` | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| `id` | string | Identificador único para el evento |
| `data` | object | Datos de evento de Azure Policy. |
| `specversion` | string | Versión de especificación del esquema CloudEvents. |

---

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| `timestamp` | string | La hora (en UTC) en la que Azure Policy examinó el recurso. En el caso de los eventos de ordenación, use esta propiedad en lugar de las propiedades de nivel superior `eventTime` o `time`. |
| `policyAssignmentId` | string | El id. de recurso de la asignación de directiva. |
| `policyDefinitionId` | string | El id. de recurso de la definición de directiva. |
| `policyDefinitionReferenceId` | string | Id. de referencia de la definición de directiva dentro de la definición de la iniciativa, si la asignación de la directiva es para una iniciativa. Puede estar vacío. |
| `complianceState` | string | El estado de cumplimiento del recurso con respecto a la asignación de directiva. |
| `subscriptionId` | string | Identificador de suscripción del recurso. |
| `complianceReasonCode` | string | Código de motivo de cumplimiento. Puede estar vacío. |

## <a name="next-steps"></a>Pasos siguientes

- Para ver un tutorial de los eventos de cambio de estado de Azure Policy, consulte [Uso de Event Grid para notificaciones de cambio en el estado de la directiva](../governance/policy/tutorials/route-state-change-events.md).
- Para obtener información general sobre la integración de Azure Policy con Event Grid, consulte [Reacción a eventos de Azure Policy mediante Event Grid](../governance/policy/concepts/event-overview.md).
- Para una introducción a Azure Event Grid, consulte [Introducción a Azure Event Grid](./overview.md).
- Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](./subscription-creation-schema.md).