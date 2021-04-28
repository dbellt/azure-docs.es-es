---
title: Habilitación de la eliminación temporal para blobs
titleSuffix: Azure Storage
description: Habilite la eliminación temporal de blobs para evitar que los datos de los blobs se eliminen o sobrescriban accidentalmente.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/27/2021
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: a1698a56ad7e92a59b664ce8f8bca2355fb44fb1
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108074716"
---
# <a name="enable-soft-delete-for-blobs"></a>Habilitación de la eliminación temporal para blobs

La eliminación temporal de blobs protege a cada uno de los blobs y sus versiones, instantáneas y metadatos de errores accidentales al borrar o sobrescribir los datos, ya que conserva en el sistema los datos eliminados durante el período de tiempo que se especifique. Durante este período de retención, los blobs pueden restaurarse a su estado original. Una vez vencido el período de retención especificado, el blob se elimina permanentemente. Para más información sobre la eliminación temporal, consulte este artículo sobre la [eliminación temporal de blobs](soft-delete-blob-overview.md).

La eliminación temporal de blobs forma parte de una exhaustiva estrategia para proteger los datos de los blobs. Si necesita más información sobre las recomendaciones de Microsoft para proteger los datos, consulte esta [introducción sobre la protección de datos](data-protection-overview.md).

## <a name="enable-blob-soft-delete"></a>Habilitación de la eliminación temporal de blobs

De forma predeterminada, en las nuevas cuentas de almacenamiento, la eliminación temporal de blobs está deshabilitada. Sin embargo, puede habilitarla o deshabilitarla en cualquier momento mediante Azure Portal, PowerShell o la CLI de Azure.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para habilitar la eliminación temporal de blobs en la cuenta de almacenamiento mediante Azure Portal, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), vaya a la cuenta de almacenamiento.
1. Busque la opción **Protección de datos** en **Blob service**.
1. En la sección **Recuperación**, seleccione **Turn on soft delete for blobs** (Activar eliminación temporal de blobs).
1. Especifique un período de retención de entre 1 y 365 días. Microsoft recomienda un período de retención mínimo de siete días.
1. Guarde los cambios.

:::image type="content" source="media/soft-delete-blob-enable/blob-soft-delete-configuration-portal.png" alt-text="Instantánea en la que se muestra cómo habilitar la eliminación temporal en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para habilitar la eliminación temporal de blobs con PowerShell, llame al comando [Enable-AzStorageBlobDeleteRetentionPolicy](/powershell/module/az.storage/enable-azstorageblobdeleteretentionpolicy) especificando el período de retención en días.

En el ejemplo siguiente, se habilita la eliminación temporal de blobs y se establece un período de retención de siete días. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurepowershell
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account> `
    -RetentionDays 7
```

Para comprobar la configuración actual de la eliminación temporal de blobs, llame al comando [Get-AzStorageBlobServiceProperty](/powershell/module/az.storage/get-azstorageblobserviceproperty):

```azurepowershell
$properties = Get-AzStorageBlobServiceProperty -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
$properties.DeleteRetentionPolicy.Enabled
$properties.DeleteRetentionPolicy.Days
```

# <a name="cli"></a>[CLI](#tab/azure-CLI)

Para habilitar la eliminación temporal de blobs con la CLI de Azure, llame al comando [az storage account blob-service-properties update](/cli/azure/storage/account/blob-service-properties#az_storage_account_blob_service_properties_update) y especifique el período de retención en días.

En el ejemplo siguiente, se habilita la eliminación temporal de blobs y se establece un período de retención de siete días. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli-interactive
az storage account blob-service-properties update --account-name <storage-account> \
    --resource-group <resource-group> \
    --enable-delete-retention true \
    --delete-retention-days 7
```

Para comprobar la configuración actual de la eliminación temporal de blobs, llame al comando [az storage account blob-service-properties show](/cli/azure/storage/account/blob-service-properties#az_storage_account_blob_service_properties_show):

```azurecli-interactive
az storage account blob-service-properties show --account-name <storage-account> \
    --resource-group <resource-group>
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Eliminación temporal para blobs](soft-delete-blob-overview.md)
- [Administración y restauración de blobs eliminados temporalmente](soft-delete-blob-manage.md)
