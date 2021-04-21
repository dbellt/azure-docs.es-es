---
title: Autenticación externa desde una tarea de ACR
description: Configure una tarea de Azure Container Registry (tarea de ACR) para leer las credenciales de Docker Hub almacenadas en un almacén de claves de Azure mediante una identidad administrada para recursos de Azure.
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: 16404f9244818d91c5333eb5eec5944bfdd9df98
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781206"
---
# <a name="external-authentication-in-an-acr-task-using-an-azure-managed-identity"></a>Autenticación externa en una tarea de ACR mediante una identidad administrada por Azure 

En una [tarea de ACR](container-registry-tasks-overview.md), puede [habilitar una identidad administrada para recursos de Azure](container-registry-tasks-authentication-managed-identity.md). La tarea puede usar la identidad para acceder a otros recursos de Azure, sin necesidad de proporcionar o de administrar credenciales. 

En este artículo, aprenderá a habilitar una identidad administrada en una tarea que tiene acceso a los secretos almacenados en un almacén de claves de Azure. 

Para crear los recursos de Azure, en este artículo es necesario ejecutar la CLI de Azure versión 2.0.68 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

## <a name="scenario-overview"></a>Información general de escenario

La tarea de ejemplo lee las credenciales de Docker Hub almacenadas en un almacén de claves de Azure. Las credenciales son para una cuenta de Docker Hub con permisos de escritura (envío de cambios) en un repositorio privado de Docker Hub. Para leer las credenciales, configurará la tarea con una identidad administrada y le asignará los permisos adecuados. La tarea asociada a la identidad crea una imagen e inicia sesión en Docker Hub para insertar la imagen en el repositorio privado. 

Los pasos que se muestran en este ejemplo utilizan una identidad administrada, bien asignada por el usuario o bien asignada por el sistema. Su elección de identidad dependerá de las necesidades de su organización.

En un escenario real, una empresa puede publicar imágenes en un repositorio privado de Docker Hub como parte de un proceso de compilación. 

## <a name="prerequisites"></a>Requisitos previos

Necesita un registro de contenedor de Azure en el que ejecutar la tarea. En este artículo, este registro se denomina *myregistry*. Reemplácelo por su propio nombre de registro en los pasos posteriores.

Si aún no tiene un registro de contenedor de Azure, consulte [Inicio rápido: Creación de un registro de contenedor privado con la CLI de Azure](container-registry-get-started-azure-cli.md). No tiene que insertar imágenes en el registro todavía.

También necesita un repositorio privado en Docker Hub y una cuenta de Docker Hub con permisos para escribir en el repositorio. En este ejemplo, este repositorio se denomina *hubuser/hubrepo*. 

## <a name="create-a-key-vault-and-store-secrets"></a>Creación de un almacén de claves y almacenamiento de secretos

En primer lugar, y si es necesario, cree un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus* con el comando siguiente [az group create][az-group-create]:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Use el comando [az keyvault create][az-keyvault-create] para crear un almacén de claves. Asegúrese de especificar un nombre de almacén de claves único. 

```azurecli-interactive
az keyvault create --name mykeyvault --resource-group myResourceGroup --location eastus
```

Almacene las credenciales necesarias de Docker Hub en el almacén de claves mediante el comando [az keyvault secret set][az-keyvault-secret-set]. En estos comandos, los valores se pasan en las variables de entorno:

```azurecli
# Store Docker Hub user name
az keyvault secret set \
  --name UserName \
  --value $USERNAME \
  --vault-name mykeyvault

# Store Docker Hub password
az keyvault secret set \
  --name Password \
  --value $PASSWORD \
  --vault-name mykeyvault
```

En un escenario real, es probable que los secretos se establezcan y se mantengan en un proceso independiente.

## <a name="define-task-steps-in-yaml-file"></a>Definición de los pasos de la tarea en el archivo YAML

