---
title: Administración de la simultaneidad en Blob Storage
titleSuffix: Azure Storage
description: Obtenga información sobre cómo administrar varios escritores en un blob implementando la simultaneidad optimista o pesimista en la aplicación. La simultaneidad optimista comprueba el valor de ETag (etiqueta de entidad) de un blob y lo compara con la etiqueta de entidad proporcionada. La simultaneidad pesimista utiliza una concesión exclusiva para bloquear el blob en otros escritores.
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 53005bffde698030221751ec0638a6cc6cbd98c7
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110478940"
---
# <a name="managing-concurrency-in-blob-storage"></a>Administrar la simultaneidad en almacenamiento de blobs

A menudo, las aplicaciones modernas tienen varios usuarios que ven y actualizan datos simultáneamente. Los desarrolladores de aplicaciones deben pensar detenidamente cómo proporcionar una experiencia predecible a sus usuarios finales, especialmente para escenarios en los que varios usuarios pueden actualizar los mismos datos. Hay tres estrategias principales de simultaneidad de datos que normalmente tienen en cuenta los desarrolladores:  

- **Simultaneidad optimista**: una aplicación que realiza una actualización determinará, como parte de dicha actualización, si los datos han cambiado desde la última vez que los leyó. Por ejemplo, si dos usuarios que ven una página wiki realizan una actualización en la misma página, la plataforma wiki deberá asegurarse de que la segunda actualización no sobrescribe la primera y que ambos usuarios comprenden si su actualización se realizó correctamente. Esta estrategia se usa con más frecuencia en aplicaciones web.

- **Simultaneidad pesimista**: una aplicación que pretende realizar una actualización realizará un bloqueo en un objeto y evitará que otros usuarios actualicen los datos hasta que el bloqueo se libere. Por ejemplo, en un escenario de replicación de datos principal/secundario, donde solamente el principal realiza actualizaciones, el principal normalmente mantendrá un bloqueo exclusivo en los datos durante un período de tiempo extendido para garantizar que ninguna otra persona puede actualizarlos.

- **Últimos casos de escritura correcta**: un enfoque que permite que las operaciones de actualización continúen sin determinar primero si otra aplicación actualizó los datos desde que se leyeron. Este enfoque se suele utilizar cuando se hacen particiones de los datos de forma que varios usuarios no tengan acceso a ellos al mismo tiempo. También puede resultar útil donde se procesen transmisiones de datos de corta duración.

Azure Storage admite las tres estrategias, aunque se distingue por su capacidad para proporcionar compatibilidad completa con la simultaneidad optimista y pesimista. Azure Storage se diseñó para adoptar un modelo de coherencia sólido que garantiza que, después de que el servicio realice una operación de inserción o actualización, las operaciones de lectura posteriores devuelven la actualización más reciente.

Además de seleccionar una estrategia de simultaneidad adecuada, los desarrolladores también deben saber cómo una plataforma de almacenamiento aísla los cambios, especialmente aquellos del mismo objeto en todas las transacciones. Azure Storage usa el aislamiento de instantáneas para permitir que las operaciones de lectura tengan lugar simultáneamente con operaciones de escritura dentro de una sola partición. El aislamiento de instantáneas garantiza que todas las operaciones de lectura devuelvan una instantánea coherente de los datos incluso mientras se realicen actualizaciones.

Puede optar por usar modelos de simultaneidad optimista o pesimista para administrar el acceso a los blobs y los contenedores. Si no especifica explícitamente ninguna estrategia, de manera predeterminada la última escritura será la correcta.  

## <a name="optimistic-concurrency"></a>Simultaneidad optimista

Azure Storage asigna un identificador a cada objeto almacenado. Este se actualiza cada vez que se realiza una operación de escritura en un objeto. El identificador se devuelve al cliente como parte de una respuesta HTTP GET en el encabezado ETag que se define mediante el protocolo HTTP.

