---
title: Notas de la versión y recursos de la versión 4 del SDK de Java para la API SQL de Azure Cosmos DB
description: Obtenga toda la información sobre la versión 4 del SDK de Java de Azure Cosmos DB para el SDK y API SQL, incluidas la fechas de lanzamiento y de retirada, y los cambios realizados entre las versiones del SDK de Java asincrónico para SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 04/06/2021
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: d31f3d1c510ffe6c3f0a739a4e41313c8c6e7cf0
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107364828"
---
# <a name="azure-cosmos-db-java-sdk-v4-for-core-sql-api-release-notes-and-resources"></a>Versión 4 del SDK de Java para la API SQL de Azure Cosmos DB: notas de la versión y recursos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [.NET SDK v2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET, versión 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Conector Spark 3 OLTP](sql-api-sdk-java-spark-v3.md)
> * [Conector Spark 2 OLTP](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

La versión 4 del SDK de Java de Azure Cosmos DB para Core (SQL) combina una API asincrónica y una API de sincronización en un solo artefacto de Maven. El SDK v4 ofrece rendimiento mejorado, nuevas características de API y compatibilidad asincrónica basada en Project Reactor y la [biblioteca Netty](https://netty.io/). Los usuarios pueden esperar un rendimiento mejorado con la versión 4 del SDK de Java de Azure Cosmos DB en comparación con la [versión 2 del SDK de Java asincrónico de Azure Cosmos DB](sql-api-sdk-async-java.md) y la [versión 2 del SDK de Java sincrónico de Azure Cosmos DB](sql-api-sdk-java.md).

> [!IMPORTANT]  
> Estas son las notas de la versión solo para la versión 4 del SDK de Java para Azure Cosmos DB. Si en la actualidad usa una versión anterior a la v4, vea la guía [Migración a la versión 4 del SDK de Java de Azure Cosmos DB](migrate-java-v4-sdk.md) a fin de obtener ayuda para actualizar a v4.
>
> Estos son tres pasos para empezar a trabajar rápidamente.
> 1. Instale el [runtime de Java mínimo admitido, JDK 8](/java/azure/jdk/) para poder usar el SDK.
> 2. Consulte la [Guía de inicio rápido para la versión 4 del SDK de Java de Azure Cosmos DB](./create-sql-api-java.md), en la que se proporciona acceso al artefacto de Maven y se describen solicitudes básicas de Azure Cosmos DB.
> 3. Lea las [sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) y las guías de [solución de problemas](troubleshoot-java-sdk-v4-sql.md) de la versión 4 del SDK de Java de Azure Cosmos DB para optimizar el SDK de la aplicación.
>
> Los [talleres y laboratorios de Azure Cosmos DB](https://aka.ms/cosmosworkshop) son otro fantástico recurso para aprender a usar la versión 4 del SDK de Java de Azure Cosmos DB.
>

## <a name="helpful-content"></a>Contenido útil

| Contenido | Vínculo |
|---|---|
|**Descarga del SDK**| [Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) |
|**Documentación de la API** | [Documentación de referencia de API](/java/api/overview/azure/cosmosdb/client) |
|**Contribuya al SDK** | [Repositorio central del SDK de Azure para Java en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos) | 
|**Introducción** | [Inicio rápido: Creación de una aplicación Java para administrar los datos de SQL API de Azure Cosmos DB](./create-sql-api-java.md) <br> [Repositorio de GitHub con código de inicio rápido](https://github.com/Azure-Samples/azure-cosmos-java-getting-started) | 
|**Ejemplos de código básicos** | [Azure Cosmos DB: ejemplos de Java para la API de SQL](sql-api-java-sdk-samples.md) <br> [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples)|
|**Aplicación de consola con fuente de cambios**| [Fuente de cambios: ejemplo de SDK de Java v4](create-sql-api-java-changefeed.md) <br> [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example)| 
|**Ejemplo de aplicación web**| [Compilación de una aplicación web con el SDK de Java v4](sql-api-java-application.md) <br> [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-todo-app)|
| **Consejos de rendimiento**| [Sugerencias de rendimiento para la versión 4 del SDK de Java](performance-tips-java-sdk-v4-sql.md)| 
| **Solución de problemas** | [Solución de problemas de la versión 4 del SDK de Java](troubleshoot-java-sdk-v4-sql.md) |
| **Migración a v4 desde un SDK anterior** | [Migración a la versión 4 del SDK de Java](migrate-java-v4-sdk.md) |
| **Tiempo de ejecución mínimo admitido**|[JDK 8](/java/azure/jdk/) | 
| **Talleres y laboratorios de Azure Cosmos DB** |[Página principal de talleres de Cosmos DB](https://aka.ms/cosmosworkshop)

[!INCLUDE[Release notes](~/azure-sdk-for-java-cosmos-db/sdk/cosmos/azure-cosmos/CHANGELOG.md)]

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)] 

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).