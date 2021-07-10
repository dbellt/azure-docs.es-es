---
title: 'Inicio rápido: Uso de Spring Data Azure Cosmos DB v3 para crear una base de datos de documentos mediante Azure Cosmos DB'
description: En este inicio rápido se presenta un ejemplo de código de Spring Data Azure Cosmos DB v3 que se puede usar para conectarse a la SQL API de Azure Cosmos DB y realizar consultas en ellas
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: quickstart
ms.date: 03/07/2021
ms.author: anfeldma
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 9241c7a95641a938b494cba5e1d661a77d0eb98a
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112235138"
---
# <a name="quickstart-build-a-spring-data-azure-cosmos-db-v3-app-to-manage-azure-cosmos-db-sql-api-data"></a>Inicio rápido: Creación de una aplicación de Spring Data Azure Cosmos DB v3 para administrar los datos de SQL API de Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [SDK para Java v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> * [Conector de Spark v3](create-sql-api-spark.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

En este inicio rápido se crea y administra una cuenta de Azure Cosmos DB para SQL API desde Azure Portal, y se usa una aplicación de Spring Data Azure Cosmos DB v3 clonada desde GitHub. En primer lugar, se crea una cuenta de SQL API de Azure Cosmos DB desde Azure Portal, luego, se crea una aplicación de Spring Boot mediante el conector de Spring Data Azure Cosmos v3 y, después, se agregan recursos a la cuenta de Cosmos DB mediante la aplicación de Spring Boot. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

