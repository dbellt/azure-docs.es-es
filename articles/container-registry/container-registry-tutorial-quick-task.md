---
title: 'Tutorial: Creación rápida de imagen de contenedor'
description: En este tutorial, aprenderá a compilar una imagen de contenedor de Docker en Azure con Azure Container Registry Tasks (ACR Tasks) para, después, implementarla en Azure Container Instances.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 282e6ea56835fba679510a29af936c1fbcb3ead2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775356"
---
# <a name="tutorial-build-and-deploy-container-images-in-the-cloud-with-azure-container-registry-tasks"></a>Tutorial: Compilación e implementación de imágenes de contenedor en la nube con Azure Container Registry Tasks

[ACR Tasks](container-registry-tasks-overview.md) es un conjunto de características de Azure Container Registry que proporcionan compilaciones de imágenes de contenedor de Docker optimizadas y eficientes en Azure. En este artículo, aprenderá a usar la característica *tarea rápida* de ACR Tasks.

El ciclo de desarrollo de "bucle interno" es el proceso iterativo de escribir código, compilar y probar la aplicación antes confirmar el control de código fuente. Una tarea rápida amplía su "bucle interno" a la nube, proporcionándole una validación de compilaciones correctas y la inserción automática de las imágenes compiladas correctamente en el registro de contenedor. Las imágenes se compilan de forma nativa en la nube, cerca del registro, lo cual permite una implementación más rápida.

Todos sus conocimientos sobre archivos de Docker se pueden transferir directamente a ACR Tasks. No tiene que cambiar sus archivos de Docker para compilar en la nube con ACR Tasks, solo ejecutar el comando. 

Este tutorial es la primera parte de una serie:

> [!div class="checklist"]
> * Obtención del código fuente de la aplicación de ejemplo
> * Compilación de una imagen de contenedor en Azure
> * Implementación de un contenedor en Azure Container Instances

En tutoriales posteriores, aprenderá a usar ACR Tasks para compilaciones automatizadas de imágenes de contenedor después de una confirmación de código y la actualización de la imagen base. Azure Container Registry Tasks también puede ejecutar [tareas de varios pasos](container-registry-tasks-multi-step.md) mediante un archivo YAML que define los pasos para compilar, insertar y, opcionalmente, probar varios contenedores.

## <a name="prerequisites"></a>Requisitos previos

### <a name="github-account"></a>Cuenta de GitHub

Cree una cuenta en https://github.com si aún no tiene una. Esta serie de tutoriales utiliza un repositorio de GitHub para mostrar compilaciones de imágenes automatizadas en ACR Tasks.

### <a name="fork-sample-repository"></a>Bifurcación del repositorio de ejemplo

A continuación, use la interfaz de usuario de GitHub para bifurcar el repositorio de ejemplo de la cuenta de GitHub. En este tutorial, compilará una imagen de contenedor desde el origen del repositorio y, en el tutorial siguiente, insertará una confirmación en la bifurcación del repositorio para iniciar una tarea automatizada.

Bifurque este repositorio: https://github.com/Azure-Samples/acr-build-helloworld-node

![Captura de pantalla del botón Bifurcar (resaltado) en GitHub][quick-build-01-fork]

### <a name="clone-your-fork"></a>Clonación de la bifurcación

Una vez que se ha bifurcado el repositorio, clone la bifurcación y especifique el directorio que contiene el clon local.

Clone el repositorio con `git` y reemplace **\<your-github-username\>** por su nombre de usuario de GitHub:

```console
git clone https://github.com/<your-github-username>/acr-build-helloworld-node
```

Especifique el directorio que contiene el código fuente:

```console
cd acr-build-helloworld-node
```

### <a name="bash-shell"></a>Shell de Bash

Los comandos de esta serie de tutoriales están formateados para el shell de Bash. Si prefiere usar PowerShell, el símbolo del sistema u otro shell, deberá ajustar la continuación de línea y el formato de la variable de entorno según sea necesario.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="build-in-azure-with-acr-tasks"></a>Compilación en Azure con ACR Tasks

Ahora que ha extraído el código fuente en su máquina, siga estos pasos para crear un registro de contenedor y compilar la imagen de contenedor con ACR Tasks.

Para facilitar la ejecución de comandos de ejemplo, los tutoriales de esta serie usan variables de entorno de shell. Ejecute el siguiente comando para establecer la variable `ACR_NAME`. Reemplace **\<registry-name\>** por un nombre único para el nuevo registro de contenedor. El nombre del registro debe ser único en Azure y contener solo minúsculas, y entre 5 y 50 caracteres alfanuméricos. Los demás recursos que cree en el tutorial se basan en este nombre con lo cual solo tendría que modificar esta primera variable.

```console
ACR_NAME=<registry-name>
```

Con la variable de entorno del registro de contenedor rellena, ya puede copiar y pegar el resto de los comandos del tutorial sin necesidad de editar ningún valor. Ejecute los siguientes comandos para crear un grupo de recursos y un registro de contenedor:

```azurecli
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location eastus
```

Ahora que tiene un registro, utilice ACR Tasks para compilar una imagen de contenedor desde el código de ejemplo. Ejecute el comando [az acr build][az-acr-build] para efectuar una *tarea rápida*:

```azurecli
az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
```

La salida del comando [az acr build][az-acr-build] se parece a la siguiente. Puede ver la carga del código fuente (el "contexto") en Azure y los detalles de la operación `docker build` que ACR Tasks ejecuta en la nube. Como ACR Tasks usa `docker build` para compilar las imágenes, no se requiere ningún cambio en los archivos de Docker para empezar a usar inmediatamente ACR Tasks.

```output
Packing source code into tar file to upload...
Sending build context (4.813 KiB) to ACR...
Queued a build with build ID: da1
Waiting for build agent...
2020/11/18 18:31:42 Using acb_vol_01185991-be5f-42f0-9403-a36bb997ff35 as the home volume
2020/11/18 18:31:42 Setting up Docker configuration...
2020/11/18 18:31:43 Successfully set up Docker configuration
2020/11/18 18:31:43 Logging in to registry: myregistry.azurecr.io
2020/11/18 18:31:55 Successfully logged in
Sending build context to Docker daemon   21.5kB
Step 1/5 : FROM node:15-alpine
15-alpine: Pulling from library/node
Digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
Status: Image is up to date for node:15-alpine
 ---> a56170f59699
Step 2/5 : COPY . /src
 ---> 88087d7e709a
Step 3/5 : RUN cd /src && npm install
 ---> Running in e80e1263ce9a
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN helloworld@1.0.0 No repository field.

up to date in 0.1s
Removing intermediate container e80e1263ce9a
 ---> 26aac291c02e
Step 4/5 : EXPOSE 80
 ---> Running in 318fb4c124ac
Removing intermediate container 318fb4c124ac
 ---> 113e157d0d5a
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in fe7027a11787
Removing intermediate container fe7027a11787
 ---> 20a27b90eb29
Successfully built 20a27b90eb29
Successfully tagged myregistry.azurecr.io/helloacrtasks:v1
2020/11/18 18:32:11 Pushing image: myregistry.azurecr.io/helloacrtasks:v1, attempt 1
The push refers to repository [myregistry.azurecr.io/helloacrtasks]
6428a18b7034: Preparing
c44b9827df52: Preparing
172ed8ca5e43: Preparing
8c9992f4e5dd: Preparing
8dfad2055603: Preparing
c44b9827df52: Pushed
172ed8ca5e43: Pushed
8dfad2055603: Pushed
6428a18b7034: Pushed
8c9992f4e5dd: Pushed
v1: digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1 size: 1366
2020/11/18 18:32:43 Successfully pushed image: myregistry.azurecr.io/helloacrtasks:v1
2020/11/18 18:32:43 Step ID acb_step_0 marked as successful (elapsed time in seconds: 15.648945)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloacrtasks
    tag: v1
    digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 15-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git: {}

Run ID: da1 was successful after 1m9.970148252s
```

Casi al final de la salida, ACR Tasks muestra las dependencias que se han detectado para la imagen. Esto permite a ACR Tasks automatizar las compilaciones de imágenes después de las actualizaciones de imágenes base como, por ejemplo, cuando se actualiza una imagen base con revisiones del sistema operativo o la plataforma. Puede aprender más información sobre la compatibilidad de ACR Tasks con las actualizaciones de imágenes base más adelante en esta serie de tutoriales.

## <a name="deploy-to-azure-container-instances"></a>Implementación en Azure Container Instances

De forma predeterminada, ACR Tasks inserta automáticamente las imágenes compiladas correctamente en el registro, lo que le permite implementarlas desde el registro de forma inmediata.

En esta sección, creará una instancia de Azure Key Vault y una entidad de servicio, posteriormente implementará el contenedor en Azure Container Instances (ACI) mediante las credenciales de la entidad de servicio.

### <a name="configure-registry-authentication"></a>Configurar la autenticación del registro

Todos los escenarios de producción deben usar [entidades de servicio][service-principal-auth] para acceder a un registro de contenedor de Azure. Las entidades de servicio permiten proporcionar control de acceso basado en rol a las imágenes de contenedor. Por ejemplo, puede configurar una entidad de servicio con acceso de solo extracción a un registro.

#### <a name="create-a-key-vault"></a>Creación de un Almacén de claves

Si todavía no tiene un almacén en [Azure Key Vault](../key-vault/index.yml), cree uno con la CLI de Azure mediante los siguientes comandos.

```azurecli
AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME
```

#### <a name="create-a-service-principal-and-store-credentials"></a>Creación de una entidad de servicio y almacenamiento de credenciales

Ahora debe crear una entidad de servicio y almacenar sus credenciales en el almacén de claves.

