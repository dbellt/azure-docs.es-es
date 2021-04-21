---
title: Ejecución rápida de tareas con una plantilla
description: Ponga en cola una tarea de ACR Tasks que desee ejecutar para compilar una imagen con una plantilla de Azure Resource Manager
ms.topic: article
ms.date: 04/22/2020
ms.openlocfilehash: af7bebc311f81bb489fcc8be419f167ff6f9460a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781242"
---
# <a name="run-acr-tasks-using-resource-manager-templates"></a>Ejecución de ACR Tasks mediante plantillas de Resource Manager

[ACR Tasks](container-registry-tasks-overview.md) es un conjunto de características de Azure Container Registry que le ayudan a administrar y modificar imágenes de contenedor durante el ciclo de vida de este. 

Este artículo contiene ejemplos de plantillas de Azure Resource Manager que ponen en cola una ejecución rápida de tareas, algo parecido a lo que puede hacer manualmente con el comando [az acr build][az-acr-build].

Utilizar una plantilla de Resource Manager para poner en cola la ejecución de una tarea resulta útil en escenarios de automatización y amplía la funcionalidad de `az acr build`. Por ejemplo:

* Utilice una plantilla para crear un registro de contenedor y poner en cola inmediatamente la ejecución de una tarea para compilar e insertar una imagen de contenedor
* Cree o habilite otros recursos que pueda usar en una ejecución rápida de tarea, como una identidad administrada de recursos de Azure.

## <a name="limitations"></a>Limitaciones

