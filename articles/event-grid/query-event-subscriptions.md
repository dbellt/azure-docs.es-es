---
title: Consulta de suscripciones de Azure Event Grid
description: En este artículo se describe cómo enumerar las suscripciones de Event Grid en una suscripción a Azure. Proporciona distintos parámetros en función del tipo de suscripción.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e8818a1b055d19153563700efd2fa16141a4fe05
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110695934"
---
# <a name="query-event-grid-subscriptions"></a>Consulta de suscripciones de Event Grid 

En este artículo se describe cómo enumerar las suscripciones de Event Grid en una suscripción a Azure. Cuando se consultan las suscripciones de Event Grid existentes, es importante conocer los distintos tipos de suscripciones, ya que se usan parámetros diferentes en función del tipo de suscripción que se desea obtener.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-groups-and-azure-subscriptions"></a>Grupos de recursos y suscripciones de Azure

Ni las suscripciones ni los grupos de recursos son recursos de Azure. Por consiguiente, las suscripciones de Event Grid a grupos de recursos o a suscripciones de Azure no tienen las mismas propiedades que las suscripciones de Event Grid a los recursos de Azure. Las suscripciones de Event Grid a grupos de recursos o a suscripciones de Azure se consideran globales.

Para obtener suscripciones de Event Grid para una suscripción de Azure y sus grupos de recursos, no es preciso especificar ningún parámetro. Asegúrese de que ha seleccionado la suscripción de Azure en la que desea realizar una consulta. Los ejemplos siguientes no obtienen suscripciones de Event Grid para temas personalizados o recursos de Azure.

Para la CLI de Azure, utilice:

```azurecli-interactive
az account set -s "My Azure Subscription"
az eventgrid event-subscription list
```

Para PowerShell, use:

```azurepowershell-interactive
Set-AzContext -Subscription "My Azure Subscription"
Get-AzEventGridSubscription
```

Para obtener las suscripciones de Event Grid para una suscripción de Azure, especifique el tipo de tema **Microsoft.Resources.Subscriptions**.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.Subscriptions" --location global
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Resources.Subscriptions"
```

Para obtener las suscripciones de Event Grid para todos los tipos de recursos de una suscripción de Azure, especifique el tipo de tema **Microsoft.Resources.Subscriptions**.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.ResourceGroups" --location global
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Resources.ResourceGroups"
```

Para obtener las suscripciones de Event Grid para un grupo de recursos especificado, indique el nombre del grupo de recursos como parámetro.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --resource-group myResourceGroup --location global
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -ResourceGroupName myResourceGroup
```

## <a name="custom-topics-and-azure-resources"></a>Temas personalizados y recursos de Azure

Los temas personalizados de Event Grid son recursos de Azure. Por consiguiente, en las suscripciones de Event Grid tanto los temas personalizados como otros recursos, como la cuenta de Blob Storage, se consultan de la misma forma. Para obtener las suscripciones de Event Grid para temas personalizados, es preciso usar parámetros que identifiquen el recurso o la ubicación del recurso. No es posible realizar consultas generales a las suscripciones de Event Grid de los recursos de su suscripción de Azure.

Para obtener las suscripciones de Event Grid para temas personalizados y otros recursos de una ubicación, especifique el nombre de dicha ubicación.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --location westus2
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -Location westus2
```

Para obtener las suscripciones a temas personalizados para una ubicación, especifique la ubicación y el tipo de tema **Microsoft.EventGrid.Topics**.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.EventGrid.Topics" --location "westus2"
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.EventGrid.Topics" -Location westus2
```

Para obtener las suscripciones a las cuentas de almacenamiento para una ubicación, especifique la ubicación y el tipo de tema **Microsoft.EventGrid.Topics**.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --topic-type "Microsoft.Storage.StorageAccounts" --location westus2
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Storage.StorageAccounts" -Location westus2
```

Para obtener las suscripciones de Event Grid para un tema personalizado, especifique tanto el nombre del tema como el de su grupo de recursos.

Para la CLI de Azure, utilice:

```azurecli-interactive
az eventgrid event-subscription list --topic-name myCustomTopic --resource-group myResourceGroup
```

Para PowerShell, use:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicName myCustomTopic -ResourceGroupName myResourceGroup
```

Para obtener las suscripciones de Event Grid para un recurso concreto, especifique el identificador del recurso.

Para la CLI de Azure, utilice:

```azurecli-interactive
resourceid=$(az resource show -n mystorage -g myResourceGroup --resource-type "Microsoft.Storage/storageaccounts" --query id --output tsv)
az eventgrid event-subscription list --resource-id $resourceid
```

Para PowerShell, use:

```azurepowershell-interactive
$resourceid = (Get-AzResource -Name mystorage -ResourceGroupName myResourceGroup).ResourceId
Get-AzEventGridSubscription -ResourceId $resourceid
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información acerca de los reintentos y las entregas de eventos, consulte [Entrega y reintento de entrega de mensajes de Event Grid](delivery-and-retry.md).
* Para obtener una introducción a Event Grid, vea [Acerca de Event Grid](overview.md).
* Para comenzar a usar rápidamente Event Grid, vea [Creación y enrutamiento de eventos personalizados con Azure Event Grid](custom-event-quickstart.md).
