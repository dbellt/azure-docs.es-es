---
title: 'Inicio rápido: Cassandra API con Node.js: Azure Cosmos DB'
description: Esta guía de inicio rápido muestra cómo usar la API de Cassandra de Azure Cosmos DB para crear una aplicación de perfil con Node.js
author: TheovanKraay
ms.author: thvankra
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 02/10/2021
ms.custom: devx-track-js
ms.openlocfilehash: deb3fa27f0efe1a48e9ac472cff6d769219d4b4c
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112235750"
---
# <a name="quickstart-build-a-cassandra-app-with-nodejs-sdk-and-azure-cosmos-db"></a>Inicio rápido: Creación de una aplicación de Cassandra con Node.js SDK y Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
>  

En este inicio rápido se crea una cuenta de Cassandra API de Azure Cosmos DB y se utiliza una aplicación Node.js de Cassandra clonada desde GitHub para crear un contenedor y una base de datos de Cassandra. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] También puede [probar gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin suscripción de Azure, sin cargos y sin compromiso.

Además, necesita:
* [Node.js](https://nodejs.org/dist/v0.10.29/x64/node-v0.10.29-x64.msi) versión v0.10.29 o superior
* [Git](https://git-scm.com/)

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Para poder crear una base de datos de documentos, debe crear una cuenta de Cassandra con Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a clonar una aplicación de la API de Cassandra desde GitHub, establecer la cadena de conexión y ejecutarla. Verá lo fácil que es trabajar con datos mediante programación. 

1. Abra un símbolo del sistema. Cree una carpeta nueva denominada `git-samples`. Después cierre el símbolo del sistema.

    ```bash
    md "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como Git Bash. Use el comando `cd` para cambiar a la nueva carpeta e instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-nodejs-getting-started.git
    ```

1. Instale las dependencias de Node.js con npm.

    ```bash
    npm install
    ```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si le interesa aprender cómo el código crea los recursos de base de datos, puede revisar los siguientes fragmentos de código. Los fragmentos de código se toman del archivo `uprofile.js` de la carpeta `C:\git-samples\azure-cosmos-db-cassandra-nodejs-getting-started`. En caso contrario, puede ir directamente a [Actualización de la cadena de conexión](#update-your-connection-string). 

* Se establecieron los valores de nombre de usuario y contraseña utilizando la página de la cadena de conexión en el Azure Portal. 

   ```javascript
    let authProvider = new cassandra.auth.PlainTextAuthProvider(
        config.username,
        config.password
    );
   ```

* El objeto `client` se inicializa con la información de contactPoint. contactPoint se recupera de Azure Portal.

    ```javascript
    let client = new cassandra.Client({
        contactPoints: [`${config.contactPoint}:10350`],
        authProvider: authProvider,
        localDataCenter: config.localDataCenter,
        sslOptions: {
            secureProtocol: "TLSv1_2_method"
        },
    });
    ```

* El objeto `client` se conecta a la API de Cassandra de Azure Cosmos DB.

    ```javascript
    client.connect();
    ```

* Se crea un espacio de claves.

    ```javascript
    var query =
        `CREATE KEYSPACE IF NOT EXISTS ${config.keySpace} WITH replication = {'class': 'NetworkTopologyStrategy', 'datacenter' : '1' }`;
    await client.execute(query);
  }
    ```

* Se crea una tabla.

   ```javascript
    query =
        `CREATE TABLE IF NOT EXISTS ${config.keySpace}.user (user_id int PRIMARY KEY, user_name text, user_bcity text)`;
    await client.execute(query);
   },
   ```

* Se insertan entidades de clave/valor.

    ```javascript
    const arr = [
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (1, 'AdrianaS', 'Seattle')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (2, 'JiriK', 'Toronto')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (3, 'IvanH', 'Mumbai')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (4, 'IvanH', 'Seattle')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (5, 'IvanaV', 'Belgaum')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (6, 'LiliyaB', 'Seattle')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (7, 'JindrichH', 'Buenos Aires')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (8, 'AdrianaS', 'Seattle')`,
        `INSERT INTO  ${config.keySpace}.user (user_id, user_name , user_bcity) VALUES (9, 'JozefM', 'Seattle')`,
    ];
    for (const element of arr) {
        await client.execute(element);
    }
    ```

* Realice una consulta para obtener todos los valores de clave.

    ```javascript
    query = `SELECT * FROM ${config.keySpace}.user`;
    const resultSelect = await client.execute(query);

    for (const row of resultSelect.rows) {
        console.log(
            "Obtained row: %d | %s | %s ",
            row.user_id,
            row.user_name,
            row.user_bcity
        );
    }
    ```  

* Realice una consulta para obtener un valor de clave.

    ```javascript
    query = `SELECT * FROM ${config.keySpace}.user where user_id=1`;
    const resultSelectWhere = await client.execute(query);

    for (const row of resultSelectWhere.rows) {
        console.log(
            "Obtained row: %d | %s | %s ",
            row.user_id,
            row.user_name,
            row.user_bcity
        );
    }
    ```  

* Cierre la conexión. 

    ```javascript
    client.shutdown();
    ```  

## <a name="update-your-connection-string"></a>Actualización de la cadena de conexión

Ahora vuelva a Azure Portal para obtener la información de la cadena de conexión y cópiela en la aplicación. La cadena de conexión permite a la aplicación comunicarse con la base de datos hospedada.

1. En la cuenta de Azure Cosmos DB, en [Azure Portal](https://portal.azure.com/), seleccione **Cadena de conexión**. 

1. Use el botón :::image type="icon" source="./media/create-cassandra-nodejs/copy.png"::: del lado derecho de la pantalla para copiar el valor superior, CONTACT POINT (Punto de contacto).

    :::image type="content" source="./media/create-cassandra-nodejs/keys.png" alt-text="Visualización y copia de los valores de PUNTO DE CONTACTO, NOMBRE DE USUARIO y CONTRASEÑA de la página de la cadena de conexión de Azure Portal":::

1. Abra el archivo `config.js` . 

1. Pegue el valor de CONTACT POINT del portal a través de `'CONTACT-POINT` en la línea 9.

    La línea 9 debe ser similar a la siguiente: 

    `contactPoint: "cosmos-db-quickstarts.cassandra.cosmosdb.azure.com",`

1. Copie el valor de NOMBRE DE USUARIO del portal y péguelo en `<FillMEIN>` en la línea 2.

    La línea 2 ahora debe ser similar a 

    `username: 'cosmos-db-quickstart',`

1. Copie el valor de CONTRASEÑA del portal y péguelo en `USERNAME` en la línea 8.

    La línea 8 ahora debe ser similar a

    `password: '2Ggkr662ifxz2Mg==',`

1. Reemplace REGION por la región de Azure en la que creó este recurso.

1. Guarde el archivo `config.js`.


## <a name="run-the-nodejs-app"></a>Ejecute la aplicación Node.js

1. En la ventana de terminal, asegúrese de que se encuentra en el directorio de ejemplo que clonó anteriormente:

    ```bash
    cd azure-cosmos-db-cassandra-nodejs-getting-started
    ```

1. Ejecute la aplicación del nodo:

    ```bash
    npm start
    ```

4. Compruebe los resultados previstos desde la línea de comandos.

    :::image type="content" source="./media/create-cassandra-nodejs/output.png" alt-text="Visualizar y comprobar el resultado":::

    Presione CTRL+C para detener la ejecución del programa y cerrar la ventana de la consola. 

5. En Azure Portal abra **Explorador de datos** para consultar, modificar y trabajar con estos nuevos datos. 

    :::image type="content" source="./media/create-cassandra-nodejs/data-explorer.png" alt-text="Visualizar los datos en el Explorador de datos"::: 

## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a crear una cuenta de Azure Cosmos DB con Cassandra API y a ejecutar una aplicación Node.js de Cassandra que cree un contenedor y una base de datos de Cassandra. Ya puede importar datos adicionales en la cuenta de Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importación de datos de Cassandra en Azure Cosmos DB](cassandra-import-data.md)