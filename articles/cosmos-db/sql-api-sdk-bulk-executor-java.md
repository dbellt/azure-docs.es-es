---
title: 'Azure Cosmos DB: API, SDK y recursos de Java para Bulk Executor'
description: Obtenga toda la información sobre la API y el SDK de Java para BulkExecutor, incluidas la fechas de lanzamiento, fechas de retirada y cambios realizados entre las versiones del SDK de Java para BulkExecutor de Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 04/06/2021
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 09fa626e77e20feff55f7b17807754ac1d2b873f
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111568853"
---
# <a name="java-bulk-executor-library-download-information"></a>Biblioteca BulkExecutor para Java: Información de descarga
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

| | Vínculo/notas |
|---|---|
|**Descripción**|La biblioteca BulkExecutor permite a las aplicaciones cliente realizar operaciones en masa en las cuentas de Azure Cosmos DB. La biblioteca BulkExecutor proporciona los espacios de nombres BulkImport y BulkUpdate. El módulo BulkImport puede ingerir documentos en masa de forma optimizada, de tal forma que la capacidad de proceso aprovisionada para una colección se consuma en el máximo nivel posible. El módulo BulkUpdate puede actualizar en masa los datos existentes en los contenedores de Azure Cosmos como revisiones.|
|**Descarga del SDK**|[Maven](https://search.maven.org/#search%7Cga%7C1%7Cdocumentdb-bulkexecutor)|
|**Biblioteca BulkExecutor en GitHub**|[GitHub](https://github.com/Azure/azure-cosmosdb-bulkexecutor-java-getting-started)|
| **Documentación de la API**| [Documentación de referencia de API](/java/api/com.microsoft.azure.documentdb.bulkexecutor)|
|**Introducción**|[Introducción al SDK de Java para la biblioteca BulkExecutor](bulk-executor-java.md)|
|**Tiempo de ejecución mínimo admitido**|[Java Development Kit (JDK) 7+](/java/azure/jdk/)|

## <a name="release-notes"></a>Notas de la versión
### <a name="2123"></a><a name="2.12.3"></a>2.12.3

* Corrección de la directiva de reintentos cuando `GoneException` se encapsula en `IllegalStateException`: este cambio es necesario para asegurarse de que la memoria caché de Gateway se actualiza en la versión 410 para que el conector de Spark (en Spark 2.4) pueda usar una directiva de reintentos personalizada a fin de permitir que las consultas se realicen correctamente durante las divisiones de particiones.

### <a name="2122"></a><a name="2.12.2"></a>2.12.2

* Corrección de un error que daba como resultado que los documentos no se importaran siempre en caso de errores transitorios.

### <a name="2121"></a><a name="2.12.1"></a>2.12.1

* Actualización para usar la última versión del SDK de Cosmos Core.

### <a name="2120"></a><a name="2.12.0"></a>2.12.0

* Mejora del control del presupuesto de RU que se proporciona a través del conector de Spark para una operación masiva. Se realiza una importación en bloque única inicial desde el conector de Spark con un valor baseBatchSize y se recopila el consumo de RU para la importación en bloque anterior.
  Se calcula el valor miniBatchSizeAdjustmentFactor en función del consumo de RU anterior y se toma como base para ajustar el tamaño del mini lote. En función del tiempo transcurrido y del consumo de RU para cada importación por lotes, se calcula una duración de la suspensión para limitar el consumo de RU por segundo y se usa para pausar el subproceso antes de la importación por lotes siguiente.

### <a name="2110"></a><a name="2.11.0"></a>2.11.0

* Corrección de un error que impedía las actualizaciones masivas al usar una clave de partición anidada.

### <a name="2100"></a><a name="2.10.0"></a>2.10.0

* Corrección de DocumentAnalyzer.java para extraer correctamente los valores de clave de partición anidada de JSON.

### <a name="294"></a><a name="2.9.4"></a>2.9.4

* Se agregó una funcionalidad en las operaciones BulkDelete para volver a intentarlo en determinados errores, así como para devolver al usuario una lista de errores que se pueden reintentar.

### <a name="293"></a><a name="2.9.3"></a>2.9.3

* Actualización de la versión 2.4.7 del SDK de Cosmos.

### <a name="292"></a><a name="2.9.2"></a>2.9.2

* Corrección de "mergeAll" para continuar en "id" y el valor de la clave de partición, de modo que las propiedades de documento con revisiones que se coloquen después de "id" y el valor de clave de partición se agreguen a la lista de elementos actualizados.

### <a name="291"></a><a name="2.9.1"></a>2.9.1

* Actualización del grado de simultaneidad de inicio a 1 y adición de registros de depuración para minilote.