---
title: Carga de grandes cantidades de datos aleatorios en paralelo en Azure Storage
description: Aprenda a usar la biblioteca cliente de Azure Storage para cargar grandes cantidades de datos aleatorios en paralelo en una cuenta de Azure Storage
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: rogarana
ms.subservice: blobs
ms.openlocfilehash: a23836ebadace19c23802a0897287932892657a0
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459672"
---
# <a name="upload-large-amounts-of-random-data-in-parallel-to-azure-storage"></a>Cargar grandes cantidades de datos aleatorios en paralelo en Azure Storage

Este tutorial es la segunda parte de una serie. En este tutorial se muestra cómo implementar una aplicación que cargue grandes cantidades de datos aleatorios en una cuenta de Azure Storage.

En la segunda parte de la serie, se aprende a:

> [!div class="checklist"]
> * Configurar la cadena de conexión
> * Compilar la aplicación
> * Ejecución de la aplicación
> * Validar el número de conexiones

Microsoft Azure Blob Storage proporciona un servicio escalable para almacenar los datos. Para asegurarse de que la aplicación tenga el máximo rendimiento posible, se recomienda comprender el funcionamiento de Blob Storage. Es importante conocer los límites de los blobs de Azure. Para más información sobre estos límites, visite: [Objetivos de escalabilidad y rendimiento de Blob Storage](../blobs/scalability-targets.md).