* Debe especificar un contexto remoto (por ejemplo, un repositorio de GitHub) como [ubicación de origen](container-registry-tasks-overview.md#context-locations) para la ejecución de la tarea. No se puede usar un contexto de origen local.
* En el caso de las ejecuciones de tareas en la que se utiliza una identidad administrada, solo se permite una identidad administrada *asignada por el usuario*.

## <a name="prerequisites"></a>Requisitos previos

* **Cuenta de GitHub**: Cree una cuenta en https://github.com si aún no tiene una. 
* **Bifurcar el repositorio de ejemplo**: en los ejemplos de tareas que se mencionan aquí, utilice la interfaz de usuario de GitHub para bifurcar el siguiente repositorio de ejemplo en la cuenta de GitHub: https://github.com/Azure-Samples/acr-build-helloworld-node. Este repositorio contiene archivos Dockerfile y código fuente de ejemplo para compilar imágenes de contenedor pequeñas.

## <a name="example-create-registry-and-queue-task-run"></a>Ejemplo: Creación de un registro y una ejecución de tarea en cola

En este ejemplo, se usa una [plantilla de ejemplo](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment/quickdockerbuild) para crear un registro de contenedor y poner en cola una ejecución de tarea que compila e inserta una imagen. 

### <a name="template-parameters"></a>Parámetros de plantilla

En este ejemplo, proporcione valores para los siguientes parámetros de la plantilla:

|Parámetro  |Value  |
|---------|---------|
|registryName     |Nombre único del registro creado         |
|repository     |Repositorio de destino de la tarea de compilación        |
|taskRunName     |Nombre de la ejecución de la tarea, que especifica la etiqueta de la imagen |
|sourceLocation     |Contexto remoto de la tarea de compilación; por ejemplo, https://github.com/Azure-Samples/acr-build-helloworld-node. El archivo Dockerfile del repositorio compila una imagen de contenedor de una aplicación web de Node.js pequeña. Si lo desea, puede usar la bifurcación del repositorio como contexto de compilación.         |

### <a name="deploy-the-template"></a>Implementación de la plantilla

Implemente la plantilla con el comando [az deployment group create][az-deployment-group-create]. En este ejemplo, la imagen *helloworld-node:testrun* se compila y se inserta en un registro llamado *mycontainerregistry*.

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/acr/master/docs/tasks/run-as-deployment/quickdockerbuild/azuredeploy.json \
  --parameters \
    registryName=mycontainerregistry \
    repository=helloworld-node \
    taskRunName=testrun \
    sourceLocation=https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
 ```

El comando anterior pasa los parámetros en la línea de comandos. Si lo prefiere, puede pasarlos en un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md).

### <a name="verify-deployment"></a>Comprobación de la implementación

Una vez que la implementación finalice correctamente, compruebe que la imagen esté compilada ejecutando [az acr repository show-tags][az-acr-repository-show-tags]:

```azurecli
az acr repository show-tags \
  --name mycontainerregistry \
  --repository helloworld-node --output table
```

Salida:

```console
Result
--------
testrun
```

### <a name="view-run-log"></a>Consulta del registro de ejecución

Para ver detalles acerca de la ejecución de la tarea, consulte el registro de ejecución.

Primero obtenga el identificador de ejecución con [az acr task list-runs][az-acr-task-list-runs].
```azurecli
az acr task list-runs \
  --registry mycontainerregistry --output table
```

La salida es parecida a esta:

```console
RUN ID    TASK    PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  ------  ----------  ---------  ---------  --------------------  ----------
ca1               linux       Succeeded  Manual     2020-03-23T17:54:28Z  00:00:48
```

Ejecute [az acr task logs][az-acr-task-logs] para buscar el identificador de ejecución en los registros de ejecución de tareas; en este caso, *ca1*:

```azurecli
az acr task logs \
  --registry mycontainerregistry \
  --run-id ca1
```

En la salida se muestra el registro de ejecución de la tarea.

También puede ver el registro de ejecución de tareas en Azure Portal. 

1. Vaya al registro de contenedor.
2. En **Servicios**, seleccione **Tareas** > **Ejecuciones**.
3. Seleccione el identificador de ejecución; en este caso, *ca1*. 

En el portal se mostrará el registro de ejecución de tareas.

## <a name="example-task-run-with-managed-identity"></a>Ejemplo: Ejecución de tareas con una identidad administrada

Use una [plantilla de ejemplo](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment/quickdockerbuildwithidentity) para poner en cola la ejecución de una tarea que permita usar una identidad administrada asignada por el usuario. Durante la ejecución de la tarea, la identidad se autenticará para extraer una imagen de otro registro de contenedor de Azure. 

Este escenario es similar al de [Autenticación entre registros en una tarea de ACR mediante una identidad administrada por Azure](container-registry-tasks-cross-registry-authentication.md). Imagine, por ejemplo, una organización que tiene un registro centralizado con imágenes base a las que tienen acceso varios equipos de desarrollo.

### <a name="prepare-base-registry"></a>Preparación del registro base

A efectos de esta demostración, cree un registro de contenedor independiente como registro base e inserte una imagen base de Node. js extraída de Docker Hub.

1. Cree un segundo registro de contenedor (por ejemplo, *mybaseregistry*) para almacenar las imágenes base.
1. Extraiga la imagen de `node:9-alpine` de Docker Hub, etiquétela para el registro base e insértela en este registro:

  ```azurecli
  docker pull node:9-alpine
  docker tag node:9-alpine mybaseregistry.azurecr.io/baseimages/node:9-alpine
  az acr login -n mybaseregistry
  docker push mybaseregistry.azurecr.io/baseimages/node:9-alpine
  ```

### <a name="create-new-dockerfile"></a>Creación de un nuevo archivo Dockerfile

Cree un archivo Dockerfile que extraiga la imagen base del registro base. Realice los pasos siguientes en la bifurcación local del repositorio de GitHub; por ejemplo, `https://github.com/myGitHubID/acr-build-helloworld-node.git`.

1. En la interfaz de usuario de GitHub, seleccione **Create new file** (Crear nuevo archivo).
1. Llame *Dockerfile-test* al archivo y pegue el siguiente contenido. Cambie el nombre del registro por *mybaseregistry*.
    ```
    FROM mybaseregistry.azurecr.io/baseimages/node:9-alpine
    COPY . /src
    RUN cd /src && npm install
    EXPOSE 80
    CMD ["node", "/src/server.js"]
    ```
 1. Seleccione **Commit new file** (Confirmar nuevo archivo).

[!INCLUDE [container-registry-tasks-user-assigned-id](../../includes/container-registry-tasks-user-assigned-id.md)]

### <a name="give-identity-pull-permissions-to-the-base-registry"></a>Concesión a la identidad de permisos de extracción del registro base

Conceda permisos a la identidad administrada para extraer contenido del registro base, *mybaseregistry*.

Use el comando [az acr show][az-acr-show] para obtener el identificador de recurso del registro base y almacenarlo en una variable:

```azurecli
baseregID=$(az acr show \
  --name mybaseregistry \
  --query id --output tsv)
```

Use el comando [az role assignment create][az-role-assignment-create] para asignar a la identidad el rol Acrpull en el registro base. Este rol solo tiene permisos para extraer imágenes del registro.

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID \
  --role acrpull
```

### <a name="template-parameters"></a>Parámetros de plantilla

En este ejemplo, proporcione valores para los siguientes parámetros de la plantilla:

|Parámetro  |Value  |
|---------|---------|
|registryName     |Nombre del registro en el que se compila la imagen  |
|repository     |Repositorio de destino de la tarea de compilación        |
|taskRunName     |Nombre de la ejecución de la tarea, que especifica la etiqueta de la imagen |
|userAssignedIdentity |Identificador de recurso de la identidad asignada por el usuario que se ha habilitado en la tarea.|
|customRegistryIdentity | Identificador de cliente de la identidad asignada por el usuario que se ha habilitado en la tarea. Este identificador se usa en la autenticación del registro personalizado. |
|customRegistry |Nombre del servidor de inicio de sesión del registro personalizado al que se accede en la tarea; por ejemplo, *mybaseregistry.azurecr.io*|
|sourceLocation     |Contexto remoto para la tarea de compilación, por ejemplo, *https://github.com/\<your-GitHub-ID\> /acr-build-helloworld-node.* |
|dockerFilePath | Ruta de acceso del archivo Dockerfile en el contexto remoto, que se usa para compilar la imagen. |

### <a name="deploy-the-template"></a>Implementación de la plantilla

Implemente la plantilla con el comando [az deployment group create][az-deployment-group-create]. En este ejemplo, la imagen *helloworld-node:testrun* se compila y se inserta en un registro llamado *mycontainerregistry*. La imagen base se extrae de *mybaseregistry.azurecr.io*.

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/acr/master/docs/tasks/run-as-deployment/quickdockerbuildwithidentity/azuredeploy.json \
  --parameters \
    registryName=mycontainerregistry \
    repository=helloworld-node \
    taskRunName=basetask \
    userAssignedIdentity=$resourceID \
    customRegistryIdentity=$clientID \
    sourceLocation=https://github.com/<your-GitHub-ID>/acr-build-helloworld-node.git#main \
    dockerFilePath=Dockerfile-test \
    customRegistry=mybaseregistry.azurecr.io
```

El comando anterior pasa los parámetros en la línea de comandos. Si lo prefiere, puede pasarlos en un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md).

