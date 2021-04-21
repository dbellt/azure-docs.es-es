---
title: 'Inicio rápido: Ejecución del primer trabajo de Batch con la CLI de Azure'
description: En este inicio rápido se muestra cómo crear una cuenta de Batch y a ejecutar un trabajo de Batch con la CLI de Azure.
ms.topic: quickstart
ms.date: 08/13/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 8d3005233320a7ba0d00f186944a0a8c0c456647
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765312"
---
# <a name="quickstart-run-your-first-batch-job-with-the-azure-cli"></a>Inicio rápido: Ejecución del primer trabajo de Batch con la CLI de Azure

Para empezar a trabajar con Azure Batch, utilice la CLI de Azure para crear una cuenta de Batch, un grupo de nodos de proceso (máquinas virtuales) y un trabajo que ejecute tareas en el grupo. Cada tarea del ejemplo ejecuta un comando básico en uno de los nodos del grupo.

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. Tras completar esta guía de inicio rápido, entenderá los conceptos clave del servicio Batch y estará listo para probar dicho servicio con cargas de trabajo más realistas y a mayor escala.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Para realizar este inicio rápido es necesaria la versión 2.0.20 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente se crea un grupo de recursos denominado *QuickstartBatch-rg* en la ubicación *eastus2*.

```azurecli-interactive
az group create \
    --name QuickstartBatch-rg \
    --location eastus2
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Puede vincular una cuenta de Azure Storage con su cuenta de Batch. Aunque no es necesaria para esta guía de inicio rápido, la cuenta de almacenamiento es útil para implementar aplicaciones y almacenar los datos de entrada y salida de la mayoría de las cargas de trabajo reales. Cree una cuenta de almacenamiento en el grupo de recursos con el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create).

```azurecli-interactive
az storage account create \
    --resource-group QuickstartBatch-rg \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```

## <a name="create-a-batch-account"></a>Crear una cuenta de Batch

Cree una cuenta de Batch con el comando [az batch account create](/cli/azure/batch/account#az_batch_account_create). Para crear los recursos de proceso (grupos de nodos de proceso) y trabajos de Batch se necesita una cuenta.

En el ejemplo siguiente se crea una cuenta de Batch llamada *mybatchaccount* en *QuickstartBatch-rg* y se vincula a la cuenta de almacenamiento que ha creado.  

```azurecli-interactive
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group QuickstartBatch-rg \
    --location eastus2
```

Para crear y administrar grupos de proceso y trabajos, es preciso autenticarse en Batch. Inicie sesión en la cuenta con el comando [az batch account login](/cli/azure/batch/account#az_batch_account_login). Tras iniciar sesión, los comandos `az batch` usan este contexto de la cuenta.

```azurecli-interactive
az batch account login \
    --name mybatchaccount \
    --resource-group QuickstartBatch-rg \
    --shared-key-auth
```

## <a name="create-a-pool-of-compute-nodes"></a>Creación de un grupo de nodos de proceso

Ahora que tiene una cuenta de Batch, cree un grupo de ejemplo de nodos de proceso de Linux mediante el comando [az batch pool create](/cli/azure/batch/pool#az_batch_pool_create). En el ejemplo siguiente se crea un grupo denominado *mypool* de dos nodos de tamaño *Standard_A1_v2* que ejecutan Ubuntu 16.04 LTS. El tamaño del nodo que se sugiere ofrece un buen equilibrio entre rendimiento y costo para este ejemplo rápido.
 
```azurecli-interactive
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image canonical:ubuntuserver:16.04-LTS \
    --node-agent-sku-id "batch.node.ubuntu 16.04"
```

Batch crea el grupo inmediatamente, pero tarda unos minutos en asignar e iniciar los nodos de proceso. Durante este tiempo, el grupo está en estado `resizing`. Para ver el estado del grupo, ejecute el comando [az batch pool show](/cli/azure/batch/pool#az_batch_pool_show). Este comando muestra todas las propiedades del grupo y puede consultar propiedades concretas. El siguiente comando obtiene el estado de asignación del grupo:

```azurecli-interactive
az batch pool show --pool-id mypool \
    --query "allocationState"
```

Realice los pasos siguientes para crear un trabajo y tareas mientras cambia el estado del grupo. El grupo está listo para ejecutar tareas cuando el estado de asignación es `steady` y todos los nodos están en ejecución.

## <a name="create-a-job"></a>Creación de un trabajo

Ahora que tiene un grupo, cree un trabajo para que se ejecute en él. Un trabajo de Batch es un grupo lógico de una o varias tareas. Un trabajo incluye valores comunes para las tareas, como la prioridad y el grupo en el que se ejecutan las tareas. Cree un trabajo de Batch mediante el comando [az batch job create](/cli/azure/batch/job#az_batch_job_create). En el ejemplo siguiente se crea un trabajo, *myjob*, en el grupo de *mypool*. Inicialmente, el trabajo no tiene tareas.

```azurecli-interactive
az batch job create \
    --id myjob \
    --pool-id mypool
```

## <a name="create-tasks"></a>Creación de tareas

Ahora use el comando [az batch task create](/cli/azure/batch/task#az_batch_task_create) para crear varias tareas que se ejecutarán en el trabajo. En este ejemplo se crean cuatro tareas idénticas. En cada una de ellas se ejecuta `command-line` para mostrar las variables de entorno de Batch en un nodo de proceso y, a continuación, se espera 90 segundos. Cuando se usa Batch, esta línea de comandos es el lugar en que se especifica la aplicación o el script. Batch proporciona varias formas de implementar aplicaciones y scripts en nodos de proceso.

El siguiente script de Bash crea cuatro tareas paralelas (*mytask1* a *mytask4*).

```azurecli-interactive
for i in {1..4}
do
   az batch task create \
    --task-id mytask$i \
    --job-id myjob \
    --command-line "/bin/bash -c 'printenv | grep AZ_BATCH; sleep 90s'"
