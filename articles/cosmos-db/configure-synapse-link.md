---
title: Configuración y uso de Azure Synapse Link para Azure Cosmos DB (versión preliminar)
description: Aprenda a habilitar Synapse Link para las cuentas de Azure Cosmos DB, crear un contenedor con el almacén analítico habilitado, conectar la base de datos de Azure Cosmos a un área de trabajo de Synapse y ejecutar consultas.
author: Rodrigossz
ms.service: cosmos-db
ms.topic: how-to
ms.date: 11/30/2020
ms.author: rosouz
ms.custom: references_regions, synapse-cosmos-db
ms.openlocfilehash: 5cfe932d4f9ea60a044ce0b594df800fa37af6f1
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904935"
---
# <a name="configure-and-use-azure-synapse-link-for-azure-cosmos-db"></a>Configuración y uso de Azure Synapse Link para Azure Cosmos DB (versión preliminar)
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

[Azure Synapse Link para Azure Cosmos DB](synapse-link.md) es una funcionalidad de procesamiento analítico y transaccional híbrido (HTAP) nativo de nube que permite ejecutar análisis casi en tiempo real de datos operativos en Azure Cosmos DB. Synapse Link crea una integración perfecta y sin contratiempos entre Azure Cosmos DB y Azure Synapse Analytics.

Azure Synapse Link está disponible para contenedores de API de SQL de Azure Cosmos DB para colecciones de Mongo DB. Siga estos pasos para ejecutar consultas analíticas con Azure Synapse Link para Azure Cosmos DB:

