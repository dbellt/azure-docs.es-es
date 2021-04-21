---
title: Configuración de CI/CD con GitHub Actions
description: Aprenda a implementar su código en Azure App Service desde una canalización de CI/CD con GitHub Actions. Personalice las tareas de compilación y ejecute implementaciones complejas.
ms.devlang: na
ms.topic: article
ms.date: 09/14/2020
ms.author: jafreebe
ms.reviewer: ushan
ms.custom: devx-track-python, github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 1ed2b007ae00516a030e67b7f6abacbd00a8d403
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772890"
---
# <a name="deploy-to-app-service-using-github-actions"></a>Implementación de App Service con Acciones de GitHub

Comience a usar [Acciones de GitHub](https://docs.github.com/en/actions/learn-github-actions) para automatizar el flujo de trabajo e implementarlo en [Azure App Service](overview.md) desde GitHub. 

## <a name="prerequisites"></a>Requisitos previos 

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Una cuenta de GitHub. Si no tiene ninguna, regístrese [gratis](https://github.com/join).  
- Una aplicación de Azure App Service en funcionamiento. 
    - .NET: [Creación de una aplicación web ASP.NET Core en Azure](quickstart-dotnetcore.md)
    - ASP.NET: [Creación de una aplicación web de ASP.NET Framework en Azure](quickstart-dotnet-framework.md)
    - JavaScript: [Creación de una aplicación web de Node.js en Azure App Service](quickstart-nodejs.md)  
    - Java: [Creación de una aplicación de Java en Azure App Service](quickstart-java.md)
    - Python: [Creación de una aplicación de Python en Azure App Service](quickstart-python.md)

## <a name="workflow-file-overview"></a>Información general sobre el archivo de flujo de trabajo

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo.

El archivo tiene tres secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | 1. Defina una entidad de servicio o un perfil de publicación. <br /> 2. Cree un secreto de GitHub. |
|**Compilar** | 1. Configure el entorno. <br /> 2. Cree la aplicación web. |
|**Implementar** | 1. Implemente la aplicación web. |

## <a name="use-the-deployment-center"></a>Uso del centro de implementación

Puede empezar a trabajar rápidamente con Acciones de GitHub mediante el centro de implementación de App Service. Se generará automáticamente un archivo de flujo de trabajo basado en la pila de aplicaciones y se confirmará en el repositorio de GitHub en el directorio correcto.

1. Vaya a la aplicación web en Azure Portal.
1. En el lado izquierdo, haga clic en **Centro de implementación**.
1. En **Implementación continua (CI/CD)** , seleccione **GitHub**.
1. A continuación, seleccione **Acciones de GitHub**.
1. Use las listas desplegables para seleccionar el repositorio de GitHub, la rama y la pila de aplicaciones.
    - Si la rama seleccionada está protegida, todavía puede seguir agregando el archivo de flujo de trabajo. Asegúrese de revisar las protecciones de la rama antes de continuar.
1. En la pantalla final, puede revisar las selecciones y obtener una versión preliminar del archivo de flujo de trabajo que se confirmará en el repositorio. Si las selecciones son correctas, haga clic en **Finalizar**.

Con esta acción se confirmará el archivo de flujo de trabajo en el repositorio. El flujo de trabajo que va a compilar e implementar la aplicación se iniciará inmediatamente.

## <a name="set-up-a-workflow-manually"></a>Configuración manual de un flujo de trabajo

También puede implementar un flujo de trabajo sin usar el centro de implementación. Para ello, primero debe generar las credenciales de implementación. 

## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.

La manera recomendada de autenticarse con Azure App Services para las Acciones de GitHub es con un perfil de publicación. También puede autenticarse con una entidad de servicio, pero el proceso requiere más pasos. 

Guarde la credencial de perfil de publicación o la entidad de servicio como [secreto de GitHub](https://docs.github.com/en/actions/reference/encrypted-secrets) para autenticarse con Azure. Tendrá acceso al secreto en el flujo de trabajo. 

# <a name="publish-profile"></a>[Perfil de publicación](#tab/applevel)

Un perfil de publicación es una credencial de nivel de aplicación. Configure el perfil de publicación como secreto de GitHub. 

1. En Azure Portal, vaya a la aplicación de App Service. 

1. En la página de **Información general**, seleccione **Obtener perfil de publicación**.

1. Guarde el archivo descargado. Usará el contenido del archivo para crear un secreto de GitHub.

> [!NOTE]
> A partir de octubre de 2020, las aplicaciones web de Linux necesitarán que la configuración de la aplicación `WEBSITE_WEBDEPLOY_USE_SCM` esté establecida en `true` **antes de descargar el perfil de publicación**. Este paso dejará de ser un requisito en el futuro.

# <a name="service-principal"></a>[Entidad de servicio](#tab/userlevel)

Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o haciendo clic en el botón **Probar**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

En este ejemplo, reemplace los marcadores de posición por su identificador de suscripción, el nombre del grupo de recursos y el nombre de la aplicación. La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la aplicación App Service similar al siguiente. Copie este objeto JSON para más adelante.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Siempre es recomendable conceder acceso mínimo. El ámbito del ejemplo anterior se limita a la aplicación específica de App Service y no a todo el grupo de recursos.

---

## <a name="configure-the-github-secret"></a>Configuración del secreto de GitHub


# <a name="publish-profile"></a>[Perfil de publicación](#tab/applevel)

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para usar las [credenciales de nivel de aplicación](#generate-deployment-credentials), pegue el contenido del archivo del perfil de publicación descargado en el campo de valor del secreto. Asigne al secreto el siguiente nombre: `AZURE_WEBAPP_PUBLISH_PROFILE`.

Al configurar el flujo de trabajo de GitHub, use `AZURE_WEBAPP_PUBLISH_PROFILE` en la acción de implementación de aplicación web de Azure. Por ejemplo:
    
```yaml
- uses: azure/webapps-deploy@v2
  with:
    publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

# <a name="service-principal"></a>[Entidad de servicio](#tab/userlevel)

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para utilizar las [credenciales de nivel de usuario](#generate-deployment-credentials), pegue toda la salida JSON del comando CLI de Azure en el campo de valor del secreto. Asigne al secreto el nombre `AZURE_CREDENTIALS`.

Cuando configure el archivo de flujo de trabajo más adelante, usará el secreto para la entrada `creds` de la acción de inicio de sesión de Azure. Por ejemplo:

```yaml
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## <a name="set-up-the-environment"></a>Configuración del entorno

La configuración del entorno puede realizarse mediante una de las acciones de configuración.

|**Lenguaje**  |**Acción de configuración**  |
|---------|---------|
|**.NET**     | `actions/setup-dotnet` |
|**ASP.NET**     | `actions/setup-dotnet` |
|**Java**     | `actions/setup-java` |
|**JavaScript** | `actions/setup-node` |
|**Python**     | `actions/setup-python` |

En los siguientes ejemplos se muestra cómo configurar el entorno para los diferentes idiomas admitidos:

**.NET**

```yaml
    - name: Setup Dotnet 3.3.x
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '3.3.x'
```

**ASP.NET**

```yaml
    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
```

**Java**

```yaml
    - name: Setup Java 1.8.x
      uses: actions/setup-java@v1
      with:
        # If your pom.xml <maven.compiler.source> version is not in 1.8.x,
        # change the Java version to match the version in pom.xml <maven.compiler.source>
        java-version: '1.8.x'
```

**JavaScript**

```yaml
env:
  NODE_VERSION: '14.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
```
**Python**

```yaml
    - name: Setup Python 3.x 
      uses: actions/setup-python@v1
      with:
        python-version: 3.x
```

## <a name="build-the-web-app"></a>Compilación de la aplicación web

El proceso de compilación de una aplicación web y la implementación en Azure App Service cambia según el lenguaje. 

En los siguientes ejemplos se muestra la parte del flujo de trabajo que compila la aplicación web en los diversos lenguajes admitidos.

En todos los lenguajes, puede establecer el directorio raíz de la aplicación web con `working-directory`. 

**.NET**

La variable de entorno `AZURE_WEBAPP_PACKAGE_PATH` establece la ruta de acceso al proyecto de aplicación web. 

```yaml
- name: dotnet build and publish
  run: |
    dotnet restore
    dotnet build --configuration Release
    dotnet publish -c Release -o '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp' 
```
**ASP.NET**

Puede restaurar las dependencias de NuGet y ejecutar MSBuild con `run`. 

```yaml
- name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
  run: nuget restore

- name: Add msbuild to PATH
  uses: microsoft/setup-msbuild@v1.0.0

- name: Run msbuild
  run: msbuild .\SampleWebApplication.sln
```

**Java**

```yaml
- name: Build with Maven
  run: mvn package --file pom.xml
```

**JavaScript**

Para Node.js, puede establecer `working-directory` o cambiarlo por el directorio NPM en `pushd`. 

```yaml
- name: npm install, build, and test
  run: |
    npm install
    npm run build --if-present
    npm run test --if-present
  working-directory: my-app-folder # set to the folder with your app if it is not the root directory
```

**Python**

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
```


## <a name="deploy-to-app-service"></a>Implementación en App Service

Para implementar el código en una aplicación de App Service, utilice la acción `azure/webapps-deploy@v2`. Esta acción tiene cuatro parámetros:

| **Parámetro**  | **Explicación**  |
|---------|---------|
| **app-name** | (Obligatorio) Especifique el nombre de la aplicación de App Service. | 
| **publish-profile** | (Opcional) Publique el contenido del archivo de perfil con secretos de Web Deploy |
| **package** | (Opcional) Ruta de acceso al paquete o la carpeta. La ruta de acceso puede incluir un archivo *.zip, *.war, *.jar, o una carpeta para implementar. |
| **slot-name** | (Opcional) Especifique una ranura existente que no sea la [ranura](deploy-staging-slots.md) de producción. |


# <a name="publish-profile"></a>[Perfil de publicación](#tab/applevel)

### <a name="net-core"></a>.NET Core

Compile e implemente una aplicación .NET Core en Azure con un perfil de publicación de Azure. La entrada `publish-profile` hace referencia al secreto de `AZURE_WEBAPP_PUBLISH_PROFILE` que creó anteriormente.

```yaml
name: .NET Core CI

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app-name    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '3.1.x'           # set this to the dot net version to use

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repo
      - uses: actions/checkout@main
      
      # Setup .NET Core SDK
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }} 
      
      # Run dotnet build and publish
      - name: dotnet build and publish
        run: |
          dotnet restore
          dotnet build --configuration Release
          dotnet publish -c Release -o '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp' 
          
      # Deploy to Azure Web apps
      - name: 'Run Azure webapp deploy action using publish profile credentials'
        uses: azure/webapps-deploy@v2
        with: 
          app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE  }} # Define secret variable in repository settings as per action documentation
          package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp'
