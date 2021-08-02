---
title: Evitación de errores de limitación de velocidad para las operaciones de Azure Cosmos DB API para MongoDB
description: Aprenda a impedir que las operaciones de Azure Cosmos DB API para MongoDB alcancen los errores de limitación de velocidad con la característica SSR (reintento en el servidor).
author: gahl-levy
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 01/13/2021
ms.author: gahllevy
ms.openlocfilehash: 3852aec23a6ad894cd3f89c09c9013564fad4c4e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110062927"
---
# <a name="prevent-rate-limiting-errors-for-azure-cosmos-db-api-for-mongodb-operations"></a>Evitación de errores de limitación de velocidad para las operaciones de Azure Cosmos DB API para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Es posible que se produzcan errores en las operaciones de Azure Cosmos DB API para MongoDB de tipo de limitación de velocidad (16500/429) si superan el límite de rendimiento de una colección (RU). 

Puede habilitar la característica de reintentos en el servidor (SSR) y dejar que el servidor vuelva a intentar realizar estas operaciones automáticamente. Las solicitudes se reintentan después de un breve período de tiempo para todas las colecciones de la cuenta. Esta característica es una alternativa práctica para controlar los errores de limitación de velocidad en la aplicación cliente.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Vaya a la cuenta de Azure Cosmos DB API para MongoDB.

1. Vaya al panel **Características** debajo de la sección **Configuración**.

1. Seleccione **Reintento en el servidor**.

1. Haga clic en **Habilitar** para habilitar esta característica para todas las colecciones de su cuenta.

:::image type="content" source="./media/prevent-rate-limiting-errors/portal-features-server-side-retry.png" alt-text="Captura de pantalla de la característica de reintentos en el servidor de Azure Cosmos DB API para MongoDB":::

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

1. Compruebe si SSR ya está habilitado para su cuenta:

   ```azurecli-interactive
   az cosmosdb show --name accountname --resource-group resourcegroupname
   ```

1. **Habilite** SSR para todas las colecciones de la cuenta de bases de datos. Este cambio puede tardar un máximo de 15 minutos en surtir efecto.

   ```azurecli-interactive
   az cosmosdb update --name accountname --resource-group resourcegroupname --capabilities EnableMongo DisableRateLimitingResponses
   ```

1. El siguiente comando **deshabilitará** el reintento en el servidor en todas las colecciones de la cuenta de base de datos, ya que elimina `DisableRateLimitingResponses` de la lista de funcionalidades. Este cambio puede tardar un máximo de 15 minutos en surtir efecto.

   ```azurecli-interactive
   az cosmosdb update --name accountname --resource-group resourcegroupname --capabilities EnableMongo
   ```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-are-requests-retried"></a>¿Cómo se reintentan las solicitudes?

Las solicitudes se reintentan continuamente (una y otra vez) hasta que se alcanza un tiempo de espera de 60 segundos. Si se alcanza el tiempo de espera, el cliente recibirá una [excepción ExceededTimeLimit (50)](mongodb-troubleshoot.md).

### <a name="how-can-i-monitor-the-effects-of-a-server-side-retry"></a>¿Cómo se pueden supervisar los efectos de un reintento en el servidor?

Puede ver los errores de limitación de velocidad (429) que se reintentan en el servidor en el panel de métricas de Cosmos DB. Tenga en cuenta que estos errores no van al cliente cuando SSR está habilitado, ya que se controlan y se reintentan en el lado servidor.

Puede buscar entradas de registro que contengan *estimatedDelayFromRateLimitingInMilliseconds* en los [registros de recursos de Cosmos DB](cosmosdb-monitor-resource-logs.md).

### <a name="will-server-side-retry-affect-my-consistency-level"></a>¿Afectará el reintento en el servidor a mi nivel de consistencia?

El reintento en el servidor no afecta a la consistencia de ninguna solicitud. Las solicitudes se reintentan en el lado servidor si tienen una frecuencia limitada (con un error 429).

### <a name="does-server-side-retry-affect-any-type-of-error-that-my-client-might-receive"></a>¿Afecta el reintento en el servidor a cualquier tipo de error que pueda recibir mi cliente?

No, solo afecta a los errores de limitación de velocidad (429) al volver a intentar en el servidor. Esta característica evita que tenga que controlar los errores de limitación de velocidad en la aplicación cliente. Todos los [otros errores](mongodb-troubleshoot.md) irán al cliente.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la solución de errores comunes, consulte este artículo:

* [Solución de errores habituales de la API de Azure Cosmos DB para MongoDB](mongodb-troubleshoot.md)
