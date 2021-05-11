---
title: Introducción a Azure Cosmos DB API para MongoDB
description: Vea cómo puede usar Azure Cosmos DB para almacenar y consultar grandes cantidades de datos mediante la API de Azure Cosmos DB para MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 04/22/2021
author: gahl-levy
ms.author: gahllevy
ms.openlocfilehash: ead8bf6620bbe53af6c28870fa94b7a16490fcb1
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108202792"
---
# <a name="azure-cosmos-db-api-for-mongodb"></a>Azure Cosmos DB API para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Azure Cosmos DB API para MongoDB facilita el uso de Cosmos DB como si fuera una base de datos de MongoDB. Puede aprovechar su experiencia en MongoDB y seguir usando sus controladores, SDK y herramientas favoritos de MongoDB apuntando la aplicación a la cadena de conexión de la cuenta de la API para MongoDB.

## <a name="why-choose-the-api-for-mongodb"></a>Motivos para elegir la API para MongoDB

La API para MongoDB tiene numerosas ventajas adicionales por estar integrada en [Azure Cosmos DB](introduction.md) en comparación con las ofertas de servicio como MongoDB Atlas:

* **Escalabilidad instantánea**: al habilitar la característica de [escalado automático](provision-throughput-autoscale.md), la base de datos se puede escalar o reducir verticalmente sin período de calentamiento.
* **Particionamiento automático y transparente:** la API para MongoDB administra toda la infraestructura automáticamente. Esto incluye el particionamiento y el número de particiones, a diferencia de otras ofertas de MongoDB, como MongoDB Atlas, que requieren que especifique y administre el particionamiento para escalar horizontalmente. Esto le proporciona más tiempo para centrarse en el desarrollo de aplicaciones para los usuarios.
* **Hasta cinco nueves de disponibilidad**: se puede configurar fácilmente una [disponibilidad del 99,999 %](high-availability.md) para asegurarse de que los datos siempre estén ahí.  
* **Escalabilidad rentable, granular e ilimitada**: las colecciones particionadas se pueden escalar a cualquier tamaño, a diferencia de otras ofertas de servicio de MongoDB. Los usuarios de la API para MongoDB están ejecutando bases de datos con más de 600 TB de almacenamiento en la actualidad. El escalado se realiza de forma rentable, ya que, a diferencia de otras ofertas de servicio de MongoDB, la plataforma Cosmos DB puede escalar en incrementos de un tamaño mínimo equivalente a una centésima parte de una máquina virtual debido a las economías de escalado y gobernanza de recursos.
* **Implementaciones sin servidor:** a diferencia de MongoDB Atlas, la API para MongoDB es una base de datos nativa en la nube que ofrece un [modo de capacidad sin servidor](serverless.md). Con la opción [sin servidor](serverless.md), solo se le cobra por operación y no paga por la base de datos cuando no la usa.
* **Nivel Gratis**: con el nivel Gratis de Azure Cosmos DB, obtendrá en la cuenta las primeras 400 RU/s y 5 GB de almacenamiento gratis para siempre, que se aplicarán en el nivel de cuenta.
* **Las actualizaciones tardan segundos**: todas las versiones de la API se encuentran dentro de un código base, lo que hace que los cambios de versión sean tan sencillos como [cambiar un conmutador](mongodb-version-upgrade.md), sin tiempo de inactividad.
* **Análisis en tiempo real (HTAP) a cualquier escala**: la API de MongoDB ofrece la posibilidad de ejecutar consultas analíticas complejas para casos de uso como la inteligencia empresarial en los datos de la base de datos en tiempo real sin afectar a la base de datos. Esto es rápido y económico, debido al almacén de columnas analítico nativo en la nube que se está empleando, sin canalizaciones de extracción, transformación y carga de datos (ETL). Más información sobre [Azure Synapse Link](synapse-link.md).

> [!NOTE]
> [Puede usar Azure Cosmos DB API para MongoDB de forma gratuita con el nivel gratis.](how-pricing-works.md) Con el nivel Gratis de Azure Cosmos DB, obtendrá en la cuenta las primeras 400 RU/s y 5 GB de almacenamiento gratis, que se aplicarán en el nivel de cuenta.


## <a name="how-the-api-works"></a>Cómo funciona la API

