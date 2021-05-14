---
title: Desencadenador de blobs de Event Grid en Azure Functions
description: Obtenga información acerca de cómo realizar configuraciones y depuraciones con el desencadenador de blobs de Event Grid
author: cachai2
ms.topic: conceptual
ms.date: 3/1/2021
ms.author: cachai
ms.openlocfilehash: 259928cbfc046930d82bc3462db6d8aad4cd6b06
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125002"
---
# <a name="azure-function-event-grid-blob-trigger"></a>Desencadenador de blobs de Event Grid en Azure Functions

En este artículo se muestra cómo depurar e implementar una función local desencadenada por blobs de Event Grid que controla los eventos generados por una cuenta de almacenamiento.

> [!NOTE]
> El desencadenador de blobs de Event Grid se encuentra en versión preliminar.

## <a name="prerequisites"></a>Requisitos previos

- Cree o use una aplicación de función existente.
- Cree o use una cuenta de almacenamiento existente.
- Debe tener instalada la versión 5.0 o posteriores de la [extensión Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2).
- Descargue [ngrok](https://ngrok.com/) para permitir que Azure llame a su función local.

## <a name="create-a-new-function"></a>Creación de una función

1. Abra la aplicación de funciones en Visual Studio Code.

1. **Presione F1** para crear una nueva función de desencadenador de blobs. Asegúrese de usar la cadena de conexión de la cuenta de almacenamiento.

1. La URL predeterminada del desencadenador de blobs de Event Grid es:

    # <a name="c"></a>[C#](#tab/csharp)

    ```http
    http://localhost:7071/runtime/webhooks/blobs?functionName={functionname}
    ```

    # <a name="python"></a>[Python](#tab/python)

    ```http
    http://localhost:7071/runtime/webhooks/blobs?functionName=Host.Functions.{functionname}
    ```

    # <a name="java"></a>[Java](#tab/java)

    ```http
    http://localhost:7071/runtime/webhooks/blobs?functionName=Host.Functions.{functionname}
    ```

    ---

    Anote el nombre de la aplicación de funciones y que el tipo de desencadenador es un desencadenador de blob, que se indica mediante el valor `blobs` en la URL. Esto será necesario al configurar puntos de conexión más adelante en la guía paso a paso.

1. Una vez creada la función, agregue el parámetro de origen de Event Grid.

    # <a name="c"></a>[C#](#tab/csharp)
    Agregue **Source = BlobTriggerSource.EventGrid** a los parámetros de la función.
    
    ```csharp
    [FunctionName("BlobTriggerCSharp")]
    public static void Run([BlobTrigger("samples-workitems/{name}", Source = BlobTriggerSource.EventGrid, Connection = "connection")]Stream myBlob, string name, ILogger log)
    {
        log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
    }
    ```

    # <a name="python"></a>[Python](#tab/python)
    Agregue **"source": "EventGrid"** a los datos de enlace de function.json.
    
    ```json
    {
      "scriptFile": "__init__.py",
      "bindings": [
        {
          "name": "myblob",
          "type": "blobTrigger",
          "direction": "in",
          "path": "samples-workitems/{name}",
          "source": "EventGrid",
          "connection": "MyStorageAccountConnectionString"
        }
      ]
    }
    ```

    # <a name="java"></a>[Java](#tab/java)
    **Presione F5** para compilar la función. Una vez completada la compilación, agregue **"source": "EventGrid"** a los datos de enlace de **function.json**.
    
    ```json
    {
      "scriptFile" : "../java-1.0-SNAPSHOT.jar",
      "entryPoint" : "com.function.{MyFunctionName}.run",
      "bindings" : [ {
        "type" : "blobTrigger",
        "direction" : "in",
        "name" : "content",
        "path" : "samples-workitems/{name}",
        "dataType" : "binary",
        "source": "EventGrid",
        "connection" : "MyStorageAccountConnectionString"
       } ]
    }
    ```

    ---

1. En la función, establezca un punto de interrupción en la línea que controla el registro.

1. Inicie una sesión de depuración.

    # <a name="c"></a>[C#](#tab/csharp)
    **Presione F5** para iniciar una sesión de depuración.

    # <a name="python"></a>[Python](#tab/python)
    **Presione F5** para iniciar una sesión de depuración.

    # <a name="java"></a>[Java](#tab/java)
    Abra un nuevo terminal y ejecute el siguiente comando mvn para iniciar la sesión de depuración.

    ```bash
    mvn azure-functions:run
    ```

    ---

[!INCLUDE [functions-event-grid-local-dev](../../includes/functions-event-grid-local-dev.md)]

## <a name="debug-the-function"></a>Depurar la función
Una vez que el desencadenador de blobs reconoce que se ha cargado un nuevo archivo en el contenedor de almacenamiento, se alcanza el punto de interrupción en la función local.

## <a name="deployment"></a>Implementación

A medida que implemente la aplicación de funciones en Azure, actualice el punto de conexión de webhook desde el punto de conexión local hacia el punto de conexión de la aplicación implementada. Para actualizar un punto de conexión, siga los pasos descritos en [Adición de un evento de almacenamiento](#add-a-storage-event) y use lo siguiente para la URL de webhook en el paso 5. El parámetro `<BLOB-EXTENSION-KEY>` se puede encontrar en la sección **Claves de aplicación** del menú izquierdo de la **Aplicación de funciones**.

# <a name="c"></a>[C#](#tab/csharp)

```http
https://<FUNCTION-APP-NAME>.azurewebsites.net/runtime/webhooks/blobs?functionName=<FUNCTION-NAME>&code=<BLOB-EXTENSION-KEY>
```

# <a name="python"></a>[Python](#tab/python)

```http
https://<FUNCTION-APP-NAME>.azurewebsites.net/runtime/webhooks/blobs?functionName=Host.Functions.<FUNCTION-NAME>&code=<BLOB-EXTENSION-KEY>
```

# <a name="java"></a>[Java](#tab/java)

```http
https://<FUNCTION-APP-NAME>.azurewebsites.net/runtime/webhooks/blobs?functionName=Host.Functions.<FUNCTION-NAME>&code=<BLOB-EXTENSION-KEY>
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos creados en este artículo, elimine la suscripción de Event Grid que creó en este tutorial.

## <a name="next-steps"></a>Pasos siguientes

- [Automatización del cambio de tamaño de las imágenes cargadas mediante Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md)
- [Desencadenador de Event Grid para Azure Functions](./functions-bindings-event-grid.md)