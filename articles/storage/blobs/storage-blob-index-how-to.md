---
title: Uso de etiquetas de índice de blobs para administrar y buscar datos en Azure Blob Storage
description: Vea ejemplos de cómo usar etiquetas de índice de blobs para clasificar, administrar y consultar objetos de blobs.
author: twooley
ms.author: twooley
ms.date: 03/05/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.reviewer: klaasl
ms.custom: devx-track-csharp
ms.openlocfilehash: 63f06b9cab7896bcd98b6b6cd9c037b7ee83b47f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451775"
---
# <a name="use-blob-index-tags-preview-to-manage-and-find-data-on-azure-blob-storage"></a>Uso de etiquetas de índice de blobs (versión preliminar) para administrar y buscar datos en Azure Blob Storage

Las etiquetas de índice de blobs clasifican los datos de la cuenta de almacenamiento mediante atributos de etiqueta clave-valor. Estas etiquetas se indexan y se exponen automáticamente como un índice multidimensional que se puede buscar para encontrar fácilmente los datos. En este artículo se muestra cómo establecer, obtener y buscar datos mediante etiquetas de índice de blobs.

> [!IMPORTANT]
> Las etiquetas de índice de blob se encuentran actualmente en **versión preliminar** y están disponibles en todas las regiones públicas. Consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

Para obtener más información sobre esta característica junto con las limitaciones y los problemas conocidos, vea [Administración y búsqueda de datos en Azure Blob con etiquetas de índice de blobs (versión preliminar)](storage-manage-find-blobs.md).

## <a name="prerequisites"></a>Prerrequisitos

# <a name="portal"></a>[Portal](#tab/azure-portal)

- Una suscripción a Azure registrada y aprobada para el acceso a la versión preliminar del índice de blobs
- Acceso a [Azure Portal](https://portal.azure.com/)

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/net)

Como el índice de blobs está en versión preliminar, el paquete de almacenamiento de .NET se publica en la fuente NuGet de la versión preliminar. Esta biblioteca está sujeta a cambios durante el período de versión preliminar.

1. Configure el proyecto de Visual Studio para empezar a usar la biblioteca cliente de Azure Blob Storage v12 para .NET. Para más información, vea esta [Inicio rápido de .NET](storage-quickstart-blobs-dotnet.md).

