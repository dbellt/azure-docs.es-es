---
title: 'Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante la CLI de Azure (versión preliminar) y Azure ABAC'
titleSuffix: Azure Storage
description: Agregue una condición de asignación de roles para restringir el acceso a los blobs mediante la CLI de Azure y el control de acceso basado en atributos de Azure (Azure ABAC).
services: storage
author: rolyon
ms.service: storage
ms.topic: tutorial
ms.author: rolyon
ms.reviewer: ''
ms.subservice: common
ms.date: 05/06/2021
ms.openlocfilehash: a2cf4c19e21ad13a2a79343713e9a0494eff0704
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109489462"
---
# <a name="tutorial-add-a-role-assignment-condition-to-restrict-access-to-blobs-using-azure-cli-preview"></a>Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante la CLI de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC de Azure se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


En la mayoría de los casos, una asignación de roles concederá los permisos necesarios para los recursos de Azure. Sin embargo, en algunos casos es posible que desee proporcionar un control de acceso más preciso mediante la adición de una condición de asignación de roles.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar una condición a una asignación de roles
> * Restringir el acceso a blobs en función de una etiqueta de índice de blobs

## <a name="prerequisites"></a>Requisitos previos

Para obtener información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones](../../role-based-access-control/conditions-prerequisites.md).

## <a name="condition"></a>Condición

En este tutorial, restringirá el acceso a blobs con una etiqueta específica. Por ejemplo, agrega una condición a una asignación de roles para que Chandra solo pueda leer archivos con la etiqueta Project=Cascade.

![Diagrama de asignación de roles con una condición.](./media/shared/condition-role-assignment-rg.png)

Si Chandra intenta leer un blob sin la etiqueta Project=Cascade, no se permite el acceso.

![Diagrama que muestra el acceso de lectura a blobs con la etiqueta Project=Cascade.](./media/shared/condition-access.png)

Este es el aspecto de la condición en el código:

```
(
    (
        !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'}
        AND
        @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})
    )
    OR
    (
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade'
    )
)
```

## <a name="step-1-sign-in-to-azure"></a>Paso 1: Inicio de sesión en Azure

