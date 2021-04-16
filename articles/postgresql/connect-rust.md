---
title: 'Inicio rápido: Conexión con Rust en Azure Database for PostgreSQL (servidor único)'
description: 'En esta guía de inicio rápido se proporcionan ejemplos de código de Rust que puede usar para conectarse y consultar datos de Azure Database for PostgreSQL: servidor único.'
author: abhirockzz
ms.author: abhishgu
ms.service: postgresql
ms.devlang: rust
ms.topic: quickstart
ms.date: 03/26/2021
ms.openlocfilehash: 00adc50ac14e627eb356a3e62ce0faa5a9716aa3
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106505562"
---
# <a name="quickstart-use-rust-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Inicio rápido: Uso de Rust para conectarse y consultar datos en Azure Database for PostgreSQL: servidor único

En este artículo, aprenderá a usar el [controlador Rust de PostgreSQL](https://github.com/sfackler/rust-postgres) para interactuar con Azure Database for PostgreSQL mediante la exploración de las operaciones CRUD (siglas en inglés de creación, lectura, actualización, eliminación) implementadas en el código de ejemplo. Por último, puede ejecutar la aplicación localmente para verla en acción.

## <a name="prerequisites"></a>Requisitos previos
Para esta guía de inicio rápido, necesitará lo siguiente:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free).
- Una versión reciente de [Rust](https://www.rust-lang.org/tools/install) instalada.
- Crear un servidor único de Azure Database for PostgreSQL mediante [Azure Portal](./quickstart-create-server-database-portal.md) <br/> o la [CLI de Azure](./quickstart-create-server-database-azure-cli.md).
- En función de si usa el acceso público o privado, complete **UNA** de las acciones siguientes para habilitar la conectividad.

  |Acción| Método de conectividad|Guía paso a paso|
  |:--------- |:--------- |:--------- |
  | **Configurar reglas de firewall** | Público | [Portal](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
  | **Configurar el punto de conexión de servicio** | Público | [Portal](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)|
  | **Configuración del vínculo privado** | Private | [Portal](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) |

- [Git](https://git-scm.com/downloads) instalado.

## <a name="get-database-connection-information"></a>Obtener información de conexión de base de datos
La conexión a una base de datos Azure Database for PostgreSQL requiere el nombre completo del servidor y las credenciales de inicio de sesión. Puede obtener esta información en el Azure Portal.

1. En el [Azure Portal](https://portal.azure.com/), busque y seleccione el nombre del servidor de Azure Database for PostgreSQL.
1. En la página de **Información general** del servidor, copie el nombre completo del **Servidor** y el **Nombre del usuario administrador**. El **nombre completo del servidor** siempre tiene el formato *\<my-server-name>.postgres.database.azure.com* y el **nombre del usuario administrador** siempre tiene el formato *\<my-admin-username>@\<my-server-name>* .

## <a name="review-the-code-optional"></a>Revisión del código (opcional)

Si le interesa saber cómo funciona el código, puede revisar los siguientes fragmentos de código. De lo contrario, puede pasar a [ejecutar la aplicación](#run-the-application).

### <a name="connect"></a>Conectar

La función `main` se inicia mediante la conexión a Azure Database for PostgreSQL y depende de las siguientes variables de entorno siguientes para obtener información de conectividad: `POSTGRES_HOST`, `POSTGRES_USER`, `POSTGRES_PASSWORD` y `POSTGRES_DBNAME`. De forma predeterminada, el servicio de base de datos de PostgreSQL está configurado para requerir una conexión `TLS`. Puede deshabilitar este requisito si la aplicación cliente no admite la conectividad de `TLS`. Para más información, consulte [Configuración de la conectividad de TSL en Azure Database for PostgreSQL: un solo servidor](./concepts-ssl-connection-security.md).

En la aplicación de ejemplo de este artículo se usa TLS con el comando [postgres-openssl crate](https://crates.io/crates/postgres-openssl/). La función [postgres::Client::connect](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.connect) se utiliza para iniciar la conexión y el programa se cierra en caso de que se produzca un error.

```rust
fn main() {
    let pg_host = std::env::var("POSTGRES_HOST").expect("missing environment variable POSTGRES_HOST");
    let pg_user = std::env::var("POSTGRES_USER").expect("missing environment variable POSTGRES_USER");
    let pg_password = std::env::var("POSTGRES_PASSWORD").expect("missing environment variable POSTGRES_PASSWORD");
    let pg_dbname = std::env::var("POSTGRES_DBNAME").unwrap_or("postgres".to_string());

    let builder = SslConnector::builder(SslMethod::tls()).unwrap();
    let tls_connector = MakeTlsConnector::new(builder.build());

    let url = format!(
        "host={} port=5432 user={} password={} dbname={} sslmode=require",
        pg_host, pg_user, pg_password, pg_dbname
    );
    let mut pg_client = postgres::Client::connect(&url, tls_connector).expect("failed to connect to postgres");
...
}
```

### <a name="drop-and-create-table"></a>Colocación y creación de tabla

La aplicación de ejemplo usa una tabla `inventory` simple para mostrar las operaciones CRUD (creación, lectura, actualización, eliminación).

```sql
CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
```

La función `drop_create_table` intenta inicialmente la operación `DROP` en la tabla `inventory` antes de crear una nueva. Esto facilita el aprendizaje o la experimentación, ya que siempre comienza con un estado conocido (limpio). El método [Execute](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.execute) se usa para las operaciones de creación y colocación. 

```rust
const CREATE_QUERY: &str =
    "CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);";

const DROP_TABLE: &str = "DROP TABLE inventory";

fn drop_create_table(pg_client: &mut postgres::Client) {
    let res = pg_client.execute(DROP_TABLE, &[]);
    match res {
        Ok(_) => println!("dropped table"),
        Err(e) => println!("failed to drop table {}", e),
    }
    pg_client
        .execute(CREATE_QUERY, &[])
        .expect("failed to create 'inventory' table");
}
```

### <a name="insert-data"></a>Insertar datos

`insert_data` agrega entradas a la tabla `inventory`. Crea una [instrucción preparada](https://docs.rs/postgres/0.19.0/postgres/struct.Statement.html) con la función [prepare](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.prepare). 


```rust
const INSERT_QUERY: &str = "INSERT INTO inventory (name, quantity) VALUES ($1, $2) RETURNING id;";

fn insert_data(pg_client: &mut postgres::Client) {

    let prep_stmt = pg_client
        .prepare(&INSERT_QUERY)
        .expect("failed to create prepared statement");

    let row = pg_client
        .query_one(&prep_stmt, &[&"item-1", &42])
        .expect("insert failed");

    let id: i32 = row.get(0);
    println!("inserted item with id {}", id);
...
}
```

Tenga en cuenta también el uso del método [prepare_typed](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.prepare_typed), que permite especificar explícitamente los tipos de parámetros de consulta.

```rust
...
let typed_prep_stmt = pg_client
        .prepare_typed(&INSERT_QUERY, &[Type::VARCHAR, Type::INT4])
        .expect("failed to create prepared statement");

let row = pg_client
        .query_one(&typed_prep_stmt, &[&"item-2", &43])
        .expect("insert failed");

let id: i32 = row.get(0);
println!("inserted item with id {}", id);
...
```

Por último, se usa un bucle `for` para agregar `item-3`, `item-4` y `item-5` con la cantidad generada de forma aleatoria para cada uno.

```rust
...
    for n in 3..=5 {
        let row = pg_client
            .query_one(
                &typed_prep_stmt,
                &[
                    &("item-".to_owned() + &n.to_string()),
                    &rand::thread_rng().gen_range(10..=50),
                ],
            )
            .expect("insert failed");

        let id: i32 = row.get(0);
        println!("inserted item with id {} ", id);
    }
...
```

### <a name="query-data"></a>Consultar datos

La función `query_data` muestra cómo recuperar datos de la tabla `inventory`. El método [query_one](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.query_one) se utiliza para obtener un elemento por su `id`. 

```rust
const SELECT_ALL_QUERY: &str = "SELECT * FROM inventory;";
const SELECT_BY_ID: &str = "SELECT name, quantity FROM inventory where id=$1;";

fn query_data(pg_client: &mut postgres::Client) {

    let prep_stmt = pg_client
        .prepare_typed(&SELECT_BY_ID, &[Type::INT4])
        .expect("failed to create prepared statement");

    let item_id = 1;

    let c = pg_client
        .query_one(&prep_stmt, &[&item_id])
        .expect("failed to query item");

    let name: String = c.get(0);
    let quantity: i32 = c.get(1);
    println!("quantity for item {} = {}", name, quantity);
...
}
```

Todas las filas de la tabla Inventory se capturan mediante una consulta `select * from` con el método [query](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.query). Las filas devueltas se recorren en iteración para extraer el valor de cada columna mediante [get](https://docs.rs/postgres/0.19.0/postgres/row/struct.Row.html#method.get). 

> [!TIP]
> Observe cómo `get` permite especificar la columna por su índice numérico en la fila o por su nombre de columna.

```rust
...
    let items = pg_client
        .query(SELECT_ALL_QUERY, &[])
        .expect("select all failed");

    println!("listing items...");

    for item in items {
        let id: i32 = item.get("id");
        let name: String = item.get("name");
        let quantity: i32 = item.get("quantity");
        println!(
            "item info: id = {}, name = {}, quantity = {} ",
            id, name, quantity
        );
    }
...
```

### <a name="update-data"></a>Actualización de datos

La función `update_date` actualiza aleatoriamente la cantidad de todos los elementos. Como la función `insert_data` había agregado `5` filas, se tiene en cuenta lo mismo en el bucle `for`: `for n in 1..=5`

> [!TIP]
> Tenga en cuenta que se usa `query` en lugar de `execute`, ya que se pretende obtener el `id` y la `quantity` recién generada (mediante la [cláusula RETURNING](https://www.postgresql.org/docs/current/dml-returning.html)).

```rust
const UPDATE_QUERY: &str = "UPDATE inventory SET quantity = $1 WHERE name = $2 RETURNING quantity;";

fn update_data(pg_client: &mut postgres::Client) {
    let stmt = pg_client
        .prepare_typed(&UPDATE_QUERY, &[Type::INT4, Type::VARCHAR])
        .expect("failed to create prepared statement");

    for id in 1..=5 {
        let row = pg_client
            .query_one(
                &stmt,
                &[
                    &rand::thread_rng().gen_range(10..=50),
                    &("item-".to_owned() + &id.to_string()),
                ],
            )
            .expect("update failed");
        
        let quantity: i32 = row.get("quantity");
        println!("updated item id {} to quantity = {}", id, quantity);
    }
}
```

### <a name="delete-data"></a>Eliminación de datos

Por último, la función `delete` muestra cómo quitar un elemento de la tabla `inventory` por su `id`. El `id` se elige aleatoriamente: es un entero aleatorio entre `1` y `5` (`5` incluído), ya que la función `insert_data` tenía `5` filas agregadas para comenzar. 

> [!TIP]
> Tenga en cuenta que se usa `query` en lugar de `execute`, ya que se pretende obtener la información sobre el elemento que se acaba de eliminar (mediante la [cláusula RETURNING](https://www.postgresql.org/docs/current/dml-returning.html)).

```rust
const DELETE_QUERY: &str = "DELETE FROM inventory WHERE id = $1 RETURNING id, name, quantity;";

fn delete(pg_client: &mut postgres::Client) {
    let stmt = pg_client
        .prepare_typed(&DELETE_QUERY, &[Type::INT4])
        .expect("failed to create prepared statement");

    let item = pg_client
        .query_one(&stmt, &[&rand::thread_rng().gen_range(1..=5)])
        .expect("delete failed");

    let id: i32 = item.get(0);
    let name: String = item.get(1);
    let quantity: i32 = item.get(2);
    println!(
        "deleted item info: id = {}, name = {}, quantity = {} ",
        id, name, quantity
    );
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

1. Para empezar, ejecute el comando siguiente para clonar el repositorio de ejemplo:

    ```bash
    git clone https://github.com/Azure-Samples/azure-postgresql-rust-quickstart.git
    ```

2. Establezca las variables de entorno necesarias con los valores que copió de Azure Portal:

    ```bash
    export POSTGRES_HOST=<server name e.g. my-server.postgres.database.azure.com>
    export POSTGRES_USER=<admin username e.g. my-admin-user@my-server>
    export POSTGRES_PASSWORD=<admin password>
    export POSTGRES_DBNAME=<database name. it is optional and defaults to postgres>
    ```

3. Para ejecutar la aplicación, cambie al directorio donde lo clonó y ejecute `cargo run`:

    ```bash
    cd azure-postgresql-rust-quickstart
    cargo run
    ```

    Debería aparecer una salida similar a la siguiente:

    ```bash
    dropped 'inventory' table
    inserted item with id 1
    inserted item with id 2
    inserted item with id 3 
    inserted item with id 4 
    inserted item with id 5 
    quantity for item item-1 = 42
    listing items...
    item info: id = 1, name = item-1, quantity = 42 
    item info: id = 2, name = item-2, quantity = 43 
    item info: id = 3, name = item-3, quantity = 11 
    item info: id = 4, name = item-4, quantity = 32 
    item info: id = 5, name = item-5, quantity = 24 
    updated item id 1 to quantity = 27
    updated item id 2 to quantity = 14
    updated item id 3 to quantity = 31
    updated item id 4 to quantity = 16
    updated item id 5 to quantity = 10
    deleted item info: id = 4, name = item-4, quantity = 16 
    ```

4. Para confirmar, también puede conectarse a Azure Database for PostgreSQL [mediante psql](./quickstart-create-server-database-portal.md#connect-to-the-server-with-psql) y ejecutar consultas en la base de datos, por ejemplo:

    ```sql
    select * from inventory;
    ```

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/postgres-doc-feedback)

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar todos los recursos utilizados durante esta guía de inicio rápido, elimine el grupo de recursos con el siguiente comando:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Administración de un servidor de Azure Database for PostgreSQL mediante Azure Portal](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Administración de un servidor de Azure Database for PostgreSQL mediante la CLI](./how-to-manage-server-cli.md)<br/>

[¿No encuentra lo que busca? Háganoslo saber.](https://aka.ms/postgres-doc-feedback)
