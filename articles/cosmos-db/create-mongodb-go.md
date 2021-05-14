---
title: Conexión de una aplicación Go a la API de Azure Cosmos DB para MongoDB
description: En este inicio rápido se muestra cómo conectar una aplicación Go existente a la API de Azure Cosmos DB para MongoDB.
author: abhirockzz
ms.author: abhishgu
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: go
ms.topic: quickstart
ms.date: 04/24/2020
ms.openlocfilehash: 18f8ac52c84fa93a74f9b822322c07c13e082b0e
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206374"
---
# <a name="quickstart-connect-a-go-application-to-azure-cosmos-dbs-api-for-mongodb"></a>Inicio rápido: Conexión de una aplicación Go a la API de Azure Cosmos DB para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal. En este inicio rápido, se va a crear y administrar una cuenta de Azure Cosmos DB mediante Azure Cloud Shell, se va a clonar una aplicación de ejemplo existente desde GitHub y se va a configurar esta para que funcione con Azure Cosmos DB. 

La aplicación de ejemplo es una herramienta de administración `todo` de línea de comandos escrita en Go. La API de Azure Cosmos DB para MongoDB es [compatible con el protocolo de conexión de MongoDB](./mongodb-introduction.md), lo que permite que cualquier controlador cliente de MongoDB se conecte a ella. Esta aplicación usa el [controlador de Go para MongoDB](https://github.com/mongodb/mongo-go-driver) de una manera transparente, de forma que sabe que los datos se almacenan en una base de datos de Azure Cosmos DB.

## <a name="prerequisites"></a>Requisitos previos
- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure. También puede usar el [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) con la cadena de conexión `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- Instalación de [Go](https://golang.org/) en el equipo y conocimientos de cómo funciona.
- [Git](https://git-scm.com/downloads).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ejecute los comandos siguientes para clonar el repositorio de ejemplo.

1. Abra un símbolo del sistema, cree una carpeta denominada `git-samples` y, después, cierre el símbolo del sistema.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo. 

    ```bash
    git clone https://github.com/Azure-Samples/cosmosdb-go-mongodb-quickstart
    ```

## <a name="review-the-code"></a>Revisión del código

Este paso es opcional. Si le interesa saber cómo funciona la aplicación, puede revisar los siguientes fragmentos de código. En caso contrario, puede ir directamente a [Ejecutar la aplicación](#run-the-application). El diseño de la aplicación es el siguiente:

```bash
.
├── go.mod
├── go.sum
└── todo.go
```

Los fragmentos de código siguientes se han tomado del archivo `todo.go`.

### <a name="connecting-the-go-app-to-azure-cosmos-db"></a>Conexión de una aplicación Go a Azure Cosmos DB

[`clientOptions`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo/options?tab=doc#ClientOptions) encapsula la cadena de conexión de Azure Cosmos DB, que se pasa mediante una variable de entorno (se pueden encontrar los detalles en la próxima sección). La conexión se inicializa mediante [`mongo.NewClient`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#NewClient) al que se pasa la instancia `clientOptions`. Se invoca la función [`Ping`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Client.Ping) para confirmar que la conectividad es correcta (es una estrategia con respuesta rápida a errores).

```go
    ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
    defer cancel()

    clientOptions := options.Client().ApplyURI(mongoDBConnectionString).SetDirect(true)
    
    c, err := mongo.NewClient(clientOptions)
    err = c.Connect(ctx)
    if err != nil {
        log.Fatalf("unable to initialize connection %v", err)
    }

    err = c.Ping(ctx, nil)
    if err != nil {
        log.Fatalf("unable to connect %v", err)
    }
```

> [!NOTE] 
> El uso de la configuración de [`SetDirect(true)`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo/options?tab=doc#ClientOptions.SetDirect) es importante, ya que sin ella obtendrá el siguiente error de conectividad: `unable to connect connection(cdb-ms-prod-<azure-region>-cm1.documents.azure.com:10255[-4]) connection is closed`.
>

### <a name="create-a-todo-item"></a>Creación de un elemento `todo`

Para crear un elemento `todo`, se obtiene un identificador para [`mongo.Collection`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection) y se invoca la función [`InsertOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.InsertOne). 

