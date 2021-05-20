---
title: 'Condiciones de asignación de roles de Azure de ejemplo (versión preliminar): Azure RBAC'
titleSuffix: Azure Storage
description: Condiciones de asignación de roles de Azure de ejemplo para el control de acceso basado en atributos de Azure (Azure ABAC).
services: storage
author: rolyon
ms.service: storage
ms.topic: conceptual
ms.author: rolyon
ms.reviewer: ''
ms.subservice: common
ms.date: 05/06/2021
ms.openlocfilehash: 062be43bda26169d27740777bdb60d335c9923e6
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490079"
---
# <a name="example-azure-role-assignment-conditions-preview"></a>Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se muestran algunos ejemplos de condiciones de asignación de roles.

## <a name="prerequisites"></a>Requisitos previos

Para obtener información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones](../../role-based-access-control/conditions-prerequisites.md).

## <a name="example-1-read-access-to-blobs-with-a-tag"></a>Ejemplo 1: Acceso de lectura a blobs con una etiqueta

Esta condición permite a los usuarios leer blobs con una clave de etiqueta de índice de blob de Project y un valor de etiqueta de Cascade. No se permitirán los intentos de acceso a los blobs sin esta etiqueta clave-valor.

> [!TIP]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md).

![Diagrama de la condición de ejemplo 1 que muestra el acceso de lectura a algún blob con una etiqueta.](./media/storage-auth-abac-examples/example-1.png)

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

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Leer contenido de un blob con condiciones de etiqueta |
| Origen del atributo | Resource |
| Atributo | Etiquetas de índice de blobs [Valores en clave] |
| Clave | {keyName} |
| Operator | StringEquals |
| Value | {keyValue} |

