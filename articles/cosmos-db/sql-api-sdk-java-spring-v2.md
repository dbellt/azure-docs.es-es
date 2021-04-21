---
title: Notas de la versión y recursos de Spring Data Azure Cosmos DB v2 para SQL API
description: Descubra toda la información de Spring Data Azure Cosmos DB v2 para SQL API, como las fechas de publicación, las fechas de retirada y los cambios realizados entre las diferentes versiones de Azure Cosmos DB SQL Async Java SDK.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 04/06/2021
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 4cd92efb7b6596288e4b6ef8e81f82f775a24e01
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363451"
---
# <a name="spring-data-azure-cosmos-db-v2-for-core-sql-api-release-notes-and-resources"></a>Spring Data Azure Cosmos DB v2 para Core (SQL) API: Notas de la versión y recursos
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
> * [Conector OLTP de Spark 3](sql-api-sdk-java-spark-v3.md)
> * [Conector OLTP de Spark 2](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

 La versión 2 de Spring Data Azure Cosmos DB para Core (SQL) permite a los desarrolladores utilizar Azure Cosmos DB en aplicaciones de Spring. Spring Data Azure Cosmos DB expone la interfaz de Spring Data para manipular las bases de datos y las colecciones, trabajar con documentos y emitir consultas. Las API sincrónicas y las API asincrónicas (reactivas) se admiten en el mismo artefacto Maven. 

[Spring Framework](https://spring.io/projects/spring-framework) es un modelo de programación y configuración que simplifica el desarrollo de aplicaciones Java. Spring simplifica las relaciones de las aplicaciones mediante la inserción dependencias. Son muchos los desarrolladores que prefieren Spring, ya que resulta más sencillo compilar y probar las aplicaciones. [Spring Boot](https://spring.io/projects/spring-boot) amplía este control de los recursos pensando en la implementación de microservicios y aplicaciones web. [Spring Data](https://spring.io/projects/spring-data) es un modelo de programación para acceder a almacenes de datos como Azure Cosmos DB desde el contexto de una aplicación de Spring o Spring Boot. 

Puede usar Spring Data Azure Cosmos DB en las aplicaciones de [Azure Spring Cloud](https://azure.microsoft.com/services/spring-cloud/).

> [!IMPORTANT]  
> Estas notas de la versión corresponden a la versión 2 de Spring Data Azure Cosmos DB. Puede consultar las [notas de la versión 3 aquí](sql-api-sdk-java-spring-v3.md). 
>
> Spring Data Azure Cosmos DB solo admite SQL API.
>
> Consulte estos artículos para más información sobre Spring Data en otras API de Azure Cosmos DB:
> * [Spring Data para Apache Cassandra con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Spring Data Gremlin con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>
> ¿Desea empezar a trabajar rápidamente?
> 1. Instale el [entorno de ejecución de Java mínimo admitido, JDK 8](/java/azure/jdk/), para poder usar el SDK.
> 2. Cree una aplicación de Spring Data Azure Cosmos DB con la utilidad [Starter](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db). Es muy sencillo.
> 3. Siga la [Guía del desarrollador de Spring Data Azure Cosmos DB](/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb), donde encontrará instrucciones sobre las solicitudes básicas de Azure Cosmos DB.
>
> Puede poner en marcha aplicaciones de Spring Boot Starter rápidamente con [Spring Initializr](https://start.spring.io/).
>

## <a name="resources"></a>Recursos

| Recurso | Vínculo |
|---|---|
| **Descarga del SDK** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/spring-data-cosmosdb) |
|**Documentación de la API** | [Documentación de referencia de Spring Data Azure Cosmos DB]() |
|**Contribuciones al SDK** | [Repositorio de Spring Data Azure Cosmos DB en GitHub](https://github.com/microsoft/spring-data-cosmosdb) | 
|**Spring Boot Starter**| [Biblioteca cliente de Azure Cosmos DB Spring Boot Starter para Java](https://github.com/MicrosoftDocs/azure-dev-docs/blob/master/articles/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db.md) |
|**Ejemplo de aplicación TODO de Spring con Azure Cosmos DB**| [Experiencia completa en Java en App Service Linux (parte 2)](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2) |
|**Guía del desarrollador** | [Guía del desarrollador de Spring Data Azure Cosmos DB](/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) | 
|**Uso de Starter** | [Uso de la utilidad Spring Boot Starter con SQL API para Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db) <br> [Repositorio de GitHub para Azure Cosmos DB Spring Boot Starter](https://github.com/MicrosoftDocs/azure-dev-docs/blob/master/articles/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db.md) |
|**Ejemplo con Azure App Service** | [Cómo usar Spring y Cosmos DB con App Service en Linux](/azure/developer/java/spring-framework/configure-spring-app-with-cosmos-db-on-app-service-linux) <br> [Ejemplo de aplicación TODO](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git) |

## <a name="release-history"></a>Historial de versiones

### <a name="230-may-21-2020"></a>2.3.0 (21 de mayo de 2020)
#### <a name="new-features"></a>Nuevas características
* Se ha actualizado la versión de Spring Boot a 2.3.0.


### <a name="225-may-19-2020"></a>2.2.5 (19 de mayo de 2020)
#### <a name="new-features"></a>Nuevas características
* Se ha actualizado la versión de Azure Cosmos DB a 3.7.3.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Contiene correcciones de fuga de memoria y actualizaciones de versiones de Netty a partir de la versión 3.7.3 de Azure Cosmos DB SDK.

### <a name="224-april-6-2020"></a>2.2.4 (6 de abril de 2020)
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido la marca `allowTelemetry` que debe tenerse en cuenta en `CosmosDbConfig`.
* Se ha corregido la propiedad `TTL` del contenedor.

### <a name="223-february-25-2020"></a>2.2.3 (25 de febrero de 2020)
#### <a name="new-features"></a>Nuevas características
* Se ha agregado un nuevo objeto `findAll` en cada API de clave de partición.
* Versión de Azure Cosmos DB actualizada a 3.7.0.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido `collectionName` -> `containerName`.
* Se ha corregido `entityClass` y `domainClass` -> `domainType`.
* Se ha corregido el error "Return entity collection saved by repository instead of input entity" ("Devolver colección de entidades guardada por el repositorio en lugar de la entidad de entrada").

### <a name="2110-february-25-2020"></a>2.1.10 (25 de febrero de 2020)
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido mediante retroimportación (backport) el error "Return entity collection saved by repository instead of input entity" ("Devolver colección de entidades guardada por el repositorio en lugar de la entidad de entrada").

### <a name="222-january-15-2020"></a>2.2.2 (15 de enero de 2020)
#### <a name="new-features"></a>Nuevas características
* La versión de Azure Cosmos DB se ha actualizado a 3.6.0.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave

### <a name="221-december-31-2019"></a>2.2.1 (31 de diciembre de 2019)
#### <a name="new-features"></a>Nuevas características
* La versión de Azure Cosmos DB SDK se ha actualizado a 3.5.0.
* Se ha agregado un campo de anotación para habilitar o deshabilitar la creación automática de colecciones.
* Se ha mejorado el control de excepciones. `CosmosClientException` se expone mediante `CosmosDBAccessException`.
* `requestCharge` y `activityId` se exponen mediante `ResponseDiagnostics`.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* La actualización 3.5.0 del SDK corrige el error "Exception when Cosmos DB HTTP response header is larger than 8192 bytes" ("Excepción cuando el encabezado de respuesta HTTP de Cosmos DB es mayor que 8192 bytes"), "ConsistencyPolicy.defaultConsistencyLevel() fails on Bounded Staleness and Consistent Prefix" ["ConsistencyPolicy.defaultConsistencyLevel() produce un error en la obsolescencia limitada y prefijo coherente"].
* Se ha corregido el comportamiento del método `findById`. Anteriormente, si no se encontraba la entidad, este método devolvía un valor vacío en lugar de generar una excepción.
* Se ha corregido un error por el que no se aplicaba la ordenación en la página siguiente cuando se usaba `CosmosPageRequest`.

### <a name="219-december-26-2019"></a>2.1.9 (26 de diciembre de 2019)
#### <a name="new-features"></a>Nuevas características
* Se ha agregado un campo de anotación para habilitar o deshabilitar la creación automática de colecciones.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
*  Se ha corregido el comportamiento del método `findById`. Anteriormente, si no se encontraba la entidad, este método devolvía un valor vacío en lugar de generar una excepción.

### <a name="220-october-21-2019"></a>2.2.0 (21 de octubre de 2019)
#### <a name="new-features"></a>Nuevas características
* Ahora el repositorio reactivo de Cosmos es totalmente compatible.
* Se admiten las métricas de consulta y de cadenas de diagnóstico de solicitud de Cosmos DB.
* Se ha actualizado la versión de Cosmos DB SDK a 3.3.1.
* Se ha actualizado la versión de Spring Framework a 5.2.0.VERSIÓN.
* Se ha actualizado la versión de Spring Data Commons a 2.2.0.VERSIÓN.
* Se han agregado las API `findByIdAndPartitionKey` y `deleteByIdAndPartitionKey`.
* Se ha quitado la dependencia de azure-doumentdb.
* El nombre DocumentDB se ha reemplazado por Azure Cosmos DB.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido el error "Sorting throws exception when pageSize is less than total items in repository" ("La ordenación genera una excepción cuando pageSize es inferior a los elementos totales del repositorio").

### <a name="218-october-18-2019"></a>2.1.8 (18 de octubre de 2019)
#### <a name="new-features"></a>Nuevas características
* Han dejado de utilizarse las API de DocumentDB.
* Se han agregado las API `findByIdAndPartitionKey` y `deleteByIdAndPartitionKey`.
* Se ha agregado un bloqueo optimista basado en `_etag`.
* Se ha habilitado la expresión SpEL para el nombre de la colección de documentos.
* Se han agregado mejoras de `ObjectMapper`.

### <a name="217-october-18-2019"></a>2.1.7 (18 de octubre de 2019)
#### <a name="new-features"></a>Nuevas características
* Se ha agregado una dependencia a la versión 3 de Azure Cosmos DB SDK.
* Se ha agregado el repositorio reactivo de Cosmos.
* Se ha actualizado la implementación de `DocumentDbTemplate` para usar la versión 3 de Azure Cosmos DB SDK.
* Se han agregado otros cambios de configuración para permitir la compatibilidad con el repositorio reactivo de Cosmos.

### <a name="212-march-19-2019"></a>2.1.2 (19 de marzo de 2019)
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha quitado la dependencia de `applicationInsights` por:
    * Riesgo potencial de contaminación de dependencias.
    * Incompatibilidad con Java 11.
    * Evitar el posible impacto en el rendimiento de la CPU o la memoria.

### <a name="207-march-20-2019"></a>2.0.7 (20 de marzo de 2019)
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha quitado mediante backport la dependencia de `applicationInsights` por:
    * Riesgo potencial de contaminación de dependencias.
    * Incompatibilidad con Java 11.
    * Evitar el posible impacto en el rendimiento de la CPU o la memoria.

### <a name="211-march-7-2019"></a>2.1.1 (7 de marzo de 2019)
#### <a name="new-features"></a>Nuevas características
* La versión principal se ha actualizado a 2.1.1.

### <a name="206-march-7-2019"></a>2.0.6 (7 de marzo de 2019)
#### <a name="new-features"></a>Nuevas características
* Se han omitido todas las excepciones de telemetría.

### <a name="210-december-17-2018"></a>2.1.0 (17 de diciembre de 2018)
#### <a name="new-features"></a>Nuevas características
* Se ha actualizado a la versión 2.1.0 para solucionar un problema.

### <a name="205-september-13-2018"></a>2.0.5 (13 de septiembre de 2018)
#### <a name="new-features"></a>Nuevas características
* Se han agregado las palabra clave `exists` y `startsWith`.
* Se ha actualizado el archivo Léame.
#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se ha corregido el error "Can't call self href directly for Entity." ("No se puede llamar directamente al elemento self href de la entidad").
* Se ha corregido el error "findAll will fail if collection is not created" ("findAll generará error si no se crea la colección").

### <a name="204-prerelease-august-23-2018"></a>2.0.4 (versión preliminar) (23 de agosto de 2018)
#### <a name="new-features"></a>Nuevas características
* El nombre del paquete documentdb ha cambiado a cosmosdb.
* Se ha agregado una nueva característica para las palabras clave del método de consulta. Ahora se admiten 16 palabras clave en SQL API.
* Se ha agregado una nueva característica de consulta que permite paginar y ordenar.
* Se ha simplificado la configuración de spring-data-cosmosdb.
* Se han agregado las API `deleteCollection` y `deleteAll`.

#### <a name="key-bug-fixes"></a>Correcciones de errores clave
* Se han corregido errores y mitigado defectos.

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Pasos siguientes
Más información acerca de [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Más información sobre [Spring Framework](https://spring.io/projects/spring-framework).

Más información sobre [Spring Boot](https://spring.io/projects/spring-boot).

Más información sobre [Spring Data](https://spring.io/projects/spring-data).