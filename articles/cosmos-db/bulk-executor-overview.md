---
title: Información general sobre la biblioteca BulkExecutor de Azure Cosmos DB
description: Realizar operaciones masivas en Azure Cosmos DB a través de las API de importación en bloque y actualización masiva que ofrece la biblioteca BulkExecutor.
author: tknandu
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/28/2019
ms.author: ramkris
ms.reviewer: sngun
ms.openlocfilehash: 159b44c4002c838a22656e8a8ddc5f5f6efd69b7
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971945"
---
# <a name="azure-cosmos-db-bulk-executor-library-overview"></a>Información general sobre la biblioteca BulkExecutor de Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
 
Azure Cosmos DB es un servicio de base de datos rápido, flexible y distribuido globalmente que está diseñado para escalar horizontalmente de forma flexible, a fin de admitir: 

* Gran capacidad de proceso de lectura y escritura (millones de operaciones por segundo).  
* Almacenamiento de grandes volúmenes de (cientos de terabytes o incluso más) y datos transaccionales y operativos con una latencia predecible de milisegundos.  

La biblioteca BulkExecutor le ayuda a aprovechar esta capacidad de proceso y almacenamiento masiva. La biblioteca BulkExecutor permite realizar operaciones masivas en Azure Cosmos DB a través de las API de importación en bloque y actualización masiva. Puede obtener más información sobre las características de la biblioteca BulkExecutor en las siguientes secciones. 

> [!NOTE] 
> Actualmente, la biblioteca BulkExecutor admite las operaciones de importación y actualización, y solo es compatible con las cuentas de API de SQL de Azure Cosmos DB y API de Gremlin.

> [!IMPORTANT]
> La biblioteca BulkExecutor no es compatible actualmente con las cuentas [sin servidor](serverless.md). En .NET, se recomienda usar [BulkSupport](https://devblogs.microsoft.com/cosmosdb/introducing-bulk-support-in-the-net-sdk/) disponible en la versión 3 del SDK.
 
## <a name="key-features-of-the-bulk-executor-library"></a>Características clave de la biblioteca BulkExecutor  
 
* Reduce significativamente los recursos de proceso del lado cliente necesarios para saturar la capacidad de proceso asignada a un contenedor. Una sola aplicación multiproceso que escribe datos mediante la API de importación masiva logra diez veces más capacidad de proceso de escritura en comparación con una aplicación multiproceso que escribe datos en paralelo mientras se satura la CPU del equipo cliente.  

* Elimina las laboriosa tarea de escribir lógica de aplicación para controlar la limitación de solicitudes, los tiempos de espera de las solicitudes y otras excepciones transitorias mediante un control eficaz de ellas en la biblioteca.  

* Proporciona un mecanismo simplificado para aplicaciones que realizan operaciones en masa para escalabilidad horizontal. Una única instancia de BulkExecutor en ejecución en una máquina virtual de Azure puede consumir más de 500 000 RU/s y puede conseguir mayor velocidad de la capacidad de proceso con la adición de instancias adicionales a las máquinas virtuales clientes individuales.  
 
* Puede importar en masa más de un terabyte de datos en una hora con el uso de una arquitectura de escalabilidad horizontal.  

* Puede actualizar en masa los datos existentes en los contenedores de Azure Cosmos como revisiones. 
 
## <a name="how-does-the-bulk-executor-operate"></a>¿Cómo funciona el ejecutor en masa? 

Cuando se desencadena una operación en masa para importar o actualizar documentos con un lote de entidades, inicialmente se ordenan aleatoriamente en cubos correspondientes a su rango con clave de particiones de Azure Cosmos DB. Dentro de cada cubo correspondiente a un rango con clave de particiones, se dividen en mini lotes y cada mini lote actúa como una carga útil que se confirma en el lado del servidor. La biblioteca BulkExecutor tiene optimizaciones integradas para la ejecución simultánea de estos mini lotes dentro y entre los rangos con clave de particiones. En la siguiente imagen se ilustra cómo BulkExecutor procesa los datos por lotes en diferentes claves de particiones:  

:::image type="content" source="./media/bulk-executor-overview/bulk-executor-architecture.png" alt-text="Arquitectura de Bulk Executor" :::

La biblioteca Bulk Executor se asegura de la máxima utilización de la capacidad de proceso asignada a una colección. Usa un  [mecanismo de control de la congestión del estilo de AIMD](https://tools.ietf.org/html/rfc5681) para que cada rango con clave de particiones de Azure Cosmos DB controle con eficacia las limitaciones de velocidad y los tiempos de espera. 

## <a name="next-steps"></a>Pasos siguientes 
  
* Puede aprender más si prueba las aplicaciones de ejemplo que usan la biblioteca BulkExecutor en [.NET](bulk-executor-dot-net.md) y [Java](bulk-executor-java.md).  
* Consulte la información sobre el SDK y las notas de la versión de BulkExecutor en [.NET](sql-api-sdk-bulk-executor-dot-net.md) y [Java](sql-api-sdk-bulk-executor-java.md).
* La biblioteca BulkExecutor está integrada en el conector de Spark a Cosmos DB; para más información, vea el artículo sobre el [conector de Spark a Azure Cosmos DB](./create-sql-api-spark.md).  
* La biblioteca de BulkExecutor también está integrada en una nueva versión del [conector de Azure Cosmos DB](../data-factory/connector-azure-cosmos-db.md) para que Azure Data Factory copie datos.