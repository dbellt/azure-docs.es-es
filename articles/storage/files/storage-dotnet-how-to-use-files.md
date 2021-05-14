---
title: Desarrollo para Azure Files con .NET | Microsoft Docs
description: Aprenda a desarrollar aplicaciones y servicios .NET que usen Azure Files para almacenar datos.
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 10/02/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-csharp
ms.openlocfilehash: 2c00f001ae3cba9420a137a42f9f696619584d50
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107817384"
---
# <a name="develop-for-azure-files-with-net"></a>Desarrollo para Azure Files con .NET

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Aprenda los conceptos básicos de desarrollar aplicaciones .NET que usen [Azure Files](storage-files-introduction.md) para almacenar datos. En este artículo se muestra cómo crear una aplicación de consola simple para hacer lo siguiente con .NET y Azure Files:

- Obtenga el contenido de un archivo.
- Establezca la cuota o tamaño máximo para un recurso compartido de archivos.
- Cree una firma de acceso compartido (SAS) para un archivo.
- Copie un archivo en otro en la misma cuenta de almacenamiento.
- Copie un archivo en un blob en la misma cuenta de almacenamiento.
- Cree una instantánea de un recurso compartido de archivos.
- Restaure un archivo desde una instantánea de recurso compartido.
- Use las métricas de Azure Storage para solucionar problemas.

Para obtener más información acerca de Azure Files, consulte [¿Qué es Azure Files?](storage-files-introduction.md).

[!INCLUDE [storage-check-out-samples-dotnet](../../../includes/storage-check-out-samples-dotnet.md)]

## <a name="understanding-the-net-apis"></a>Descripción de las API de .NET

Azure Files proporciona dos enfoques generales para las aplicaciones cliente: bloque de mensajes del servidor (SMB) y REST. Dentro de. NET, las API `System.IO` y `Azure.Storage.Files.Shares` extraen estos métodos.

API | Cuándo se usa | Notas
----|-------------|------
[System.IO](/dotnet/api/system.io) | Su aplicación: <ul><li>Es necesario leer o escribir archivos mediante SMB</li><li>Se ejecuta en un dispositivo que tenga acceso a través del puerto 445 a su cuenta de Azure Files</li><li>No es necesario administrar cualquiera de las opciones administrativas del recurso compartido de archivos</li></ul> | La E/S de archivos de Azure Files a través de SMB normalmente es igual que la E/S con cualquier recurso compartido de red o dispositivo de almacenamiento local. Para obtener una introducción a una serie de características en .NET, incluida la E/S de archivos, consulte [Aplicación de consola](/dotnet/csharp/tutorials/console-teleprompter).
[Azure.Storage.Files.Shares](/dotnet/api/azure.storage.files.shares) | Su aplicación: <ul><li>No se puede tener acceso a Azure Files mediante SMB en el puerto 445 debido a restricciones de ISP o firewall</li><li>Requiere funcionalidad administrativa, como la capacidad de establecer la cuota de un recurso compartido de archivo o crear una firma de acceso compartido</li></ul> | Este artículo muestra el uso de `Azure.Storage.Files.Shares` para la E/S de archivos con REST (en lugar de SMB) y la administración del recurso compartido de archivos.

## <a name="create-the-console-application-and-obtain-the-assembly"></a>Creación de la aplicación de consola y obtención del ensamblado

Puede usar la biblioteca cliente de Azure Files en cualquier tipo de aplicación .NET. Estas aplicaciones incluyen la nube de Azure, la Web, el escritorio y aplicaciones móviles. En esta guía, creamos una aplicación de consola para hacerlo más sencillo.

En Visual Studio, cree una nueva aplicación de consola de Windows. Los siguientes pasos muestran cómo crear una aplicación de consola en Visual Studio 2019. Los pasos son similares en otras versiones de Visual Studio.

1. Inicie Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, elija **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
1. En **Configure su nuevo proyecto**, escriba un nombre para la app y seleccione **Crear**.