### <a name="verify-deployment"></a>Comprobación de la implementación

Una vez que la implementación finalice correctamente, compruebe que la imagen esté compilada ejecutando [az acr repository show-tags][az-acr-repository-show-tags]:

```azurecli
az acr repository show-tags \
  --name mycontainerregistry \
  --repository helloworld-node --output table
```

Salida:

```console
Result
--------
basetask
```

### <a name="view-run-log"></a>Consulta del registro de ejecución

Para ver el registro de ejecución, consulte los pasos de la [sección anterior](#view-run-log).

## <a name="next-steps"></a>Pasos siguientes

 * Vea más ejemplos de plantillas en el [repositorio de ACR GitHub](https://github.com/Azure/acr/tree/master/docs/tasks/run-as-deployment).
 * Para más información sobre las propiedades de plantilla, consulte la referencia de plantillas de [ejecuciones de tareas](/azure/templates/microsoft.containerregistry/2019-06-01-preview/registries/taskruns) y [tareas](/azure/templates/microsoft.containerregistry/2019-06-01-preview/registries/tasks).


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-logs]: /cli/azure/acr/task#az_acr_task_logs
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az_acr_repository_show_tags
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-deployment-group-create]: /cli/azure/deployment/group#az_deployment_group_create
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-identity-show]: /cli/azure/identity#az_identity_show
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