> [!IMPORTANT]  
> Estas notas de la versión corresponden a la versión 3 de Spring Data Azure Cosmos DB. Puede encontrar las [notas de la versión 2 aquí](sql-api-sdk-java-spring-v2.md). 
>
> Spring Data Azure Cosmos DB solo admite la API de SQL.
>
> Consulte estos artículos para obtener información sobre Spring Data en otras API de Azure Cosmos DB:
> * [Spring Data para Apache Cassandra con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB con Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
>

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure. También puede usar el [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) con el identificador URI `https://localhost:8081` y la clave `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`.
- [Java Development Kit (JDK) 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Apunte su variable de entorno `JAVA_HOME` a la carpeta donde está instalado el JDK.
- Un [archivo binario de Maven](https://maven.apache.org/download.cgi). En Ubuntu, ejecute `apt-get install maven` para instalar Maven.
- [Git](https://www.git-scm.com/downloads). En Ubuntu, ejecute `sudo apt-get install git` para instalar Git.

## <a name="introductory-notes"></a>Notas de introducción

*Estructura de una cuenta de Cosmos DB*: independientemente de la API o el lenguaje de programación, una *cuenta* de Cosmos DB contiene de cero o varias *bases de datos*; una *base de datos* contiene de cero a varios *contenedores* y un *contenedor* contiene de cero a varios elementos, como se muestra en el diagrama siguiente:

:::image type="content" source="./media/account-databases-containers-items/cosmos-entities.png" alt-text="Entidades de cuenta de Azure Cosmos" border="false":::

Puede leer más información sobre las bases de datos, los contenedores y los elementos [aquí](account-databases-containers-items.md). Algunas propiedades importantes se definen en el nivel del contenedor, entre ellas la *capacidad de proceso aprovisionada* y la *clave de partición*. 

La capacidad de proceso aprovisionada se mide en unidades de solicitud (*RU*) que tienen un precio monetario y son un factor determinante crucial en el costo operativo de la cuenta. La capacidad de proceso aprovisionada se puede seleccionar en una granularidad por contenedor o por base de datos; sin embargo, se suele preferir la especificación de capacidad de proceso a nivel de contenedor. Para más información sobre el aprovisionamiento de capacidad de proceso [aquí](set-throughput.md).

A medida que los elementos se insertan en un contenedor de Cosmos DB, la base de datos crece horizontalmente al agregar más almacenamiento y procesos para controlar las solicitudes. La capacidad de almacenamiento y de proceso se agregan en unidades discontinuas conocidas como *particiones* y debe elegir un campo en los documentos para que sea la clave de partición para asignar cada documento a una partición. La forma de administración de las particiones es la asignación a cada partición de un segmento aproximadamente igual fuera del intervalo de valores de la clave de partición. Por lo tanto, se recomienda elegir una clave de partición relativamente aleatoria o distribuida uniformemente. De lo contrario, algunas particiones verán bastantes más solicitudes (*partición de uso frecuente*) mientras que otras verán muchas menos (*partición de uso esporádico*), comportamiento que se debe evitar. [Aquí](partitioning-overview.md) encontrará más información sobre la creación de particiones.

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Para poder crear una base de datos de documentos, debe crear una cuenta de SQL API con Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount](includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-container"></a>Agregar un contenedor

[!INCLUDE [cosmos-db-create-collection](includes/cosmos-db-create-collection.md)]

<a id="add-sample-data"></a>
## <a name="add-sample-data"></a>Adición de datos de ejemplo

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Consulta de los datos

[!INCLUDE [cosmos-db-create-sql-api-query-data](includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a empezar a trabajar con el código. Vamos a clonar una aplicación de SQL API desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación. 

Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

```bash
git clone https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started.git
```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Ejecutar la aplicación](#run-the-app). 

### <a name="application-configuration-file"></a>Archivo de configuración de la aplicación

Aquí se muestra la forma en que Spring Boot y Spring Data mejoran la experiencia del usuario (el proceso de establecer un cliente de Cosmos y de conectarse a los recursos de Cosmos es ahora la configuración en lugar del código). En el inicio de la aplicación, Spring Boot controla todo este texto reutilizable mediante la configuración de **application.properties**:

```xml
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

Una vez que crea una cuenta, una base de datos y un contenedor de Azure Cosmos DB, solo tiene que rellenar los espacios en blanco en el archivo de configuración y Spring Boot/Spring Data realizarán automáticamente las siguientes operaciones: (1) crear una instancia de `CosmosClient` de SDK de Java subyacente con el identificador URI y la clave, y (2) conectar a la base de datos y al contenedor. Ya está todo establecido: **no hace falta más código de administración de recursos**.

### <a name="java-source"></a>Origen de Java

El valor agregado de Spring Data también procede de su interfaz simple, limpia, estandarizada e independiente de la plataforma para trabajar en almacenes de datos. Basándose en el ejemplo de GitHub de Spring Data con el vínculo de arriba, a continuación hay ejemplos de CRUD y de consultas para manipular documentos de Azure Cosmos DB con Spring Data Azure Cosmos DB.

* Creación y actualizaciones de elementos mediante el método `save`.

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Create)]
   
* Lecturas puntuales mediante el método de consulta derivado definido en el repositorio. `findByIdAndLastName` realiza lecturas puntuales para `UserRepository`. Los campos mencionados en el nombre del método hacen que Spring Data ejecute una lectura puntual definida por los campos `id` y `lastName`:

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Read)]

* Eliminaciones de elementos mediante `deleteAll`:

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Delete)]

* Consulta derivada basada en el nombre de método del repositorio. Spring Data implementa el método `UserRepository` `findByFirstName` como una consulta SQL del SDK de Java en el campo `firstName` (esta consulta no se pudo implementar como una lectura puntual):

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Query)]

## <a name="run-the-app"></a>Ejecución la aplicación

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión e iniciar la aplicación con la información del punto de conexión. Esto permite que la aplicación se comunique con la base de datos hospedada.

1. En la ventana de terminal de GIT, `cd` a la carpeta del código de ejemplo.

    ```bash
    cd azure-spring-data-cosmos-java-sql-api-getting-started/azure-spring-data-cosmos-java-getting-started/
    ```

2. En la ventana del terminal de GIT, use el comando siguiente para instalar los paquetes de Spring Data Azure Cosmos DB necesarios.

    ```bash
    mvn clean package
    ```

3. En la ventana del terminal de GIT, use el comando siguiente para iniciar la aplicación de Spring Data Azure Cosmos DB:

    ```bash
    mvn spring-boot:run
    ```
    
4. La aplicación carga **application.properties** y conecta los recursos de la cuenta de Azure Cosmos DB.
5. La aplicación realizará las operaciones CRUD puntuales descritas anteriormente.
6. La aplicación realizará una consulta derivada.
7. La aplicación no elimina los recursos. Vuelva al portal para [limpiar los recursos](#clean-up-resources) de su cuenta si desea evitar los cargos en que se incurren.

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de SQL API de Azure Cosmos DB, a crear una base de datos de documentos y un contenedor mediante Data Explorer, y a ejecutar una aplicación de Spring Data para realizar lo mismo mediante programación. Ya puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos a Azure Cosmos DB](import-data.md)