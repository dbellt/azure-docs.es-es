---
title: 'Inicio rápido: Creación de una aplicación de función en Azure Arc'
description: Para empezar a trabajar con Azure Functions en Azure Arc, implemente su primera aplicación de funciones.
ms.topic: quickstart
ms.date: 05/10/2021
ms.openlocfilehash: da00cc070e14490bcd3f4661d30b65bd97553c2e
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2021
ms.locfileid: "111528475"
---
# <a name="create-your-first-function-on-azure-arc-preview"></a>Creación de la primera función en Azure Arc (versión preliminar)

En este inicio rápido, creará un proyecto de Azure Functions y lo implementará en una aplicación de funciones que se ejecuta en un [clúster de Kubernetes habilitado para ](../azure-arc/kubernetes/overview.md). Para más información, consulte [App Service, Functions y Logic Apps en Azure Arc](../app-service/overview-arc-integration.md). Este escenario solo admite aplicaciones de funciones que se ejecutan en Linux.   

> [!NOTE]
> La compatibilidad con la ejecución de funciones en un clúster de Kubernetes habilitado para Arc se encuentra actualmente en versión preliminar.  
>  
> Actualmente no se admite la publicación de proyectos de función de PowerShell en clústeres de Kubernetes habilitados para Azure Arc. Si necesita implementar funciones de PowerShell en clústeres de Kubernetes habilitados para Azure Arc, [cree la aplicación de funciones en un contenedor](create-first-function-arc-custom-container.md). 

## <a name="prerequisites"></a>Requisitos previos

En el equipo local:

# <a name="c"></a>[C\#](#tab/csharp)