* [Habilitación de Synapse Link para las cuentas de Azure Cosmos DB](#enable-synapse-link)
* [Creación de un contenedor de Azure Cosmos DB habilitado para el almacén analítico](#create-analytical-ttl)
* [Opcional: actualización del TTL del almacén analítico para un contenedor de Azure Cosmos DB](#update-analytical-ttl)
* [Conexión de la base de datos de Azure Cosmos DB a un área de trabajo de Synapse](#connect-to-cosmos-database)
* [Consulta del almacén analítico mediante Spark en Synapse](#query-analytical-store-spark)
* [Consulta del almacén analítico mediante un grupo de SQL sin servidor](#query-analytical-store-sql-on-demand)
* [Uso de un grupo de SQL sin servidor para analizar datos en Power BI y visualizarlos](#analyze-with-powerbi)

## <a name="enable-azure-synapse-link-for-azure-cosmos-db-accounts"></a><a id="enable-synapse-link"></a>Habilitación de Azure Synapse Link para cuentas de Azure Cosmos DB

### <a name="azure-portal"></a>Azure portal

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).

1. [Cree una cuenta de Azure](create-sql-api-dotnet.md#create-account), o bien seleccione una cuenta de Azure Cosmos DB ya existente.

1. Vaya a la cuenta de Azure Cosmos DB y abra el panel **Características**.

1. Seleccione **Synapse Link** en la lista de características.

   :::image type="content" source="./media/configure-synapse-link/find-synapse-link-feature.png" alt-text="Búsqueda de la característica Synapse Link":::

1. A continuación, se le pedirá que habilite Synapse Link en su cuenta. Seleccione **Habilitar**. Este proceso puede tardar de 1 a 5 minutos en completarse.

   :::image type="content" source="./media/configure-synapse-link/enable-synapse-link-feature.png" alt-text="Habilitación de la característica Synapse Link":::

1. Ahora su cuenta está habilitada para usar Synapse Link. A continuación, consulte cómo crear contenedores habilitados para el almacén analítico para iniciar de forma automática la replicación de sus datos operativos desde el almacén transaccional hacia el almacén analítico.

> [!NOTE]
> La activación de Synapse Link no activa automáticamente el almacén analítico. Una vez habilitado Synapse Link en la cuenta de Cosmos DB, habilite el almacén analítico en contenedores al crearlos para empezar a replicar los datos de la operación en este almacén analítico. 

### <a name="azure-cli"></a>Azure CLI

En los vínculos siguientes se muestra cómo habilitar Synapse Link mediante la CLI de Azure:

* [Creación de una nueva cuenta de Azure Cosmos DB con Synapse Link habilitado](/cli/azure/cosmosdb#az_cosmosdb_create-optional-parameters)
* [Actualización de una cuenta de Azure Cosmos DB existente para habilitar Synapse Link](/cli/azure/cosmosdb#az_cosmosdb_update-optional-parameters)

### <a name="powershell"></a>PowerShell

* [Creación de una nueva cuenta de Azure Cosmos DB con Synapse Link habilitado](/powershell/module/az.cosmosdb/new-azcosmosdbaccount#description)
* [Actualización de una cuenta de Azure Cosmos DB existente para habilitar Synapse Link](/powershell/module/az.cosmosdb/update-azcosmosdbaccount)


En los vínculos siguientes se muestra cómo habilitar Synapse Link mediante PowerShell:

## <a name="create-an-azure-cosmos-container-with-analytical-store"></a><a id="create-analytical-ttl"></a>Creación de un contenedor de Azure Cosmos con un almacén analítico

Puede activar el almacén analítico en un contenedor de Azure Cosmos durante su creación. Puede usar Azure Portal o configurar la propiedad `analyticalTTL` durante la creación del contenedor mediante los SDK de Azure Cosmos DB.

> [!NOTE]
> En este momento, puede habilitar el almacén analítico para los **nuevos** contenedores (tanto en cuentas nuevas como ya existentes). Se pueden migrar los datos de los contenedores existentes a los nuevos mediante las [herramientas de migración de Azure Cosmos DB](cosmosdb-migrationchoices.md).

### <a name="azure-portal"></a>Azure portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) o en el [Explorador de Azure Cosmos DB](https://cosmos.azure.com/).

1. Vaya a la cuenta de Azure Cosmos DB y abra la pestaña **Explorador de datos**.

1. Seleccione **Nuevo contenedor** y escriba un nombre para la base de datos, el contenedor, la clave de partición y los detalles de capacidad de proceso. Active la opción **Analytical store** (Almacén analítico). Después de habilitar el almacén analítico, se crea un contenedor con la propiedad `AnalyicalTTL` establecida en el valor predeterminado de -1 (retención de datos infinita). Este almacén analítico conserva todas las versiones históricas de los registros.

   :::image type="content" source="./media/configure-synapse-link/create-container-analytical-store.png" alt-text="Activación de un almacén analítico para un contenedor de Azure Cosmos":::

1. Si no ha habilitado Synapse Link anteriormente en esta cuenta, se le pedirá que lo haga, ya que se trata de un requisito previo para crear un contenedor habilitado para almacén analítico. Si se le solicita, seleccione **Enable Synapse Link** (Habilitar Synapse Link). Este proceso puede tardar de 1 a 5 minutos en completarse.

1. Seleccione **Aceptar** para crear un contenedor de Azure Cosmos habilitado para el almacén analítico.

1. Una vez creado el contenedor, compruebe que se ha habilitado el almacén analítico; para ello, haga clic en **Configuración**, justo debajo de Documentos en Data Explorer y compruebe si está activada la opción **Período de vida del almacén analítico**.

### <a name="net-sdk"></a>.NET SDK

En el código siguiente se crea un contenedor con un almacén analítico mediante el SDK de .NET. Establezca la propiedad de TTL analítico en el valor requerido. Para ver la lista de valores permitidos, consulte el artículo [Valores de TTL analítico admitidos](analytical-store-introduction.md#analytical-ttl):

```csharp
// Create a container with a partition key, and analytical TTL configured to -1 (infinite retention)
ContainerProperties properties = new ContainerProperties()
{
    Id = "myContainerId",
    PartitionKeyPath = "/id",
    AnalyticalStoreTimeToLiveInSeconds = -1,
};
CosmosClient cosmosClient = new CosmosClient("myConnectionString");
await cosmosClient.GetDatabase("myDatabase").CreateContainerAsync(properties);
```

### <a name="java-v4-sdk"></a>SDK de Java V4

En el código siguiente se crea un contenedor con un almacén analítico mediante el SDK de Java V4. Establezca la propiedad `AnalyticalStoreTimeToLiveInSeconds` en el valor requerido:

```java
// Create a container with a partition key and  analytical TTL configured to  -1 (infinite retention) 
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");

containerProperties.setAnalyticalStoreTimeToLiveInSeconds(-1);

container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

### <a name="python-v4-sdk"></a>SDK para Python v4

El SDK para Python 2.7 y el SDK de Azure Cosmos DB 4.1.0 son las versiones mínimas necesarias, y el SDK solo es compatible con la API de SQL.

El primer paso consiste en asegurarse de que está usando como mínimo la versión 4.1.0 del SDK de [Azure Cosmos DB para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos):

```python
import azure.cosmos as cosmos

print (cosmos.__version__)
```
En el siguiente paso se crea un contenedor con un almacén analítico mediante el SDK de Azure Cosmos DB para Python:

```python
# Azure Cosmos DB Python SDK, for SQL API only.
# Creating an analytical store enabled container.

import azure.cosmos.cosmos_client as cosmos_client
import azure.cosmos.exceptions as exceptions
from azure.cosmos.partition_key import PartitionKey

HOST = 'your-cosmos-db-account-URI'
KEY = 'your-cosmos-db-account-key'
DATABASE = 'your-cosmos-db-database-name'
CONTAINER = 'your-cosmos-db-container-name'

client = cosmos_client.CosmosClient(HOST,  KEY )
# setup database for this sample. 
# If doesn't exist, creates a new one with the name informed above.
try:
    db = client.create_database(DATABASE)

except exceptions.CosmosResourceExistsError:
    db = client.get_database_client(DATABASE)

# Creating the container with analytical store enabled, using the name informed above.
# If a container with the same name exists, an error is returned.
#
# The 3 options for the analytical_storage_ttl parameter are:
# 1) 0 or Null or not informed (Not enabled).
# 2) -1 (The data will be stored in analytical store infinitely).
# 3) Any other number is the actual ttl, in seconds.

try:
    container = db.create_container(
        id=CONTAINER,
        partition_key=PartitionKey(path='/id', kind='Hash'),analytical_storage_ttl=-1
    )
    properties = container.read()
    print('Container with id \'{0}\' created'.format(container.id))
    print('Partition Key - \'{0}\''.format(properties['partitionKey']))

except exceptions.CosmosResourceExistsError:
    print('A container with already exists')
```

### <a name="azure-cli"></a>Azure CLI

En los vínculos siguientes se muestra cómo crear un contenedor habilitado para el almacén analítico mediante la CLI de Azure:

* [API de Azure Cosmos DB para Mongo DB](/cli/azure/cosmosdb/mongodb/collection#az_cosmosdb_mongodb_collection_create-examples)
* [API de SQL de Azure Cosmos DB](/cli/azure/cosmosdb/sql/container#az_cosmosdb_sql_container_create)

### <a name="powershell"></a>PowerShell

En los vínculos siguientes se muestra cómo crear un contenedor habilitado para el almacén analítico mediante PowerShell:

* [API de Azure Cosmos DB para Mongo DB](/powershell/module/az.cosmosdb/new-azcosmosdbmongodbcollection#description)
* [API de SQL de Azure Cosmos DB](/cli/azure/cosmosdb/sql/container#az_cosmosdb_sql_container_create)


## <a name="optional---update-the-analytical-store-time-to-live"></a><a id="update-analytical-ttl"></a> Opcional: actualización del período de vida del almacén analítico

Después de habilitar el almacén analítico con un valor de TTL determinado, tal vez quiera actualizarlo a un valor válido diferente más adelante. Puede actualizar el valor desde Azure Portal, mediante la CLI de Azure, desde PowerShell o con los SDK de Cosmos DB. Para obtener información sobre las distintas opciones de configuración de TTL analítico, consulte el artículo [valores de TTL analítico admitidos](analytical-store-introduction.md#analytical-ttl).


### <a name="azure-portal"></a>Azure Portal

Si creó un contenedor habilitado para el almacén analítico mediante Azure Portal, dicho contenedor contiene un TTL analítico predeterminado de -1. Siga estos pasos para actualizar dicho valor:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) o en el [Explorador de Azure Cosmos DB](https://cosmos.azure.com/).

1. Vaya a la cuenta de Azure Cosmos DB y abra la pestaña **Explorador de datos**.

1. Seleccione un contenedor existente que tenga habilitado el almacén analítico. Expanda el contenedor y modifique los valores siguientes:

  * Abra la ventana **Escala y configuración**.
  * En **Configuración**, busque **Período de vida de almacenamiento analítico**.
  * Seleccione **Activado (valor no predeterminado)** o seleccione **Activado** y establezca un valor para el período de vida
  * Haga clic en **Guardar** para guardar los cambios.

### <a name="net-sdk"></a>.NET SDK

En el código siguiente se muestra cómo actualizar el TTL para el almacén analítico mediante el SDK de .NET:

```csharp
// Get the container, update AnalyticalStorageTimeToLiveInSeconds 
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Update analytical store TTL
containerResponse.Resource. AnalyticalStorageTimeToLiveInSeconds = 60 * 60 * 24 * 180  // Expire analytical store data in 6 months;
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

### <a name="java-v4-sdk"></a>SDK de Java V4

En el código siguiente se muestra cómo actualizar el TTL para el almacén analítico mediante el SDK de Java V4:

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");

// Update analytical store TTL to expire analytical store data in 6 months;
containerProperties.setAnalyticalStoreTimeToLiveInSeconds (60 * 60 * 24 * 180 );  
 
// Update container settings
container.replace(containerProperties).block();
```

### <a name="python-v4-sdk"></a>SDK para Python v4

Actualmente no se admite.


### <a name="azure-cli"></a>Azure CLI

Los vínculos siguientes muestran cómo actualizar el TTL analítico de los contenedores mediante la CLI de Azure:

* [API de Azure Cosmos DB para Mongo DB](/cli/azure/cosmosdb/mongodb/collection#az_cosmosdb_mongodb_collection_update)
* [API de SQL de Azure Cosmos DB](/cli/azure/cosmosdb/sql/container#az_cosmosdb_sql_container_update)

### <a name="powershell"></a>PowerShell

Los vínculos siguientes muestran cómo actualizar el TTL analítico de los contenedores mediante PowerShell:

* [API de Azure Cosmos DB para Mongo DB](/powershell/module/az.cosmosdb/update-azcosmosdbmongodbcollection)
* [API de SQL de Azure Cosmos DB](/powershell/module/az.cosmosdb/update-azcosmosdbsqlcontainer)


## <a name="connect-to-a-synapse-workspace"></a><a id="connect-to-cosmos-database"></a> Conexión a un área de trabajo de Synapse

Siga las instrucciones del artículo [Conexión a Azure Synapse Link](../synapse-analytics/synapse-link/how-to-connect-synapse-link-cosmos-db.md) sobre cómo acceder a una base de datos de Azure Cosmos DB desde Azure Synapse Analytics Studio con Azure Synapse Link.

## <a name="query-analytical-store-using-apache-spark-for-azure-synapse-analytics"></a><a id="query-analytical-store-spark"></a> Consulta del almacén analítico mediante Apache Spark para Azure Synapse Analytics

Siga las instrucciones que se indican en el artículo [Consulta al almacén analítico de Azure Cosmos DB](../synapse-analytics/synapse-link/how-to-query-analytical-store-spark.md) sobre cómo realizar consultas con Spark en Synapse. En este artículo se ofrecen algunos ejemplos sobre cómo puede interactuar con el almacén analítico desde los gestos de Synapse. Dichos gestos se muestran cuando se hace clic con el botón secundario en un contenedor. Con ellos, puede generar código rápidamente y adaptarlo a sus necesidades. También son ideales para detectar datos con un solo clic.

## <a name="query-the-analytical-store-using-serverless-sql-pool-in-azure-synapse-analytics"></a><a id="query-analytical-store-sql-on-demand"></a> Consultar el almacén analítico mediante un grupo de SQL sin servidor en Azure Synapse Analytics

El grupo de SQL sin servidor permite consultar y analizar los datos de los contenedores de Azure Cosmos DB que están habilitados con Azure Synapse Link. Se pueden analizar los datos casi en tiempo real sin que afecte al rendimiento de las cargas de trabajo transaccionales. Ofrece una sintaxis T-SQL familiar para consultar los datos del almacén analítico y la conectividad integrada en una amplia gama de herramientas de consulta ad hoc y de BI a través de la interfaz de T-SQL. Para obtener más información, vea el artículo [Consulta del almacén analítico mediante un grupo de SQL sin servidor](../synapse-analytics/sql/query-cosmos-db-analytical-store.md).

## <a name="use-serverless-sql-pool-to-analyze-and-visualize-data-in-power-bi"></a><a id="analyze-with-powerbi"></a>Uso de un grupo de SQL sin servidor para analizar datos en Power BI y visualizarlos

Se puede compilar una base de datos de grupo de SQL sin servidor y vistas a través de Synapse Link para Azure Cosmos DB. Posteriormente se pueden consultar los contenedores de Azure Cosmos y, después, compilar un modelo con Power BI sobre esas vistas para reflejar la consulta. Para obtener más información, vea el artículo sobre cómo usar un [grupo de SQL sin servidor para analizar los datos de Azure Cosmos DB con Synapse Link](synapse-link-power-bi.md).

## <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure

La [plantilla de Azure Resource Manager](./manage-with-templates.md#azure-cosmos-account-with-analytical-store) crea una cuenta de Azure Cosmos DB habilitada para Synapse Link para la API de SQL. Esta plantilla crea una cuenta de Core (SQL) API en una región con un contenedor configurado con TTL analítico habilitado y una opción para usar la capacidad de proceso manual o de escalado automático. Para implementar esta plantilla, haga clic en **Implementar en Azure** en la página Léame.

## <a name="getting-started-with-azure-synapse-link---samples"></a><a id="cosmosdb-synapse-link-samples"></a> Introducción a Azure Synapse Link: ejemplos

Puede encontrar ejemplos para empezar a trabajar con Azure Synapse Link en [GitHub](https://aka.ms/cosmosdb-synapselink-samples). Estos presentan soluciones de un extremo a otro con escenarios de IoT y de venta minorista. También puede encontrar los ejemplos correspondientes a Azure Cosmos DB API para MongoDB en el mismo repositorio en la carpeta [MongoDB](https://github.com/Azure-Samples/Synapse/tree/main/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples). 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la siguiente documentación:

* [Azure Synapse Link para Azure Cosmos DB.](synapse-link.md)

* [Introducción al almacén analítico de Azure Cosmos DB.](analytical-store-introduction.md)

* [Preguntas frecuentes sobre Synapse Link para Azure Cosmos DB.](synapse-link-frequently-asked-questions.yml)

* [Apache Spark en Azure Synapse Analytics](../synapse-analytics/spark/apache-spark-concepts.md).

* [Compatibilidad del entorno de ejecución del grupo de SQL sin servidor con Azure Synapse Analytics](../synapse-analytics/sql/on-demand-workspace-overview.md).
