---
title: Autenticación entre registros desde la tarea de ACR
description: Configuración de una tarea de Azure Container Registry (Tarea de ACR) para acceder a otro registro de contenedor privado de Azure mediante una identidad administrada de recursos de Azure
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: a9b70a44de0cfccb9a61bc24575281e440db6e32
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781131"
---
# <a name="cross-registry-authentication-in-an-acr-task-using-an-azure-managed-identity"></a>Autenticación entre registros en una tarea de ACR mediante una identidad administrada por Azure 

En una [tarea de ACR](container-registry-tasks-overview.md), puede [habilitar una identidad administrada para recursos de Azure](container-registry-tasks-authentication-managed-identity.md). La tarea puede usar la identidad para acceder a otros recursos de Azure, sin necesidad de proporcionar o de administrar credenciales. 

En este artículo aprenderá a habilitar una identidad administrada en una tarea que extrae una imagen de un registro diferente del que se usó para ejecutarla.

Para crear los recursos de Azure, en este artículo es necesario ejecutar la CLI de Azure versión 2.0.68 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

## <a name="scenario-overview"></a>Información general de escenario

La tarea de ejemplo extrae una imagen base de otro registro de contenedor de Azure para compilar e insertar una imagen de la aplicación. Para extraer la imagen base, configurará la tarea con una identidad administrada y le asignará los permisos adecuados. 

Los pasos que se muestran en este ejemplo utilizan una identidad administrada, bien asignada por el usuario o bien asignada por el sistema. Su elección de identidad dependerá de las necesidades de su organización.

En un escenario real, una organización puede mantener un conjunto de imágenes base usadas por todos los equipos de desarrollo para compilar sus aplicaciones. Estas imágenes base se almacenan en un registro corporativo, para el que cada equipo de desarrollo solo tiene derechos de extracción. 

## <a name="prerequisites"></a>Requisitos previos

En este artículo necesitará dos registros de contenedor de Azure:

* Utilice el primer registro para crear y ejecutar ACR Tasks. En este artículo, este registro se denomina *myregistry*. 
* El segundo registro hospeda una imagen base que se usa para que la tarea compile una imagen. En este artículo, el segundo registro se denomina *mybaseregistry*. 

Reemplácelos por sus propios nombres de registro en pasos posteriores.

Si aún no tiene los registros de contenedor de Azure necesarios, consulte [Inicio rápido: Creación de un registro de contenedor privado con la CLI de Azure](container-registry-get-started-azure-cli.md). No tiene que insertar imágenes en el registro todavía.

## <a name="prepare-base-registry"></a>Preparación del registro base

Para fines de demostración, como una operación única, ejecute [az acr import][az-acr-import] para importar una imagen de Node.js pública de Docker Hub en el registro base. En la práctica, otro equipo o proceso de la organización podría mantener imágenes en el registro base.

```azurecli
az acr import --name mybaseregistry \
  --source docker.io/library/node:15-alpine \
  --image baseimages/node:15-alpine 
```

## <a name="define-task-steps-in-yaml-file"></a>Definición de los pasos de la tarea en el archivo YAML

Los pasos de esta [tarea de varios pasos](container-registry-tasks-multi-step.md) de ejemplo se definen en un [archivo YAML](container-registry-tasks-reference-yaml.md). Cree un archivo denominado `helloworldtask.yaml` en el directorio de trabajo local y pegue en él el siguiente contenido. Actualice el valor de `REGISTRY_NAME` en el paso de compilación con el nombre de servidor del registro base.

```yml
version: v1.1.0
steps:
# Replace mybaseregistry with the name of your registry containing the base image
  - build: -t $Registry/hello-world:$ID  https://github.com/Azure-Samples/acr-build-helloworld-node.git#main -f Dockerfile-app --build-arg REGISTRY_NAME=mybaseregistry.azurecr.io
  - push: ["$Registry/hello-world:$ID"]
```

