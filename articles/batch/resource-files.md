---
title: Crear y usar archivos de recursos
description: Aprenda a crear archivos de recursos de Azure Batch desde diversos orígenes de entrada. En este artículo se abordan algunos métodos comunes para crear archivos de recursos y colocarlos en una máquina virtual.
ms.date: 05/25/2021
ms.topic: how-to
ms.openlocfilehash: 1ef8cde8c345cebeb166cddd67a1951d71eea810
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467699"
---
# <a name="creating-and-using-resource-files"></a>Crear y usar archivos de recursos

A menudo, una tarea de Azure Batch requiere algún tipo de procesamiento de datos. Los archivos de recursos constituyen el modo de proporcionar estos datos a la máquina virtual de Batch a través de una tarea. Todos los tipos de tareas admiten archivos de recursos: tareas de inicio, tareas de preparación del trabajo, tareas de liberación del trabajo, etc. En este artículo se abordan algunos métodos comunes para crear archivos de recursos y colocarlos en una máquina virtual.  

Los archivos de recursos colocan datos en una máquina virtual en Batch. El tipo de datos y cómo se usen son flexibles. De todas formas, existen algunos casos de uso comunes:

- Aprovisionamiento de archivos comunes en cada VM usando archivos de recursos en una tarea de inicio.
- Aprovisionamiento de datos de entrada que se van a procesar en las tareas.

Los archivos comunes pueden ser, por ejemplo, archivos de una tarea de inicio usados para instalar las aplicaciones que ejecutan las tareas. Los datos de entrada pueden ser cualquier dato de vídeo o imagen sin procesar, o información que Batch va a procesar.

## <a name="types-of-resource-files"></a>Tipos de archivos de recursos