Use el comando [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] para crear la entidad de servicio y [az keyvault secret set][az-keyvault-secret-set] para almacenar la **contraseña** de la entidad de servicio en el almacén:

```azurecli
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)
```

El argumento `--role` en el comando anterior configura la entidad de servicio con el rol *acrpull*, que le concede de solo extracción acceso al registro. Para conceder acceso de inserción y extracción, cambie el argumento `--role` a *acrpush*.

A continuación, almacene el *appId* de la entidad de servicio en el almacén, que es el **nombre de usuario** que pasa a Azure Container Registry para la autenticación:

```azurecli
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

Ha creado una instancia de Azure Key Vault y almacenado dos secretos en ella:

* `$ACR_NAME-pull-usr`: el identificador de la entidad de servicio, para su uso como **nombre de usuario** del registro de contenedor.
* `$ACR_NAME-pull-pwd`: la contraseña de la entidad de servicio, para su uso como **contraseña** del registro de contenedor.

Ahora puede hacer referencia a estos secretos por nombre cuando usted o sus aplicaciones y servicios extraigan imágenes del registro.

### <a name="deploy-a-container-with-azure-cli"></a>Implementación de un contenedor con la CLI de Azure

Ahora que las credenciales de la entidad de servicio están almacenadas como secretos de Azure Key Vault, las aplicaciones y servicios pueden utilizarlas para acceder a su registro privado.

Ejecute el comando [az container create][az-container-create] siguiente para implementar una instancia de contenedor. El comando usa las credenciales de la entidad de servicio almacenadas en Azure Key Vault para autenticarse en el registro de contenedor.

```azurecli
az container create \
    --resource-group $RES_GROUP \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-tasks-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table
```

El valor `--dns-name-label` debe ser único en Azure, por lo que el comando anterior anexa el nombre del registro de contenedor a la etiqueta de nombre DNS del contenedor. La salida del comando muestra el nombre de dominio completo (FQDN) del contenedor, por ejemplo:

```output
FQDN
----------------------------------------------
acr-tasks-myregistry.eastus.azurecontainer.io
```

Tome nota del nombre de dominio completo del contenedor ya que lo utilizará en la sección siguiente.

### <a name="verify-the-deployment"></a>Comprobar la implementación

Para inspeccionar el proceso de inicio del contenedor, use el comando [az container attach][az-container-attach]:

```azurecli
az container attach --resource-group $RES_GROUP --name acr-tasks
```

La salida `az container attach` muestra en primer lugar el estado del contenedor cuando este extrae la imagen y se inicia. Posteriormente, enlaza los flujos STDOUT y STDERR de la consola local con los del contenedor.

```output
Container 'acr-tasks' is in state 'Running'...
(count: 1) (last timestamp: 2020-11-18 18:39:10+00:00) pulling image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2020-11-18 18:39:15+00:00) Successfully pulled image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2020-11-18 18:39:17+00:00) Created container
(count: 1) (last timestamp: 2020-11-18 18:39:17+00:00) Started container

Start streaming logs:
Server running at http://localhost:80
```

Cuando aparezca `Server running at http://localhost:80`, vaya al nombre de dominio completo del contenedor en el explorador para ver la aplicación en ejecución. El nombre de dominio completo debe haber aparecido en la salida del comando `az container create` que ejecutó en la sección anterior.

:::image type="content" source="media/container-registry-tutorial-quick-build/quick-build-02-browser.png" alt-text="Aplicación de ejemplo que se ejecuta en el explorador.":::

Para desasociar la consola del contenedor, pulse `Control+C`.

## <a name="clean-up-resources"></a>Limpieza de recursos

Detenga la instancia de contenedor con el comando [az container delete][az-container-delete]:

```azurecli
az container delete --resource-group $RES_GROUP --name acr-tasks
```

Para quitar *todos* los recursos que ha creado en este tutorial, incluido el registro de contenedor, el almacén de claves y la entidad de servicio, ejecute los comandos siguientes. Estos recursos se utilizarán en el [siguiente tutorial](container-registry-tutorial-build-task.md) de la serie, por lo que puede que quiera conservarlos si pasa directamente al tutorial siguiente.

```azurecli
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha probado el bucle interno con una tarea rápida, configure una **tarea de compilación** que desencadene las compilaciones de imágenes de contenedor cuando confirma el código fuente en un repositorio de Git:

> [!div class="nextstepaction"]
> [Desencadenamiento de compilaciones automáticas con tareas](container-registry-tutorial-build-task.md)

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az_ad_sp_create_for_rbac
[az-container-attach]: /cli/azure/container#az_container_attach
[az-container-create]: /cli/azure/container#az_container_create
[az-container-delete]: /cli/azure/container#az_container_delete
[az-keyvault-create]: /cli/azure/keyvault/secret#az_keyvault_create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az_keyvault_secret_set
[az-login]: /cli/azure/reference-index#az_login
[service-principal-auth]: container-registry-auth-service-principal.md

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