La [nomenclatura de las particiones](../blobs/storage-performance-checklist.md#partitioning) es otro factor posiblemente importante al diseñar una aplicación de alto rendimiento con blobs. En el caso de tamaños de bloques mayores o iguales que 4 MiB, se usan los [blobs en bloques de alto rendimiento](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/) y la nomenclatura de las particiones no afecta al rendimiento. En el caso de tamaños de bloques menores de 4 MiB, Azure Storage usa un esquema de partición basado en intervalo para escalar y equilibrar la carga. Esta configuración significa que los archivos con prefijos o convenciones de nomenclatura similares van a la misma partición. Esta lógica incluye el nombre del contenedor donde se cargan los archivos. En este tutorial, usará archivos que tienen GUID de nombres, así como contenido generado de manera aleatoria. Estos se cargan posteriormente en cinco contenedores diferentes con nombres aleatorios.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, debe haber completado el tutorial anterior sobre el almacenamiento: [Creación de una máquina virtual y una cuenta de almacenamiento para una aplicación escalable][previous-tutorial].

## <a name="remote-into-your-virtual-machine"></a>Sesión remota en la máquina virtual

Ejecute el comando siguiente en el equipo local para crear una sesión de escritorio remoto con la máquina virtual. Reemplace la dirección IP con el valor de publicIPAddress de la máquina virtual. Cuando se le solicite, escriba las credenciales usadas al crear la máquina virtual.

```console
mstsc /v:<publicIpAddress>
```

## <a name="configure-the-connection-string"></a>Configurar la cadena de conexión

En Azure Portal, vaya a la cuenta de almacenamiento. Seleccione **Claves de acceso** en **Configuración**, en su cuenta de almacenamiento. Copie la **cadena de conexión** de la clave principal o secundaria. Inicie sesión en la máquina virtual creada en el tutorial anterior. Abra un **símbolo del sistema** como administrador y ejecute el comando `setx` con el conmutador `/m`. Este comando guarda una variable de entorno de configuración de la máquina. La variable de entorno no está disponible hasta que se vuelve a cargar el **símbolo del sistema**. Reemplace **\<storageConnectionString\>** en el ejemplo siguiente:

```console
setx storageconnectionstring "<storageConnectionString>" /m
```

Cuando termine, abra otro **símbolo del sistema**, vaya a `D:\git\storage-dotnet-perf-scale-app` y escriba `dotnet build` para compilar la aplicación.

## <a name="run-the-application"></a>Ejecución de la aplicación

Vaya a `D:\git\storage-dotnet-perf-scale-app`.

Escriba `dotnet run` para ejecutar la aplicación. La primera vez que se ejecuta `dotnet`, rellena la caché de paquetes local, para mejorar la velocidad de restauración y habilitar el acceso sin conexión. Este comando tarda un minuto como máximo en ejecutarse y solo lo hace una vez.

```console
dotnet run
```

La aplicación crea cinco contenedores con nombres aleatorios y comienza a cargar los archivos del directorio de almacenamiento provisional en la cuenta de almacenamiento.

El método `UploadFilesAsync` se muestra en el ejemplo siguiente:

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Scalable.cs" id="Snippet_UploadFilesAsync":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Los números mínimo y máximo de subprocesos se establecen en 100 para garantizar que se permite un gran número de conexiones simultáneas.

```csharp
private static async Task UploadFilesAsync()
{
    // Create five randomly named containers to store the uploaded files.
    CloudBlobContainer[] containers = await GetRandomContainersAsync();

    var currentdir = System.IO.Directory.GetCurrentDirectory();

    // Path to the directory to upload
    string uploadPath = currentdir + "\\upload";

    // Start a timer to measure how long it takes to upload all the files.
    Stopwatch time = Stopwatch.StartNew();

    try
    {
        Console.WriteLine("Iterating in directory: {0}", uploadPath);

        int count = 0;
        int max_outstanding = 100;
        int completed_count = 0;

        // Define the BlobRequestOptions on the upload.
        // This includes defining an exponential retry policy to ensure that failed connections
        // are retried with a back off policy. As multiple large files are being uploaded using
        // large block sizes, this can cause an issue if an exponential retry policy is not defined.
        // Additionally, parallel operations are enabled with a thread count of 8.
        // This should be a multiple of the number of processor cores in the machine.
        // Lastly, MD5 hash validation is disabled for this example, improving the upload speed.
        BlobRequestOptions options = new BlobRequestOptions
        {
            ParallelOperationThreadCount = 8,
            DisableContentMD5Validation = true,
            StoreBlobContentMD5 = false
        };

        // Create a new instance of the SemaphoreSlim class to 
        // define the number of threads to use in the application.
        SemaphoreSlim sem = new SemaphoreSlim(max_outstanding, max_outstanding);

        List<Task> tasks = new List<Task>();
        Console.WriteLine("Found {0} file(s)", Directory.GetFiles(uploadPath).Count());

        // Iterate through the files
        foreach (string path in Directory.GetFiles(uploadPath))
        {
            var container = containers[count % 5];
            string fileName = Path.GetFileName(path);
            Console.WriteLine("Uploading {0} to container {1}", path, container.Name);
            CloudBlockBlob blockBlob = container.GetBlockBlobReference(fileName);

            // Set the block size to 100MB.
            blockBlob.StreamWriteSizeInBytes = 100 * 1024 * 1024;

            await sem.WaitAsync();

            // Create a task for each file to upload. The tasks are
            // added to a collection and all run asynchronously.
            tasks.Add(blockBlob.UploadFromFileAsync(path, null, options, null).ContinueWith((t) =>
            {
                sem.Release();
                Interlocked.Increment(ref completed_count);
            }));

            count++;
        }

        // Run all the tasks asynchronously.
        await Task.WhenAll(tasks);

        time.Stop();

        Console.WriteLine("Upload has been completed in {0} seconds. Press any key to continue", time.Elapsed.TotalSeconds.ToString());

        Console.ReadLine();
    }
    catch (DirectoryNotFoundException ex)
    {
        Console.WriteLine("Error parsing files in the directory: {0}", ex.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```
Además de establecer la configuración del límite de subprocesos y de conexiones, la clase [BlobRequestOptions](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions) del método [UploadFromStreamAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromstreamasync) está configurada para usar el paralelismo y deshabilitar la validación de hash MD5. Los archivos se cargan en bloques de 100 MB. Esta configuración proporciona un mejor rendimiento, pero puede aumentar el costo si se usa una red con un rendimiento deficiente, ya que si se produce un error, se reintenta todo el bloque de 100 MB.

|Propiedad|Valor|Descripción|
|---|---|---|
|[ParallelOperationThreadCount](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.paralleloperationthreadcount)| 8| La configuración divide el blob en bloques al realizar la carga. Para conseguir el máximo rendimiento, este valor debe ser 8 veces el número de núcleos. |
|[DisableContentMD5Validation](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.disablecontentmd5validation)| true| Esta propiedad deshabilita la función de comprobación del hash MD5 del contenido cargado. Al deshabilitar la validación de MD5, se agiliza la transferencia. Aún así, hay que tener en cuenta que no se puede confirmar la validez o integridad de los archivos que se transfieren.   |
|[StoreBlobContentMD5](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.storeblobcontentmd5)| false| Esta propiedad determina si se calcula y almacena un hash MD5 con el archivo.   |
| [RetryPolicy](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.retrypolicy)| Retroceso de 2 segundos con 10 reintentos como máximo |Determina la directiva de reintentos de las solicitudes. Los errores de conexión se reintentan y, en este ejemplo, una directiva [ExponentialRetry](/dotnet/api/microsoft.azure.batch.common.exponentialretry) está configurada con un retroceso de 2 segundos y un número máximo de reintentos de 10. Esta configuración es importante cuando la aplicación se acerca a los objetivos de escalabilidad y rendimiento de Blob Storage. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Blob Storage](../blobs/scalability-targets.md).  |

