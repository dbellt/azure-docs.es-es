---
title: 'API de SQL de Azure Cosmos DB: Ejemplos de Spring Data v3'
description: Busque ejemplos de Spring Data v3 en GitHub para tareas comunes mediante la API de SQL de Azure Cosmos DB, incluidas las operaciones CRUD.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 09/23/2020
ms.custom: devx-track-java
ms.author: anfeldma
ms.openlocfilehash: 86ad1447261de86035730a86c8c214cd922b4ffd
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110063036"
---
# <a name="azure-cosmos-db-sql-api-spring-data-azure-cosmos-db-v3-examples"></a>API de SQL de Azure Cosmos DB: Ejemplos de Spring Data Azure Cosmos DB v3
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Ejemplos del SDK de .NET V2](sql-api-dotnet-samples.md)
> * [Ejemplos del SDK de .NET V3](sql-api-dotnet-v3sdk-samples.md)
> * [Ejemplos del SDK de Java V4](sql-api-java-sdk-samples.md)
> * [Ejemplos del SDK de Spring Data V3](sql-api-spring-data-sdk-samples.md)
> * [Ejemplos de Node.js](sql-api-nodejs-samples.md)
> * [Ejemplos de Python](sql-api-python-samples.md)
> * [Galería de ejemplos de código de Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

> [!IMPORTANT]  
> Estas notas de la versión corresponden a la versión 3 de Spring Data Azure Cosmos DB. Puede encontrar las [notas de la versión 2 aquí](sql-api-sdk-java-spring-v2.md). 
>
> Spring Data Azure Cosmos DB solo admite la API de SQL.
>
> Consulte estos artículos para obtener información sobre Spring Data en otras API de Azure Cosmos DB:
> * [Spring Data para Apache Cassandra con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
>

> [!IMPORTANT]  
>[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
>  
>- Puede [activar los beneficios de suscripción a Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): su suscripción a Visual Studio le proporciona créditos todos los meses que puede usar para servicios de Azure de pago.
>
>[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]
>

En el repositorio de GitHub [azure-spring-data-cosmos-java-sql-api-samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples) se incluyen las aplicaciones de ejemplo más recientes que realizan operaciones CRUD y otras operaciones comunes en recursos de Azure Cosmos DB. Este artículo ofrece:

* Vínculos a las tareas de cada uno de los archivos de proyecto de ejemplo de Spring Data Azure Cosmos DB. 
* Vínculos al contenido de referencia de la API relacionada.

**Requisitos previos**

Necesita lo siguiente para poder ejecutar esta aplicación de ejemplo:

* Java Development Kit 8
* Spring Data Azure Cosmos DB v3

También puede usar Maven para obtener los archivos binarios más recientes del SDK de Spring Data Azure Cosmos DB v3 para usarlos en el proyecto. Maven se encarga de agregar automáticamente las dependencias necesarias. En caso contrario, puede descargar directamente las dependencias que se enumeran en el archivo **pom.xml** y agregarlas a la ruta de acceso de la compilación.

```bash
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-cosmos</artifactId>
    <version>LATEST</version>
</dependency>
```

**Ejecutar las aplicaciones de ejemplo**

Clone el repositorio de ejemplo:
```bash
$ git clone https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples

$ cd azure-spring-data-cosmos-java-sql-api-samples
```

Puede ejecutar los ejemplos mediante un IDE (Eclipse, IntelliJ o VSCODE) o bien desde la línea de comandos mediante Maven.

En **application.properties**, estas variables de entorno deben estar establecidas

```xml
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

con el fin de dar a los ejemplos acceso de lectura y escritura a su cuenta, sus bases de datos y sus contenedores.

El IDE puede proporcionar la capacidad de ejecutar el código de ejemplo de Spring Data. De lo contrario, puede usar el siguiente comando de terminal para ejecutar el ejemplo:

```bash
mvn spring-boot:run
```

## <a name="document-crud-examples"></a>Ejemplos de CRUD de documentos
El archivo [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) muestra cómo realizar las siguientes tareas. Para obtener información sobre los documentos de Azure Cosmos antes de ejecutar los ejemplos siguientes, consulte el artículo conceptual [Uso de bases de datos, contenedores y elementos](account-databases-containers-items.md).

| Tarea | Referencia de API |
| --- | --- |
| [Creación de un documento](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L46-L47) | CosmosRepository.save |
| [Leer un documento por identificador](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L56-L58) | CosmosRepository.derivedQueryMethod |
| [Eliminar todos los documentos](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L39-L41) | CosmosRepository.deleteAll |

## <a name="derived-query-method-examples"></a>Ejemplos de métodos de consulta derivados
El archivo [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) muestra cómo realizar las siguientes tareas. Para obtener información sobre las consultas de Azure Cosmos DB antes de ejecutar los ejemplos siguientes, es posible que le resulte útil leer el artículo [Derived Query Methods in Spring](https://www.baeldung.com/spring-data-derived-queries) (Métodos de consulta derivados en Spring) de Baeldung.

| [Consulta de documentos](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L73-L77) | CosmosRepository.derivedQueryMethod |

## <a name="custom-query-examples"></a>Ejemplos de consultas personalizadas
El archivo [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) muestra cómo realizar las siguientes tareas con la gramática de consultas SQL. Para más información sobre la referencia de consultas SQL en Azure Cosmos DB antes de ejecutar los ejemplos siguientes, consulte los [ejemplos de consultas SQL para Azure Cosmos DB](./sql-query-getting-started.md). 


| Tarea | Referencia de API |
| --- | --- |
| [Consulta de todos los documentos](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L20-L22) | Anotación @Query |
| [Consulta de igualdad mediante ==](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L24-L26) | Anotación @Query |
| [Consulta de desigualdad mediante != y NOT](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L28-L38) | Anotación @Query |
| [Consulta mediante operadores de intervalo como &gt;, &lt;, &gt;= o &lt;=](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L40-L42) | Anotación @Query |
| [Consulta con operadores de intervalo en cadenas](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L44-L46) | Anotación @Query |
| [Consulta con ORDER BY](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L48-L50) | Anotación @Query |
| [Query with DISTINCT](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L52-L54) | Anotación @Query |
| [Consulta con funciones de agregado](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L56-L62) | Anotación @Query |
| [Uso de subdocumentos](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L64-L66) | Anotación @Query |
| [Consulta con Joins dentro de documentos](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L68-L85) | Anotación @Query |
| [Consulta con operadores de cadena, matemáticos y de matriz](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L87-L97) | Anotación @Query |