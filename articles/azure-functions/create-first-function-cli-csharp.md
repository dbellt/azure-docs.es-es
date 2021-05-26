---
title: 'Creación de una función de C# desde la línea de comandos: Azure Functions'
description: Aprenda a crear una función de C# desde la línea de comandos y, luego, a publicar el proyecto local en el hospedaje sin servidor en Azure Functions.
ms.date: 10/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-csharp
- devx-track-azurecli
- devx-track-azurepowershell
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-cli-csharp-ieux
ms.openlocfilehash: c5366587e11018b57142a2ed8ebf60c42a03ef4b
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369602"
---
# <a name="quickstart-create-a-c-function-in-azure-from-the-command-line"></a>Inicio rápido: Creación de una función de C# en Azure desde la línea de comandos

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

En este artículo se usan herramientas de línea de comandos para crear una función basada en biblioteca de clases de C# que responda a solicitudes HTTP. Después de probar el código localmente, se implementa en el entorno sin servidor de Azure Functions.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

También hay una [versión basada en Visual Studio Code](create-first-function-vs-code-csharp.md) de este artículo.

## <a name="configure-your-local-environment"></a>Configuración del entorno local

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download)

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x.

+ Una de las siguientes herramientas para crear recursos de Azure:

    + [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

    + [Azure PowerShell](/powershell/azure/install-az-ps), versión 5.0 o posterior.

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

Compruebe los requisitos previos, que dependen de si usa la CLI de Azure o Azure PowerShell para crear recursos de Azure:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `az --version` para comprobar que la versión de la CLI de Azure es la 2.4 o posterior.

+ Ejecute `az login` para iniciar sesión en Azure y comprobar una suscripción activa.

+ Ejecute `dotnet --list-sdks` para comprobar que SDK de .NET Core versión 3.1.x está instalado

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `(Get-Module -ListAvailable Az).Version` y compruebe que es la versión 5.0 o posterior. 

+ Ejecute `Connect-AzAccount` para iniciar sesión en Azure y comprobar una suscripción activa.

+ Ejecute `dotnet --list-sdks` para comprobar que SDK de .NET Core versión 3.1.x está instalado

---

## <a name="create-a-local-function-project"></a>Creación de un proyecto de función local

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

1. Ejecute el comando `func init`, de la manera siguiente, para crear un proyecto de funciones en una carpeta llamada *LocalFunctionProj* con el entorno de ejecución especificado:  

    ```csharp
    func init LocalFunctionProj --dotnet
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

    `func new` crea un archivo de código HttpExample.cs.

### <a name="optional-examine-the-file-contents"></a>(Opcional) Examen del contenido del archivo

Si lo desea, puede ir a [Ejecución local de la función](#run-the-function-locally) y examine el contenido del archivo posteriormente.

#### <a name="httpexamplecs"></a>HttpExample.cs

*HttpExample.cs* contiene un método `Run` que recibe datos de solicitud en la variable `req` de tipo [HttpRequest](/dotnet/api/microsoft.aspnetcore.http.httprequest) que está decorado con **HttpTriggerAttribute**, que define el comportamiento del desencadenador.

:::code language="csharp" source="~/functions-docs-csharp/http-trigger-template/HttpExample.cs":::

El objeto de devolución es un objeto [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) que devuelve un mensaje de respuesta [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200) o [BadRequestObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestobjectresult) (400). Para más información, vea [Enlaces y desencadenadores HTTP de Azure Functions](./functions-bindings-http-webhook.md?tabs=csharp).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Cree la aplicación de funciones en Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime dotnet --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    El comando [az functionapp create](/cli/azure/functionapp#az_functionapp_create) crea la aplicación de funciones en Azure. 
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime dotnet -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    El cmdlet [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) crea la aplicación de funciones en Azure. 
    
    ---
    
    En el ejemplo anterior, reemplace `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior y reemplace `<APP_NAME>` por un nombre único global que le resulte adecuado. `<APP_NAME>` también es el dominio DNS predeterminado de la aplicación de función. 
    
    Este comando crea una aplicación de funciones que se ejecuta en el entorno de ejecución del lenguaje especificado en el [plan de consumo de Azure Functions](consumption-plan.md), que es gratuito para la cantidad de uso que se realiza aquí. El comando también aprovisiona una instancia asociada de Azure Application Insights en el mismo grupo de recursos con la que puede supervisar la aplicación de funciones y ver registros. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md). La instancia no incurrirá en ningún costo hasta que se active.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage]

[Conexión a una cola de Azure Storage]: functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp
