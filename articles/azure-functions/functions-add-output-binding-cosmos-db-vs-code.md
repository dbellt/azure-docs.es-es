---
title: Conexión de Azure Functions a Azure Cosmos DB mediante Visual Studio Code
description: Aprenda a conectar Azure Functions a una cuenta de Azure Cosmos DB mediante la incorporación de un enlace de salida al proyecto de Visual Studio Code.
author: ThomasWeiss
ms.date: 03/23/2021
ms.topic: quickstart
ms.author: thweiss
zone_pivot_groups: programming-languages-set-functions-temp
ms.openlocfilehash: 884ed08dc6dc6587b4f464c18e8231a7f5342422
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112234555"
---
# <a name="connect-azure-functions-to-azure-cosmos-db-using-visual-studio-code"></a>Conexión de Azure Functions a Azure Cosmos DB mediante Visual Studio Code

[!INCLUDE [functions-add-storage-binding-intro](../../includes/functions-add-storage-binding-intro.md)]

En este artículo se muestra cómo usar Visual Studio Code para conectar [Microsoft Azure Cosmos DB](../cosmos-db/introduction.md) a la función que creó en el artículo de inicio rápido anterior. El enlace de salida que se agrega a esta función escribe datos de la solicitud HTTP en un documento JSON almacenado en un contenedor de Azure Cosmos DB. 

