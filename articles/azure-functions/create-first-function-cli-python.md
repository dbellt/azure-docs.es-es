---
title: 'Creación de una función de Python desde la línea de comandos: Azure Functions'
description: Aprenda a crear una función de Python desde la línea de comandos y, luego, a publicar el proyecto local en el hospedaje sin servidor de Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-python
- devx-track-azurecli
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-cli-python-uiex
ms.openlocfilehash: f5c51630d111bd68e311a93100abb8266e2a8e27
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787438"
---
# <a name="quickstart-create-a-python-function-in-azure-from-the-command-line"></a>Inicio rápido: Creación de una función de Python en Azure desde la línea de comandos

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

En este artículo se usan herramientas de línea de comandos para crear una función de Python que responda a solicitudes HTTP. Después de probar el código localmente, se implementa en el entorno sin servidor de Azure Functions.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

También hay una [versión basada en Visual Studio Code](create-first-function-vs-code-python.md) de este artículo.

## <a name="configure-your-local-environment"></a>Configuración del entorno local

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x.
  
+ Una de las siguientes herramientas para crear recursos de Azure:

    + [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

    + [Azure PowerShell](/powershell/azure/install-az-ps), versión 5.0 o posterior.

+ [Versiones de Python compatibles con Azure Functions](supported-languages.md#languages-by-runtime-version)

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

Compruebe los requisitos previos, que dependen de si usa la CLI de Azure o Azure PowerShell para crear recursos de Azure:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `az --version` para comprobar que la versión de la CLI de Azure es la 2.4 o posterior.

+ Ejecute `az login` para iniciar sesión en Azure y comprobar una suscripción activa.

+ Ejecute `python --version` (Linux y macOS) o `py --version` (Windows) para comprobar que la versión de Python es la 3.8.x, 3.7.x o 3.6.x.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es la 3.x.

+ Ejecute `(Get-Module -ListAvailable Az).Version` y compruebe que es la versión 5.0 o posterior. 

+ Ejecute `Connect-AzAccount` para iniciar sesión en Azure y comprobar una suscripción activa.

+ Ejecute `python --version` (Linux y macOS) o `py --version` (Windows) para comprobar que la versión de Python es la 3.8.x, 3.7.x o 3.6.x.

---

## <a name="create-and-activate-a-virtual-environment"></a><a name="create-venv"></a>Creación y activación de un entorno virtual

En una carpeta adecuada, ejecute los comandos siguientes para crear y activar un entorno virtual denominado `.venv`. Asegúrese de usar Python 3.8, 3.7 o 3.6, que son compatibles con Azure Functions.

# <a name="bash"></a>[bash](#tab/bash)

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

Si Python no instaló el paquete venv en la distribución de Linux, ejecute el siguiente comando:

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
py -m venv .venv
```

```powershell
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -m venv .venv
```

```cmd
.venv\scripts\activate
```

---

Ejecute todos los comandos siguientes en este entorno virtual activado. 

## <a name="create-a-local-function-project"></a>Creación de un proyecto de función local

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

1. Ejecute el comando `func init`, de la manera siguiente, para crear un proyecto de funciones en una carpeta llamada *LocalFunctionProj* con el entorno de ejecución especificado:  

    ```console
    func init LocalFunctionProj --python
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

#### <a name="__init__py"></a>\_\_init\_\_.py

*\_\_init\_\_.py* contiene una función `main()` de Python que se desencadena en función de la configuración de *function.json*.

:::code language="python" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/__init__.py":::

Con un desencadenador HTTP, la función recibe datos de solicitud en la variable `req`, como se define en *function.json*. `req` es una instancia de la [clase azure.functions.HttpRequest](/python/api/azure-functions/azure.functions.httprequest). El objeto devuelto, definido como `$return` en *function.json*, es una instancia de la [clase azure.functions.HttpResponse](/python/api/azure-functions/azure.functions.httpresponse). Para más información, vea [Enlaces y desencadenadores HTTP de Azure Functions](./functions-bindings-http-webhook.md?tabs=python).

#### <a name="functionjson"></a>function.json

*function.json* es un archivo de configuración que define la entrada y salida `bindings` de la función, lo que incluye el tipo de desencadenador.

Si lo desea, puede cambiar `scriptFile` para invocar otro un archivo de Python.

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::

Cada enlace requiere una dirección, un tipo y un nombre único. El desencadenador HTTP tiene un enlace de entrada de tipo [`httpTrigger`](functions-bindings-http-webhook-trigger.md) y un enlace de salida de tipo [`http`](functions-bindings-http-webhook-output.md).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

## <a name="create-supporting-azure-resources-for-your-function"></a>Creación de recursos auxiliares de Azure para la función

Antes de poder implementar el código de la función en Azure, debe crear tres recursos:

- Un grupo de recursos, que es un contenedor lógico de recursos relacionados.
- Una cuenta de almacenamiento, que mantiene el estado y otra información sobre los proyectos.
- Una aplicación de funciones, que proporciona el entorno para ejecutar el código de función. Una aplicación de funciones se asigna al proyecto de funciones y le permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación y el uso compartido de recursos.

Utilice los comandos siguientes para crear los elementos. Se admiten las CLI de Azure y de PowerShell.

1. Si todavía no lo ha hecho, inicie sesión en Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    ```azurecli
    az login
    ```

    El comando [az login](/cli/azure/reference-index#az_login) inicia sesión en su cuenta de Azure.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell) 
    ```azurepowershell
    Connect-AzAccount
    ```

    El cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) inicia sesión en la cuenta de Azure.

    ---

1. Cree un grupo de recursos llamado `AzureFunctionsQuickstart-rg` en la región `westeurope`. 

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    ```azurecli
    az group create --name AzureFunctionsQuickstart-rg --location westeurope
    ```
 
    El comando [az group create](/cli/azure/group#az_group_create) crea un grupo de recursos. Por lo general, los recursos y los grupos de recursos se crean en una región cerca de usted, y se utiliza alguna de las regiones devueltas por el comando `az account list-locations`.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzResourceGroup -Name AzureFunctionsQuickstart-rg -Location westeurope
    ```

    El comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) crea un grupo de recursos. Por lo general, los recursos y los grupos de recursos se crean en una región cerca de usted, y se utiliza alguna de las regiones devueltas por el cmdlet [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

    ---

    > [!NOTE]
    > No se pueden hospedar aplicaciones de Windows y Linux en el mismo grupo de recursos. Si tiene un grupo de recursos existente llamado `AzureFunctionsQuickstart-rg` con una aplicación web o aplicación de función de Windows, debe usar un grupo de recursos diferente.

1. Cree una cuenta de almacenamiento de uso general en el grupo de recursos y la región:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location westeurope --resource-group AzureFunctionsQuickstart-rg --sku Standard_LRS
    ```

    El comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) crea la cuenta de almacenamiento. 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzStorageAccount -ResourceGroupName AzureFunctionsQuickstart-rg -Name <STORAGE_NAME> -SkuName Standard_LRS -Location westeurope
    ```

    El cmdlet [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) crea la cuenta de almacenamiento.

    ---

    En el ejemplo anterior, reemplace `<STORAGE_NAME>` por un nombre que sea apropiado para usted y único en Azure Storage. Los nombres deben contener entre 3 y 24 caracteres y solo letras minúsculas. `Standard_LRS` especifica una cuenta de uso general, que es [compatible con Functions](storage-considerations.md#storage-account-requirements).
    
    Este inicio rápido solo le supondrá unos pequeños gastos a la cuenta de almacenamiento.

1. Cree la aplicación de funciones en Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime python --runtime-version 3.8 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME> --os-type linux
    ```
    
    El comando [az functionapp create](/cli/azure/functionapp#az_functionapp_create) crea la aplicación de funciones en Azure. Si usa Python 3.7 o 3.6, cambie `--runtime-version` a `3.7` o `3.6`, respectivamente.
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -FunctionsVersion 3 -RuntimeVersion 3.8 -Runtime python -Location 'West Europe'
    ```
    
    El cmdlet [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) crea la aplicación de funciones en Azure. Si usa Python 3.7 o 3.6, cambie `-RuntimeVersion` a `3.7` o `3.6`, respectivamente.

    ---
    
    En el ejemplo anterior, reemplace `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior y reemplace `<APP_NAME>` por un nombre único global que le resulte adecuado.  `<APP_NAME>` también es el dominio DNS predeterminado de la aplicación de función. 
    
    Este comando crea una aplicación de funciones que se ejecuta en el entorno de ejecución del lenguaje especificado en el [plan de consumo de Azure Functions](consumption-plan.md), que es gratuito para la cantidad de uso que se realiza aquí. El comando también aprovisiona una instancia asociada de Azure Application Insights en el mismo grupo de recursos con la que puede supervisar la aplicación de funciones y ver registros. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md). La instancia no incurrirá en ningún costo hasta que se active.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

Ejecute el siguiente comando para ver los [registros de streaming](functions-run-local.md#enable-streaming-logs) casi en tiempo real en Application Insights en Azure Portal:

```console
func azure functionapp logstream <APP_NAME> --browser
```

En una ventana de terminal independiente o en el explorador, vuelva a llamar a la función remota. En la ventana de terminal, se mostrará un registro detallado de la ejecución de la función en Azure. 

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-python)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/python-functions-qs-survey)