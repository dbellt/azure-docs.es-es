---
title: 'Inicio rápido: Creación de una aplicación de funciones en Azure Arc en un contenedor personalizado'
description: Para empezar a trabajar con Azure Functions en Azure Arc, implemente su primera aplicación de funciones en un contenedor de Linux personalizado.
ms.topic: quickstart
ms.date: 05/11/2021
ms.openlocfilehash: 28079b9a7821350736647d49815c541f6a79d119
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2021
ms.locfileid: "111525667"
---
# <a name="create-your-first-function-on-azure-arc-using-a-custom-container-preview"></a>Creación de la primera función en Azure Arc mediante un contenedor personalizado (versión preliminar)

En este inicio rápido, creará un proyecto de Azure Functions que se ejecuta en un contenedor personalizado y lo implementará en un [clúster de Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/overview.md) desde la cuenta de Docker Hub. Para más información, consulte [App Service, Functions y Logic Apps en Azure Arc](../app-service/overview-arc-integration.md). Este escenario solo admite aplicaciones de funciones que se ejecutan en Linux.   

> [!NOTE]
> La compatibilidad con la ejecución de funciones en un clúster de Kubernetes habilitado para Arc se encuentra actualmente en versión preliminar.  

## <a name="prerequisites"></a>Requisitos previos

En el equipo local:

# <a name="c"></a>[C\#](#tab/csharp)