```

### <a name="aspnet"></a>ASP.NET

Compile e implemente una aplicación ASP.NET MVC que use NuGet y `publish-profile` para la autenticación. 


```yaml
name: Deploy ASP.NET MVC App deploy to Azure Web App

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NUGET_VERSION: '5.3.x'           # set this to the dot net version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:

    - uses: actions/checkout@main  
    
    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
    - name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
      run: nuget restore
  
    - name: Add msbuild to PATH
      uses: microsoft/setup-msbuild@v1.0.0

    - name: Run MSBuild
      run: msbuild .\SampleWebApplication.sln
       
    - name: 'Run Azure webapp deploy action using publish profile credentials'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE  }} # Define secret variable in repository settings as per action documentation
        package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/SampleWebApplication/'
```

### <a name="java"></a>Java

Compile e implemente una aplicación Java Spring en Azure con un perfil de publicación de Azure. La entrada `publish-profile` hace referencia al secreto de `AZURE_WEBAPP_PUBLISH_PROFILE` que creó anteriormente.

```yaml
name: Java CI with Maven

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Build with Maven
      run: mvn -B package --file pom.xml
      working-directory: my-app-path
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: my-app-name
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: my/target/*.jar
```

Para implementar un archivo `war` en lugar de `jar`, cambie el valor de `package`. 


```yaml
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: my-app-name
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: my/target/*.war
```

### <a name="javascript"></a>JavaScript 

Compile e implemente una aplicación Node.js en Azure con el perfil de publicación de la aplicación. La entrada `publish-profile` hace referencia al secreto de `AZURE_WEBAPP_PUBLISH_PROFILE` que creó anteriormente.

```yaml
# File: .github/workflows/workflow.yml
name: JavaScript CI

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app-name   # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: 'my-app-path'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '14.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
    - name: npm install, build, and test
      run: |
        # Build and test the project, then
        # deploy to Azure Web App.
        npm install
        npm run build --if-present
        npm run test --if-present
      working-directory: my-app-path
    - name: 'Deploy to Azure WebApp'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

### <a name="python"></a>Python 

Compile e implemente una aplicación Python en Azure con el perfil de publicación de la aplicación. Observe cómo la entrada `publish-profile` hace referencia al secreto de `AZURE_WEBAPP_PUBLISH_PROFILE` que creó anteriormente.

```yaml
name: Python CI

on:
  [push]

env:
  AZURE_WEBAPP_NAME: my-web-app # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.x
      uses: actions/setup-python@v2
      with:
        python-version: 3.x
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Building web app
      uses: azure/appservice-build@v2
    - name: Deploy web App using GH Action azure/webapps-deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

# <a name="service-principal"></a>[Entidad de servicio](#tab/userlevel)

### <a name="net-core"></a>.NET Core 

Compile e implemente una aplicación .NET Core en Azure con una entidad de servicio de Azure. Observe cómo la entrada `creds` hace referencia al secreto de `AZURE_CREDENTIALS` que creó anteriormente.


```yaml
name: .NET Core

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '3.1.x'           # set this to the dot net version to use

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repo
      - uses: actions/checkout@main
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      
      # Setup .NET Core SDK
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }} 
      
      # Run dotnet build and publish
      - name: dotnet build and publish
        run: |
          dotnet restore
          dotnet build --configuration Release
          dotnet publish -c Release -o '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp' 
          
      # Deploy to Azure Web apps
      - name: 'Run Azure webapp deploy action using publish profile credentials'
        uses: azure/webapps-deploy@v2
        with: 
          app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
          package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp'
      
      - name: logout
        run: |
          az logout
```

### <a name="aspnet"></a>ASP.NET

Compile e implemente una aplicación ASP.NET MVC en Azure con una entidad de servicio de Azure. Observe cómo la entrada `creds` hace referencia al secreto de `AZURE_CREDENTIALS` que creó anteriormente.

```yaml
name: Deploy ASP.NET MVC App deploy to Azure Web App

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NUGET_VERSION: '5.3.x'           # set this to the dot net version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:

    # checkout the repo
    - uses: actions/checkout@main
    
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
    - name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
      run: nuget restore
  
    - name: Add msbuild to PATH
      uses: microsoft/setup-msbuild@v1.0.0

    - name: Run MSBuild
      run: msbuild .\SampleWebApplication.sln
       
    - name: 'Run Azure webapp deploy action using publish profile credentials'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
        package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/SampleWebApplication/'
  
    # Azure logout 
    - name: logout
      run: |
        az logout
```

### <a name="java"></a>Java 

Compile e implemente una aplicación Java Spring en Azure con una entidad de servicio de Azure. Observe cómo la entrada `creds` hace referencia al secreto de `AZURE_CREDENTIALS` que creó anteriormente.

```yaml
name: Java CI with Maven

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Build with Maven
      run: mvn -B package --file pom.xml
      working-directory: complete
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: my-app-name
        package: my/target/*.jar

    # Azure logout 
    - name: logout
      run: |
        az logout
```

### <a name="javascript"></a>JavaScript 

Compile e implemente una aplicación Node.js en Azure con una entidad de servicio de Azure. Observe cómo la entrada `creds` hace referencia al secreto de `AZURE_CREDENTIALS` que creó anteriormente.

```yaml
name: JavaScript CI

on: [push]

name: Node.js

env:
  AZURE_WEBAPP_NAME: my-app   # set this to your application's name
  NODE_VERSION: '14.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@main
   
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        
    - name: Setup Node ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
    
    - name: 'npm install, build, and test'
      run: |
        npm install
        npm run build --if-present
        npm run test --if-present
      working-directory:  my-app-path
               
    # deploy web app using Azure credentials
    - uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

    # Azure logout 
    - name: logout
      run: |
        az logout
```

### <a name="python"></a>Python 

Compile e implemente una aplicación Python en Azure con una entidad de servicio de Azure. Observe cómo la entrada `creds` hace referencia al secreto de `AZURE_CREDENTIALS` que creó anteriormente.

```yaml
name: Python application

on:
  [push]

env:
  AZURE_WEBAPP_NAME: my-app # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Set up Python 3.x
      uses: actions/setup-python@v2
      with:
        python-version: 3.x
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Deploy web App using GH Action azure/webapps-deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
    - name: logout
      run: |
        az logout
```


---

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar nuestro conjunto de acciones agrupadas en distintos repositorios de GitHub, cada uno con documentación y ejemplos que le ayudarán a usar GitHub con CI/CD y a implementar sus aplicaciones en Azure.

- [Flujos de trabajo de acciones para implementar en Azure](https://github.com/Azure/actions-workflow-samples)

- [Inicio de sesión de Azure](https://github.com/Azure/login)

- [Azure WebApp](https://github.com/Azure/webapps-deploy)

- [Azure WebApp for containers](https://github.com/Azure/webapps-container-deploy)

- [Inicio y cierre de sesión de Docker](https://github.com/Azure/docker-login)

- [Eventos que desencadenan flujos de trabajo](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)

- [Implementación de K8s](https://github.com/Azure/k8s-deploy)

- [Flujos de trabajo de inicio](https://github.com/actions/starter-workflows)
