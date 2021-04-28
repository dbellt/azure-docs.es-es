---
title: 'Inicio rápido: Creación de una instancia de Azure Data Factory con la CLI de Azure'
description: En esta guía de inicio rápido se crea una instancia de Azure Data Factory, incluidos un servicio vinculado, conjuntos de datos y una canalización. Puede ejecutar la canalización para realizar una acción de copia de archivos.
author: linda33wj
ms.author: jingwang
ms.service: azure-cli
ms.topic: quickstart
ms.date: 03/24/2021
ms.custom:
- template-quickstart
- devx-track-azurecli
ms.openlocfilehash: b40407f4c4fb81bbf76bd0b552f3c9f2c827232a
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107871542"
---
# <a name="quickstart-create-an-azure-data-factory-using-azure-cli"></a>Inicio rápido: Creación de una instancia de Azure Data Factory con la CLI de Azure

En esta guía de inicio rápido se describe cómo usar la CLI de Azure para crear una instancia de Azure Data Factory. La canalización que se crea en esta factoría de datos copia los datos de una carpeta a otra en Azure Blob Storage. Para obtener información sobre cómo transformar datos mediante Azure Data Factory, consulte [Transformar datos en Azure Data Factory](transform-data.md).

Para ver una introducción al servicio Azure Data Factory, consulte [Introducción al servicio Azure Data Factory](introduction.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [azure-cli-prepare-your-environment](../../includes/azure-cli-prepare-your-environment.md)]

