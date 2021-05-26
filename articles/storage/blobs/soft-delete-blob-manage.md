---
title: Administración y restauración de blobs eliminados temporalmente
titleSuffix: Azure Storage
description: Administre y restaure las instantáneas y los blobs eliminados temporalmente con Azure Portal o con las bibliotecas cliente de Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/27/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 7e951de46b5220e5c2edde2fcd84673c9a16cebc
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110477712"
---
# <a name="manage-and-restore-soft-deleted-blobs"></a>Administración y restauración de blobs eliminados temporalmente

La eliminación temporal de blobs protege a cada uno de los blobs y sus versiones, instantáneas y metadatos de errores accidentales al borrar o sobrescribir los datos, ya que conserva en el sistema los datos eliminados durante el período de tiempo que se especifique. Durante este período de retención, los blobs pueden restaurarse a su estado original. Una vez vencido el período de retención especificado, el blob se elimina permanentemente. Para más información sobre la eliminación temporal, consulte este artículo sobre la [eliminación temporal de blobs](soft-delete-blob-overview.md).

La eliminación temporal de blobs forma parte de una exhaustiva estrategia para proteger los datos de los blobs. Si necesita más información sobre las recomendaciones de Microsoft para proteger los datos, consulte [Información general sobre la protección de datos](data-protection-overview.md).

## <a name="manage-soft-deleted-blobs-with-the-azure-portal"></a>Administración de blobs eliminados temporalmente con Azure Portal

Puede usar Azure Portal para ver y restaurar las instantáneas y los blobs eliminados temporalmente.

### <a name="view-deleted-blobs"></a>Visualización de los blobs eliminados

Cuando los blobs se eliminan temporalmente, son invisibles en Azure Portal de manera predeterminada. Para ver los blobs eliminados temporalmente, vaya a la página **Información general** del contenedor y active la opción **Mostrar blobs eliminados**. Los blobs eliminados temporalmente se muestran con el estado **Eliminado**.

:::image type="content" source="media/soft-delete-blob-manage/soft-deleted-blobs-list-portal.png" alt-text="Captura de pantalla que muestra cómo enumerar los blobs eliminados temporalmente en Azure Portal":::

A continuación, seleccione el blob eliminado de la lista de blobs para mostrar sus propiedades. En la pestaña **Información general**, observe que el estado del blob es **Eliminado**. El portal también muestra el número de días hasta que el blob se elimina permanentemente.

:::image type="content" source="media/soft-delete-blob-manage/soft-deleted-blob-properties-portal.png" alt-text="Captura de pantalla que muestra las propiedades del blob eliminado temporalmente en Azure Portal":::

### <a name="view-deleted-snapshots"></a>Visualización de las instantáneas eliminadas

Al eliminar un blob también se eliminan todas las instantáneas que tiene asociadas. Si un blob eliminado temporalmente tiene instantáneas, las instantáneas eliminadas también se pueden mostrar en el portal. Muestre las propiedades del blob eliminado temporalmente, vaya a la pestaña **Instantáneas** y active la opción **Mostrar instantáneas eliminadas**.

:::image type="content" source="media/soft-delete-blob-manage/soft-deleted-blob-snapshots-portal.png" alt-text="Captura de pantalla que muestra ":::

### <a name="restore-soft-deleted-objects-when-versioning-is-disabled"></a>Restauración de objetos eliminados temporalmente cuando el control de versiones está deshabilitado

Para restaurar un blob eliminado temporalmente en Azure Portal cuando el control de versiones de blob no esté habilitado, primero muestre las propiedades del blob y, a continuación, seleccione el botón **Recuperar** en la pestaña **Información general**. La restauración de un blob también restaura las instantáneas que se eliminaron durante el período de retención de eliminación temporal.

:::image type="content" source="media/soft-delete-blob-manage/undelete-soft-deleted-blob-portal.png" alt-text="Captura de pantalla que muestra cómo restaurar un contenedor eliminado temporalmente en Azure Portal":::

Para promover una instantánea eliminada temporalmente en el blob base, primero asegúrese de que se han restaurado las instantáneas de eliminación temporal del blob. Seleccione el botón **Recuperar** para restaurar las instantáneas eliminadas temporalmente del blob, incluso si el blob base no se ha eliminado temporalmente. A continuación, seleccione la instantánea que desea promover y use el botón **Promover instantánea** para sobrescribir el blob base con el contenido de la instantánea.

:::image type="content" source="media/soft-delete-blob-manage/promote-snapshot.png" alt-text="Captura de pantalla que muestra cómo promover una instantánea al blob base":::

### <a name="restore-soft-deleted-blobs-when-versioning-is-enabled"></a>Restauración de blobs eliminados temporalmente cuando el control de versiones está habilitado

