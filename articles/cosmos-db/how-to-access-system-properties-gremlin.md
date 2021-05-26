---
title: Acceso a las propiedades de documentos del sistema mediante Azure Cosmos DB Graph
description: Obtenga información sobre cómo leer y escribir propiedades de documentos del sistema de Cosmos DB mediante la API de Gremlin.
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 09/10/2019
author: SnehaGunda
ms.author: sngun
ms.openlocfilehash: ad5d24acb96170bdccd3d3b06d21b6f2ee86eb4b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110073655"
---
# <a name="system-document-properties"></a>Propiedades de documentos del sistema
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Azure Cosmos DB tiene [propiedades del sistema](/rest/api/cosmos-db/databases) como ```_ts```, ```_self```, ```_attachments```, ```_rid``` y ```_etag``` en todos los documentos. Además, el motor de Gremlin agrega las propiedades ```inVPartition``` y ```outVPartition``` en los bordes. De forma predeterminada, estas propiedades están disponibles para el recorrido. Sin embargo, es posible incluir propiedades específicas o todas ellas en el recorrido de Gremlin.

```
g.withStrategies(ProjectionStrategy.build().IncludeSystemProperties('_ts').create())
```

## <a name="e-tag"></a>E-Tag

Esta propiedad se usa para el control de simultaneidad optimista. Si la aplicación necesita interrumpir la operación en unos recorridos distintos, puede usar la propiedad e-Tag para evitar la pérdida de datos en las operaciones de escritura simultáneas.

```
g.withStrategies(ProjectionStrategy.build().IncludeSystemProperties('_etag').create()).V('1').has('_etag', '"00000100-0000-0800-0000-5d03edac0000"').property('test', '1')
```

## <a name="time-to-live-ttl"></a>Período de vida (TTL)

Si la colección tiene habilitada la caducidad del documento y los documentos tienen la propiedad ```ttl``` establecida, esta propiedad estará disponible en el recorrido de Gremlin como una propiedad normal del vértice o borde. ```ProjectionStrategy``` no es necesario para habilitar la exposición de la propiedad de período de vida.

Los vértices creados con el recorrido siguiente se eliminarán automáticamente en **123 segundos**.

```
g.addV('vertex-one').property('ttl', 123)
```

## <a name="next-steps"></a>Pasos siguientes
* [Simultaneidad optimista de Cosmos DB](faq.yml#how-does-the-sql-api-provide-concurrency-)
* [Período de vida (TTL)](time-to-live.md) en Azure Cosmos DB
