---
title: Obtención de la clave de acceso para un recurso de Event Grid
description: En este artículo se describe cómo obtener una clave de acceso para un tema o dominio de Event Grid.
ms.topic: how-to
ms.date: 07/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6052c93dc91403fccaed62719b163c24486557c5
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702524"
---
# <a name="get-access-keys-for-event-grid-resources-topics-or-domains"></a>Obtención de claves de acceso para recursos de Event Grid (temas o dominios)
Las claves de acceso se utilizan para autenticar los eventos de publicación de una aplicación en recursos de Azure Event Grid (temas y dominios). Se recomienda regenerar las claves periódicamente y almacenarlas de forma segura. Como se le proporcionan dos claves de acceso, puede mantener las conexiones con una mientras se regenera la otra.

En este artículo se describe cómo obtener las claves de acceso de un recurso de Event Grid (tema o dominio) mediante Azure Portal, PowerShell o la CLI. 

## <a name="azure-portal"></a>Azure portal
En Azure Portal, cambie a la pestaña **Claves de acceso** de la página **Tema de Event Grid** o **Dominio de Event Grid** para su tema o dominio.  

:::image type="content" source="./media/get-access-keys/azure-portal.png" alt-text="Página de claves de acceso":::

## <a name="azure-powershell"></a>Azure PowerShell
Use el comando [Get-AzEventGridTopicKey](/powershell/module/az.eventgrid/get-azeventgridtopickey) para obtener claves de acceso para los temas. 

```azurepowershell-interactive
Get-AzEventGridTopicKey -ResourceGroup <RESOURCE GROUP NAME> -Name <TOPIC NAME>
```

Use el comando [Get-AzEventGridDomainKey](/powershell/module/az.eventgrid/get-azeventgriddomainkey) para obtener claves de acceso para los dominios. 

```azurepowershell-interactive
Get-AzEventGridDomainKey -ResourceGroup <RESOURCE GROUP NAME> -Name <DOMAIN NAME>
```

## <a name="azure-cli"></a>Azure CLI
Use la [lista de claves del tema az eventgrid](/cli/azure/eventgrid/topic/key#az_eventgrid_topic_key_list) para obtener las claves de acceso para los temas. 

```azurecli-interactive
az eventgrid topic key list --resource-group <RESOURCE GROUP NAME> --name <TOPIC NAME>
```

Use la [lista de claves del dominio az eventgrid](/cli/azure/eventgrid/domain/key#az_eventgrid_domain_key_list) para obtener las claves de acceso para los dominios. 

```azurecli-interactive
az eventgrid domain key list --resource-group <RESOURCE GROUP NAME> --name <DOMAIN NAME>
```

## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo siguiente: [Autenticación de clientes de publicación](security-authenticate-publishing-clients.md). 
