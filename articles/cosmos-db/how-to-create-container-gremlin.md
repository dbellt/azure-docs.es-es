---
title: Creación de un contenedor en Gremlin API de Azure Cosmos DB
description: Obtenga información sobre cómo crear un contenedor en Gremlin API de Azure Cosmos DB mediante Azure Portal, .NET y otros SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-csharp
ms.openlocfilehash: b6ae973e9008b6753113e6e71361658ab825e4b2
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108074257"
---
# <a name="create-a-container-in-azure-cosmos-db-gremlin-api"></a>Creación de un contenedor en Gremlin API de Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

En este artículo se explican las distintas formas de crear un contenedor en Gremlin API de Azure Cosmos DB. Muestra cómo crear un contenedor mediante Azure Portal, CLI de Azure, PowerShell o los SDK admitidos. En este artículo se muestra cómo crear un contenedor, especificar la clave de partición y aprovisionar el rendimiento.

En este artículo se explican las distintas formas de crear un contenedor en Gremlin API de Azure Cosmos DB. Si usa una API diferente, consulte los artículos [API para MongoDB](how-to-create-container-mongodb.md), [Cassandra API](how-to-create-container-cassandra.md), [Table API](how-to-create-container-table.md) y [API de SQL](how-to-create-container.md) para crear el contenedor.

> [!NOTE]
> Cuando cree contenedores, asegúrese de no utilizar el mismo nombre en dos de ellos con distintas mayúsculas y minúsculas. Algunos componentes de la plataforma de Azure no distinguen mayúsculas de minúsculas y esto puede producir confusión o problemas con los datos telemetría y las acciones que se realicen en los contenedores con estos nombres.

## <a name="create-using-azure-portal"></a><a id="portal-gremlin"></a>Creación mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-graph-dotnet.md#create-a-database-account) o seleccione una ya existente.

1. Abra el panel de **Data Explorer** y seleccione **New Graph** (Nuevo grafo). Después, proporcione los detalles siguientes:

   * Indique si va a crear una nueva base de datos o a usar una existente.
   * Escriba un identificador de grafo.
   * Seleccione la capacidad de almacenamiento **Sin límite**.
   * Escriba una clave de partición para los vértices.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-gremlin.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar grafo de la API Gremlin":::

## <a name="create-using-net-sdk"></a><a id="dotnet-sql-graph"></a>Creación mediante el SDK de .NET

Si se produce una excepción de tiempo de espera al crear una colección, realice una operación de lectura para validar si la colección se ha creado correctamente. La operación de lectura emite una excepción hasta que la operación de creación de la colección se realiza correctamente. Para la lista de códigos de estado admitidos por la operación de creación, consulte el artículo [Códigos de estado HTTP para Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

```csharp
// Create a container with a partition key and provision 1000 RU/s throughput.
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "myContainerName";
myCollection.PartitionKey.Paths.Add("/myPartitionKey");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    myCollection,
    new RequestOptions { OfferThroughput = 1000 });
```

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Creación mediante la CLI de Azure

[Cree un grafo de Gremlin con la CLI de Azure](./scripts/cli/gremlin/create.md). Para obtener una lista de todos los ejemplos de la CLI de Azure en todas las API de Azure Cosmos DB, vea [Ejemplos de la CLI de Azure para Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Creación mediante PowerShell

[Cree un grafo de Gremlin con PowerShell](./scripts/powershell/gremlin/create.md). Para obtener una lista de todos los ejemplos de PowerShell en todas las API de Azure Cosmos DB, vea [Ejemplos de PowerShell](powershell-samples.md).

## <a name="next-steps"></a>Pasos siguientes

* [Creación de particiones en Azure Cosmos DB](partitioning-overview.md)
* [Unidades de solicitud en Azure Cosmos DB](request-units.md)
* [Aprovisionamiento del rendimiento en contenedores y bases de datos](set-throughput.md)
* [Uso de la cuenta de Azure Cosmos](./account-databases-containers-items.md)