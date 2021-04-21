---
title: 'Visualización de registros de ejecución de tareas: Tasks'
description: Cómo ver y administrar registros de ejecución generados por ACR Tasks.
ms.topic: article
ms.date: 03/09/2020
ms.openlocfilehash: ce5f33853be2aa48bcfd1916c7f8b94b9702f38c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781062"
---
# <a name="view-and-manage-task-run-logs"></a>Visualización de registros de ejecución de tareas

La ejecución de cada tarea en [Azure Container Registry Tasks](container-registry-tasks-overview.md) genera un resultado de registro que se puede examinar para determinar si los pasos de dicha tarea se han ejecutado correctamente. 

En este artículo se explica cómo ver y administrar registros de ejecución de tareas.

## <a name="view-streamed-logs"></a>Visualización de registros transmitidos

Cuando se desencadena una tarea manualmente, la salida del registro se transmite directamente a la consola. Por ejemplo, al desencadenar una tarea manualmente mediante el comando [az acr build](/cli/azure/acr#az_acr_build), [az acr run](/cli/azure/acr#az_acr_run) o [az acr task run](/cli/azure/acr/task#az_acr_task_run), se muestra la salida del registro transmitido a la consola. 

El siguiente comando [az acr run](/cli/azure/acr#az_acr_run) de ejemplo desencadena manualmente una tarea que ejecuta un contenedor extraído del mismo registro:

```azurecli
az acr run --registry mycontainerregistry1220 \
  --cmd '$Registry/samples/hello-world:v1' /dev/null
```

Registro transmitido:

```console
Queued a run with ID: cf4
Waiting for an agent...
2020/03/09 20:30:10 Alias support enabled for version >= 1.1.0, please see https://aka.ms/acr/tasks/task-aliases for more information.
2020/03/09 20:30:10 Creating Docker network: acb_default_network, driver: 'bridge'
2020/03/09 20:30:10 Successfully set up Docker network: acb_default_network
2020/03/09 20:30:10 Setting up Docker configuration...
2020/03/09 20:30:11 Successfully set up Docker configuration
2020/03/09 20:30:11 Logging in to registry: mycontainerregistry1220azurecr.io
2020/03/09 20:30:12 Successfully logged into mycontainerregistry1220azurecr.io
2020/03/09 20:30:12 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2020/03/09 20:30:12 Launching container with name: acb_step_0
Unable to find image 'mycontainerregistry1220azurecr.io/samples/hello-world:v1' locally
v1: Pulling from samples/hello-world
Digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e888a
Status: Downloaded newer image for mycontainerregistry1220azurecr.io/samples/hello-world:v1

Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]

2020/03/09 20:30:13 Successfully executed container: acb_step_0
2020/03/09 20:30:13 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 1.180081)

Run ID: cf4 was successful after 5s
```

## <a name="view-stored-logs"></a>Visualización de registros almacenados 

Azure Container Registry almacena registros de ejecución de todas las tareas. Puede ver los registros de ejecución almacenados en Azure Portal. También puede usar el comando [az acr task logs](/cli/azure/acr/task#az_acr_task_logs) para ver un registro seleccionado. De forma predeterminada, los registros se conservan 30 días.

Si una tarea se desencadena automáticamente, por ejemplo, mediante una actualización de código fuente, acceder a los registros almacenados es el *único* modo de ver los registros de ejecución. Los desencadenadores de tareas automáticas incluyen confirmaciones de código fuente o solicitudes de incorporación de cambios, actualizaciones de imagen base y desencadenadores de temporizador.

Para ver registros de ejecución en el portal:

1. Vaya al registro de contenedor.
1. En **Servicios**, seleccione **Tareas** > **Ejecuciones**.
1. Seleccione un **Identificador de ejecución** para ver el estado de la ejecución y los registros de ejecución. El registro contiene la misma información que un registro transmitido, si se genera uno.

![Portal de inicio de sesión para visualización de ejecución de tareas](./media/container-registry-tasks-logs/portal-task-run-logs.png)

Para ver un registro mediante la CLI de Azure, ejecute [az acr task logs](/cli/azure/acr/task#az_acr_task_logs) y especifique un identificador de ejecución, un nombre de tarea o una imagen específica creada por una tarea de compilación. Si se especifica un nombre de tarea, el comando muestra el registro de la última ejecución creada.

En el ejemplo siguiente se genera el registro de la ejecución con el identificador *cf4*:

```azurecli
az acr task logs --registry mycontainerregistry1220 \
  --run-id cf4
```

## <a name="alternative-log-storage"></a>Almacenamiento de registros alternativo

Es posible que desee almacenar los registros de ejecución de tareas en un sistema de archivos local o utilizar una solución de archivado alternativa, como Azure Storage.

Por ejemplo, cree un directorio local *tasklogs* y redirija el resultado de [az acr task logs](/cli/azure/acr/task#az_acr_task_logs) a un archivo local:

```azurecli
mkdir ~/tasklogs

az acr task logs --registry mycontainerregistry1220 \
  --run-id cf4 > ~/tasklogs/cf4.log
```

También puede guardar archivos de registro locales en Azure Storage. Por ejemplo, use la [CLI de Azure](../storage/blobs/storage-quickstart-blobs-cli.md), [Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md) u otros métodos para cargar archivos en una cuenta de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de [Azure Container Registry Tasks](container-registry-tasks-overview.md).


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-pack-build]: /cli/azure/acr/pack#az_acr_pack_build
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-update]: /cli/azure/acr/task#az_acr_task_update
[az-login]: /cli/azure/reference-index#az_login
[az-login-service-principal]: /cli/azure/authenticate-azure-cli

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
