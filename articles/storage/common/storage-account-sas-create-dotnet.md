---
title: Creación de una SAS de cuenta con .NET
titleSuffix: Azure Storage
description: Obtenga información sobre cómo crear una firma de acceso compartido (SAS) de cuenta mediante la biblioteca cliente de .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/12/2020
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 3e08962c0cff2e9aa04601f89e677334878e2f53
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110461868"
---
# <a name="create-an-account-sas-with-net"></a>Creación de una SAS de cuenta con .NET

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

En este artículo se muestra cómo usar la clave de cuenta de almacenamiento para crear una SAS de cuenta con la [biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage).

## <a name="create-an-account-sas"></a>Creación de una SAS de cuenta

### <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

Se firma una SAS de una cuenta con la clave de acceso de la cuenta. Use la clase [StorageSharedKeyCredential](/dotnet/api/azure.storage.storagesharedkeycredential) para crear la credencial que se usa para firmar la SAS. A continuación, cree un objeto [AccountSasBuilder](/dotnet/api/azure.storage.sas.accountsasbuilder) y llame al elemento [ToSasQueryParameters](/dotnet/api/azure.storage.sas.accountsasbuilder.tosasqueryparameters) para obtener la cadena de token de SAS.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_GetAccountSASToken":::

### <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnetv11)

Para crear una SAS de cuenta para un contenedor, llame al método [CloudStorageAccount.GetSharedAccessSignature](/dotnet/api/microsoft.azure.storage.cloudstorageaccount.getsharedaccesssignature).

En el ejemplo de código siguiente se crea una SAS de cuenta que es válida para los servicios Blob y Archivo y da al cliente permisos de lectura, escritura y lista para acceder a las API de nivel de servicio. La SAS de cuenta restringe el protocolo a HTTPS, por lo que la solicitud se debe realizar con HTTPS. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use Shared Key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=<storage-account>;AccountKey=<account-key>";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

---

## <a name="use-an-account-sas-from-a-client"></a>Uso de una SAS de cuenta desde un cliente

Para usar la SAS de cuenta a fin de acceder a las API de nivel de servicio para Blob service, construya un objeto de cliente de Blob service con la SAS y el punto de conexión de Blob Storage para la cuenta de almacenamiento.

### <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_UseAccountSAS":::

### <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnetv11)

En este fragmento de código, reemplace el marcador de posición `<storage-account>` por el nombre de la cuenta de almacenamiento.

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "<storage-account>", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](storage-sas-overview.md)
- [Creación de una SAS de cuenta](/rest/api/storageservices/create-account-sas)