```go
func create(desc string) {
    c := connect()
    ctx := context.Background()
    defer c.Disconnect(ctx)

    todoCollection := c.Database(database).Collection(collection)
    r, err := todoCollection.InsertOne(ctx, Todo{Description: desc, Status: statusPending})
    if err != nil {
        log.Fatalf("failed to add todo %v", err)
    }
```

Se pasa un struct `Todo` que contiene la descripción y el estado (que se establece inicialmente en `pending`).

```go
type Todo struct {
    ID          primitive.ObjectID `bson:"_id,omitempty"`
    Description string             `bson:"description"`
    Status      string             `bson:"status"`
}
```
### <a name="list-todo-items"></a>Enumeración de elementos `todo`

Se pueden enumerar los elementos TODO según unos criterios. Se crea un objeto [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D) para encapsular los criterios de filtro.

```go
func list(status string) {
    .....
    var filter interface{}
    switch status {
    case listAllCriteria:
        filter = bson.D{}
    case statusCompleted:
        filter = bson.D{{statusAttribute, statusCompleted}}
    case statusPending:
        filter = bson.D{{statusAttribute, statusPending}}
    default:
        log.Fatal("invalid criteria for listing todo(s)")
    }
```

Se usa [`Find`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.Find) para buscar documentos según el filtro y el resultado se convierte en un sector de `Todo`

```go
    todoCollection := c.Database(database).Collection(collection)
    rs, err := todoCollection.Find(ctx, filter)
    if err != nil {
        log.Fatalf("failed to list todo(s) %v", err)
    }
    var todos []Todo
    err = rs.All(ctx, &todos)
    if err != nil {
        log.Fatalf("failed to list todo(s) %v", err)
    }
```

Por último, la información se representa en formato tabular.

```go
    todoTable := [][]string{}

    for _, todo := range todos {
        s, _ := todo.ID.MarshalJSON()
        todoTable = append(todoTable, []string{string(s), todo.Description, todo.Status})
    }

    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader([]string{"ID", "Description", "Status"})

    for _, v := range todoTable {
        table.Append(v)
    }
    table.Render()
```

### <a name="update-a-todo-item"></a>Actualización de un elemento `todo`

Un elemento `todo` se puede actualizar en función de su valor de `_id`. Se crea un filtro [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D) basado en el valor de `_id` y otro para la información actualizada, que tiene un nuevo estado (`completed` o `pending`) en este caso. Por último, se invoca la función [`UpdateOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.UpdateOne) con el filtro y el documento actualizado.

```go
func update(todoid, newStatus string) {
....
    todoCollection := c.Database(database).Collection(collection)
    oid, err := primitive.ObjectIDFromHex(todoid)
    if err != nil {
        log.Fatalf("failed to update todo %v", err)
    }
    filter := bson.D{{"_id", oid}}
    update := bson.D{{"$set", bson.D{{statusAttribute, newStatus}}}}
    _, err = todoCollection.UpdateOne(ctx, filter, update)
    if err != nil {
        log.Fatalf("failed to update todo %v", err)
    }
```

### <a name="delete-a-todo"></a>Eliminación de un elemento `todo`

Un elemento `todo` se elimina según su valor de `_id` y se encapsula en forma de una instancia [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D). Se invoca a [`DeleteOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.DeleteOne) para eliminar el documento.

```go
func delete(todoid string) {
....
    todoCollection := c.Database(database).Collection(collection)
    oid, err := primitive.ObjectIDFromHex(todoid)
    if err != nil {
        log.Fatalf("invalid todo ID %v", err)
    }
    filter := bson.D{{"_id", oid}}
    _, err = todoCollection.DeleteOne(ctx, filter)
    if err != nil {
        log.Fatalf("failed to delete todo %v", err)
    }
}
```

## <a name="build-the-application"></a>Compilar la aplicación

Cambie al directorio en el que ha clonado la aplicación y compílela (mediante `go build`).

```bash
cd monogdb-go-quickstart
go build -o todo
```

Para confirmar que la aplicación se compiló correctamente:

```bash
./todo --help
```

## <a name="setup-azure-cosmos-db"></a>Configuración de Azure Cosmos DB

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]. 

