---
title: Solución de problemas de excepciones de recurso de Azure Cosmos DB no encontrado
description: Descubra cómo diagnosticar y corregir excepciones de recurso no encontrado.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 05/26/2021
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: b9a78c1adeb5aa833c7c60be1dc4e553ae7d5393
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110538851"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-not-found-exceptions"></a>Diagnóstico y solución de problemas de excepciones recurso de Azure Cosmos DB no encontrado
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

El código de estado HTTP 404 representa que el recurso ya no existe.

## <a name="expected-behavior"></a>Comportamiento esperado
Hay muchos escenarios válidos en los que una aplicación espera un código 404 y administra correctamente el escenario.

## <a name="a-not-found-exception-was-returned-for-an-item-that-should-exist-or-does-exist"></a>Se ha devuelto la excepción de recurso no encontrado para un elemento que existe o debería existir
A continuación se ofrecen las posibles razones para que se devuelva un código de estado 404 cuando el elemento existe o debería existir.

### <a name="the-read-session-is-not-available-for-the-input-session-token"></a>La sesión de lectura no está disponible para el token de sesión de entrada.

#### <a name="solution"></a>Solución:
1. Actualice el SDK actual a la versión más reciente disponible. Las causas más comunes de este error concreto se han corregido en las versiones más recientes del SDK.

### <a name="race-condition"></a>Condición de carrera
Hay varias instancias de cliente de SDK y la lectura se produjo antes de la escritura.

#### <a name="solution"></a>Solución:
1. La coherencia de la cuenta predeterminada para Azure Cosmos DB es la coherencia de la sesión. Cuando se crea o se actualiza un elemento, la respuesta devolverá un token de sesión que se puede pasar de una instancia de SDK a otra para garantizar que la solicitud de lectura lea desde una réplica que tiene ese cambio.
1. Cambie el [nivel de coherencia](./consistency-levels.md) a un [nivel más seguro](./consistency-levels.md).

### <a name="reading-throughput-for-a-container-or-database-resource"></a>Lectura del rendimiento de un contenedor o recurso de base de datos
Uso de PowerShell o la CLI de Azure y recepción de un mensaje de error *no encontrado*.

#### <a name="solution"></a>Solución:
El rendimiento se puede aprovisionar en el nivel de base de datos, en el nivel de contenedor o en ambos. Si recibe un error *no encontrado*, intente leer el rendimiento del recurso de base de datos principal o el recurso de contenedor secundario.

### <a name="invalid-partition-key-and-id-combination"></a>Combinación no válida de clave de partición e identificador
La combinación de la clave de partición y el identificador no es válida.

#### <a name="solution"></a>Solución:
Corrija la lógica de aplicación que provoca la combinación incorrecta. 

### <a name="invalid-character-in-an-item-id"></a>Carácter no válido en un identificador de elemento
Se ha insertado un elemento en Cosmos DB con un [carácter no válido](/dotnet/api/microsoft.azure.documents.resource.id#remarks) en el identificador de elemento.

#### <a name="solution"></a>Solución:
Cambie el identificador a un valor que no contenga el carácter especial. Si cambiar de identificador no es una alternativa, puede codificar el identificador en Base64 para omitir los caracteres especiales. Base64 aún puede producir un nombre con un carácter '/' no válido que es necesario reemplazar.

Los elementos ya insertados en el contenedor para el identificador se pueden reemplazar con valores RID en lugar de referencias basadas en nombres.
```c#
// Get a container reference that uses RID values.
ContainerProperties containerProperties = await this.Container.ReadContainerAsync();
string[] selfLinkSegments = containerProperties.SelfLink.Split('/');
string databaseRid = selfLinkSegments[1];
string containerRid = selfLinkSegments[3];
Container containerByRid = this.cosmosClient.GetContainer(databaseRid, containerRid);

// Invalid characters are listed here.
//https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id#remarks
FeedIterator<JObject> invalidItemsIterator = this.Container.GetItemQueryIterator<JObject>(
    @"select * from t where CONTAINS(t.id, ""/"") or CONTAINS(t.id, ""#"") or CONTAINS(t.id, ""?"") or CONTAINS(t.id, ""\\"") ");
while (invalidItemsIterator.HasMoreResults)
{
    foreach (JObject itemWithInvalidId in await invalidItemsIterator.ReadNextAsync())
    {
        // Choose a new ID that doesn't contain special characters.
        // If that isn't possible, then Base64 encode the ID to escape the special characters.
        byte[] plainTextBytes = Encoding.UTF8.GetBytes(itemWithInvalidId["id"].ToString());
        itemWithInvalidId["id"] = Convert.ToBase64String(plainTextBytes).Replace('/', '!');

        // Update the item with the new ID value by using the RID-based container reference.
        JObject item = await containerByRid.ReplaceItemAsync<JObject>(
            item: itemWithInvalidId,
            ID: itemWithInvalidId["_rid"].ToString(),
            partitionKey: new Cosmos.PartitionKey(itemWithInvalidId["status"].ToString()));

        // Validating the new ID can be read by using the original name-based container reference.
        await this.Container.ReadItemAsync<ToDoActivity>(
            item["id"].ToString(),
            new Cosmos.PartitionKey(item["status"].ToString())); ;
    }
}
```

### <a name="time-to-live-purge"></a>Purga de la propiedad Período de vida
El elemento tenía definida la propiedad [Período de vida (TTL)](./time-to-live.md). El elemento se purgó porque la propiedad TTL expiró.

#### <a name="solution"></a>Solución:
Cambie la propiedad TTL para evitar que el elemento se purgue.

### <a name="lazy-indexing"></a>Indexación diferida
La [indexación diferida](index-policy.md#indexing-mode) no se ha actualizado.

#### <a name="solution"></a>Solución:
Espere a que la indexación se actualice o cambie la directiva de indexación.

### <a name="parent-resource-deleted"></a>Recurso primario eliminado
Se eliminó la base de datos o el contenedor donde se encuentra el elemento.

#### <a name="solution"></a>Solución:
1. [Restaure](./configure-periodic-backup-restore.md#request-restore) el recurso primario o vuelva a crearlo.
1. Cree un nuevo recurso para reemplazar el recurso eliminado.

### <a name="7-containercollection-names-are-case-sensitive"></a>7. Los nombres de contenedor o colección distinguen en mayúsculas y minúsculas
En Cosmos DB, los nombres de contenedor o colección distinguen mayúsculas de minúsculas.

#### <a name="solution"></a>Solución:
Asegúrese de usar el nombre exacto mientras se conecta a Cosmos DB.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).
* [Diagnóstico y solución de problemas](troubleshoot-java-sdk-v4-sql.md) al utilizar el SDK de Azure Cosmos DB para Java v4.
* Obtenga información sobre las instrucciones de rendimiento del [SDK para Java v4](performance-tips-java-sdk-v4-sql.md).
