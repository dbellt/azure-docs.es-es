---
title: Habilitación y administración de la restauración a un momento dado para blobs en bloques
titleSuffix: Azure Storage
description: Aprenda a usar la restauración a un momento dado para restaurar un conjunto de blobs en bloques a su estado anterior en un momento determinado.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 01/29/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: cd58ae6aa0083b0f2a02ef1c008e76f92f39b766
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110676307"
---
# <a name="perform-a-point-in-time-restore-on-block-blob-data"></a>Habilitación y administración de la restauración a un momento dado para blobs en bloques

Puede usar la restauración a un momento dado para restaurar uno o más conjuntos de blobs en bloques a un estado anterior. En este artículo se describe cómo habilitar la restauración a un momento dado de una cuenta de almacenamiento y cómo realizar una operación de restauración.

Para más información sobre la restauración a un momento dado, consulte [Restauración a un momento dado para blobs en bloques](point-in-time-restore-overview.md).

> [!CAUTION]
> La restauración a un momento dado solo admite la restauración de operaciones en blobs en bloques. No se pueden restaurar las operaciones en contenedores. Si elimina un contenedor de la cuenta de almacenamiento llamando a la operación [Eliminar contenedor](/rest/api/storageservices/delete-container), dicho contenedor no se puede restaurar con una operación de restauración. En lugar de eliminar un contenedor completo, elimine blobs individuales, por si desea restaurarlos más adelante. Además, Microsoft recomienda habilitar la eliminación temporal de contenedores y blobs para proteger contra la eliminación accidental. Para más información, consulte [Eliminación temporal de contenedores (versión preliminar)](soft-delete-container-overview.md) y [Eliminación temporal de blobs](soft-delete-blob-overview.md).

## <a name="enable-and-configure-point-in-time-restore"></a>Habilitar y configurar la restauración a un momento dado

Antes de habilitar y configurar la restauración a un momento dado, habilite los requisitos previos para la cuenta de almacenamiento: eliminación temporal, fuente de cambios y control de versiones de blob. Para obtener más información sobre cómo habilitar cada una de estas características, consulte estos artículos:

- [Habilitación de la eliminación temporal para blobs](./soft-delete-blob-enable.md)
- [Habilitar y deshabilitar la fuente de cambios](storage-blob-change-feed.md#enable-and-disable-the-change-feed)
- [Habilitar y administrar las versiones de blob](versioning-enable.md)

> [!IMPORTANT]
> La habilitación de la eliminación temporal, la fuente de cambios y el control de versiones de blobs pueden generar cargos adicionales. Para más información, consulte [Eliminación temporal para blobs](soft-delete-blob-overview.md), [Compatibilidad con la fuente de cambios en Azure Blob Storage](storage-blob-change-feed.md) y [Control de versiones de blobs](versioning-overview.md).

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para configurar la restauración a un momento dado con Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento en Azure Portal.
1. En **Configuración**, elija **Protección de datos**.
1. Seleccione **Activar la restauración a un momento dado**. Al seleccionar esta opción, también se habilitan la eliminación temporal para blobs, el control de versiones y la fuente de cambios.
1. Establezca el punto de restauración máximo para la restauración a un momento dado, en días. Este número debe ser al menos un día menos que el período de retención especificado para la eliminación temporal de blobs.
1. Guarde los cambios.

En la imagen siguiente se muestra una cuenta de almacenamiento configurada para la restauración a un momento dado con un punto de restauración de siete días atrás y un período de retención para la eliminación temporal de blobs de 14 días.

:::image type="content" source="media/point-in-time-restore-manage/configure-point-in-time-restore-portal.png" alt-text="Captura de pantalla que muestra cómo configurar la restauración a un momento dado en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para configurar la restauración a un momento dado con PowerShell, primero instale la versión 2.6.0 o posterior del módulo [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage). Luego, llame al comando [Enable-AzStorageBlobRestorePolicy](/powershell/module/az.storage/enable-azstorageblobrestorepolicy) para habilitar la restauración a un momento dado de la cuenta de almacenamiento.

En el ejemplo siguiente se habilita la eliminación temporal y se establece el período de retención de eliminación temporal, se habilita la fuente de cambios y el control de versiones y, luego, se habilita la restauración a un momento dado. Al ejecutar el ejemplo, no olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```powershell
# Set resource group and account variables.
$rgName = "<resource-group>"
$accountName = "<storage-account>"

# Enable blob soft delete with a retention of 14 days.
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -RetentionDays 14

# Enable change feed and versioning.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EnableChangeFeed $true `
    -IsVersioningEnabled $true

# Enable point-in-time restore with a retention period of 7 days.
# The retention period for point-in-time restore must be at least
# one day less than that set for soft delete.
Enable-AzStorageBlobRestorePolicy -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -RestoreDays 7

# View the service settings.
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgName `
    -StorageAccountName $accountName
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para configurar la restauración a un momento dado con la CLI de Azure, instale primero la versión 2.2.0 o posterior de la CLI de Azure. Después, llame al comando [az storage account blob-service-properties update](/cli/azure/storage/account/blob-service-properties#az_storage_account_blob_service_properties_update) para habilitar la restauración a un momento dado y la otra configuración de protección de datos necesaria para la cuenta de almacenamiento.

En el ejemplo siguiente se habilita la eliminación temporal y se establece el período de retención de la eliminación temporal en 14 días, se habilita la fuente de cambios y el control de versiones y se habilita la restauración a un momento dado con un período de restauración de 7 días. Al ejecutar el ejemplo, no olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```azurecli
az storage account blob-service-properties update \
    --resource-group <resource_group> \
    --account-name <storage-account> \
    --enable-delete-retention true \
    --delete-retention-days 14 \
    --enable-versioning true \
    --enable-change-feed true \
    --enable-restore-policy true \
    --restore-days 7
```

---

## <a name="choose-a-restore-point"></a>Selección de un punto de restauración

El punto de restauración es la fecha y la hora en que se restauran los datos. Azure Storage siempre usa un valor de fecha y hora UTC como punto de restauración. Sin embargo, Azure Portal permite especificar el punto de restauración en la hora local y, a continuación, convierte ese valor de fecha y hora en un valor de fecha y hora UTC para realizar la operación de restauración.

Cuando realice una operación de restauración con PowerShell o la CLI de Azure, debe especificar el punto de restauración como un valor de fecha y hora UTC. Si el punto de restauración se especifica con un valor de hora local en lugar de un valor de hora UTC, es posible que la operación de restauración se siga comportando según lo esperado en algunos casos. Por ejemplo, si la hora local es UTC menos cinco horas, al especificar un valor de hora local se produce un punto de restauración que es cinco horas anterior al valor proporcionado. Si no se realizaron cambios en los datos del intervalo que se va a restaurar durante ese período de cinco horas, la operación de restauración producirá los mismos resultados independientemente del valor de tiempo proporcionado. Se recomienda especificar una hora UTC para el punto de restauración con el fin de evitar resultados inesperados.

## <a name="perform-a-restore-operation"></a>Realizar una operación de restauración

Puede restaurar todos los contenedores de la cuenta de almacenamiento, o bien puede restaurar un intervalo de blobs en uno o varios contenedores. Un rango de blobs se define de manera lexicográfica, lo que significa que está en orden alfabético. Se admiten hasta diez rangos lexicográficos por operación de restauración. El inicio del rango es inclusivo y el final, exclusivo.

El patrón de contenedor especificado para el rango de inicio y el rango de finalización debe incluir un mínimo de tres caracteres. La barra diagonal (/) que se usa para separar un nombre de contenedor de un nombre de blob no cuenta para este mínimo.

No se admiten caracteres comodín en un rango lexicográfico. Los caracteres comodín se tratan como caracteres estándar.

Puede restaurar los blobs en los contenedores `$root` y `$web` si los especifica de forma explícita en un rango que se pasa a una operación de restauración. Los contenedores `$root` y `$web` se restauran solo si se especifican explícitamente. No se pueden restaurar otros contenedores del sistema.

Solo se restauran los blobs en bloques. Los blobs en páginas y los blobs en anexión no se incluyen en la operación de restauración. Para más información sobre las limitaciones relacionadas con los blobs en anexión, consulte [Restauración a un momento dado para blobs en bloques](point-in-time-restore-overview.md).

> [!IMPORTANT]
> Cuando se realiza una operación de restauración, Azure Storage bloquea las operaciones de datos en los blobs del intervalo que se está restaurando mientras dure la operación. Las operaciones de lectura, escritura y eliminación se bloquean en la ubicación principal. Por este motivo, es posible que las operaciones como la enumeración de contenedores en Azure Portal no funcionen según lo esperado mientras se realiza la operación de restauración.
>
> Asimismo, las operaciones de lectura de la ubicación secundaria pueden continuar durante la operación de restauración si la cuenta de almacenamiento tiene replicación geográfica.
>
> El tiempo que se tarda en restaurar un conjunto de datos se basa en el número de operaciones de escritura y eliminación realizadas durante el período de restauración. Por ejemplo, una cuenta con un millón de objetos con 3000 objetos agregados al día y 1000 objetos eliminados al día requerirá aproximadamente dos horas para restaurar a un período de 30 días determinado del pasado. No se recomendarían un período de retención ni una restauración superiores a 90 días en el pasado para una cuenta con esta tasa de cambio.

### <a name="restore-all-containers-in-the-account"></a>Restaurar todos los contenedores de la cuenta

Puede restaurar todos los contenedores de la cuenta de almacenamiento para devolverlos a su estado anterior en un momento dado.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para restaurar todos los contenedores y blobs de la cuenta de almacenamiento con Azure Portal, siga estos pasos:

1. Vaya a la lista de contenedores de la cuenta de almacenamiento.
1. En la barra de herramientas, elija **Restaurar contenedores** y, luego, **Restaurar todo**.
1. En el panel **Restauración de todos los contenedores**, proporcione una fecha y hora para especificar el punto de restauración.
1. Active la casilla para confirmar que quiere continuar.
1. Seleccione **Restaurar** para iniciar la operación de restauración.

    :::image type="content" source="media/point-in-time-restore-manage/restore-all-containers-portal.png" alt-text="Captura de pantalla que muestra cómo restaurar todos los contenedores a un punto de restauración especificado":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para restaurar todos los contenedores y blobs de la cuenta de almacenamiento con PowerShell, llame al comando **Restore-AzStorageBlobRange** y proporcione el punto de restauración como un valor de fecha y hora UTC. De manera predeterminada, el comando **Restore-AzStorageBlobRange** se ejecuta asincrónicamente y devuelve un objeto de tipo **PSBlobRestoreStatus** que se puede usar para comprobar el estado de la operación de restauración.

En el ejemplo siguiente los contenedores de la cuenta de almacenamiento se restauran de manera asincrónica a su estado 12 horas antes del momento actual y se comprueban algunas de las propiedades de la operación de restauración:

```powershell
# Specify -TimeToRestore as a UTC value
$restoreOperation = Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -TimeToRestore (Get-Date).ToUniversalTime().AddHours(-12)

# Get the status of the restore operation.
$restoreOperation.Status
# Get the ID for the restore operation.
$restoreOperation.RestoreId
# Get the restore point in UTC time.
$restoreOperation.Parameters.TimeToRestore
```

Para ejecutar sincrónicamente la operación de restauración, incluya el parámetro **-WaitForComplete** en el comando. Cuando el parámetro **-WaitForComplete** está presente, PowerShell muestra un mensaje que incluye el id. de restauración de la operación y, luego, bloquea la ejecución hasta que se completa la operación de restauración. Tenga en cuenta que el período de tiempo que requiere una operación de restauración depende de la cantidad de datos que se van a restaurar y una operación de restauración de gran tamaño puede tardar hasta una hora en completarse.

```powershell
Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -TimeToRestore (Get-Date).AddHours(-12) -WaitForComplete
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para restaurar todos los contenedores y blobs de la cuenta de almacenamiento con la CLI de Azure, llame al comando [az storage blob restore](/cli/azure/storage/blob#az_storage_blob_restore) y proporcione el punto de restauración como un valor de fecha y hora UTC.

En el siguiente ejemplo se restauran asincrónicamente todos los contenedores de la cuenta de almacenamiento a su estado 12 horas antes de una fecha y hora especificadas. Para comprobar el estado de la operación de restauración, llame a [az storage account show](/cli/azure/storage/account#az_storage_account_show):

```azurecli
az storage blob restore \
    --resource-group <resource_group> \
    --account-name <storage-account> \
    --time-to-restore 2021-01-14T06:31:22Z \
    --no-wait
```

Para comprobar las propiedades de una operación de restauración, llame a [az storage account show](/cli/azure/storage/account#az_storage_account_show) y expanda la propiedad **blobRestoreStatus**. En el siguiente ejemplo se muestra cómo se comprueba la propiedad **status**.

```azurecli
az storage account show \
    --name <storage-account> \
    --resource-group <resource_group> \ 
    --expand blobRestoreStatus \
    --query blobRestoreStatus.status \
    --output tsv
```

Para ejecutar el comando **az storage blob restore** sincrónicamente y bloquear en la ejecución hasta que se complete la operación de restauración, omita el parámetro `--no-wait`.

---

### <a name="restore-ranges-of-block-blobs"></a>Restauración de rangos de blobs en bloques

Puede restaurar uno o varios rangos lexicográficos de blobs dentro de un solo contenedor o en varios contenedores para devolver esos blobs a su estado anterior en un momento dado.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para restaurar un rango de blobs en uno o varios contenedores con Azure Portal, siga estos pasos:

1. Vaya a la lista de contenedores de la cuenta de almacenamiento.
1. Seleccione el contenedor o los contenedores que quiere restaurar.
1. En la barra de herramientas, elija **Restaurar contenedores** y, luego, **Restaurar selección**.
1. En el panel **Restauración de los contenedores seleccionados**, proporcione una fecha y hora para especificar el punto de restauración.
1. Especifique los rangos que quiere restaurar. Use una barra diagonal (/) para delimitar el nombre del contenedor del prefijo de blob.
1. De manera predeterminada, en el panel **Restauración de los contenedores seleccionados** se especifica un rango que incluye todos los blobs del contenedor. Elimine este rango si no quiere restaurar todo el contenedor. El rango predeterminado se muestra en la imagen siguiente.

    :::image type="content" source="media/point-in-time-restore-manage/delete-default-blob-range.png" alt-text="Captura de pantalla que muestra el rango predeterminado de blobs que se va a eliminar antes de especificar el rango personalizado":::

1. Active la casilla para confirmar que quiere continuar.
1. Seleccione **Restaurar** para iniciar la operación de restauración.

En la imagen siguiente se muestra una operación de restauración en un conjunto de rangos.

:::image type="content" source="media/point-in-time-restore-manage/restore-multiple-container-ranges-portal.png" alt-text="Captura de pantalla que muestra cómo restaurar rangos de blobs en uno o varios contenedores":::

La operación de restauración que se muestra en la imagen realiza las acciones siguientes:

- Restaura todo el contenido de *container1*.
- Restaura los blobs del rango lexicográfico *blob1* a *blob5* en *container2*. Este rango restaura blobs con nombres como *blob1*, *blob11*, *blob100*, *blob2*, etc. Como el final del rango es exclusivo, restaura los blobs cuyos nombres empiezan con *blob4*, pero no los blobs cuyos nombres empiezan con *blob5*.
- Restaura todos los blobs en *container3* y *container4*. Como el final del rango es exclusivo, este rango no restaura *container5*.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para restaurar un solo rango de blobs, llame al comando **Restore-AzStorageBlobRange** y especifique un rango lexicográfico de nombres de contenedor y blob para el parámetro `-BlobRestoreRange`. Por ejemplo, para restaurar los blobs en un único contenedor denominado *container1*, puede especificar un rango que empiece con *container1* y que termine con *container2*. No es necesario que existan los contenedores denominados en los rangos de inicio y finalización. Como el final del rango es exclusivo, incluso si la cuenta de almacenamiento incluye un contenedor denominado *container2*, solo se restaurará el contenedor denominado *container1*:

```powershell
$range = New-AzStorageBlobRangeToRestore -StartRange container1 `
    -EndRange container2
```

Para especificar un subconjunto de blobs en un contenedor para restaurar, use una barra diagonal (/) para separar el nombre del contenedor del patrón de prefijo de blob. Por ejemplo, el intervalo siguiente selecciona los blobs en un único contenedor cuyos nombres comienzan por las letras *d* hasta la *f*:

```powershell
$range = New-AzStorageBlobRangeToRestore -StartRange container1/d `
    -EndRange container1/g
```

A continuación, proporcione el rango al comando **Restore-AzStorageBlobRange**. Especifique el punto de restauración proporcionando un valor UTC **DateTime** para el parámetro `-TimeToRestore`. En el siguiente ejemplo se restauran los blobs del rango especificado a su estado 3 días antes del momento actual:

```powershell
# Specify -TimeToRestore as a UTC value
Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -BlobRestoreRange $range `
    -TimeToRestore (Get-Date).AddDays(-3)
```

De manera predeterminada, el comando **Restore-AzStorageBlobRange** se ejecuta asincrónicamente. Al iniciar una operación de restauración de manera asincrónica, PowerShell muestra de inmediato una tabla de propiedades para la operación:  

```powershell
Status     RestoreId                            FailureReason Parameters.TimeToRestore     Parameters.BlobRanges
------     ---------                            ------------- ------------------------     ---------------------
InProgress 459c2305-d14a-4394-b02c-48300b368c63               2020-09-15T23:23:07.1490859Z ["container1/d" -> "container1/g"]
```

Para restaurar varios rangos de blobs en bloques, especifique una matriz de rangos para el parámetro `-BlobRestoreRange`. En el ejemplo siguiente se especifican dos rangos para restaurar todo el contenido de *container1* y *container4* a su estado de 24 horas atrás y guarda el resultado en una variable:

```powershell
# Specify a range that includes the complete contents of container1.
$range1 = New-AzStorageBlobRangeToRestore -StartRange container1 `
    -EndRange container2
# Specify a range that includes the complete contents of container4.
$range2 = New-AzStorageBlobRangeToRestore -StartRange container4 `
    -EndRange container5

$restoreOperation = Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -TimeToRestore (Get-Date).AddHours(-24) `
    -BlobRestoreRange @($range1, $range2)

# Get the status of the restore operation.
$restoreOperation.Status
# Get the ID for the restore operation.
$restoreOperation.RestoreId
# Get the blob ranges specified for the operation.
$restoreOperation.Parameters.BlobRanges
```

Para ejecutar la operación de restauración de manera sincrónica y bloquear la ejecución hasta que se complete, incluya el parámetro **-WaitForComplete** en el comando.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para restaurar un rango de blobs, llame al comando [az storage blob restore](/cli/azure/storage/blob#az_storage_blob_restore) y especifique un rango lexicográfico de nombres de contenedor y blob para el parámetro `--blob-range`. Para especificar varios intervalos, proporcione el parámetro `--blob-range` para cada intervalo distinto.

Por ejemplo, para restaurar los blobs en un único contenedor denominado *container1*, puede especificar un rango que empiece con *container1* y que termine con *container2*. No es necesario que existan los contenedores denominados en los rangos de inicio y finalización. Como el final del rango es exclusivo, incluso si la cuenta de almacenamiento incluye un contenedor denominado *container2*, solo se restaurará el contenedor denominado *container1*.

Para especificar un subconjunto de blobs en un contenedor para restaurar, use una barra diagonal (/) para separar el nombre del contenedor del patrón de prefijo de blob. En el ejemplo siguiente se restaura asincrónicamente un intervalo de blobs en un contenedor cuyos nombres comienzan por las letras desde `d` hasta `f`.

```azurecli
az storage blob restore \
    --account-name <storage-account> \
    --time-to-restore 2021-01-14T06:31:22Z \
    --blob-range container1 container2
    --blob-range container3/d container3/g
    --no-wait
```

Para ejecutar el comando **az storage blob restore** sincrónicamente y bloquear en la ejecución hasta que se complete la operación de restauración, omita el parámetro `--no-wait`.

---

## <a name="next-steps"></a>Pasos siguientes

- [Restauración a un momento dado para blobs en bloques](point-in-time-restore-overview.md)
- [Eliminación temporal](./soft-delete-blob-overview.md)
- [Fuente de cambios](storage-blob-change-feed.md)
- [Control de versiones de blobs](versioning-overview.md)