---

El siguiente ejemplo es una salida de aplicación truncada que se ejecuta en un sistema Windows.

```console
Created container 2dbb45f4-099e-49eb-880c-5b02ebac135e
Created container 0d784365-3bdf-4ef2-b2b2-c17b6480792b
Created container 42ac67f2-a316-49c9-8fdb-860fb32845d7
Created container f0357772-cb04-45c3-b6ad-ff9b7a5ee467
Created container 92480da9-f695-4a42-abe8-fb35e71eb887
Iterating in directory: C:\git\myapp\upload
Found 5 file(s)
Uploading 1d596d16-f6de-4c4c-8058-50ebd8141e4d.pdf to container 2dbb45f4-099e-49eb-880c-5b02ebac135e
Uploading 242ff392-78be-41fb-b9d4-aee8152a6279.pdf to container 0d784365-3bdf-4ef2-b2b2-c17b6480792b
Uploading 38d4d7e2-acb4-4efc-ba39-f9611d0d55ef.pdf to container 42ac67f2-a316-49c9-8fdb-860fb32845d7
Uploading 45930d63-b0d0-425f-a766-cda27ff00d32.pdf to container f0357772-cb04-45c3-b6ad-ff9b7a5ee467
Uploading 5129b385-5781-43be-8bac-e2fbb7d2bd82.pdf to container 92480da9-f695-4a42-abe8-fb35e71eb887
Uploaded 5 files in 16.9552163 seconds
```

### <a name="validate-the-connections"></a>Validar las conexiones

Mientras se cargan los archivos, puede comprobar el número de conexiones simultáneas de su cuenta de almacenamiento. Abra una ventana de consola y escriba `netstat -a | find /c "blob:https"`. Este comando muestra el número de conexiones abiertas actualmente. Como puede ver en el ejemplo siguiente, más de 800 conexiones estaban abiertas al cargar los archivos aleatorios en la cuenta de almacenamiento. Este valor cambia durante la ejecución de la carga. Al realizar la carga en fragmentos de bloques paralelos, se reduce enormemente la cantidad de tiempo necesaria para transferir el contenido.

```
C:\>netstat -a | find /c "blob:https"
800

C:\>
```

## <a name="next-steps"></a>Pasos siguientes

En la segunda parte de la serie aprendió no solo cómo cargar grandes cantidades de datos aleatorios en una cuenta de almacenamiento en paralelo, si no que también aprendió lo siguiente:

> [!div class="checklist"]
> * Configurar la cadena de conexión
> * Compilar la aplicación
> * Ejecución de la aplicación
> * Validar el número de conexiones

Continúe en la tercera parte de la serie para descargar grandes cantidades de datos de una cuenta de almacenamiento.

> [!div class="nextstepaction"]
> [Descarga de grandes cantidades de datos aleatorios desde Azure Storage](storage-blob-scalable-app-download-files.md)

[previous-tutorial]: storage-blob-scalable-app-create-vm.md