> [!NOTE]
> Para crear instancias de Data Factory, la cuenta de usuario que use para iniciar sesión en Azure debe ser un miembro de los roles colaborador o propietario, o ser administrador de la suscripción de Azure. Para más información, consulte [Roles de Azure](quickstart-create-data-factory-powershell.md#azure-roles).

## <a name="prepare-a-container-and-test-file"></a>Preparación de un contenedor y un archivo de prueba

En esta guía de inicio rápido se usa una cuenta de Azure Storage, que incluye un contenedor con un archivo.

1. Para crear un grupo de recursos llamado `ADFQuickStartRG`, use el comando [az group create](/cli/azure/group#az_group_create):

   ```azurecli
   az group create --name ADFQuickStartRG --location eastus
   ```

1. Para crear una cuenta de almacenamiento, use el comando [az storage account create](/cli/azure/storage/container#az_storage_container_create):

   ```azurecli
   az storage account create --resource-group ADFQuickStartRG \
       --name adfquickstartstorage --location eastus
   ```

1. Para crear un contenedor llamado `adftutorial`, use el comando [az storage container create](/cli/azure/storage/container#az_storage_container_create):

   ```azurecli
   az storage container create --resource-group ADFQuickStartRG --name adftutorial \
       --account-name adfquickstartstorage --auth-mode key
   ```

1. En el directorio local, cree un archivo llamado `emp.txt` para cargarlo. Si trabaja en Azure Cloud Shell, puede encontrar el directorio de trabajo actual mediante el comando de Bash `echo $PWD`. Puede usar comandos de Bash estándar, como `cat`, para crear un archivo:

   ```console
   cat > emp.txt
   This is text.
   ```

   Use **Ctrl + D** para guardar el nuevo archivo.

1. Para cargar el nuevo archivo en el contenedor de Azure Storage, use el comando [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload):

   ```azurecli
   az storage blob upload --account-name adfquickstartstorage --name input/emp.txt \
       --container-name adftutorial --file emp.txt --auth-mode key
   ```

   Este comando realiza la carga en una nueva carpeta llamada `input`.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

Para crear una factoría de datos de Azure, ejecute el comando [az datafactory factory create](/cli/azure/datafactory/factory#az_datafactory_factory_create):

```azurecli
az datafactory factory create --resource-group ADFQuickStartRG \
    --factory-name ADFTutorialFactory
```

> [!IMPORTANT]
> Reemplace `ADFTutorialFactory` por un nombre único global para la factoría de datos, por ejemplo, ADFTutorialFactorySP1127.

Puede ver la factoría de datos que ha creado con el comando [az datafactory factory show](/cli/azure/datafactory/factory#az_datafactory_factory_show):

```azurecli
az datafactory factory show --resource-group ADFQuickStartRG \
    --factory-name ADFTutorialFactory
```

## <a name="create-a-linked-service-and-datasets"></a>Creación de un servicio vinculado y conjuntos de datos

A continuación, cree un servicio vinculado y dos conjuntos de datos.

1. Obtenga la cadena de conexión de la cuenta de almacenamiento mediante el comando [az storage account show-connection-string](/cli/azure/datafactory/factory#az_datafactory_factory_show):

   ```azurecli
   az storage account show-connection-string --resource-group ADFQuickStartRG \
       --name adfquickstartstorage --key primary
   ```

1. En el directorio de trabajo, cree un archivo JSON con este contenido, que incluye su propia cadena de conexión del paso anterior. Asigne al archivo el nombre `AzureStorageLinkedService.json`:

   ```json
   {
       "type":"AzureStorage",
           "typeProperties":{
           "connectionString":{
           "type": "SecureString",
           "value":"DefaultEndpointsProtocol=https;AccountName=adfquickstartstorage;AccountKey=K9F4Xk/EhYrMBIR98rtgJ0HRSIDU4eWQILLh2iXo05Xnr145+syIKNczQfORkQ3QIOZAd/eSDsvED19dAwW/tw==;EndpointSuffix=core.windows.net"
           }
       }
   }
   ```

1. Cree un servicio vinculado, llamado `AzureStorageLinkedService`, mediante el comando [az datafactory linked-service create](/cli/azure/datafactory/linked-service#az_datafactory_linked_service_create):

   ```azurecli
   az datafactory linked-service create --resource-group ADFQuickStartRG \
       --factory-name ADFTutorialFactory --linked-service-name AzureStorageLinkedService \
       --properties @AzureStorageLinkedService.json
   ```

1. En el directorio de trabajo, cree un archivo JSON con este contenido llamado `InputDataset.json`:

   ```json
   {
       "type": 
           "AzureBlob",
           "linkedServiceName": {
               "type":"LinkedServiceReference",
               "referenceName":"AzureStorageLinkedService"
               },
           "annotations": [],
           "type": "Binary",
           "typeProperties": {
               "location": {
                   "type": "AzureBlobStorageLocation",
                   "fileName": "emp.txt",
                   "folderPath": "input",
                   "container": "adftutorial"
           }
       }
   }
   ```

1. Cree un conjunto de datos de entrada llamado `InputDataset` mediante el comando [az datafactory dataset create](/cli/azure/datafactory/dataset#az_datafactory_dataset_create):

   ```azurecli
   az datafactory dataset create --resource-group ADFQuickStartRG \
       --dataset-name InputDataset --factory-name ADFQuickStartFactory \
       --properties @InputDataset.json
   ```

1. En el directorio de trabajo, cree un archivo JSON con este contenido llamado `OutputDataset.json`:

   ```json
   {
       "type": 
           "AzureBlob",
           "linkedServiceName": {
               "type":"LinkedServiceReference",
               "referenceName":"AzureStorageLinkedService"
               },
           "annotations": [],
           "type": "Binary",
           "typeProperties": {
               "location": {
                   "type": "AzureBlobStorageLocation",
                   "fileName": "emp.txt",
                   "folderPath": "output",
                   "container": "adftutorial"
           }
       }
   }
   ```

1. Cree un conjunto de datos de salida llamado `OutputDataset` mediante el comando [az datafactory dataset create](/cli/azure/datafactory/dataset#az_datafactory_dataset_create):

   ```azurecli
   az datafactory dataset create --resource-group ADFQuickStartRG \
       --dataset-name OutputDataset --factory-name ADFQuickStartFactory \
       --properties @OutputDataset.json
   ```

## <a name="create-and-run-the-pipeline"></a>Creación y ejecución de la canalización

Por último, cree y ejecute la canalización.

1. En el directorio de trabajo, cree un archivo JSON con este contenido llamado `Adfv2QuickStartPipeline.json`:

   ```json
   {
       "name": "Adfv2QuickStartPipeline",
       "properties": {
           "activities": [
               {
                   "name": "CopyFromBlobToBlob",
                   "type": "Copy",
                   "dependsOn": [],
                   "policy": {
                       "timeout": "7.00:00:00",
                       "retry": 0,
                       "retryIntervalInSeconds": 30,
                       "secureOutput": false,
                       "secureInput": false
                   },
                   "userProperties": [],
                   "typeProperties": {
                          "source": {
                           "type": "BinarySource",
                           "storeSettings": {
                               "type": "AzureBlobStorageReadSettings",
                               "recursive": true
                           }
                       },
                       "sink": {
                           "type": "BinarySink",
                           "storeSettings": {
                               "type": "AzureBlobStorageWriteSettings"
                           }
                       },
                       "enableStaging": false
                   },
                   "inputs": [
                       {
                           "referenceName": "InputDataset",
                           "type": "DatasetReference"
                       }
                   ],
                   "outputs": [
                       {
                           "referenceName": "OutputDataset",
                           "type": "DatasetReference"
                       }
                   ]
               }
           ],
           "annotations": []
       }
   }
   ```

1. Cree una canalización llamada `Adfv2QuickStartPipeline` mediante el comando [az datafactory pipeline create](/cli/azure/datafactory/pipeline#az_datafactory_pipeline_create):

   ```azurecli
   az datafactory pipeline create --resource-group ADFQuickStartRG \
       --factory-name ADFTutorialFactory --name Adfv2QuickStartPipeline \
       --pipeline @Adfv2QuickStartPipeline.json
   ```

1. Ejecute la canalización mediante el comando [az datafactory pipeline create-run](/cli/azure/datafactory/pipeline#az_datafactory_pipeline_create_run):

   ```azurecli
   az datafactory pipeline create-run --resource-group ADFQuickStartRG \
       --name Adfv2QuickStartPipeline --factory-name ADFTutorialFactory
   ```

   Este comando devuelve un identificador de ejecución. Cópielo para usarlo en el comando siguiente.

1. Compruebe que la ejecución de la canalización se ejecutó correctamente mediante el comando [az datafactory pipeline-run show](/cli/azure/datafactory/pipeline-run#az_datafactory_pipeline_run_show):

   ```azurecli
   az datafactory pipeline-run show --resource-group ADFQuickStartRG \
       --factory-name ADFTutorialFactory --run-id 00000000-0000-0000-0000-000000000000
   ```

También puede comprobar que la canalización se ejecutó como se esperaba mediante [Azure Portal](https://portal.azure.com/). Para más información, consulte [Revisión de los recursos implementados](quickstart-create-data-factory-powershell.md#review-deployed-resources).

## <a name="clean-up-resources"></a>Limpieza de recursos

Todos los recursos de esta guía de inicio rápido forman parte del mismo grupo de recursos. Puede usar el comando [az group delete](/cli/azure/group#az_group_delete) para eliminarlos todos.

```azurecli
az group delete --name ADFQuickStartRG
```

Si usa este grupo de recursos para cualquier otra cosa, elimine los recursos individuales. Por ejemplo, para quitar el servicio vinculado, use el comando [az datafactory linked-service delete](/cli/azure/datafactory/linked-service#az_datafactory_linked_service_delete).

En esta guía de inicio rápido ha creado los siguientes archivos JSON:

- AzureStorageLinkedService.json
- InputDataset.json
- OutputDataset.json
- Adfv2QuickStartPipeline.json

Elimínelos mediante los comandos estándar de Bash.

## <a name="next-steps"></a>Pasos siguientes

- [Canalizaciones y actividades en Azure Data Factory](concepts-pipelines-activities.md)
- [Servicios vinculados en Azure Data Factory](concepts-linked-services.md)
- [Conjuntos de datos en Azure Data Factory](concepts-datasets-linked-services.md)
- [Transformar datos en Azure Data Factory](transform-data.md)
