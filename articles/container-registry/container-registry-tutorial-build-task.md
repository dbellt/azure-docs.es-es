---
title: 'Tutorial: Creación de una imagen en la confirmación de código'
description: En este tutorial, aprenderá a configurar una tarea de Azure Container Registry Tasks que desencadena automáticamente compilaciones de imágenes de contenedor en la nube cuando envía código fuente a un repositorio Git.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: b4806ed30319ff058df6dfae0340a73ad4cb6132
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780792"
---
# <a name="tutorial-automate-container-image-builds-in-the-cloud-when-you-commit-source-code"></a>Tutorial: Automatización de las compilaciones de imágenes de contenedor en la nube al confirmar código fuente

Además de [tareas rápidas](container-registry-tutorial-quick-task.md), ACR Tasks admite compilaciones de imágenes de contenedor Docker automatizadas en la nube al confirmar el código fuente en un repositorio de Git. Los contextos de Git admitidos para ACR Tasks incluyen GitHub público o privado y Azure Repos.

> [!NOTE]
> Actualmente, ACR Tasks no admite desencadenadores de solicitud de confirmación o de extracción en repositorios de GitHub Enterprise.

En este tutorial, la tarea ACR compila e inserta una única imagen de contenedor especificada en un archivo Dockerfile cuando confirma el código fuente en un repositorio de Git. Para crear una [tarea de varios pasos](container-registry-tasks-multi-step.md) que use un archivo YAML para definir los pasos para compilar, insertar y, opcionalmente, probar varios contenedores durante la confirmación del código, consulte el [Tutorial: Ejecución de un flujo de trabajo de contenedor de varios pasos en la nube al confirmar el código fuente](container-registry-tutorial-multistep-task.md). Para información general sobre ACR Tasks, consulte [Automatización de la aplicación de revisiones de sistema operativo y plataforma con ACR Tasks](container-registry-tasks-overview.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crea una tarea.
> * Prueba de la tarea
> * Visualización del estado de la tarea
> * Desencadenamiento de la tarea con una confirmación de código

En este tutorial se da por hecho que ya ha realizado los pasos de los [tutoriales anteriores](container-registry-tutorial-quick-task.md). Si aún no lo ha hecho, complete los pasos de la sección [Requisitos previos](container-registry-tutorial-quick-task.md#prerequisites) del tutorial anterior antes de continuar.

[!INCLUDE [container-registry-task-tutorial-prereq.md](../../includes/container-registry-task-tutorial-prereq.md)]
[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="create-the-build-task"></a>Creación de la tarea de compilación

Ahora que ha completado los pasos necesarios para habilitar ACR Tasks para que lea el estado de confirmación y cree webhooks en un repositorio, puede crear una tarea que desencadene una compilación de imagen de contenedor después de las confirmaciones en el repositorio.

Primero, rellene estas variables de entorno de shell con valores adecuados para el entorno. Este paso no es estrictamente necesario, pero hace que la ejecución de los comandos de varias líneas de la CLI de Azure en este tutorial sea un poco más fácil. Si no rellena estas variables de entorno, debe reemplazar manualmente cada valor siempre que aparezca en los comandos de ejemplo.

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the previous section
```

Ahora, cree la tarea mediante la ejecución del siguiente comando [az acr task create][az-acr-task-create]:

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file Dockerfile \
    --git-access-token $GIT_PAT
```


Esta tarea determina que siempre que se confirme código en la rama *maestra* del repositorio especificado por `--context`, ACR Tasks compilará la imagen de contenedor desde el código de esa rama. La instancia de Dockerfile que especifica `--file` en la raíz del repositorio se usa para compilar la imagen. El argumento `--image` especifica un valor parametrizado de `{{.Run.ID}}` para la porción de la versión de la etiqueta de la imagen, lo que garantiza que la imagen compilada es correlativa a una compilación específica y que está etiquetada de forma exclusiva.

La salida de un comando [az acr task create][az-acr-task-create] que se ha ejecutado correctamente debe ser parecida a la siguiente:

```output
{
  "agentConfiguration": {
    "cpu": 2
  },
  "creationDate": "2010-11-19T22:42:32.972298+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myregistry/providers/Microsoft.ContainerRegistry/registries/myregistry/tasks/taskhelloworld",
  "location": "westcentralus",
  "name": "taskhelloworld",
  "platform": {
    "architecture": "amd64",
    "os": "Linux",
    "variant": null
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "myregistry",
  "status": "Enabled",
  "step": {
    "arguments": [],
    "baseImageDependencies": null,
    "contextPath": "https://github.com/gituser/acr-build-helloworld-node#main",
    "dockerFilePath": "Dockerfile",
    "imageNames": [
      "helloworld:{{.Run.ID}}"
    ],
    "isPushEnabled": true,
    "noCache": false,
    "type&quot;: &quot;Docker"
  },
  "tags": null,
  "timeout": 3600,
  "trigger": {
    "baseImageTrigger": {
      "baseImageTriggerType": "Runtime",
      "name": "defaultBaseimageTriggerName",
      "status&quot;: &quot;Enabled"
    },
    "sourceTriggers": [
      {
        "name": "defaultSourceTriggerName",
        "sourceRepository": {
          "branch": "main",
          "repositoryUrl": "https://github.com/gituser/acr-build-helloworld-node#main",
          "sourceControlAuthProperties": null,
          "sourceControlType&quot;: &quot;GitHub"
        },
        "sourceTriggerEvents": [
          "commit"
        ],
        "status&quot;: &quot;Enabled"
      }
    ]
  },
  "type&quot;: &quot;Microsoft.ContainerRegistry/registries/tasks"
}
```

## <a name="test-the-build-task"></a>Comprobación de la tarea de compilación

Ahora tiene una tarea que define la compilación. Para probar la canalización de compilación, desencadene una compilación manualmente mediante la ejecución del comando [az acr task run][az-acr-task-run]:

```azurecli
az acr task run --registry $ACR_NAME --name taskhelloworld
```

De forma predeterminada, el comando `az acr task run` hace streaming de la salida del registro a la consola cuando se ejecuta el comando. La salida está condensada para mostrar los pasos principales.

```output
2020/11/19 22:51:00 Using acb_vol_9ee1f28c-4fd4-43c8-a651-f0ed027bbf0e as the home volume
2020/11/19 22:51:00 Setting up Docker configuration...
2020/11/19 22:51:02 Successfully set up Docker configuration
2020/11/19 22:51:02 Logging in to registry: myregistry.azurecr.io
2020/11/19 22:51:03 Successfully logged in
2020/11/19 22:51:03 Executing step: build
2020/11/19 22:51:03 Obtaining source code and scanning for dependencies...
2020/11/19 22:51:05 Successfully obtained source code and scanned for dependencies
Sending build context to Docker daemon  23.04kB
Step 1/5 : FROM node:15-alpine
[...]
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in 7382eea2a56a
Removing intermediate container 7382eea2a56a
 ---> e33cd684027b
Successfully built e33cd684027b
Successfully tagged myregistry.azurecr.io/helloworld:da2
2020/11/19 22:51:11 Executing step: push
2020/11/19 22:51:11 Pushing image: myregistry.azurecr.io/helloworld:da2, attempt 1
The push refers to repository [myregistry.azurecr.io/helloworld]
4a853682c993: Preparing
[...]
4a853682c993: Pushed
[...]
da2: digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419 size: 1366
2020/11/19 22:51:21 Successfully pushed image: myregistry.azurecr.io/helloworld:da2
2020/11/19 22:51:21 Step id: build marked as successful (elapsed time in seconds: 7.198937)
2020/11/19 22:51:21 Populating digests for step id: build...
2020/11/19 22:51:22 Successfully populated digests for step id: build
2020/11/19 22:51:22 Step id: push marked as successful (elapsed time in seconds: 10.180456)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloworld
    tag: da2
    digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 9-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 68cdf2a37cdae0873b8e2f1c4d80ca60541029bf


Run ID: ca6 was successful after 27s
```

## <a name="trigger-a-build-with-a-commit"></a>Desencadenamiento de una compilación con una confirmación

Ahora que ha probado la tarea ejecutándola manualmente, desencadénela automáticamente con un cambio en el código fuente.

En primer lugar, asegúrese de que se encuentra en el directorio que contiene el clon local del [repositorio][sample-repo]:

```console
cd acr-build-helloworld-node
```

A continuación, ejecute los siguientes comandos para crear, confirmar e insertar un nuevo archivo en la bifurcación del repositorio en GitHub:

```console
echo "Hello World!" > hello.txt
git add hello.txt
git commit -m "Testing ACR Tasks"
git push origin main
```

Se le pedirá que proporcione sus credenciales de GitHub cuando ejecute el comando `git push`. Proporcione el nombre de usuario de GitHub y escriba el token de acceso personal (PAT) que creó anteriormente para la contraseña.

```azurecli
Username for 'https://github.com': <github-username>
Password for 'https://githubuser@github.com': <personal-access-token>
```

Una vez que ha insertado una confirmación en el repositorio, el webhook que creó ACR Tasks activa e inicia una compilación en Azure Container Registry. Muestre los registros de la tarea que se está ejecutando actualmente para comprobar y supervisar el progreso de la compilación:

```azurecli
az acr task logs --registry $ACR_NAME
```

La salida es parecida a la siguiente, que muestra la tarea que se está ejecutando actualmente (o la última que se ejecutó):

```output
Showing logs of the last created run.
Run ID: ca7

[...]

Run ID: ca7 was successful after 38s
```

## <a name="list-builds"></a>Lista de compilaciones

Para ver una lista de las tareas que ACR Tasks ha completado para el registro, ejecute el comando [az acr task list-runs][az-acr-task-list-runs]:

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

La salida del comando debe ser parecida a la siguiente. Se muestran las ejecuciones que ha ejecutado ACR Tasks y aparece "Confirmación de Git" en la columna DESENCADENADOR de la tarea más reciente:

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------------  ----------  ---------  ---------  --------------------  ----------
ca7       taskhelloworld  linux       Succeeded  Commit     2020-11-19T22:54:34Z  00:00:29
ca6       taskhelloworld  linux       Succeeded  Manual     2020-11-19T22:51:47Z  00:00:24
ca5                       linux       Succeeded  Manual     2020-11-19T22:23:42Z  00:00:23
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a usar una tarea que desencadena automáticamente compilaciones de imágenes de contenedor en Azure cuando confirma código fuente en un repositorio Git. Pase al siguiente tutorial para aprender a crear tareas que desencadenan compilaciones cuando se actualiza la imagen base de la imagen de contenedor.

> [!div class="nextstepaction"]
> [Automatización de compilaciones después de una actualización de imagen base](container-registry-tutorial-base-image-update.md)

<!-- LINKS - External -->
[sample-repo]: https://github.com/Azure-Samples/acr-build-helloworld-node

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-login]: /cli/azure/reference-index#az_login
