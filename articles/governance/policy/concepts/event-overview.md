---
title: Reacción a los eventos de cambio de estado de Azure Policy
description: Use Azure Event Grid para suscribirse a eventos de App Policy, que permiten a las aplicaciones reaccionar a los cambios en los estados sin necesidad de usar código complicado.
ms.date: 03/29/2021
ms.topic: conceptual
ms.openlocfilehash: c100d5038a8c2506f5339ea0962012a8c32e22cf
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105734921"
---
# <a name="reacting-to-azure-policy-state-change-events"></a>Reacción a los eventos de cambio de estado de Azure Policy

Los eventos de Azure Policy permiten a las aplicaciones reaccionar a los cambios de estado. Esta integración se consigue sin necesidad de usar código complejo ni servicios de sondeo costosos e ineficientes. En su lugar, se insertan eventos mediante [Azure Event Grid](../../../event-grid/index.yml) a los suscriptores como [Azure Functions](../../../azure-functions/index.yml), [Azure Logic Apps](../../../logic-apps/index.yml) o incluso su propio cliente de escucha http.
De forma crítica, solo paga por lo que utiliza.

Los eventos de Azure Policy se envían a Azure Event Grid, que proporciona servicios de entrega confiables para sus aplicaciones mediante directivas de reintento enriquecidas y entrega de mensajes fallidos. Para más información, consulte [Entrega y reintento de entrega de mensajes de Event Grid](../../../event-grid/delivery-and-retry.md).

El escenario de eventos común de Azure Policy es realizar el seguimiento cuando cambia el estado de cumplimiento de un recurso durante la evaluación de la directiva. La arquitectura basada en eventos es una manera eficaz de reaccionar a estos cambios en lugar de examinar el estado de cumplimiento de los recursos según una programación fija.

> [!NOTE]
> Los eventos de cambio de estado de Azure Policy se envían a Event Grid después de que un [desencadenador de evaluación](../how-to/get-compliance-data.md#evaluation-triggers) finalice la evaluación de recursos.

Consulte [Enrutamiento de eventos de cambio de estado de directivas a Event Grid con la CLI de Azure](../tutorials/route-state-change-events.md) para obtener un tutorial completo.

:::image type="content" source="../../../event-grid/media/overview/functional-model.png" alt-text="Modelo de orígenes y controladores de Event Grid" lightbox="../../../event-grid/media/overview/functional-model-big.png":::

## <a name="available-azure-policy-events"></a>Eventos de Azure Policy disponibles

Event Grid usa las [suscripciones a eventos](../../../event-grid/concepts.md#event-subscriptions) para enrutar los mensajes de eventos a los suscriptores. Las suscripciones a eventos de Azure Policy pueden incluir tres tipos de eventos:

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.PolicyInsights.PolicyStateCreated | Se genera cuando se crea un estado de cumplimiento de directiva. |
| Microsoft.PolicyInsights.PolicyStateChanged | Se genera cuando se cambia un estado de cumplimiento de directiva. |
| Microsoft.PolicyInsights.PolicyStateDeleted | Se genera cuando se elimina un estado de cumplimiento de directiva. |

## <a name="event-schema"></a>Esquema del evento

Los eventos de Azure Policy contienen toda la información necesaria para responder a cualquier cambio que se produzca en los datos. Puede identificar un evento de Azure Policy cuando la propiedad `eventType` comienza con "Microsoft.PolicyInsights".
Puede encontrar información adicional acerca del uso de las propiedades de los eventos de Event Grid en  
el [esquema de eventos de Event Grid](../../../event-grid/event-schema.md).

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| `id` | string | Identificador único para el evento |
| `topic` | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| `subject` | string | Id. completo del recurso para el que se cambia el estado de cumplimiento, incluido el nombre del recurso y el tipo de recurso. Usa el formato `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>/providers/<ProviderNamespace>/<ResourceType>/<ResourceName>` |
| `data` | object | Datos de evento de Azure Policy. |
| `data.timestamp` | string | La hora (en UTC) en la que Azure Policy examinó el recurso. En el caso de los eventos de ordenación, use esta propiedad en lugar de las propiedades de nivel superior `eventTime` o `time`. |
| `data.policyAssignmentId` | string | El id. de recurso de la asignación de directiva. |
| `data.policyDefinitionId` | string | El id. de recurso de la definición de directiva. |
| `data.policyDefinitionReferenceId` | string | Id. de referencia de la definición de directiva dentro de la definición de la iniciativa, si la asignación de la directiva es para una iniciativa. Puede estar vacío. |
| `data.complianceState` | string | El estado de cumplimiento del recurso con respecto a la asignación de directiva. |
| `data.subscriptionId` | string | Identificador de suscripción del recurso. |
| `data.complianceReasonCode` | string | Código de motivo de cumplimiento. Puede estar vacío. |
| `eventType` | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| `eventTime` | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| `dataVersion` | string | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| `metadataVersion` | string | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |

A continuación se muestra un ejemplo de un evento de cambio de estado de directiva:

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

Para obtener más información, consulte el [esquema de eventos de Azure Policy](../../../event-grid/event-schema-policy.md).

## <a name="practices-for-consuming-events"></a>Prácticas para consumir eventos

Las aplicaciones que administran los eventos de Azure Policy deben seguir estos procedimientos recomendados:

> [!div class="checklist"]
> - Se pueden configurar varias suscripciones para enrutar los eventos al mismo controlador de eventos, por lo que no hay que asumir que los eventos procedan de un origen determinado. En su lugar, consulte el tema del mensaje para asegurarse de que la asignación de directiva, la definición de directiva y el recurso para el cual es el evento de cambio de estado.
> - Compruebe `eventType` y no asuma que todos los eventos que recibe son del tipo que espera.
> - Use `data.timestamp` para determinar el orden de los eventos en Azure Policy, en lugar de las propiedades `eventTime` o `time` de nivel superior.
> - Use el campo del asunto para obtener acceso al recurso que tenía un cambio de estado de directiva.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Event Grid y pruebe los eventos de cambio de estado de Azure Policy:

- [Enrutamiento de eventos de cambio de estado de directivas a Event Grid con la CLI de Azure](../tutorials/route-state-change-events.md)
- [Detalles del esquema de Azure Policy para Event Grid](../../../event-grid/event-schema-policy.md)
- [Una introducción a Azure Event Grid](../../../event-grid/overview.md)