Para restaurar un blob eliminado temporalmente en Azure Portal cuando el control de versiones está habilitado, seleccione el blob eliminado temporalmente para mostrar sus propiedades y, a continuación, seleccione la pestaña **Versiones**. Seleccione la versión que desea promover como versión actual y, después, seleccione **Crear versión actual**.  

:::image type="content" source="media/soft-delete-blob-manage/soft-deleted-blob-promote-version-portal.png" alt-text="Captura de pantalla que muestra cómo promover una versión para restaurar un blob en Azure Portal":::

Para restaurar las versiones o instantáneas eliminadas cuando el control de versiones está habilitado, muestre las propiedades del blob y, a continuación, seleccione el botón **Recuperar** en la pestaña **Información general**.

> [!NOTE]
> Cuando el control de versiones está habilitado, al seleccionar el botón **Recuperar** en un blob eliminado, se restauran las instantáneas o versiones eliminadas temporalmente, pero no se restaura el blob base. Para restaurar el blob base, debe promover una versión anterior.

## <a name="manage-soft-deleted-blobs-with-code"></a>Administración de blobs eliminados temporalmente con código

Puede usar las bibliotecas cliente de Azure Storage para restaurar un blob o una instantánea eliminados temporalmente. En el siguiente ejemplo, se muestra cómo usar la biblioteca cliente de .NET.

### <a name="restore-soft-deleted-objects-when-versioning-is-disabled"></a>Restauración de objetos eliminados temporalmente cuando el control de versiones está deshabilitado

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Para restaurar los blobs eliminados cuando el control de versiones no está habilitado, llame a la operación [Undelete Blob](/rest/api/storageservices/undelete-blob) en esos blobs. La operación **Undelete Blob** restaura los blobs eliminados temporalmente y las instantáneas eliminadas que tienen asociadas.

La llamada a **Undelete Blob** en un blob que no se ha eliminado no tiene ningún efecto. En el ejemplo siguiente se llama a **Undelete Blob** en todos los blobs de un contenedor y se restauran los blobs eliminados temporalmente y sus instantáneas:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverDeletedBlobs":::

Para restaurar una versión específica, primero llame a la operación **Undelete Blob** en la versión o el blob base y, a continuación, copie la versión deseada en el blob base. En el ejemplo siguiente se restaura un blob en bloques a la última versión guardada:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverSpecificBlobSnapshot":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Para restaurar los blobs eliminados cuando el control de versiones no está habilitado, llame a la operación [Undelete Blob](/rest/api/storageservices/undelete-blob) en esos blobs. La operación **Undelete Blob** restaura los blobs eliminados temporalmente y las instantáneas eliminadas que tienen asociadas.

La llamada a **Undelete Blob** en un blob que no se ha eliminado no tiene ningún efecto. En el ejemplo siguiente se llama a **Undelete Blob** en todos los blobs de un contenedor y se restauran los blobs eliminados temporalmente y sus instantáneas:

```csharp
// Restore all blobs in a container.
foreach (CloudBlob blob in container.ListBlobs(useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Deleted))
{
       await blob.UndeleteAsync();
}
```

Para restaurar una instantánea específica, primero llame a la operación **Undelete Blob** en el blob base y, a continuación, copie la instantánea deseada en el blob base. En el ejemplo siguiente se restaura un blob en bloques a su última instantánea generada:

```csharp
// Restore the block blob.
await blockBlob.UndeleteAsync();

// List all blobs and snapshots in the container, prefixed by the blob name.
IEnumerable<IListBlobItem> allBlobSnapshots = container.ListBlobs(
    prefix: blockBlob.Name, useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Snapshots);

// Copy the most recently generated snapshot to the base blob.
CloudBlockBlob copySource = allBlobSnapshots.First(snapshot => ((CloudBlockBlob)version).IsSnapshot &&
    ((CloudBlockBlob)snapshot).Name == blockBlob.Name) as CloudBlockBlob;
blockBlob.StartCopy(copySource);
```  

---

### <a name="restore-soft-deleted-blobs-when-versioning-is-enabled"></a>Restauración de blobs eliminados temporalmente cuando el control de versiones está habilitado

Para restaurar un blob eliminado temporalmente cuando el control de versiones está habilitado, copie una versión anterior en el blob base con una operación [Copy Blob](/rest/api/storageservices/copy-blob) o [Copy Blob From URL](/rest/api/storageservices/copy-blob-from-url).  

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RestorePreviousVersion":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

No es aplicable. El control de versiones de blobs solo se admite en la versión 12.x de las bibliotecas cliente de Azure Storage y en versiones posteriores.

---

## <a name="next-steps"></a>Pasos siguientes

- [Eliminación temporal para Blob Storage](./soft-delete-blob-overview.md)
- [Habilitación de la eliminación temporal para blobs](soft-delete-blob-enable.md)
- [Control de versiones de blobs](versioning-overview.md)