2. En el administrador de paquetes de NuGet, busque el paquete **Azure.Storage.Blobs** e instale la versión **12.7.0-preview.1** o posterior en el proyecto. También puede ejecutar el comando de PowerShell: `Install-Package Azure.Storage.Blobs -Version 12.7.0-preview.1`

   Para saber cómo hacerlo, consulte [Búsqueda e instalación de un paquete](/nuget/consume-packages/install-use-packages-visual-studio#find-and-install-a-package).

3. Agregue las instrucciones USING siguientes al inicio del archivo de código.

    ```csharp
    using Azure;
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Storage.Blobs.Specialized;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    ```

---

## <a name="upload-a-new-blob-with-index-tags"></a>Carga de un nuevo blob con etiquetas de índice

Esta tarea puede realizarla un [propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) o una entidad de seguridad a la que se haya concedido permiso para la [operación de proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` a través de un rol personalizado de Azure.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de almacenamiento. 

2. Vaya a la opción **Contenedores** de **Blob service** y seleccione el contenedor.

3. Seleccione el botón **Cargar** y examine el sistema de archivos local para buscar un archivo para cargarlo como un blob en bloques.

4. Expanda la lista desplegable **Avanzado** y vaya a la sección **Blob Index Tags** (Etiquetas de índice de blobs).

5. Escriba las etiquetas de índice de blobs de clave-valor que quiere aplicar a los datos.

6. Seleccione el botón **Cargar** para cargar el blob.

:::image type="content" source="media/storage-blob-index-concepts/blob-index-upload-data-with-tags.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra cómo cargar un blob con etiquetas de índice.":::

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/net)

En el ejemplo siguiente se muestra cómo crear un blob en anexos con etiquetas establecidas durante la creación.

```csharp
static async Task BlobIndexTagsOnCreate()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container = serviceClient.GetBlobContainerClient("mycontainer");

      try
      {
          // Create a container
          await container.CreateIfNotExistsAsync();

          // Create an append blob
          AppendBlobClient appendBlobWithTags = container.GetAppendBlobClient("myAppendBlob0.logs");

          // Blob index tags to upload
          AppendBlobCreateOptions appendOptions = new AppendBlobCreateOptions();
          appendOptions.Tags = new Dictionary<string, string>
          {
              { "Sealed", "false" },
              { "Content", "logs" },
              { "Date", "2020-04-20" }
          };

          // Upload data with tags set on creation
          await appendBlobWithTags.CreateAsync(appendOptions);
      }
      finally
      {
      }
   }
```

---

## <a name="get-set-and-update-blob-index-tags"></a>Obtención, establecimiento y actualización de etiquetas de índice de blobs

La obtención de las etiquetas de índice de blob puede realizarla un [propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) o una entidad de seguridad a la que se haya concedido permiso para la [operación de proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` a través de un rol personalizado de Azure.

El establecimiento y actualización de las etiquetas de índice de blob puede realizarla un [propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) o una entidad de seguridad a la que se haya concedido permiso para la [operación de proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` a través de un rol personalizado de Azure.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de almacenamiento. 

2. Vaya a la opción **Contenedores** de **Blob service** y seleccione el contenedor.

3. Seleccione el blob en la lista de blobs dentro del contenedor seleccionado.

4. En la pestaña Información general del blob se muestran las propiedades del blob, incluida cualquier **etiqueta de índice de blobs**.

5. Puede obtener, establecer, modificar o eliminar cualquiera de las etiquetas de índice de clave-valor del blob.

6. Seleccione el botón **Guardar** para confirmar las actualizaciones del blob.

:::image type="content" source="media/storage-blob-index-concepts/blob-index-get-set-tags.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra cómo obtener, establecer, actualizar y eliminar etiquetas de índice en blobs.":::

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/net)

```csharp
static async Task BlobIndexTagsExample()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container = serviceClient.GetBlobContainerClient("mycontainer");

      try
      {
          // Create a container
          await container.CreateIfNotExistsAsync();

          // Create a new append blob
          AppendBlobClient appendBlob = container.GetAppendBlobClient("myAppendBlob1.logs");
          await appendBlob.CreateAsync();

          // Set or update blob index tags on existing blob
          Dictionary<string, string> tags = new Dictionary<string, string>
          {
              { "Project", "Contoso" },
              { "Status", "Unprocessed" },
              { "Sealed", "true" }
          };
          await appendBlob.SetTagsAsync(tags);

          // Get blob index tags
          Response<IDictionary<string, string>> tagsResponse = await appendBlob.GetTagsAsync();
          Console.WriteLine(appendBlob.Name);
          foreach (KeyValuePair<string, string> tag in tagsResponse.Value)
          {
              Console.WriteLine($"{tag.Key}={tag.Value}");
          }

          // List blobs with all options returned including blob index tags
          await foreach (BlobItem blobItem in container.GetBlobsAsync(BlobTraits.All))
          {
              Console.WriteLine(Environment.NewLine + blobItem.Name);
              foreach (KeyValuePair<string, string> tag in blobItem.Tags)
              {
                  Console.WriteLine($"{tag.Key}={tag.Value}");
              }
          }

          // Delete existing blob index tags by replacing all tags
          Dictionary<string, string> noTags = new Dictionary<string, string>();
          await appendBlob.SetTagsAsync(noTags);

      }
      finally
      {
      }
   }
```

---

## <a name="filter-and-find-data-with-blob-index-tags"></a>Filtrado y búsqueda de datos con etiquetas de índice de blobs

Esta tarea puede realizarla un [propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) o una entidad de seguridad a la que se haya concedido permiso para la [operación de proveedor de recursos de Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action` a través de un rol personalizado de Azure.

# <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, el filtro de las etiquetas de índice de blobs aplica automáticamente el parámetro `@container` para limitar el ámbito del contenedor seleccionado. Si quiere filtrar y buscar datos etiquetados en toda la cuenta de almacenamiento, use la API REST, los SDK o las herramientas.

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de almacenamiento. 

2. Vaya a la opción **Contenedores** de **Blob service** y seleccione el contenedor.

3. Seleccione el botón **Blob Index tags filter** (Filtro de etiquetas de índice de blobs) para filtrar en el contenedor seleccionado.

4. Entrada de un valor de clave y etiqueta de índice de blobs

5. Seleccione el botón **Blob Index tags filter** (Filtro de etiquetas de índice de blobs) para agregar filtros de etiquetas adicionales (hasta diez).

:::image type="content" source="media/storage-blob-index-concepts/blob-index-tag-filter-within-container.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra cómo filtrar y buscar blobs etiquetados mediante etiquetas de índice":::

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/net)