![Captura de pantalla del editor de condición de ejemplo 1 en Azure Portal.](./media/storage-auth-abac-examples/example-1-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Cascade'))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
Get-AzStorageBlob -Container <containerName> -Blob <blobName> -Context $bearerCtx 
```

## <a name="example-2-new-blobs-must-include-a-tag"></a>Ejemplo 2: Los nuevos blobs deben incluir una etiqueta

Esta condición requiere que los nuevos blobs incluyan una clave de etiqueta de índice de blob de Project y un valor de etiqueta de Cascade.

> [!TIP]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md).

Hay dos permisos que le permiten crear nuevos blobs, por lo que debe tener como destino ambos. Debe agregar esta condición a todas las asignaciones de roles que incluyan uno de los permisos siguientes.

- /blobs/write (crear o actualizar) 
- /blobs/add/action (crear)

![Diagrama de la condición de ejemplo 2 que muestra que los nuevos blobs deben incluir una etiqueta.](./media/storage-auth-abac-examples/example-2.png)

```
(
 (
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'})
 )
 OR 
 (
  @Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade'
 )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Escritura en un blob con etiquetas de índice de blobs<br/>Escritura de contenido en un blob con etiquetas de índice de blobs |
| Origen del atributo | Solicitud |
| Atributo | Etiquetas de índice de blobs [Valores en clave] |
| Clave | {keyName} |
| Operator | StringEquals |
| Value | {keyValue} |

![Captura de pantalla del editor de la condición 1 del ejemplo 2 en Azure Portal.](./media/storage-auth-abac-examples/example-2-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'})) OR (@Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] StringEquals 'Cascade'))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$localSrcFile = # path to an example file, can be an empty txt
$ungrantedTag = @{'Project'='Baker'}
$grantedTag = @{'Project'='Cascade'}
# Get new context for request
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# try ungranted tags
$content = Set-AzStorageBlobContent -File $localSrcFile -Container example2 -Blob "Example2.txt" -Tag $ungrantedTag -Context $bearerCtx
# try granted tags
$content = Set-AzStorageBlobContent -File $localSrcFile -Container example2 -Blob "Example2.txt" -Tag $grantedTag -Context $bearerCtx
```

## <a name="example-3-existing-blobs-must-have-tag-keys"></a>Ejemplo 3: Los blobs existentes deben tener claves de etiqueta

Esta condición requiere que los blobs existentes se etiqueten con al menos una de las claves de etiqueta de índice de blobs permitidas, Project o Program. Esta condición es útil para agregar gobernanza a los blobs existentes.

> [!TIP]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md).

Hay dos permisos que le permiten actualizar etiquetas en blobs existentes, por lo que debe tener como destino ambos. Debe agregar esta condición a todas las asignaciones de roles que incluyan uno de los permisos siguientes.

- /blobs/write (actualizar o crear, no se puede excluir crear)
- /blobs/tags/write

![Diagrama de la condición de ejemplo 3 que muestra que los blobs existentes deben tener claves de etiqueta.](./media/storage-auth-abac-examples/example-3.png)

```
(
 (
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write'})
 )
 OR 
 (
  @Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags&$keys$&] ForAllOfAnyValues:StringEquals {'Project', 'Program'}
 )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Escritura en un blob con etiquetas de índice de blobs<br/>Escritura de etiquetas de índice de blobs |
| Origen del atributo | Solicitud |
| Atributo | Etiquetas de índice de blobs [claves] |
| Operator | ForAllOfAnyValues:StringEquals |
| Value | {keyName1}<br/>{keyName2} |

![Captura de pantalla del editor de la condición 1 del ejemplo 3 en Azure Portal.](./media/storage-auth-abac-examples/example-3-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write'})) OR (@Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags&`$keys`$&] ForAllOfAnyValues:StringEquals {'Project', 'Program'}))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$localSrcFile = # path to an example file, can be an empty txt
$ungrantedTag = @{'Mode'='Baker'}
$grantedTag = @{'Program'='Alpine';'Project'='Cascade'}
# Get new context for request
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# try ungranted tags
$content = Set-AzStorageBlobContent -File $localSrcFile -Container example3 -Blob "Example3.txt" -Tag $ungrantedTag -Context $bearerCtx
# try granted tags
$content = Set-AzStorageBlobContent -File $localSrcFile -Container example3 -Blob "Example3.txt" -Tag $grantedTag -Context $bearerCtx
```

## <a name="example-4-existing-blobs-must-have-a-tag-key-and-values"></a>Ejemplo 4: Los blobs existentes deben tener una clave de etiqueta y valores

Esta condición requiere que todos los blobs existentes tengan una clave de etiqueta de índice de blobs de Project y valores de etiqueta de Cascade, Baker o Skagit. Esta condición es útil para agregar gobernanza a los blobs existentes.

> [!TIP]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md).

Hay dos permisos que le permiten actualizar etiquetas en blobs existentes, por lo que debe tener como destino ambos. Debe agregar esta condición a todas las asignaciones de roles que incluyan uno de los permisos siguientes.

- /blobs/write (actualizar o crear, no se puede excluir crear)
- /blobs/tags/write

![Diagrama de la condición de ejemplo 4 que muestra que los blobs existentes deben tener una clave de etiqueta y valores.](./media/storage-auth-abac-examples/example-4.png)

```
(
 (
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write'})
 )
 OR 
 (
  @Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags&$keys$&] ForAnyOfAnyValues:StringEquals {'Project'}
  AND
  @Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] ForAllOfAnyValues:StringEquals {'Cascade', 'Baker', 'Skagit'}
 )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Escritura en un blob con etiquetas de índice de blobs<br/>Escritura de etiquetas de índice de blobs |
| Origen del atributo | Solicitud |
| Atributo | Etiquetas de índice de blobs [claves] |
| Operator | ForAnyOfAnyValues:StringEquals |
| Value | {keyName} |
| Operator | And |
| **Expresión 2** |  |
| Origen del atributo | Solicitud |
| Atributo | Etiquetas de índice de blobs [Valores en clave] |
| Clave | {keyName} |
| Operator | ForAllOfAnyValues:StringEquals |
| Value | {keyValue1}<br/>{keyValue2}<br/>{keyValue3} |

![Captura de pantalla del editor de la condición 1 del ejemplo 4 en Azure Portal.](./media/storage-auth-abac-examples/example-4-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Write.WithTagHeaders'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write'})) OR (@Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags&`$keys`$&] ForAnyOfAnyValues:StringEquals {'Project'} AND @Request[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<`$key_case_sensitive`$>] ForAllOfAnyValues:StringEquals {'Cascade', 'Baker', 'Skagit'}))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$localSrcFile = <pathToLocalFile>
$ungrantedTag = @{'Project'='Alpine'}
$grantedTag1 = @{'Project'='Cascade'}
$grantedTag2 = @{'Project'='Baker'}
$grantedTag3 = @{'Project'='Skagit'}
# Get new context for request
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# try ungranted tags
Set-AzStorageBlobTag -Container example4 -Blob "Example4.txt" -Tag $ungrantedTag -Context $bearerCtx
# try granted tags
Set-AzStorageBlobTag -Container example4 -Blob "Example4.txt" -Tag $grantedTag1 -Context $bearerCtx
Set-AzStorageBlobTag -Container example4 -Blob "Example4.txt" -Tag $grantedTag2 -Context $bearerCtx
Set-AzStorageBlobTag -Container example4 -Blob "Example4.txt" -Tag $grantedTag3 -Context $bearerCtx
```

## <a name="example-5-read-write-or-delete-blobs-in-named-containers"></a>Ejemplo 5: Lectura, escritura o eliminación de blobs en contenedores con nombre

Esta condición permite a los usuarios leer, escribir o eliminar blobs en contenedores de almacenamiento denominados blobs-example-container. Esta condición es útil para compartir contenedores de almacenamiento específicos con otros usuarios de una suscripción.

Hay cuatro permisos para leer, escribir y eliminar blobs existentes, por lo que debe tener como destino todos los permisos. Debe agregar esta condición a todas las asignaciones de roles que incluyan uno de los permisos siguientes.

- /blobs/delete
- /blobs/read
- /blobs/write (actualizar o crear)
- /blobs/add/action (crear)

Las suboperaciones no se usan en esta condición porque subOperation solo se necesita cuando se crean condiciones basadas en etiquetas.

![Diagrama de la condición de ejemplo 5 que muestra los blobs de lectura, escritura o eliminación en contenedores con nombre.](./media/storage-auth-abac-examples/example-5.png)

```
(
 (
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action'})
 )
 OR 
 (
  @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'
 )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Eliminar un blob<br/>Leer un blob<br/>Escribir en un blob<br/>Crear un blob o una instantánea, o anexar datos |
| Origen del atributo | Resource |
| Atributo | Nombre del contenedor |
| Operator | StringEquals |
| Value | {containerName} |

![Captura de pantalla del editor de la condición 1 del ejemplo 5 en Azure Portal.](./media/storage-auth-abac-examples/example-5-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$localSrcFile = <pathToLocalFile>
$grantedContainer = "blobs-example-container"
$ungrantedContainer = "ungranted"
# Get new context for request
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# Ungranted Container actions
$content = Set-AzStorageBlobContent -File $localSrcFile -Container $ungrantedContainer -Blob "Example5.txt" -Context $bearerCtx
$content = Get-AzStorageBlobContent -Container $ungrantedContainer -Blob "Example5.txt" -Context $bearerCtx
$content = Remove-AzStorageBlob -Container $ungrantedContainer -Blob "Example5.txt" -Context $bearerCtx
# Granted Container actions
$content = Set-AzStorageBlobContent -File $localSrcFile -Container $grantedContainer -Blob "Example5.txt" -Context $bearerCtx
$content = Get-AzStorageBlobContent -Container $grantedContainer -Blob "Example5.txt" -Context $bearerCtx
$content = Remove-AzStorageBlob -Container $grantedContainer -Blob "Example5.txt" -Context $bearerCtx
```

## <a name="example-6-read-access-to-blobs-in-named-containers-with-a-path"></a>Ejemplo 6: Acceso de lectura a blobs en contenedores con nombre con una ruta de acceso

Esta condición permite el acceso de lectura a contenedores de almacenamiento denominados blobs-example-container con una ruta de acceso de blob de solo readonly/*. Esta condición es útil para compartir partes específicas de contenedores de almacenamiento para el acceso de lectura con otros usuarios de la suscripción.

Debe agregar esta condición a todas las asignaciones de roles que incluyan el permiso siguiente.

- /blobs/read

![Diagrama de la condición de ejemplo 6 que muestra el acceso de lectura a blobs en contenedores con nombre con una ruta de acceso.](./media/storage-auth-abac-examples/example-6.png)

```
(
    (
        !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})
    )
    OR
    (
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container'
        AND
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path] StringLike 'readonly/*'
    )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Leer un blob |
