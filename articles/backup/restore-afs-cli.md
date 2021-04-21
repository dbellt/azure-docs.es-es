---
title: Restauración de recursos compartidos de archivos de Azure con la CLI de Azure
description: Aprenda a usar la CLI de Azure para restaurar recursos compartidos de archivos de Azure con copia de seguridad en almacenes de Recovery Services
ms.topic: conceptual
ms.date: 01/16/2020
ms.openlocfilehash: 2edc2281c29023bb8dfe0268f23eacfe081d413e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768527"
---
# <a name="restore-azure-file-shares-with-the-azure-cli"></a>Restauración de recursos compartidos de archivos de Azure con la CLI de Azure

La CLI de Azure es la forma de usar la línea de comandos para administrar los recursos de Azure. Es una herramienta excelente para personalizar la automatización del uso de los recursos de Azure. En este artículo se explica cómo restaurar un recurso compartido de archivos completo o archivos específicos desde un punto de restauración creado por [Azure Backup](./backup-overview.md) mediante la CLI de Azure. Estos pasos también se pueden llevar a cabo con [Azure PowerShell](./backup-azure-afs-automation.md) o en [Azure Portal](backup-afs.md).

Al acabar este tutorial, habrá aprendido cómo realizar las siguientes operaciones con la CLI de Azure:

* Visualización de los puntos de restauración de un recurso compartido de archivos de Azure con copia de seguridad.
* Restauración de un recurso compartido de archivos de Azure completo.
* Restauración de archivos o carpetas individuales.

