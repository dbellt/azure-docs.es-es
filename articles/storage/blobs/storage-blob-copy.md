---
title: Copia de un blob con las API de Azure Storage
description: Aprenda a copiar un blob mediante las bibliotecas cliente de Azure Storage.
services: storage
author: normesta
ms.author: normesta
ms.date: 01/08/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 64bffb127c7d3fbcf56619ea17b23fc0a58d6a37
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110652884"
---
# <a name="copy-a-blob-with-azure-storage-client-libraries"></a>Copia de un blob con las bibliotecas cliente de Azure Storage

En este artículo se muestra cómo copiar un blob en una cuenta de Azure Storage. También se muestra cómo anular una operación de copia asincrónica. El código de ejemplo usa las bibliotecas cliente de Azure Storage.

## <a name="about-copying-blobs"></a>Acerca de la copia de blobs

Cuando se copia un blob dentro de la misma cuenta de almacenamiento, se trata de una operación sincrónica. Cuando la copia se realiza entre cuentas, se trata de una operación asincrónica.

El blob de origen para una operación de copia puede ser un blob en bloques, un blob en anexos, un blob en páginas o una instantánea. Si el blob de destino ya existe, debe ser del mismo tipo que el blob de origen. Si existe un blob de destino, se sobrescribirá.

No se puede modificar el blob de destino mientras haya una operación de copia en curso. Un blob de destino solo puede tener una operación de copia pendiente. En otras palabras, un blob no puede ser el destino de varias operaciones de copia pendientes.

Siempre se copia el blob o el archivo de origen completo, ya que no se admite la copia de un intervalo de bytes o un conjunto de bloques.

Cuando se copia un blob, las propiedades del sistema al que pertenece se copian en el blob de destino con los mismos valores.

Una operación de copia puede tomar cualquiera de las siguientes formas:

- Copie un blob de origen en un blob de destino con un nombre diferente. El blob de destino puede ser un blob existente del mismo tipo (en bloques, anexos o páginas), o puede ser un nuevo blob creado por la operación de copia.
- Copie un blob de origen en un blob de destino con el mismo nombre, con lo que se reemplazará de forma efectiva el blob de destino. Este tipo de operación de copia quita los bloques sin confirmar y sobrescribe los metadatos del blob de destino.
- Copie un archivo de origen del servicio Azure File en un blob de destino. El blob de destino puede ser un blob en bloques existente o un nuevo blob de ese tipo creado por la operación de copia. No se admite la copia de archivos a blobs en páginas o blobs en anexos.
- Copie una instantánea sobre su blob base. Si se mueve una instantánea a la posición del blob, puede restaurar una versión anterior de un blob.
- Copie una instantánea en un blob de destino con otro nombre. El blob resultante de destino es un blob en el que se puede escribir y no una instantánea.

## <a name="copy-a-blob"></a>Copia de un blob

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Para crear un blob, llame a uno de los métodos siguientes:

- [StartCopyFromUri](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuri)
- [StartCopyFromUriAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuriasync)

Los métodos `StartCopyFromUri` y `StartCopyFromUriAsync` devuelven un objeto [CopyFromUriOperation](/dotnet/api/azure.storage.blobs.models.copyfromurioperation) que contiene información sobre la operación de copia.

En el ejemplo de código siguiente se obtiene un objeto [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) que representa un blob creado previamente y se copia en un nuevo blob del mismo contenedor:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_CopyBlob":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Para crear un blob, llame a uno de los métodos siguientes:

- [StartCopy](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopy)
- [StartCopyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopyasync)

Los métodos `StartCopy` y `StartCopyAsync` devuelven un valor de Id. de copia que se usa para comprobar el estado de la operación de copia o anularla.

En el ejemplo de código siguiente se obtiene una referencia a un blob creado previamente y se copia en un nuevo blob del mismo contenedor:

```csharp
private static async Task CopyBlockBlobAsync(CloudBlobContainer container)
{
    CloudBlockBlob sourceBlob = null;
    CloudBlockBlob destBlob = null;
    string leaseId = null;

    try
    {
        // Get a block blob from the container to use as the source.
        sourceBlob = container.ListBlobs().OfType<CloudBlockBlob>().FirstOrDefault();

        // Lease the source blob for the copy operation 
        // to prevent another client from modifying it.
        // Specifying null for the lease interval creates an infinite lease.
        leaseId = await sourceBlob.AcquireLeaseAsync(null);

        // Get a reference to a destination blob (in this case, a new blob).
        destBlob = container.GetBlockBlobReference("copy of " + sourceBlob.Name);

        // Ensure that the source blob exists.
        if (await sourceBlob.ExistsAsync())
        {
            // Get the ID of the copy operation.
            string copyId = await destBlob.StartCopyAsync(sourceBlob);

            // Fetch the destination blob's properties before checking the copy state.
            await destBlob.FetchAttributesAsync();

            Console.WriteLine("Status of copy operation: {0}", destBlob.CopyState.Status);
            Console.WriteLine("Completion time: {0}", destBlob.CopyState.CompletionTime);
            Console.WriteLine("Bytes copied: {0}", destBlob.CopyState.BytesCopied.ToString());
            Console.WriteLine("Total bytes: {0}", destBlob.CopyState.TotalBytes.ToString());
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
    finally
    {
        // Break the lease on the source blob.
        if (sourceBlob != null)
        {
            await sourceBlob.FetchAttributesAsync();

            if (sourceBlob.Properties.LeaseState != LeaseState.Available)
            {
                await sourceBlob.BreakLeaseAsync(new TimeSpan(0));
            }
        }
    }
}
```

# <a name="python-v12-sdk"></a>[SDK para Python v12](#tab/python)

Para copiar un blob, llame al método [start_copy_from_url](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#start-copy-from-url-source-url--metadata-none--incremental-copy-false----kwargs-). El método `start_copy_from_url` devuelve un diccionario que contiene información sobre la operación de copia.

En el ejemplo de código siguiente se obtiene un objeto [BlobClient](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient) que representa un blob creado previamente y se copia en un nuevo blob del mismo contenedor:

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/copy_blob.py" id="Snippet_BlobCopy":::

---

## <a name="abort-a-copy-operation"></a>Anulación de una operación de copia

La anulación una operación de copia da como resultado un blob de destino de longitud cero. Sin embargo, los nuevos valores de los metadatos del blob de destino se copiarán del blob de origen o se establecerán explícitamente en la llamada a la operación de copia. Para conservar los metadatos originales anteriores a la copia, cree una instantánea del blob de destino antes de llamar a uno de los métodos de copia.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Compruebe la propiedad [BlobProperties.CopyStatus](/dotnet/api/azure.storage.blobs.models.blobproperties.copystatus) en el blob de destino para obtener el estado de la operación de copia. El blob final se confirmará cuando se complete la copia.

Cuando se anula una operación de copia, el estado de la copia del blob de destino se establece en [CopyState.Aborted](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Los métodos [AbortCopyFromUri](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuri) y [AbortCopyFromUriAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuriasync) cancelan una operación de copia en curso.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_StopBlobCopy":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Compruebe la propiedad [CopyState.Status](/dotnet/api/microsoft.azure.storage.blob.copystate.status) en el blob de destino para obtener el estado de la operación de copia. El blob final se confirmará cuando se complete la copia.

Cuando se anula una operación de copia, el estado de la copia del blob de destino se establece en [CopyState.Aborted](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Los métodos [AbortCopy](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopy) y [AbortCopyAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopyasync) cancelan una operación de copia en curso.

```csharp
// Fetch the destination blob's properties before checking the copy state.
await destBlob.FetchAttributesAsync();

// Check the copy status. If it is still pending, abort the copy operation.
if (destBlob.CopyState.Status == CopyStatus.Pending)
{
    await destBlob.AbortCopyAsync(copyId);
    Console.WriteLine("Copy operation {0} has been aborted.", copyId);
}
```

# <a name="python-v12-sdk"></a>[SDK para Python v12](#tab/python)

Compruebe en el diccionario [CopyProperties](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) la entrada "status" devuelta por el método [get_blob_properties](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#get-blob-properties---kwargs-) para obtener el estado de la operación de copia. El blob final se confirmará cuando se complete la copia.

Cuando se anula una operación de copia, el valor de [status](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) se establece en "aborted".

El método [abort_copy](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#abort-copy-copy-id----kwargs-) cancela una operación de copia en curso.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/copy_blob.py" id="Snippet_StopBlobCopy":::

---

## <a name="azure-sdks"></a>SDK de Azure

Obtenga más información sobre los SDK de Azure:

 - [SDK de Azure para .NET](https://github.com/azure/azure-sdk-for-net)
 - [SDK de Azure para Java](https://github.com/azure/azure-sdk-for-java)
 - [SDK de Azure para Python](https://github.com/azure/azure-sdk-for-python)
 - [Azure SDK para JavaScript](https://github.com/azure/azure-sdk-for-js)

## <a name="next-steps"></a>Pasos siguientes

Los temas siguientes contienen información sobre la copia de blobs y la anulación de las operaciones de copia en curso mediante las API REST de Azure.

- [Copy Blob](/rest/api/storageservices/copy-blob)
- [Abort Copy Blob](/rest/api/storageservices/abort-copy-blob)
