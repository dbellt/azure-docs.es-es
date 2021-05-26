---
title: Creación de funciones de Azure en Linux mediante una imagen personalizada
description: Aprenda a crear funciones de Azure que se ejecutan en una imagen de Linux personalizada.
ms.date: 12/2/2020
ms.topic: tutorial
ms.custom: devx-track-csharp, mvc, devx-track-python, devx-track-azurepowershell, devx-track-azurecli
zone_pivot_groups: programming-languages-set-functions-full
ms.openlocfilehash: 3eb0c54fda14711e45af3974638d53807994df91
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459617"
---
# <a name="create-a-function-on-linux-using-a-custom-container"></a>Creación de una función en Linux con un contenedor personalizado

En este tutorial, va a crear e implementar código en Azure Functions como un contenedor de Docker personalizado mediante una imagen base de Linux. Normalmente usa una imagen personalizada cuando las funciones necesitan una versión de lenguaje determinada o requieren una dependencia o configuración específica que no se proporciona en la imagen integrada.

::: zone pivot="programming-language-other"
Azure Functions admite cualquier lenguaje o entorno de ejecución que use [controladores personalizados](functions-custom-handlers.md). En algunos lenguajes, como el lenguaje de programación R que se usa en este tutorial, se deben instalar el entorno de ejecución o las bibliotecas adicionales como dependencias que requieren un contenedor personalizado.
::: zone-end