En el paso de compilación se usa el archivo `Dockerfile-app` del repositorio [Azure-Samples/acr-build-helloworld-node](https://github.com/Azure-Samples/acr-build-helloworld-node.git) para compilar una imagen. `--build-arg` hace referencia al registro base del que se extrae la imagen base. Cuando se compila correctamente, la imagen se inserta en el registro usado para ejecutar la tarea.

## <a name="option-1-create-task-with-user-assigned-identity"></a>Opción 1: Creación de una tarea con una identidad asignada por el sistema

En los pasos de esta sección se crea una tarea y se habilita una identidad asignada por el usuario. Si en su lugar quiere habilitar una identidad asignada por el sistema, consulte [Opción 2: Creación de una tarea con una identidad asignada por el sistema](#option-2-create-task-with-system-assigned-identity). 

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="create-task"></a>Crear la tarea

Cree la tarea *helloworldtask* mediante la ejecución del siguiente comando [az acr task create][az-acr-task-create]. La tarea se ejecuta sin contexto de código fuente y el comando hace referencia al archivo `helloworldtask.yaml` del directorio de trabajo. El parámetro `--assign-identity` pasa el identificador de recurso de la identidad asignada por el usuario. 

```azurecli
az acr task create \
  --registry myregistry \
  --name helloworldtask \
  --context /dev/null \
  --file helloworldtask.yaml \
  --assign-identity $resourceID
```

[!INCLUDE [container-registry-tasks-user-id-properties](../../includes/container-registry-tasks-user-id-properties.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Concesión a la identidad de permisos de extracción del registro base

En esta sección se conceden permisos a la identidad administrada para extraer del registro base, *mybaseregistry*.

Use el comando [az acr show][az-acr-show] para obtener el identificador de recurso del registro base y almacenarlo en una variable:

```azurecli
baseregID=$(az acr show --name mybaseregistry --query id --output tsv)
```

Use el comando [az role assignment create][az-role-assignment-create] para asignar a la identidad el rol `acrpull` al registro base. Este rol solo tiene permisos para extraer imágenes del registro.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

Continúe con la [Incorporación de las credenciales del registro de destino a la tarea](#add-target-registry-credentials-to-task).

## <a name="option-2-create-task-with-system-assigned-identity"></a>Opción 2: Creación de una tarea con una identidad asignada por el sistema

En los pasos de esta sección se crea una tarea y se habilita una identidad asignada por el sistema. Si, en cambio, quiere habilitar una identidad asignada por el usuario, consulte [Opción 1: Creación de una tarea con una identidad asignada por el usuario](#option-1-create-task-with-user-assigned-identity). 

### <a name="create-task"></a>Crear la tarea

Cree la tarea *helloworldtask* mediante la ejecución del siguiente comando [az acr task create][az-acr-task-create]. La tarea se ejecuta sin contexto de código fuente y el comando hace referencia al archivo `helloworldtask.yaml` del directorio de trabajo. El parámetro `--assign-identity` sin valor habilita la identidad asignada por el sistema en la tarea. 

```azurecli
az acr task create \
  --registry myregistry \
  --name helloworldtask \
  --context /dev/null \
  --file helloworldtask.yaml \
  --assign-identity 
```
[!INCLUDE [container-registry-tasks-system-id-properties](../../includes/container-registry-tasks-system-id-properties.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Concesión a la identidad de permisos de extracción del registro base

En esta sección se conceden permisos a la identidad administrada para extraer del registro base, *mybaseregistry*.

Use el comando [az acr show][az-acr-show] para obtener el identificador de recurso del registro base y almacenarlo en una variable:

```azurecli
baseregID=$(az acr show --name mybaseregistry --query id --output tsv)
```

Use el comando [az role assignment create][az-role-assignment-create] para asignar a la identidad el rol `acrpull` al registro base. Este rol solo tiene permisos para extraer imágenes del registro.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

## <a name="add-target-registry-credentials-to-task"></a>Incorporación de las credenciales del registro de destino a la tarea

Use ahora el comando [az acr task credential add][az-acr-task-credential-add] para permitir que la tarea se autentique con el registro base mediante las credenciales de la identidad. Ejecute el comando correspondiente al tipo de identidad administrada que habilitó en la tarea. Si ha habilitado una identidad asignada por el usuario, pase `--use-identity` con el identificador de cliente de la identidad. Si ha habilitado una identidad asignada por el sistema, pase `--use-identity [system]`.

```azurecli
# Add credentials for user-assigned identity to the task
az acr task credential add \
  --name helloworldtask \
  --registry myregistry \
  --login-server mybaseregistry.azurecr.io \
  --use-identity $clientID

# Add credentials for system-assigned identity to the task
az acr task credential add \
  --name helloworldtask \
  --registry myregistry \
  --login-server mybaseregistry.azurecr.io \
  --use-identity [system]
```

## <a name="manually-run-the-task"></a>Ejecución manual de la tarea

Para comprobar que la tarea en la que habilitó una identidad administrada se ejecuta correctamente, desencadene manualmente la tarea con el comando [az acr task run][az-acr-task-run]. 

```azurecli
az acr task run \
  --name helloworldtask \
  --registry myregistry
```

Si la tarea se ejecuta correctamente, la salida es similar a:

```
Queued a run with ID: cf10
Waiting for an agent...
2019/06/14 22:47:32 Using acb_vol_dbfbe232-fd76-4ca3-bd4a-687e84cb4ce2 as the home volume
2019/06/14 22:47:39 Creating Docker network: acb_default_network, driver: 'bridge'
2019/06/14 22:47:40 Successfully set up Docker network: acb_default_network
2019/06/14 22:47:40 Setting up Docker configuration...
2019/06/14 22:47:41 Successfully set up Docker configuration
2019/06/14 22:47:41 Logging in to registry: myregistry.azurecr.io
2019/06/14 22:47:42 Successfully logged into myregistry.azurecr.io
2019/06/14 22:47:42 Logging in to registry: mybaseregistry.azurecr.io
2019/06/14 22:47:43 Successfully logged into mybaseregistry.azurecr.io
2019/06/14 22:47:43 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/14 22:47:43 Scanning for dependencies...
2019/06/14 22:47:45 Successfully scanned dependencies
2019/06/14 22:47:45 Launching container with name: acb_step_0
Sending build context to Docker daemon   25.6kB
Step 1/6 : ARG REGISTRY_NAME
Step 2/6 : FROM ${REGISTRY_NAME}/baseimages/node:15-alpine
15-alpine: Pulling from baseimages/node
[...]
Successfully built 41b49a112663
Successfully tagged myregistry.azurecr.io/hello-world:cf10
2019/06/14 22:47:56 Successfully executed container: acb_step_0
2019/06/14 22:47:56 Executing step ID: acb_step_1. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/14 22:47:56 Pushing image: myregistry.azurecr.io/hello-world:cf10, attempt 1
The push refers to repository [myregistry.azurecr.io/hello-world]
[...]
2019/06/14 22:48:00 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 2.517011)
2019/06/14 22:48:00 The following dependencies were found:
2019/06/14 22:48:00
- image:
    registry: myregistry.azurecr.io
    repository: hello-world
    tag: cf10
    digest: sha256:611cf6e3ae3cb99b23fadcd89fa144e18aa1b1c9171ad4a0da4b62b31b4e38d1
  runtime-dependency:
    registry: mybaseregistry.azurecr.io
    repository: baseimages/node
    tag: 15-alpine
    digest: sha256:e8e92cffd464fce3be9a3eefd1b65dc9cbe2484da31c11e813a4effc6105c00f
  git:
    git-head-revision: 0f988779c97fe0bfc7f2f74b88531617f4421643

Run ID: cf10 was successful after 32s
```

Ejecute el comando [az acr repository show-tags][az-acr-repository-show-tags] para comprobar que la imagen se ha compilado y se ha insertado correctamente en *myregistry*:

```azurecli
az acr repository show-tags --name myregistry --repository hello-world --output tsv
```

Salida de ejemplo:

```console
cf10
```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [cómo habilitar una identidad administrada en una tarea de ACR](container-registry-tasks-authentication-managed-identity.md).
* Consulte la [referencia de YAML de tareas de ACR](container-registry-tasks-reference-yaml.md).

<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az_login
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az_acr_repository_show_tags
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
[az-acr-login]: /cli/azure/acr#az_acr_login
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-show]: /cli/azure/acr/task#az_acr_task_show
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az_acr_task_credential_add
[az-group-create]: /cli/azure/group?#az_group_create
