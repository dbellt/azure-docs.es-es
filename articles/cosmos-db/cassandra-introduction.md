---
title: Introducción a Cassandra API de Azure Cosmos DB
description: Aprenda a usar Azure Cosmos DB para migrar mediante "lift-and-shift" las aplicaciones existentes y compilar nuevas aplicaciones mediante los controladores de Cassandra y CQL
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: overview
ms.date: 11/25/2020
ms.openlocfilehash: 3a97f62030227eb0c18bd70af2fabbd1f5ef2863
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108771278"
---
# <a name="introduction-to-the-azure-cosmos-db-cassandra-api"></a>Introducción a Cassandra API de Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cassandra API de Azure Cosmos DB se puede usar como almacén de datos para aplicaciones escritas para [Apache Cassandra](https://cassandra.apache.org). Esto significa que si se usan [controladores de Apache](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver) existentes compatibles con CQLv4, la aplicación de Cassandra existente se puede comunicar con Cassandra API de Azure Cosmos DB. En muchos casos puede pasar de usar Apache Cassandra a emplear Cassandra API de Azure Cosmos DB con solo cambiar una cadena de conexión. 

Cassandra API permite interactuar con los datos almacenados en Azure Cosmos DB mediante Cassandra Query Language (CQL), herramientas basadas en Cassandra (como cqlsh) y controladores de cliente de Cassandra con los que ya está familiarizado.

> [!NOTE]
> El [modo de capacidad sin servidor](serverless.md) ahora está disponible en Cassandra API de Azure Cosmos DB.

## <a name="what-is-the-benefit-of-using-apache-cassandra-api-for-azure-cosmos-db"></a>¿Cuál es la ventaja de usar la API Apache Cassandra para Azure Cosmos DB?

**No se produce la administración de las operaciones**: como servicio en la nube completamente administrado, Cassandra API de Azure Cosmos DB evita la sobrecarga de administrar y supervisar innumerables configuraciones en el sistema operativo, JVM y los archivos yaml y sus interacciones. Azure Cosmos DB proporciona supervisión de rendimiento, latencia, almacenamiento, disponibilidad y alertas configurables.

**Estándar de código abierto**: a pesar de ser un servicio totalmente administrado, Cassandra API sigue siendo compatible con una gran superficie del [protocolo de conexión de Apache Cassandra](cassandra-support.md) nativo, lo que le permite crear aplicaciones basadas en un estándar de código abierto independiente de la nube y de uso generalizado.

**Administración del rendimiento**: Azure Cosmos DB proporciona lecturas y escrituras de baja latencia garantizadas en el percentil 99, respaldadas por el Acuerdo de Nivel de Servicio. Los usuarios no tienen que preocuparse por la sobrecarga operativa de garantizar un alto rendimiento y lecturas y escrituras de baja latencia. Esto significa que los usuarios no tienen que programar la compactación, administrar marcadores de exclusión ni configurar filtros de bloom ni réplicas manualmente. Azure Cosmos DB acaba con la sobrecarga de tener que administrar estos problemas y permite centrarse en la lógica de la aplicación.

**Posibilidad de usar código y herramientas existentes**: Azure Cosmos DB proporciona compatibilidad de nivel de protocolo de conexión con SDK y herramientas existentes de Cassandra. Esta compatibilidad garantiza que pueda usar el código base existente con Cassandra API de Azure Cosmos DB con cambios triviales.

**Elasticidad del rendimiento y el almacenamiento**: Azure Cosmos DB proporciona rendimiento en todas las regiones y puede escalar el rendimiento aprovisionado con operaciones de Azure Portal, PowerShell o la CLI. Se puede [escalar de forma elástica](manage-scale-cassandra.md) el almacenamiento y el rendimiento de las tablas según sea necesario con un rendimiento predecible.

**Disponibilidad y distribución globales**: Azure Cosmos DB ofrece la posibilidad de distribuir los datos globalmente en todas las regiones de Azure y de proporcionarlos localmente a la vez que garantiza un acceso a datos de baja latencia y alta disponibilidad. Azure Cosmos DB proporciona una alta disponibilidad del 99,99 % dentro de una región y una disponibilidad de lectura y escritura del 99,999 % en varias regiones sin sobrecarga de operaciones. Obtenga más información en el artículo [Distribución de datos global](distribute-data-globally.md). 

**Opción de coherencia**: Azure Cosmos DB proporciona la opción de cinco niveles de coherencia bien definidos para lograr un equilibrio óptimo entre la coherencia y el rendimiento. Estos niveles de coherencia son Alta, Obsolescencia limitada, Sesión, Prefijo coherente y Posible. Estos niveles de coherencia bien definidos, prácticos e intuitivos permiten a los desarrolladores lograr un equilibrio preciso entre la coherencia, la disponibilidad y la latencia. Obtenga más información en el artículo [Niveles de coherencia](consistency-levels.md). 

**Nivel empresarial**: Azure Cosmos DB proporciona [certificaciones de cumplimiento normativo](https://www.microsoft.com/trustcenter) para garantizar que los usuarios puedan usar la plataforma de forma segura. Azure Cosmos DB también proporciona cifrado en reposo y en movimiento, firewall de IP y registros de auditoría para actividades del plano de control.

**Abastecimiento de eventos**: Cassandra API proporciona acceso a un registro de cambios persistente, la [fuente de cambios](cassandra-change-feed.md), que puede facilitar el abastecimiento de eventos directamente desde la base de datos. En Apache Cassandra, el único equivalente es la captura de datos modificados (CDC), que es simplemente un mecanismo para marcar tablas específicas para el archivado, así como para rechazar escrituras en esas tablas una vez que se alcanza un tamaño en disco configurable para el registro CDC (estas funcionalidades son redundantes en Cosmos DB dado que los aspectos pertinentes se regulan automáticamente).

## <a name="next-steps"></a>Pasos siguientes

* Puede empezar rápidamente a compilar las siguientes aplicaciones específicas de lenguaje para crear y administrar datos de Cassandra API:
  - [Aplicación Node.js](create-cassandra-nodejs.md)
  - [Aplicación .NET](create-cassandra-dotnet.md)
  - [Aplicación Python](create-cassandra-python.md)

* Introducción a la [creación de una cuenta de Cassandra API, una base de datos y una tabla](create-cassandra-api-account-java.md) mediante una aplicación de Java.

* [Carga de datos de ejemplo en la tabla de Cassandra API](cassandra-api-load-data.md) mediante el uso de una aplicación de Java.

* [Consulta de datos de la cuenta de Cassandra API](cassandra-api-query-data.md) mediante el uso de una aplicación de Java.

* Para obtener más información sobre las características de Apache Cassandra admitidas por Cassandra API de Azure Cosmos DB, vea el artículo [Cassandra support](cassandra-support.md) (Compatibilidad de Cassandra).

* Lea las [Preguntas más frecuentes](cassandra-faq.yml).
