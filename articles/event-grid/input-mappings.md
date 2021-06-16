---
title: Asignación de campos personalizados a esquemas de Azure Event Grid
description: En este artículo se explica cómo convertir el esquema personalizado en el esquema de Azure Event Grid cuando los datos de un evento no coinciden con el esquema de Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6676134f878ea77168ee56f065ec1d4cd5b08505
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704933"
---
# <a name="map-custom-fields-to-event-grid-schema"></a>Asignación de campos personalizados a esquemas de Event Grid

Si los datos del evento no coinciden con el [esquema de Event Grid](event-schema.md) esperado, podrá seguir usando Event Grid para enrutar el evento a los suscriptores. En este artículo se describe cómo asignar el esquema al de Event Grid.

## <a name="original-event-schema"></a>Esquema del evento original

Supongamos que tiene una aplicación que envía eventos con el siguiente formato:

```json
[
  {
    "myEventTypeField":"Created",
    "resource":"Users/example/Messages/1000",
    "resourceData":{"someDataField1":"SomeDataFieldValue"}
  }
]
```

Aunque dicho formato no coincide con el esquema requerido, Event Grid permite asignar los campos al esquema. También pueden recibir los valores en el esquema original.

## <a name="create-custom-topic-with-mapped-fields"></a>Creación de temas personalizados con campos asignados

Al crear un tema personalizado, especifique cómo asignar campos del evento original al esquema de Event Grid. Hay tres valores que se pueden usar para personalizar la asignación:

* El valor del **esquema de entrada** especifica el tipo de esquema. Las opciones disponibles son el esquema de CloudEvents, el esquema de eventos personalizados o el esquema de Event Grid. El valor predeterminado es el esquema de Event Grid. Al crear la asignación personalizada entre el esquema y el esquema de Event Grid, use el esquema de eventos personalizados. Si los eventos están en el formato CloudEvents, use el esquema CloudEvents.

* La propiedad **mapping default values** (asignación de valores predeterminados) especifica los valores predeterminados para los campos en el esquema de Event Grid. Puede establecer valores predeterminados para `subject`, `eventtype` y `dataversion`. Normalmente, se usa este parámetro si el esquema personalizado no incluye un campo que se corresponde con alguno de esos tres campos. Por ejemplo, puede especificar que la versión de datos siempre se establezca en **1.0**.

* El valor de **mapping fields** (asignación de campos) asigna los campos del esquema al esquema de Event Grid. Especifique los valores en pares clave-valor separados por espacios. Para el nombre de clave, usa el nombre del campo de Event Grid. Para el valor, utilice el nombre del campo. Puede usar nombres de clave para `id`, `topic`, `eventtime`, `subject`, `eventtype` y `dataversion`.

Para crear un tema personalizado con la CLI de Azure, use:

```azurecli-interactive
az eventgrid topic create \
  -n demotopic \
  -l eastus2 \
  -g myResourceGroup \
  --input-schema customeventschema \
  --input-mapping-fields eventType=myEventTypeField \
  --input-mapping-default-values subject=DefaultSubject dataVersion=1.0
```

Para PowerShell, use:

```azurepowershell-interactive
New-AzEventGridTopic `
  -ResourceGroupName myResourceGroup `
  -Name demotopic `
  -Location eastus2 `
  -InputSchema CustomEventSchema `
  -InputMappingField @{eventType="myEventTypeField"} `
  -InputMappingDefaultValue @{subject="DefaultSubject"; dataVersion="1.0" }
```

## <a name="subscribe-to-event-grid-topic"></a>Suscripción a temas de Event Grid

Al suscribirse al tema personalizado, especifique el esquema que le gustaría utilizar para recibir los eventos. Puede especificar el esquema de CloudEvents, el esquema de eventos personalizados o el esquema de Event Grid. El valor predeterminado es el esquema de Event Grid.

En el ejemplo siguiente se realiza la suscripción a un tema de Event Grid y se usa el esquema de Event Grid. Para la CLI de Azure, utilice:

