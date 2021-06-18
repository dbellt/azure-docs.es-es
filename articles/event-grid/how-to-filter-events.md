---
title: Filtrado de eventos para Azure Event Grid
description: En este artículo se muestra cómo filtrar eventos (por tipo de evento, por asunto, por operadores y datos, etc.) al crear una suscripción de Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8d3fef1135583fde3c68f20e841fcfc0c0a7334a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110698469"
---
# <a name="filter-events-for-event-grid"></a>Filtrado de eventos para Event Grid

En este artículo se muestra cómo filtrar eventos al crear una suscripción de Event Grid. Para obtener información sobre las opciones de filtrado de eventos, consulte [Understand event filtering for Event Grid subscriptions](event-filtering.md) (Descripción del filtrado de events para suscripciones de Event Grid).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="filter-by-event-type"></a>Filtrar por tipo de evento

Al crear una suscripción de Event Grid, puede especificar qué [tipos de evento](event-schema.md) se envían al punto de conexión. En los ejemplos de esta sección se crean suscripciones de eventos para un grupo de recursos, pero se limitan los eventos que se envían a `Microsoft.Resources.ResourceWriteFailure` y `Microsoft.Resources.ResourceWriteSuccess`. Si necesita más flexibilidad al filtrar eventos por tipos de eventos, consulte Filtrado por operadores avanzados y campos de datos.

Para PowerShell, use el parámetro `-IncludedEventType` al crear la suscripción.

```powershell
$includedEventTypes = "Microsoft.Resources.ResourceWriteFailure", "Microsoft.Resources.ResourceWriteSuccess"

New-AzEventGridSubscription `
  -EventSubscriptionName demoSubToResourceGroup `
  -ResourceGroupName myResourceGroup `
  -Endpoint <endpoint-URL> `
  -IncludedEventType $includedEventTypes
```

Para la CLI de Azure, use el parámetro `--included-event-types`. En el ejemplo siguiente se usa la CLI de Azure en un shell de Bash:

```azurecli
includedEventTypes="Microsoft.Resources.ResourceWriteFailure Microsoft.Resources.ResourceWriteSuccess"

az eventgrid event-subscription create \
  --name demoSubToResourceGroup \
  --resource-group myResourceGroup \
  --endpoint <endpoint-URL> \
  --included-event-types $includedEventTypes
```

Para una plantilla de Resource Manager, use la propiedad `includedEventTypes`.

```json
"resources": [
  {
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "name": "[parameters('eventSubName')]",
    "apiVersion": "2018-09-15-preview",
    "properties": {
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "[parameters('endpoint')]"
        }
      },
      "filter": {
        "subjectBeginsWith": "",
        "subjectEndsWith": "",
        "isSubjectCaseSensitive": false,
        "includedEventTypes": [
          "Microsoft.Resources.ResourceWriteFailure",
          "Microsoft.Resources.ResourceWriteSuccess"
        ]
      }
    }
  }
]
```

## <a name="filter-by-subject"></a>Filtrar por asunto

Puede filtrar eventos por el asunto de los datos del evento. Puede especificar un valor que debe coincidir con el principio o el final del asunto. Si necesita más flexibilidad al filtrar eventos por asunto, consulte Filtrado por operadores avanzados y campos de datos.

En el siguiente ejemplo de PowerShell, se crea una suscripción de eventos que filtra por el principio del asunto. Se usa el parámetro `-SubjectBeginsWith` para limitar los eventos a aquellos de un recurso específico. Se pasa el identificador de recurso de un grupo de seguridad de red.

```powershell
$resourceId = (Get-AzResource -ResourceName demoSecurityGroup -ResourceGroupName myResourceGroup).ResourceId

New-AzEventGridSubscription `
  -Endpoint <endpoint-URL> `
  -EventSubscriptionName demoSubscriptionToResourceGroup `
  -ResourceGroupName myResourceGroup `
  -SubjectBeginsWith $resourceId
```

En el siguiente ejemplo de PowerShell se crea una suscripción para un almacenamiento de blobs. Se limitan los eventos a aquellos con un asunto que termina por `.jpg`.

```powershell
$storageId = (Get-AzStorageAccount -ResourceGroupName myResourceGroup -AccountName $storageName).Id

New-AzEventGridSubscription `
  -EventSubscriptionName demoSubToStorage `
  -Endpoint <endpoint-URL> `
  -ResourceId $storageId `
  -SubjectEndsWith ".jpg"
```

En el siguiente ejemplo de la CLI de Azure, se crea una suscripción de eventos que filtra por el principio del asunto. Se usa el parámetro `--subject-begins-with` para limitar los eventos a aquellos de un recurso específico. Se pasa el identificador de recurso de un grupo de seguridad de red.

```azurecli
resourceId=$(az resource show --name demoSecurityGroup --resource-group myResourceGroup --resource-type Microsoft.Network/networkSecurityGroups --query id --output tsv)

az eventgrid event-subscription create \
  --name demoSubscriptionToResourceGroup \
  --resource-group myResourceGroup \
  --endpoint <endpoint-URL> \
  --subject-begins-with $resourceId
```

En el siguiente ejemplo de la CLI de Azure se crea una suscripción para un almacenamiento de blobs. Se limitan los eventos a aquellos con un asunto que termina por `.jpg`.

```azurecli
storageid=$(az storage account show --name $storageName --resource-group myResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --resource-id $storageid \
  --name demoSubToStorage \
  --endpoint <endpoint-URL> \
  --subject-ends-with ".jpg"
```

