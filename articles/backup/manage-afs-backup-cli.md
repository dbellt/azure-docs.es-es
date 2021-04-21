---
title: Administración de copias de seguridad de recursos compartidos de archivos de Azure con la CLI de Azure
description: Obtenga información sobre cómo usar la CLI de Azure para administrar y supervisar los recursos compartidos de archivos de Azure de los que Azure Backup ha realizado una copia de seguridad.
ms.topic: conceptual
ms.date: 01/15/2020
ms.openlocfilehash: e389f5cde12734ef4bf0be4ecfba69ba33f5e030
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773610"
---
# <a name="manage-azure-file-share-backups-with-the-azure-cli"></a>Administración de copias de seguridad de recursos compartidos de archivos de Azure con la CLI de Azure

La CLI de Azure es la forma de usar la línea de comandos para administrar los recursos de Azure. Es una herramienta excelente para personalizar la automatización del uso de los recursos de Azure. En este artículo se explica cómo realizar tareas para administrar y supervisar los recursos compartidos de archivos de Azure de los que se ha realizado una copia de seguridad mediante [Azure Backup](./backup-overview.md). También puede llevar a cabo estos pasos en [Azure Portal](https://portal.azure.com/).

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por supuesto que ya tiene una copia de seguridad de un recurso compartido de archivos de Azure realizada por [Azure Backup](./backup-overview.md). Si no la tiene, consulte [Copia de seguridad de recursos compartidos de archivos de Azure con la CLI](backup-afs-cli.md) para configurar la copia de seguridad para los recursos compartidos de archivos. En este artículo, se usarán los siguientes recursos:
   -  **Grupo de recursos**: *azurefiles*
   -  **Almacén de Recovery Services**: *azurefilesvault*
   -  **Cuenta de almacenamiento**: *afsaccount*
   -  **Recurso compartido de archivos**: *azurefiles*
  
  [!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
   - Este tutorial requiere la versión 2.0.18 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="monitor-jobs"></a>Supervisión de trabajos

Al desencadenar operaciones de copia de seguridad o restauración, el servicio de copia de seguridad crea un trabajo para realizar un seguimiento. Para supervisar los trabajos completados o en ejecución, use el cmdlet [az backup job list](/cli/azure/backup/job#az_backup_job_list). Con la CLI, también puede [suspender un trabajo que se esté ejecutando actualmente](/cli/azure/backup/job#az_backup_job_stop) o [esperar hasta que se complete un trabajo](/cli/azure/backup/job#az_backup_job_wait).

En el ejemplo siguiente se muestra el estado de los trabajos de copia de seguridad para el almacén de Recovery Services *azurefilesvault*:

```azurecli-interactive
az backup job list --resource-group azurefiles --vault-name azurefilesvault
```

```output
[
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/d477dfb6-b292-4f24-bb43-6b14e9d06ab5",
    "location": null,
    "name": "d477dfb6-b292-4f24-bb43-6b14e9d06ab5",
    "properties": {
      "actionsInfo": null,
      "activityId": "3cef43ed-0af4-43e2-b9cb-1322c496ccb4",
      "backupManagementType": "AzureStorage",
      "duration": "0:00:29.718011",
      "endTime": "2020-01-13T08:05:29.180606+00:00",
      "entityFriendlyName": "azurefiles",
      "errorDetails": null,
      "extendedInfo": null,
      "jobType": "AzureStorageJob",
      "operation": "Backup",
      "startTime": "2020-01-13T08:04:59.462595+00:00",
      "status": "Completed",
      "storageAccountName": "afsaccount",
      "storageAccountVersion": "MicrosoftStorage"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupJobs"
  },
  {
    "eTag": null,
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/1b9399bf-c23c-4caa-933a-5fc2bf884519",
    "location": null,
    "name": "1b9399bf-c23c-4caa-933a-5fc2bf884519",
    "properties": {
      "actionsInfo": null,
      "activityId": "2663449c-94f1-4735-aaf9-5bb991e7e00c",
      "backupManagementType": "AzureStorage",
      "duration": "0:00:28.145216",
      "endTime": "2020-01-13T08:05:27.519826+00:00",
      "entityFriendlyName": "azurefilesresource",
      "errorDetails": null,
      "extendedInfo": null,
      "jobType": "AzureStorageJob",
      "operation": "Backup",
      "startTime": "2020-01-13T08:04:59.374610+00:00",
      "status": "Completed",
      "storageAccountName": "afsaccount",
      "storageAccountVersion": "MicrosoftStorage"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupJobs"
  }
]
```

## <a name="modify-policy"></a>Modificación de directivas

Puede modificar una directiva de copia de seguridad para cambiar la frecuencia de las copias de seguridad o la duración de retención mediante [az backup item set-policy](/cli/azure/backup/item#az_backup_item_set_policy).

Para cambiar la directiva, defina los siguientes parámetros:

* **--container-name**: nombre de la cuenta de almacenamiento que contiene el recurso compartido de archivos. Para recuperar el **nombre** o **nombre descriptivo** del contenedor, use el comando [az backup container list](/cli/azure/backup/container#az_backup_container_list).
* **--name**: nombre del recurso compartido de archivos para el que desea cambiar la directiva. Para recuperar el **nombre** o **nombre descriptivo** del elemento de copia de seguridad, use el comando [az backup item list](/cli/azure/backup/item#az_backup_item_list).
* **--policy-name**: nombre de la directiva de copia de seguridad que desea establecer para el recurso compartido de archivos. Puede usar [az backup policy list](/cli/azure/backup/policy#az_backup_policy_list) para ver todas las directivas del almacén.

En el ejemplo siguiente se establece la directiva de copia de seguridad *schedule2* para el recurso compartido de archivos *azurefiles* presente en la cuenta de almacenamiento *afsaccount*.

```azurecli-interactive
az backup item set-policy --policy-name schedule2 --name azurefiles --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --name "AzureFileShare;azurefiles" --backup-management-type azurestorage --out table
```

También puede ejecutar el comando anterior con los nombres descriptivos del contenedor y el elemento proporcionando los dos parámetros adicionales siguientes:

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup item set-policy --policy-name schedule2 --name azurefiles --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --name azurefiles --backup-management-type azurestorage --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
fec6f004-0e35-407f-9928-10a163f123e5  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de cambio de directiva. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="stop-protection-on-a-file-share"></a>Detención de la protección en un recurso compartido de archivos

Hay dos maneras de dejar de proteger recursos compartidos de archivos de Azure:

* Detener todos los trabajos futuros de copia de seguridad y *eliminar* todos los puntos de recuperación.
* Detener todos los trabajos futuros de copia de seguridad pero *dejar* los puntos de recuperación.

Puede que dejar los puntos de recuperación en el almacenamiento conlleve un costo asociado, dado que las instantáneas subyacentes creadas por Azure Backup se conservarán. La ventaja de dejarlos es que tiene la opción de restaurar el recurso compartido de archivos más adelante, si así lo desea. Para más información sobre el costo de dejar los puntos de recuperación, consulte la [información sobre precios](https://azure.microsoft.com/pricing/details/storage/files). Si opta por eliminar todos los puntos de recuperación, no podrá restaurar el recurso compartido de archivos.

Para detener la protección del recurso compartido de archivos, defina los parámetros siguientes:

* **--container-name**: nombre de la cuenta de almacenamiento que contiene el recurso compartido de archivos. Para recuperar el **nombre** o **nombre descriptivo** del contenedor, use el comando [az backup container list](/cli/azure/backup/container#az_backup_container_list).
* **--item-name**: nombre del recurso compartido de archivos para el que desea detener la protección. Para recuperar el **nombre** o **nombre descriptivo** del elemento de copia de seguridad, use el comando [az backup item list](/cli/azure/backup/item#az_backup_item_list).

### <a name="stop-protection-and-retain-recovery-points"></a>Detención de la protección y conservación de los puntos de recuperación

Para detener la protección conservando los datos, use el cmdlet [az backup protection disable](/cli/azure/backup/protection#az_backup_protection_disable).

En el ejemplo siguiente se detiene la protección del recurso compartido de archivos *azurefiles*, pero se conservan todos los puntos de recuperación.

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name “AzureFileShare;azurefiles” --out table
```

También puede ejecutar el comando anterior con el nombre descriptivo del contenedor y el elemento proporcionando los dos parámetros adicionales siguientes:

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --item-name azurefiles --workload-type azurefileshare --backup-management-type Azurestorage --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
fec6f004-0e35-407f-9928-10a163f123e5  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de detención de la protección. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

### <a name="stop-protection-without-retaining-recovery-points"></a>Detención de la protección sin conservar los puntos de recuperación

Para detener la protección sin conservar los puntos de recuperación, use el cmdlet [az backup protection disable](/cli/azure/backup/protection#az_backup_protection_disable) con la opción **delete-backup-data** establecida en **true**.

En el ejemplo siguiente se detiene la protección del recurso compartido de archivos *azurefiles* sin conservar puntos de recuperación.

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name “AzureFileShare;azurefiles” --delete-backup-data true --out table
```

También puede ejecutar el comando anterior con el nombre descriptivo del contenedor y el elemento proporcionando los dos parámetros adicionales siguientes:

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup protection disable --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --item-name azurefiles --workload-type azurefileshare --backup-management-type Azurestorage --delete-backup-data true --out table
```

## <a name="resume-protection-on-a-file-share"></a>Reanudación de la protección en un recurso compartido de archivos

Si ha detenido la protección de un recurso compartido de archivos de Azure pero ha conservado los puntos de recuperación, puede reanudar la protección más adelante. Si no conserva los puntos de recuperación, no puede reanudar la protección.

Para reanudar la protección del recurso compartido de archivos, defina los parámetros siguientes:

* **--container-name**: nombre de la cuenta de almacenamiento que contiene el recurso compartido de archivos. Para recuperar el **nombre** o **nombre descriptivo** del contenedor, use el comando [az backup container list](/cli/azure/backup/container#az_backup_container_list).
* **--item-name**: nombre del recurso compartido de archivos para el que desea reanudar la protección. Para recuperar el **nombre** o **nombre descriptivo** del elemento de copia de seguridad, use el comando [az backup item list](/cli/azure/backup/item#az_backup_item_list).
* **--policy-name**: nombre de la directiva de copia de seguridad para la que desea reanudar la protección del recurso compartido de archivos.

En el ejemplo siguiente se usa el cmdlet [az backup protection resume](/cli/azure/backup/protection#az_backup_protection_resume) para reanudar la protección del recurso compartido de archivos *azurefiles* con la directiva de copia de seguridad *schedule1*.

```azurecli-interactive
az backup protection resume --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount” --item-name “AzureFileShare;azurefiles” --policy-name schedule2 --out table
```

También puede ejecutar el comando anterior con el nombre descriptivo del contenedor y el elemento proporcionando los dos parámetros adicionales siguientes:

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup protection resume --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --item-name azurefiles --workload-type azurefileshare --backup-management-type Azurestorage --policy-name schedule2 --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
75115ab0-43b0-4065-8698-55022a234b7f  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de reanudación de la protección. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="unregister-a-storage-account"></a>Anulación del registro de una cuenta de almacenamiento

Si desea proteger los recursos compartidos de archivos en una cuenta de almacenamiento determinada mediante un almacén de Recovery Services diferente, primero debe [detener la protección de todos los recursos compartidos de archivos](#stop-protection-on-a-file-share) en esa cuenta de almacenamiento. A continuación, anule el registro de la cuenta del almacén de Recovery Services que se usa actualmente para la protección.

Debe proporcionar un nombre de contenedor para anular el registro de la cuenta de almacenamiento. Para recuperar el **nombre** o **nombre descriptivo** del contenedor, use el comando [az backup container list](/cli/azure/backup/container#az_backup_container_list).

En el ejemplo siguiente se anula el registro de la cuenta de almacenamiento *afsaccount* de *azurefilesvault* con el cmdlet [az backup container unregister](/cli/azure/backup/container#az_backup_container_unregister).

```azurecli-interactive
az backup container unregister --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --out table
```

También puede ejecutar el cmdlet anterior con el nombre descriptivo del contenedor proporcionando el parámetro adicional siguiente:

* **--backup-management-type**: *azurestorage*

```azurecli-interactive
az backup container unregister --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --backup-management-type azurestorage --out table
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, vea [Solución de problemas de las copias de seguridad de recursos compartidos de archivos de Azure](troubleshoot-azure-files.md).