Los pasos de esta tarea de ejemplo se definen en un [archivo YAML](container-registry-tasks-reference-yaml.md). Cree un archivo denominado `dockerhubtask.yaml` en un directorio de trabajo local y pegue el siguiente contenido: Asegúrese de reemplazar el nombre del almacén de claves del archivo por el nombre de su almacén de claves.

```yml
version: v1.1.0
# Replace mykeyvault with the name of your key vault
secrets:
  - id: username
    keyvault: https://mykeyvault.vault.azure.net/secrets/UserName
  - id: password
    keyvault: https://mykeyvault.vault.azure.net/secrets/Password
steps:
# Log in to Docker Hub
  - cmd: bash echo '{{.Secrets.password}}' | docker login --username '{{.Secrets.username}}' --password-stdin 
# Build image
  - build: -t {{.Values.PrivateRepo}}:$ID https://github.com/Azure-Samples/acr-tasks.git -f hello-world.dockerfile
# Push image to private repo in Docker Hub
  - push:
    - {{.Values.PrivateRepo}}:$ID
```

Los pasos de la tarea son los siguientes:

* Administrar las credenciales de secretos para autenticarse en Docker Hub
* Pasar los secretos al comando `docker login` para autenticarse en Docker Hub
* Crear una imagen con un Dockerfile de ejemplo en el repositorio [Azure-Samples/acr-tasks](https://github.com/Azure-Samples/acr-tasks.git).
* Insertar la imagen en el repositorio privado de Docker Hub


## <a name="option-1-create-task-with-user-assigned-identity"></a>Opción 1: Creación de una tarea con una identidad asignada por el sistema

En los pasos de esta sección se crea una tarea y se habilita una identidad asignada por el usuario. Si en su lugar quiere habilitar una identidad asignada por el sistema, consulte [Opción 2: Creación de una tarea con una identidad asignada por el sistema](#option-2-create-task-with-system-assigned-identity). 

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="create-task"></a>Crear la tarea

Cree la tarea *dockerhubtask* mediante la ejecución del comando [az acr task create][az-acr-task-create]. La tarea se ejecuta sin contexto de código fuente y el comando hace referencia al archivo `dockerhubtask.yaml` del directorio de trabajo. El parámetro `--assign-identity` pasa el identificador de recurso de la identidad asignada por el usuario. 

```azurecli
az acr task create \
  --name dockerhubtask \
  --registry myregistry \
  --context /dev/null \
  --file dockerhubtask.yaml \
  --assign-identity $resourceID
```

[!INCLUDE [container-registry-tasks-user-id-properties](../../includes/container-registry-tasks-user-id-properties.md)]


### <a name="grant-identity-access-to-key-vault"></a>Concesión a la identidad acceso al almacén de claves

Ejecute el comando [az keyvault set-policy][az-keyvault-set-policy] siguiente para establecer una directiva de acceso en el almacén de claves. En el ejemplo siguiente se permite que la identidad lea secretos del almacén de claves. 

```azurecli
az keyvault set-policy --name mykeyvault \
  --resource-group myResourceGroup \
  --object-id $principalID \
  --secret-permissions get
```

Proceda a [Ejecutar manualmente la tarea](#manually-run-the-task).

## <a name="option-2-create-task-with-system-assigned-identity"></a>Opción 2: Creación de una tarea con una identidad asignada por el sistema

En los pasos de esta sección se crea una tarea y se habilita una identidad asignada por el sistema. Si, en cambio, quiere habilitar una identidad asignada por el usuario, consulte [Opción 1: Creación de una tarea con una identidad asignada por el usuario](#option-1-create-task-with-user-assigned-identity). 

### <a name="create-task"></a>Crear la tarea

Cree la tarea *dockerhubtask* mediante la ejecución del comando [az acr task create][az-acr-task-create]. La tarea se ejecuta sin contexto de código fuente y el comando hace referencia al archivo `dockerhubtask.yaml` del directorio de trabajo. El parámetro `--assign-identity` sin valor habilita la identidad asignada por el sistema en la tarea.  

```azurecli
az acr task create \
  --name dockerhubtask \
  --registry myregistry \
  --context /dev/null \
  --file dockerhubtask.yaml \
  --assign-identity 
```

[!INCLUDE [container-registry-tasks-system-id-properties](../../includes/container-registry-tasks-system-id-properties.md)]

### <a name="grant-identity-access-to-key-vault"></a>Concesión a la identidad acceso al almacén de claves

Ejecute el comando [az keyvault set-policy][az-keyvault-set-policy] siguiente para establecer una directiva de acceso en el almacén de claves. En el ejemplo siguiente se permite que la identidad lea secretos del almacén de claves. 

```azurecli
az keyvault set-policy --name mykeyvault \
  --resource-group myResourceGroup \
  --object-id $principalID \
  --secret-permissions get
```

## <a name="manually-run-the-task"></a>Ejecución manual de la tarea

Para comprobar que la tarea en la que habilitó una identidad administrada se ejecuta correctamente, desencadene manualmente la tarea con el comando [az acr task run][az-acr-task-run]. El parámetro `--set` se usa para pasar el nombre del repositorio privado a la tarea. En este ejemplo, el nombre del repositorio del marcador de posición es *hubuser/hubrepo*.

```azurecli
az acr task run --name dockerhubtask --registry myregistry --set PrivateRepo=hubuser/hubrepo
```

Cuando la tarea se ejecuta correctamente, la salida muestra una autenticación correcta en Docker Hub y la imagen se crea y se inserta correctamente en el repositorio privado:

```console
Queued a run with ID: cf24
Waiting for an agent...
2019/06/20 18:05:55 Using acb_vol_b1edae11-30de-4f2b-a9c7-7d743e811101 as the home volume
2019/06/20 18:05:58 Creating Docker network: acb_default_network, driver: 'bridge'
2019/06/20 18:05:58 Successfully set up Docker network: acb_default_network
2019/06/20 18:05:58 Setting up Docker configuration...
2019/06/20 18:05:59 Successfully set up Docker configuration
2019/06/20 18:05:59 Logging in to registry: myregistry.azurecr.io
2019/06/20 18:06:00 Successfully logged into myregistry.azurecr.io
2019/06/20 18:06:00 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/20 18:06:00 Launching container with name: acb_step_0
[...]
Login Succeeded
2019/06/20 18:06:02 Successfully executed container: acb_step_0
2019/06/20 18:06:02 Executing step ID: acb_step_1. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2019/06/20 18:06:02 Scanning for dependencies...
2019/06/20 18:06:04 Successfully scanned dependencies
2019/06/20 18:06:04 Launching container with name: acb_step_1
Sending build context to Docker daemon    129kB
[...]
2019/06/20 18:06:07 Successfully pushed image: hubuser/hubrepo:cf24
2019/06/20 18:06:07 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 2.064353)
2019/06/20 18:06:07 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 2.594061)
2019/06/20 18:06:07 Populating digests for step ID: acb_step_1...
2019/06/20 18:06:09 Successfully populated digests for step ID: acb_step_1
2019/06/20 18:06:09 Step ID: acb_step_2 marked as successful (elapsed time in seconds: 2.743923)
2019/06/20 18:06:09 The following dependencies were found:
2019/06/20 18:06:09
- image:
    registry: registry.hub.docker.com
    repository: hubuser/hubrepo
    tag: cf24
    digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: latest
    digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
  git:
    git-head-revision: b0ffa6043dd893a4c75644c5fed384c82ebb5f9e

Run ID: cf24 was successful after 15s
```

Para confirmar que la imagen se ha insertado, busque la etiqueta (`cf24` en este ejemplo) en el repositorio privado de Docker Hub.

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
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-identity-show]: /cli/azure/identity#az_identity_show
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-show]: /cli/azure/acr/task#az_acr_task_show
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az_acr_task_credential_add
[az-group-create]: /cli/azure/group?#az_group_create
[az-keyvault-create]: /cli/azure/keyvault?#az_keyvault_create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az_keyvault_secret_set
[az-keyvault-set-policy]: /cli/azure/keyvault#az_keyvault_set_policy
