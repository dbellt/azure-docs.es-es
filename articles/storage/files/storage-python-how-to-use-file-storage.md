---
title: Desarrollo para Azure Files con Python | Microsoft Docs
description: Aprenda a desarrollar aplicaciones y servicios de Python que utilizan Azure Files para almacenar datos de archivos.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/08/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-python
ms.openlocfilehash: 8739bfaf1a41758ef3267c71cba883ef2445c39d
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107817799"
---
# <a name="develop-for-azure-files-with-python"></a>Desarrollo para Azure Files con Python

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Conozca los aspectos básicos del uso de Python para desarrollar aplicaciones o servicios que usan Azure Files para almacenar datos de archivos. Cree una aplicación de consola sencilla y aprenda a realizar acciones básicas con Python y Azure Files:

- Crear recursos compartidos de archivos de Azure
- Crear directorios
- Enumerar los archivos y directorios de un recurso compartido de Azure File
- Cargar, descargar y eliminar un archivo
- Crear copias de seguridad de recursos compartidos de archivos mediante instantáneas

> [!NOTE]
> Dado que se puede acceder a Azure Files a través de SMB, es posible escribir aplicaciones simples que accedan al recurso compartido de archivos de Azure mediante las clases y funciones estándar de E/S de Python. En este artículo se describe cómo escribir aplicaciones que usen el SDK de Python de Azure Files Storage, que emplea la [API de REST de Azure Files](/rest/api/storageservices/file-service-rest-api) para comunicarse con Azure Files.

## <a name="download-and-install-azure-storage-sdk-for-python"></a>Descarga e instalación del SDK de Azure Storage para Python

> [!NOTE]
> Si va a actualizar desde el SDK de Azure Storage para Python 0.36 o una versión anterior, desinstale el SDK anterior mediante `pip uninstall azure-storage` antes de instalar el paquete más reciente.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

La [biblioteca cliente v12.x de Azure File Storage para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-share) requiere la versión de Python 2.7 o 3.6.

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