done
```

La salida del comando muestra la configuración de cada una de las tareas. Batch distribuye las tareas a los nodos de proceso.

## <a name="view-task-status"></a>Visualización del estado de la tarea

Después de crear una tarea, Batch la pone en cola para ejecutarla en el grupo. Una vez que un nodo está disponible para ejecutarla, la tarea se ejecuta.

Use el comando [az batch task show](/cli/azure/batch/task#az_batch_task_show) para ver el estado de las tareas de Batch. En el ejemplo siguiente se muestran detalles acerca de *mytask1*, que se ejecuta en uno de los nodos de grupo.

```azurecli-interactive
az batch task show \
    --job-id myjob \
    --task-id mytask1
```

La salida del comando incluye muchos detalles, pero anote el `exitCode` de la línea de comandos de la tarea y el `nodeId`. Si el valor `exitCode` de 0, indica que la línea de comandos de la tarea se completó correctamente. `nodeId` indica el identificador del nodo de grupo en que se ejecutó la tarea.

## <a name="view-task-output"></a>Visualización de la salida de la tarea

Para enumerar los archivos que ha creado una tarea en un nodo de proceso, use el comando [az batch task file list](/cli/azure/batch/task). El comando siguiente enumera los archivos que ha creado *mytask1*:

```azurecli-interactive
az batch task file list \
    --job-id myjob \
    --task-id mytask1 \
    --output table
```

La salida es similar a la siguiente:

```
Name        URL                                                                                         Is Directory      Content Length
----------  ------------------------------------------------------------------------------------------  --------------  ----------------
stdout.txt  https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/stdout.txt  False                  695
certs       https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/certs       True
wd          https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/wd          True
stderr.txt  https://mybatchaccount.eastus2.batch.azure.com/jobs/myjob/tasks/mytask1/files/stderr.txt  False                     0

```

Para descargar uno de los archivos de salida en un directorio local, use el comando [az batch task file download](/cli/azure/batch/task). En este ejemplo, la salida de la tarea está en `stdout.txt`.

```azurecli-interactive
az batch task file download \
    --job-id myjob \
    --task-id mytask1 \
    --file-path stdout.txt \
    --destination ./stdout.txt
```

El contenido de `stdout.txt` se puede ver en un editor de texto. El contenido muestra las variables de entorno de Azure Batch que se definen en el nodo. Al crear sus propios trabajos de Batch, puede hacer referencia a estas variables de entorno tanto en las líneas de comandos de la tarea como en las aplicaciones y los scripts que ejecutan las líneas de comandos. Por ejemplo:

```
AZ_BATCH_TASK_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1
AZ_BATCH_NODE_STARTUP_DIR=/mnt/batch/tasks/startup
AZ_BATCH_CERTIFICATES_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1/certs
AZ_BATCH_ACCOUNT_URL=https://mybatchaccount.eastus2.batch.azure.com/
AZ_BATCH_TASK_WORKING_DIR=/mnt/batch/tasks/workitems/myjob/job-1/mytask1/wd
AZ_BATCH_NODE_SHARED_DIR=/mnt/batch/tasks/shared
AZ_BATCH_TASK_USER=_azbatch
AZ_BATCH_NODE_ROOT_DIR=/mnt/batch/tasks
AZ_BATCH_JOB_ID=myjobl
AZ_BATCH_NODE_IS_DEDICATED=true
AZ_BATCH_NODE_ID=tvm-257509324_2-20180703t215033z
AZ_BATCH_POOL_ID=mypool
AZ_BATCH_TASK_ID=mytask1
AZ_BATCH_ACCOUNT_NAME=mybatchaccount
AZ_BATCH_TASK_USER_IDENTITY=PoolNonAdmin
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea continuar con los ejemplos y tutoriales de Batch, utilice la cuenta de Batch y la cuenta de almacenamiento vinculada que se ha creado en esta guía de inicio rápido. Por la propia cuenta de Batch no se cobra nada.

Se cobran los grupos mientras que los nodos estén en ejecución, aunque no haya trabajos programados. Cuando deje de necesitar alguno de los grupos, elimínelo con el comando [az batch pool delete](/cli/azure/batch/pool#az_batch_pool_delete). Al eliminar el grupo, las salidas de tarea de los nodos también se eliminan.

```azurecli-interactive
az batch pool delete --pool-id mypool
```

Cuando no necesite el grupo de recursos, la cuenta de Batch, los grupos y todos los recursos relacionados, puede usar el comando [az group delete](/cli/azure/group#az_group_delete) para quitarlos. Los recursos se eliminan como se indica a continuación:

```azurecli-interactive
az group delete --name QuickstartBatch-rg
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una cuenta de Batch, un grupo de Batch y un trabajo de Batch. El trabajo ha ejecutado tareas de ejemplo y se ha visto la salida que se ha creado en uno de los nodos. Ahora que conoce los conceptos clave del servicio Batch, ya esta listo para probar dicho servicio con cargas de trabajo más realistas y a mayor escala. Para más información acerca de Azure Batch, continúe con los tutoriales de Azure Batch.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Batch](./tutorial-parallel-dotnet.md)
