---
title: Desarrollo con Java para Azure Files | Microsoft Docs
description: Aprenda a desarrollar aplicaciones y servicios Java que usen Azure Files para almacenar datos de archivos.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2020
ms.custom: devx-track-java
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 115c55a5833906aa0dcc616a5b1b659468647282
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107814557"
---
# <a name="develop-for-azure-files-with-java"></a>Desarrollo con Java para Azure Files

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Aprenda los conceptos básicos de desarrollar aplicaciones de Java que usen Azure Files para almacenar datos. Cree una aplicación de consola y aprenda las acciones básicas sobre el uso de las API de Azure Files:

- Crear y eliminar recursos compartidos de archivos de Azure
- Crear y eliminar directorios
- Enumerar los archivos y directorios de un recurso compartido de Azure File
- Cargar, descargar y eliminar un archivo

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="create-a-java-application"></a>Creación de una aplicación Java

Para compilar las muestras, se necesitará el Kit de desarrollo de Java (JDK) y el [SDK de Azure Storage para Java](https://github.com/azure/azure-sdk-for-java). También deberá haber creado una cuenta de almacenamiento de Azure.

## <a name="set-up-your-application-to-use-azure-files"></a>Configuración de la aplicación para usar Azure Files

Para utilizar las API de Azure Files, agregue el siguiente código al principio del archivo Java desde el que desea acceder a Azure Files.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_ImportStatements":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

```java
// Include the following imports to use Azure Files APIs v11
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.file.*;
```

---

## <a name="set-up-an-azure-storage-connection-string"></a>Configuración de una cadena de conexión de Almacenamiento de Azure

Para usar Azure Files, debe conectarse a su cuenta de almacenamiento de Azure. Configure una cadena de conexión y úsela para conectarse a su cuenta de almacenamiento. Defina una variable estática que contenga la cadena de conexión.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

Reemplace *\<storage_account_name\>* y *\<storage_account_key\>* por los valores reales de la cuenta de almacenamiento.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_ConnectionString":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

Reemplace *your_storage_account_name* y *your_storage_account_key* por los valores reales de la cuenta de almacenamiento.

```java
// Configure the connection-string with your values
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account_name;" +
    "AccountKey=your_storage_account_key";
```

---

## <a name="access-an-azure-file-share"></a>Acceso a un recurso compartido de archivos de Azure

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

Para acceder a Azure Files, cree un objeto [ShareClient](/java/api/com.azure.storage.file.share.shareclient). Use la clase [ShareClientBuilder](/java/api/com.azure.storage.file.share.shareclientbuilder) para crear un nuevo objeto **ShareClient**.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createClient":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

Para acceder a su cuenta de almacenamiento, use el objeto **CloudStorageAccount**, pasando la cadena de conexión a su método de **análisis**.

```java
// Use the CloudStorageAccount object to connect to your storage account
try {
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
} catch (InvalidKeyException invalidKey) {
    // Handle the exception
}
```

**CloudStorageAccount.parse** produce una InvalidKeyException, por lo que deberá colocarla dentro de un bloque try/catch.

---

## <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos

Todos los archivos y directorios de Azure Files se almacenan en un contenedor denominado recurso compartido.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

El método [ShareClient.create](/java/api/com.azure.storage.file.share.shareclient.create) produce una excepción si el recurso compartido ya existe. Coloque la llamada a **create** en un bloque `try/catch` y controle la excepción.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createFileShare":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

Para acceder a un recurso compartido y su contenido, cree un cliente de Azure Files.

```java
// Create the Azure Files client.
CloudFileClient fileClient = storageAccount.createCloudFileClient();
```

Se usa el cliente de Azure Files para obtener una referencia a un recurso compartido.

```java
// Get a reference to the file share
CloudFileShare share = fileClient.getShareReference("sampleshare");
```

Para crear el recurso compartido, utilice el método **createIfNotExists** del objeto **CloudFileShare**.

```java
if (share.createIfNotExists()) {
    System.out.println("New share created");
}
```

En este punto, **share** contiene una referencia a un recurso compartido denominado **sample share**.

---

## <a name="delete-a-file-share"></a>Eliminación de un recurso compartido de archivos

En el siguiente código de ejemplo se elimina un recurso compartido de archivos.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

Elimine un recurso compartido llamando al método [ShareClient.delete](/java/api/com.azure.storage.file.share.shareclient.delete).

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteFileShare":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

Para eliminar un recurso compartido, llame al método **deleteIfExists** en un objeto **CloudFileShare**.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    // Create the file client.
   CloudFileClient fileClient = storageAccount.createCloudFileClient();

   // Get a reference to the file share
   CloudFileShare share = fileClient.getShareReference("sampleshare");

   if (share.deleteIfExists()) {
       System.out.println("sampleshare deleted");
   }
} catch (Exception e) {
    e.printStackTrace();
}
```

---

## <a name="create-a-directory"></a>Creación de un directorio

Para organizar el almacenamiento, coloque los archivos en los subdirectorios, en lugar de mantenerlos todos en el directorio raíz.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

En el código siguiente se crea un directorio llamando a [ShareDirectoryClient.create](/java/api/com.azure.storage.file.share.sharedirectoryclient.create). El método de ejemplo devuelve un valor `Boolean` que indica si se el directorio creó correctamente.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createDirectory":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

En el código siguiente se crea un subdirectorio denominado **sampledir** en el directorio raíz.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the sampledir directory
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

if (sampleDir.createIfNotExists()) {
    System.out.println("sampledir created");
} else {
    System.out.println("sampledir already exists");
}
```

