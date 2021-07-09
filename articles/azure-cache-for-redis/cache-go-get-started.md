---
title: Uso de Azure Cache for Redis con Go
description: En este inicio rápido, aprenderá a crear una aplicación de Go que usa Azure Cache for Redis.
author: abhirockzz
ms.author: abhishgu
ms.service: cache
ms.devlang: go
ms.topic: quickstart
ms.date: 01/08/2021
ms.openlocfilehash: 5303289cd0629fea1c78e4ae746427e875f80520
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111891642"
---
# <a name="quickstart-use-azure-cache-for-redis-with-go"></a>Inicio rápido: Uso de Azure Cache for Redis con Go

En este artículo, aprenderá a compilar una API REST en Go que almacenará y recuperará la información de usuario respaldada por una estructura de datos [HASH](https://redis.io/topics/data-types-intro#redis-hashes) en [Azure Cache for Redis](./cache-overview.md).

## <a name="skip-to-the-code-on-github"></a>Ir al código en GitHub

Si quiere pasar directamente al código, consulte [Inicio rápido: Uso de Azure Cache for Redis con Go](https://github.com/Azure-Samples/azure-redis-cache-go-quickstart/) en GitHub.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- [Go](https://golang.org/doc/install) (preferiblemente la versión 1.13 o superior)
- [Git](https://git-scm.com/downloads)
- Un cliente HTTP, como [curl](https://curl.se/)

## <a name="create-an-azure-cache-for-redis-instance"></a>Creación de una instancia de Azure Redis Cache

[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

## <a name="review-the-code-optional"></a>Revisión del código (opcional)

Si le interesa saber cómo funciona el código, puede revisar los siguientes fragmentos de código. De lo contrario, puede pasar a [ejecutar la aplicación](#run-the-application).

La biblioteca [go-redis](https://github.com/go-redis/redis) de código abierto se usa para interactuar con Azure Cache for Redis.

La función `main` comienza leyendo el nombre de host y la contraseña (clave de acceso) de la instancia de Azure Cache for Redis.

```go
func main() {
    redisHost := os.Getenv("REDIS_HOST")
    redisPassword := os.Getenv("REDIS_PASSWORD")
...
```

A continuación, se establece la conexión con Azure Cache for Redis. Usamos [tls.Config](https://golang.org/pkg/crypto/tls/#Config); Azure Cache for Redis solo acepta conexiones seguras con [TLS 1.2, la versión mínima exigida](cache-remove-tls-10-11.md).

```go
...
op := &redis.Options{Addr: redisHost, Password: redisPassword, TLSConfig: &tls.Config{MinVersion: tls.VersionTLS12}}
client := redis.NewClient(op)

ctx := context.Background()
err := client.Ping(ctx).Err()
if err != nil {
    log.Fatalf("failed to connect with redis instance at %s - %v", redisHost, err)
}
...
```

Si la conexión es correcta, se configuran [controladores HTTP](https://golang.org/pkg/net/http/#HandleFunc) para administrar las operaciones `POST` y `GET` y se inicia el servidor HTTP.

> [!NOTE]
> La [biblioteca gorilla mux](https://github.com/gorilla/mux) se usa para el enrutamiento (aunque no es estrictamente necesario y podríamos haber hecho uso de la biblioteca estándar para esta aplicación de ejemplo).
>

```go
uh := userHandler{client: client}

router := mux.NewRouter()
router.HandleFunc("/users/", uh.createUser).Methods(http.MethodPost)
router.HandleFunc("/users/{userid}", uh.getUser).Methods(http.MethodGet)

log.Fatal(http.ListenAndServe(":8080", router))
```

La estructura `userHandler` encapsula un elemento [redis.Client](https://pkg.go.dev/github.com/go-redis/redis/v8#Client), que usan los métodos `createUser` y `getUser`; por razones de brevedad, el código de estos métodos no se ha incluido.

- `createUser`: acepta una carga JSON (que contiene información de usuario) y la guarda como `HASH` en Azure Cache for Redis.
- `getUser`: captura la información de usuario de `HASH` o devuelve una respuesta HTTP `404` si no se encuentra.

```go
type userHandler struct {
    client *redis.Client
}
...

func (uh userHandler) createUser(rw http.ResponseWriter, r *http.Request) {
    // details omitted
}
...

func (uh userHandler) getUser(rw http.ResponseWriter, r *http.Request) {
    // details omitted
}
```

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Comience por clonar la aplicación desde GitHub.

1. Abra un símbolo del sistema y cree una nueva carpeta llamada `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

1. Abra una ventana de terminal de Git, como Git Bash. Use el comando `cd` para cambiar a la nueva carpeta en la que va a clonar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

1. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo.

    ```bash
    git clone https://github.com/Azure-Samples/azure-redis-cache-go-quickstart.git
    ```

## <a name="run-the-application"></a>Ejecución de la aplicación

La aplicación acepta la conectividad y las credenciales en forma de variables de entorno.

1. Recupere el **nombre de host** y las **claves de acceso** (disponibles mediante la opción Claves de acceso) de la instancia de Azure Cache for Redis en [Azure Portal](https://portal.azure.com/).

1. Establezca estos valores en las variables de entorno respectivas:

    ```shell
    set REDIS_HOST=<Host name>:<port> (e.g. <name of cache>.redis.cache.windows.net:6380)
    set REDIS_PASSWORD=<Primary Access Key>
    ```

1. En la ventana de terminal, cambie a la carpeta correcta. Por ejemplo:

    ```shell
    cd "C:\git-samples\azure-redis-cache-go-quickstart"
    ```

1. En el terminal, ejecute el comando siguiente para iniciar la aplicación.

    ```shell
    go run main.go
    ```

El servidor HTTP se iniciará en el puerto `8080`.

## <a name="test-the-application"></a>Prueba de la aplicación

1. Cree algunas entradas de usuario. En el ejemplo siguiente se usa curl:

    ```bash
    curl -i -X POST -d '{"id":"1","name":"foo1", "email":"foo1@baz.com"}' localhost:8080/users/
    curl -i -X POST -d '{"id":"2","name":"foo2", "email":"foo2@baz.com"}' localhost:8080/users/
    curl -i -X POST -d '{"id":"3","name":"foo3", "email":"foo3@baz.com"}' localhost:8080/users/
    ```

1. Captura de un usuario existente con su `id`:

    ```bash
    curl -i localhost:8080/users/1
    ```

    Debería obtener una respuesta JSON, como:

    ```json
    {
        "email": "foo1@bar",
        "id": "1",
        "name": "foo1"
    }
    ```

1. Si intenta capturar un usuario que no existe, obtendrá una respuesta HTTP `404`. Por ejemplo:

    ```bash
    curl -i localhost:8080/users/100
    
    #response

    HTTP/1.1 404 Not Found
    Date: Fri, 08 Jan 2021 13:43:39 GMT
    Content-Length: 0
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya ha terminado con el grupo de recursos y los recursos de Azure que creó en esta guía de inicio rápido, puede eliminarlos para evitar cargos.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible, y el grupo de recursos y todos los recursos que contiene se eliminarán de forma permanente. Si ha creado la instancia de Azure Cache for Redis en un grupo de recursos existente que desea conservar, puede eliminar solo la memoria caché si selecciona **Eliminar** en la página **Introducción** de la memoria caché.

Para eliminar el grupo de recursos y su instancia de Azure Cache for Redis:

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **Grupos de recursos**.
1. En el cuadro de texto **Filtrar por nombre**, escriba el nombre del grupo de recursos que contiene la instancia de caché y, a continuación, selecciónela en los resultados de la búsqueda.
1. En la página del grupo de recursos, seleccione **Eliminar grupo de recursos**.
1. Escriba el nombre del grupo de recursos y, a continuación, seleccione **Eliminar**.

   ![Eliminación del grupo de recursos para Azure Cache for Redis](./media/cache-python-get-started/delete-your-resource-group-for-azure-cache-for-redis.png)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido los primeros pasos para usar Go con Azure Cache for Redis. Ha configurado y ejecutado una aplicación sencilla basada en la API REST para crear y obtener información de usuario respaldada por una estructura de datos `HASH` de Redis.

> [!div class="nextstepaction"]
> [Creación de una aplicación web ASP.NET sencilla que use Azure Redis Cache.](./cache-web-app-howto.md)