::: zone pivot="programming-language-csharp"
Antes de empezar, debe completar el [Inicio rápido: Creación de una función de C# en Azure mediante Visual Studio Code](create-first-function-vs-code-csharp.md). Si ya ha limpiado los recursos al final de ese artículo, vuelva a recorrer los pasos para crear de nuevo la aplicación de función y los recursos relacionados en Azure.
::: zone-end
::: zone pivot="programming-language-javascript"  
Antes de empezar, debe completar el [Inicio rápido: Creación de una función de JavaScript en Azure mediante Visual Studio Code](create-first-function-vs-code-node.md). Si ya ha limpiado los recursos al final de ese artículo, vuelva a recorrer los pasos para crear de nuevo la aplicación de función y los recursos relacionados en Azure.  
::: zone-end

## <a name="configure-your-environment"></a>Configurar su entorno

Antes de empezar, asegúrese de instalar la [extensión Azure Databases](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb) para Visual Studio Code.

## <a name="create-your-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

> [!IMPORTANT]
> [Azure Cosmos DB sin servidor](../cosmos-db/serverless.md) está actualmente disponible en versión preliminar. Este modo basado en el consumo hace que Azure Cosmos DB sea una opción segura para las cargas de trabajo sin servidor. Para usar Azure Cosmos DB en modo sin servidor, elija **Sin servidor** en **Capacity mode** (Modo de capacidad) al crear la cuenta.

1. En una nueva ventana del explorador, inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Haga clic en **Crear un recurso** > **Bases de datos** > **Azure Cosmos DB**.
   
    :::image type="content" source="../cosmos-db/includes/media/cosmos-db-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png" alt-text="El panel de las bases de datos de Azure Portal" border="true":::

3. En la página **Create Azure Cosmos DB Account** (Creación de una cuenta de Azure Cosmos DB), especifique la configuración de la nueva cuenta de Azure Cosmos DB. 
 
    Configuración|Value|Descripción
    ---|---|---
    Subscription|*Su suscripción*|Elija la suscripción de Azure en la que creó la instancia de Function App en el [artículo anterior](./create-first-function-vs-code-csharp.md).
    Grupo de recursos|*El grupo de recursos*|Elija el grupo de recursos en el que creó la instancia de Function App en el [artículo anterior](./create-first-function-vs-code-csharp.md).
    Nombre de cuenta|*Escriba un nombre único*|Escriba un nombre único para identificar la cuenta de Azure Cosmos DB.<br><br>El nombre de la cuenta solo puede utilizar letras minúsculas, números y guiones (-), y debe tener entre 3 y 31 caracteres de longitud.
    API|Core (SQL)|Seleccione **Core (SQL)** para crear una base de datos de documentos en la que se puedan realizar consultas mediante una sintaxis de SQL. [Más información sobre la SQL API de Azure Cosmos DB](../cosmos-db/introduction.md).|
    Location|*Seleccione la región más cercana a su ubicación*|Seleccione una ubicación geográfica para hospedar la cuenta de Azure Cosmos DB. Use la ubicación que esté más cercana a usted o a los usuarios para obtener el acceso más rápido a los datos.
    Capacity mode (Modo de capacidad)|Rendimiento aprovisionado o sin servidor|Seleccione **Serverless** (Sin servidor) para crear una cuenta en modo [sin servidor](../cosmos-db/serverless.md). Seleccione **Provisioned throughput** (Rendimiento aprovisionado) para crear una cuenta en modo de [rendimiento aprovisionado](../cosmos-db/set-throughput.md).<br><br>Elija sin **Serverless** (Sin servidor) si va a empezar a trabajar con Azure Cosmos DB.

4. Haga clic en **Revisar y crear**. Puede omitir las secciones **Red** y **Etiquetas**. 

5. Revise la información de resumen y haga clic en **Crear**. 

6. Espere hasta que se cree una base de datos de Azure Cosmos DB y seleccione **Go to resource** (Ir al recurso).

    :::image type="content" source="../cosmos-db/media/create-cosmosdb-resources-portal/azure-cosmos-db-account-deployment-successful.png" alt-text="Se ha completado la creación de la cuenta de Azure Cosmos DB" border="true":::

## <a name="create-an-azure-cosmos-db-database-and-container"></a>Creación de un contenedor y una base de datos de Azure Cosmos DB

En la cuenta de Azure Cosmos DB, seleccione **Explorador de datos** y luego **Nuevo contenedor**. Cree una base de datos denominada *my-database*, un contenedor denominado *my-container* y elija `/id` como [clave de partición](../cosmos-db/partitioning-overview.md).

:::image type="content" source="./media/functions-add-output-binding-cosmos-db-vs-code/create-container.png" alt-text="Creación de un contenedor de Azure Cosmos DB desde Azure Portal" border="true":::

## <a name="update-your-function-app-settings"></a>Actualización de la configuración de la aplicación de funciones

En el [artículo del inicio rápido anterior](./create-first-function-vs-code-csharp.md), creó una aplicación de funciones en Azure. En este artículo, se actualiza una instancia de Function App para escribir documentos JSON en el contenedor de Azure Cosmos DB que ha creado anteriormente. Para conectarse a una cuenta de Azure Cosmos DB, debe agregar su cadena de conexión a la configuración de la aplicación. Después, descargue la nueva configuración en el archivo local.settings.json para que pueda conectarse a su cuenta de Azure Cosmos DB cuando realice la ejecución en un entorno local.

1. En Visual Studio Code, busque la cuenta de Azure Cosmos DB que acaba de crear. Haga clic con el botón derecho en su nombre y seleccione **Copy Connection String** (Copiar cadena de conexión).

    :::image type="content" source="./media/functions-add-output-binding-cosmos-db-vs-code/copy-connection-string.png" alt-text="Copia de la cadena de conexión de Azure Cosmos DB" border="true":::

1. Presione <kbd>F1</kbd> para abrir la paleta de comandos y busque y ejecute el comando `Azure Functions: Add New Setting...`.

1. Elija la aplicación de funciones que creó en el artículo anterior. Escriba la siguiente información cuando se le indique:

    + **Escriba el nombre de la nueva configuración de la aplicación**: escriba `CosmosDbConnectionString`.

    + **Escriba el valor de "CosmosDbConnectionString"** : pegue la cadena de conexión de la cuenta de Azure Cosmos DB que copió anteriormente.

1. Vuelva a presionar <kbd>F1</kbd> para abrir la paleta de comandos y busque y ejecute el comando `Azure Functions: Download Remote Settings...`. 

1. Elija la aplicación de funciones que creó en el artículo anterior. Seleccione **Sí a todo** para sobrescribir la configuración local existente. 

## <a name="register-binding-extensions"></a>Registro de extensiones de enlace

Dado que va a utilizar un enlace de salida de Azure Cosmos DB, para poder ejecutar el proyecto debe tener instalada la extensión de enlaces correspondiente. 

::: zone pivot="programming-language-csharp"

A excepción de los desencadenadores HTTP y el temporizador, los enlaces se implementan como paquetes de extensión. Ejecute el siguiente comando [dotnet add package](/dotnet/core/tools/dotnet-add-package) en la ventana Terminal para agregar el paquete de extensión de Storage al proyecto.

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB
```

::: zone-end

::: zone pivot="programming-language-javascript"

El proyecto se ha configurado para usar [conjuntos de extensiones](functions-bindings-register.md#extension-bundles), que instalan automáticamente un conjunto predefinido de paquetes de extensiones. 

El uso de conjuntos de extensiones se habilita en el archivo host.json en la raíz del proyecto, que tiene un aspecto similar al siguiente:

:::code language="json" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/host.json":::

::: zone-end

Ya puede agregar el enlace de salida de Azure Cosmos DB a su proyecto.

## <a name="add-an-output-binding"></a>Adición de un enlace de salida

En Functions, para cada tipo de enlace es necesario definir los elementos `direction`, `type` y un valor único de `name` en el archivo function.json. La manera de definir estos atributos depende del lenguaje de la aplicación de funciones.

::: zone pivot="programming-language-csharp"

En un proyecto de biblioteca de clases de C#, los enlaces se definen como atributos de enlace en el método de función. Después se genera automáticamente el archivo *function.json*, que necesita Functions, en función de estos atributos.

Abra el archivo de proyecto *HttpExample.cs* y agregue el parámetro siguiente a la definición del método `Run`:

```csharp
[CosmosDB(
    databaseName: "my-database",
    collectionName: "my-container",
    ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
```

El parámetro `documentsOut` es del tipo IAsyncCollector<T>, que representa una colección de documentos JSON que se escribirán en el contenedor de Azure Cosmos DB cuando finaliza la función. Atributos específicos especifica el nombre del contenedor y el nombre de su base de datos primaria. La cadena de conexión de la cuenta de Azure Cosmos DB la establece `ConnectionStringSettingAttribute`.

La definición del método Run debe ahora parecerse a la siguiente:  

```csharp
[FunctionName("HttpExample")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
    [CosmosDB(
        databaseName: "my-database",
        collectionName: "my-container",
        ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
    ILogger log)
```

::: zone-end

::: zone pivot="programming-language-javascript"

Los atributos de enlace se definen directamente en el archivo function.json. Según el tipo de enlace, pueden ser necesarias propiedades adicionales. La [configuración de salida de Azure Cosmos DB](./functions-bindings-cosmosdb-v2-output.md#configuration) describe los campos necesarios para un enlace de salida de Azure Cosmos DB. La extensión facilita la incorporación de enlaces al archivo function.json. 

Para crear un enlace, haga clic con el botón derecho (Ctrl + clic en macOS) en el archivo `function.json` de la carpeta HttpTrigger y elija **Agregar enlace...** Siga las indicaciones para definir las siguientes propiedades de enlace para el nuevo enlace:

| Prompt | Value | Descripción |
| -------- | ----- | ----------- |
| **Select binding direction** (Seleccionar dirección de enlace) | `out` | El enlace es un enlace de salida. |
| **Select binding with direction "out" (Seleccionar enlace con dirección de salida)** | `Azure Cosmos DB` | El enlace es de Azure Cosmos DB. |
| **The name used to identify this binding in your code** (Nombre identificativo del enlace en el código) | `outputDocument` | Nombre que identifica el parámetro de enlace al que se hace referencia en el código. |
| **La base de datos de Cosmos DB en la que se escribirán los datos** | `my-database` | El nombre de la base de datos de Azure Cosmos DB que contiene el contenedor de destino. |
| **Colección de bases de datos en la que se escribirán los datos** | `my-container` | El nombre del contenedor de Azure Cosmos DB en que se escribirán los documentos JSON. |
| **Si es true, crea la base de datos y la colección de Cosmos DB** | `false` | El contenedor y la base de datos de destino ya existen. |
| **Select setting from "local.setting.json"** (Seleccionar configuración de "local.setting.json") | `CosmosDbConnectionString` | El nombre de una configuración de la aplicación que contiene la cadena de conexión de la cuenta de Azure Cosmos DB. |
| **Clave de partición (opcional)** | *déjelo en blanco* | Solo se requiere cuando el enlace de salida crea el contenedor. |
| **Rendimiento de la colección (opcional)** | *déjelo en blanco* | Solo se requiere cuando el enlace de salida crea el contenedor. |

Se agrega un enlace a la matriz `bindings` en function.json, que debería ser similar al siguiente:

```json
{
    "type": "cosmosDB",
    "direction": "out",
    "name": "outputDocument",
    "databaseName": "my-database",
    "collectionName": "my-container",
    "createIfNotExists": "false",
    "connectionStringSetting": "CosmosDbConnectionString"
}
```

::: zone-end

## <a name="add-code-that-uses-the-output-binding"></a>Adición de código que utilice el enlace de salida

::: zone pivot="programming-language-csharp"  

Agregue código que use el objeto de enlace de salida `documentsOut` para crear un documento JSON. Agregue este código antes de la devolución del método.

```csharp
if (!string.IsNullOrEmpty(name))
{
    // Add a JSON document to the output container.
    await documentsOut.AddAsync(new
    {
        // create a random ID
        id = System.Guid.NewGuid().ToString(),
        name = name
    });
}
```

En este momento, la función debe tener el aspecto siguiente:

```csharp
[FunctionName("HttpExample")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
    [CosmosDB(
        databaseName: "my-database",
        collectionName: "my-container",
        ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    if (!string.IsNullOrEmpty(name))
    {
        // Add a JSON document to the output container.
        await documentsOut.AddAsync(new
        {
            // create a random ID
            id = System.Guid.NewGuid().ToString(),
            name = name
        });
    }

    string responseMessage = string.IsNullOrEmpty(name)
        ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
        : $"Hello, {name}. This HTTP triggered function executed successfully.";

    return new OkObjectResult(responseMessage);
}
```

::: zone-end

::: zone pivot="programming-language-javascript"  

Agregue código que use el objeto de enlace de salida `outputDocument` en `context.bindings` para crear un documento JSON. Agregue este código antes de la instrucción `context.res`.

```javascript
if (name) {
    context.bindings.outputDocument = JSON.stringify({
        // create a random ID
        id: new Date().toISOString() + Math.random().toString().substr(2,8),
        name: name
    });
}
```

En este momento, la función debe tener el aspecto siguiente:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

    if (name) {
        context.bindings.outputDocument = JSON.stringify({
            // create a random ID
            id: new Date().toISOString() + Math.random().toString().substr(2,8),
            name: name
        });
    }

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responseMessage
    };
}
```

::: zone-end  

## <a name="run-the-function-locally"></a>Ejecución local de la función

1. Como en el artículo anterior, presione <kbd>F5</kbd> para iniciar el proyecto de aplicación de función y Core Tools. 

1. Mientras se ejecuta Core Tools, vaya al área de **Azure: Área Functions**. En **Functions**, expanda **Proyecto local** > **Functions**. Haga clic con el botón derecho (Ctrl + clic en Mac) en la función `HttpExample` y elija **Execute Function Now...** (Ejecutar la función ahora...).

    :::image type="content" source="../../includes/media/functions-run-function-test-local-vs-code/execute-function-now.png" alt-text="Ejecutar la función ahora desde Visual Studio Code":::

1. En **Enter request body** (Especificar el cuerpo de la solicitud) verá el valor del cuerpo del mensaje de solicitud de `{ "name": "Azure" }`. Presione Entrar para enviar este mensaje de solicitud a la función.  
 
1. Una vez que se haya devuelto una respuesta, presione <kbd>Ctrl + C</kbd> para detener Core Tools.

### <a name="verify-that-a-json-document-has-been-created"></a>Comprobación de que se ha creado un documento JSON

1. En Azure Portal, vuelva a su cuenta de Azure Cosmos DB y seleccione **Explorador de datos**.

1. Expanda la base de datos y el contenedor, y seleccione **Elementos** para enumerar los documentos creados en el contenedor.

1. Compruebe que el enlace de salida ha creado un documento JSON.

    :::image type="content" source="./media/functions-add-output-binding-cosmos-db-vs-code/verify-output.png" alt-text="Comprobando que se ha creado un documento en el contenedor de Azure Cosmos DB" border="true":::

## <a name="redeploy-and-verify-the-updated-app"></a>Reimplementación y comprobación de la aplicación actualizada

1. En Visual Studio Code, presione F1 para abrir la paleta de comandos. En la paleta de comandos, busque y seleccione `Azure Functions: Deploy to function app...`.

1. Elija la aplicación de funciones que creó en el primer artículo. Dado que va a volver a implementar el proyecto en la misma aplicación, seleccione **Implementar** para descartar la advertencia sobre la sobrescritura de archivos.

1. Una vez finalizada la implementación, puede volver a usar la característica **Execute Function Now...** (Ejecutar la función ahora...) para desencadenar la función en Azure.

1. Vuelva a [comprobar los documentos creados en el contenedor de Azure Cosmos DB](#verify-that-a-json-document-has-been-created) para tener la certeza de que el enlace de salida vuelve a generar un nuevo documento JSON.

## <a name="clean-up-resources"></a>Limpieza de recursos

En Azure, los *recursos* son aplicaciones de funciones, funciones o cuentas de almacenamiento, entre otros. Se agrupan en *grupos de recursos* y se puede eliminar todo el contenido de un grupo si este se elimina.

Ha creado recursos para completar estas guías de inicio rápido. Se le pueden facturar por estos recursos, dependiendo del [estado de la cuentade los ](https://azure.microsoft.com/account/) y [precios de los servicios](https://azure.microsoft.com/pricing/). Si ya no necesita los recursos, aquí se indica cómo eliminarlos:

[!INCLUDE [functions-cleanup-resources-vs-code-inner.md](../../includes/functions-cleanup-resources-vs-code-inner.md)]

## <a name="next-steps"></a>Pasos siguientes

Ha actualizado la función desencadenada por HTTP para escribir documentos JSON en un contenedor de Azure Cosmos DB. Ahora, puede obtener más información sobre el desarrollo de Functions mediante Visual Studio Code:

+ [Desarrollo de Azure Functions mediante Visual Studio Code](functions-develop-vs-code.md)

+ [Enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md).
::: zone pivot="programming-language-csharp"  
+ [Ejemplos de proyectos de Function completos en C#](/samples/browse/?products=azure-functions&languages=csharp).

+ [Referencia para desarrolladores de C# de Azure Functions](functions-dotnet-class-library.md)  
::: zone-end 
::: zone pivot="programming-language-javascript"  
+ [Ejemplos de proyectos de Function completos en JavaScript](/samples/browse/?products=azure-functions&languages=javascript).

+ [Guía para el desarrollador de JavaScript para Azure Functions](functions-reference-node.md)  
::: zone-end  