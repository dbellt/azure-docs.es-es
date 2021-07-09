---
title: Compilación de una aplicación web mediante la API de Azure Cosmos DB para MongoDB y el SDK de .NET
description: Se presenta un ejemplo de código de .NET que se puede usar para conectarse a la API de Azure Cosmos DB para MongoDB y realizar consultas.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/15/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 387455f170d64c78ba4e1aab700a91a81bef2ed0
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112235606"
---
# <a name="quickstart-build-a-net-web-app-using-azure-cosmos-dbs-api-for-mongodb"></a>Inicio rápido: Compilación de una aplicación web .NET mediante la API de Azure Cosmos DB para MongoDB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB es la base de datos NoSQL rápida de Microsoft con API abiertas para cualquier escala. Puede crear rápidamente bases de datos de documentos, clave-valor y grafos, y realizar consultas en ellas. Todas las bases de datos se beneficiarán de las funcionalidades de distribución global y escala horizontal en Cosmos DB. 

En este inicio rápido se muestra cómo crear una cuenta de Cosmos con la [API de Azure Cosmos DB para MongoDB](mongodb-introduction.md). Después, compilará e implementará una aplicación web de lista de tareas compilada mediante el [controlador .NET de MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/).

## <a name="prerequisites-to-run-the-sample-app"></a>Requisitos previos para ejecutar la aplicación de ejemplo

* [Visual Studio](https://www.visualstudio.com/downloads/)
* Una cuenta de Azure Cosmos DB.

Si no tiene Visual Studio, descargue [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) con la carga de trabajo **ASP.NET y desarrollo web** instalada con el programa de instalación.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

<a id="create-account"></a>
## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

[!INCLUDE [cosmos-db-create-dbaccount](includes/cosmos-db-create-dbaccount-mongodb.md)]

El ejemplo que se describe en este artículo es compatible con la versión 2.6.1 de MongoDB.Driver.

## <a name="clone-the-sample-app"></a>Clonación de la aplicación de ejemplo

Ejecute los siguientes comandos en una ventana de comandos habilitada para GitHub, como [Git Bash](https://git-scm.com/downloads):

```bash
mkdir "C:\git-samples"
cd "C:\git-samples"
git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started.git
```

Los comandos anteriores:

1. Cree el directorio *C:\git-samples* para el ejemplo. Elija una carpeta adecuada para el sistema operativo.
1. Cambie el directorio actual a la carpeta *C:\git-samples*.
1. Clone el ejemplo en la carpeta *C:\git-samples*.

Si no desea usar git, también puede [descargar el proyecto como un archivo ZIP](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started/archive/master.zip).

## <a name="review-the-code"></a>Revisión del código

1. En Visual Studio, haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y, después, haga clic en **Administrar paquetes NuGet**.
1. En el cuadro **Examinar** de NuGet, escriba *MongoDB.Driver*.
1. En los resultados, instale la biblioteca **MongoDB.Driver**. De este modo, se instalan el paquete de MongoDB.Driver y todas las dependencias.

Los pasos siguientes son opcionales. Si está interesado en aprender cómo se crean los recursos de base de datos en el código, revise los siguientes fragmentos de código. En caso contrario, vaya directamente a [Actualización de la cadena de conexión](#update-the-connection-string).

Los fragmentos de código siguientes son del archivo *DAL/Dal.cs*.

* El código siguiente inicializa el cliente:

    ```cs
        MongoClientSettings settings = new MongoClientSettings();
        settings.Server = new MongoServerAddress(host, 10255);
        settings.UseSsl = true;
        settings.SslSettings = new SslSettings();
        settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

        MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
        MongoIdentityEvidence evidence = new PasswordEvidence(password);

        settings.Credential = new MongoCredential("SCRAM-SHA-1", identity, evidence);

        MongoClient client = new MongoClient(settings);
    ```

* El código siguiente recupera la base de datos y la colección:

    ```cs
    private string dbName = "Tasks";
    private string collectionName = "TasksList";

    var database = client.GetDatabase(dbName);
    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
    ```

* El código siguiente recupera todos los documentos:

    ```cs
    collection.Find(new BsonDocument()).ToList();
    ```

El código siguiente crea una tarea y la inserta en la colección:

   ```csharp
    public void CreateTask(MyTask task)
    {
        var collection = GetTasksCollectionForEdit();
        try
        {
            collection.InsertOne(task);
        }
        catch (MongoCommandException ex)
        {
            string msg = ex.Message;
        }
    }
   ```
   De forma similar, puede actualizar y eliminar los documentos mediante los métodos [collection.UpdateOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.updateOne/index.html) y [collection.DeleteOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.deleteOne/index.html).

## <a name="update-the-connection-string"></a>Actualización de la cadena de conexión

En Azure Portal, copie la información de la cadena de conexión:

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Cosmos; en el panel de navegación izquierdo, haga clic en **Cadena de conexión** y, a continuación, en **Claves de lectura y escritura**. Deberá usar los botones de copia del lado derecho de la pantalla para copiar el nombre de usuario, la contraseña y el host en el archivo Dal.cs en el paso siguiente.

2. Abra el archivo *DAL/Dal.cs*.

3. Copie el valor del **nombre de usuario** del portal (con el botón de copia) y conviértalo en el valor del **nombre de usuario** en el archivo **Dal.cs**.

4. Copie el valor del **host** del portal y conviértalo en el valor del **host** en el archivo **Dal.cs**.

5. Copie el valor de la **contraseña** del portal y conviértalo en el valor de la **contraseña** en el archivo **Dal.cs**.

<!-- TODO Store PW correctly-->
> [!WARNING]
> Nunca compruebe las contraseñas u otros datos confidenciales en el código fuente.

Ya ha actualizado la aplicación con toda la información que necesita para comunicarse con Cosmos DB.

## <a name="run-the-web-app"></a>Ejecución de la aplicación web

1. Pulse en CTRL+F5 para ejecutar la aplicación. El explorador predeterminado se inicia con la aplicación. 
1. Haga clic en **Crear** en el explorador y cree algunas tareas en la aplicación de lista de tareas.

<!-- 
## Deploy the app to Azure 
1. In VS, right click .. publish
2. This is so easy, why is this critical step missed?
-->
## <a name="review-slas-in-the-azure-portal"></a>Revisión de los SLA en Azure Portal

[!INCLUDE [cosmosdb-tutorial-review-slas](includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Cosmos, crear una colección y ejecutar una aplicación de consola. Ahora puede importar datos adicionales en la base de datos de Cosmos. 

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