Existen diversas opciones disponibles para generar archivos de recursos, cada una con sus propios [métodos](/dotnet/api/microsoft.azure.batch.resourcefile#methods). El proceso de creación de los archivos de recursos varía en función de dónde se almacenen los datos originales y de si se deben crear varios archivos.

- [Dirección URL del contenedor de almacenamiento](#storage-container-url): genera un archivo de recursos a partir de cualquier contenedor de almacenamiento de Azure.
- [Nombre del contenedor de almacenamiento](#storage-container-name-autostorage): genera un archivo de recursos a partir del nombre de un contenedor de la cuenta de Azure Storage vinculada a la cuenta de Batch (cuenta de almacenamiento automático).
- [Archivo de recurso único desde el punto de conexión web](#single-resource-file-from-web-endpoint): genera un único archivo de recursos a partir de cualquier dirección URL HTTP válida.

### <a name="storage-container-url"></a>Dirección URL del contenedor de almacenamiento

El uso de una dirección URL de contenedor de almacenamiento conlleva que se puede acceder a los archivos de cualquier contenedor de almacenamiento de Azure, si se tienen los permisos correctos.

En este ejemplo de C#, los archivos ya se han cargado en un contenedor de almacenamiento de Azure como Blob Storage. Para acceder a los datos necesarios para crear un archivo de recursos, primero hay que acceder al contenedor de almacenamiento.

Cree un URI de firma de acceso compartido (SAS) con los permisos correctos para acceder al contenedor de almacenamiento. Establezca la fecha de vencimiento y los permisos de la SAS. En este caso, no se especifica ninguna fecha de inicio, por lo que la SAS entrará en vigor inmediatamente y expirará dos horas después de que se genere.

```csharp
SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
{
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(2),
    Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List
};
```

> [!NOTE]
> Para acceder al contenedor, debe tener los permisos `Read` y `List`, mientras que para acceder al blob solo se necesita el permiso `Read`.

Una vez configurados los permisos, cree el token de la SAS y dé formato a la dirección URL de la SAS para acceder al contenedor de almacenamiento. Con la dirección URL de SAS formateada del contenedor de almacenamiento, genere un archivo de recursos con [FromStorageContainerUrl](/dotnet/api/microsoft.azure.batch.resourcefile.fromstoragecontainerurl).

```csharp
CloudBlobContainer container = blobClient.GetContainerReference(containerName);

string sasToken = container.GetSharedAccessSignature(sasConstraints);
string containerSasUrl = String.Format("{0}{1}", container.Uri, sasToken);

ResourceFile inputFile = ResourceFile.FromStorageContainerUrl(containerSasUrl);
```

Si lo desea, puede usar la propiedad [blobPrefix](/dotnet/api/microsoft.azure.batch.resourcefile.blobprefix) para limitar las descargas solo a los blobs cuyo nombre comienza con un prefijo especificado:

```csharp
ResourceFile inputFile = ResourceFile.FromStorageContainerUrl(containerSasUrl, blobPrefix = yourPrefix);
```

Otra forma de generar una dirección URL de SAS consiste en habilitar el acceso de lectura anónimo y público a un contenedor y sus blobs en Azure Blob Storage. Al hacerlo, puede conceder acceso de solo lectura a estos recursos sin compartir la clave de cuenta y sin necesidad de una SAS. El acceso de lectura público se suele usar en escenarios donde se quiere que ciertos blobs estén siempre disponibles para el acceso de lectura anónimo. Si este escenario encaja con su solución, vea el artículo [Administración del acceso de lectura anónimo a contenedores y blobs](../storage/blobs/anonymous-read-access-configure.md) para obtener más información sobre cómo administrar el acceso a los datos de blob.

### <a name="storage-container-name-autostorage"></a>Nombre del contenedor de almacenamiento (almacenamiento automático)

En lugar de configurar y crear una dirección URL de SAS, puede usar el nombre del contenedor de almacenamiento de Azure para tener acceso a los datos de blob. El contenedor de almacenamiento que utilice debe estar en la cuenta de Azure Storate que esté vinculada a su cuenta de Batch, a veces conocida como *cuenta de almacenamiento automático*. El contenedor de almacenamiento automático permite omitir la configuración y creación de una dirección URL de SAS para acceder a un contenedor de almacenamiento. En su lugar, tiene que indicar el nombre del contenedor de almacenamiento en la cuenta de almacenamiento vinculada.

Si no tiene una cuenta de almacenamiento automático, vea los pasos de [Creación de una cuenta de Batch](batch-account-create-portal.md) para obtener más información sobre cómo crear y vincular una cuenta de almacenamiento.

En el ejemplo siguiente se usa [AutoStorageContainer](/dotnet/api/microsoft.azure.batch.resourcefile.fromautostoragecontainer) para generar el archivo a partir de los datos de la cuenta de almacenamiento automático.

```csharp
ResourceFile inputFile = ResourceFile.FromAutoStorageContainer(containerName);
```

Igual que con una dirección URL de contenedor de almacenamiento, puede usar la propiedad [blobPrefix](/dotnet/api/microsoft.azure.batch.resourcefile.blobprefix) para especificar qué blobs se descargarán:

```csharp
ResourceFile inputFile = ResourceFile.FromAutoStorageContainer(containerName, blobPrefix = yourPrefix);
```

### <a name="single-resource-file-from-web-endpoint"></a>Archivo de recurso único desde el punto de conexión web

Para crear un único archivo de recursos, puede especificar una dirección URL HTTP válida que contenga los datos de entrada. La dirección URL se suministra a la API de Batch y, seguidamente, se usan los datos para crear un archivo de recursos. Este método se puede usar tanto si los datos para crear el archivo de recursos están en Azure Storage como en cualquier otra ubicación web, como un punto de conexión de GitHub.

En el ejemplo siguiente se usa [FromUrl](/dotnet/api/microsoft.azure.batch.resourcefile.fromurl) para recuperar el archivo de una cadena que contiene una dirección URL válida y, a continuación, se genera un archivo de recursos para utilizarlo en la tarea. En este escenario no se requieren credenciales. (Las credenciales son necesarias si se usa Blob Storage, a menos que el acceso de lectura público esté habilitado en el contenedor de blobs).

```csharp
ResourceFile inputFile = ResourceFile.FromUrl(yourURL, filePath);
```

También puede usar una cadena que defina como una dirección URL (o una combinación de cadenas que, juntas, creen la dirección URL completa del archivo).

```csharp
ResourceFile inputFile = ResourceFile.FromUrl(yourDomain + yourFile, filePath);
```

## <a name="tips-and-suggestions"></a>Recomendaciones y sugerencias

Las tareas de Azure Batch pueden usar archivos de muchas maneras, por eso Batch proporciona distintas opciones para administrar archivos en tareas. Los siguientes escenarios no pretenden ser exhaustivos, pero abordan algunas situaciones comunes y ofrecen recomendaciones.

### <a name="many-resource-files"></a>Muchos archivos de recursos

Si los archivos de tareas comunes se comparten entre muchas tareas del trabajo de Batch, puede usar un [paquete de aplicación](batch-application-packages.md) que contenga esos archivos. Los paquetes de aplicación proporcionan optimización para la velocidad de descarga y los datos de los paquetes de aplicación se almacenan en caché entre tareas. Con los paquetes de aplicación, no es necesario administrar varios archivos de recursos manualmente ni generar direcciones URL de SAS para acceder a los archivos en Azure Storage. Batch funciona en segundo plano con Azure Storage para almacenar paquetes de aplicación e implementarlos en los nodos de proceso. Si los archivos de tareas no cambian con frecuencia, los paquetes de aplicación pueden ser una buena opción para su solución.

Contrariamente, si la tarea tiene un gran número de archivos únicos para la tarea, es probable que los archivos de recursos sean la mejor opción. Las tareas que utilizan archivos únicos se deben actualizar o reemplazar a menudo, lo cual no es tan fácil de hacer con el contenido de los paquetes de aplicación. Los archivos de recursos reportan una mayor flexibilidad a la hora de actualizar, agregar o modificar archivos individuales.

### <a name="number-of-resource-files-per-task"></a>Número de archivos de recursos por tarea

Si en una tarea se especifican varios cientos de archivos de recursos, Batch puede rechazar la tarea por ser demasiado grande. Lo mejor es mantener tareas pequeñas, reduciendo para ello el número de archivos de recursos en la propia tarea.

Si no hay ninguna manera de reducir el número de archivos que la tarea necesita, puede optimizar la tarea creando un único archivo de recursos que haga referencia a un contenedor de almacenamiento de archivos de recursos. Para ello, coloque los archivos de recursos en un contenedor de Azure Storage y use uno de los métodos descritos anteriormente para generar archivos de recursos según sea necesario.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre los [paquetes de aplicación](batch-application-packages.md) como alternativa a los archivos de recursos.
- Obtenga información sobre el [uso de contenedores](batch-docker-container-workloads.md) para archivos de recursos.
- Obtenga información sobre cómo [recopilar y guardar los datos de salida de las tareas](batch-task-output.md).
- Obtenga información acerca de las [API y herramientas de Batch](batch-apis-tools.md) disponibles para la creación de soluciones de Batch.
