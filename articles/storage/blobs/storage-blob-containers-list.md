---
title: 'Enumeración de contenedores de blob con .NET: Azure Storage'
description: Obtenga información sobre cómo enumerar los contenedores de blobs en la cuenta de Azure Storage mediante la biblioteca cliente .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 10/14/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 40dade86882cdfa8f85015bebc8769430fbf3f1a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110477626"
---
# <a name="list-blob-containers-with-net"></a>Enumeración de contenedores de blob con .NET

Al enumerar los contenedores de una cuenta de Azure Storage desde el código, puede especificar una serie de opciones para administrar cómo se devuelven los resultados de Azure Storage. En este artículo se muestra cómo enumerar contenedores con [la biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage).  

## <a name="understand-container-listing-options"></a>Descripción de las opciones de enumeración de contenedores

Para enumerar los contenedores de la cuenta de almacenamiento, llame a uno de los métodos siguientes:

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

- [GetBlobContainers](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainers)
- [GetBlobContainersAsync](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainersasync)

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

- [ListContainersSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmented)
- [ListContainersSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmentedasync)

---

Las sobrecargas de estos métodos proporcionan opciones adicionales para administrar cómo devuelve los contenedores la operación de enumeración. Estas opciones se describen en las secciones siguientes.

### <a name="manage-how-many-results-are-returned"></a>Administración del número de resultados que se devuelven

De forma predeterminada, una operación de enumeración devuelve hasta 5000 resultados a la vez. Para devolver un conjunto de resultados más pequeño, proporcione un valor distinto de cero para el tamaño de la página de resultados que se va a devolver.

Si la cuenta de almacenamiento contiene más de 5000 contenedores o si ha especificado un tamaño de página para que la operación de enumeración devuelva un subconjunto de contenedores de la cuenta de almacenamiento, Azure Storage devuelve un *token de continuación* con la lista de contenedores. Un token de continuación es un valor opaco que se puede usar para recuperar el siguiente conjunto de resultados de Azure Storage.

En el código, compruebe el valor del token de continuación para determinar si está vacío (para la versión 12 de .NET) o es NULL (para la versión 11 y anteriores de .NET). Cuando el token de continuación es NULL, el conjunto de resultados está completo. Si el token de continuación no es NULL, llame de nuevo al método de enumeración y pase el token de continuación para recuperar el conjunto de resultados siguiente, hasta que el token de continuación sea NULL.

### <a name="filter-results-with-a-prefix"></a>Filtrado de los resultados con un prefijo

Para filtrar la lista de contenedores, especifique una cadena para el parámetro `prefix`. La cadena de prefijo puede incluir uno o varios caracteres. Después, Azure Storage solo devuelve los contenedores cuyos nombres empiecen por ese prefijo.

### <a name="return-metadata"></a>Devolución de metadatos

Para devolver metadatos de contenedor con los resultados, especifique el valor **Metadata** de la enumeración [BlobContainerTraits](/dotnet/api/azure.storage.blobs.models.blobcontainertraits) (para la versión 12 de .NET) o la enumeración [ContainerListingDetails](/dotnet/api/microsoft.azure.storage.blob.containerlistingdetails) (para la versión 11 y anteriores de .NET). Azure Storage incluye metadatos con cada contenedor devuelto, por lo que no es necesario capturar también los metadatos del contenedor.

## <a name="example-list-containers"></a>Ejemplo: Enumerar contenedores

En el ejemplo siguiente se enumeran de forma asincrónica los contenedores de una cuenta de almacenamiento que comienzan con un prefijo especificado. En el ejemplo se muestran los contenedores que comienzan con el prefijo especificado y se devuelve el número especificado de resultados por llamada a la operación de enumeración. A continuación, usa el token de continuación para obtener el segmento de resultados siguiente. En el ejemplo también se devuelven metadatos de contenedor con los resultados.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="Snippet_ListContainers":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

```csharp
private static async Task ListContainersWithPrefixAsync(CloudBlobClient blobClient,
                                                        string prefix,
                                                        int? segmentSize)
{
    Console.WriteLine("List containers beginning with prefix {0}, plus container metadata:", prefix);

    BlobContinuationToken continuationToken = null;
    ContainerResultSegment resultSegment;

    try
    {
        do
        {
            // List containers beginning with the specified prefix,
            // returning segments of 5 results each.
            // Passing in null for the maxResults parameter returns the maximum number of results (up to 5000).
            // Requesting the container's metadata as part of the listing operation populates the metadata,
            // so it's not necessary to call FetchAttributes() to read the metadata.
            resultSegment = await blobClient.ListContainersSegmentedAsync(
                prefix, ContainerListingDetails.Metadata, segmentSize, continuationToken, null, null);

            // Enumerate the containers returned.
            foreach (var container in resultSegment.Results)
            {
                Console.WriteLine("\tContainer:" + container.Name);

                // Write the container's metadata keys and values.
                foreach (var metadataItem in container.Metadata)
                {
                    Console.WriteLine("\t\tMetadata key: " + metadataItem.Key);
                    Console.WriteLine("\t\tMetadata value: " + metadataItem.Value);
                }
            }

            // Get the continuation token.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);

        Console.WriteLine();
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>Consulte también

- [Enumerador de contenedores](/rest/api/storageservices/list-containers2)
- [Enumeración de recursos de blob](/rest/api/storageservices/enumerating-blob-resources)