| Origen del atributo | Resource |
| Atributo | Nombre del contenedor |
| Operator | StringEquals |
| Value | {containerName} |
| **Expresión 2** |  |
| Operator | And |
| Origen del atributo | Resource |
| Atributo | Ruta del blob |
| Operator | StringLike |
| Value | {pathString} |

![Captura de pantalla del editor de la condición 1 del ejemplo 6 en Azure Portal.](./media/storage-auth-abac-examples/example-6-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'blobs-example-container' AND @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path] StringLike 'readonly/*'))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$grantedContainer = "blobs-example-container"
# Get new context for request
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# Try to get ungranted blob
$content = Get-AzStorageBlobContent -Container $grantedContainer -Blob "Ungranted.txt" -Context $bearerCtx
# Try to get granted blob
$content = Get-AzStorageBlobContent -Container $grantedContainer -Blob "readonly/Example6.txt" -Context $bearerCtx
```

## <a name="example-7-write-access-to-blobs-in-named-containers-with-a-path"></a>Ejemplo 7: Acceso de lectura a blobs en contenedores con nombre con una ruta de acceso

Esta condición permite a un asociado (un invitado de Azure AD) colocar archivos en contenedores de almacenamiento denominados Contosocorp con una ruta de acceso de uploads/contoso/*. Esta condición es útil para permitir que otros usuarios coloquen datos en contenedores de almacenamiento.

Debe agregar esta condición a todas las asignaciones de roles que incluyan los permisos siguientes.

- /blobs/write (crear o actualizar)
- /blobs/add/action (crear)

![Diagrama de la condición de ejemplo 7 que muestra el acceso de escritura a blobs en contenedores con nombre con una ruta de acceso.](./media/storage-auth-abac-examples/example-7.png)

```
(
 (
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'})
  AND
  !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action'})
 )
 OR 
 (
  @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'contosocorp'
  AND
  @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path] StringLike 'uploads/contoso/*'
 )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Escribir en un blob<br/>Crear un blob o una instantánea, o anexar datos |
