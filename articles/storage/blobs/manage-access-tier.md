---
title: Administración del nivel de acceso de un blob en una cuenta de Azure Storage
description: Aprenda a cambiar el nivel de acceso de un blob en una cuenta de GPv2 o de Blob Storage
author: tamram
ms.author: tamram
ms.date: 01/11/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.reviewer: klaasl
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 457e598493ada3dbcbc9dbaaf9f40ea7469bd5b8
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110664868"
---
# <a name="manage-the-access-tier-of-a-blob-in-an-azure-storage-account"></a>Administración del nivel de acceso de un blob en una cuenta de Azure Storage

Cada blob tiene un nivel de acceso predeterminado: frecuente, esporádico o de archivo. Un blob toma el nivel de acceso predeterminado de la cuenta de Azure Storage en la que se crea. Las cuentas de Blob Storage y GPv2 exponen el atributo **access tier** en el nivel de cuenta. Este atributo especifica el nivel de acceso predeterminado de cualquier blob para el que no se haya establecido explícitamente en el nivel de objeto. En el caso de objetos con el nivel establecido en el nivel de objeto, el nivel de cuenta no se aplica. El nivel de archivo solo puede aplicarse en el nivel de objeto. Puede cambiar entre los niveles de acceso en cualquier momento siguiendo los pasos que se indican a continuación.

## <a name="change-the-tier-of-a-blob-in-a-gpv2-or-blob-storage-account"></a>Cambio del nivel de un blob en una cuenta de GPv2 o de Blob Storage

En los escenarios siguientes se usa Azure Portal o PowerShell:

# <a name="portal"></a>[Portal](#tab/portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione **Todos los recursos**.

1. Seleccione la cuenta de almacenamiento.

1. Seleccione el contenedor y, luego, seleccione el blob.

1. En las **propiedades del blob**, seleccione **Cambiar nivel**.

1. Seleccione el nivel de acceso **Frecuente**, **Esporádico** o **Archivo**. Si actualmente el blob usa el nivel de acceso de archivo y quiere rehidratarlo como un nivel en línea, también puede seleccionar una prioridad de rehidratación **Estándar** o **Alta**.

1. En la parte inferior, seleccione **Guardar**.

![Cambio del nivel de blob en Azure Portal](media/storage-tiers/blob-access-tier.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

El siguiente script de PowerShell puede utilizarse para cambiar el nivel de blob. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento. La variable `$containerName` se debe inicializar con el nombre del contenedor. La variable `$blobName` se debe inicializar con el nombre del blob.

```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$containerName = ""
$blobName == ""

#Select the storage account and get the context
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Select the blob from a container
$blob = Get-AzStorageBlob -Container $containerName -Blob $blobName -Context $ctx

#Change the blob’s access tier to archive
$blob.ICloudBlob.SetStandardBlobTier("Archive")
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Administración del nivel de acceso predeterminado en una cuenta de Azure Storage](../common/manage-account-default-access-tier.md)
- [Obtenga información sobre la rehidratación de datos de blob desde el nivel de archivo](storage-blob-rehydration.md)
- [Comprobación de los precios de los niveles de acceso frecuente, esporádico y de archivo en cuentas de Blob Storage y de GPv2 por región](https://azure.microsoft.com/pricing/details/storage/)
