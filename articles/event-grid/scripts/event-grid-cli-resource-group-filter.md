---
title: 'CLI de Azure: suscripción a grupos de recursos y filtros por recurso'
description: En este artículo se proporciona un script de la CLI de Azure de ejemplo que muestra cómo suscribirse a los eventos de Azure Event Grid de un recurso y un filtro de un recurso.
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/08/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2662795e5ebc43e37eab7cab3dfc03582f072f8b
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107871440"
---
# <a name="subscribe-to-events-for-a-resource-group-and-filter-for-a-resource-with-azure-cli"></a>Suscripción a eventos de un grupo de recursos y filtrado de los eventos de un recurso con la CLI de Azure

Este script crea una suscripción de Event Grid a los eventos de un grupo de recursos. Utiliza un filtro para obtener solo los eventos de un recurso especificado en el grupo de recursos.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

El script de ejemplo de la versión preliminar requiere la extensión de Event Grid. Para instalarla, ejecute `az extension add --name eventgrid`.

## <a name="sample-script---stable"></a>Script de ejemplo: estable

[!code-azurecli[main](../../../cli_scripts/event-grid/filter-events/filter-events.sh "Subscribe to Azure subscription")]

## <a name="sample-script---preview-extension"></a>Script de ejemplo: extensión de la versión preliminar

[!code-azurecli[main](../../../cli_scripts/event-grid/filter-events-preview/filter-events-preview.sh "Subscribe to Azure subscription")]

## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para crear la suscripción de eventos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create) | Cree una suscripción de Event Grid. |
| [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create): versión de la extensión | Cree una suscripción de Event Grid. |

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la consulta de suscripciones, consulte [Consulta de suscripciones de Event Grid](../query-event-subscriptions.md).
* Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
