---
title: Obtención del tipo de cuenta de almacenamiento y el nombre de SKU con .NET
titleSuffix: Azure Storage
description: Aprenda a obtener el tipo de cuenta de Azure Storage y el nombre de SKU mediante la biblioteca cliente de .NET.
services: storage
author: twooley
ms.author: twooley
ms.date: 11/12/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 96c77291a1c540e2ff95fb911b2321e78f20e604
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110478850"
---
# <a name="get-storage-account-type-and-sku-name-with-net"></a>Obtención del tipo de cuenta de almacenamiento y el nombre de SKU con .NET

En este artículo se muestra cómo obtener el tipo de cuenta de Azure Storage y el nombre de SKU de un blob mediante la [biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage).

## <a name="about-account-type-and-sku-name"></a>Acerca del tipo de cuenta y el nombre de SKU

**Tipo de cuenta**: Entre los tipos de cuenta válidos se incluyen `BlobStorage`, `BlockBlobStorage`, `FileStorage`, `Storage` y `StorageV2`. La sección de [información general sobre la cuenta de Azure Storage](storage-account-overview.md) contiene más información, incluidas las descripciones de las diversas cuentas de almacenamiento.

**Nombre de SKU**: Entre los nombres de SKU válidos se incluyen `Premium_LRS`, `Premium_ZRS`, `Standard_GRS`, `Standard_GZRS`, `Standard_LRS`, `Standard_RAGRS`, `Standard_RAGZRS` y `Standard_ZRS`. Los nombres de SKU distinguen mayúsculas de minúsculas y son campos de cadena de la [clase SkuName](/dotnet/api/microsoft.azure.management.storage.models.skuname).

## <a name="retrieve-account-information"></a>Recuperación de la información de la cuenta

En el ejemplo de código siguiente se recuperan y se muestran las propiedades de la cuenta de solo lectura.

# <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Para obtener el tipo de cuenta de almacenamiento y el nombre de SKU asociados a un blob, llame al método [GetAccountInfo](/dotnet/api/azure.storage.blobs.blobserviceclient.getaccountinfo) o [GetAccountInfoAsync](/dotnet/api/azure.storage.blobs.blobserviceclient.getaccountinfoasync).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Account.cs" id="Snippet_GetAccountInfo":::

# <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

Para obtener el tipo de cuenta de almacenamiento y el nombre de SKU asociados a un blob, llame al método [GetAccountProperties](/dotnet/api/microsoft.azure.storage.blob.cloudblob.getaccountproperties) o [GetAccountPropertiesAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.getaccountpropertiesasync).

```csharp
private static async Task GetAccountInfoAsync(CloudBlob blob)
{
    try
    {
        // Get the blob's storage account properties.
        AccountProperties acctProps = await blob.GetAccountPropertiesAsync();

        // Display the properties.
        Console.WriteLine("Account properties");
        Console.WriteLine("  AccountKind: {0}", acctProps.AccountKind);
        Console.WriteLine("      SkuName: {0}", acctProps.SkuName);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre otras operaciones que puede realizar en una cuenta de almacenamiento mediante [Azure Portal](https://portal.azure.com) y la API REST de Azure.

- [Obtención de una operación de información de la cuenta (REST)](/rest/api/storageservices/get-account-information)