El [SDK de Azure Storage para Python](https://github.com/azure/azure-storage-python) requiere Python 2.7 o 3.6.

---

## <a name="install-via-pypi"></a>Instalación mediante PyPI

Para realizar la instalación mediante el índice de paquetes de Python (PyPI), escriba:

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

```console
pip install azure-storage-file-share
```

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```console
pip install azure-storage-file
```

### <a name="view-the-sample-application"></a>Visualización de la aplicación de ejemplo

Para ver y ejecutar una aplicación de ejemplo que muestra cómo usar Python con Azure Files, consulte [Azure Storage: Getting Started with Azure Files in Python](https://github.com/Azure-Samples/storage-file-python-getting-started) (Azure Storage: Introducción a Azure Files en Python).

Para ejecutar la aplicación de ejemplo, asegúrese de que ha instalado los paquetes `azure-storage-file` y `azure-storage-common`.

---

## <a name="set-up-your-application-to-use-azure-files"></a>Configuración de la aplicación para usar Azure Files

Agregue los elementos siguientes cerca de la parte superior de un archivo de origen de Python para usar los fragmentos de código de este artículo.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_Imports":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```python
from azure.storage.file import FileService
```

---

## <a name="set-up-a-connection-to-azure-files"></a>Configuración de una conexión a Azure Files

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

[ShareServiceClient](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareserviceclient) le permite trabajar con recursos compartidos, directorios y archivos. El código siguiente crea un objeto `ShareServiceClient` con la cadena de conexión de la cuenta de almacenamiento.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateShareServiceClient":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

El objeto [FileService](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true) le permite trabajar con archivos, directorios y recursos compartidos. El código siguiente crea un objeto `FileService` mediante el nombre de la cuenta de almacenamiento y la clave de la cuenta. Reemplace `<myaccount>` y `<mykey>` por el nombre y la clave de la de cuenta.

```python
file_service = FileService(account_name='myaccount', account_key='mykey')
```

---

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

En el siguiente código de ejemplo, puede usar un objeto [ShareClient](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient) para crear el recurso compartido, en caso de que no exista.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateFileShare":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

En el siguiente ejemplo de código, puede usar un objeto [FileService](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true) para crear el recurso compartido si no existe.

```python
file_service.create_share('myshare')
```

---

## <a name="create-a-directory"></a>Creación de un directorio

Para organizar el almacenamiento, coloque los archivos en los subdirectorios, en lugar de mantenerlos todos en el directorio raíz.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

El método siguiente crea un directorio en la raíz del recurso compartido de archivos especificado utilizando un objeto [ShareDirectoryClient](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharedirectoryclient).

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateDirectory":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

El código siguiente creará un subdirectorio denominado *sampledir* en el directorio raíz.

```python
file_service.create_directory('myshare', 'sampledir')
```

---

## <a name="upload-a-file"></a>Carga de un archivo

En esta sección, aprenderá a cargar un archivo de almacenamiento local en Azure Files Storage.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

El método siguiente carga el contenido del archivo especificado en el directorio que está especificado en el recurso compartido de archivos de Azure especificado.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_UploadFile":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

Un recurso compartido de archivos de Azure contiene como mínimo un directorio raíz donde pueden residir los archivos. Para crear un archivo y cargar datos, use los métodos [create_file_from_path](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-path-share-name--directory-name--file-name--local-file-path--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object---timeout-none-), [create_file_from_stream](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-stream-share-name--directory-name--file-name--stream--count--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--), [create_file_from_bytes](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-bytes-share-name--directory-name--file-name--file--index-0--count-none--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--) o [create_file_from_text](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-text-share-name--directory-name--file-name--text--encoding--utf-8---content-settings-none--metadata-none--validate-content-false--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--). Se trata de métodos de alto nivel que realizan la fragmentación necesaria cuando el tamaño de los datos supera los 64 MB.

`create_file_from_path` carga el contenido de un archivo de la ruta de acceso especificada y `create_file_from_stream` carga el contenido de un archivo o secuencia ya abiertos. `create_file_from_bytes` carga una matriz de bytes y `create_file_from_text`carga el valor de texto especificado con la codificación especificada (el valor predeterminado es UTF-8).

En el siguiente ejemplo se carga el contenido del archivo *sunset.png* en el archivo **myfile**.

```python
from azure.storage.file import ContentSettings
file_service.create_file_from_path(
    'myshare',
    None,  # We want to create this file in the root directory, so we specify None for the directory_name
    'myfile',
    'sunset.png',
    content_settings=ContentSettings(content_type='image/png'))
```

---

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Enumerar los archivos y directorios de un recurso compartido de Azure File

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

Para enumerar los archivos y directorios de un subdirectorio, use el método [list_directories_and_files](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient#list-directories-and-files-directory-name-none--name-starts-with-none--marker-none----kwargs-). Este método devuelve un objeto iterable de paginación automática. El código siguiente genera el **nombre** de cada archivo y subdirectorio del directorio especificado en la consola.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_ListFilesAndDirs":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

Para mostrar los archivos y los directorios de un recurso compartido, use el método [list_directories_and_files](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#list-directories-and-files-share-name--directory-name-none--num-results-none--marker-none--timeout-none--prefix-none--snapshot-none-). Este método devuelve un generador. El código siguiente ofrece el **nombre** de todos los archivos y el directorio de un recurso compartido de la consola.

```python
generator = file_service.list_directories_and_files('myshare')
for file_or_dir in generator:
    print(file_or_dir.name)
```

---

## <a name="download-a-file"></a>Descarga de un archivo

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

Para descargar los datos de un archivo, use [download_file](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharefileclient#download-file-offset-none--length-none----kwargs-).

En el ejemplo siguiente se muestra cómo usar `download_file` para obtener el contenido del archivo especificado y almacenarlo localmente con el valor **DOWNLOADED-** antepuesto al nombre de archivo.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DownloadFile":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

Para descargar datos de un archivo, use [get_file_to_path](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-path-share-name--directory-name--file-name--file-path--open-mode--wb---start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-), [get_file_to_stream](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-stream-share-name--directory-name--file-name--stream--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-), [get_file_to_bytes](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-bytes-share-name--directory-name--file-name--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-) o [get_file_to_text](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-text-share-name--directory-name--file-name--encoding--utf-8---start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-). Se trata de métodos de alto nivel que realizan la fragmentación necesaria cuando el tamaño de los datos supera los 64 MB.

En el ejemplo siguiente se muestra cómo usar `get_file_to_path` para descargar el contenido en el archivo **myfile** y almacenarlo en el archivo *out-sunset.png*.

```python
file_service.get_file_to_path('myshare', None, 'myfile', 'out-sunset.png')
```

---

## <a name="create-a-share-snapshot"></a>Creación de una instantánea de recurso compartido

Puede crear una copia a un punto dado de todo el recurso compartido de archivos.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateSnapshot":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```python
snapshot = file_service.snapshot_share(share_name)
snapshot_id = snapshot.snapshot
```

**Creación de una instantánea de recurso compartido con metadatos**

```python
metadata = {"foo": "bar"}
snapshot = file_service.snapshot_share(share_name, metadata=metadata)
```

---

## <a name="list-shares-and-snapshots"></a>Enumeración de recursos compartidos e instantáneas

Puede enumerar todas las instantáneas para un recurso compartido determinado.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_ListSharesAndSnapshots":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```python
shares = list(file_service.list_shares(include_snapshots=True))
```

---

## <a name="browse-share-snapshot"></a>Examinación de instantáneas de recurso compartido

Puede examinar cada instantánea de recurso compartido para recuperar archivos y directorios desde ese punto dado.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_BrowseSnapshotDir":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```python
directories_and_files = list(
    file_service.list_directories_and_files(share_name, snapshot=snapshot_id))
```

---

## <a name="get-file-from-share-snapshot"></a>Obtención del archivo desde una instantánea de recurso compartido

Puede descargar un archivo desde una instantánea de recurso compartido. Esto le permite restaurar una versión anterior de un archivo.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DownloadSnapshotFile":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```python
with open(FILE_PATH, 'wb') as stream:
    file = file_service.get_file_to_stream(
        share_name, directory_name, file_name, stream, snapshot=snapshot_id)
```

---

## <a name="delete-a-single-share-snapshot"></a>Eliminación de una instantánea de recurso compartido única
Puede eliminar una instantánea de recurso compartido única.

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteSnapshot":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

```python
file_service.delete_share(share_name, snapshot=snapshot_id)
```

---

## <a name="delete-a-file"></a>Eliminación de un archivo

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

Para eliminar un archivo, llame a [delete_file](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharefileclient#delete-file---kwargs-).

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteFile":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

Para eliminar un archivo, llame a [delete_file](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#delete-file-share-name--directory-name--file-name--timeout-none-).

```python
file_service.delete_file('myshare', None, 'myfile')
```

---

## <a name="delete-share-when-share-snapshots-exist"></a>Eliminación de un recurso compartido cuando existen instantáneas de recurso compartido

# <a name="azure-python-sdk-v12"></a>[SDK de Python de Azure v12](#tab/python)

Para eliminar un recurso compartido que contenga instantáneas, llame a [delete_share](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient#delete-share-delete-snapshots-false----kwargs-) con `delete_snapshots=True`.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteShare":::

# <a name="azure-python-sdk-v2"></a>[SDK de Python de Azure v2](#tab/python2)

No se puede eliminar un recurso compartido que contenga instantáneas a no ser que todas las instantáneas se eliminen primero.

```python
file_service.delete_share(share_name, delete_snapshots=DeleteSnapshot.Include)
```

---

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a manipular Azure Files con Python, siga estos vínculos para obtener más información.

- [Centro para desarrolladores de Python](/azure/developer/python/)
- [API de REST de servicios de Azure Storage](/rest/api/azure/)
- [SDK de Microsoft Azure Storage para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage)
