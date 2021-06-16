---
title: 'Comprobación del estado de cifrado de un blob: Azure Storage'
description: Aprenda a usar Azure Portal, PowerShell o la CLI de Azure para comprobar si un determinado blob está cifrado. Si un blob no está cifrado, aprenda a usar AzCopy para forzar el cifrado descargando y volviendo a cargar el blob.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/26/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f9392b0645b86ea055d67a7d11f9e62f1bf08743
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677272"
---
# <a name="check-the-encryption-status-of-a-blob"></a>Comprobación del estado de cifrado de un blob

Cada blob en bloques, blob en anexos o blob en páginas que se escribió en Azure Storage después del 20 de octubre de 2017 está cifrado con Azure Storage. Los blobs creados antes de esta fecha se siguen cifrando mediante un proceso en segundo plano.

En este artículo se muestra cómo determinar si un blob determinado se ha cifrado.

## <a name="check-a-blobs-encryption-status"></a>Comprobación del estado de cifrado de un blob

Use Azure Portal, PowerShell o la CLI de Azure para determinar si un determinado blob está cifrado.

### <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para usar Azure Portal para comprobar si un blob se ha cifrado, siga estos pasos:

1. En Azure Portal, vaya a la cuenta de almacenamiento.
1. Seleccione **Contenedores** para ir a una lista de los contenedores de la cuenta.
1. Busque el blob y abra su pestaña de **información general**.
1. Vea la propiedad **Servidor cifrado**. Si es **True**, tal y como se muestra en la siguiente imagen, el blob está cifrado. Tenga en cuenta que las propiedades del blob también incluyen la fecha y la hora en que se creó el blob.

    ![Captura de pantalla que muestra cómo comprobar la propiedad Servidor cifrado en Azure Portal](media/storage-blob-encryption-status/blob-encryption-property-portal.png)

### <a name="powershell"></a>[PowerShell](#tab/powershell)

Para usar PowerShell con el fin de comprobar si se ha cifrado un blob, consulte la propiedad **IsServerEncrypted** del mismo. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account>
$blob = Get-AzStorageBlob -Context $account.Context `
    -Container <container> `
    -Blob <blob>
$blob.ICloudBlob.Properties.IsServerEncrypted
```

Para determinar cuándo se creó el blob, compruebe el valor de la propiedad **Fecha de creación**:

```powershell
$blob.ICloudBlob.Properties.IsServerEncrypted
```

### <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para usar la CLI de Azure con el fin de comprobar si se ha cifrado un blob, consulte la propiedad **IsServerEncrypted** del mismo. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```azurecli-interactive
az storage blob show \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob> \
    --query "properties.serverEncrypted"
```

Para determinar cuándo se creó el blob, compruebe el valor de la propiedad **Fecha de creación**.

---

## <a name="force-encryption-of-a-blob"></a>Forzar el cifrado de un blob

Si un blob que se creó antes del 20 de octubre de 2017 aún no se ha cifrado con el proceso en segundo plano, puede forzar el cifrado para que se produzca inmediatamente si descarga y vuelve a cargar el blob. Una manera sencilla de hacerlo es con AzCopy.

Para descargar un blob en el sistema de archivos local con AzCopy, use la siguiente sintaxis:

```
azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'

Example:
azcopy copy 'https://storagesamples.blob.core.windows.net/sample-container/blob1.txt' 'C:\temp\blob1.txt'
```

Para volver a cargar el blob en Azure Storage con AzCopy, use la siguiente sintaxis:

```
azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'

Example:
azcopy copy 'C:\temp\blob1.txt' 'https://storagesamples.blob.core.windows.net/sample-container/blob1.txt'
```

Para más información sobre el uso de AzCopy para copiar datos de blobs, consulte [Transferencia de datos con AzCopy y Blob Storage](../common/storage-use-azcopy-v10.md#transfer-data).

## <a name="next-steps"></a>Pasos siguientes

[Cifrado de Azure Storage para datos en reposo](../common/storage-service-encryption.md)
