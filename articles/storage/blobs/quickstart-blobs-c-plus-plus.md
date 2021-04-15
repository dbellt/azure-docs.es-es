---
title: 'Inicio rápido: Biblioteca de Azure Blob Storage v12: C++'
description: En este inicio rápido, aprenderá a usar la versión 12 de la biblioteca cliente de Azure Blob Storage para C++ para crear un contenedor y un blob en Blob Storage (objeto). A continuación, aprenderá a descargar el blob en un equipo local y a enumerar todos los blobs en un contenedor.
author: twooley
ms.author: twooley
ms.date: 10/21/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: e6832cc835a464a2a3d17d5ed286e71b17bc980e
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107312287"
---
# <a name="quickstart-azure-blob-storage-client-library-v12-for-c"></a>Inicio rápido: Versión 12 de la biblioteca cliente de Azure Blob Storage para C++

Introducción a la versión 12 de la biblioteca cliente de Azure Blob Storage para C++. Azure Blob Storage es la solución de almacenamiento de objetos de Microsoft para la nube. Siga los pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados.

La versión 12 de la biblioteca cliente de Azure Blob Storage para C++ se puede usar para:

- Crear un contenedor
- Cargar un blob en Azure Storage
- Enumerar todos los blobs de un contenedor
- Descargar el blob en el equipo local
- Eliminación de un contenedor

Recursos:

- [Documentación de referencia de API](https://azure.github.io/azure-sdk-for-cpp/storage.html)
- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-cpp/tree/master/sdk/storage)
- [Muestras](../common/storage-samples-c-plus-plus.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Suscripción de Azure](https://azure.microsoft.com/free/)
- [Cuenta de Almacenamiento de Azure](../common/storage-account-create.md)
- [Compilador de C++](https://azure.github.io/azure-sdk/cpp_implementation.html#supported-platforms)
- [CMake](https://cmake.org/)
- [Vcpkg: administrador de paquetes de C y C++](https://github.com/microsoft/vcpkg/blob/master/docs/README.md)
- [LibCurl](https://curl.haxx.se/libcurl/)
- [LibXML2](http://www.xmlsoft.org/)

## <a name="setting-up"></a>Instalación

En esta sección se explica cómo preparar un proyecto para que funcione con la versión 12 de la biblioteca cliente de Azure Blob Storage para C++.

### <a name="install-the-packages"></a>Instalación de los paquetes

Si aún no lo ha hecho, instale los paquetes LibCurl y LibXML2 mediante el comando `vcpkg install`.

```console
vcpkg.exe install libxml2:x64-windows curl:x64-windows
```

Siga las instrucciones de GitHub para adquirir y compilar [Azure SDK para C++](https://github.com/Azure/azure-sdk-for-cpp/).

### <a name="create-the-project"></a>Creación del proyecto

En Visual Studio, cree una nueva aplicación de consola de C++ para Windows llamada *BlobQuickstartV12*.

:::image type="content" source="./media/quickstart-blobs-c-plus-plus/vs-create-project.jpg" alt-text="Cuadro de diálogo de Visual Studio para configurar una nueva aplicación de consola de Windows en C++":::

Agregue las siguientes bibliotecas al proyecto:

- libcurl.lib
- libxml2.lib
- bcrypt.lib
- Crypt32.Lib
- WS2_32.Lib
- azure-core.lib
- azure-storage-common.lib
- azure-storage-blobs.lib

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modelo de objetos

Azure Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados. Los datos no estructurados son datos que no se ciñen a ningún un modelo de datos o definición concretos, como texto o datos binarios. Blob Storage ofrece tres tipos de recursos:

- La cuenta de almacenamiento
- Un contenedor en la cuenta de almacenamiento
- Un blob en el contenedor

En el siguiente diagrama se muestra la relación entre estos recursos.

![Diagrama de la arquitectura de Blob Storage](./media/storage-blobs-introduction/blob1.png)

Use las siguientes clases de C++ para interactuar con estos recursos:

- [BlobServiceClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_service_client.html): La clase `BlobServiceClient` permite manipular recursos de Azure Storage y contenedores de blobs.
- [BlobContainerClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html): La clase `BlobContainerClient` permite manipular contenedores de Azure Storage y sus blobs.
- [BlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html): La clase `BlobClient` permite manipular los blobs de Azure Storage. Es la clase base para todas las clases de blob especializadas.
- [BlockBlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_block_blob_client.html): La clase `BlockBlobClient` permite manipular los blobs en bloques de Azure Storage.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código de ejemplo muestran cómo realizar las siguientes tareas con la biblioteca cliente de Azure Blob Storage para C++:

- [Adición de archivos de inclusión](#add-include-files)
- [Obtención de la cadena de conexión](#get-the-connection-string)
- [Creación de un contenedor](#create-a-container)
- [Carga de los blobs en un contenedor](#upload-blobs-to-a-container)
- [Enumeración de los blobs de un contenedor](#list-the-blobs-in-a-container)
- [Descarga de los blobs](#download-blobs)
- [Eliminación de un contenedor](#delete-a-container)

### <a name="add-include-files"></a>Adición de archivos de inclusión

Desde el directorio del proyecto:

1. Abra el archivo de soluciones *BlobQuickstartV12.sln* en Visual Studio.
1. En Visual Studio, abra el archivo de origen *BlobQuickstartV12.cpp*
1. Quite todo el código incluido en `main` que se haya generado automáticamente
1. Agregue instrucciones `#include`.

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_Includes":::

### <a name="get-the-connection-string"></a>Obtención de la cadena de conexión

El código siguiente recupera la cadena de conexión de la cuenta de almacenamiento de la variable de entorno creada en [Configuración de la cadena de conexión de almacenamiento](#configure-your-storage-connection-string).

Agregue este código a `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_ConnectionString":::

### <a name="create-a-container"></a>Crear un contenedor

Cree una instancia de la clase [BlobContainerClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html) llamando a la función [CreateFromConnectionString](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#ad103d1e3a7ce7c53a82da887d12ce6fe). Luego, llame a [Create](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#a22a1e001068a4ec52bb1b6bd8b52c047) para crear el contenedor real en la cuenta de almacenamiento.

> [!IMPORTANT]
> Los nombres de contenedor deben estar en minúsculas. Para más información acerca de los contenedores de nomenclatura y los blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y realización de referencias a contenedores, blobs y metadatos).

Agregue este código al final de `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_CreateContainer":::

### <a name="upload-blobs-to-a-container"></a>Carga de los blobs en un contenedor

El siguiente fragmento de código:

1. Declara una cadena que contiene "Hello Azure!".
1. Obtiene una referencia a un objeto [BlockBlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_block_blob_client.html) llamando al contenedor [GetBlockBlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#acd8c68e3f37268fde0010dd478ff048f) de la sección [Crear un contenedor](#create-a-container).
1. Carga la cadena en el blob llamando a la función [UploadFrom](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_block_blob_client.html#af93af7e37f8806e39481596ef253f93d). Esta función crea el blob, en caso de que no exista, o lo actualiza si existe.

Agregue este código al final de `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_UploadBlob":::

### <a name="list-the-blobs-in-a-container"></a>Enumeración de los blobs de un contenedor

Enumere los blobs en el contenedor llamando a la función [ListBlobsFlatSegment](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#ac7712bc46909dc061d6bf554b496550c). Se ha agregado un solo blob al contenedor, por lo que la operación devuelve simplemente dicho blob.

Agregue este código al final de `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_ListBlobs":::

### <a name="download-blobs"></a>Descargar blobs

Obtenga las propiedades del blob cargado. Luego, declare y cambie el tamaño de un nuevo objeto `std::string` mediante las propiedades del blob cargado. Descargue el blob que ha creado anteriormente en el nuevo objeto `std::string`, para lo que debe llamar a la función [DownloadTo](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html#aa844f37a8c216f3cb0f27912b114c4d2) en la clase base [BlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html). Por último, muestre los datos del blob descargado.

Agregue este código al final de `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_DownloadBlob":::

### <a name="delete-a-blob"></a>Eliminación de un blob

El código siguiente elimina el blob del contenedor de Azure Blob Storage, para lo que llama a la función [BlobClient.Delete](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html#a621eabcc8d23893ca1eb106494198615).

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_DeleteBlob":::

### <a name="delete-a-container"></a>Eliminación de un contenedor

El código siguiente limpia los recursos que creó la aplicación; para ello, elimina todo el contenedor mediante [BlobContainerClient.Delete](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#aa6b1db52697ae92e9a1227e2e02a5178).

Agregue este código al final de `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_DeleteContainer":::

## <a name="run-the-code"></a>Ejecución del código

Esta aplicación crea un contenedor y carga un archivo de texto en Azure Blob Storage. Luego, en el ejemplo se enumeran los blobs del contenedor, se descarga el archivo y se muestra su contenido. Por último, la aplicación elimina tanto el blob como el contenedor.

La salida de la aplicación es similar a la del ejemplo siguiente:

```output
Azure Blob Storage v12 - C++ quickstart sample
Creating container: myblobcontainer
Uploading blob: blob.txt
Listing blobs...
Blob name: blob.txt
Downloaded blob contents: Hello Azure!
Deleting blob: blob.txt
Deleting container: myblobcontainer
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a cargar, descargar y enumerar blobs mediante C++. También ha aprendido a crear y eliminar un contenedor de Azure Blob Storage.

Para ver un ejemplo de Blob Storage para C++, siga estos pasos:

> [!div class="nextstepaction"]
> [Ejemplo del SDK de Azure Blob Storage v12 para C++](https://github.com/Azure/azure-sdk-for-cpp/tree/master/sdk/storage/azure-storage-blobs/sample)
