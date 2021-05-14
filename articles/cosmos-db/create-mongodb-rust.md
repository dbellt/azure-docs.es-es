---
title: Conexión de una aplicación Rust a la API de Azure Cosmos DB para MongoDB
description: En este inicio rápido se muestra cómo compilar una aplicación Rust con el respaldo de la API de Azure Cosmos DB para MongoDB.
author: abhirockzz
ms.author: abhishgu
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: rust
ms.topic: quickstart
ms.date: 01/12/2021
ms.openlocfilehash: a9a6c2b0a106b08d8aad880b0b5f18879ac5645b
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206266"
---
# <a name="quickstart-connect-a-rust-application-to-azure-cosmos-dbs-api-for-mongodb"></a>Inicio rápido: Conexión de una aplicación Rust a la API de Azure Cosmos DB para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
> * [Rust](create-mongodb-rust.md)
>

Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal. El ejemplo que se ha presentado en este artículo es una sencilla aplicación basada en línea de comandos que usa el [controlador de Rust para MongoDB](https://github.com/mongodb/mongo-rust-driver). Como la API de Azure Cosmos DB para MongoDB es [compatible con el protocolo de conexión de MongoDB](./mongodb-introduction.md), cualquier controlador cliente de MongoDB puede conectarse a ella.

Aprenderá a usar el controlador Rust de MongoDB para interactuar con la API de Azure Cosmos DB para MongoDB mediante la exploración de las operaciones de CRUD (crear, leer, actualizar y eliminar) implementadas en el código de ejemplo. Por último, puede ejecutar la aplicación localmente para verla en acción.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure. También puede usar el [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) con la cadena de conexión `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- [Rust](https://www.rust-lang.org/tools/install) (versión 1.39 o superiores)
- [Git](https://git-scm.com/downloads)

## <a name="set-up-azure-cosmos-db"></a>Configuración de Azure Cosmos DB

Para configurar una cuenta de Azure Cosmos DB, siga las [instrucciones que encontrará aquí](create-mongodb-dotnet.md). La aplicación necesitará la cadena de conexión de MongoDB que se puede capturar mediante Azure Portal. Para más información, consulte [Obtención de la cadena de conexión de MongoDB para personalizar](connect-mongodb-account.md#get-the-mongodb-connection-string-to-customize).

## <a name="run-the-application"></a>Ejecución de la aplicación

### <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ejecute los comandos siguientes para clonar el repositorio de ejemplo.

1. Abra un símbolo del sistema, cree una carpeta denominada `git-samples` y, después, cierre el símbolo del sistema.

    ```bash
    mkdir "C:\git-samples"
    ```

1. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

1. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo. 

    ```bash
    git clone https://github.com/Azure-Samples/cosmosdb-rust-mongodb-quickstart
    ```

### <a name="build-the-application"></a>Compilar la aplicación

Para compilar el binario:

```bash
cargo build --release
```

### <a name="configure-the-application"></a>Configuración de la aplicación 

Exporte la cadena de conexión, la base de datos de MongoDB y los nombres de colección como variables de entorno. 

```bash
export MONGODB_URL="mongodb://<COSMOSDB_ACCOUNT_NAME>:<COSMOSDB_PASSWORD>@<COSMOSDB_ACCOUNT_NAME>.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&maxIdleTimeMS=120000&appName=@<COSMOSDB_ACCOUNT_NAME>@"
```

> [!NOTE] 
> La opción `ssl=true` es importante debido a los requisitos de Cosmos DB. Para más información, consulte [Requisitos de la cadena de conexión](connect-mongodb-account.md#connection-string-requirements).
>

En la variable de entorno `MONGODB_URL`, reemplace los marcadores de posición de `<COSMOSDB_ACCOUNT_NAME>` y `<COSMOSDB_PASSWORD>`.

- `<COSMOSDB_ACCOUNT_NAME>`: el nombre de la cuenta de Azure Cosmos DB que ha creado.
- `<COSMOSDB_PASSWORD>`: la clave de base de datos extraída en el paso anterior.

```bash
export MONGODB_DATABASE=todos_db
export MONGODB_COLLECTION=todos
```

Puede elegir los valores que prefiera para `MONGODB_DATABASE` y `MONGODB_COLLECTION` o dejarlos como están.

Para ejecutar la aplicación, cambie a la carpeta correcta (donde se encuentra el binario de la aplicación):

```bash
cd target/release
```

Para crear un elemento `todo`:

```bash
./todo create "Create an Azure Cosmos DB database account"
```

Si la operación es correcta, verá una salida con el valor de `_id` de MongoDB del documento recién creado:

```bash
inserted todo with id = ObjectId("5ffd1ca3004cc935004a0959")
```

Creación de otro elemento `todo`:

```bash
./todo create "Get the MongoDB connection string using the Azure CLI"
```

Enumeración de todos los elementos `todo`:

```bash
./todo list all
```

Verá los que acaba de agregar:

```bash
todo_id: 5ffd1ca3004cc935004a0959 | description: Create an Azure Cosmos DB database account | status: pending
todo_id: 5ffd1cbe003bcec40022c81c | description: Get the MongoDB connection string using the Azure CLI | status: pending
```

Para actualizar el estado de un elemento `todo` (por ejemplo, cambiarlo al estado `completed`), use el identificador de `todo` tal cual:

```bash
./todo update 5ffd1ca3004cc935004a0959 completed

#output
updating todo_id 5ffd1ca3004cc935004a0959 status to completed
updated status for todo id 5ffd1ca3004cc935004a0959
```

Enumeración de solo los elementos `todo` completados:

```bash
./todo list completed
```

Verá el que acaba de actualizar:

```bash
listing 'completed' todos

todo_id: 5ffd1ca3004cc935004a0959 | description: Create an Azure Cosmos DB database account | status: completed
```

Eliminación de un elemento `todo` mediante su identificador:

```bash
./todo delete 5ffd1ca3004cc935004a0959
```

Enumeración de los elementos `todo` que se van a confirmar:

```bash
./todo list all
```

El elemento `todo` que acaba de eliminar no debe estar presente.

### <a name="view-data-in-data-explorer"></a>Ver datos en el Explorador de datos

Los datos almacenados en una instancia de Azure Cosmos DB se pueden ver y consultar en Azure Portal.

Para ver y consultar los datos de usuario creados en el paso anterior y trabajar con ellos, inicie sesión en [Azure Portal](https://portal.azure.com) en el explorador web.

En el cuadro de búsqueda superior, escriba **Azure Cosmos DB**. Cuando se abra la hoja de la cuenta de Cosmos, seleccione su cuenta de Cosmos. En el panel de navegación izquierdo, seleccione **Explorador de datos**. Expanda la colección en el panel Colecciones. Ahora puede ver los documentos de la colección, consultar los datos e incluso crear y ejecutar procedimientos almacenados, desencadenadores y UDF.

## <a name="review-the-code-optional"></a>Revisión del código (opcional)

Si le interesa saber cómo funciona la aplicación, puede revisar los siguientes fragmentos de código de esta sección. Los siguientes fragmentos de código se han tomado del archivo `src/main.rs`.

La función `main` es el punto de entrada de la aplicación `todo`. Espera que la variable de entorno `MONGODB_URL` proporcione la dirección URL de conexión para la API de Azure Cosmos DB para MongoDB. Se crea una nueva instancia de `TodoManager`, seguida de una [expresión `match` ](https://doc.rust-lang.org/book/ch06-02-match.html) que delega en el método `TodoManager` adecuado en función de la operación elegida por el usuario (`create`, `update`, `list`o `delete`).

```rust
fn main() {
    let conn_string = std::env::var_os("MONGODB_URL").expect("missing environment variable MONGODB_URL").to_str().expect("failed to get MONGODB_URL").to_owned();
    let todos_db_name = std::env::var_os("MONGODB_DATABASE").expect("missing environment variable MONGODB_DATABASE").to_str().expect("failed to get MONGODB_DATABASE").to_owned();
    let todos_collection_name = std::env::var_os("MONGODB_COLLECTION").expect("missing environment variable MONGODB_COLLECTION").to_str().expect("failed to get MONGODB_COLLECTION").to_owned();

    let tm = TodoManager::new(conn_string,todos_db_name.as_str(), todos_collection_name.as_str());
      
    let ops: Vec<String> = std::env::args().collect();
    let op = ops[1].as_str();

    match op {
        CREATE_OPERATION_NAME => tm.add_todo(ops[2].as_str()),
        LIST_OPERATION_NAME => tm.list_todos(ops[2].as_str()),
        UPDATE_OPERATION_NAME => tm.update_todo_status(ops[2].as_str(), ops[3].as_str()),
        DELETE_OPERATION_NAME => tm.delete_todo(ops[2].as_str()),
        _ => panic!(INVALID_OP_ERR_MSG)
    }
}
```

`TodoManager` es un elemento `struct` que encapsula [mongodb::sync::Collection](https://docs.rs/mongodb/1.1.1/mongodb/sync/struct.Collection.html). Al intentar crear una instancia de `TodoManager` mediante la función `new`, inicia una conexión a la API de Azure Cosmos DB para MongoDB.

```rust
struct TodoManager {
    coll: Collection
}
....
impl TodoManager{
    fn new(conn_string: String, db_name: &str, coll_name: &str) -> Self{
        let mongo_client = Client::with_uri_str(&*conn_string).expect("failed to create client");
        let todo_coll = mongo_client.database(db_name).collection(coll_name);
            
        TodoManager{coll: todo_coll}
    }
....
```

Y lo que es más importante, `TodoManager` tiene métodos que le ayudan a administrar `todo`s. Vamos a verlos de uno en uno.

El método `add_todo` toma una descripción de `todo` que proporciona el usuario y crea una instancia del struct `Todo` similar a esta. El marco de [serde](https://github.com/serde-rs/serde) se usa para asignar (serializar/deserializar) datos de BSON a instancias de structs de `Todo`. Observe cómo se usan los atributos de campos de `serde` para personalizar el proceso de serialización/deserialzación. Por ejemplo, el campo `todo_id` del `struct` de Todo es `ObjectId` y se almacena en MongoDB como `_id`.

```rust
#[derive(Serialize, Deserialize)]
struct Todo {
    #[serde(rename = "_id", skip_serializing_if = "Option::is_none")]
    todo_id: Option<bson::oid::ObjectId>,
    #[serde(rename = "description")]
    desc: String,
    status: String,
}
```

[Collection.insert_one](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.insert_one) acepta un [documento](https://docs.rs/bson/1.1.0/bson/document/struct.Document.html) que representa los detalles de `todo` que se van a agregar. Tenga en cuenta que la conversión de `Todo` a `Document` es un proceso de dos pasos, que se consigue mediante una combinación de [to_bson](https://docs.rs/bson/1.1.0/bson/ser/fn.to_bson.html) y [as_document](https://docs.rs/bson/1.1.0/bson/enum.Bson.html#method.as_document).

```rust
fn add_todo(self, desc: &str) {
    let new_todo = Todo {
        todo_id: None,
        desc: String::from(desc),
        status: String::from(TODO_PENDING_STATUS),
    };
    
    let todo_doc = mongodb::bson::to_bson(&new_todo).expect("struct to BSON conversion failed").as_document().expect("BSON to Document conversion failed").to_owned();
        
    let r = self.coll.insert_one(todo_doc, None).expect("failed to add todo");    
    println!("inserted todo with id = {}", r.inserted_id);
}
```

[Collection.find](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.find) se usa para recuperar *todos los*  `todo`s o filtrarlos por el estado proporcionado por el usuario (`pending` o `completed`). Observe que en el bucle `while`, cada `Document` que se obtiene como resultado de la búsqueda se convierte en un struct de `Todo` mediante [bson::from_bson](https://docs.rs/bson/1.1.0/bson/de/fn.from_bson.html). Esto es lo contrario de lo que se hacía en el método `add_todo`.

```rust
fn list_todos(self, status_filter: &str) {
    let mut filter = doc!{};
    if status_filter == TODO_PENDING_STATUS ||  status_filter == TODO_COMPLETED_STATUS{
        println!("listing '{}' todos",status_filter);
        filter = doc!{"status": status_filter}
    } else if status_filter != "all" {
        panic!(INVALID_FILTER_ERR_MSG)
    }

    let mut todos = self.coll.find(filter, None).expect("failed to find todos");
    
    while let Some(result) = todos.next() {
        let todo_doc = result.expect("todo not present");
        let todo: Todo = bson::from_bson(Bson::Document(todo_doc)).expect("BSON to struct conversion failed");
        println!("todo_id: {} | description: {} | status: {}", todo.todo_id.expect("todo id missing"), todo.desc, todo.status);
    }
}
```

Un estado `todo` se puede actualizar (de `pending` a `completed` o viceversa). `todo` se convierte en [bson::oid::ObjectId](https://docs.rs/bson/1.1.0/bson/oid/struct.ObjectId.html) que luego usa el método[Collection.update_one](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.update_one) para buscar el documento que se debe actualizar.

```rust
fn update_todo_status(self, todo_id: &str, status: &str) {

    if status != TODO_COMPLETED_STATUS && status != TODO_PENDING_STATUS {
        panic!(INVALID_FILTER_ERR_MSG)
    }

    println!("updating todo_id {} status to {}", todo_id, status);
    
    let id_filter = doc! {"_id": bson::oid::ObjectId::with_string(todo_id).expect("todo_id is not valid ObjectID")};

    let r = self.coll.update_one(id_filter, doc! {"$set": { "status": status }}, None).expect("update failed");
    if r.modified_count == 1 {
        println!("updated status for todo id {}",todo_id);
    } else if r.matched_count == 0 {
        println!("could not update. check todo id {}",todo_id);
    }
}
```

La eliminación de `todo` se realiza fácilmente mediante el método [Collection.delete_one](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.delete_one).


```rust
fn delete_todo(self, todo_id: &str) {
    println!("deleting todo {}", todo_id);
    
    let id_filter = doc! {"_id": bson::oid::ObjectId::with_string(todo_id).expect("todo_id is not valid ObjectID")};

    self.coll.delete_one(id_filter, None).expect("delete failed").deleted_count;
}
``` 

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de la API de Azure Cosmos DB para MongoDB mediante Azure Cloud Shell y a crear y ejecutar una aplicación de línea de comandos Rust para administrar elementos `todo`. Ahora puede importar datos adicionales en la cuenta de Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