+ [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download)
+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3245 o superior.
+ [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.
+ [Docker](https://docs.docker.com/install/)  
+ [Identificador de Docker](https://hub.docker.com/signup)

# <a name="javascript"></a>[JavaScript](#tab/nodejs)

+ [Node.js](https://nodejs.org/), versión 12. También se admite la versión 10 de Node.js.
+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3245 o superior.
+ [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.
+ [Docker](https://docs.docker.com/install/)  
+ [Identificador de Docker](https://hub.docker.com/signup)

# <a name="python"></a>[Python](#tab/python)

+ [Versiones de Python compatibles con Azure Functions](supported-languages.md#languages-by-runtime-version)
+ [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.0.3245 o superior.
+ [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o posterior.
+ [Docker](https://docs.docker.com/install/)  
+ [Identificador de Docker](https://hub.docker.com/signup)

---

[!INCLUDE [functions-arc-create-environment](../../includes/functions-arc-create-environment.md)]

[!INCLUDE [app-service-arc-cli-install-extensions](../../includes/app-service-arc-cli-install-extensions.md)]

## <a name="create-the-local-function-project"></a>Creación del proyecto de función local

En Azure Functions, un proyecto de función es el contexto de una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, se crea un proyecto de función que contiene una sola función.

1. Ejecute el comando `func init`, de la manera siguiente, para crear un proyecto de funciones en una carpeta llamada *LocalFunctionProj* con el entorno de ejecución especificado:  

    # <a name="c"></a>[C\#](#tab/csharp)

    ```console
    func init LocalFunctionProj --dotnet --docker
    ```
    # <a name="javascript"></a>[JavaScript](#tab/nodejs)

    ```console
    func init LocalFunctionProj --javascript --docker
    ```

    # <a name="python"></a>[Python](#tab/python)

    Python requiere un entorno virtual, cuyos comandos difieren entre Bash y una línea de comandos de Windows.
    
     + Bash: 

        ```bash
        python -m venv .venv
        source .venv/bin/activate
        ```
    
     + Línea de comandos:

        ```cmd
        py -m venv .venv
        .venv\scripts\activate
        ```  

    Ahora, creará el proyecto dentro del entorno virtual. 
    
    ```console
    func init LocalFunctionProj --python --docker
    ```
    ---

    La opción `--docker` genera un archivo `Dockerfile` para el proyecto que permite definir un contenedor personalizado adecuado para su uso con Azure Functions y el entorno de ejecución seleccionado.

1. Vaya a la carpeta del proyecto:

    ```console
    cd LocalFunctionProj
    ```

    Esta carpeta contiene el archivo Dockerfile y otros archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). De forma predeterminada, el archivo *local.settings.json* se ha excluido del control de código fuente en el archivo *.gitignore*. Esta exclusión se debe a que el archivo puede contener secretos que se descargan de Azure.

1. Abra el `Dockerfile` generado y busque la etiqueta `3.0` de la imagen base. Si hay una etiqueta `3.0`, reemplácela por una etiqueta `3.0.15885`. Por ejemplo, en una aplicación de JavaScript, el archivo de Docker debe modificarse para que contenga `FROM mcr.microsoft.com/azure-functions/node:3.0.15885`. Esta versión de la imagen base admite la implementación en un clúster de Kubernetes habilitado para Azure Arc. 

1. Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función (HttpExample) y el argumento `--template` especifica el desencadenador de esta (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ```  

## <a name="build-the-container-image-and-test-locally"></a>Creación de la imagen de contenedor y prueba local

El Dockerfile de la raíz del proyecto describe el entorno mínimo necesario para ejecutar la aplicación de funciones en un contenedor. La lista completa de imágenes base admitidas para Azure Functions se puede encontrar en la [página de imágenes base de Azure Functions](https://hub.docker.com/_/microsoft-azure-functions-base).

1. En la carpeta raíz del proyecto, ejecute el comando [docker build](https://docs.docker.com/engine/reference/commandline/build/) y especifique un nombre, `azurefunctionsimage`, y una etiqueta, `v1.0.0`.   

    El siguiente comando compila la imagen de Docker del contenedor.

    ```console
    docker build --tag <DOCKER_ID>/azurefunctionsimage:v1.0.0 .
    ```

    En este ejemplo, reemplace `<DOCKER_ID>` por el identificador de la cuenta de Docker Hub. Cuando el comando se complete, podrá ejecutar el nuevo contenedor de forma local.
    
1. Para probar la compilación, ejecute la imagen en un contenedor local con el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) y agregue el argumento de puertos `-p 8080:80`.

    ```console
    docker run -p 8080:80 -it <docker_id>/azurefunctionsimage:v1.0.0
    ```

    De nuevo, reemplace `<DOCKER_ID` por el identificador de Docker y agregue el argumento de puertos `-p 8080:80`.

1. Una vez que la imagen se esté ejecutando en un contenedor local, vaya a `http://localhost:8080/api/HttpExample?name=Functions`, donde debería aparecer el mismo mensaje "hello" que anteriormente. Dado que la función desencadenada por HTTP usa autorización anónima, puede seguir llamando a la función aunque se esté ejecutando en el contenedor. La configuración de la clave de acceso de la función se aplica cuando se ejecuta localmente en un contenedor. Si tiene problemas para llamar a la función, asegúrese de que el [acceso a la función](functions-bindings-http-webhook-trigger.md#authorization-keys) está establecido en anónimo.  

Después de haber comprobado la aplicación de funciones en el contenedor, detenga Docker con **Ctrl**+**C**.

## <a name="push-the-image-to-docker-hub"></a>Inserción de la imagen en Docker Hub

Docker Hub es un registro de contenedor que hospeda imágenes y proporciona servicios de imágenes y contenedores. Para compartir la imagen, lo cual incluye la implementación en Azure, debe insertarla en un registro.

1. Si aún no ha iniciado sesión en Docker, hágalo con el comando [docker login](https://docs.docker.com/engine/reference/commandline/login/), reemplazando `<docker_id>` por el identificador de Docker. Este comando le solicita su nombre de usuario y contraseña. Un mensaje de inicio de sesión correcto confirmará que ha iniciado sesión.

    ```console
    docker login
    ```
    
1. Cuando haya iniciado sesión, inserte la imagen en Docker Hub mediante el comando [docker push](https://docs.docker.com/engine/reference/commandline/push/) y reemplazando de nuevo `<docker_id>` por el identificador de Docker.

    ```console
    docker push <docker_id>/azurefunctionsimage:v1.0.0
    ```

1. Según la velocidad de la red, la inserción de la imagen la primera vez puede tardar unos minutos (la inserción de los cambios posteriores es mucho más rápida). Mientras espera, puede continuar con la sección siguiente y crear recursos de Azure en otro terminal.

[!INCLUDE [functions-arc-get-custom-location](../../includes/functions-arc-get-custom-location.md)]

## <a name="create-azure-resources"></a>Creación de recursos de Azure 

Para poder implementar el contenedor en el nuevo entorno de Kubernetes de App Service, debe crear dos recursos más:

- Una [cuenta de almacenamiento](../storage/common/storage-account-create.md), que actualmente es necesaria para las herramientas y no forma parte del entorno.
- Una aplicación de funciones, que proporciona el contexto para ejecutar el contenedor. La aplicación de funciones se ejecuta en el entorno de Kubernetes de App Service y se asigna al proyecto de función local. Una aplicación de función permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación y el uso compartido de recursos.

> [!NOTE]
> Las aplicaciones de funciones se ejecutan en un entorno de Kubernetes de App Service en un plan dedicado (App Service). Al crear la aplicación de funciones sin un plan existente, se crea automáticamente un plan.  

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
az functionapp create --resource-group MyResourceGroup --name <APP_NAME> --custom-location <CUSTOM_LOCATION_ID> --storage-account <STORAGE_NAME> --functions-version 3 --runtime dotnet --deployment-container-image-name <DOCKER_ID>/azurefunctionsimage:v1.0.0
```

# <a name="javascript"></a>[JavaScript](#tab/nodejs)  
```azurecli
az functionapp create --resource-group MyResourceGroup --name <APP_NAME> --custom-location <CUSTOM_LOCATION_ID> --storage-account <STORAGE_NAME> --functions-version 3 --runtime node --runtime-version 12 --deployment-container-image-name <DOCKER_ID>/azurefunctionsimage:v1.0.0
```

# <a name="python"></a>[Python](#tab/python)  
```azurecli
az functionapp create --resource-group MyResourceGroup --name <APP_NAME> --custom-location <CUSTOM_LOCATION_ID> --storage-account <STORAGE_NAME> --functions-version 3 --runtime python --runtime-version 3.8 --deployment-container-image-name <DOCKER_ID>/azurefunctionsimage:v1.0.0
```
---

En este ejemplo, reemplace `<CUSTOM_LOCATION_ID>` por el identificador de la ubicación personalizada que ha determinado para el entorno de Kubernetes de App Service. Además, reemplace `<STORAGE_NAME>` por el nombre de la cuenta que usó en el paso anterior, `<APP_NAME>` por un nombre globalmente único adecuado para usted y `<DOCKER_ID>` por el identificador de Docker Hub. 

El parámetro *deployment-container-image-name* especifica la imagen que se va a usar para la aplicación de funciones. Puede usar el comando [az functionapp config container show](/cli/azure/functionapp/config/container#az_functionapp_config_container_show) para ver información sobre la imagen que se utilizó en la implementación. También puede usar el comando [az functionapp config container set](/cli/azure/functionapp/config/container#az_functionapp_config_container_set) para realizar la implementación desde una imagen diferente.

La primera vez que se crea la aplicación de funciones, se extrae la imagen inicial de Docker Hub. También puede [habilitar la implementación continua en Azure](functions-create-function-linux-custom-image.md#enable-continuous-deployment-to-azure) desde Docker Hub.  

Para aprender a habilitar SSH en la imagen, consulte [Habilitación de conexiones SSH](functions-create-function-linux-custom-image.md#enable-ssh-connections).

### <a name="set-required-app-settings"></a>Establecimiento de la configuración de aplicación necesaria

Ejecute los siguientes comandos para crear una configuración de aplicación para la cadena de conexión de la cuenta de almacenamiento:

```azurecli-interactive
storageConnectionString=$(az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <STORAGE_NAME> --query connectionString --output tsv)
az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=$storageConnectionString
```

Este código debe ejecutarse en Cloud Shell o en Bash en el equipo local. Reemplace `<STORAGE_NAME>` por el nombre de la cuenta de almacenamiento y `<APP_NAME>` por el nombre de la aplicación de funciones.  

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
