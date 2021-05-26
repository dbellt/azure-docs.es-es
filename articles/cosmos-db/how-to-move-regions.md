---
title: Traslado de una cuenta de Azure Cosmos DB a otra región
description: Obtenga información sobre cómo trasladar una cuenta de Azure Cosmos DB a otra región.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 05/13/2021
ms.author: mjbrown
ms.openlocfilehash: e8d39f6215e76c4cd9da9458f5e8a7af61622089
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110094841"
---
# <a name="move-an-azure-cosmos-db-account-to-another-region"></a>Traslado de una cuenta de Azure Cosmos DB a otra región
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

En este artículo se describe cómo:

- Trasladar una región en la que se replican los datos en Azure Cosmos DB.
- Migrar los metadatos y los datos de la cuenta (Azure Resource Manager) de una región a otra.

## <a name="move-data-from-one-region-to-another"></a>Traslado de datos de una región a otra

Azure Cosmos DB admite la replicación de datos de forma nativa, por lo que es sencillo trasladar los datos de una región a otra. Para hacer esto, puede usar Azure Portal, Azure PowerShell o la CLI de Azure. Es necesario seguir estos pasos:

1. Agregue una nueva región a la cuenta.

    Para agregar una nueva región a una cuenta de Azure Cosmos DB, vea [Incorporación o eliminación de regiones en una cuenta de Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account).

1. Realice una conmutación por error manual en la nueva región.

    Cuando la región que se va a quitar es actualmente la región de escritura de la cuenta, debe iniciar una conmutación por error en la nueva región agregada en el paso anterior. Se trata de una operación sin ningún tiempo de inactividad. Si va a mover una región de lectura en una cuenta de varias regiones, puede omitir este paso. 
    
    Para iniciar una conmutación por error, vea [Realización de una conmutación por error manual en una cuenta de Azure Cosmos](how-to-manage-database-account.md#manual-failover).

1. Quite la región original.

    Para quitar una región de una cuenta de Azure Cosmos DB, vea [Incorporación o eliminación de regiones en una cuenta de Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account).

## <a name="migrate-azure-cosmos-db-account-metadata"></a>Migración de metadatos de una cuenta de Azure Cosmos DB

Azure Cosmos DB no admite de forma nativa la migración de los metadatos de la cuenta de una región a otra. Para migrar los metadatos de la cuenta y los datos del cliente de una región a otra, debe crear una cuenta en la región deseada y, a continuación, copiar los datos manualmente. 

Una migración casi sin tiempo de inactividad para la API de SQL requiere el uso de la [fuente de cambios](change-feed.md) o una herramienta que la use. Si va a migrar la API de MongoDB, Cassandra API u otra API, o para obtener más información sobre las opciones para migrar datos entre cuentas, vea [Opciones para migrar los datos locales o en la nube a Azure Cosmos DB](cosmosdb-migrationchoices.md). 

En los pasos siguientes se muestra cómo migrar una cuenta de Azure Cosmos DB para la API de SQL y sus datos de una región a otra:

1. Cree una cuenta de Azure Cosmos DB en la región deseada.

    Para crear una cuenta con Azure Portal, PowerShell o la CLI de Azure, vea [Creación de una cuenta de Azure Cosmos DB](how-to-manage-database-account.md#create-an-account).

1. Cree una base de datos y un contenedor.

    Para crear una base de datos y un contenedor, vea [Creación de un contenedor de Azure Cosmos](how-to-create-container.md).

1. Migre los datos con la herramienta Live Data Migrator de Azure Cosmos DB.

    Para migrar datos casi sin tiempo de inactividad, vea [Herramienta Live Data Migrator de Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmosdb-live-data-migrator).

1. Actualice la cadena de conexión de la aplicación.

    Con la herramienta Live Data Migrator todavía en ejecución, actualice la información de conexión en la nueva implementación de la aplicación. Puede recuperar los puntos de conexión y las claves de la aplicación desde Azure Portal.

    :::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-portal.png" alt-text="Control de acceso en Azure Portal: demostración de la seguridad de bases de datos NoSQL":::

1. Redirija las solicitudes a la nueva aplicación.

    Una vez que la nueva aplicación se conecta a Azure Cosmos DB, puede redirigir las solicitudes de cliente a la nueva implementación.

1. Elimine los recursos que ya no necesita.

    Con las solicitudes ahora redirigidas por completo a la nueva instancia, puede eliminar la cuenta de Azure Cosmos DB anterior y la herramienta Live Data Migrator.

## <a name="next-steps"></a>Pasos siguientes

Para más información y ejemplos sobre cómo administrar la cuenta de Azure Cosmos, así como las bases de datos y los contenedores, lea estos artículos:

* [Administración de una cuenta de Azure Cosmos](how-to-manage-database-account.md)
* [Fuente de cambios en Azure Cosmos DB](change-feed.md)