Para implementar el código de función de un contenedor de Linux personalizado, se necesita un [plan premium](functions-premium-plan.md) o el hospedaje de un [plan dedicado (App Service)](dedicated-plan.md). La realización de este tutorial generará un pequeño costo en la cuenta de Azure, que puede minimizar con la [limpieza de los recursos](#clean-up-resources) cuando haya acabado.

También puede usar un contenedor de Azure App Service predeterminado como se indica en [Creación de su primera función hospedada en Linux](./create-first-function-cli-csharp.md?pivots=programming-language-python). Se pueden encontrar imágenes base admitidas para Azure Functions en el [repositorio de imágenes base de Azure Functions](https://hub.docker.com/_/microsoft-azure-functions-base).

En este tutorial, aprenderá a:

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"
> [!div class="checklist"]
> * Crear una aplicación de funciones y un archivo Dockerfile mediante Azure Functions Core Tools.
> * Crear una imagen personalizada con Docker
> * Publicar una imagen personalizada en un registro de contenedor
> * Crear recursos auxiliares en Azure para la aplicación de funciones
> * Implementar una aplicación de función desde Docker Hub
> * Agregar la configuración de aplicación a la aplicación de función
> * Habilite la implementación continua.
> * Habilitar las conexiones SSH al contenedor.
> * Agregar un enlace de salida de Queue Storage. 
::: zone-end
::: zone pivot="programming-language-other"
> [!div class="checklist"]
> * Crear una aplicación de funciones y un archivo Dockerfile mediante Azure Functions Core Tools.
> * Crear una imagen personalizada con Docker
> * Publicar una imagen personalizada en un registro de contenedor
> * Crear recursos auxiliares en Azure para la aplicación de funciones
> * Implementar una aplicación de función desde Docker Hub
> * Agregar la configuración de aplicación a la aplicación de función
> * Habilite la implementación continua.
> * Habilitar las conexiones SSH al contenedor.
::: zone-end

Puede seguir este tutorial en cualquier equipo que ejecute Windows, macOS o Linux. 

[!INCLUDE [functions-requirements-cli](../../includes/functions-requirements-cli.md)]

<!---Requirements specific to Docker --->
+ [Docker](https://docs.docker.com/install/)  

+ Un [identificador de Docker](https://hub.docker.com/signup)

[!INCLUDE [functions-cli-create-venv](../../includes/functions-cli-create-venv.md)]

## <a name="create-and-test-the-local-functions-project"></a>Creación y prueba de un proyecto local de Functions

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python"  
En un terminal o símbolo del sistema, ejecute el siguiente comando para el lenguaje elegido para crear un proyecto de aplicación de funciones en una carpeta denominada `LocalFunctionsProject`.  
::: zone-end  
::: zone pivot="programming-language-csharp"  
```console
func init LocalFunctionsProject --worker-runtime dotnet --docker
```
::: zone-end  
::: zone pivot="programming-language-javascript"  
```console
func init LocalFunctionsProject --worker-runtime node --language javascript --docker
```
::: zone-end  
::: zone pivot="programming-language-powershell"  
```console
func init LocalFunctionsProject --worker-runtime powershell --docker
```
::: zone-end  
::: zone pivot="programming-language-python"  
```console
func init LocalFunctionsProject --worker-runtime python --docker
```
::: zone-end  
::: zone pivot="programming-language-typescript"  
```console
func init LocalFunctionsProject --worker-runtime node --language typescript --docker
```
::: zone-end
::: zone pivot="programming-language-java"  
En una carpeta vacía, ejecute el comando siguiente para generar el proyecto de Functions desde un [arquetipo Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html).

# <a name="bash"></a>[Bash](#tab/bash)
```bash
mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DjavaVersion=8 -Ddocker
```
# <a name="powershell"></a>[PowerShell](#tab/powershell)
```powershell
mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" "-Ddocker"
```
# <a name="cmd"></a>[Cmd](#tab/cmd)
```cmd
mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" "-Ddocker"
```
---

El parámetro `-DjavaVersion` indica al entorno de ejecución de Functions qué versión de Java debe usar. Use `-DjavaVersion=11` si quiere que las funciones se ejecuten en Java 11. Cuando no se especifica `-DjavaVersion`, el valor predeterminado de Maven es Java 8. Para obtener más información, consulte [Versiones de Java](functions-reference-java.md#java-versions).

> [!IMPORTANT]
> Para llevar a cabo los pasos de este artículo, la variable de entorno `JAVA_HOME` se debe establecer en la ubicación de instalación de la versión correcta del JDK.

Maven le pide los valores necesarios para finalizar la generación del proyecto en la implementación.   
Indique los siguientes valores cuando se le solicite:

| Prompt | Valor | Descripción |
| ------ | ----- | ----------- |
| **groupId** | `com.fabrikam` | Un valor que identifica de forma única su proyecto entre todos los demás y que sigue las [reglas de nomenclatura de paquetes](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7) de Java. |
| **artifactId** | `fabrikam-functions` | Un valor que es el nombre del archivo jar, sin un número de versión. |
| **version** | `1.0-SNAPSHOT` | Elija el valor predeterminado. |
| **package** | `com.fabrikam.functions` | Un valor que es el paquete de Java para el código de función generado. Use el valor predeterminado. |

Escriba `Y` o presione Entrar para confirmar.

Maven crea los archivos del proyecto en una carpeta nueva llamada _artifactId_ que, en este ejemplo, es `fabrikam-functions`. 
::: zone-end

::: zone pivot="programming-language-other"  
```console
func init LocalFunctionsProject --worker-runtime custom --docker
```
::: zone-end

La opción `--docker` genera un archivo `Dockerfile` para el proyecto que permite definir un contenedor personalizado adecuado para su uso con Azure Functions y el entorno de ejecución seleccionado.

Vaya a la carpeta del proyecto:
::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-other"  
```console
cd LocalFunctionsProject
```
::: zone-end  
::: zone pivot="programming-language-java"  
```console
cd fabrikam-functions
```
::: zone-end  
::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python" 
Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función y el argumento `--template` especifica el desencadenador de esta. `func new` crea una subcarpeta que coincide con el nombre de la función que contiene un archivo de código apropiado para el lenguaje elegido del proyecto y un archivo de configuración denominado *function.json*.

```console
func new --name HttpExample --template "HTTP trigger"
```
::: zone-end

::: zone pivot="programming-language-other" 
Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función y el argumento `--template` especifica el desencadenador de esta. `func new` crea una subcarpeta que coincide con el nombre de función que contiene un archivo de configuración denominado *function.json*.

```console
func new --name HttpExample --template "HTTP trigger"
```

En un editor de texto, cree un archivo en la carpeta de proyecto denominada *handler.R*. Agregue lo siguiente como contenido del archivo.

```r
library(httpuv)

PORTEnv <- Sys.getenv("FUNCTIONS_CUSTOMHANDLER_PORT")
PORT <- strtoi(PORTEnv , base = 0L)

http_not_found <- list(
  status=404,
  body='404 Not Found'
)

http_method_not_allowed <- list(
  status=405,
  body='405 Method Not Allowed'
)

hello_handler <- list(
  GET = function (request) {
    list(body=paste(
      "Hello,",
      if(substr(request$QUERY_STRING,1,6)=="?name=") 
        substr(request$QUERY_STRING,7,40) else "World",
      sep=" "))
  }
)

routes <- list(
  '/api/HttpExample' = hello_handler
)

router <- function (routes, request) {
  if (!request$PATH_INFO %in% names(routes)) {
    return(http_not_found)
  }
  path_handler <- routes[[request$PATH_INFO]]

  if (!request$REQUEST_METHOD %in% names(path_handler)) {
    return(http_method_not_allowed)
  }
  method_handler <- path_handler[[request$REQUEST_METHOD]]

  return(method_handler(request))
}

app <- list(
  call = function (request) {
    response <- router(routes, request)
    if (!'status' %in% names(response)) {
      response$status <- 200
    }
    if (!'headers' %in% names(response)) {
      response$headers <- list()
    }
    if (!'Content-Type' %in% names(response$headers)) {
      response$headers[['Content-Type']] <- 'text/plain'
    }

    return(response)
  }
)

cat(paste0("Server listening on :", PORT, "...\n"))
runServer("0.0.0.0", PORT, app)
```

En *host.json*, modifique la sección `customHandler` para configurar el comando de inicio del controlador personalizado.

```json
"customHandler": {
  "description": {
      "defaultExecutablePath": "Rscript",
      "arguments": [
      "handler.R"
    ]
  },
  "enableForwardingHttpRequest": true
}
```
::: zone-end

Para probar la función localmente, inicie el host en tiempo de ejecución de Azure Functions local en la carpeta raíz del proyecto: 
::: zone pivot="programming-language-csharp"  
```console
func start  
```
::: zone-end  
::: zone pivot="programming-language-javascript,programming-language-powershell,programming-language-python"   
```console
func start  
```
::: zone-end  
::: zone pivot="programming-language-typescript"  
```console
npm install
npm start
```
::: zone-end  
::: zone pivot="programming-language-java"  
```console
mvn clean package  
mvn azure-functions:run
```
::: zone-end
::: zone pivot="programming-language-other"
```console
R -e "install.packages('httpuv', repos='http://cran.rstudio.com/')"
func start
```
::: zone-end 

Una vez que vea el punto de conexión `HttpExample` aparecer en la salida, vaya a `http://localhost:7071/api/HttpExample?name=Functions`. El explorador debe mostrar un mensaje "hello" que devuelve `Functions`, el valor proporcionado al parámetro de consulta `name`.

Use **Ctrl**-**C** para detener el host.

## <a name="build-the-container-image-and-test-locally"></a>Creación de la imagen de contenedor y prueba local

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-powershell,programming-language-python,programming-language-java,programming-language-typescript"
(Opcional) Examine el archivo *Dockerfile* en la carpeta raíz del proyecto. Este archivo describe el entorno necesario para ejecutar la aplicación de funciones en Linux.  La lista completa de imágenes base admitidas para Azure Functions se puede encontrar en la [página de imágenes base de Azure Functions](https://hub.docker.com/_/microsoft-azure-functions-base).
::: zone-end

::: zone pivot="programming-language-other"
Examine el archivo *Dockerfile* en la carpeta raíz del proyecto. Este archivo describe el entorno necesario para ejecutar la aplicación de funciones en Linux. Las aplicaciones de controlador personalizado usan la imagen `mcr.microsoft.com/azure-functions/dotnet:3.0-appservice` como base.

Modifique el archivo *Dockerfile* para instalar R. Reemplace el contenido de *Dockerfile* por el siguiente.

```dockerfile
FROM mcr.microsoft.com/azure-functions/dotnet:3.0-appservice 
ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true

RUN apt update && \
    apt install -y r-base && \
    R -e "install.packages('httpuv', repos='http://cran.rstudio.com/')"

COPY . /home/site/wwwroot
```
::: zone-end

En la carpeta raíz del proyecto, ejecute el comando [docker build](https://docs.docker.com/engine/reference/commandline/build/) y especifique un nombre, `azurefunctionsimage`, y una etiqueta, `v1.0.0`. Reemplace `<DOCKER_ID>` por el identificador de su cuenta de Docker Hub. Este comando compila la imagen de Docker del contenedor.

```console
docker build --tag <DOCKER_ID>/azurefunctionsimage:v1.0.0 .
```

Cuando el comando se complete, podrá ejecutar el nuevo contenedor de forma local.
    
Para probar la compilación, ejecute la imagen en un contenedor local con el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/). Para ello, reemplace de nuevo `<DOCKER_ID` por el identificador de Docker y agregue el argumento de puertos, `-p 8080:80`:

```console
docker run -p 8080:80 -it <docker_id>/azurefunctionsimage:v1.0.0
```

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-other"  
Una vez que la imagen se esté ejecutando en un contenedor local, abra un explorador que vaya a `http://localhost:8080` y aquí debería aparecer la imagen del marcador de posición que se muestra a continuación. La imagen aparece en este momento porque la función se ejecuta en el contenedor local, como lo haría en Azure, lo que significa que está protegida por una clave de acceso, como se define en *function.json* con la propiedad `"authLevel": "function"`. No obstante, el contenedor no se ha publicado aún en una aplicación de funciones de Azure, por lo que la clave aún no está disponible. Si desea realizar una prueba en el contenedor local, detenga Docker, cambie la propiedad de autorización a `"authLevel": "anonymous"`, vuelva a compilar la imagen y reinicie Docker. Después, restablezca `"authLevel": "function"` en *function.json*. Para más información, consulte [Claves de autorización](functions-bindings-http-webhook-trigger.md#authorization-keys).

![Imagen de marcador de posición que indica que el contenedor se está ejecutando localmente](./media/functions-create-function-linux-custom-image/run-image-local-success.png)

::: zone-end
::: zone pivot="programming-language-java"  
Una vez que la imagen se esté ejecutando en un contenedor local, vaya a `http://localhost:8080/api/HttpExample?name=Functions`, donde debería aparecer el mismo mensaje "hello" que anteriormente. Dado que el arquetipo Maven genera una función desencadenada por HTTP que usa autorización anónima, aún puede llamar a la función aunque se esté ejecutando en el contenedor. 
::: zone-end  

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

## <a name="create-supporting-azure-resources-for-your-function"></a>Creación de recursos auxiliares de Azure para la función

Para implementar el código de la función en Azure, debe crear tres recursos:

- Un grupo de recursos, que es un contenedor lógico de recursos relacionados.
- Una cuenta de almacenamiento, que mantiene el estado e información adicional sobre los proyectos.
- Una aplicación de funciones, que proporciona el entorno para ejecutar el código de función. Una aplicación de funciones se asigna al proyecto de funciones y le permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación y el uso compartido de recursos.

Puede utilizar comandos de la CLI de Azure para crear estos elementos. Cada comando proporciona una salida JSON al finalizar.

1. Inicie sesión en Azure con el comando [az login](/cli/azure/reference-index#az_login):

    ```azurecli
    az login
    ```
    
1. Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create). En el ejemplo siguiente se crea un grupo de recursos denominado `AzureFunctionsContainers-rg` en la región `westeurope`. (Por lo general, tanto los grupos de recursos como los recursos se crean en una región cercana y se utiliza alguna de las regiones disponibles del comando `az account list-locations`).

    ```azurecli
    az group create --name AzureFunctionsContainers-rg --location westeurope
    ```
    
    > [!NOTE]
    > No se pueden hospedar aplicaciones de Windows y Linux en el mismo grupo de recursos. Si tiene un grupo de recursos existente llamado `AzureFunctionsContainers-rg` con una aplicación web o aplicación de función de Windows, debe usar un grupo de recursos diferente.
    
1. Cree una cuenta de almacenamiento de uso general en el grupo de recursos con el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create). En el siguiente ejemplo, reemplace `<storage_name>` por un nombre único global que sea adecuado. Los nombres deben contener entre 3 y 24 caracteres y solo letras minúsculas. `Standard_LRS` especifica una cuenta típica de uso general.

    ```azurecli
    az storage account create --name <storage_name> --location westeurope --resource-group AzureFunctionsContainers-rg --sku Standard_LRS
    ```
    
    La cuenta de almacenamiento incurrirá solo en un costo insignificante para este tutorial.
    
1. Use el comando para crear un plan Premium para Azure Functions denominado `myPremiumPlan` en el plan de tarifa **Premium elástico 1** (`--sku EP1`), en la región Oeste de Europa (`-location westeurope`, o use una región adecuada cerca de usted) y en un contenedor de Linux (`--is-linux`).

    ```azurecli
    az functionapp plan create --resource-group AzureFunctionsContainers-rg --name myPremiumPlan --location westeurope --number-of-workers 1 --sku EP1 --is-linux
    ```   

    Aquí se usa el plan Premium, que se puede escalar según sea necesario. Para obtener más información sobre el hospedaje, consulte [Comparación de los planes de hospedaje de Azure Functions](functions-scale.md). Para calcular los costos, consulte la [página de precios de Functions](https://azure.microsoft.com/pricing/details/functions/).

    El comando también aprovisiona una instancia asociada de Azure Application Insights en el mismo grupo de recursos con la que puede supervisar la aplicación de funciones y ver registros. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md). La instancia no incurrirá en ningún costo hasta que se active.

## <a name="create-and-configure-a-function-app-on-azure-with-the-image"></a>Creación y configuración de una aplicación de funciones en Azure con la imagen

Una aplicación de funciones en Azure administra la ejecución de las funciones en el plan de hospedaje. En esta sección, usará los recursos de Azure de la sección anterior para crear una aplicación de funciones a partir de una imagen en Docker Hub y configurarla con una cadena de conexión a Azure Storage.

1. Cree la aplicación de funciones con el comando [az functionapp create](/cli/azure/functionapp#az_functionapp_create). En el ejemplo siguiente, reemplace `<storage_name>` por el nombre que utilizó en la sección anterior para la cuenta de almacenamiento. Reemplace también `<app_name>` por un nombre único global que sea adecuado y `<docker_id>` por el identificador de Docker.

    ::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"
    ```azurecli
    az functionapp create --name <app_name> --storage-account <storage_name> --resource-group AzureFunctionsContainers-rg --plan myPremiumPlan --runtime <functions runtime stack> --deployment-container-image-name <docker_id>/azurefunctionsimage:v1.0.0
    ```
    ::: zone-end
    ::: zone pivot="programming-language-other"
    ```azurecli
    az functionapp create --name <app_name> --storage-account <storage_name> --resource-group AzureFunctionsContainers-rg --plan myPremiumPlan --runtime custom --deployment-container-image-name <docker_id>/azurefunctionsimage:v1.0.0
    ```
    ::: zone-end
    
    El parámetro *deployment-container-image-name* especifica la imagen que se va a usar para la aplicación de funciones. Puede usar el comando [az functionapp config container show](/cli/azure/functionapp/config/container#az_functionapp_config_container_show) para ver información sobre la imagen que se utilizó en la implementación. También puede usar el comando [az functionapp config container set](/cli/azure/functionapp/config/container#az_functionapp_config_container_set) para realizar la implementación desde una imagen diferente.

1. Muestre la cadena de conexión para la cuenta de almacenamiento que creó mediante el comando [az storage account show-connection-string](/cli/azure/storage/account). Reemplace `<storage-name>` por el nombre de la cuenta de almacenamiento que creó anteriormente:

    ```azurecli
    az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <storage_name> --query connectionString --output tsv
    ```
    
1. Agregue esta configuración a la aplicación de funciones mediante el comando [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az_functionapp_config_ppsettings_set). En el siguiente comando, reemplace `<app_name>` por el nombre de la aplicación de funciones y reemplace `<connection_string>` por la cadena de conexión del paso anterior (una cadena larga codificada que comienza por "DefaultEndpointProtocol ="):
 
    ```azurecli
    az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=<connection_string>
    ```

    > [!TIP]
    > En Bash, puede usar una variable del shell para capturar la cadena de conexión en lugar de usar el portapapeles. En primer lugar, use el siguiente comando para crear una variable con la cadena de conexión:
    > 
    > ```bash
    > storageConnectionString=$(az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <storage_name> --query connectionString --output tsv)
    > ```
    > 
    > Después, haga referencia a la variable en el segundo comando:
    > 
    > ```azurecli
    > az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=$storageConnectionString
    > ```

1. La función ahora puede usar esta cadena de conexión para acceder a la cuenta de almacenamiento.

> [!NOTE]    
> Si publica la imagen personalizada en una cuenta de contenedor privada, debe usar, en su lugar, variables de entorno en el archivo Dockerfile para la cadena de conexión. Para más información, consulte la [instrucción ENV](https://docs.docker.com/engine/reference/builder/#env). También debe establecer las variables `DOCKER_REGISTRY_SERVER_USERNAME` y `DOCKER_REGISTRY_SERVER_PASSWORD`. Para usar los valores, debe recompilar la imagen, insertarla en el registro y, a continuación, reiniciar la aplicación de funciones en Azure.

## <a name="verify-your-functions-on-azure"></a>Comprobación de las funciones en Azure

Con la imagen implementada en la aplicación de funciones en Azure, ya puede invocar la función mediante solicitudes HTTP. Dado que la definición de *function.json* incluye la propiedad `"authLevel": "function"`, primero debe obtener la clave de acceso (también denominada "clave de función") e incluirla como parámetro de dirección URL en las solicitudes al punto de conexión.

1. Recupere la dirección URL con la clave de acceso (función) mediante Azure Portal o mediante la CLI de Azure con el comando `az rest`.

    # <a name="portal"></a>[Portal](#tab/portal)

    1. Inicie sesión en Azure Portal, busque y seleccione **Aplicación de funciones**.

    1. Seleccione la función que quiera comprobar.

    1. En el panel de navegación izquierdo, seleccione **Funciones** y, a continuación, seleccione la función que quiera comprobar.

        ![Elija su función en Azure Portal](./media/functions-create-function-linux-custom-image/functions-portal-select-function.png)   

    
    1. Seleccione **Obtener la dirección URL de la función**.

        ![Opción Copiar la dirección URL de Azure Portal](./media/functions-create-function-linux-custom-image/functions-portal-get-function-url.png)   

    
    1. En la ventana emergente, seleccione **predeterminada (clave de función)** y, a continuación, copie la dirección URL en el portapapeles. La clave es la cadena de caracteres que sigue a `?code=`.

        ![Elija la tecla de acceso predeterminada de la función](./media/functions-create-function-linux-custom-image/functions-portal-copy-url.png)   


    > [!NOTE]  
    > Dado que la aplicación de función se implementa como un contenedor, no se pueden realizar cambios en el código de la función en el portal. En su lugar, debe actualizar el proyecto en la imagen local, insertar de nuevo la imagen en el registro y, a continuación, volver a implementarlo en Azure. Puede configurar la implementación continua en una sección posterior.
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

    1. Construya una cadena de dirección URL con el formato siguiente, reemplazando `<subscription_id>`, `<resource_group>` y `<app_name>` por el identificador de suscripción de Azure, el grupo de recursos de la aplicación de funciones y el nombre de esta, respectivamente:

        ```
        "/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Web/sites/<app_name>/host/default/listKeys?api-version=2018-11-01"
        ```

        Por ejemplo, la dirección URL podría ser parecida a la siguiente:

        ```
        "/subscriptions/1234aaf4-1234-abcd-a79a-245ed34eabcd/resourceGroups/AzureFunctionsContainers-rg/providers/Microsoft.Web/sites/msdocsfunctionscontainer/host/default/listKeys?api-version=2018-11-01"
        ```

        > [!TIP]
        > Para mayor comodidad, puede asignar la dirección URL a una variable de entorno y usarla en el comando `az rest`.
    
    1. Ejecute el siguiente comando `az rest` (disponible en la versión 2.0.77 y posteriores de la CLI de Azure), reemplazando `<uri>` por la cadena de URI del último paso, incluidas las comillas:

        ```azurecli
        az rest --method post --uri <uri> --query functionKeys.default --output tsv
        ```

    1. La salida del comando es la clave de función. La dirección URL completa de la función es `https://<app_name>.azurewebsites.net/api/<function_name>?code=<key>`, reemplazando `<app_name>`, `<function_name>` y `<key>` por sus valores específicos.
    
        > [!NOTE]
        > La clave recuperada aquí es la clave de *host* que funciona para todas las funciones de la aplicación. El método que se muestra para el portal recupera la clave solo para una función.

    ---

1. Pegue la dirección URL de la función en la barra de direcciones del explorador y agregue el parámetro `&name=Azure` al final de esta dirección URL. Debería aparecer un texto del tipo "Hello Azure" en el explorador.

    ![Respuesta de la función en el explorador.](./media/functions-create-function-linux-custom-image/function-app-browser-testing.png)

1. Para probar la autorización, quite el parámetro `code=` de la dirección URL y compruebe que no obtiene ninguna respuesta de la función.


## <a name="enable-continuous-deployment-to-azure"></a>Habilitar implementación continua en Azure

Puede habilitar Azure Functions para que actualice automáticamente la implementación de una imagen cada vez que actualice la imagen en el registro.

1. Habilite la implementación continua mediante el comando [az functionapp deployment container config](/cli/azure/functionapp/deployment/container#az_functionapp_deployment_container_config), reemplazando `<app_name>` por el nombre de la aplicación de funciones:

    ```azurecli
    az functionapp deployment container config --enable-cd --query CI_CD_URL --output tsv --name <app_name> --resource-group AzureFunctionsContainers-rg
    ```
    
    Este comando permite habilitar la implementación continua y devuelve la dirección URL del webhook de implementación. (Puede recuperar esta dirección URL en cualquier momento posterior mediante el comando [az functionapp deployment container show-cd-url](/cli/azure/functionapp/deployment/container#az_functionapp_deployment_container_show_cd_url).)

1. Copie la dirección URL del webhook de implementación en el portapapeles.

1. Abra [Docker Hub](https://hub.docker.com/), inicie sesión y seleccione **Repositorios** en la barra de navegación. Busque y seleccione la imagen, seleccione la pestaña **Webhooks**, especifique un **nombre de webhook**, pegue la dirección URL en **Dirección URL de webhook** y, finalmente, seleccione **Crear**:

    ![Agregar el webhook al repositorio de DockerHub](./media/functions-create-function-linux-custom-image/dockerhub-set-continuous-webhook.png)  

1. Una vez establecido el webhook, Azure Functions volverá a implementar la imagen cada vez que la actualice en Docker Hub.

## <a name="enable-ssh-connections"></a>Habilitación de las conexiones SSH

SSH habilita la comunicación segura entre un contenedor y un cliente. Con SSH habilitado, puede conectarse al contenedor mediante las herramientas avanzadas de App Service (Kudu). Para facilitar la conexión al contenedor mediante SSH, Azure Functions proporciona una imagen base que ya tiene SSH habilitado. Solo necesita editar el archivo de Dockerfile y, después, recompilar y volver a implementar la imagen. Después, puede conectar el contenedor mediante Herramientas avanzadas (Kudu)

1. En el archivo de Dockerfile, anexe la cadena `-appservice` a la imagen base en la instrucción `FROM`:

    ::: zone pivot="programming-language-csharp"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/dotnet:3.0-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-javascript"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/node:2.0-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-powershell"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/powershell:2.0-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-python"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/python:2.0-python3.7-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-typescript"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/node:2.0-appservice
    ```
    ::: zone-end
    
1. Vuelva a recompilar la imagen mediante el comando `docker build`, sustituyendo `<docker_id>` por el identificador de Docker:

    ```console
    docker build --tag <docker_id>/azurefunctionsimage:v1.0.0 .
    ```
    
1. Inserte la imagen actualizada en Docker Hub, lo cual ahora debe tardar mucho menos tiempo que la primera vez ya que solo se cargarán los segmentos actualizados de la imagen.

    ```console
    docker push <docker_id>/azurefunctionsimage:v1.0.0
    ```
    
1. Azure Functions vuelve a implementar automáticamente la imagen en la aplicación de funciones. El proceso se realiza en menos de un minuto.

1. En un explorador, abra `https://<app_name>.scm.azurewebsites.net/`, sustituyendo `<app_name>` por su nombre único. Esta dirección URL es el punto de conexión de Herramientas avanzadas (KUDU) para el contenedor de la aplicación de funciones.

1. Inicie sesión en su cuenta de Azure y, después, seleccione **SSH** para establecer una conexión con el contenedor. La conexión puede tardar unos minutos si Azure todavía está actualizando la imagen de contenedor.

1. Una vez establecida la conexión con el contenedor, ejecute el comando `top` para ver los procesos que se están ejecutando actualmente. 

    ![Comando top de Linux en ejecución en una sesión SSH](media/functions-create-function-linux-custom-image/linux-custom-kudu-ssh-top.png)

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"

## <a name="write-to-an-azure-storage-queue"></a>Escritura en una cola de Azure Storage

Azure Functions permite conectar funciones a otros servicios y recursos de Azure sin tener que escribir su propio código de integración. Estos *enlaces*, que representan la entrada y la salida, se declaran dentro de la definición de función. Los datos de los enlaces se proporcionan a la función como parámetros. Un *desencadenador* es un tipo especial de enlace de entrada. Si bien una función tiene un único desencadenador, puede tener varios enlaces de entrada y salida. Para más información, consulte [Conceptos básicos sobre los enlaces y desencadenadores de Azure Functions](functions-triggers-bindings.md).

En esta sección se muestra cómo integrar la función con una cola de Azure Storage. El enlace de salida que se agrega a esta función escribe datos de una solicitud HTTP en un mensaje de la cola.

[!INCLUDE [functions-cli-get-storage-connection](../../includes/functions-cli-get-storage-connection.md)]
::: zone-end

[!INCLUDE [functions-register-storage-binding-extension-csharp](../../includes/functions-register-storage-binding-extension-csharp.md)]

[!INCLUDE [functions-add-output-binding-cli](../../includes/functions-add-output-binding-cli.md)]

::: zone pivot="programming-language-csharp"  
[!INCLUDE [functions-add-storage-binding-csharp-library](../../includes/functions-add-storage-binding-csharp-library.md)]  
::: zone-end  
::: zone pivot="programming-language-java" 
[!INCLUDE [functions-add-output-binding-java-cli](../../includes/functions-add-output-binding-java-cli.md)]
::: zone-end  

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"

## <a name="add-code-to-use-the-output-binding"></a>Adición de código para usar el enlace de salida

Con el enlace de cola definido, ahora puede actualizar la función para que reciba el parámetro de salida `msg` y escribir mensajes en la cola.
::: zone-end

::: zone pivot="programming-language-python"     
[!INCLUDE [functions-add-output-binding-python](../../includes/functions-add-output-binding-python.md)]
::: zone-end  

::: zone pivot="programming-language-javascript"  
[!INCLUDE [functions-add-output-binding-js](../../includes/functions-add-output-binding-js.md)]
::: zone-end  

::: zone pivot="programming-language-typescript"  
[!INCLUDE [functions-add-output-binding-ts](../../includes/functions-add-output-binding-ts.md)]
::: zone-end  

::: zone pivot="programming-language-powershell"  
[!INCLUDE [functions-add-output-binding-powershell](../../includes/functions-add-output-binding-powershell.md)]  
::: zone-end

::: zone pivot="programming-language-csharp"  
[!INCLUDE [functions-add-storage-binding-csharp-library-code](../../includes/functions-add-storage-binding-csharp-library-code.md)]
::: zone-end 

::: zone pivot="programming-language-java"
[!INCLUDE [functions-add-output-binding-java-code](../../includes/functions-add-output-binding-java-code.md)]

[!INCLUDE [functions-add-output-binding-java-test-cli](../../includes/functions-add-output-binding-java-test-cli.md)]
::: zone-end

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"
### <a name="update-the-image-in-the-registry"></a>Actualización de la imagen en el registro

1. En la carpeta raíz, ejecute de nuevo `docker build` y, en esta ocasión, actualice la versión que está en la etiqueta a `v1.0.1`. Igual que antes, reemplace `<docker_id>` por el identificador de la cuenta de Docker Hub:

    ```console
    docker build --tag <docker_id>/azurefunctionsimage:v1.0.1 .
    ```
    
1. Inserte la imagen actualizada de nuevo en el repositorio con `docker push`:

    ```console
    docker push <docker_id>/azurefunctionsimage:v1.0.1
    ```

1. Dado que configuró la entrega continua, si actualiza la imagen en el registro se actualizará automáticamente la aplicación de funciones de nuevo en Azure.

## <a name="view-the-message-in-the-azure-storage-queue"></a>Visualización del mensaje en la cola de Azure Storage

En un explorador, use la misma dirección URL de antes para invocar la función. El explorador debe mostrar la misma respuesta que antes, ya que no se modificó esa parte del código de la función. No obstante, el mensaje agregado escribió un mensaje mediante el parámetro `name` de URL en la cola de almacenamiento `outqueue`.

[!INCLUDE [functions-add-output-binding-view-queue-cli](../../includes/functions-add-output-binding-view-queue-cli.md)]

::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea seguir trabajando con Azure Functions y con los recursos que creó en este tutorial, puede dejar todos esos recursos activos. Como ha creado un plan Premium para Azure Functions, incurrirá en un pequeño costo continuo por día.

Para evitar estos costos, elimine el grupo de recursos `AzureFunctionsContainer-rg` para limpiar todos los recursos de ese grupo: 

```azurecli
az group delete --name AzureFunctionsContainer-rg
```

## <a name="next-steps"></a>Pasos siguientes

+ [Supervisión de funciones](functions-monitoring.md)
+ [Opciones de escalado y hospedaje](functions-scale.md)
+ [Hospedaje sin servidor basado en Kubernetes](functions-kubernetes-keda.md)