Un cliente que realiza una actualización puede enviar la etiqueta ETag original junto con un encabezado condicional a fin de garantizar que solo se realizará una actualización si se cumple una determinada condición. Por ejemplo, si se especifica el encabezado **If-Match**, Azure Storage verificará que el valor de ETag especificado en la solicitud de actualización sea el mismo que el de la etiqueta de entidad del objeto que se está actualizando. Para obtener más información sobre los encabezados condicionales, consulte [Especificación de encabezados condicionales para las operaciones de Blob service](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).

El esquema de este proceso es el siguiente:  

1. Recupere un blob de Azure Storage. La respuesta incluye un valor de encabezado ETag HTTP que identifica la versión actual del objeto.
1. Cuando actualice el blob, incluya el valor de ETag recibido en el paso 1 en el encabezado condicional **If-Match** de la solicitud de escritura. Azure Storage comparará el valor de ETag de la solicitud con el valor de ETag actual del blob.
1. Si el valor de ETag actual del blob difiere del especificado en el encabezado condicional **If-Match** proporcionado en la solicitud, Azure Storage devolverá el código de estado HTTP 412 (error de condición previa). Este error indica al cliente que otro proceso ha actualizado el blob desde que el cliente lo recuperó por primera vez.
1. Si la versión del valor de ETag actual del blob es la misma que la del valor de ETag del encabezado condicional **If-Match** de la solicitud, Azure Storage realizará la operación solicitada y actualizará el valor de ETag actual del blob.  

En los siguientes ejemplos de código se muestra cómo construir una condición **If-Match** en la solicitud de escritura que compruebe el valor de ETag para un blob. Azure Storage evalúa si el valor de ETag actual del blob es igual al de la etiqueta de entidad proporcionada en la solicitud y realiza la operación de escritura solo si ambos coinciden. Si otro proceso ha actualizado el blob provisionalmente, Blob Storage devuelve un mensaje de estado HTTP 412 (error en la condición previa).  

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Concurrency.cs" id="Snippet_DemonstrateOptimisticConcurrencyBlob":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnetv11)

```csharp
public void DemonstrateOptimisticConcurrencyBlob(string containerName, string blobName)
{
    Console.WriteLine("Demonstrate optimistic concurrency");

    // Parse connection string and create container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExists();

    // Create test blob. The default strategy is last writer wins, so
    // write operation will overwrite existing blob if present.
    CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
    blockBlob.UploadText("Hello World!");

    // Retrieve the ETag from the newly created blob.
    string originalETag = blockBlob.Properties.ETag;
    Console.WriteLine("Blob added. Original ETag = {0}", originalETag);

    /// This code simulates an update by another client.
    string helloText = "Blob updated by another client.";
    // No ETag was provided, so original blob is overwritten and ETag updated.
    blockBlob.UploadText(helloText);
    Console.WriteLine("Blob updated. Updated ETag = {0}", blockBlob.Properties.ETag);

    // Now try to update the blob using the original ETag value.
    try
    {
        Console.WriteLine(@"Attempt to update blob using original ETag
                            to generate if-match access condition");
        blockBlob.UploadText(helloText, accessCondition: AccessCondition.GenerateIfMatchCondition(originalETag));
    }
    catch (StorageException ex)
    {
        if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        {
            Console.WriteLine(@"Precondition failure as expected.
                                Blob's ETag does not match.");
        }
        else
        {
            throw;
        }
    }
    Console.WriteLine();
}
```

---

Azure Storage también admite otros encabezados condicionales, incluidos **If-Modified-Since**, **If-Unmodified-Since** y **If-None-Match**. Para obtener más información, consulte [Especificación de encabezados condicionales para las operaciones de Blob Service](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).  

## <a name="pessimistic-concurrency-for-blobs"></a>Simultaneidad pesimista para blobs

