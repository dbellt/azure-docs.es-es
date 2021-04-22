---
title: Notas de la versión y recursos de Spring Data Azure Cosmos DB v3 para API SQL
description: Conozca Spring Data Azure Cosmos DB v3 para la API de SQL, incluidas las fechas de lanzamiento y de retirada, y los cambios realizados entre las versiones del SDK de Java asincrónico para SQL de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 04/06/2021
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 594d38425be0304a9f7737bdfba60b29187a2e2d
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363519"
---
# <a name="spring-data-azure-cosmos-db-v3-for-core-sql-api-release-notes-and-resources"></a>Spring Data Azure Cosmos DB v3 para la API Core (SQL): Notas de la versión y recursos
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

La versión 3 de Spring Data Azure Cosmos DB para Core (SQL) permite que los desarrolladores utilicen Azure Cosmos DB en aplicaciones de Spring. Spring Data Azure Cosmos DB expone la interfaz de Spring Data para manipular las bases de datos y las colecciones, trabajar con documentos y emitir consultas. Las API sincrónicas y las API asincrónicas (reactivas) se admiten en el mismo artefacto Maven. 

> [!IMPORTANT]
> Spring Data Azure Cosmos DB tiene una dependencia en la plataforma de Spring Data.
> 
> Las versiones 3.0.0 a 3.4.0 de azure-spring-data-cosmos admiten las versiones 2.2 y 2.3 de Spring Data.
> 
> Las versiones 3.5.0 y superiores de azure-spring-data-cosmos admiten las versiones 2.4.3 y superiores de Spring Data.
>

[Spring Framework](https://spring.io/projects/spring-framework) es un modelo de programación y configuración que simplifica el desarrollo de aplicaciones Java. Spring simplifica las relaciones de las aplicaciones mediante la inserción dependencias. Son muchos los desarrolladores que prefieren Spring, ya que resulta más sencillo compilar y probar las aplicaciones. [Spring Boot](https://spring.io/projects/spring-boot) amplía este control de los recursos pensando en la implementación de microservicios y aplicaciones web. [Spring Data](https://spring.io/projects/spring-data) es un modelo y una plataforma de programación para acceder a almacenes de datos como Azure Cosmos DB desde el contexto de una aplicación de Spring o Spring Boot. 

Puede usar Spring Data Azure Cosmos DB en las aplicaciones de [Azure Spring Cloud](https://azure.microsoft.com/services/spring-cloud/).

> [!IMPORTANT]  
> Estas notas de la versión corresponden a la versión 3 de Spring Data Azure Cosmos DB. Puede encontrar las [notas de la versión 2 aquí](sql-api-sdk-java-spring-v2.md). 
>
> Spring Data Azure Cosmos DB solo admite la API de SQL.
>
> Consulte estos artículos para obtener información sobre Spring Data en otras API de Azure Cosmos DB:
> * [Spring Data para Apache Cassandra con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Spring Data Gremlin con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>

## <a name="get-started-fast"></a>Empiece rápidamente

  Empiece a trabajar rápidamente con Spring Data Azure Cosmos DB gracias a la [Guía de Spring Boot Starter](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db). El enfoque de Spring Boot Starter es el método recomendado para empezar a usar el conector Spring Data Azure Cosmos DB.

  También puede agregar la dependencia Spring Data Azure Cosmos DB al archivo `pom.xml`, como se muestra a continuación:

  ```xml
  <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-spring-data-cosmos</artifactId>
      <version>latest-version</version>
  </dependency>
  ```

## <a name="helpful-content"></a>Contenido útil

| Contenido | Vínculo |
|---|---|
|**Descarga del SDK**| [Maven](https://mvnrepository.com/artifact/com.azure/azure-spring-data-cosmos) |
|**Documentación de la API** | [Documentación de referencia de API](/java/api/com.azure.spring.data.cosmos) |
|**Contribuya al SDK** | [Repositorio central del SDK de Azure para Java en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-cosmos) | 
|**Introducción** | [Inicio rápido: Creación de una aplicación de Spring Data Azure Cosmos DB para administrar los datos de SQL API de Azure Cosmos DB](./create-sql-api-spring-data.md) <br> [Repositorio de GitHub con código de inicio rápido](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started) | 
|**Ejemplos de código básicos** | [Azure Cosmos DB: Ejemplos de Spring Data Azure Cosmos DB para SQL API](sql-api-spring-data-sdk-samples.md) <br> [Repositorio de GitHub con código de ejemplo](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples)|
| **Consejos de rendimiento**| [Sugerencias de rendimiento para el SDK de Java v4 (aplicable a Spring Data)](performance-tips-java-sdk-v4-sql.md)| 
| **Solución de problemas** | [Solución de problemas del SDK de Java v4 (aplicable a Spring Data)](troubleshoot-java-sdk-v4-sql.md) | 
| **Talleres y laboratorios de Azure Cosmos DB** |[Página principal de talleres de Cosmos DB](https://aka.ms/cosmosworkshop)

[!INCLUDE[Release notes](~/azure-sdk-for-java-cosmos-db/sdk/cosmos/azure-spring-data-cosmos/CHANGELOG.md)]

## <a name="additional-notes"></a>Notas adicionales

* Spring Data Azure Cosmos DB admite el SDK de Java 8 y Java JDK 11.
* Actualmente se admite Spring Data 2.3; Spring Data 2.4 no se admite en este momento.

## <a name="faq"></a>Preguntas más frecuentes

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Más información sobre [Spring Framework](https://spring.io/projects/spring-framework).

Más información sobre [Spring Boot](https://spring.io/projects/spring-boot).

Más información sobre [Spring Data](https://spring.io/projects/spring-data).