Agregue todos los ejemplos de código en este artículo a la clase `Program` en el archivo *Program.cs*.

## <a name="use-nuget-to-install-the-required-packages"></a>Uso de NuGet para instalar los paquetes necesarios

Consulte estos paquetes en el proyecto:

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

- [Biblioteca de Azure Core para .NET](https://www.nuget.org/packages/Azure.Core/): este paquete es la implementación de la canalización de clientes de Azure.
- [Biblioteca cliente de blob de Azure Storage para .NET](https://www.nuget.org/packages/Azure.Storage.Blobs/): Este paquete proporciona acceso mediante programación a los recursos de blob de la cuenta de almacenamiento.
- [Biblioteca cliente de archivos de Azure Storage para .NET](https://www.nuget.org/packages/Azure.Storage.Files.Shares/): Este paquete proporciona acceso mediante programación a los recursos de archivo de la cuenta de almacenamiento.
- [Biblioteca de Configuration Manager del sistema para .NET](https://www.nuget.org/packages/System.Configuration.ConfigurationManager/): este paquete proporciona una clase que almacena y recupera valores en un archivo de configuración.

Puede usar NuGet para obtener los paquetes. Siga estos pasos:

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes de NuGet**.
1. En **Administrador de paquetes NuGet**, seleccione **Examinar**. A continuación, busque y elija **Azure.Core**, y, después, seleccione **Instalar**.

   En este paso se instala el paquete y sus dependencias.

1. Busque e instale estos paquetes:

   - **Azure.Storage.Blobs**
   - **Azure.Storage.Files.Shares**
   - **System.Configuration.ConfigurationManager**

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

- [Biblioteca común de Microsoft Azure Storage para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/): Este paquete proporciona acceso mediante programación a los recursos comunes de la cuenta de almacenamiento.
- [Biblioteca de Microsoft Azure Storage Blob para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/): Este paquete proporciona acceso mediante programación a los recursos de blob de la cuenta de almacenamiento.
- [Biblioteca de archivos de Microsoft Azure Storage para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/): Este paquete proporciona acceso mediante programación a los recursos de archivo de la cuenta de almacenamiento.
- [Biblioteca del Administrador de configuración de Microsoft Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/): Este paquete proporciona una clase para analizar una cadena de conexión en un archivo de configuración, independientemente de dónde se ejecute la aplicación.

Puede usar NuGet para obtener los paquetes. Siga estos pasos:

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes de NuGet**.
1. En **Administrador de paquetes NuGet**, seleccione **Examinar**. Después, busque y elija **Microsoft.Azure.Storage.Blob** y, a continuación, seleccione **Instalar**.

   En este paso se instala el paquete y sus dependencias.
1. Busque e instale estos paquetes:

   - **Microsoft.Azure.Storage.Common**
   - **Microsoft.Azure.Storage.File**
   - **Microsoft.Azure.ConfigurationManager**

---

## <a name="save-your-storage-account-credentials-to-the-appconfig-file"></a>Guardar las credenciales de la cuenta de almacenamiento en el archivo app.config

A continuación, guardará las credenciales en el archivo *App.config* del proyecto. En el **Explorador de soluciones**, haga doble clic en `App.config` y edite el archivo para que sea similar al ejemplo siguiente.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

Reemplace `myaccount` por el nombre de la cuenta de almacenamiento y `mykey` por la clave de la cuenta de almacenamiento.

:::code language="xml" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/app.config" highlight="5,6,7":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

Reemplace `myaccount` por el nombre de la cuenta de almacenamiento y `StorageAccountKeyEndingIn==` por la clave de la cuenta de almacenamiento.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
  </startup>
  <appSettings>
    <add key="StorageConnectionString"
      value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==" />
  </appSettings>
</configuration>
```

---

> [!NOTE]
> El emulador de almacenamiento de Azurite no admite Azure Files actualmente. La cadena de conexión debe hacer referencia a una cuenta de Azure Storage en la nube para trabajar con Azure Files.

## <a name="add-using-directives"></a>Adición de directivas using

En el **Explorador de soluciones**, abra el archivo *Program.cs* y agregue lo siguiente mediante directivas al principio del archivo.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_UsingStatements":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
using Microsoft.Azure; // Namespace for Azure Configuration Manager
using Microsoft.Azure.Storage; // Namespace for Storage Client Library
using Microsoft.Azure.Storage.Blob; // Namespace for Azure Blobs
using Microsoft.Azure.Storage.File; // Namespace for Azure Files
```

[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

---

## <a name="access-the-file-share-programmatically"></a>Obtener acceso al recurso compartido de archivos mediante programación

En el archivo *Program.cs*, agregue el código siguiente para obtener acceso al recurso compartido de archivos mediante programación.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

El método siguiente crea un recurso compartido de archivos si aún no existe. El método se inicia creando un objeto [ShareClient](/dotnet/api/azure.storage.files.shares.shareclient) a partir de una cadena de conexión. A continuación, en el ejemplo se intenta descargar un archivo que creamos anteriormente. Llame a este método desde `Main()`.

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CreateShare":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

A continuación, agregue el siguiente contenido al método `Main()` (después del código mostrado anteriormente) para recuperar la cadena de conexión. Este código obtiene una referencia al archivo que creamos anteriormente y envía su contenido.

```csharp
// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile file = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the file exists.
        if (file.Exists())
        {
            // Write the contents of the file to the console window.
            Console.WriteLine(file.DownloadTextAsync().Result);
        }
    }
}
```

Ejecute la aplicación de consola para ver la salida.

---

## <a name="set-the-maximum-size-for-a-file-share"></a>Establecer el tamaño máximo para un recurso compartido de archivos

A partir de la versión 5.x de la biblioteca cliente de Azure Files, se puede establecer la cuota (tamaño máximo) de un recurso compartido de archivos. También puede comprobar cuántos datos se almacenan actualmente en el recurso compartido.

Al establecer la cuota para un recurso compartido, se limita el tamaño total de los archivos almacenados en el recurso compartido. Si el tamaño total de los archivos del recurso compartido supera la cuota, los clientes no pueden aumentar el tamaño de los archivos existentes. Los clientes tampoco pueden crear archivos nuevos, a menos que estén vacíos.

En el ejemplo siguiente se muestra cómo comprobar el uso actual de un recurso compartido y cómo establecer la cuota para el recurso compartido.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_SetMaxShareSize":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Check current usage stats for the share.
    // Note that the ShareStats object is part of the protocol layer for the File service.
    Microsoft.Azure.Storage.File.Protocol.ShareStats stats = share.GetStats();
    Console.WriteLine("Current share usage: {0} GB", stats.Usage.ToString());

    // Specify the maximum size of the share, in GB.
    // This line sets the quota to be 10 GB greater than the current usage of the share.
    share.Properties.Quota = 10 + stats.Usage;
    share.SetProperties();

    // Now check the quota for the share. Call FetchAttributes() to populate the share's properties.
    share.FetchAttributes();
    Console.WriteLine("Current share quota: {0} GB", share.Properties.Quota);
}
```

---

### <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>Generar una firma de acceso compartido para un archivo o recurso compartido de archivos

A partir de la versión 5.x de la biblioteca cliente de Azure Files, puede generar una firma de acceso compartido (SAS) para un recurso compartido de archivos o para un archivo individual.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

El método de ejemplo siguiente devuelve una SAS en un archivo del recurso compartido especificado.

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_GetFileSasUri":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

También puede crear una directiva de acceso almacenada en un recurso compartido de archivos para administrar firmas de acceso compartido. Le recomendamos que cree una directiva de acceso almacenada, ya que le permite revocar la clave SAS si se encuentra en peligro. En el ejemplo siguiente se crea una directiva de acceso almacenada en un recurso compartido. En el ejemplo se usa esa directiva para proporcionar las restricciones para una clave SAS en un archivo del recurso compartido.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    string policyName = "sampleSharePolicy" + DateTime.UtcNow.Ticks;

    // Create a new stored access policy and define its constraints.
    SharedAccessFilePolicy sharedPolicy = new SharedAccessFilePolicy()
        {
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessFilePermissions.Read | SharedAccessFilePermissions.Write
        };

    // Get existing permissions for the share.
    FileSharePermissions permissions = share.GetPermissions();

    // Add the stored access policy to the share's policies. Note that each policy must have a unique name.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    share.SetPermissions(permissions);

    // Generate a SAS for a file in the share and associate this access policy with it.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");
    CloudFile file = sampleDir.GetFileReference("Log1.txt");
    string sasToken = file.GetSharedAccessSignature(null, policyName);
    Uri fileSasUri = new Uri(file.StorageUri.PrimaryUri.ToString() + sasToken);

    // Create a new CloudFile object from the SAS, and write some text to the file.
    CloudFile fileSas = new CloudFile(fileSasUri);
    fileSas.UploadText("This write operation is authorized via SAS.");
    Console.WriteLine(fileSas.DownloadText());
}
```

---

Para más información sobre la creación y el uso de firmas de acceso compartido, consulte [Funcionamiento de una firma de acceso compartido](../common/storage-sas-overview.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#how-a-shared-access-signature-works).

## <a name="copy-files"></a>Copiar archivos

A partir de la versión 5.x de la biblioteca cliente de Azure Files, puede copiar un archivo en otro, un archivo en un blob o un blob en un archivo.

También puede usar AzCopy para copiar un archivo en otro o para copiar un blob en un archivo o viceversa. Consulte [Introducción a AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

> [!NOTE]
> Si va a copiar un blob en un archivo o un archivo en un blob, debe usar una firma de acceso compartido (SAS) para autorizar el acceso al objeto de origen, incluso si está copiando en la misma cuenta de almacenamiento.

### <a name="copy-a-file-to-another-file"></a>Copiar un archivo en otro

En el ejemplo siguiente se copia un archivo en otro en el mismo recurso compartido. Puede usar la [autenticación de clave compartida](/rest/api/storageservices/authorize-with-shared-key) para realizar la copia porque en esta operación se copian archivos de la misma cuenta de almacenamiento.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CopyFile":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile sourceFile = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the source file exists.
        if (sourceFile.Exists())
        {
            // Get a reference to the destination file.
            CloudFile destFile = sampleDir.GetFileReference("Log1Copy.txt");

            // Start the copy operation.
            destFile.StartCopy(sourceFile);

            // Write the contents of the destination file to the console window.
            Console.WriteLine(destFile.DownloadText());
        }
    }
}
```

---

### <a name="copy-a-file-to-a-blob"></a>Copiar un archivo en un blob

En el ejemplo siguiente se crea un archivo y se copia en un blob de la misma cuenta de almacenamiento. El ejemplo crea una SAS para el archivo de origen, que el servicio usa para autorizar el acceso al archivo de origen durante la operación de copia.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CopyFileToBlob":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Create a new file share, if it does not already exist.
CloudFileShare share = fileClient.GetShareReference("sample-share");
share.CreateIfNotExists();

// Create a new file in the root directory.
CloudFile sourceFile = share.GetRootDirectoryReference().GetFileReference("sample-file.txt");
sourceFile.UploadText("A sample file in the root directory.");

// Get a reference to the blob to which the file will be copied.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
container.CreateIfNotExists();
CloudBlockBlob destBlob = container.GetBlockBlobReference("sample-blob.txt");

// Create a SAS for the file that's valid for 24 hours.
// Note that when you are copying a file to a blob, or a blob to a file, you must use a SAS
// to authorize access to the source object, even if you are copying within the same
// storage account.
string fileSas = sourceFile.GetSharedAccessSignature(new SharedAccessFilePolicy()
{
    // Only read permissions are required for the source file.
    Permissions = SharedAccessFilePermissions.Read,
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
});

// Construct the URI to the source file, including the SAS token.
Uri fileSasUri = new Uri(sourceFile.StorageUri.PrimaryUri.ToString() + fileSas);

// Copy the file to the blob.
destBlob.StartCopy(fileSasUri);

// Write the contents of the file to the console window.
Console.WriteLine("Source file contents: {0}", sourceFile.DownloadText());
Console.WriteLine("Destination blob contents: {0}", destBlob.DownloadText());
```

---

Puede copiar un blob en un archivo de la misma manera. Si el objeto de origen es un blob, cree una SAS para autorizar el acceso a dicho blob durante la operación de copia.

## <a name="share-snapshots"></a>Instantáneas de recursos compartido

A partir de la versión 8.5 de la biblioteca cliente de Azure Files, se puede crear una instantánea de recurso compartido. Las instantáneas de recursos compartidos también se pueden enumerar, explorar y eliminar. Una vez creadas, las instantáneas de recursos compartidos son de solo lectura.

### <a name="create-share-snapshots"></a>Creación de instantáneas de recurso compartido

En el siguiente ejemplo se crea una instantánea de recurso compartido.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CreateShareSnapshot":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
storageAccount = CloudStorageAccount.Parse(ConnectionString); 
fClient = storageAccount.CreateCloudFileClient(); 
string baseShareName = "myazurefileshare"; 
CloudFileShare myShare = fClient.GetShareReference(baseShareName); 
var snapshotShare = myShare.Snapshot();

```

---

### <a name="list-share-snapshots"></a>Enumerar instantáneas del recurso compartido

En el ejemplo siguiente se enumeran las instantáneas de un recurso compartido.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_ListShareSnapshots":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
var shares = fClient.ListShares(baseShareName, ShareListingDetails.All);
```

---

### <a name="list-files-and-directories-within-share-snapshots"></a>Enumeración de archivos y directorios en instantáneas de recursos compartidos

En el ejemplo siguiente se examinan los archivos y directorios de instantáneas de recursos compartidos.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_ListSnapshotContents":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); 
var rootDirectory = mySnapshot.GetRootDirectoryReference(); 
var items = rootDirectory.ListFilesAndDirectories();
```

---

### <a name="restore-file-shares-or-files-from-share-snapshots"></a>Restauración de recursos compartidos de archivos o archivos desde instantáneas de recursos compartidos

La toma de una instantánea de un recurso compartido de archivos permite recuperar archivos individuales o todo el recurso compartido de archivos.

Para restaurar un archivo de una instantánea del recurso compartido de archivos, consulte las instantáneas de recursos compartidos de un recurso compartido de archivos. Después, puede recuperar un archivo que pertenezca a una instantánea de recurso compartido determinada. Use esa versión para leer directamente el archivo o restaurarlo.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_RestoreFileFromSnapshot":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
CloudFileShare liveShare = fClient.GetShareReference(baseShareName);
var rootDirOfliveShare = liveShare.GetRootDirectoryReference();
var dirInliveShare = rootDirOfliveShare.GetDirectoryReference(dirName);
var fileInliveShare = dirInliveShare.GetFileReference(fileName);

CloudFileShare snapshot = fClient.GetShareReference(baseShareName, snapshotTime);
var rootDirOfSnapshot = snapshot.GetRootDirectoryReference();
var dirInSnapshot = rootDirOfSnapshot.GetDirectoryReference(dirName);
var fileInSnapshot = dir1InSnapshot.GetFileReference(fileName);

string sasContainerToken = string.Empty;
SharedAccessFilePolicy sasConstraints = new SharedAccessFilePolicy();
sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
sasConstraints.Permissions = SharedAccessFilePermissions.Read;

//Generate the shared access signature on the container, setting the constraints directly on the signature.
sasContainerToken = fileInSnapshot.GetSharedAccessSignature(sasConstraints);

string sourceUri = (fileInSnapshot.Uri.ToString() + sasContainerToken + "&" + fileInSnapshot.SnapshotTime.ToString()); ;
fileInliveShare.StartCopyAsync(new Uri(sourceUri));
```

---

### <a name="delete-share-snapshots"></a>Eliminación de instantáneas de recursos compartidos

En el siguiente ejemplo se elimina una instantánea de recurso compartido.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_DeleteSnapshot":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); mySnapshot.Delete(null, null, null);
```

---

## <a name="troubleshoot-azure-files-by-using-metrics"></a>Solución de problemas de Azure Files mediante métricas<a name="troubleshooting-azure-files-using-metrics"></a>

Azure Storage Analytics admite métricas para Azure Files. Con los datos de las métricas, es posible seguir paso a paso las solicitudes y diagnosticar problemas.

Puede habilitar las métricas para Azure Files mediante [Azure Portal](https://portal.azure.com). La métrica también se puede habilitar mediante programación. Para ello, hay que llamar a la operación [Set File Service Properties](/rest/api/storageservices/set-file-service-properties) con la API de REST o una de sus análogas de la biblioteca cliente de Azure Files.

En el siguiente ejemplo de código se muestra cómo usar la biblioteca cliente de .NET a fin de habilitar las métricas para Azure Files.

# <a name="azure-net-sdk-v12"></a>[Azure \.NET SDK v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_UseMetrics":::

# <a name="azure-net-sdk-v11"></a>[Azure \.NET SDK v11](#tab/dotnetv11)

En primer lugar, agregue las siguientes directivas `using` a su archivo *Program.cs*, además de las que ya ha agregado:

```csharp
using Microsoft.Azure.Storage.File.Protocol;
using Microsoft.Azure.Storage.Shared.Protocol;
```

Aunque Azure Blobs, Azure Tables y Azure Queues utilizan el tipo compartido `ServiceProperties` en el espacio de nombres `Microsoft.Azure.Storage.Shared.Protocol`, Azure Files utiliza el suyo propio, el tipo `FileServiceProperties` en el espacio de nombres `Microsoft.Azure.Storage.File.Protocol`. No obstante, debe hacer referencia a ambos espacios de nombres en el código para que se pueda compilar.

```csharp
// Parse your storage connection string from your application's configuration file.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
// Create the File service client.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Set metrics properties for File service.
// Note that the File service currently uses its own service properties type,
// available in the Microsoft.Azure.Storage.File.Protocol namespace.
fileClient.SetServiceProperties(new FileServiceProperties()
{
    // Set hour metrics
    HourMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 14,
        Version = "1.0"
    },
    // Set minute metrics
    MinuteMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 7,
        Version = "1.0"
    }
});

// Read the metrics properties we just set.
FileServiceProperties serviceProperties = fileClient.GetServiceProperties();
Console.WriteLine("Hour metrics:");
Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
Console.WriteLine(serviceProperties.HourMetrics.Version);
Console.WriteLine();
Console.WriteLine("Minute metrics:");
Console.WriteLine(serviceProperties.MinuteMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.MinuteMetrics.RetentionDays);
Console.WriteLine(serviceProperties.MinuteMetrics.Version);
```

---

Si encuentra algún problema, puede consultar [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure Files, consulte los siguientes recursos:

### <a name="conceptual-articles-and-videos"></a>Artículos y vídeos conceptuales

- [Azure Files: un sistema de archivos SMB en la nube sin dificultades para Windows y Linux](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
- [Uso de Azure Files con Linux](storage-how-to-use-files-linux.md)

### <a name="tooling-support-for-file-storage"></a>Compatibilidad de herramientas con el almacenamiento de archivos

- [Introducción a AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)
- [Solucione problemas de Azure Files en Windows](./storage-troubleshoot-windows-file-connection-problems.md)

### <a name="reference"></a>Referencia

- [API de Azure Storage para .NET](/dotnet/api/overview/azure/storage)
- [File Service REST API](/rest/api/storageservices/File-Service-REST-API) (API de REST de File Service)