Para bloquear un blob para uso exclusivo, puede adquirir una concesión en él. Al adquirir la concesión, se especifica la duración de la concesión. Una concesión finita puede ser válida entre 15 y 60 segundos. Una concesión también puede ser infinita, lo que equivale a un bloqueo exclusivo. Puede renovar una concesión finita para extenderla y puede liberarla cuando haya terminado con ella. Azure Storage libera automáticamente concesiones finitas cuando expiran.  

Las concesiones permiten que se admitan diferentes estrategias de sincronización, como, por ejemplo, las siguientes: operaciones de escritura exclusiva o lectura compartida, operaciones de escritura exclusiva o lectura exclusiva y operaciones de escritura compartida o lectura exclusiva. Cuando existe una concesión, Azure Storage fuerza el acceso exclusivo a las operaciones de escritura para el titular de la concesión. Sin embargo, el hecho de asegurar la exclusividad para las operaciones de lectura requiere que el desarrollador garantice que todas las aplicaciones cliente usan un identificador de concesión y que solamente un cliente tiene un identificador de concesión válido en cada momento. Las operaciones de lectura que no incluyen un identificador de concesión, dan lugar a lecturas compartidas.  

En los siguientes ejemplos de código se muestra cómo adquirir una concesión exclusiva en un blob, actualizar el contenido del blob al proporcionar el identificador de la concesión y, a continuación, liberar la concesión. Si la concesión está activa y el identificador de concesión no se proporciona en una solicitud de escritura, se produce un error en la operación de escritura con el código de error 412 (error de condición previa).  

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Concurrency.cs" id="Snippet_DemonstratePessimisticConcurrencyBlob":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnetv11)

```csharp
public void DemonstratePessimisticConcurrencyBlob(string containerName, string blobName)
{
    Console.WriteLine("Demonstrate pessimistic concurrency");

    // Parse connection string and create container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExists();

    CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
    blockBlob.UploadText("Hello World!");
    Console.WriteLine("Blob added.");

    // Acquire lease for 15 seconds.
    string lease = blockBlob.AcquireLease(TimeSpan.FromSeconds(15), null);
    Console.WriteLine("Blob lease acquired. Lease = {0}", lease);

    // Update blob using lease. This operation should succeed.
    const string helloText = "Blob updated";
    var accessCondition = AccessCondition.GenerateLeaseCondition(lease);
    blockBlob.UploadText(helloText, accessCondition: accessCondition);
    Console.WriteLine("Blob updated using an exclusive lease");

    // Simulate another client attempting to update to blob without providing lease.
    try
    {
        // Operation will fail as no valid lease was provided.
        Console.WriteLine("Now try to update blob without valid lease.");
        blockBlob.UploadText("Update operation will fail without lease.");
    }
    catch (StorageException ex)
    {
        if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        {
            Console.WriteLine(@"Precondition failure error as expected.
                                Blob lease not provided.");
        }
        else
        {
            throw;
        }
    }

    // Release lease proactively.
    blockBlob.ReleaseLease(accessCondition);
    Console.WriteLine();
}
```

---

## <a name="pessimistic-concurrency-for-containers"></a>Simultaneidad pesimista para contenedores

Las concesiones en contenedores permiten las mismas estrategias de sincronización que se admiten para los blobs, incluidas las siguientes: escritura exclusiva/lectura compartida, escritura exclusiva/lectura exclusiva y escritura compartida/lectura exclusiva. Sin embargo, en el caso de los contenedores, el bloqueo exclusivo solo se aplica en las operaciones de eliminación. Para eliminar un contenedor con una concesión activa, un cliente debe incluir el identificador de concesión activo con la solicitud de eliminación. Todas las demás operaciones de contenedor se realizarán correctamente en un contenedor sujeto a una concesión sin necesidad de incluir el identificador de concesión.  

## <a name="next-steps"></a>Pasos siguientes

* [Especificación de encabezados condicionales para las operaciones de Blob service](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations)
* [Lease Container](/rest/api/storageservices/lease-container)
* [Concesión de blobs](/rest/api/storageservices/lease-blob)
