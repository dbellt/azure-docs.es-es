---
title: Introducción a API de Azure Cosmos DB para MongoDB
description: Vea cómo puede usar Azure Cosmos DB para almacenar y consultar grandes cantidades de datos mediante la API de Azure Cosmos DB para MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 04/21/2021
author: sivethe
ms.author: sivethe
adobe-target: true
adobe-target-activity: DocsExp– 396298–A/B–Docs–IntroToCosmosDBAPIforMongoDB-Revamp–FY21Q4
adobe-target-experience: Experience B
adobe-target-content: ./mongodb-introduction-experiment
ms.openlocfilehash: 518eaadf75a5ff2cabc541586fcdf029b0ca1c60
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107887102"
---
# <a name="azure-cosmos-dbs-api-for-mongodb"></a>API de Azure Cosmos DB para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

[Azure Cosmos DB](introduction.md) es un servicio de base de datos con varios modelos y distribución global de Microsoft para aplicaciones críticas. Azure Cosmos DB ofrece una [distribución global inmediata](distribute-data-globally.md), [escalado elástico de rendimiento y almacenamiento](partitioning-overview.md) en todo el mundo, latencias de menos de 10 ms en el percentil 99 y alta disponibilidad garantizada, todo ello respaldado por [acuerdos de nivel de servicio líderes del sector](https://azure.microsoft.com/support/legal/sla/cosmos-db/). Azure Cosmos DB [indexa datos automáticamente](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) sin que haya que ocuparse de la administración de esquemas ni de índices. Sigue varios modelos y es compatible con los modelos de datos de documento, de clave-valor, de grafo y de columnas. El servicio Cosmos DB implementa los protocolos de conexión para las API de NoSQL comunes como Cassandra, MongoDB, Gremlin y Azure Table Storage. Esto le permite usar los controladores de cliente de NoSQL y las herramientas que ya conoce para interactuar con la base de datos de Cosmos.

> [!NOTE]
> El [modo de capacidad sin servidor](serverless.md) ahora está disponible en la API para MongoDB de Azure Cosmos DB.

## <a name="wire-protocol-compatibility"></a>Compatibilidad de protocolo de conexión

Azure Cosmos DB implementa el protocolo de conexión para MongoDB. Esta implementación permite una compatibilidad transparente con las herramientas, los controladores y los SDK de cliente nativos de MongoDB. Azure Cosmos DB no hospeda el motor de base de datos de MongoDB. Los detalles de las características admitidas por MongoDB se pueden encontrar aquí: 
- [API de Azure Cosmos DB para MongoDB versión 4.0](mongodb-feature-support-40.md)
- [API de Azure Cosmos DB para MongoDB versión 3.6](mongodb-feature-support-36.md)

De forma predeterminada, las nuevas cuentas creadas mediante la API de Azure Cosmos DB para MongoDB son compatibles con la versión 3.6 del protocolo de conexión de MongoDB. Cualquier controlador cliente de MongoDB que reconozca esta versión del protocolo debería poder conectarse de forma nativa a Cosmos DB.

:::image type="content" source="./media/mongodb-introduction/cosmosdb-mongodb.png" alt-text="API de Azure Cosmos DB para MongoDB" border="false":::

## <a name="key-benefits"></a>Ventajas principales

Las principales ventajas de Cosmos DB como base de datos completamente administrada y distribuida globalmente como servicio se describen [aquí](introduction.md). Además, al implementar de forma nativa los protocolos de conexión de API de NoSQL populares, Cosmos DB ofrece las siguientes ventajas:

* Migrar fácilmente la aplicación a Cosmos DB, al tiempo que conserva partes importantes de la lógica de aplicación.
* Mantener la aplicación portátil y permanecer independiente de proveedores de nube.
* Obtenga contratos de nivel de servicio con respaldo financiero líderes del sector para API de NoSQL comunes con tecnología de Cosmos DB.
* Escale de forma elástica el rendimiento y el almacenamiento aprovisionados para sus bases de datos de Cosmos según sus necesidades y pague solo el rendimiento y el almacenamiento que necesita. lo que se traduce en un importante ahorro de costos.
* Distribución global llave en mano con replicación de escritura en varias regiones.

## <a name="cosmos-dbs-api-for-mongodb"></a>API de Cosmos DB para MongoDB

Siga los artículos de inicio rápido si desea crear una cuenta de Azure Cosmos y migrar la aplicación de MongoDB para usar Azure Cosmos DB, o bien compile una nueva:

* [Migración de una aplicación web Node.js de MongoDB existente](create-mongodb-nodejs.md)
* [Compilación de una aplicación web mediante la API de Azure Cosmos DB para MongoDB y el SDK de .NET](create-mongodb-dotnet.md)
* [Compilación de una aplicación de consola mediante la API de Azure Cosmos DB para MongoDB y el SDK de Java](create-mongodb-java.md)

## <a name="next-steps"></a>Pasos siguientes

Estas son algunas sugerencias para comenzar:

* Para aprender a obtener la información de cadena de conexión de la cuenta, siga el tutorial de [conexión de una aplicación de MongoDB a Azure Cosmos DB](connect-mongodb-account.md).
* Para aprender a crear una conexión entre la base de datos de Azure Cosmos DB y la aplicación de MongoDB en Studio 3T, siga el tutorial de [uso de Studio 3T con Azure Cosmos DB](mongodb-mongochef.md).
* Para importar los datos a una base de datos de Cosmos, siga el tutorial de [importación de datos de MongoDB en Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json).
* Conéctese a una cuenta de Cosmos con [Robo 3T](mongodb-robomongo.md).
* Aprenda a [configurar las preferencias de lectura para las aplicaciones distribuidas globalmente](../cosmos-db/tutorial-global-distribution-mongodb.md).
* Busque las soluciones a los errores más comunes en nuestra [Guía de solución de problemas](mongodb-troubleshoot.md).


<sup>Nota: En este artículo se describe una característica de Azure Cosmos DB que proporciona compatibilidad del protocolo de conexión con bases de datos de MongoDB. Microsoft no ejecuta bases de datos de MongoDB que ofrezcan este servicio. Azure Cosmos DB no está afiliado a MongoDB, Inc.</sup>
