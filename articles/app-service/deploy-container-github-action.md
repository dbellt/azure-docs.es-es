---
title: Contenedor personalizado de CI/CD de GitHub Actions
description: Aprenda a usar GitHub Actions para implementar contenedores Linux personalizados en App Service desde una canalización de CI/CD.
ms.devlang: na
ms.topic: article
ms.date: 12/04/2020
ms.author: jafreebe
ms.reviewer: ushan
ms.custom: github-actions-azure
ms.openlocfilehash: 1fe09970bcb9b9432b9b6f22de04bb24f1e84fa8
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106582089"
---
# <a name="deploy-a-custom-container-to-app-service-using-github-actions"></a>Implementación de un contenedor personalizado en App Service con Acciones de GitHub

[Acciones de GitHub](https://docs.github.com/en/actions) le ofrece la flexibilidad de compilar un flujo de trabajo de desarrollo de software automatizado. Gracias a la [acción de Azure Web Deploy](https://github.com/Azure/webapps-deploy), puede automatizar el flujo de trabajo para implementar contenedores personalizados en [App Service](overview.md) con Acciones de GitHub.

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que se incluyen en el flujo de trabajo.

Para un flujo de trabajo de contenedor de Azure App Service, el archivo tiene tres secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | 1. Recupere una entidad de servicio o un perfil de publicación. <br /> 2. Cree un secreto de GitHub. |
|**Compilar** | 1. Cree el entorno. <br /> 2. Compile la imagen de contenedor. |
|**Implementación** | 1. Implemente la imagen de contenedor. |

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree su cuenta de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Una cuenta de GitHub. Si no tiene ninguna, regístrese [gratis](https://github.com/join). Debe tener código en un repositorio de GitHub para implementar en Azure App Service. 
- Un registro de contenedor en funcionamiento y la aplicación de Azure App Service para contenedores. En este ejemplo, se utiliza Azure Container Registry. Asegúrese de completar toda la implementación en Azure App Service para contenedores. A diferencia de las aplicaciones web normales, las aplicaciones web para contenedores no tienen una página de aterrizaje predeterminada. Publique el contenedor para tener un ejemplo práctico.
    - [Aprenda a crear una aplicación de Node.js en contenedor con Docker, insertar la imagen de contenedor en un registro y, a continuación, implementar la imagen en Azure App Service](/azure/developer/javascript/tutorial-vscode-docker-node-01).
        
## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.

La manera recomendada de autenticarse con Azure App Services para las Acciones de GitHub es con un perfil de publicación. También puede autenticarse con una entidad de servicio, pero el proceso requiere más pasos. 

Guarde la credencial de perfil de publicación o la entidad de servicio como [secreto de GitHub](https://docs.github.com/en/actions/reference/encrypted-secrets) para autenticarse con Azure. Tendrá acceso al secreto en el flujo de trabajo. 

# <a name="publish-profile"></a>[Perfil de publicación](#tab/publish-profile)

Un perfil de publicación es una credencial de nivel de aplicación. Configure el perfil de publicación como secreto de GitHub. 

1. En Azure Portal, vaya a la aplicación de App Service. 

1. En la página de **Información general**, seleccione **Obtener perfil de publicación**.

    > [!NOTE]
    > A partir de octubre de 2020, las aplicaciones web de Linux deberán tener la opción de configuración de la aplicación `WEBSITE_WEBDEPLOY_USE_SCM` establecida en `true` **antes de descargar el archivo**. Este paso dejará de ser un requisito en el futuro. Consulte [Configurar una aplicación de App Service en Azure Portal](./configure-common.md) para aprender a configurar las opciones de aplicaciones web comunes.  

1. Guarde el archivo descargado. Usará el contenido del archivo para crear un secreto de GitHub.

# <a name="service-principal"></a>[Entidad de servicio](#tab/service-principal)

Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o haciendo clic en el botón **Probar**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

En este ejemplo, reemplace los marcadores de posición por su identificador de suscripción, el nombre del grupo de recursos y el nombre de la aplicación. La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la aplicación de App Service. Copie este objeto JSON para más adelante.

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
## <a name="configure-the-github-secret-for-authentication"></a>Configuración del secreto de GitHub para la autenticación

# <a name="publish-profile"></a>[Perfil de publicación](#tab/publish-profile)

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para usar las [credenciales de nivel de aplicación](#generate-deployment-credentials), pegue el contenido del archivo del perfil de publicación descargado en el campo de valor del secreto. Asigne al secreto el siguiente nombre: `AZURE_WEBAPP_PUBLISH_PROFILE`.

Al configurar el flujo de trabajo de GitHub, use `AZURE_WEBAPP_PUBLISH_PROFILE` en la acción de implementación de aplicación web de Azure. Por ejemplo:
    
```yaml
- uses: azure/webapps-deploy@v2
  with:
    publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

# <a name="service-principal"></a>[Entidad de servicio](#tab/service-principal)

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para utilizar las [credenciales de nivel de usuario](#generate-deployment-credentials), pegue toda la salida JSON del comando CLI de Azure en el campo de valor del secreto. Asigne al secreto un nombre como `AZURE_CREDENTIALS`.

Cuando configure el archivo de flujo de trabajo más adelante, usará el secreto para la entrada `creds` de la acción de inicio de sesión de Azure. Por ejemplo:

```yaml
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## <a name="configure-github-secrets-for-your-registry"></a>Configuración de secretos de GitHub para el registro

Defina los secretos que se usarán con la acción de inicio de sesión de Docker. En el ejemplo de este documento se utiliza Azure Container Registry para el registro del contenedor. 

1. Vaya al contenedor en Azure Portal o Docker y copie el nombre de usuario y la contraseña. Puede encontrar el nombre de usuario y la contraseña de Azure Container Registry en Azure Portal en **Configuración** > **Claves de acceso** para el registro. 

2. Defina un nuevo secreto para el nombre de usuario del registro llamado `REGISTRY_USERNAME`. 

3. Defina un nuevo secreto para la contraseña del registro llamada `REGISTRY_PASSWORD`. 

## <a name="build-the-container-image"></a>Compilación de la imagen de contenedor

En el ejemplo siguiente se muestra parte del flujo de trabajo que compila una imagen de Docker de Node.js. Use el [inicio de sesión de Docker](https://github.com/azure/docker-login) en un registro de contenedor privado. En este ejemplo se usa Azure Container Registry, pero la misma acción funciona para otros registros. 


```yaml
name: Linux Container Node Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     
```

También puede usar el [inicio de sesión de Docker](https://github.com/azure/docker-login) para iniciar sesión en varios registros de contenedor al mismo tiempo. Este ejemplo incluye dos nuevos secretos de GitHub para la autenticación con docker.io. En el ejemplo se da por supuesto que hay un Dockerfile en el nivel raíz del registro. 

```yml
name: Linux Container Node Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - uses: azure/docker-login@v1
      with:
        login-server: index.docker.io
        username: ${{ secrets.DOCKERIO_USERNAME }}
        password: ${{ secrets.DOCKERIO_PASSWORD }}
    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     
```

## <a name="deploy-to-an-app-service-container"></a>Implementación en un contenedor de App Service

Para implementar la imagen en un contenedor personalizado en App Service, use la acción `azure/webapps-deploy@v2`. Esta acción tiene siete parámetros:

| **Parámetro**  | **Explicación**  |
|---------|---------|
| **app-name** | (Obligatorio) Especifique el nombre de la aplicación de App Service. | 
| **publish-profile** | (Opcional) Se aplica a Web Apps (Windows y Linux) y a los contenedores de aplicaciones web (Linux). No se admite el escenario de varios contenedores. Publique el contenido del archivo de perfil (\*.publishsettings) con secretos de Web Deploy. | 
| **slot-name** | (Opcional) Especifique un espacio existente que no sea el de producción. |
| **package** | (Opcional) Solo se aplica a la aplicación web: ruta de acceso al paquete o la carpeta. Un archivo \*.zip, \*.war o \*.jar o una carpeta para la implementación. |
| **images** | (Necesario) Solo se aplica a contenedores de aplicaciones web: especifique el nombre completo de las imágenes de contenedor. Por ejemplo, 'myregistry.azurecr.io/nginx:latest' o 'python:3.7.2-alpine/'. En el caso de una aplicación de varios contenedores, se pueden especificar varios nombres de imagen de contenedor (separados por varias líneas). |
| **configuration-file** | (Opcional) Solo se aplica a contenedores de aplicaciones web: ruta de acceso del archivo Docker Compose. Debe ser una ruta de acceso completa o relativa al directorio de trabajo predeterminado. Se requiere para las aplicaciones de varios contenedores. |
| **startup-command** | (Opcional) Escriba el comando de inicio. Por ejemplo, dotnet run o dotnet filename.dll |

# <a name="publish-profile"></a>[Perfil de publicación](#tab/publish-profile)

```yaml
name: Linux Container Node Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}

    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     

    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'myapp'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        images: 'mycontainer.azurecr.io/myapp:${{ github.sha }}'
```
# <a name="service-principal"></a>[Entidad de servicio](#tab/service-principal)

```yaml
on: [push]

name: Linux_Container_Node_Workflow

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@main
    
    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     
      
    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'myapp'
        images: 'mycontainer.azurecr.io/myapp:${{ github.sha }}'
    
    - name: Azure logout
      run: |
        az logout
```

---

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar nuestro conjunto de acciones agrupadas en distintos repositorios de GitHub, cada uno con documentación y ejemplos que le ayudarán a usar GitHub con CI/CD y a implementar sus aplicaciones en Azure.

- [Flujos de trabajo de acciones para implementar en Azure](https://github.com/Azure/actions-workflow-samples)

- [Inicio de sesión de Azure](https://github.com/Azure/login)

- [Azure WebApp](https://github.com/Azure/webapps-deploy)

- [Inicio y cierre de sesión de Docker](https://github.com/Azure/docker-login)

- [Eventos que desencadenan flujos de trabajo](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)

- [Implementación de K8s](https://github.com/Azure/k8s-deploy)

- [Flujos de trabajo de inicio](https://github.com/actions/starter-workflows)