| Origen del atributo | Resource |
| Atributo | Nombre del contenedor |
| Operator | StringEquals |
| Value | {containerName} |
| **Expresión 2** |  |
| Operator | And |
| Origen del atributo | Resource |
| Atributo | Ruta del blob |
| Operator | StringLike |
| Value | {pathString} |

![Captura de pantalla del editor de la condición 1 del ejemplo 7 en Azure Portal.](./media/storage-auth-abac-examples/example-7-condition-1-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write'}) AND !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name] StringEquals 'contosocorp' AND @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path] StringLike 'uploads/contoso/*'))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$grantedContainer = "contosocorp"
$localSrcFile = <pathToLocalFile>
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# Try to set ungranted blob
$content = Set-AzStorageBlobContent -Container $grantedContainer -Blob "Example7.txt" -Context $bearerCtx -File $localSrcFile
# Try to set granted blob
$content = Set-AzStorageBlobContent -Container $grantedContainer -Blob "uploads/contoso/Example7.txt" -Context $bearerCtx -File $localSrcFile
```

## <a name="example-8-read-access-to-blobs-with-a-tag-and-a-path"></a>Ejemplo 8: Acceso de lectura a blobs con una etiqueta y una ruta de acceso

Esta condición permite a un usuario leer blobs con una clave de etiqueta de índice de blob de Program, un valor de etiqueta de Alpine y una ruta de acceso de blob de registros *. La ruta de acceso de blob de los registros* también incluye el nombre del blob.

> [!TIP]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md).

Debe agregar esta condición a todas las asignaciones de roles que incluyan el permiso siguiente.

- /blobs/read

![Diagrama de la condición de ejemplo 8 que muestra el acceso de lectura a algún blob con una etiqueta.](./media/storage-auth-abac-examples/example-8.png)

```
(
    (
        !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'}
        AND
        @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})
    )
    OR
    (
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Program<$key_case_sensitive$>] StringEquals 'Alpine'
    )
)
AND
(
    (
        !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})
    )
    OR
    (
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path] StringLike 'logs*'
    )
)
```

#### <a name="azure-portal"></a>Portal de Azure

Esta es la configuración para agregar esta condición mediante Azure Portal.

| Condición 1 | Configuración |
| --- | --- |
| Acciones | Leer contenido de un blob con condiciones de etiqueta |
| Origen del atributo | Resource |
| Atributo | Etiquetas de índice de blobs [valores en clave] |
| Clave | {keyName} |
| Operator | StringEquals |
| Value | {keyValue} |

![Captura de pantalla del editor de la condición 1 del ejemplo 8 en Azure Portal.](./media/storage-auth-abac-examples/example-8-condition-1-portal.png)

| Condición 2 | Configuración |
| --- | --- |
| Acciones | Leer un blob |
| Origen del atributo | Resource |
| Atributo | Ruta del blob |
| Operator | StringLike |
| Value | {pathString} |

![Captura de pantalla del editor de la condición 2 del ejemplo 8 en Azure Portal.](./media/storage-auth-abac-examples/example-8-condition-2-portal.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Aquí se muestra cómo agregar esta condición mediante Azure PowerShell.

```azurepowershell
$condition = "((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'} AND @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Program<`$key_case_sensitive`$>] StringEquals 'Alpine')) AND ((!(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})) OR (@Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path] StringLike 'logs*'))"
$testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectID
$testRa.Condition = $condition
$testRa.ConditionVersion = "2.0"
Set-AzRoleAssignment -InputObject $testRa -PassThru
```

Aquí se muestra cómo probar esta condición.

```azurepowershell
$grantedContainer = "contosocorp"
# Get new context for request
$bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
# Try to get ungranted blobs
# Wrong name but right tags
$content = Get-AzStorageBlobContent -Container $grantedContainer -Blob "AlpineFile.txt" -Context $bearerCtx
# Right name but wrong tags
$content = Get-AzStorageBlobContent -Container $grantedContainer -Blob "logsAlpine.txt" -Context $bearerCtx
# Try to get granted blob
$content = Get-AzStorageBlobContent -Container $grantedContainer -Blob "logs/AlpineFile.txt" -Context $bearerCtx
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure Portal (versión preliminar)](storage-auth-abac-portal.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md)
- [Sintaxis y formato de las condiciones de asignación de roles de Azure (versión preliminar)](../../role-based-access-control/conditions-format.md)