```csharp
static async Task FindBlobsByTagsExample()
   {
      BlobServiceClient serviceClient = new BlobServiceClient(ConnectionString);
      BlobContainerClient container1 = serviceClient.GetBlobContainerClient("mycontainer");
      BlobContainerClient container2 = serviceClient.GetBlobContainerClient("mycontainer2");

      // Blob index queries and selection
      String singleEqualityQuery = @"""Archive"" = 'false'";
      String andQuery = @"""Archive"" = 'false' AND ""Priority"" = '01'";
      String rangeQuery = @"""Date"" >= '2020-04-20' AND ""Date"" <= '2020-04-30'";
      String containerScopedQuery = @"@container = 'mycontainer' AND ""Archive"" = 'false'";

      String queryToUse = containerScopedQuery;

      try
      {
          // Create a container
          await container1.CreateIfNotExistsAsync();
          await container2.CreateIfNotExistsAsync();

          // Create append blobs
          AppendBlobClient appendBlobWithTags0 = container1.GetAppendBlobClient("myAppendBlob00.logs");
          AppendBlobClient appendBlobWithTags1 = container1.GetAppendBlobClient("myAppendBlob01.logs");
          AppendBlobClient appendBlobWithTags2 = container1.GetAppendBlobClient("myAppendBlob02.logs");
          AppendBlobClient appendBlobWithTags3 = container2.GetAppendBlobClient("myAppendBlob03.logs");
          AppendBlobClient appendBlobWithTags4 = container2.GetAppendBlobClient("myAppendBlob04.logs");
          AppendBlobClient appendBlobWithTags5 = container2.GetAppendBlobClient("myAppendBlob05.logs");
           
          // Blob index tags to upload
          CreateAppendBlobOptions appendOptions = new CreateAppendBlobOptions();
          appendOptions.Tags = new Dictionary<string, string>
          {
              { "Archive", "false" },
              { "Priority", "01" },
              { "Date", "2020-04-20" }
          };
          
          CreateAppendBlobOptions appendOptions2 = new CreateAppendBlobOptions();
          appendOptions2.Tags = new Dictionary<string, string>
          {
              { "Archive", "true" },
              { "Priority", "02" },
              { "Date", "2020-04-24" }
          };

          // Upload data with tags set on creation
          await appendBlobWithTags0.CreateAsync(appendOptions);
          await appendBlobWithTags1.CreateAsync(appendOptions);
          await appendBlobWithTags2.CreateAsync(appendOptions2);
          await appendBlobWithTags3.CreateAsync(appendOptions);
          await appendBlobWithTags4.CreateAsync(appendOptions2);
          await appendBlobWithTags5.CreateAsync(appendOptions2);

          // Find Blobs given a tags query
          Console.WriteLine("Find Blob by Tags query: " + queryToUse + Environment.NewLine);

          List<TaggedBlobItem> blobs = new List<TaggedBlobItem>();
          await foreach (TaggedBlobItem taggedBlobItem in serviceClient.FindBlobsByTagsAsync(queryToUse))
          {
              blobs.Add(taggedBlobItem);
          }

          foreach (var filteredBlob in blobs)
          {
              Console.WriteLine($"BlobIndex result: ContainerName= {filteredBlob.ContainerName}, " +
                  $"BlobName= {filteredBlob.Name}");
          }

      }
      finally
      {
      }
   }
```

---

## <a name="lifecycle-management-with-blob-index-tag-filters"></a>Administración del ciclo de vida con filtros de etiquetas de índice de blobs

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de almacenamiento. 

2. Vaya a la opción **Administración del ciclo de vida** de **Blob service**.

3. Seleccione *Agregar regla* y rellene los campos del formulario Conjunto de acciones.

4. Seleccione **Conjunto de filtros** para agregar un filtro opcional para la coincidencia de prefijo y la coincidencia de índice de blobs.

  :::image type="content" source="media/storage-blob-index-concepts/blob-index-match-lifecycle-filter-set.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra cómo agregar etiquetas de índice para la administración del ciclo de vida.":::

5. Seleccione **Revisar y agregar** para revisar la configuración de la regla.

  :::image type="content" source="media/storage-blob-index-concepts/blob-index-lifecycle-management-example.png" alt-text="Captura de pantalla de Azure Portal en la que se muestra una regla de administración del ciclo de vida con un ejemplo de filtro de etiquetas de índice de blob":::

6. Seleccione **Agregar** para aplicar la nueva regla a la directiva de administración del ciclo de vida.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/net)

Las directivas de [administración del ciclo de vida](storage-lifecycle-management-concepts.md) se aplican a cada cuenta de almacenamiento en el nivel del plano de control. Para .NET, instale la versión 16.0.0 o posterior de la [biblioteca Microsoft.Azure.Management.Storage](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage/).

---

## <a name="next-steps"></a>Pasos siguientes

 - Para obtener más información sobre las etiquetas de índice de blobs, vea [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](storage-manage-find-blobs.md ).
 - Para obtener más información sobre la administración del ciclo de vida, vea [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md)
