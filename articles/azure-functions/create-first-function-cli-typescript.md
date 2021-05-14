---
title: 'Creación de una función de TypeScript desde la línea de comandos: Azure Functions'
description: Aprenda a crear una función de TypeScript desde la línea de comandos y, luego, a publicar el proyecto local en el hospedaje sin servidor en Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: c7b37700d93ea129ba46de81989f06dd2afe52ba
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831823"
---
# <a name="quickstart-create-a-typescript-function-in-azure-from-the-command-line"></a>Inicio rápido: Creación de una función de TypeScript en Azure desde la línea de comandos

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

En este artículo se usan herramientas de línea de comandos para crear una función de TypeScript que responda a solicitudes HTTP. Después de probar el código localmente, se implementa en el entorno sin servidor de Azure Functions.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

También hay una [versión basada en Visual Studio Code](create-first-function-vs-code-typescript.md) de este artículo.

## <a name="configure-your-local-environment"></a>Configuración del entorno local

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x.

+ Una de las siguientes herramientas para crear recursos de Azure:

    + [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

    + [Azure PowerShell](/powershell/azure/install-az-ps), versión 5.0 o posterior.

+ Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

Compruebe los requisitos previos, que dependen de si usa la CLI de Azure o Azure PowerShell para crear recursos de Azure:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `az --version` para comprobar que la versión de la CLI de Azure es la 2.4 o posterior.

+ Ejecute `az login` para iniciar sesión en Azure y comprobar una suscripción activa.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `(Get-Module -ListAvailable Az).Version` y compruebe que es la versión 5.0 o posterior. 

+ Ejecute `Connect-AzAccount` para iniciar sesión en Azure y comprobar una suscripción activa.

---

## <a name="create-a-local-function-project"></a>Creación de un proyecto de función local

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

1. Ejecute el comando `func init`, de la manera siguiente, para crear un proyecto de funciones en una carpeta llamada *LocalFunctionProj* con el entorno de ejecución especificado:  

    ```console
    func init LocalFunctionProj --typescript
    ```

1. Vaya a la carpeta del proyecto:

    ```console
    cd LocalFunctionProj
    ```
    
    Esta carpeta contiene varios archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). Como *local.settings.json* puede contener secretos descargados de Azure, el archivo se excluye del control de código fuente de forma predeterminada en el archivo *.gitignore*.

1. Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función (HttpExample) y el argumento `--template` especifica el desencadenador de esta (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ``` 
    
    `func new` crea una subcarpeta que coincide con el nombre de la función que contiene un archivo de código apropiado para el lenguaje elegido del proyecto y un archivo de configuración denominado *function.json*.

### <a name="optional-examine-the-file-contents"></a>(Opcional) Examen del contenido del archivo

Si lo desea, puede ir a [Ejecución local de la función](#run-the-function-locally) y examine el contenido del archivo posteriormente.

#### <a name="indexts"></a>index.ts

*index.ts* exporta una función que se desencadena de acuerdo con la configuración de *function.json*.

:::code language="typescript" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-TypeScript/index.ts":::

Con un desencadenador HTTP, la función recibe datos de solicitud en la variable `req` de tipo **HttpRequest**, como se define en *function.json*. El objeto devuelto, definido como `$return` en *function.json*, es la respuesta. 

#### <a name="functionjson"></a>function.json

*function.json* es un archivo de configuración que define la entrada y salida `bindings` de la función, lo que incluye el tipo de desencadenador. 

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/function.json":::

Cada enlace requiere una dirección, un tipo y un nombre único. El desencadenador HTTP tiene un enlace de entrada de tipo [`httpTrigger`](functions-bindings-http-webhook-trigger.md) y un enlace de salida de tipo [`http`](functions-bindings-http-webhook-output.md).

## <a name="run-the-function-locally"></a>Ejecución local de la función

1. Para ejecutar una función, inicie el host en tiempo de ejecución local de Azure Functions desde la carpeta *LocalFunctionProj*:

    ```console
    npm install
    npm start
    ```
    
    Hacia el final de la salida, deberían aparecer las líneas siguientes:
    
    <pre>
    ...
    
    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.
    
    Http Functions:
    
            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    
    </pre>
    
    >[!NOTE]  
    > Si HttpExample no aparece como se muestra a continuación, es probable que haya iniciado el host desde fuera de la carpeta raíz del proyecto. En ese caso, use **Ctrl**+**C** para detener el host, vaya a la carpeta raíz del proyecto y vuelva a ejecutar el comando anterior.

1. Copie la dirección URL de la función `HttpExample` de esta salida en un explorador y anexe la cadena de consulta `?name=<your-name>`, lo que hará que la dirección URL completa sea `http://localhost:7071/api/HttpExample?name=Functions`. El explorador debe mostrar un mensaje como este `Hello Functions`:

    ![Resultado de la ejecución de la función localmente en el explorador](./media/functions-create-first-azure-function-azure-cli/function-test-local-browser.png)
    
    El terminal en el que inició el proyecto también muestra la salida del registro cuando realiza solicitudes.

1. Cuando esté listo, presione **Ctrl**+**C** y elija `y` para detener el host de Functions.

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Cree la aplicación de funciones en Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime node --runtime-version 12 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    El comando [az functionapp create](/cli/azure/functionapp#az_functionapp_create) crea la aplicación de funciones en Azure. Si usa Node.js 10, cambie también `--runtime-version` a `10`.
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime node -RuntimeVersion 12 -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    El cmdlet [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) crea la aplicación de funciones en Azure. Si usa Node.js 10, cambie `-RuntimeVersion` a `10`.

    ---
        
    En el ejemplo anterior, reemplace `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior y reemplace `<APP_NAME>` por un nombre único global que le resulte adecuado. `<APP_NAME>` también es el dominio DNS predeterminado de la aplicación de función. 
    
    Este comando crea una aplicación de funciones que se ejecuta en el entorno de ejecución del lenguaje especificado en el [plan de consumo de Azure Functions](consumption-plan.md), que es gratuito para la cantidad de uso que se realiza aquí. El comando también aprovisiona una instancia asociada de Azure Application Insights en el mismo grupo de recursos con la que puede supervisar la aplicación de funciones y ver registros. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md). La instancia no incurrirá en ningún costo hasta que se active.

## <a name="deploy-the-function-project-to-azure"></a>Implementación del proyecto de función en Azure

Antes de usar Core Tools para implementar el proyecto en Azure, creará una compilación lista para producción de archivos de JavaScript a partir de los archivos de origen de TypeScript.

1. Use el siguiente comando para preparar el proyecto de TypeScript para la implementación:

    ```console
    npm run build:production 
    ```

1. Con los recursos necesarios en vigor, ya está listo para implementar el proyecto de funciones local en la aplicación de funciones de Azure mediante el comando [func azure functionapp publish](functions-run-local.md#project-file-deployment). En el ejemplo siguiente, reemplace `<APP_NAME>` por el nombre de su aplicación.

    ```console
    func azure functionapp publish <APP_NAME>
    ```
    
    Si aparece el error "Can't find app with name..." (No se encuentra la aplicación...), espere unos segundos e inténtelo de nuevo, ya que es posible que Azure no haya inicializado completamente la aplicación después del comando `az functionapp create` anterior.
    
    El comando de publicación muestra un resultado similar a lo que se muestra a continuación (se ha truncado para que resulte más simple):
    
    <pre>
    ...
    
    Getting site publishing info...
    Creating archive for current directory...
    Performing remote build for functions project.
    
    ...
    
    Deployment successful.
    Remote build succeeded!
    Syncing triggers...
    Functions in msdocs-azurefunctions-qs:
        HttpExample - [httpTrigger]
            Invoke url: https://msdocs-azurefunctions-qs.azurewebsites.net/api/httpexample?code=KYHrydo4GFe9y0000000qRgRJ8NdLFKpkakGJQfC3izYVidzzDN4gQ==
    </pre>

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-typescript)