Azure Cosmos DB API para MongoDB implementa el protocolo de conexión para MongoDB. Esta implementación permite una compatibilidad transparente con las herramientas, los controladores y los SDK de cliente nativos de MongoDB. Azure Cosmos DB no hospeda el motor de base de datos de MongoDB. Cualquier controlador de cliente de MongoDB compatible con la versión de API que use debe poder conectarse, sin ninguna configuración especial.

Compatibilidad de características de MongoDB:

Azure Cosmos DB API para MongoDB es compatible con las siguientes versiones del servidor de MongoDB:
- [Versión 4.0](mongodb-feature-support-40.md)
- [Versión 3.6](mongodb-feature-support-36.md)
- [Versión 3.2](mongodb-feature-support.md)

Todas las versiones de la API para MongoDB se ejecutan en el mismo código base, lo que hace que las actualizaciones sean una tarea sencilla que se puede completar en segundos sin tiempo de inactividad. Azure Cosmos DB simplemente cambia algunas marcas de características para pasar de una versión a otra.  Las marcas de características también habilitan la compatibilidad continua con versiones anteriores de la API, como la versión 3.2 y la versión 3.6. Puede elegir la versión de servidor que más le convenga.

:::image type="content" source="./media/mongodb-introduction/cosmosdb-mongodb.png" alt-text="API de Azure Cosmos DB para MongoDB" border="false":::

## <a name="what-you-need-to-know-to-get-started"></a>Lo que necesita saber para comenzar

* No se le facturará por las máquinas virtuales de un clúster. Los [precios](how-pricing-works.md) se basan en el rendimiento expresado en unidades de solicitud (RU) configuradas por cada base de datos o por cada colección. Las primeras 400 RU por segundo son gratuitas con el [nivel Gratis](how-pricing-works.md).

* Hay tres maneras de implementar Azure Cosmos DB API para MongoDB:
     * [Rendimiento aprovisionado](set-throughput.md): establezca un número de RU/s y cámbielo manualmente. Este modelo se adapta mejor a las cargas de trabajo coherentes.
     * [Escalado automático](provision-throughput-autoscale.md): establezca un límite superior en el rendimiento que necesita. El rendimiento se escala al instante para satisfacer sus necesidades. Este modelo se adapta mejor a las cargas de trabajo que cambian con frecuencia y optimiza los costos.
     * [Sin servidor](serverless.md) (versión preliminar): pague solo por el rendimiento que use, punto. Este modelo se adapta mejor a las cargas de trabajo de desarrollo y pruebas. 

* El rendimiento del clúster particionado depende de la clave de partición que elija al crear una colección. Elija cuidadosamente una clave de partición para asegurarse de que los datos se distribuyan uniformemente entre las particiones.

## <a name="quickstart"></a>Inicio rápido

* [Migración de una aplicación web Node.js de MongoDB existente](create-mongodb-nodejs.md)
* [Compilación de una aplicación web mediante la API de Azure Cosmos DB para MongoDB y el SDK de .NET](create-mongodb-dotnet.md)
* [Compilación de una aplicación de consola mediante la API de Azure Cosmos DB para MongoDB y el SDK de Java](create-mongodb-java.md)

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a obtener la información de cadena de conexión de la cuenta, siga el tutorial de [conexión de una aplicación de MongoDB a Azure Cosmos DB](connect-mongodb-account.md).
* Para aprender a crear una conexión entre la base de datos de Azure Cosmos DB y la aplicación de MongoDB en Studio 3T, siga el tutorial de [uso de Studio 3T con Azure Cosmos DB](mongodb-mongochef.md).
* Para importar los datos a una base de datos de Cosmos, siga el tutorial de [importación de datos de MongoDB en Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json).
* Conéctese a una cuenta de Cosmos con [Robo 3T](mongodb-robomongo.md).
* Aprenda a [configurar las preferencias de lectura para las aplicaciones distribuidas globalmente](../cosmos-db/tutorial-global-distribution-mongodb.md).
* Busque las soluciones a los errores más comunes en nuestra [Guía de solución de problemas](mongodb-troubleshoot.md).


<sup>Nota: En este artículo se describe una característica de Azure Cosmos DB que proporciona compatibilidad del protocolo de conexión con bases de datos de MongoDB. Microsoft no ejecuta bases de datos de MongoDB que ofrezcan este servicio. Azure Cosmos DB no está afiliado a MongoDB, Inc.</sup>