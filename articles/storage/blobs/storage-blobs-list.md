---
title: Enumeración de blobs con API de Azure Storage
description: Obtenga información sobre cómo enumerar los blobs de la cuenta de almacenamiento mediante el uso de las bibliotecas de cliente de Azure Storage. En los ejemplos de código se muestra cómo enumerar los blobs en una lista plana o cómo enumerar los blobs de manera jerárquica, como si estuvieran organizados en directorios o carpetas.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/24/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 9dbcb4aba8b1c25aa7f00850d71872c9f58b94a9
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110462038"
---
# <a name="list-blobs-with-azure-storage-client-libraries"></a>Enumeración de blobs con bibliotecas de cliente de Azure Storage

Al enumerar blobs desde el código, puede especificar una serie de opciones para administrar cómo se devuelven los resultados de Azure Storage. Puede especificar el número de resultados que se van a devolver en cada conjunto de resultados y luego recuperar los conjuntos subsiguientes. Puede especificar un prefijo para devolver los blobs cuyos nombres comienzan por ese carácter o cadena. También puede enumerar los blobs en una estructura de lista plana o de forma jerárquica. Una lista jerárquica devuelve los blobs como si estuvieran organizados en carpetas.

## <a name="understand-blob-listing-options"></a>Descripción de las opciones de enumeración de blobs

Para enumerar los blobs de una cuenta de almacenamiento, llame a uno de estos métodos:

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

- [BlobContainerClient.GetBlobs](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobs)
- [BlobContainerClient.GetBlobsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsasync)
- [BlobContainerClient.GetBlobsByHierarchy](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchy)
- [BlobContainerClient.GetBlobsByHierarchyAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchyasync)

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

- [CloudBlobClient.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobs)
- [CloudBlobClient.ListBlobsSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobssegmented)
- [CloudBlobClient.ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobssegmentedasync)

Para enumerar los blobs de un contenedor, llame a uno de estos métodos:

- [CloudBlobContainer.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobs)
- [CloudBlobContainer.ListBlobsSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmented)
- [CloudBlobContainer.ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync)

# <a name="python-v12-sdk"></a>[SDK para Python v12](#tab/python)