```azurecli-interactive
topicid=$(az eventgrid topic show --name demoTopic -g myResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name eventsub1 \
  --event-delivery-schema eventgridschema \
  --endpoint <endpoint_URL>
```

En el ejemplo siguiente se usa el esquema de entrada del evento:

```azurecli-interactive
az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name eventsub2 \
  --event-delivery-schema custominputschema \
  --endpoint <endpoint_URL>
```

En el ejemplo siguiente se realiza la suscripción a un tema de Event Grid y se usa el esquema de Event Grid. Para PowerShell, use:

```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName myResourceGroup -Name demoTopic).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName eventsub1 `
  -EndpointType webhook `
  -Endpoint <endpoint-url> `
  -DeliverySchema EventGridSchema
```

En el ejemplo siguiente se usa el esquema de entrada del evento:

```azurepowershell-interactive
New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName eventsub2 `
  -EndpointType webhook `
  -Endpoint <endpoint-url> `
  -DeliverySchema CustomInputSchema
```

## <a name="publish-event-to-topic"></a>Publicación de eventos en temas

Ahora ya puede enviar un evento al tema personalizado y ver el resultado de la asignación. A continuación se indica el script para publicar un evento en el [esquema de ejemplo](#original-event-schema):

Para la CLI de Azure, utilice:

```azurecli-interactive
endpoint=$(az eventgrid topic show --name demotopic -g myResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name demotopic -g myResourceGroup --query "key1" --output tsv)

event='[ { "myEventTypeField":"Created", "resource":"Users/example/Messages/1000", "resourceData":{"someDataField1":"SomeDataFieldValue"} } ]'

curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
```

Para PowerShell, use:

```azurepowershell-interactive
$endpoint = (Get-AzEventGridTopic -ResourceGroupName myResourceGroup -Name demotopic).Endpoint
$keys = Get-AzEventGridTopicKey -ResourceGroupName myResourceGroup -Name demotopic

$htbody = @{
    myEventTypeField="Created"
    resource="Users/example/Messages/1000"
    resourceData= @{
        someDataField1="SomeDataFieldValue"
    }
}

$body = "["+(ConvertTo-Json $htbody)+"]"
Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
```

Ahora, examinemos el punto de conexión de WebHook. Las dos suscripciones entregaron eventos en diferentes esquemas.

La primera suscripción usó el esquema de Event Grid. El formato del evento entregado es:

```json
{
  "id": "aa5b8e2a-1235-4032-be8f-5223395b9eae",
  "eventTime": "2018-11-07T23:59:14.7997564Z",
  "eventType": "Created",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.EventGrid/topics/demotopic",
  "subject": "DefaultSubject",
  "data": {
    "myEventTypeField": "Created",
    "resource": "Users/example/Messages/1000",
    "resourceData": {
      "someDataField1": "SomeDataFieldValue"
    }
  }
}
```

Estos campos contienen las asignaciones del tema personalizado. **myEventTypeField** se asigna a **EventType**. Se usan los valores predeterminados para **DataVersion** y **Subject**. El objeto **Data** contiene los campos del esquema del evento original.

La segunda suscripción usó el esquema del evento de entrada. El formato del evento entregado es:

```json
{
  "myEventTypeField": "Created",
  "resource": "Users/example/Messages/1000",
  "resourceData": {
    "someDataField1": "SomeDataFieldValue"
  }
}
```

Tenga en cuenta que se entregaron los campos originales.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información acerca de los reintentos y las entregas de eventos, consulte [Entrega y reintento de entrega de mensajes de Event Grid](delivery-and-retry.md).
* Para obtener una introducción a Event Grid, vea [Acerca de Event Grid](overview.md).
* Para comenzar a usar rápidamente Event Grid, vea [Creación y enrutamiento de eventos personalizados con Azure Event Grid](custom-event-quickstart.md).
