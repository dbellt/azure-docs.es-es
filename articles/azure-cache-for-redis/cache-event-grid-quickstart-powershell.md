---
title: 'Inicio rápido: Enrutamiento de eventos de Azure Cache for Redis al punto de conexión web mediante PowerShell'
description: Use Azure Event Grid para suscribirse a eventos de Azure Cache for Redis, enviar los eventos a un webhook y controlar los eventos en una aplicación web.
ms.date: 1/5/2021
author: curib
ms.author: cauribeg
ms.topic: quickstart
ms.service: cache
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fa1ae9a7a2200944bd5da0211be88ba4955e3d03
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833893"
---
# <a name="quickstart-route-azure-cache-for-redis-events-to-web-endpoint-with-powershell"></a>Inicio rápido: Enrutamiento de eventos de Azure Cache for Redis al punto de conexión web mediante PowerShell

Azure Event Grid es un servicio de eventos para la nube. En esta guía de inicio rápido, va a usar Azure PowerShell para suscribirse a eventos de Azure Cache for Redis, desencadenar un evento y ver los resultados. 

Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. Sin embargo, para simplificar la guía de inicio rápido, enviará eventos a una aplicación web que recopilará y mostrará los mensajes. Al completar los pasos descritos en esta guía de inicio rápido, verá que los datos de eventos se han enviado a la aplicación web.

## <a name="setup"></a>Configurar

Esta guía de inicio rápido requiere la ejecución de la versión más reciente de Azure PowerShell. Si necesita instalarlas o actualizarlas, vea [Install and configure Azure PowerShell](/powershell/azure/install-Az-ps) (Instalación y configuración de Azure PowerShell).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en la suscripción de Azure con el comando `Connect-AzAccount` y siga las instrucciones que aparecen en pantalla para autenticarse.

```powershell
Connect-AzAccount
```

Este ejemplo utiliza **westus2** y almacena la selección en una variable que se puede usar en todo momento.

```powershell
$location = "westus2"
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los temas de Event Grid se implementan como recursos individuales de Azure y se deben aprovisionar en un grupo de recursos de Azure. Un grupo de recursos de Azure es una colección lógica en la que se implementan y administran los recursos de Azure.

Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

En el ejemplo siguiente, se crea un grupo de recursos denominado **gridResourceGroup** en la ubicación **westus2**.  

```powershell
$resourceGroup = "gridResourceGroup"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-an-azure-cache-for-redis-instance"></a>Creación de una instancia de Azure Redis Cache 

```powershell
New-AzRedisCache
   -ResourceGroupName <String>
   -Name <String>
   -Location <String>
   [-Size <String>]
   [-Sku <String>]
   [-RedisConfiguration <Hashtable>]
   [-EnableNonSslPort <Boolean>]
   [-TenantSettings <Hashtable>]
   [-ShardCount <Int32>]
   [-MinimumTlsVersion <String>]
   [-SubnetId <String>]
   [-StaticIP <String>]
   [-Tag <Hashtable>]
   [-Zone <String[]>]
   [-DefaultProfile <IAzureContextContainer>]
   [-WhatIf]
   [-Confirm]
   [<CommonParameters>]
```
Para obtener más información sobre cómo crear una instancia de caché en PowerShell, consulte la documentación de [referencia de Azure PowerShell](/powershell/module/az.rediscache/new-azrediscache). 

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje

Antes de suscribirse al tema, vamos a crear el punto de conexión para el mensaje de evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. Para simplificar esta guía de inicio rápido, se implementa una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestra los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

Reemplace `<your-site-name>` por un nombre único para la aplicación web. El nombre de la aplicación web debe ser único, ya que es parte de la entrada DNS.

```powershell
$sitename="<your-site-name>"

New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" `
  -siteName $sitename `
  -hostingPlanName viewerhost
```

La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

Debería ver el sitio, que no muestra ningún mensaje actualmente.

:::image type="content" source="media/cache-event-grid-portal/blank-event-grid-viewer.png" alt-text="El sitio del visor de Event Grid vacío.":::

## <a name="subscribe-to-your-azure-cache-for-redis-event"></a>Suscripción al evento de Azure Cache for Redis

En este paso, se va a suscribir a un tema para indicar a Event Grid los eventos a los que desea hacer un seguimiento. En el ejemplo siguiente se suscribirá a la instancia de caché que ha creado y pasará la dirección URL de la aplicación web como el punto de conexión de la notificación de eventos. El punto de conexión de la aplicación web debe incluir el sufijo `/api/updates/`.

```powershell
$cacheId = (Get-AzRedisCache -ResourceGroupName $resourceGroup -Name $cacheName).Id
$endpoint="https://$sitename.azurewebsites.net/api/updates"

New-AzEventGridSubscription `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint $endpoint `
  -ResourceId $cacheId
```

Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Seleccione el icono del ojo para expandir los datos del evento. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. La aplicación web incluye código para validar la suscripción.

  :::image type="content" source="media/cache-event-grid-portal/subscription-event.png" alt-text="El visor de Azure Event Grid.":::

## <a name="trigger-an-event-from-azure-cache-for-redis"></a>Desencadenamiento de un evento desde Azure Cache for Redis

Ahora, vamos a desencadenar un evento para ver cómo Event Grid distribuye el mensaje al punto de conexión.

```powershell
Import-AzRedisCache
      [-ResourceGroupName <String>]
      -Name <String>
      -Files <String[]>
      [-Format <String>]
      [-Force]
      [-PassThru]
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
```
Para obtener más información sobre la importación en PowerShell, consulte la documentación de [referencia de Azure PowerShell](/powershell/module/az.rediscache/import-azrediscache). 

Ha desencadenado el evento y Event Grid ha enviado el mensaje al punto de conexión que configuró al realizar la suscripción. Vaya a la aplicación web para ver el evento que acaba de enviar.

```json
[{
"id": "e1ceb52d-575c-4ce4-8056-115dec723cff",
  "eventType": "Microsoft.Cache.ImportRDBCompleted",
  "topic": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
  "data": {
    "name": "ImportRDBCompleted",
    "timestamp": "2020-12-10T18:07:54.4937063+00:00",
    "status": "Succeeded"
  },
  "subject": "ImportRDBCompleted",
  "dataversion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-12-10T18:07:54.4937063+00:00"
}]

```

## <a name="clean-up-resources"></a>Limpieza de recursos
Si tiene previsto seguir trabajando con esta instancia y suscripción a eventos de Azure Cache for Redis, no limpie los recursos creados en esta guía de inicio rápido. Si no va a continuar, use el siguiente comando para eliminar los recursos que creó en esta guía de inicio rápido.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo crear suscripciones a temas y eventos, obtenga más información sobre los eventos de Azure Cache for Redis y lo que Event Grid puede ayudarle a hacer:

- [Reacción a eventos de Azure Cache for Redis](cache-event-grid.md)
- [Una introducción a Azure Event Grid](../event-grid/overview.md)
