---
title: 'Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure PowerShell (versión preliminar) y Azure ABAC'
titleSuffix: Azure Storage
description: Agregue una condición de asignación de roles para restringir el acceso a los blobs mediante Azure PowerShell y el control de acceso basado en atributos de Azure (Azure ABAC).
services: storage
author: rolyon
ms.service: storage
ms.topic: tutorial
ms.author: rolyon
ms.reviewer: ''
ms.subservice: common
ms.date: 05/06/2021
ms.openlocfilehash: 8d634cf7cb5a500e8ff36222419600b6059c9a74
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109489433"
---
# <a name="tutorial-add-a-role-assignment-condition-to-restrict-access-to-blobs-using-azure-powershell-preview"></a>Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure PowerShell (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En la mayoría de los casos, una asignación de roles concederá los permisos necesarios para los recursos de Azure. Sin embargo, en algunos casos es posible que desee proporcionar un control de acceso más preciso mediante la adición de una condición de asignación de roles.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar una condición a una asignación de roles
> * Restringir el acceso a blobs en función de una etiqueta de índice de blobs

## <a name="prerequisites"></a>Requisitos previos

Para obtener información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones.](../../role-based-access-control/conditions-prerequisites.md)

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

## <a name="step-1-install-prerequisites"></a>Paso 1: Requisitos previos de instalación

1. Abra una ventana de PowerShell.

1. Use [Get-InstalledModule](/powershell/module/powershellget/get-installedmodule) para comprobar las versiones de los módulos instalados.

    ```azurepowershell
    Get-InstalledModule -Name Az
    Get-InstalledModule -Name Az.Resources
    Get-InstalledModule -Name Az.Storage
    ```

