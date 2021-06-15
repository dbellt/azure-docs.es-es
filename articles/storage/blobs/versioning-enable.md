---
title: Habilitación y administración del control de versiones de blob
titleSuffix: Azure Storage
description: Aprenda a habilitar el control de versiones de blob desde Azure Portal o con una plantilla de Azure Resource Manager.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/09/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp, devx-track-azurepowershell
ms.openlocfilehash: 0647a777207d5add66b0480a79a52034ee1c3068
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110664916"
---
# <a name="enable-and-manage-blob-versioning"></a>Habilitación y administración del control de versiones de blob

Puede habilitar el control de versiones de Blob Storage para conservar automáticamente las versiones anteriores de un blob cuando se modifica o elimina. Cuando el control de versiones de blobs está habilitado, puede restaurar una versión anterior de un blob para recuperar los datos si se modifican o eliminan por error.

En este artículo se muestra cómo habilitar o deshabilitar el control de versiones de blobs para la cuenta de almacenamiento mediante Azure Portal o una plantilla de Azure Resource Manager. Para más información sobre el control de versiones de blobs, consulte [Control de versiones de blobs](versioning-overview.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="enable-blob-versioning"></a>Habilitación del control de versiones de blob

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para habilitar el control de versiones de blobs para una cuenta de almacenamiento en Azure Portal:

1. Vaya a la cuenta de almacenamiento desde el portal.
1. En **Blob service**, elija **Protección de datos**.
1. En la sección **Control de versiones**, seleccione **Habilitado**.

:::image type="content" source="media/versioning-enable/portal-enable-versioning.png" alt-text="Captura de pantalla que muestra cómo habilitar el control de versiones de blobs en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para habilitar el control de versiones de blobs para una cuenta de almacenamiento con PowerShell, primero instale la versión de módulo [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage) 2.3.0 o posterior. A continuación, llame al comando [Update-AzStorageBlobServiceProperty](/powershell/module/az.storage/update-azstorageblobserviceproperty) para habilitar el control de versiones, tal como se muestra en el ejemplo siguiente. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```powershell
# Set resource group and account variables.
$rgName = "<resource-group>"
$accountName = "<storage-account>"

# Enable versioning.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -IsVersioningEnabled $true
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para habilitar el control de versiones de blobs para una cuenta de almacenamiento con la CLI de Azure, instale primero la versión de la CLI de Azure 2.2.0 o posterior. A continuación, llame al comando [az storage account blob-service-properties update](/cli/azure/storage/account/blob-service-properties#az_storage_account_blob_service_properties_update) para habilitar el control de versiones, tal como se muestra en el ejemplo siguiente. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```azurecli
az storage account blob-service-properties update \
    --resource-group <resource_group> \
    --account-name <storage-account> \
    --enable-versioning true
```

# <a name="template"></a>[Plantilla](#tab/template)

Para habilitar el control de versiones de blobs con una plantilla, cree una plantilla con la propiedad **IsVersioningEnabled** definida como **true**. En los siguientes pasos se muestra cómo crear una plantilla en Azure Portal.

1. En Azure Portal, elija **Crear un recurso**.
1. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
1. Elija **Template Deployment**, elija **Crear** y, luego, **Cree su propia plantilla en el editor**.
1. En el editor de plantillas, pegue el código JSON siguiente. Reemplace el marcador de posición `<accountName>` por el nombre de la cuenta de almacenamiento.
1. Guarde la plantilla.
1. Especifique el grupo de recursos de la cuenta y, luego, elija el botón **Comprar** para implementar la plantilla y habilitar el control de versiones de blob.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Storage/storageAccounts/blobServices",
                "apiVersion": "2019-06-01",
                "name": "<accountName>/default",
                "properties": {
                    "IsVersioningEnabled": true
                }
            }
        ]
    }
    ```

Para obtener más información sobre la implementación de recursos con plantillas en Azure Portal, vea [Implementación de recursos con Azure Portal](../../azure-resource-manager/templates/deploy-portal.md).

---

## <a name="modify-a-blob-to-trigger-a-new-version"></a>Modificar un blob para desencadenar una nueva versión

En el ejemplo de código siguiente se muestra cómo desencadenar la creación de una nueva versión con la biblioteca de cliente de Azure Storage para .NET versión [12.5.1](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.1) o posteriores. Antes de ejecutar este ejemplo, asegúrese de que ha habilitado el control de versiones para la cuenta de almacenamiento.

En el ejemplo se crea un blob en bloques y, a continuación, se actualizan los metadatos del blob. La actualización de los metadatos del blob desencadena la creación de una nueva versión. En el ejemplo se recupera la versión inicial y la versión actual, y se muestra que solo la versión actual incluye los metadatos.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_UpdateVersionedBlobMetadata":::

## <a name="list-blob-versions"></a>Enumeración de las versiones de blob

Para enumerar las versiones o instantáneas de blob con la biblioteca cliente de .NET versión 12, especifique el parámetro [BlobStates](/dotnet/api/azure.storage.blobs.models.blobstates) con el campo **Version**.

En el ejemplo de código siguiente se muestra cómo enumerar las versiones de blob con la biblioteca cliente de Azure Storage para .NET versión [12.5.1](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.1) o posteriores. Antes de ejecutar este ejemplo, asegúrese de que ha habilitado el control de versiones para la cuenta de almacenamiento.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobVersions":::

## <a name="next-steps"></a>Pasos siguientes

- [Control de versiones de blobs](versioning-overview.md)
- [Eliminación temporal de blobs de Azure Storage](./soft-delete-blob-overview.md)
