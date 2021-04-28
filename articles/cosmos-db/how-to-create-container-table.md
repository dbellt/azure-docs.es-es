---
title: Creación de un contenedor en Table API de Azure Cosmos DB
description: Aprenda a crear un contenedor en Table API de Azure Cosmos DB mediante Azure Portal, .NET, Java, Python, Node.js y otros SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-csharp
ms.openlocfilehash: f4aa180eb6068aadf9ff33fa09d1797065993558
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108074212"
---
# <a name="create-a-container-in-azure-cosmos-db-table-api"></a>Creación de un contenedor en Table API de Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

En este artículo se explican las distintas formas de crear un contenedor en Table API de Azure Cosmos DB. Muestra cómo crear un contenedor mediante Azure Portal, CLI de Azure, PowerShell o los SDK admitidos. En este artículo se muestra cómo crear un contenedor, especificar la clave de partición y aprovisionar el rendimiento.

En este artículo se explican las distintas formas de crear un contenedor en Table API de Azure Cosmos DB. Si usa una API diferente, consulte los artículos [API para MongoDB](how-to-create-container-mongodb.md), [Cassandra API](how-to-create-container-cassandra.md), [Gremlin API](how-to-create-container-gremlin.md) y [API de SQL](how-to-create-container.md) para crear el contenedor.

> [!NOTE]
> Cuando cree contenedores, asegúrese de no utilizar el mismo nombre en dos de ellos con distintas mayúsculas y minúsculas. Algunos componentes de la plataforma de Azure no distinguen mayúsculas de minúsculas y esto puede producir confusión o problemas con los datos telemetría y las acciones que se realicen en los contenedores con estos nombres.

## <a name="create-using-azure-portal"></a><a id="portal-table"></a>Creación mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. [Cree una cuenta de Azure Cosmos](create-table-dotnet.md#create-a-database-account) o seleccione una ya existente.

1. Abra el panel **Data Explorer** y seleccione **Nueva tabla**. Después, proporcione los detalles siguientes:

   * Escriba un identificador de tabla.
   * Escriba el rendimiento que se va a aprovisionar (por ejemplo, 1000 RU).
   * Seleccione **Aceptar**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-table.png" alt-text="Captura de pantalla del cuadro de diálogo Agregar tabla de Table API":::

> [!Note]
> Con Table API, la clave de partición se especifica cada vez que se agrega una nueva fila.

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Creación mediante la CLI de Azure

[Cree una tabla de Table API con la CLI de Azure](./scripts/cli/table/create.md). Para obtener una lista de todos los ejemplos de la CLI de Azure en todas las API de Azure Cosmos DB, vea [Ejemplos de la CLI de Azure para Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Creación mediante PowerShell

[Cree una tabla de Table API con PowerShell](./scripts/powershell/table/create.md). Para obtener una lista de todos los ejemplos de PowerShell en todas las API de Azure Cosmos DB, vea [Ejemplos de PowerShell](powershell-samples.md).

## <a name="next-steps"></a>Pasos siguientes

* [Creación de particiones en Azure Cosmos DB](partitioning-overview.md)
* [Unidades de solicitud en Azure Cosmos DB](request-units.md)
* [Aprovisionamiento del rendimiento en contenedores y bases de datos](set-throughput.md)
* [Uso de la cuenta de Azure Cosmos](./account-databases-containers-items.md)