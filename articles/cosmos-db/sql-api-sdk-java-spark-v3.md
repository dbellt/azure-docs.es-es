---
title: Notas de la versión y recursos del conector OLTP de Azure Cosmos DB Apache Spark 3 para SQL API (versión preliminar)
description: Conozca el conector OLPT de Azure Cosmos DB Apache Spark 3 para SQL API (versión preliminar), como las fechas de lanzamiento, las fechas de retirada y los cambios realizados entre versiones del SDK de Java de Azure Cosmos DB para SQL.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 04/06/2021
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 9b898ea6c12b79035d2a9e968f38d7b202f19ddc
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747642"
---
# <a name="azure-cosmos-db-apache-spark-3-oltp-connector-for-core-sql-api-preview-release-notes-and-resources"></a>Conector OLTP de Azure Cosmos DB Apache Spark 3 para Core (SQL API) (versión preliminar): notas de la versión y recursos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [.NET SDK v2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET, versión 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK para Java v4](sql-api-sdk-java-v4.md)
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

El **conector OLPT de Azure Cosmos DB Spark 3 (versión preliminar)** proporciona compatibilidad con Apache Spark v3 en Azure Cosmos DB mediante SQL API.
[Azure Cosmos DB](introduction.md) es un servicio de base de datos de distribución global que permite a los desarrolladores trabajar con datos mediante diversas API estándar, como SQL, MongoDB, Cassandra, Graph y Table.

> [!Note]
> Esta versión del conector OLPT de Azure Cosmos DB Spark 3 es una compilación en versión preliminar.
> No se ha probado la carga ni el rendimiento de esta compilación, por lo que
> no se recomienda el uso de esta compilación en escenarios de producción.
>

## <a name="documentation"></a>Documentación

- [Introducción](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos-spark_3-1_2-12/docs/quick-start.md)
- [API de catálogo](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos-spark_3-1_2-12/docs/catalog-api.md)
- [Referencia de parámetros de configuración](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos-spark_3-1_2-12/docs/configuration-reference.md)


## <a name="version-compatibility"></a>Compatibilidad de versiones

| Conector     | Spark         | Versión mínima de Java | Versiones de Scala admitidas |
| ------------- | ------------- | -------------------- | -----------------------  |
| 4.0.0-beta.1  | 3.1.1         |        8             | 2,12                     |

## <a name="download"></a>Descargar

Puede usar la coordenada de Maven del archivo JAR para instalar automáticamente el conector de Spark en Databricks Runtime 8 desde Maven: `com.azure.cosmos.spark:azure-cosmos-spark_3-1_2-12:4.0.0-beta.1`.

También puede realizar la integración con el conector Cosmos DB Spark en el proyecto de SBT:
```scala
libraryDependencies += "com.azure.cosmos.spark" % "azure-cosmos-spark_3-1_2-12" % "4.0.0-beta.1"
```

El conector Cosmos DB Spark está disponible en el [repositorio central de Maven](https://search.maven.org/artifact/com.azure.cosmos.spark/azure-cosmos-spark_3-1_2-12/4.0.0-beta.1/jar).

### <a name="general"></a>General

Si encuentra algún error, registre una incidencia [aquí](https://github.com/Azure/azure-sdk-for-java/issues/new).

Para sugerir una nueva característica o cambios que se podrían realizar, registre una incidencia de la misma manera que haría en el caso de un error.

[!INCLUDE[Changelog](~/azure-sdk-for-java-cosmos-db/sdk/cosmos/azure-cosmos-spark_3-1_2-12/CHANGELOG.md)]

## <a name="next-steps"></a>Pasos siguientes

Revise nuestra [guía de inicio rápido para trabajar con el conector OLTP de Azure Cosmos DB Spark 3](create-sql-api-spark.md).