- [ContainerClient.list_blobs](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.containerclient#list-blobs-name-starts-with-none--include-none----kwargs-)

---

### <a name="manage-how-many-results-are-returned"></a>Administración del número de resultados que se devuelven

De forma predeterminada, una operación de enumeración devuelve hasta 5000 resultados a la vez, pero puede especificar el número de resultados que quiere que devuelva. En los ejemplos que se presentan en este artículo muestran cómo devolver resultados por páginas.

### <a name="filter-results-with-a-prefix"></a>Filtrado de los resultados con un prefijo

Para filtrar la lista de blobs, especifique una cadena para el parámetro `prefix`. La cadena de prefijo puede incluir uno o varios caracteres. Después, Azure Storage solo devuelve los blobs cuyos nombres empiecen por ese prefijo.

### <a name="return-metadata"></a>Devolución de metadatos

Puede devolver metadatos de blob con los resultados.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Especifique el valor de **metadatos** para la enumeración [BlobTraits](/dotnet/api/azure.storage.blobs.models.blobtraits).

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Especifique el valor de **metadatos** para la enumeración [BlobListingDetails](/dotnet/api/microsoft.azure.storage.blob.bloblistingdetails). Azure Storage incluye metadatos con cada contenedor que se devuelve, por lo que en este contexto no hay que llamar a uno de los métodos **FetchAttributes** para recuperar los metadatos del contenedor.

# <a name="python-v12-sdk"></a>[SDK para Python v12](#tab/python)

Especifique `metadata` para el parámetro `include=` al llamar a [list_blobs](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.containerclient#list-blobs-name-starts-with-none--include-none----kwargs-).

---

### <a name="list-blob-versions-or-snapshots"></a>Enumeración de versiones o instantáneas de blob

- Para enumerar las versiones o instantáneas de blob con la biblioteca cliente de .NET versión 12, especifique el parámetro [BlobStates](/dotnet/api/azure.storage.blobs.models.blobstates) con el campo **Version** o **Snapshot**. Las instantáneas y las versiones aparecen ordenadas de más antigua a más reciente. Para obtener más información sobre la enumeración de versiones, consulte [Enumeración de versiones de blob](versioning-enable.md#list-blob-versions).

- Para enumerar el número de instantáneas con la biblioteca de cliente de Python V12, especifique `num_snapshots` en el parámetro `include=` al llamar a [list_blobs](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.containerclient#list-blobs-name-starts-with-none--include-none----kwargs-).

### <a name="flat-listing-versus-hierarchical-listing"></a>Lista plana frente a lista jerárquica

Los blobs de Azure Storage están organizados en un paradigma plano, en lugar de un paradigma jerárquico (como un sistema de archivos clásico). Sin embargo, los blobs se pueden organizar en *directorios virtuales* para imitar una estructura de carpetas. Un directorio virtual forma parte del nombre del blob y se indica mediante el carácter delimitador.

Para organizar los blobs en directorios virtuales, use un carácter delimitador en el nombre del blob. El carácter delimitador predeterminado es una barra diagonal (/), pero puede especificar cualquier carácter como delimitador.

Si usa un delimitador para asignar nombres a los blobs, puede optar por enumerar los blobs de forma jerárquica. En el caso de una operación de enumeración jerárquica, Azure Storage devuelve los directorios virtuales y los blobs que hay debajo del objeto primario. Puede llamar a la operación de enumeración de forma recursiva para recorrer la jerarquía, de manera similar a como se haría con un sistema de archivos clásico mediante programación.

## <a name="use-a-flat-listing"></a>Uso de una lista plana

De forma predeterminada, una operación de enumeración devuelve los blobs en una lista plana. En una lista plana, los blobs no se organizan por directorio virtual.

En el ejemplo siguiente se enumeran los blobs del contenedor especificado mediante una lista plana, con un tamaño de segmento opcional especificado, y se escribe el nombre del blob en una ventana de consola.

Si ha habilitado la característica de espacio de nombres jerárquico en su cuenta, los directorios no son virtuales. En su lugar, son objetos independientes y concretos. Por lo tanto, los directorios aparecen en la lista como blobs de longitud cero.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobsFlatListing":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Si una operación de enumeración devuelve más de 5000 blobs, o si el número de blobs disponibles supera el número especificado, Azure Storage devuelve un *token de continuación* con la lista de blobs. Un token de continuación es un valor opaco que se puede usar para recuperar el siguiente conjunto de resultados de Azure Storage.

En el código, compruebe el valor del token de continuación para determinar si es NULL. Cuando el token de continuación es NULL, el conjunto de resultados está completo. Si el token de continuación no es NULL, llame de nuevo a la operación de enumeración y pase el token de continuación para recuperar el siguiente conjunto de resultados, hasta que el token de continuación sea NULL.

```csharp
private static async Task ListBlobsFlatListingAsync(CloudBlobContainer container, int? segmentSize)
{
    BlobContinuationToken continuationToken = null;
    CloudBlob blob;

    try
    {
        // Call the listing operation and enumerate the result segment.
        // When the continuation token is null, the last segment has been returned
        // and execution can exit the loop.
        do
        {
            BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(string.Empty,
                true, BlobListingDetails.Metadata, segmentSize, continuationToken, null, null);

            foreach (var blobItem in resultSegment.Results)
            {
                blob = (CloudBlob)blobItem;

                // Write out some blob properties.
                Console.WriteLine("Blob name: {0}", blob.Name);
            }

            Console.WriteLine();

           // Get the continuation token and loop until it is null.
           continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

# <a name="python-v12-sdk"></a>[SDK para Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/list_blobs.py" id="Snippet_ListBlobs":::

---

La salida es parecida a esta:

```console
Blob name: FolderA/blob1.txt
Blob name: FolderA/blob2.txt
Blob name: FolderA/blob3.txt
Blob name: FolderA/FolderB/blob1.txt
Blob name: FolderA/FolderB/blob2.txt
Blob name: FolderA/FolderB/blob3.txt
Blob name: FolderA/FolderB/FolderC/blob1.txt
Blob name: FolderA/FolderB/FolderC/blob2.txt
Blob name: FolderA/FolderB/FolderC/blob3.txt
```

## <a name="use-a-hierarchical-listing"></a>Uso de una lista jerárquica

Cuando se llama a una operación de enumeración jerárquicamente, Azure Storage devuelve los directorios virtuales y los blobs en el primer nivel de la jerarquía.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Para enumerar los blobs de forma jerárquica, llame al método [BlobContainerClient.GetBlobsByHierarchy](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchy) o [BlobContainerClient.GetBlobsByHierarchyAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchyasync).

En el ejemplo siguiente se enumeran los blobs del contenedor especificado mediante una lista jerárquica, con un tamaño de segmento opcional especificado, y se escribe el nombre del blob en la ventana de consola.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobsHierarchicalListing":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

La propiedad [Prefix](/dotnet/api/microsoft.azure.storage.blob.cloudblobdirectory.prefix) de cada directorio virtual se establece de manera que pueda pasar el prefijo en una llamada recursiva para recuperar el siguiente directorio.

Para enumerar los blobs de forma jerárquica, establezca el parámetro `useFlatBlobListing` del método de enumeración en **false**.

En el ejemplo siguiente se enumeran los blobs del contenedor especificado mediante una lista plana, con un tamaño de segmento opcional especificado, y se escribe el nombre del blob en la ventana de consola.

```csharp
private static async Task ListBlobsHierarchicalListingAsync(CloudBlobContainer container, string prefix)
{
    CloudBlobDirectory dir;
    CloudBlob blob;
    BlobContinuationToken continuationToken = null;

    try
    {
        // Call the listing operation and enumerate the result segment.
        // When the continuation token is null, the last segment has been returned and
        // execution can exit the loop.
        do
        {
            BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(prefix,
                false, BlobListingDetails.Metadata, null, continuationToken, null, null);
            foreach (var blobItem in resultSegment.Results)
            {
                // A hierarchical listing may return both virtual directories and blobs.
                if (blobItem is CloudBlobDirectory)
                {
                    dir = (CloudBlobDirectory)blobItem;

                    // Write out the prefix of the virtual directory.
                    Console.WriteLine("Virtual directory prefix: {0}", dir.Prefix);

                    // Call recursively with the prefix to traverse the virtual directory.
                    await ListBlobsHierarchicalListingAsync(container, dir.Prefix);
                }
                else
                {
                    // Write out the name of the blob.
                    blob = (CloudBlob)blobItem;

                    Console.WriteLine("Blob name: {0}", blob.Name);
                }
                Console.WriteLine();
            }

            // Get the continuation token and loop until it is null.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

# <a name="python-v12-sdk"></a>[SDK para Python v12](#tab/python)

Para enumerar los blobs jerárquicamente, llame al método [walk_blobs](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.containerclient#walk-blobs-name-starts-with-none--include-none--delimiter--------kwargs-).

En el ejemplo siguiente se enumeran los blobs del contenedor especificado mediante una lista jerárquica, con un tamaño de segmento opcional especificado, y se escribe el nombre del blob en la ventana de consola.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/list_blobs.py" id="Snippet_WalkHierarchy":::

---

La salida es parecida a esta:

```console
Virtual directory prefix: FolderA/
Blob name: FolderA/blob1.txt
Blob name: FolderA/blob2.txt
Blob name: FolderA/blob3.txt

Virtual directory prefix: FolderA/FolderB/
Blob name: FolderA/FolderB/blob1.txt
Blob name: FolderA/FolderB/blob2.txt
Blob name: FolderA/FolderB/blob3.txt

Virtual directory prefix: FolderA/FolderB/FolderC/
Blob name: FolderA/FolderB/FolderC/blob1.txt
Blob name: FolderA/FolderB/FolderC/blob2.txt
Blob name: FolderA/FolderB/FolderC/blob3.txt
```

> [!NOTE]
> Las instantáneas de blob no se pueden enumerar en una operación de enumeración jerárquica.

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Enumeración de blobs](/rest/api/storageservices/list-blobs)
- [Enumeración de recursos de blob](/rest/api/storageservices/enumerating-blob-resources)