1. Use el comando [az login](/cli/azure/reference-index#az_login) y siga las instrucciones que se muestran para iniciar sesión en el directorio como [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) o [Propietario](../../role-based-access-control/built-in-roles.md#owner).

    ```azurecli
    az login
    ```

1. Use [az account show](/cli/azure/account#az_account_show) para obtener el identificador de las suscripciones.

    ```azurecli
    az account show
    ```

1. Determine el identificador de la suscripción e inicialice la variable.

    ```azurecli
    subscriptionId="<subscriptionId>"
    ```

1. Si aún no lo ha hecho, registre la suscripción para usar etiquetas de índice de blobs. Para más información, consulte [Registro de la suscripción (versión preliminar)](../blobs/storage-manage-find-blobs.md#register-your-subscription-preview).

## <a name="step-2-create-a-user"></a>Paso 2: Creación de un usuario

1. Use [az ad user create](/cli/azure/ad/user#az_ad_user_create) crear un usuario o buscar uno existente. En este tutorial se usa Chandra como ejemplo.

1. Inicialice la variable para el identificador de objeto del usuario.

    ```azurecli
    userObjectId="<userObjectId>"
    ```

## <a name="step-3-set-up-storage"></a>Paso 3: Configuración del almacenamiento

Puede autorizar el acceso al almacenamiento de blobs desde la CLI de Azure con las credenciales de Azure AD o con la clave de acceso de la cuenta de almacenamiento. En este artículo se muestra cómo autorizar las operaciones de almacenamiento de blobs mediante Azure AD. Para más información, consulte [Inicio rápido: Creación, descarga y enumeración de blobs mediante la CLI de Azure](../blobs/storage-quickstart-blobs-cli.md).

1. Use [az storage account](/cli/azure/storage/account) para crear una cuenta de almacenamiento compatible con la característica de índice de blobs. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md#regional-availability-and-storage-account-support).

1. Use [az storage container](/cli/azure/storage/container) para crear un nuevo contenedor de blobs dentro de la cuenta de almacenamiento y establezca el nivel de acceso Público en **Privado (sin acceso anónimo)** .

1. Use [az storage blob upload](/cli/azure/storage/blob#az_storage_blob_upload) para cargar un archivo de texto en el contenedor.

1. Agregue la siguiente etiqueta de índice de blobs al archivo de texto. Para más información, consulte [Uso de etiquetas de índice de blobs (versión preliminar) para administrar y buscar datos en Azure Blob Storage](../blobs/storage-blob-index-how-to.md).

    > [!NOTE]
    > Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. 

    | Clave | Value |
    | --- | --- |
    | Project  | Cascade |

1. Cargue un segundo archivo de texto en el contenedor.

1. Agregue la siguiente etiqueta de índice de blobs al segundo archivo de texto.

    | Clave | Value |
    | --- | --- |
    | Project  | Baker |

1. Inicialice las siguientes variables con los nombres que ha usado.

    ```azurecli
    resourceGroup="<resourceGroup>"
    storageAccountName="<storageAccountName>"
    containerName="<containerName>"
    blobNameCascade="<blobNameCascade>"
    blobNameBaker="<blobNameBaker>"
    ```

## <a name="step-4-assign-a-role-with-a-condition"></a>Paso 4: Asignación de un rol con una condición

1. Inicialice las variables del rol [Lector de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader).

    ```azurecli
    roleDefinitionName="Storage Blob Data Reader"
    roleDefinitionId="2a2b9908-6ea1-4ae2-8e65-a410df84e7d1"
    ```

1. Inicialice el ámbito del grupo de recursos.

    ```azurecli
    scope="/subscriptions/$subscriptionId/resourceGroups/$resourceGroup"
    ```

1. Inicialice la condición.

    ```azurecli
    condition="((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<\$key_case_sensitive\$>] StringEquals 'Cascade'))"
    ```

    En Bash, si la expansión del historial está habilitada, es posible que vea el mensaje `bash: !: event not found` debido al signo de exclamación (!). En este caso, puede deshabilitar la expansión del historial con el comando `set +H`. Para volver a habilitar la expansión del historial, utilice `set -H`.

    En Bash, un signo de dólar ($) tiene un significado especial para la expansión. Si la condición incluye un signo de dólar ($), es posible que tenga que agregarle una barra diagonal inversa (\\) como prefijo. Por ejemplo, esta condición usa signos de dólar para rodear el nombre de la clave de etiqueta. Para más información sobre las reglas de comillas en Bash, consulte [Comillas dobles](https://www.gnu.org/software/bash/manual/html_node/Double-Quotes.html).

1. Inicialice la versión de la condición y la descripción.

    ```azurecli
    conditionVersion="2.0"
    description="Read access to blobs with the tag Project=Cascade"
    ```

1. Use [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) para asignar el rol [Lector de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) con una condición al usuario en un ámbito del grupo de recursos.

    ```azurecli
    az role assignment create --assignee-object-id $userObjectId --scope $scope --role $roleDefinitionId --description "$description" --condition "$condition" --condition-version $conditionVersion
    ```

    Este es un ejemplo de la salida:

    ```azurecli
    {
      "canDelegate": null,
      "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade'))",
      "conditionVersion": "2.0",
      "description": "Read access to blobs with the tag Project=Cascade",
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
      "name": "{roleAssignmentId}",
      "principalId": "{userObjectId}",
      "principalType": "User",
      "resourceGroup": "{resourceGroup}",
      "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
      "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    ```

## <a name="step-5-optional-view-the-condition-in-the-azure-portal"></a>Paso 5: (Opcional) Visualización de la condición en Azure Portal

1. En Azure Portal, abra el grupo de recursos.

1. Haga clic en **Control de acceso (IAM).**

1. En la pestaña Asignaciones de roles, busque la asignación de roles.

1. En la columna **Condición**, haga clic en **Ver/Editar** para ver la condición.

    ![Captura de pantalla de la adición de una condición de asignación de roles en Azure Portal](./media/shared/condition-view.png)

## <a name="step-6-test-the-condition"></a>Paso 6: Prueba de la condición

1. Abra una nueva ventana de comando

1. Utilice [az login](/cli/azure/reference-index#az_login) para iniciar sesión como Chandra.

    ```azurecli
    az login
    ```

1. Inicialice las siguientes variables con los nombres que ha usado.

    ```azurecli
    storageAccountName="<storageAccountName>"
    containerName="<containerName>"
    blobNameBaker="<blobNameBaker>"
    blobNameCascade="<blobNameCascade>"
    ```

1. Use [az storage blob show](/cli/azure/storage/blob#az_storage_blob_show) para intentar leer las propiedades del archivo para el proyecto Baker.

    ```azurecli
    az storage blob show --account-name $storageAccountName --container-name $containerName --name $blobNameBaker --auth-mode login
    ```

    Este es un ejemplo de la salida. Tenga en cuenta que **no puede** leer el archivo debido a la condición que ha agregado.
    
    ```azurecli
    You do not have the required permissions needed to perform this operation.
    Depending on your operation, you may need to be assigned one of the following roles:
        "Storage Blob Data Contributor"
        "Storage Blob Data Reader"
        "Storage Queue Data Contributor"
        "Storage Queue Data Reader"
    
    If you want to use the old authentication method and allow querying for the right account key, please use the "--auth-mode" parameter and "key" value.
    ```
    
1. Lea las propiedades del archivo para el proyecto Cascade.

    ```azurecli
    az storage blob show --account-name $storageAccountName --container-name $containerName --name $blobNameCascade --auth-mode login 
    ```

    Este es un ejemplo de la salida. Observe que puede leer las propiedades del archivo porque tiene la etiqueta Project=Cascade.
    
    ```azurecli
    {
      "container": "<containerName>",
      "content": "",
      "deleted": false,
      "encryptedMetadata": null,
      "encryptionKeySha256": null,
      "encryptionScope": null,
      "isAppendBlobSealed": null,
      "isCurrentVersion": null,
      "lastAccessedOn": null,
      "metadata": {},
      "name": "<blobNameCascade>",
      "objectReplicationDestinationPolicy": null,
      "objectReplicationSourceProperties": [],
      "properties": {
        "appendBlobCommittedBlockCount": null,
        "blobTier": "Hot",
        "blobTierChangeTime": null,
        "blobTierInferred": true,
        "blobType": "BlockBlob",
        "contentLength": 7,
        "contentRange": null,

      ...

    }
    ```

## <a name="step-7-optional-edit-the-condition"></a>Paso 7: (Opcional) Edición de la condición

1. En la otra ventana de comandos, use [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list) para obtener la asignación de roles que ha agregado.

    ```azurecli
    az role assignment list --assignee $userObjectId --resource-group $resourceGroup
    ```

    El resultado será similar al siguiente:
    
    ```azurecli
    [
      {
        "canDelegate": null,
        "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade'))",
        "conditionVersion": "2.0",
        "description": "Read access to blobs with the tag Project=Cascade",
        "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
        "name": "{roleAssignmentId}",
        "principalId": "{userObjectId}",
        "principalName": "chandra@contoso.com",
        "principalType": "User",
        "resourceGroup": "{resourceGroup}",
        "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "roleDefinitionName": "Storage Blob Data Reader",
        "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
        "type": "Microsoft.Authorization/roleAssignments"
      }
    ]
    ```

1. Cree un archivo JSON con el formato siguiente y actualice las propiedades `condition` y `description`.

    ```json
    {
        "canDelegate": null,
        "condition": "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Baker'))",
        "conditionVersion": "2.0",
        "description": "Read access to blobs with the tag Project=Cascade or Project=Baker",
        "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
        "name": "{roleAssignmentId}",
        "principalId": "{userObjectId}",
        "principalName": "chandra@contoso.com",
        "principalType": "User",
        "resourceGroup": "{resourceGroup}",
        "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "roleDefinitionName": "Storage Blob Data Reader",
        "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
        "type": "Microsoft.Authorization/roleAssignments"
    }
    ```

1. Use [az role assignment update](/cli/azure/role/assignment#az_role_assignment_update) para actualizar la condición de la asignación de roles.

    ```azurecli
    az role assignment update --role-assignment "./path/roleassignment.json"
    ```
    
## <a name="step-8-clean-up-resources"></a>Paso 8: Limpieza de recursos

1. Use [az role assignment delete](/cli/azure/role/assignment#az_role_assignment_delete) para quitar la asignación de roles y la condición que ha agregado.

    ```azurecli
    az role assignment delete --assignee $userObjectId --role "$roleDefinitionName" --resource-group $resourceGroup
    ```

1. Elimine la cuenta de almacenamiento que ha creado.

1. Elimine el usuario que ha creado.

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo](storage-auth-abac-examples.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md)
- [Sintaxis y formato de las condiciones de asignación de roles de Azure](../../role-based-access-control/conditions-format.md)