1. Si es necesario, use [Install-Module](/powershell/module/powershellget/install-module) para instalar las versiones necesarias para los módulos [Az](https://www.powershellgallery.com/packages/Az/), [Az.Resources](https://www.powershellgallery.com/packages/Az.Resources/) y [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage/2.5.2-preview).

    ```azurepowershell
    Install-Module -Name Az -RequiredVersion 5.5.0
    Install-Module -Name Az.Resources -RequiredVersion 3.2.1
    Install-Module -Name Az.Storage -RequiredVersion 2.5.2-preview -AllowPrerelease
    ```

1. Cierre y vuelva a abrir PowerShell para actualizar la sesión.

## <a name="step-2-sign-in-to-azure"></a>Paso 2: Inicio de sesión en Azure

1. Use el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) y siga las instrucciones que se muestran para iniciar sesión en el directorio como [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) o [Propietario](../../role-based-access-control/built-in-roles.md#owner).

    ```azurepowershell
    Connect-AzAccount
    ```

1. Use [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) para enumerar todas las suscripciones.

    ```azurepowershell
    Get-AzSubscription
    ```

1. Determine el identificador de la suscripción e inicialice la variable.

    ```azurepowershell
    $subscriptionId = "<subscriptionId>"
    ```

1. Establezca la suscripción como suscripción activa.

    ```azurepowershell
    $context = Get-AzSubscription -SubscriptionId $subscriptionId
    Set-AzContext $context
    ```

1. Si aún no lo ha hecho, registre la suscripción para usar etiquetas de índice de blobs. Para más información, consulte [Registro de la suscripción (versión preliminar)](../blobs/storage-manage-find-blobs.md#register-your-subscription-preview).

## <a name="step-3-create-a-user"></a>Paso 3: Creación de un usuario

1. Use [New-AzureADUser](/powershell/module/azuread/new-azureaduser) para crear un usuario o buscar uno existente. En este tutorial se usa Chandra como ejemplo.

1. Inicialice la variable para el identificador de objeto del usuario.

    ```azurepowershell
    $userObjectId = "<userObjectId>"
    ```

## <a name="step-4-set-up-storage"></a>Paso 4: Configuración del almacenamiento

1. Use [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) para crear una cuenta de almacenamiento compatible con la característica de índice de blobs. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md#regional-availability-and-storage-account-support).

1. Use [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer) para crear un nuevo contenedor de blobs dentro de la cuenta de almacenamiento y establezca el nivel de acceso Público en **Privado (sin acceso anónimo)** .

1. Use [Set-AzStorageBlobContent](/powershell/module/az.storage/set-azstorageblobcontent) para cargar un archivo de texto en el contenedor.

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

    ```azurepowershell
    $resourceGroup = "<resourceGroup>"
    $storageAccountName = "<storageAccountName>"
    $containerName = "<containerName>"
    $blobNameCascade = "<blobNameCascade>"
    $blobNameBaker = "<blobNameBaker>"
    ```

## <a name="step-5-assign-a-role-with-a-condition"></a>Paso 5: Asignación de un rol con una condición

1. Inicialice las variables del rol [Lector de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader).

    ```azurepowershell
    $roleDefinitionName = "Storage Blob Data Reader"
    $roleDefinitionId = "2a2b9908-6ea1-4ae2-8e65-a410df84e7d1"
    ```

1. Inicialice el ámbito del grupo de recursos.

    ```azurepowershell
    $scope = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroup"
    ```

1. Inicialice la condición.

    ```azurepowershell
    $condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Cascade'))"
    ```

    En PowerShell, si la condición incluye un signo de dólar ($), debe agregarle un carácter de acento grave (\`) como prefijo. Por ejemplo, esta condición usa signos de dólar para delinear el nombre de la clave de etiqueta.

1. Inicialice la versión de la condición y la descripción.

    ```azurepowershell
    $conditionVersion = "2.0"
    $description = "Read access to blobs with the tag Project=Cascade"
    ```

1. Use [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) para asignar el rol [Lector de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) con una condición para el usuario en un ámbito del grupo de recursos.

    ```azurepowershell
    New-AzRoleAssignment -ObjectId $userObjectId -Scope $scope -RoleDefinitionId $roleDefinitionId -Description $description -Condition $condition -ConditionVersion $conditionVersion
    ```

    Este es un ejemplo de la salida:
    
    ```azurepowershell
    RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microso
                         ft.Authorization/roleAssignments/<roleAssignmentId>
    Scope              : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    DisplayName        : Chandra
    SignInName         : chandra@contoso.com
    RoleDefinitionName : Storage Blob Data Reader
    RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
    ObjectId           : <userObjectId>
    ObjectType         : User
    CanDelegate        : False
    Description        : Read access to blobs with the tag Project=Cascade
    ConditionVersion   : 2.0
    Condition          : ((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND
                         @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR
                         (@Resource[Microsoft.Storage/storageAccounts/blobServices/co
                         ntainers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade'))
    ```

## <a name="step-6-optional-view-the-condition-in-the-azure-portal"></a>Paso 6: (Opcional) Ver la condición en Azure Portal

1. En Azure Portal, abra el grupo de recursos.

1. Haga clic en **Control de acceso (IAM).**

1. En la pestaña Asignaciones de roles, busque la asignación de roles.

1. En la columna **Condición**, haga clic en **Ver/Editar** para ver la condición.

    ![Captura de pantalla de Adición de una condición de asignación de roles en Azure Portal.](./media/shared/condition-view.png)

## <a name="step-7-test-the-condition"></a>Paso 7: Prueba de la condición

1. Abra una nueva ventana de PowerShell.

1. Use [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión como Chandra.

    ```azurepowershell
    Connect-AzAccount
    ```

1. Inicialice las siguientes variables con los nombres que ha usado.

    ```azurepowershell
    $storageAccountName = "<storageAccountName>"
    $containerName = "<containerName>"
    $blobNameBaker = "<blobNameBaker>"
    $blobNameCascade = "<blobNameCascade>"
    ```

1. Use [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext) para crear un contexto específico para acceder a la cuenta de almacenamiento más fácilmente.

    ```azurepowershell
    $bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
    ```

1. Use [Get-AzStorageBlob](/powershell/module/az.storage/get-azstorageblob) para intentar leer el archivo del proyecto Baker.

    ```azurepowershell
    Get-AzStorageBlob -Container $containerName -Blob $blobNameBaker -Context $bearerCtx 
    ```

    Este es un ejemplo de la salida. Tenga en cuenta que **no puede** leer el archivo debido a la condición que ha agregado.
    
    ```azurepowershell
    Get-AzStorageBlob : This request is not authorized to perform this operation using this permission. HTTP Status Code:
    403 - HTTP Error Message: This request is not authorized to perform this operation using this permission.
    ErrorCode: AuthorizationPermissionMismatch
    ErrorMessage: This request is not authorized to perform this operation using this permission.
    RequestId: <requestId>
    Time: Sat, 24 Apr 2021 13:26:25 GMT
    At line:1 char:1
    + Get-AzStorageBlob -Container $containerName -Blob $blobNameBaker -Con ...
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : CloseError: (:) [Get-AzStorageBlob], StorageException
        + FullyQualifiedErrorId : StorageException,Microsoft.WindowsAzure.Commands.Storage.Blob.Cmdlet.GetAzureStorageBlob
       Command
    ```
    
1. Lea el archivo del proyecto Cascade.

    ```azurepowershell
    Get-AzStorageBlob -Container $containerName -Blob $blobNameCascade -Context $bearerCtx 
    ```

    Este es un ejemplo de la salida. Observe que sí puede leer el archivo porque tiene la etiqueta Project=Cascade.
    
    ```azurepowershell
       AccountName: <storageAccountName>, ContainerName: <containerName>
    
    Name                 BlobType  Length          ContentType                    LastModified         AccessTier SnapshotT
                                                                                                                  ime
    ----                 --------  ------          -----------                    ------------         ---------- ---------
    CascadeFile.txt      BlockBlob 7               text/plain                     2021-04-24 05:35:24Z Hot
    ```

## <a name="step-8-optional-edit-the-condition"></a>Paso 8: (Opcional) Edición de la condición

1. En la otra ventana de PowerShell, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) para obtener la asignación de roles que ha agregado.

    ```azurepowershell
    $testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectId
    ```

1. Edite la condición.

    ```azurepowershell
    $condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Cascade' OR @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Baker'))"
    ```

1. Inicialice la condición y la descripción.

    ```azurepowershell
    $testRa.Condition = $condition
    $testRa.Description = "Read access to blobs with the tag Project=Cascade or Project=Baker"
    ```

1. Use [Set-AzRoleAssignment](/powershell/module/az.resources/set-azroleassignment) para actualizar la condición de la asignación de roles.

    ```azurepowershell
    Set-AzRoleAssignment -InputObject $testRa -PassThru
    ```

    Este es un ejemplo de la salida:

    ```azurepowershell
    RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microso
                         ft.Authorization/roleAssignments/<roleAssignmentId>
    Scope              : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    DisplayName        : Chandra
    SignInName         : chandra@contoso.com
    RoleDefinitionName : Storage Blob Data Reader
    RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
    ObjectId           : <userObjectId>
    ObjectType         : User
    CanDelegate        : False
    Description        : Read access to blobs with the tag Project=Cascade or Project=Baker
    ConditionVersion   : 2.0
    Condition          : ((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND
                         @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR
                         (@Resource[Microsoft.Storage/storageAccounts/blobServices/co
                         ntainers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade' OR @Resource[Microsoft.S
                         torage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>]
                         StringEquals 'Baker'))
    ```
    
## <a name="step-9-clean-up-resources"></a>Paso 9: Limpieza de recursos

1. Use [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) para quitar la asignación de roles y la condición que ha agregado.

    ```azurepowershell
    Remove-AzRoleAssignment -ObjectId $userObjectId -RoleDefinitionName $roleDefinitionName -ResourceGroupName $resourceGroup
    ```

1. Elimine la cuenta de almacenamiento que ha creado.

1. Elimine el usuario que ha creado.

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo](storage-auth-abac-examples.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md)
- [Sintaxis y formato de las condiciones de asignación de roles de Azure](../../role-based-access-control/conditions-format.md)