>[!NOTE]
> Azure Backup admite ahora la restauración de varios archivos o carpetas en la ubicación original o alternativa mediante la CLI de Azure. Consulte la sección [Restauración de varios archivos o carpetas en una ubicación original o alternativa](#restore-multiple-files-or-folders-to-original-or-alternate-location) de este documento para obtener más información.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se presupone que ya tiene un recurso compartido de archivos de Azure con una copia de seguridad hecha con Azure Backup. Si no la tiene, consulte [Copia de seguridad de recursos compartidos de archivos de Azure con la CLI](backup-afs-cli.md) para configurar la copia de seguridad para los recursos compartidos de archivos. En este artículo, se usarán los siguientes recursos:

| Recurso compartido de archivos | Cuenta de almacenamiento | Region | Detalles |
|---|---|---|---|
| *azurefiles* | *afsaccount* | EastUS | Copia de seguridad de origen hecha con Azure Backup |
| *azurefiles1* | *afaccount1* | EastUS | Origen de destino para la recuperación de ubicación alternativa |

Puede usar una estructura similar para los recursos compartidos de archivos para probar los distintos tipos de restauraciones que se explican en este artículo.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

 - Este tutorial requiere la versión 2.0.18 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="fetch-recovery-points-for-the-azure-file-share"></a>Captura de puntos de recuperación para el recurso compartido de archivos de Azure

Use el cmdlet [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_list) para enumerar todos los puntos de recuperación del recurso compartido de archivos del que se ha realizado una copia de seguridad.

En el ejemplo siguiente se captura la lista de puntos de recuperación del recurso compartido de archivos *azurefiles* en la cuenta de almacenamiento *afsaccount*.

```azurecli-interactive
az backup recoverypoint list --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --backup-management-type azurestorage --item-name "AzureFileShare;azurefiles" --workload-type azurefileshare --out table
```

También puede ejecutar el cmdlet anterior con el nombre descriptivo del contenedor y el elemento proporcionando los dos parámetros adicionales siguientes:

* **--backup-management-type**: *azurestorage*
* **--workload-type**: *azurefileshare*

```azurecli-interactive
az backup recoverypoint list --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --backup-management-type azurestorage --item-name azurefiles --workload-type azurefileshare --out table
```

El conjunto de resultados es una lista de puntos de recuperación con detalles de tiempo y coherencia de cada punto de restauración.

```output
Name                Time                        Consistency
------------------  -------------------------   --------------------
932887541532871865  2020-01-05T07:08:23+00:00   FileSystemConsistent
932885927361238054  2020-01-05T07:08:10+00:00   FileSystemConsistent
932879614553967772  2020-01-04T21:33:04+00:00   FileSystemConsistent
```

El atributo **Name** de la salida corresponde al nombre del punto de recuperación que se puede utilizar como valor para el parámetro **--rp-name** en las operaciones de recuperación.

## <a name="full-share-recovery-by-using-the-azure-cli"></a>Recuperación de recursos compartidos completos mediante la CLI de Azure

Puede usar esta opción para restaurar el recurso compartido de archivos completo en la ubicación original o en otra alternativa.

Defina los parámetros siguientes para realizar operaciones de restauración:

* **--container-name**: nombre de la cuenta de almacenamiento que hospeda el recurso compartido de archivos original del que se ha realizado una copia de seguridad. Para recuperar el nombre o nombre descriptivo del contenedor, use el comando [az backup container list](/cli/azure/backup/container#az_backup_container_list).
* **--item-name**: nombre del recurso compartido de archivos original del que se ha realizado una copia de seguridad que se desea usar para la operación de restauración. Para recuperar el nombre o nombre descriptivo del elemento de copia de seguridad, use el comando [az backup item list](/cli/azure/backup/item#az_backup_item_list).

### <a name="restore-a-full-share-to-the-original-location"></a>Restauración de un recurso compartido completo en la ubicación original

Al realizar la restauración en una ubicación original, no es necesario especificar parámetros de destino. Solo debe proporcionarse un valor para **Resolve Conflict** (Resolver conflicto).

En el ejemplo siguiente se usa el cmdlet [az backup restore restore-azurefileshare](/cli/azure/backup/restore#az_backup_restore_restore_azurefileshare) con el modo de restauración establecido en *originallocation* para restaurar el recurso compartido de archivos *azurefiles* en la ubicación original. Use el punto de recuperación 932883129628959823 obtenido con [Captura de puntos de recuperación para el recurso compartido de archivos de Azure](#fetch-recovery-points-for-the-azure-file-share):

```azurecli-interactive
az backup restore restore-azurefileshare --vault-name azurefilesvault --resource-group azurefiles --rp-name 932887541532871865   --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation --resolve-conflict overwrite --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
6a27cc23-9283-4310-9c27-dcfb81b7b4bb  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de restauración. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

### <a name="restore-a-full-share-to-an-alternate-location"></a>Restauración del recurso compartido completo en una ubicación alternativa

Use esta opción para restaurar el recurso compartido de archivos en una ubicación alternativa y mantener el original tal cual. Especifique los parámetros siguientes para la recuperación en una ubicación alternativa:

* **--target-storage-account**: La cuenta de almacenamiento en la que se restaura el contenido de la copia de seguridad. La cuenta de almacenamiento de destino debe estar en la misma ubicación que el almacén.
* **--target-file-share**: recurso compartido de archivos de la cuenta de almacenamiento de destino en la que se restaura el contenido de la copia de seguridad.
* **--target-folder**: La carpeta del recurso compartido de archivos en la que se restauran los datos. Si el contenido de la copia de seguridad debe restaurarse en una carpeta raíz, asigne los valores de la carpeta de destino como una cadena vacía.
* **--resolve-conflict**: Instrucción en caso de conflicto con los datos restaurados. Acepta **Overwrite** o **Skip**.

En el ejemplo siguiente se usa [az backup restore restore-azurefileshare](/cli/azure/backup/restore#az_backup_restore_restore_azurefileshare) con el modo de restauración en *alternatelocation* para restaurar el recurso compartido de archivos *azurefiles* de la cuenta de almacenamiento *afsaccount* en el recurso compartido de archivos *azurefiles1"* de la cuenta de almacenamiento *afaccount1*.

```azurecli-interactive
az backup restore restore-azurefileshare --vault-name azurefilesvault --resource-group azurefiles --rp-name 932883129628959823 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode alternatelocation --target-storage-account afaccount1 --target-file-share azurefiles1 --target-folder restoredata --resolve-conflict overwrite --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
babeb61c-d73d-4b91-9830-b8bfa83c349a  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de restauración. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="item-level-recovery"></a>Recuperación a nivel de elemento

Puede usar esta opción para restaurar archivos o carpetas en la ubicación original o en otra alternativa.

Defina los parámetros siguientes para realizar operaciones de restauración:

* **--container-name**: nombre de la cuenta de almacenamiento que hospeda el recurso compartido de archivos original del que se ha realizado una copia de seguridad. Para recuperar el nombre o nombre descriptivo del contenedor, use el comando [az backup container list](/cli/azure/backup/container#az_backup_container_list).
* **--item-name**: nombre del recurso compartido de archivos original del que se ha realizado una copia de seguridad que se desea usar para la operación de restauración. Para recuperar el nombre o nombre descriptivo del elemento de copia de seguridad, use el comando [az backup item list](/cli/azure/backup/item#az_backup_item_list).

Especifique los parámetros siguientes para los elementos que desea recuperar:

* **SourceFilePath**: La ruta de acceso absoluta del archivo que se va a restaurar en el recurso compartido de archivos, como una cadena. Esta ruta de acceso es la misma que se usa en los comandos [az storage file download](/cli/azure/storage/file#az_storage_file_download) o [az storage file show](/cli/azure/storage/file#az_storage_file_show) de la CLI.
* **SourceFileType**: elija si se ha seleccionado un directorio o un archivo. Admite **Directory** o **File**.
* **ResolveConflict**: Instrucción en caso de conflicto con los datos restaurados. Acepta **Overwrite** o **Skip**.

### <a name="restore-individual-files-or-folders-to-the-original-location"></a>Restauración de archivos o carpetas individuales en la ubicación original

Use el cmdlet [az backup restore restore-azurefiles](/cli/azure/backup/restore#az_backup_restore_restore_azurefiles) con el modo de restauración establecido en *originallocation* para restaurar archivos o carpetas en su ubicación original.

En el ejemplo siguiente se restaura el archivo *RestoreTest.txt* en la ubicación original: el recurso compartido de archivos *azurefiles*.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932881556234035474 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation  --source-file-type file --source-file-path "Restore/RestoreTest.txt" --resolve-conflict overwrite  --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
df4d9024-0dcb-4edc-bf8c-0a3d18a25319  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de restauración. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

### <a name="restore-individual-files-or-folders-to-an-alternate-location"></a>Restauración de archivos o carpetas individuales en una ubicación alternativa

Para restaurar archivos o carpetas específicos en una ubicación alternativa, use el cmdlet [az backup restore restore-azurefiles](/cli/azure/backup/restore#az_backup_restore_restore_azurefiles) con el modo de restauración establecido en *alternatelocation* y especifique los parámetros de destino siguientes:

* **--target-storage-account**: La cuenta de almacenamiento en la que se restaura el contenido de la copia de seguridad. La cuenta de almacenamiento de destino debe estar en la misma ubicación que el almacén.
* **--target-file-share**: recurso compartido de archivos de la cuenta de almacenamiento de destino en la que se restaura el contenido de la copia de seguridad.
* **--target-folder**: La carpeta del recurso compartido de archivos en la que se restauran los datos. Si el contenido de la copia de seguridad debe restaurarse en una carpeta raíz, asigne el valor de la carpeta de destino como cadena vacía.

En el ejemplo siguiente se restaura el archivo *RestoreTest.txt* presente al principio en el recurso compartido de archivos *azurefiles* en una ubicación alternativa: la carpeta *restoredata* del recurso compartido de archivos *azurefiles1* hospedado en la cuenta de almacenamiento *afaccount1*.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932881556234035474 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode alternatelocation --target-storage-account afaccount1 --target-file-share azurefiles1 --target-folder restoredata --resolve-conflict overwrite --source-file-type file --source-file-path "Restore/RestoreTest.txt" --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
df4d9024-0dcb-4edc-bf8c-0a3d18a25319  azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de restauración. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="restore-multiple-files-or-folders-to-original-or-alternate-location"></a>Restauración de varios archivos o carpetas en una ubicación original o alternativa

Para realizar la restauración de varios elementos, pase el valor para el parámetro **source-file-path** como rutas **separadas por espacios** de todos los archivos o carpetas que quiera restaurar.

En el ejemplo siguiente se restauran los archivos *Restore.txt* y *AFS testing Report.docx* a su ubicación original.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932889937058317910 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation  --source-file-type file --source-file-path "Restore Test.txt" "AFS Testing Report.docx" --resolve-conflict overwrite  --out table
```

El resultado será similar al siguiente:

```output
Name                                          ResourceGroup
------------------------------------          ---------------
649b0c14-4a94-4945-995a-19e2aace0305          azurefiles
```

El atributo **Name** de la salida se corresponde con el nombre del trabajo creado por el servicio de copia de seguridad para la operación de restauración. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job#az_backup_job_show).

Si quiere restaurar varios archivos en una ubicación alternativa, use el comando anterior y especifique parámetros relacionados con el destino, tal y como se describe en la sección [Restauración de archivos o carpetas individuales en una ubicación alternativa](#restore-individual-files-or-folders-to-an-alternate-location).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [Administración de copias de seguridad de recursos compartidos de archivos de Azure con la CLI de Azure](manage-afs-backup-cli.md).