En el siguiente ejemplo de plantilla de Resource Manager, se crea una suscripción de eventos que filtra por el principio del asunto. Se usa la propiedad `subjectBeginsWith` para limitar los eventos a aquellos de un recurso específico. Se pasa el identificador de recurso de un grupo de seguridad de red.

```json
"resources": [
  {
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "name": "[parameters('eventSubName')]",
    "apiVersion": "2018-09-15-preview",
    "properties": {
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "[parameters('endpoint')]"
        }
      },
      "filter": {
        "subjectBeginsWith": "[resourceId('Microsoft.Network/networkSecurityGroups','demoSecurityGroup')]",
        "subjectEndsWith": "",
        "isSubjectCaseSensitive": false,
        "includedEventTypes": [ "All" ]
      }
    }
  }
]
```

En el siguiente ejemplo de plantilla de Resource Manager se crea una suscripción para un almacenamiento de blobs. Se limitan los eventos a aquellos con un asunto que termina por `.jpg`.

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts/providers/eventSubscriptions",
    "name": "[concat(parameters('storageName'), '/Microsoft.EventGrid/', parameters('eventSubName'))]",
    "apiVersion": "2018-09-15-preview",
    "properties": {
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "[parameters('endpoint')]"
        }
      },
      "filter": {
        "subjectEndsWith": ".jpg",
        "subjectBeginsWith": "",
        "isSubjectCaseSensitive": false,
        "includedEventTypes": [ "All" ]
      }
    }
  }
]
```

## <a name="filter-by-operators-and-data"></a>Filtrado por operadores y datos

Para obtener más flexibilidad en el filtrado, puede usar propiedades de datos y operadores para filtrar eventos.

### <a name="subscribe-with-advanced-filters"></a>Suscripción con filtros avanzados

Para obtener información acerca de los operadores y las claves que puede usar para el filtrado avanzado, consulte [Advanced filtering](event-filtering.md#advanced-filtering) (Filtrado avanzado).

En estos ejemplos se crea un tema personalizado. Se suscriben al tema personalizado y filtra por un valor en el objeto de datos. Los eventos que tienen la propiedad de color establecida en azul, rojo o verde se envían a la suscripción.

Para la CLI de Azure, utilice:

```azurecli
topicName=<your-topic-name>
endpointURL=<endpoint-URL>

az group create -n gridResourceGroup -l eastus2
az eventgrid topic create --name $topicName -l eastus2 -g gridResourceGroup

topicid=$(az eventgrid topic show --name $topicName -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  -n demoAdvancedSub \
  --advanced-filter data.color stringin blue red green \
  --endpoint $endpointURL \
  --expiration-date "<yyyy-mm-dd>"
```

Tenga en cuenta que se ha establecido una [fecha de expiración](concepts.md#event-subscription-expiration) para la suscripción.

Para PowerShell, use:

```powershell
$topicName = <your-topic-name>
$endpointURL = <endpoint-URL>

New-AzResourceGroup -Name gridResourceGroup -Location eastus2
New-AzEventGridTopic -ResourceGroupName gridResourceGroup -Location eastus2 -Name $topicName

$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name $topicName).Id

$expDate = '<mm/dd/yyyy hh:mm:ss>' | Get-Date
$AdvFilter1=@{operator="StringIn"; key="Data.color"; Values=@('blue', 'red', 'green')}

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint $endpointURL `
  -ExpirationDate $expDate `
  -AdvancedFilter @($AdvFilter1)
```

### <a name="test-filter"></a>Prueba del filtro

Para probar el filtro, envíe un evento con el campo de color establecido en verde. Dado que verde es uno de los valores del filtro, el evento se entrega en el punto de conexión.

Para la CLI de Azure, utilice:

```azurecli
topicEndpoint=$(az eventgrid topic show --name $topicName -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name $topicName -g gridResourceGroup --query "key1" --output tsv)

event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/cars", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "model": "SUV", "color": "green"},"dataVersion": "1.0"} ]'

curl -X POST -H "aeg-sas-key: $key" -d "$event" $topicEndpoint
```

Para PowerShell, use:

```powershell
$endpoint = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name $topicName).Endpoint
$keys = Get-AzEventGridTopicKey -ResourceGroupName gridResourceGroup -Name $topicName

$eventID = Get-Random 99999
$eventDate = Get-Date -Format s

$htbody = @{
    id= $eventID
    eventType="recordInserted"
    subject="myapp/vehicles/cars"
    eventTime= $eventDate
    data= @{
        model="SUV"
        color="green"
    }
    dataVersion="1.0"
}

$body = "["+(ConvertTo-Json $htbody)+"]"

Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
```

Para probar un escenario donde no se envía el evento, envíe un evento con el campo de color establecido en amarillo. El amarillo no es uno de los valores especificados en la suscripción, por lo que el evento no se entrega a su suscripción.

Para la CLI de Azure, utilice:

```azurecli
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/cars", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "model": "SUV", "color": "yellow"},"dataVersion": "1.0"} ]'

curl -X POST -H "aeg-sas-key: $key" -d "$event" $topicEndpoint
```
Para PowerShell, use:

```powershell
$htbody = @{
    id= $eventID
    eventType="recordInserted"
    subject="myapp/vehicles/cars"
    eventTime= $eventDate
    data= @{
        model="SUV"
        color="yellow"
    }
    dataVersion="1.0"
}

$body = "["+(ConvertTo-Json $htbody)+"]"

Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
```

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la supervisión de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para más información sobre la clave de autenticación, vea [Seguridad y autenticación de Event Grid](security-authentication.md).
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
