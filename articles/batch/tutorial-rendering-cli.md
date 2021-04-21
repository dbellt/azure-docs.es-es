---
title: 'Tutorial: Representación de una escena en la nube'
description: Aprenda a representar una escena de Autodesk 3DS Max con Arnold mediante el servicio de representación de Batch y la interfaz de la línea de comandos de Azure
ms.topic: tutorial
ms.date: 12/30/2020
ms.custom: mvc, devx-track-azurecli
ROBOTS: NOINDEX
ms.openlocfilehash: dc55ca15c74bd71365e514424757df3d37cec0f6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768186"
---
# <a name="tutorial-render-a-scene-with-azure-batch"></a>Tutorial: Representación de una escena con Azure Batch

Azure Batch proporciona funcionalidades de representación para la nube según una modalidad de pago por uso. Azure Batch admite aplicaciones de representación entre las que se incluyen Autodesk Maya, 3ds Max, Arnold y V-Ray. Este tutorial muestra los pasos necesarios para representar una escena pequeña con Batch mediante la interfaz de la línea de comandos de Azure. Aprenderá a:

> [!div class="checklist"]
> - Cargar una escena en Azure Storage
> - Crear un grupo de Batch para la representación
> - Representar una escena de un fotograma
> - Escalar el grupo y representar una escena de varios fotogramas
> - Descargar la salida representada