---

## <a name="delete-a-directory"></a>Eliminación de un directorio

Eliminar un directorio es una tarea sencilla. No se puede eliminar un directorio que todavía contenga archivos o subdirectorios.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

El método [ShareDirectoryClient.delete](/java/api/com.azure.storage.file.share.sharedirectoryclient.delete) produce una excepción si el directorio no existe o no está vacío. Coloque la llamada a **delete** en un bloque `try/catch` y controle la excepción.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteDirectory":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory you want to delete
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

// Delete the directory
if ( containerDir.deleteIfExists() ) {
    System.out.println("Directory deleted");
}
```

---

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Enumerar los archivos y directorios de un recurso compartido de Azure File

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

Obtenga una lista de archivos y directorios llamando [ShareDirectoryClient.listFilesAndDirectories](/java/api/com.azure.storage.file.share.sharedirectoryclient.listfilesanddirectories). El método devuelve una lista de objetos [ShareFileItem](/java/api/com.azure.storage.file.share.models.sharefileitem) en los que puede efectuar la iteración. En el código siguiente se enumeran los archivos y directorios del directorio especificado por el parámetro *dirName*.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_enumerateFilesAndDirs":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

Obtenga una lista de archivos y directorios mediante una llamada a **listFilesAndDirectories** en una referencia de **CloudFileDirectory**. El método devuelve una lista de objetos **ListFileItem** en los que puede efectuar la iteración. El siguiente código enumera archivos y directorios dentro del directorio raíz.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
    System.out.println(fileItem.getUri());
}
```

---

## <a name="upload-a-file"></a>Cargar un archivo

Obtenga información sobre cómo cargar un archivo desde el almacenamiento local.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

El código siguiente carga un archivo local en el almacenamiento de Azure File Storage mediante una llamada al método [ShareFileClient.uploadFromFile](/java/api/com.azure.storage.file.share.sharefileclient.uploadfromfile). El método de ejemplo siguiente devuelve un valor `Boolean` que indica si el archivo especificado se cargó correctamente.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_uploadFile":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

Obtenga una referencia al directorio en el que se cargará el archivo mediante una llamada al método **getRootDirectoryReference** en el objeto del recurso compartido.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();
```

Ahora que tiene una referencia al directorio raíz del recurso compartido, puede cargar un archivo en él utilizando el código siguiente.

```java
// Define the path to a local file.
final String filePath = "C:\\temp\\Readme.txt";

CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
cloudFile.uploadFromFile(filePath);
```

---

## <a name="download-a-file"></a>Descarga de un archivo

Una de las operaciones más frecuentes es descargar archivos de un recurso compartido de archivos de Azure.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

En el siguiente ejemplo se descarga el archivo especificado en el directorio local especificado en el parámetro *destDir*. El método de ejemplo hace que el nombre de archivo descargado sea único al anteponer la fecha y la hora.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_downloadFile":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

En el ejemplo siguiente, se descarga el archivo SampleFile.txt y se muestra su contenido.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the directory that contains the file
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

//Get a reference to the file you want to download
CloudFile file = sampleDir.getFileReference("SampleFile.txt");

//Write the contents of the file to the console.
System.out.println(file.downloadText());
```

---

## <a name="delete-a-file"></a>Eliminación de un archivo

Otra operación habitual en Azure Files es la eliminación de archivos.

# <a name="azure-java-sdk-v12"></a>[SDK de Java de Azure v12](#tab/java)

En el código siguiente se elimina el archivo especificado. En primer lugar, el ejemplo crea un método [ShareDirectoryClient](/java/api/com.azure.storage.file.share.sharedirectoryclient) basándose en el parámetro *dirName*. A continuación, el código obtiene un [ShareFileClient](/java/api/com.azure.storage.file.share.sharefileclient) desde el cliente de directorio, basado en el parámetro *fileName*. Por último, el método de ejemplo llama a [ShareFileClient.delete](/java/api/com.azure.storage.file.share.sharefileclient.delete) para eliminar el archivo.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteFile":::

# <a name="azure-java-sdk-v11"></a>[SDK de Java de Azure v11](#tab/java11)

El código siguiente elimina un archivo denominado SampleFile.txt que se almacena en un directorio denominado **sampledir**.

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory where the file to be deleted is in
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

String filename = "SampleFile.txt"
CloudFile file;

file = containerDir.getFileReference(filename)
if ( file.deleteIfExists() ) {
    System.out.println(filename + " was deleted");
}
```

---

## <a name="next-steps"></a>Pasos siguientes

Si desea obtener más información acerca de otras API de almacenamiento de Azure, siga estos vínculos.

- [Azure para desarrolladores de Java](/azure/developer/java)
- [SDK de Azure para Java](https://github.com/azure/azure-sdk-for-java)
- [SDK de Azure para Android](https://github.com/azure/azure-sdk-for-android)
- [Referencia del SDK de la biblioteca cliente de recursos compartidos de archivos de Azure para Java](/java/api/overview/azure/storage-file-share-readme)
- [API de REST de servicios de Azure Storage](/rest/api/storageservices/)
- [Blog del equipo de Azure Storage](https://azure.microsoft.com/blog/topics/storage-backup-and-recovery/)
- [Transferencia de datos con la utilidad en línea de comandos AzCopy](../common/storage-use-azcopy-v10.md)
- [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)
