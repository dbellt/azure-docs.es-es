---
title: Creación de una función de Python desde la línea de comandos de Azure Functions
description: Aprenda a crear una función de Python desde la línea de comandos y a publicar el proyecto local en el hospedaje sin servidor de Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-python
- devx-track-azurecli
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 626cff867a336880689373c289087e2332a816ee
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787456"
---
# <a name="quickstart-create-a-python-function-in-azure-from-the-command-line"></a>Inicio rápido: Creación de una función de Python en Azure desde la línea de comandos

> [!div class="op_single_selector" title1="Seleccione el lenguaje de la función: "]
> - [Python](create-first-function-cli-python.md)
> - [C#](create-first-function-cli-csharp.md)
> - [Java](create-first-function-cli-java.md)
> - [JavaScript](create-first-function-cli-node.md)
> - [PowerShell](create-first-function-cli-powershell.md)
> - [TypeScript](create-first-function-cli-typescript.md)

En este artículo se usan herramientas de línea de comandos para crear una función de Python que responda a solicitudes HTTP. Después de probar el código localmente, lo implementará en el entorno <abbr title="Un entorno informático en tiempo de ejecución en el que todos los detalles del servidor son transparentes para los desarrolladores de aplicaciones, lo que simplifica el proceso de implementación y administración de código.">sin servidor</abbr> de <abbr title="Un servicio de Azure que proporciona un entorno informático sin servidor de bajo costo dirigido a las aplicaciones.">Azure Functions</abbr>.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

También hay una [versión basada en Visual Studio Code](create-first-function-vs-code-python.md) de este artículo.

## <a name="1-configure-your-environment"></a>1. Configurar su entorno

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure <abbr title="El perfil que mantiene la información de facturación en relación con el uso de Azure.">account</abbr> con una suscripción activa. <abbr title="La estructura organizativa básica en la que administra los recursos de Azure, que normalmente se asocia a un individuo o departamento dentro de una organización.">subscription</abbr>. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x. 
  
+ O <abbr title="Un conjunto de herramientas de línea de comandos multiplataforma para trabajar con recursos de Azure desde el equipo de desarrollo local, como alternativa al uso de Azure Portal.">Azure CLI</abbr> o bien <abbr title="Un módulo de PowerShell que proporciona comandos para trabajar con recursos de Azure desde el equipo de desarrollo local, como alternativa al uso de Azure Portal.">Azure PowerShell</abbr> para crear recursos de Azure:

    + [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

    + [Azure PowerShell](/powershell/azure/install-az-ps), versión 5.0 o posterior.

+ [Python 3.8 (64 bits)](https://www.python.org/downloads/release/python-382/), [Python 3.7 (64 bits)](https://www.python.org/downloads/release/python-375/), [Python 3.6 (64 bits)](https://www.python.org/downloads/release/python-368/), que son compatibles con la versión 3.x de Azure Functions.

### <a name="11-prerequisite-check"></a>1.1 Comprobación de requisitos previos

Compruebe los requisitos previos, que dependen de si usa la CLI de Azure o Azure PowerShell para crear recursos de Azure:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que <abbr title="El conjunto de herramientas de línea de comandos para trabajar con Azure Functions en su equipo local.">Azure Functions Core Tools</abbr> representan la versión 3.x.

+ Ejecute `az --version` para comprobar que la versión de la CLI de Azure es la 2.4 o posterior.

+ Ejecute `az login` para iniciar sesión en Azure y comprobar una suscripción activa.

+ Ejecute `python --version` (Linux y macOS) o `py --version` (Windows) para comprobar que la versión de Python es la 3.8.x, 3.7.x o 3.6.x.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que <abbr title="El conjunto de herramientas de línea de comandos para trabajar con Azure Functions en su equipo local.">Azure Functions Core Tools</abbr> representan la versión 3.x.

+ Ejecute `(Get-Module -ListAvailable Az).Version` y compruebe que es la versión 5.0 o posterior. 

+ Ejecute `Connect-AzAccount` para iniciar sesión en Azure y comprobar una suscripción activa.

+ Ejecute `python --version` (Linux y macOS) o `py --version` (Windows) para comprobar que la versión de Python es la 3.8.x, 3.7.x o 3.6.x.

---

<br/>

---

## <a name="2-create-and-activate-a-virtual-environment"></a>2. <a name="create-venv"></a>Creación y activación de un entorno virtual

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

<br/>

---

## <a name="3-create-a-local-function-project"></a>3. Creación de un proyecto de función local

En esta sección, va a crear un proyecto <abbr title="Un contenedor lógico para una o varias funciones individuales que se pueden implementar y administrar juntas.">local de Azure Functions</abbr> en Python. Cada función del proyecto responde a un determinado <abbr title="El tipo de evento que invoca el código de la función, como una solicitud HTTP, un mensaje de cola o una hora específica.">desencadenador</abbr>.

1. Ejecute el comando `func init` para crear un proyecto de Functions en una carpeta llamada *LocalFunctionProj* con el entorno de ejecución especificado:  

    ```console
    func init LocalFunctionProj --python
    ```

1. Vaya a la carpeta del proyecto:

    ```console
    cd LocalFunctionProj
    ```
    
    <br/>
    <details>
    <summary><strong>¿Qué se crea en la carpeta LocalFunctionProj?</strong></summary>
    
    Esta carpeta contiene varios archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). Como *local.settings.json* puede contener secretos descargados de Azure, el archivo se excluye del control de código fuente de forma predeterminada en el archivo *.gitignore*.
    </details>

1. Para agregar una función a su proyecto, use el siguiente comando:

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ```   
    El argumento `--name` es el nombre único de la función (HttpExample).

    El argumento `--template` especifica el desencadenador de la función (HTTP).
    
    `func new` crea una subcarpeta que coincide con el nombre de la función que contiene un archivo *\_\_init\_\_.py* con el código de la función y un archivo de configuración denominado *function.json*.

    <br/>    
    <details>
    <summary><strong>Código de __init__.py</strong></summary>
    
    *\_\_init\_\_.py* contiene una función `main()` de Python que se desencadena en función de la configuración de *function.json*.
    
    :::code language="python" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/__init__.py":::
    
    Con un desencadenador HTTP, la función recibe datos de solicitud en la variable `req`, como se define en *function.json*. `req` es una instancia de la [clase azure.functions.HttpRequest](/python/api/azure-functions/azure.functions.httprequest). El objeto devuelto, definido como `$return` en *function.json*, es una instancia de la [clase azure.functions.HttpResponse](/python/api/azure-functions/azure.functions.httpresponse). Para más información, vea [Enlaces y desencadenadores HTTP de Azure Functions](./functions-bindings-http-webhook.md?tabs=python).
    </details>

    <br/>
    <details>
    <summary><strong>Código de function.json</strong></summary>

    *function.json* es un archivo de configuración que define los <abbr title="Conexiones declarativas entre una función y otros recursos. Un enlace de entrada proporciona datos a la función, mientras que un enlace de salida proporciona datos de la función a otros recursos.">enlaces de entrada y de salida</abbr> de la función, incluido el tipo de desencadenador.
    
    Si lo desea, puede cambiar `scriptFile` para invocar otro un archivo de Python.
    
    :::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::
    
    Cada enlace requiere una dirección, un tipo y un nombre único. El desencadenador HTTP tiene un enlace de entrada de tipo [`httpTrigger`](functions-bindings-http-webhook-trigger.md) y un enlace de salida de tipo [`http`](functions-bindings-http-webhook-output.md).    
    </details>

<br/>

---

## <a name="4-run-the-function-locally"></a>4. Ejecución local de la función

1. Para ejecutar una función, inicie el host en tiempo de ejecución local de Azure Functions desde la carpeta *LocalFunctionProj*:

    ```
    func start
    ```

    Hacia el final de la salida, deberían aparecer las líneas siguientes: 
    
    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    ...
    
    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.
    
    Http Functions:
    
            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    
    </pre>
    
    <br/>
    <details>
    <summary><strong>No veo HttpExample en la salida</strong></summary>

    Si HttpExample no aparece, es probable que haya iniciado el host desde fuera de la carpeta raíz del proyecto. En ese caso, use <kbd>Ctrl+C</kbd> para detener el host, vaya a la carpeta raíz del proyecto y vuelva a ejecutar el comando anterior.
    </details>

1. Copie la dirección URL de la función **HttpExample** de esta salida en un explorador y anexe la cadena de consulta **?name=<YOUR_NAME>** , lo que hará que la dirección URL completa sea **http://localhost:7071/api/HttpExample?name=Functions** . El explorador debe mostrar un mensaje como **Hello Functions**:

    ![Resultado de la ejecución de la función localmente en el explorador](../../includes/media/functions-run-function-test-local-cli/function-test-local-browser.png)

1. El terminal en el que inició el proyecto también muestra la salida del registro cuando realiza solicitudes.

1. Cuando termine, presione <kbd>Ctrl+C</kbd> y seleccione <kbd>y</kbd> para detener el host de Functions.

<br/>

---

## <a name="5-create-supporting-azure-resources-for-your-function"></a>5. Creación de recursos auxiliares de Azure para la función

Antes de poder implementar el código de la función en Azure, debe crear un <abbr title="Un contenedor lógico para recursos relacionados de Azure que se pueden administrar como una unidad.">resource group</abbr>, una <abbr title="Una cuenta que contiene todos los objetos de datos de Azure Storage. La cuenta de almacenamiento proporciona un espacio de nombres único para sus datos.">Cuenta de almacenamiento</abbr>y una <abbr title="El recurso en la nube que hospeda las funciones sin servidor de Azure, que proporciona el entorno de proceso subyacente en el que se ejecutan las funciones.">aplicación de funciones</abbr> mediante los siguientes comandos:

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
 
    El comando [az group create](/cli/azure/group#az_group_create) crea un grupo de recursos. Conviene crear el grupo de recursos y los recursos en una <abbr title="Una referencia geográfica a un centro de datos de Azure específico en el que se asignan recursos.">region</abbr> cercana al usuario, mediante una región disponible devuelta por el comando `az account list-locations`.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzResourceGroup -Name AzureFunctionsQuickstart-rg -Location westeurope
    ```

    El comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) crea un grupo de recursos. Por lo general, los recursos y los grupos de recursos se crean en una región cerca de usted, y se utiliza alguna de las regiones devueltas por el cmdlet [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

    ---

    No se pueden hospedar aplicaciones de Windows y Linux en el mismo grupo de recursos. Si tiene un grupo de recursos existente llamado `AzureFunctionsQuickstart-rg` con una aplicación web o aplicación de función de Windows, debe usar un grupo de recursos diferente.

1. Cree una cuenta de Azure Storage de uso general en su grupo de recursos y región:

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

    Sustituya `<STORAGE_NAME>` por un nombre que sea adecuado y que sea <abbr title="El nombre debe ser único en todas las cuentas de almacenamiento utilizadas por todos los clientes de Azure de forma global. Por ejemplo, se puede usar una combinación del nombre personal o de la empresa, el nombre de la aplicación y un identificador numérico, como en contosobizappstorage20.">único en Azure Storage</abbr>. Los nombres deben contener entre 3 y 24 caracteres y solo letras minúsculas. `Standard_LRS` especifica una cuenta de uso general, que es [compatible con Functions](storage-considerations.md#storage-account-requirements).
    
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
    
    Sustituya `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior.

    Sustituya `<APP_NAME>` por un <abbr title="Un nombre que debe ser único para todos los clientes de Azure de forma global. Por ejemplo, se puede usar una combinación del nombre personal o de la empresa, el nombre de la aplicación y un identificador numérico, como en contoso-bizapp-func-20.">nombre único global adecuado para usted</abbr>. `<APP_NAME>` también es el dominio DNS predeterminado de la aplicación de función. 
    
    <br/>
    <details>
    <summary><strong>¿Cuál es el costo de los recursos aprovisionados en Azure?</strong></summary>

    Este comando crea una aplicación de funciones que se ejecuta en el entorno de ejecución del lenguaje especificado en el [plan de consumo de Azure Functions](functions-scale.md#overview-of-plans), que es gratuito para la cantidad de uso que se realiza aquí. El comando también aprovisiona una instancia asociada de Azure Application Insights en el mismo grupo de recursos con la que puede supervisar la aplicación de funciones y ver registros. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md). La instancia no incurrirá en ningún costo hasta que se active.
    </details>

<br/>

---

## <a name="6-deploy-the-function-project-to-azure"></a>6. Implementación del proyecto de función en Azure

Después de haber creado correctamente su aplicación de funciones en Azure, estará listo para **implementar el proyecto de funciones local** mediante el comando [func azure functionapp publish](functions-run-local.md#project-file-deployment).  

En el ejemplo siguiente, reemplace `<APP_NAME>` por el nombre de su aplicación.

```console
func azure functionapp publish <APP_NAME>
```

El comando `publish` muestra un resultado similar a lo que se muestra a continuación (se ha truncado para que resulte más simple):

<pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
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
        Invoke url: https://msdocs-azurefunctions-qs.azurewebsites.net/api/httpexample
</pre>

<br/>

---

## <a name="7-invoke-the-function-on-azure"></a>7. Invocación de la función en Azure

Como la función usa un desencadenador HTTP, para invocarla es preciso enviar una solicitud HTTP a la dirección URL correspondiente en el explorador o mediante una herramienta como CURL. <abbr title="Una herramienta de línea de comandos para generar solicitudes HTTP a una dirección URL. Consulte https://curl.se/">curl</abbr>. 

# <a name="browser"></a>[Browser](#tab/browser)

Copie el valor completo de la **dirección URL de invocación** que se muestra en la salida del comando `publish` en una barra de direcciones del explorador y anexe el parámetro de consulta **&name=Functions**. El explorador debe mostrar una salida similar a cuando ejecutó la función localmente.

![La salida de la función ejecutada en Azure en un explorador](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-browser.png)

# <a name="curl"></a>[curl](#tab/curl)

Ejecute [`curl`](https://curl.haxx.se/) con la **dirección URL de invocación**, y anexe el parámetro **&name=Functions**. La salida del comando será el texto "Hello Functions".

![La salida de la función ejecutada en Azure en un mediante curl](../../includes/media/functions-run-remote-azure-cli/function-test-cloud-curl.png)

---

### <a name="71-view-real-time-streaming-logs"></a>7.1 Visualización de registros de streaming en tiempo real

Ejecute el siguiente comando para ver los [registros de streaming](functions-run-local.md#enable-streaming-logs) casi en tiempo real en Application Insights en Azure Portal:

```console
func azure functionapp logstream <APP_NAME> --browser
```

Reemplace `<APP_NAME>` por el nombre de la aplicación de función.

En una ventana de terminal independiente o en el explorador, vuelva a llamar a la función remota. En la ventana de terminal, se mostrará un registro detallado de la ejecución de la función en Azure. 

<br/>

---

## <a name="8-clean-up-resources"></a>8. Limpieza de recursos

Si va al [paso siguiente](#next-steps) y agrega un <abbr title="Una forma de asociar una función a una cola de almacenamiento, de tal manera que pueda crear mensajes en la cola. "> Enlace de salida de la cola de Azure Storage</abbr>mantenga todos los recursos en su lugar, ya que se basará en lo que ha hecho hasta ahora.

De lo contrario, use el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene para evitar incurrir en costos adicionales.

 # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name AzureFunctionsQuickstart-rg
```

<br/>

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-python)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/python-functions-qs-survey)