En este tutorial se representa una escena 3DS Max con Batch mediante el representador de seguimiento de rayo [Arnold](https://www.autodesk.com/products/arnold/overview). El grupo de Batch usa una imagen de Azure Marketplace con gráficos preinstalados y las aplicaciones de representación que proporcionan las licencias de pago por uso.

## <a name="prerequisites"></a>Prerrequisitos

- Necesita una suscripción de pago por uso u otra opción de compra de Azure para usar aplicaciones de representación en Batch sobre una base de pago por uso. **Las licencias de pago por uso no se admiten si usa una oferta gratis de Azure que proporciona un crédito monetario.**

- La escena de 3DS Max de este tutorial se encuentra en [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/tree/master/batch/render-scene), junto con un script de ejemplo de Bash y archivos de configuración JSON. La escena 3DS Max procede de los [archivos de ejemplo de Autodesk 3DS Max](https://download.autodesk.com/us/support/files/3dsmax_sample_files/2017/Autodesk_3ds_Max_2017_English_Win_Samples_Files.exe). (Los archivos de ejemplo de Autodesk 3DS Max están disponibles con licencias tipo Creative Commons Attribution-NonCommercial-Share. Copyright &copy; Autodesk, Inc.)

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Este tutorial requiere la versión 2.0.20 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

> [!TIP]
> Puede ver [plantillas de trabajo de Arnold](https://github.com/Azure/batch-extension-templates/tree/master/templates/arnold/render-windows-frames) en el repositorio de GitHub de plantillas de extensión de Azure Batch.

## <a name="create-a-batch-account"></a>Crear una cuenta de Batch

Si no lo ha hecho ya, cree un grupo de recursos, una cuenta de Batch y una de Storage vinculada en la suscripción.

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create). En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus2*.

```azurecli-interactive
az group create \
    --name myResourceGroup \
    --location eastus2
```

Cree una cuenta de Azure Storage en el grupo de recursos con el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create). Para este tutorial se utiliza la cuenta de Storage para almacenar una escena 3 DS Max de entrada y la salida representada.

```azurecli-interactive
az storage account create \
    --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```

Cree una cuenta de Batch con el comando [az batch account create](/cli/azure/batch/account#az_batch_account_create). En el ejemplo siguiente se crea una cuenta de Batch llamada *mybatchaccount* en *myResourceGroup* y se vincula a la cuenta de almacenamiento que ha creado.  

```azurecli-interactive
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus2
```

Para crear y administrar grupos de proceso y trabajos, es preciso autenticarse en Batch. Inicie sesión en la cuenta con el comando [az batch account login](/cli/azure/batch/account#az_batch_account_login). Tras iniciar sesión, los comandos `az batch` usan este contexto de la cuenta. En el ejemplo siguiente se utiliza la autenticación de clave compartida, en función del nombre y la clave de la cuenta de Batch. Batch también admite la autenticación a través de [Azure Active Directory](batch-aad-auth.md) para usuarios individuales o una aplicaciones desatendidas.

```azurecli-interactive
az batch account login \
    --name mybatchaccount \
    --resource-group myResourceGroup \
    --shared-key-auth
```

## <a name="upload-a-scene-to-storage"></a>Carga de una escena en Storage

Para cargar la escena de entrada escena en Storage, acceda primero a la cuenta de Storage y cree un contenedor de destino para los blobs. Para acceder a la cuenta de Azure Storage, exporte las variables de entorno `AZURE_STORAGE_KEY` y `AZURE_STORAGE_ACCOUNT`. El primer comando de shell de Bash usa el comando [az storage account keys list](/cli/azure/storage/account/keys#az_storage_account_keys_list) para obtener la primera clave de cuenta. Después de establecer estas variables de entorno, los comandos de almacenamiento usan este contexto de la cuenta.

```azurecli-interactive
export AZURE_STORAGE_KEY=$(az storage account keys list --account-name mystorageaccount --resource-group myResourceGroup -o tsv --query [0].value)

export AZURE_STORAGE_ACCOUNT=mystorageaccount
```

Ahora crearemos un contenedor de blobs en la cuenta de Storage para los archivos de la escena. En el ejemplo siguiente se usa el comando [az storage container create](/cli/azure/storage/container#az_storage_container_create) para crear un contenedor de blobs denominado *scenefiles* que permita el acceso de lectura público.

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name scenefiles
```

Descargue la escena `MotionBlur-Dragon-Flying.max` de [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max) en un directorio de trabajo local. Por ejemplo:

```azurecli-interactive
wget -O MotionBlur-DragonFlying.max https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max
```

Cargue el archivo de la escena del directorio de trabajo local en el contenedor de blobs. En el ejemplo siguiente se usa el comando [az storage blob upload-batch](/cli/azure/storage/blob#az_storage_blob_upload_batch), que puede cargar varios archivos:

```azurecli-interactive
az storage blob upload-batch \
    --destination scenefiles \
    --source ./
```

## <a name="create-a-rendering-pool"></a>Creación de un grupo de representación

Cree un grupo de Batch para la representación mediante el comando [az batch pool create](/cli/azure/batch/pool#az_batch_pool_create). En este ejemplo se especifica la configuración del grupo en un archivo JSON. En el shell actual, cree un archivo denominado *mypool.json*, y copie y pegue el contenido siguiente. Asegúrese de copiar todo el texto correctamente. (Puede descargar el archivo de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/mypool.json)).


```json
{
  "id": "myrenderpool",
  "vmSize": "standard_d2_v2",
  "virtualMachineConfiguration": {
    "imageReference": {
      "publisher": "batch",
      "offer": "rendering-windows2016",
      "sku": "rendering",
      "version": "1.3.8"
    },
    "nodeAgentSKUId": "batch.node.windows amd64"
  },
  "targetDedicatedNodes": 0,
  "targetLowPriorityNodes": 1,
  "enableAutoScale": false,
  "applicationLicenses":[
         "3dsmax",
         "arnold"
      ],
  "enableInterNodeCommunication": false 
}
```

Batch admite nodos especializados y [nodos de prioridad baja](batch-low-pri-vms.md), y en los grupos puede utilizar ambos. Los nodos dedicados están reservados para el grupo. Los nodos de prioridad baja se ofrecen a precio reducido por la capacidad sobrante de las máquinas virtuales de Azure. Los nodos de prioridad baja dejan de estar disponibles si Azure no tiene capacidad suficiente.

El grupo especificado contiene un único nodo de prioridad baja que se ejecuta en una imagen de Windows Server con el software para el servicio de representación de Batch. Este grupo tiene licencia de representación con 3DS Max y Arnold. En un paso posterior, se escala el grupo a un mayor número de nodos.

Si aún no ha iniciado sesión en su cuenta de Batch, utilice el comando [az batch account login](/cli/azure/batch/account#az_batch_account_login) para hacerlo. Después, cree el grupo; para ello, pase el archivo JSON al comando `az batch pool create`:

```azurecli-interactive
az batch pool create \
    --json-file mypool.json
```

El grupo tardará unos minutos en aprovisionarse. Para ver el estado del grupo, ejecute el comando [az batch pool show](/cli/azure/batch/pool#az_batch_pool_show). El siguiente comando obtiene el estado de asignación del grupo:

```azurecli-interactive
az batch pool show \
    --pool-id myrenderpool \
    --query "allocationState"
```

Realice los pasos siguientes para crear un trabajo y tareas mientras cambia el estado del grupo. El grupo está completamente aprovisionado cuando el estado de asignación es `steady` y todos los nodos se encuentran en ejecución.  

## <a name="create-a-blob-container-for-output"></a>Creación de un contenedor de blobs para la salida

En los ejemplos de este tutorial, todas las tareas del trabajo de representación crean un archivo de salida. Antes de programar el trabajo, cree un contenedor de blobs en la cuenta de Storage como destino de los archivos de salida. En el ejemplo siguiente se usa el comando [az storage container create](/cli/azure/storage/container#az_storage_container_create) para crear el contenedor *job-myrenderjob* con acceso de lectura público.

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name job-myrenderjob
```

Para escribir archivos de salida en el contenedor, Batch debe usar un token de firma de acceso compartido (SAS). Cree el token con el comando [az storage account enerate-sas](/cli/azure/storage/account#az_storage_account_generate_sas). En este ejemplo se crea un token para escribir en cualquier contenedor de blobs de la cuenta que expira el 15 de noviembre de 2021:

```azurecli-interactive
az storage account generate-sas \
    --permissions w \
    --resource-types co \
    --services b \
    --expiry 2021-11-15
```

Tome nota del token que devuelve el comando, que tiene un aspecto similar al siguiente. Se usará en un paso posterior.

`se=2021-11-15&sp=rw&sv=2019-09-24&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

## <a name="render-a-single-frame-scene"></a>Representar una escena de un fotograma

### <a name="create-a-job"></a>Creación de un trabajo

Cree un trabajo de representación con el comando [az batch job create](/cli/azure/batch/job#az_batch_job_create) para ejecutarlo en el grupo. Inicialmente, el trabajo no tiene tareas.

```azurecli-interactive
az batch job create \
    --id myrenderjob \
    --pool-id myrenderpool
```

### <a name="create-a-task"></a>Crea una tarea.

Use el comando [az batch task create](/cli/azure/batch/task#az_batch_task_create) para crear una tarea de representación en el trabajo. En este ejemplo se especifica la configuración de la tarea en un archivo JSON. En el shell actual, cree un archivo denominado *myrendertask.json*, y copie y pegue el contenido siguiente. Asegúrese de copiar todo el texto correctamente. (Puede descargar el archivo de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask.json)).

La tarea especifica un comando de 3DS Max para representar un fotograma individual de la escena *MotionBlur-DragonFlying.max*.

Modifique los elementos `blobSource` y `containerURL` del archivo JSON para que incluyan el nombre de la cuenta de Storage y el token de SAS. 

> [!TIP]
> `containerURL` termina por el token de SAS y es similar a: `https://mystorageaccount.blob.core.windows.net/job-myrenderjob/$TaskOutput?se=2018-11-15&sp=rw&sv=2017-04-17&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`.

```json
{
  "id": "myrendertask",
  "commandLine": "cmd /c \"%3DSMAX_2018%3dsmaxcmdio.exe -secure off -v:5 -rfw:0 -start:1 -end:1 -outputName:\"dragon.jpg\" -w 400 -h 300 MotionBlur-DragonFlying.max\"",
  "resourceFiles": [
    {
        "httpUrl": "https://mystorageaccount.blob.core.windows.net/scenefiles/MotionBlur-DragonFlying.max",
        "filePath": "MotionBlur-DragonFlying.max"
    }
  ],
    "outputFiles": [
        {
            "filePattern": "dragon*.jpg",
            "destination": {
                "container": {
                    "containerUrl": "https://mystorageaccount.blob.core.windows.net/job-myrenderjob/myrendertask/$TaskOutput?Add_Your_SAS_Token_Here"
                }
            },
            "uploadOptions": {
                "uploadCondition": "TaskSuccess"
            }
        }
    ],
  "userIdentity": {
    "autoUser": {
      "scope": "task",
      "elevationLevel": "nonAdmin"
    }
  }
}
```

Agregue la tarea al trabajo con el siguiente comando:

```azurecli-interactive
az batch task create \
    --job-id myrenderjob \
    --json-file myrendertask.json
```

Batch programa la tarea y esta se ejecuta en cuanto haya un nodo del grupo disponible.

### <a name="view-task-output"></a>Visualización de la salida de la tarea

La tarea tarda unos minutos en ejecutarse. Use el comando [az batch task show](/cli/azure/batch/task#az_batch_task_show) para ver los detalles de la tarea.

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id myrendertask
```

La tarea genera el archivo *dragon0001.jpg* en el nodo de proceso y lo carga en el contenedor *job-myrenderjob* de la cuenta de Storage. Para ver la salida, descargue el archivo de Storage en el equipo local mediante el comando [az storage blob download](/cli/azure/storage/blob#az_storage_blob_download).

```azurecli-interactive
az storage blob download \
    --container-name job-myrenderjob \
    --file dragon.jpg \
    --name dragon0001.jpg

```

Abra *dragon.jpg* en el equipo. La imagen representada será similar a la siguiente:

![Fotograma de dragón representado 1](./media/tutorial-rendering-cli/dragon-frame.png) 

## <a name="scale-the-pool"></a>Escalado del grupo

Ahora, modifique el grupo para prepararnos para un trabajo de representación mayor, con varios fotogramas. Batch ofrece distintas maneras de escalar los recursos de proceso, como el [escalado automático](batch-automatic-scaling.md), que agrega o quita nodos al cambiar las necesidades de la tarea. En este ejemplo básico, utilice el comando [az batch pool resize](/cli/azure/batch/pool#az_batch_pool_resize) para aumentar el número de nodos de baja prioridad del grupo a *6*:

```azurecli-interactive
az batch pool resize --pool-id myrenderpool --target-dedicated-nodes 0 --target-low-priority-nodes 6
```

El tamaño del grupo tardará unos minutos en cambiar. Mientras se realiza el proceso, configure las tareas siguientes para que se ejecuten en el trabajo de representación existente.

## <a name="render-a-multiframe-scene"></a>Representación de una escena de varios fotogramas

Como en el ejemplo de un solo fotograma, utilice el comando [az batch task create](/cli/azure/batch/task#az_batch_task_create) para crear tareas de representación en el trabajo denominado *myrenderjob*. En este caso, especifique la configuración de la tarea en un archivo JSON denominado *myrendertask_multi.json*. (Puede descargar el archivo de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask_multi.json)). Cada una de las seis tareas especifica una línea de comandos Arnold que representa un fotograma de la escena 3DS Max *MotionBlur-DragonFlying.max*.

Cree un archivo en el shell actual con el nombre *myrendertask_multi.json*, y copie y pegue el contenido del archivo descargado. Modifique los elementos `blobSource` y `containerURL` del archivo JSON de manera que incluyan el nombre de la cuenta de Storage y el token de SAS. Asegúrese de cambiar la configuración de las seis tareas. Guarde el archivo y ejecute el siguiente comando para poner las tareas en cola:

```azurecli-interactive
az batch task create --job-id myrenderjob --json-file myrendertask_multi.json
```

### <a name="view-task-output"></a>Visualización de la salida de la tarea

La tarea tarda unos minutos en ejecutarse. Use el comando [az batch task list](/cli/azure/batch/task#az_batch_task_list) para ver el estado de las tareas. Por ejemplo:

```azurecli-interactive
az batch task list \
    --job-id myrenderjob \
    --output table
```

Use el comando [az batch task show](/cli/azure/batch/task#az_batch_task_show) para ver los detalles de una tarea. Por ejemplo:

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id mymultitask1
```

Las tareas generan archivos de salida denominados *dragon0002.jpg* - *dragon0007.jpg* en los nodos de proceso y los cargan en el contenedor *job-myrenderjob* de la cuenta de Storage. Para ver la salida, descargue los archivos en una carpeta del equipo local mediante el comando [az storage blob download-batch](/cli/azure/storage/blob). Por ejemplo:

```azurecli-interactive
az storage blob download-batch \
    --source job-myrenderjob \
    --destination .
```

Abra uno de los archivos en el equipo. El fotograma representado 6 será similar al siguiente:

![Fotograma de dragón representado 6](./media/tutorial-rendering-cli/dragon-frame6.png) 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no necesite el grupo de recursos, la cuenta de Batch, los grupos y todos los recursos relacionados, puede usar el comando [az group delete](/cli/azure/group#az_group_delete) para quitarlos. Los recursos se eliminan como se indica a continuación:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> - Cargar escenas en Azure Storage
> - Crear un grupo de Batch para la representación
> - Representar una escena de un fotograma con Arnold
> - Escalar el grupo y representar una escena de varios fotogramas
> - Descargar la salida representada

Para más información sobre la representación a escala de nube, consulte la documentación de representación de Batch.

> [!div class="nextstepaction"]
> [Servicio de representación de Batch](batch-rendering-service.md)
