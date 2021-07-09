---
title: Uso de Azure Cache for Redis con Rust
description: En este inicio rápido, aprenderá a interaccionar con Azure Redis Cache con Rust.
author: abhirockzz
ms.author: abhishgu
ms.service: cache
ms.devlang: rust
ms.topic: quickstart
ms.date: 01/08/2021
ms.openlocfilehash: acbf5933f01a465ad1855c049796901da5d1ff90
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110059741"
---
# <a name="quickstart-use-azure-cache-for-redis-with-rust"></a>Inicio rápido: Uso de Azure Cache for Redis con Rust

En este artículo, aprenderá a usar el [lenguaje de programación Rust](https://www.rust-lang.org/) para interactuar con [Azure Cache for Redis](./cache-overview.md). También obtendrá información sobre las estructuras de datos de Redis más usadas: 

* [String](https://redis.io/topics/data-types-intro#redis-strings) 
* [Hash](https://redis.io/topics/data-types-intro#redis-hashes) 
* [Lista](https://redis.io/topics/data-types-intro#redis-lists) 

En este ejemplo, usará la biblioteca [redis-rs](https://github.com/mitsuhiko/redis-rs) para Redis. Este cliente expone las API de alto y bajo nivel, y verá ambos estilos en acción.

## <a name="skip-to-the-code-on-github"></a>Ir al código en GitHub

Si quiere pasar directamente al código, consulte [Inicio rápido: Uso de Azure Cache for Redis con Rust](https://github.com/Azure-Samples/azure-redis-cache-rust-quickstart/) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- [Rust](https://www.rust-lang.org/tools/install) (versión 1.39 o superiores)
- [Git](https://git-scm.com/downloads)

## <a name="create-an-azure-cache-for-redis-instance"></a>Creación de una instancia de Azure Redis Cache
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

## <a name="review-the-code-optional"></a>Revisión del código (opcional)

Si le interesa saber cómo funciona el código, puede revisar los siguientes fragmentos de código. De lo contrario, puede pasar a [ejecutar la aplicación](#run-the-application).

La función `connect` se usa para establecer una conexión con Azure Cache for Redis. Espera que el nombre de host y la contraseña (clave de acceso) se pasen a través de variables de entorno `REDIS_HOSTNAME` y `REDIS_PASSWORD` respectivamente. El formato de la dirección URL de conexión es `rediss://<username>:<password>@<hostname>`: Azure Cache for Redis solo acepta conexiones seguras con [TLS 1.2 como la versión mínima exigida](cache-remove-tls-10-11.md).

La llamada a [redis::Client::open](https://docs.rs/redis/0.19.0/redis/struct.Client.html#method.open) realiza una validación básica mientras [get_connection()](https://docs.rs/redis/0.19.0/redis/struct.Client.html#method.get_connection) inicia realmente la conexión. El programa se detiene si se produce un error en la conectividad por cualquier motivo. Por ejemplo, un motivo podría ser una contraseña incorrecta.

```rust
fn connect() -> redis::Connection {
    let redis_host_name =
        env::var("REDIS_HOSTNAME").expect("missing environment variable REDIS_HOSTNAME");
    let redis_password =
        env::var("REDIS_PASSWORD").expect("missing environment variable REDIS_PASSWORD");
    let redis_conn_url = format!("rediss://:{}@{}", redis_password, redis_host_name);

    redis::Client::open(redis_conn_url)
        .expect("invalid connection URL")
        .get_connection()
        .expect("failed to connect to redis")
}
```

La función `basics` abarca los comandos [SET](https://redis.io/commands/set), [GET](https://redis.io/commands/get) e [INCR](https://redis.io/commands/incr). 

La API de bajo nivel se usa para `SET` y `GET`, que establece y recupera el valor de una clave denominada `foo`. 

El comando `INCRBY` se ejecuta mediante una API de alto nivel, es decir, [incr](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.incr) incrementa el valor de una clave (denominada `counter`) mediante `2` seguido de una llamada a [get](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.get) para recuperarlo.

```rust
fn basics() {
    let mut conn = connect();
    let _: () = redis::cmd("SET")
        .arg("foo")
        .arg("bar")
        .query(&mut conn)
        .expect("failed to execute SET for 'foo'");

    let bar: String = redis::cmd("GET")
        .arg("foo")
        .query(&mut conn)
        .expect("failed to execute GET for 'foo'");
    println!("value for 'foo' = {}", bar);

    let _: () = conn
        .incr("counter", 2)
        .expect("failed to execute INCR for 'counter'");
    let val: i32 = conn
        .get("counter")
        .expect("failed to execute GET for 'counter'");
    println!("counter = {}", val);
}
```

El siguiente fragmento de código muestra la funcionalidad de una estructura de datos `HASH` de Redis. [HSET](https://redis.io/commands/hset) se llama mediante la API de bajo nivel para almacenar información (`name`, `version`, `repo`) sobre los controladores de REDIS (clientes). Por ejemplo, los detalles del controlador de Rust (uno que se usa en este código de ejemplo) se capturan en forma de una estructura [BTreeMap](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html) y, a continuación, se pasan a la API de bajo nivel. A continuación, se recupera mediante [HGETALL](https://redis.io/commands/hgetall).

`HSET` también se puede ejecutar mediante una API de alto nivel mediante [hset_multiple](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.hset_multiple) que acepta una matriz de tuplas. A continuación, se ejecuta [hget](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.hget) para capturar el valor de un atributo único (en este caso, `repo`).

```rust
fn hash() {
    let mut conn = connect();

    let mut driver: BTreeMap<String, String> = BTreeMap::new();
    let prefix = "redis-driver";
    driver.insert(String::from("name"), String::from("redis-rs"));
    driver.insert(String::from("version"), String::from("0.19.0"));
    driver.insert(
        String::from("repo"),
        String::from("https://github.com/mitsuhiko/redis-rs"),
    );

    let _: () = redis::cmd("HSET")
        .arg(format!("{}:{}", prefix, "rust"))
        .arg(driver)
        .query(&mut conn)
        .expect("failed to execute HSET");

    let info: BTreeMap<String, String> = redis::cmd("HGETALL")
        .arg(format!("{}:{}", prefix, "rust"))
        .query(&mut conn)
        .expect("failed to execute HGETALL");
    println!("info for rust redis driver: {:?}", info);

    let _: () = conn
        .hset_multiple(
            format!("{}:{}", prefix, "go"),
            &[
                ("name", "go-redis"),
                ("version", "8.4.6"),
                ("repo", "https://github.com/go-redis/redis"),
            ],
        )
        .expect("failed to execute HSET");

    let repo_name: String = conn
        .hget(format!("{}:{}", prefix, "go"), "repo")
        .expect("HGET failed");
    println!("go redis driver repo name: {:?}", repo_name);
}
```

En la función siguiente, puede ver cómo usar una estructura de datos `LIST`. [LPUSH](https://redis.io/commands/lpush) se ejecuta (con la API de bajo nivel) para agregar una entrada a la lista y se usa el método de alto nivel [lpop](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.lpop) para recuperarlo de la lista. A continuación, se usa el método [rpush](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.rpush) para agregar un par de entradas a la lista, que luego se capturan mediante el método [lrange](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.lrange) de bajo nivel.

```rust
fn list() {
    let mut conn = connect();
    let list_name = "items";

    let _: () = redis::cmd("LPUSH")
        .arg(list_name)
        .arg("item-1")
        .query(&mut conn)
        .expect("failed to execute LPUSH for 'items'");

    let item: String = conn
        .lpop(list_name)
        .expect("failed to execute LPOP for 'items'");
    println!("first item: {}", item);

    let _: () = conn.rpush(list_name, "item-2").expect("RPUSH failed");
    let _: () = conn.rpush(list_name, "item-3").expect("RPUSH failed");

    let len: isize = conn
        .llen(list_name)
        .expect("failed to execute LLEN for 'items'");
    println!("no. of items in list = {}", len);

    let items: Vec<String> = conn
        .lrange(list_name, 0, len - 1)
        .expect("failed to execute LRANGE for 'items'");

    println!("listing items in list");
    for item in items {
        println!("item: {}", item)
    }
}
```

Aquí puede ver algunas de las operaciones `SET`. El método [sadd](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.sadd) (API de alto nivel) se usa para agregar un par de entradas a un objeto `SET` denominado `users`. A continuación, se ejecuta [SISMEMBER](https://redis.io/commands/hset) (API de bajo nivel) para comprobar si `user1` existe. Por último, [smembers](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.smembers) se usa para capturar y recorrer en iteración todas las entradas establecidas en forma de vector ([Vec<String>](https://doc.rust-lang.org/std/vec/struct.Vec.html)).

```rust
fn set() {
    let mut conn = connect();
    let set_name = "users";

    let _: () = conn
        .sadd(set_name, "user1")
        .expect("failed to execute SADD for 'users'");
    let _: () = conn
        .sadd(set_name, "user2")
        .expect("failed to execute SADD for 'users'");

    let ismember: bool = redis::cmd("SISMEMBER")
        .arg(set_name)
        .arg("user1")
        .query(&mut conn)
        .expect("failed to execute SISMEMBER for 'users'");
    println!("does user1 exist in the set? {}", ismember);

    let users: Vec<String> = conn.smembers(set_name).expect("failed to execute SMEMBERS");
    println!("listing users in set");

    for user in users {
        println!("user: {}", user)
    }
}
```

La función `sorted_set` siguiente muestra la estructura de datos del conjunto ordenado. Se llama al comando [ZADD](https://redis.io/commands/zadd) con la API de bajo nivel para agregar una puntuación de números enteros aleatorios para un reproductor (`player-1`). A continuación, se usa el método [zadd](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.zadd) (API de alto nivel) para agregar más reproductores (`player-2` a `player-5`) y sus puntuaciones respectivas (generadas de forma aleatoria). El número de entradas del conjunto ordenado se determina mediante [ZCARD](https://redis.io/commands/zcard). Se usa como límite para el comando [ZRANGE](https://redis.io/commands/zrange) (invocado con la API de bajo nivel) para enumerar los reproductores con sus puntuaciones en orden ascendente.

```rust
fn sorted_set() {
    let mut conn = connect();
    let sorted_set = "leaderboard";

    let _: () = redis::cmd("ZADD")
        .arg(sorted_set)
        .arg(rand::thread_rng().gen_range(1..10))
        .arg("player-1")
        .query(&mut conn)
        .expect("failed to execute ZADD for 'leaderboard'");

    for num in 2..=5 {
        let _: () = conn
            .zadd(
                sorted_set,
                String::from("player-") + &num.to_string(),
                rand::thread_rng().gen_range(1..10),
            )
            .expect("failed to execute ZADD for 'leaderboard'");
    }

    let count: isize = conn
        .zcard(sorted_set)
        .expect("failed to execute ZCARD for 'leaderboard'");

    let leaderboard: Vec<(String, isize)> = conn
        .zrange_withscores(sorted_set, 0, count - 1)
        .expect("ZRANGE failed");

    println!("listing players and scores in ascending order");

    for item in leaderboard {
        println!("{} = {}", item.0, item.1)
    }
}
```

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Comience por clonar la aplicación desde GitHub.

1. Abra un símbolo del sistema y cree una nueva carpeta llamada `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

1. Abra una ventana de terminal de Git, como Git Bash. Use `cd` para cambiar a la nueva carpeta en la que va a clonar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

1. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-redis-cache-rust-quickstart.git
    ```

## <a name="run-the-application"></a>Ejecución de la aplicación

La aplicación acepta la conectividad y las credenciales en forma de variables de entorno. 

1. Recupere el **nombre de host** y las **claves de acceso** (disponibles mediante la opción Claves de acceso) de la instancia de Azure Cache for Redis en [Azure Portal](https://portal.azure.com/).

1. Establezca estos valores en las variables de entorno respectivas:

    ```shell
    set REDIS_HOSTNAME=<Host name>:<port> (e.g. <name of cache>.redis.cache.windows.net:6380)
    set REDIS_PASSWORD=<Primary Access Key>
    ```

1. En la ventana de terminal, cambie a la carpeta correcta. Por ejemplo:

    ```shell
    cd "C:\git-samples\azure-redis-cache-rust-quickstart"
    ```

1. En el terminal, ejecute el comando siguiente para iniciar la aplicación.

    ```shell
    cargo run
    ```
    
    Verá este resultado:
    
    ```bash
    ******* Running SET, GET, INCR commands *******
    value for 'foo' = bar
    counter = 2
    ******* Running HASH commands *******
    info for rust redis driver: {"name": "redis-rs", "repo": "https://github.com/mitsuhiko/redis-rs", "version": "0.19.0"}
    go redis driver repo name: "https://github.com/go-redis/redis"
    ******* Running LIST commands *******
    first item: item-1
    no. of items in list = 2
    listing items in list
    item: item-2
    item: item-3
    ******* Running SET commands *******
    does user1 exist in the set? true
    listing users in set
    user: user2
    user: user1
    user: user3
    ******* Running SORTED SET commands *******
    listing players and scores
    player-2 = 2
    player-4 = 4
    player-1 = 7
    player-5 = 6
    player-3 = 8
    ```
    
    Si desea ejecutar una función específica, marque como comentario otras funciones de la función `main`:
    
    ```rust
    fn main() {
        basics();
        hash();
        list();
        set();
        sorted_set();
    }
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede eliminar el grupo de recursos y los recursos cuando haya terminado con ellos. Al eliminar los que creó en esta guía de inicio rápido, evitará que se le cobre por ellos.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible, y el grupo de recursos y todos los recursos que contiene se eliminarán de forma permanente. Si ha creado la instancia de Azure Cache for Redis en un grupo de recursos existente que desea conservar, puede eliminar solo la memoria caché si selecciona **Eliminar** en la página **Introducción** de la memoria caché. 

Para eliminar el grupo de recursos y su instancia de Azure Cache for Redis:

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **Grupos de recursos**.
1. En el cuadro de texto **Filtrar por nombre**, escriba el nombre del grupo de recursos que contiene la instancia de caché. A continuación, selecciónelo en los resultados de la búsqueda. 
1. En la página del grupo de recursos, seleccione **Eliminar grupo de recursos**.
1. Escriba el nombre del grupo de recursos y, a continuación, seleccione **Eliminar**.
   
   ![Eliminación del grupo de recursos para Azure Cache for Redis](./media/cache-python-get-started/delete-your-resource-group-for-azure-cache-for-redis.png)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar el controlador de Rust para Redis para conectar y ejecutar operaciones en Azure Cache for Redis.

> [!div class="nextstepaction"]
> [Creación de una aplicación web ASP.NET sencilla que use Azure Redis Cache.](./cache-web-app-howto.md)