Si usa una CLI de Azure instalada, inicie sesión en la suscripción de Azure con el comando [az login](/cli/azure/reference-index#az_login) y siga las instrucciones de la pantalla. Puede omitir este paso si usa Azure Cloud Shell.

```azurecli
az login 
``` 
   
### <a name="add-the-azure-cosmos-db-module"></a>Agregar el módulo de Azure Cosmos DB

Si va a usar una CLI de Azure instalada, ejecute el comando `az` para comprobar si el componente `cosmosdb` ya está instalado. Si `cosmosdb` está en la lista de comandos de referencia, continúe con el comando siguiente. Puede omitir este paso si usa Azure Cloud Shell.

Si `cosmosdb` no está en la lista de comandos de referencia, vuelva a instalar la [CLI de Azure](/cli/azure/install-azure-cli).

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos](../azure-resource-manager/management/overview.md) con el comando [az group create](/cli/azure/group#az_group_create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran recursos de Azure como aplicaciones web, bases de datos y cuentas de almacenamiento. 

En el ejemplo siguiente se crea un grupo de recursos en la región de Oeste de Europa. Elija un nombre único para el grupo de recursos.

Si usa Azure Cloud Shell, seleccione **Probarlo**, siga las indicaciones de la pantalla para iniciar sesión y, a continuación, copie el comando en el símbolo del sistema.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

### <a name="create-an-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Cree una cuenta de Cosmos con el comando [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create).

En el comando siguiente, sustituya su propio nombre único de la cuenta de Cosmos donde vea el marcador de posición `<cosmosdb-name>`. Este nombre único se usará como parte del punto de conexión de Cosmos DB (`https://<cosmosdb-name>.documents.azure.com/`), por lo que debe ser único entre todas las cuentas de Cosmos de Azure. 

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

El parámetro `--kind MongoDB` habilita las conexiones de cliente de MongoDB.

Cuando se crea la cuenta de Azure Cosmos DB, la CLI de Azure muestra información similar a la del ejemplo siguiente. 

> [!NOTE]
> Este ejemplo usa JSON como el formato de salida de la CLI de Azure, que es el valor predeterminado. Para usar otro formato de salida, consulte [Formatos de salida para los comandos de la CLI de Azure](/cli/azure/format-output-azure-cli). 

```json
{
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb-name>.documents.azure.com:443/",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
DB/databaseAccounts/<cosmosdb-name>",
  "kind": "MongoDB",
  "location": "West Europe",
  "name": "<cosmosdb-name>",
  "readLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "writeLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ]
} 
```

### <a name="retrieve-the-database-key"></a>Recuperación de la clave de base de datos

Para conectarse a una base de datos de Cosmos, necesita la clave de base de datos. Use el comando [az cosmosdb keys list](/cli/azure/cosmosdb/keys#az_cosmosdb_keys_list) para recuperar la clave principal.

```azurecli-interactive
az cosmosdb keys list --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

La CLI de Azure genera información similar a la del ejemplo siguiente. 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

## <a name="configure-the-application"></a>Configuración de la aplicación 

<a name="devconfig"></a>
### <a name="export-the-connection-string-mongodb-database-and-collection-names-as-environment-variables"></a>Exporte la cadena de conexión, la base de datos de MongoDB y los nombres de colección como variables de entorno. 

```bash
export MONGODB_CONNECTION_STRING="mongodb://<COSMOSDB_ACCOUNT_NAME>:<COSMOSDB_PASSWORD>@<COSMOSDB_ACCOUNT_NAME>.documents.azure.com:10255/?ssl=true&replicaSet=globaldb&maxIdleTimeMS=120000&appName=@<COSMOSDB_ACCOUNT_NAME>@"
```

> [!NOTE] 
> La opción `ssl=true` es importante debido a los requisitos de Cosmos DB. Para más información, consulte [Requisitos de la cadena de conexión](connect-mongodb-account.md#connection-string-requirements).
>

En la variable de entorno `MONGODB_CONNECTION_STRING`, reemplace los marcadores de posición de `<COSMOSDB_ACCOUNT_NAME>` y `<COSMOSDB_PASSWORD>`.

1. `<COSMOSDB_ACCOUNT_NAME>`: el nombre de la cuenta de Azure Cosmos DB que ha creado.
2. `<COSMOSDB_PASSWORD>`: la clave de base de datos extraída en el paso anterior.

```bash
export MONGODB_DATABASE=todo-db
export MONGODB_COLLECTION=todos
```

Puede elegir los valores que prefiera para `MONGODB_DATABASE` y `MONGODB_COLLECTION` o dejarlos como están.

## <a name="run-the-application"></a>Ejecución de la aplicación

Para crear un elemento `todo`:

```bash
./todo --create "Create an Azure Cosmos DB database account"
```

Si la operación es correcta, verá una salida con el valor de `_id` de MongoDB del documento recién creado:

```bash
added todo ObjectID("5e9fd6befd2f076d1f03bd8a")
```

Creación de otro elemento `todo`:

```bash
./todo --create "Get the MongoDB connection string using the Azure CLI"
```

Enumeración de todos los elementos `todo`:

```bash
./todo --list all
```

Verá los que acaba de agregar en formato tabular:

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6b1bcd2fa6bd267d4c4" | Create an Azure Cosmos DB      | pending   |
|                            | database account               |           |
| "5e9fd6befd2f076d1f03bd8a" | Get the MongoDB connection     | pending   |
|                            | string using the Azure CLI     |           |
+----------------------------+--------------------------------+-----------+
```

Para actualizar el estado de un elemento `todo` (por ejemplo, cambiarlo al estado `completed`), use el identificador de `todo`:

```bash
./todo --update 5e9fd6b1bcd2fa6bd267d4c4,completed
```

Enumeración de solo los elementos `todo` completados:

```bash
./todo --list completed
```

Verá el que acaba de actualizar:

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6b1bcd2fa6bd267d4c4" | Create an Azure Cosmos DB      | completed |
|                            | database account               |           |
+----------------------------+--------------------------------+-----------+
```

### <a name="view-data-in-data-explorer"></a>Ver datos en el Explorador de datos

Los datos almacenados en una instancia de Azure Cosmos DB se pueden ver y consultar en Azure Portal.

Para ver y consultar los datos de usuario creados en el paso anterior y trabajar con ellos, inicie sesión en [Azure Portal](https://portal.azure.com) en el explorador web.

En el cuadro de búsqueda superior, escriba **Azure Cosmos DB**. Cuando se abra la hoja de la cuenta de Cosmos, seleccione su cuenta de Cosmos. En el panel de navegación izquierdo, seleccione **Explorador de datos**. Expanda la colección en el panel Colecciones. Ahora puede ver los documentos de la colección, consultar los datos e incluso crear y ejecutar procedimientos almacenados, desencadenadores y UDF. 

:::image type="content" source="./media/create-mongodb-go/go-cosmos-db-data-explorer.png" alt-text="El Explorador de datos muestra el documento recién creado":::


Eliminación de un elemento `todo` mediante su identificador:

```bash
./todo --delete 5e9fd6b1bcd2fa6bd267d4c4,completed
```

Enumeración de los elementos `todo` que se van a confirmar:

```bash
./todo --list all
```

El elemento `todo` que acaba de eliminar no debe estar presente:

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6befd2f076d1f03bd8a" | Get the MongoDB connection     | pending   |
|                            | string using the Azure CLI     |           |
+----------------------------+--------------------------------+-----------+
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de la API de Azure Cosmos DB para MongoDB mediante Azure Cloud Shell, y a crear y ejecutar una aplicación Go de línea de comandos para administrar elementos `todo`. Ahora puede importar datos adicionales en la cuenta de Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)