+ [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download)
+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3245 o superior.
+ [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

+ [Node.js](https://nodejs.org/), versión 12. También se admite la versión 10 de Node.js.
+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3245 o superior.
+ [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

# <a name="python"></a>[Python](#tab/python)

+ [Versiones de Python compatibles con Azure Functions](supported-languages.md#languages-by-runtime-version)
+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3245 o superior.
+ [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.

---

[!INCLUDE [functions-arc-create-environment](../../includes/functions-arc-create-environment.md)]

[!INCLUDE [app-service-arc-cli-install-extensions](../../includes/app-service-arc-cli-install-extensions.md)]

## <a name="create-the-local-function-project"></a>Creación del proyecto de función local

En Azure Functions, un proyecto de función es la unidad de implementación y ejecución de una o varias funciones individuales que responden cada una a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

1. Ejecute el comando `func init`, de la manera siguiente, para crear un proyecto de funciones en una carpeta llamada *LocalFunctionProj* con el entorno de ejecución especificado:  

    # <a name="c"></a>[C\#](#tab/csharp)

    ```console
    func init LocalFunctionProj --dotnet
    ```
    # <a name="javascript"></a>[JavaScript](#tab/nodejs)

    ```console
    func init LocalFunctionProj --javascript
    ```

    # <a name="python"></a>[Python](#tab/python)

    Python requiere un entorno virtual, cuyos comandos difieren entre Bash y una línea de comandos de Windows.
    
     + bash: 

        ```bash
        python -m venv .venv
        source .venv/bin/activate
        ```
    
     + línea de comandos:

        ```cmd
        py -m venv .venv
        .venv\scripts\activate
        ```  

    Ahora, creará el proyecto dentro del entorno virtual. 
    
    ```console
    func init LocalFunctionProj --python
    ```

    ---

1. Vaya a la carpeta del proyecto:

    ```console
    cd LocalFunctionProj
    ```

    Esta carpeta contiene varios archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). De forma predeterminada, el archivo *local.settings.json* se ha excluido del control de código fuente en el archivo *.gitignore*. Esta exclusión se debe a que el archivo puede contener secretos que se descargan de Azure.

1. Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función (HttpExample) y el argumento `--template` especifica el desencadenador de esta (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ```  
[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

[!INCLUDE [functions-arc-get-custom-location](../../includes/functions-arc-get-custom-location.md)]

## <a name="create-azure-resources"></a>Creación de recursos de Azure 

Para poder implementar el código de función en el nuevo entorno de Kubernetes de App Service, debe crear dos recursos más:

- Una [cuenta de almacenamiento](../storage/common/storage-account-create.md), que actualmente es necesaria para las herramientas y no forma parte del entorno.
- Una aplicación de funciones, que proporciona el contexto para ejecutar el código de función. La aplicación de funciones se ejecuta en el entorno de Kubernetes de App Service y se asigna al proyecto de función local. Una aplicación de función permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación y el uso compartido de recursos.

> [!NOTE]
> Las aplicaciones de funciones se ejecutan en un entorno de Kubernetes de App Service en un plan dedicado (App Service). Al crear la aplicación de funciones sin un plan existente, se crea automáticamente el plan correcto.  

### <a name="create-storage-account"></a>Crear cuenta de almacenamiento

Use el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) para crear una cuenta de almacenamiento de uso general en el grupo de recursos y la región:

```azurecli
az storage account create --name <STORAGE_NAME> --location westeurope --resource-group myResourceGroup --sku Standard_LRS
```

> [!NOTE]  
> Actualmente, las herramientas de Azure Functions necesitan una cuenta de almacenamiento. 

En el ejemplo anterior, reemplace `<STORAGE_NAME>` por un nombre que sea apropiado para usted y único en Azure Storage. Los nombres deben contener entre 3 y 24 caracteres y solo letras minúsculas. `Standard_LRS` especifica una cuenta de uso general, que es [compatible con Functions](storage-considerations.md#storage-account-requirements). El valor `--location` es una región estándar de Azure. 

### <a name="create-the-function-app"></a>Crear la aplicación de función

Ejecute el comando [az functionapp create](/cli/azure/functionapp#az_functionapp_create) para crear una aplicación de funciones en el entorno.

# <a name="c"></a>[C\#](#tab/csharp)  
```azurecli
az functionapp create --resource-group MyResourceGroup --name <APP_NAME> --custom-location <CUSTOM_LOCATION_ID> --storage-account <STORAGE_NAME> --functions-version 3 --runtime dotnet 
```

# <a name="javascript"></a>[JavaScript](#tab/nodejs)  
```azurecli
az functionapp create --resource-group MyResourceGroup --name <APP_NAME> --custom-location <CUSTOM_LOCATION_ID> --storage-account <STORAGE_NAME> --functions-version 3 --runtime node --runtime-version 12
```

# <a name="python"></a>[Python](#tab/python)  
```azurecli
az functionapp create --resource-group MyResourceGroup --name <APP_NAME> --custom-location <CUSTOM_LOCATION_ID> --storage-account <STORAGE_NAME> --functions-version 3 --runtime python --runtime-version 3.8
```
---

En este ejemplo, reemplace `<CUSTOM_LOCATION_ID>` por el identificador de la ubicación personalizada que ha determinado para el entorno de Kubernetes de App Service. También, reemplace `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior y `<APP_NAME>` por un nombre único global que le resulte adecuado. 

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

Dado que una implementación completa puede tardar algún tiempo en completarse en un clúster de Kubernetes habilitado para Azure Arc, puede que quiera volver a ejecutar el siguiente comando para comprobar las funciones publicadas:

```command
func azure functionapp list-functions
``` 

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene la aplicación de funciones ejecutándose en un contenedor de un entorno de Kubernetes de App Service habilitado para Arc, puede conectarla a Azure Storage agregando un enlace de salida de Queue Storage.

# <a name="c"></a>[C\#](#tab/csharp)  

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp)

# <a name="javascript"></a>[JavaScript](#tab/nodejs)  

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-javascript)

# <a name="python"></a>[Python](#tab/python)  

> [!div class="nextstepaction"]
> [Conexión a una cola de Azure Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-python)

---
