---
title: 'Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs: Azure Storage'
description: Obtenga información sobre cómo usar la compatibilidad WORM (escribir una vez, leer muchas) con el almacenamiento de blobs (objetos) para almacenar datos en un estado no modificable durante un intervalo de tiempo especificado.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/26/2019
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8e8fd100b1c9dd630698944780263100c40575d4
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677324"
---
# <a name="set-and-manage-immutability-policies-for-blob-storage"></a>Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs

El almacenamiento inmutable para Azure Blob Storage permite a los usuarios almacenar objetos de datos críticos para la empresa en un estado WORM (escribir una vez, leer muchas). En este estado, los usuarios no pueden borrar ni modificar los datos durante el intervalo de tiempo especificado por el usuario. Mientras dure el intervalo de retención, se pueden crear y leer blobs, pero no modificar ni eliminar. El almacenamiento inmutable está disponible en las cuentas de uso general v2 y en las cuentas de Blob Storage de todas las regiones de Azure.

En este artículo se explica cómo establecer y administrar directivas de inmutabilidad y suspensiones legales relativas a los datos en Blob Storage mediante Azure Portal, PowerShell o la CLI de Azure. Para más información sobre el almacenamiento inmutable, vea [Almacenamiento inmutable de los datos críticos para la empresa en Azure Blob Storage](storage-blob-immutable-storage.md).

## <a name="set-retention-policies-and-legal-holds"></a>Establecimiento de directivas de retención y suspensiones legales

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Cree un nuevo contenedor o seleccione un contenedor existente para almacenar los blobs que se deben conservar en estado inmutable. El contenedor debe estar en una cuenta de uso general v2 o de Blob Storage.

2. Seleccione **Access policy** (Directiva de acceso) en la configuración del contenedor. Luego, seleccione **Agregar directiva** en **Almacenamiento de blobs inmutable**.

    ![Configuración del contenedor en el portal](media/storage-blob-immutability-policies-manage/portal-image-1.png)

3. Para habilitar la retención con duración definida, seleccione **Retención con duración definida** en el menú desplegable.

    !["Retención con duración definida" seleccionado en "Tipo de directiva"](media/storage-blob-immutability-policies-manage/portal-image-2.png)

4. Especifique el intervalo de retención en días (los valores aceptables oscilan entre 1 y 146 000 días).

    ![Cuadro "Actualizar el período de retención a"](media/storage-blob-immutability-policies-manage/portal-image-5-retention-interval.png)

    El estado inicial de la directiva es desbloqueada, lo que permite probar la característica y realizar cambios en la directiva antes de bloquearla. El bloqueo es esencial para el cumplimiento de normas como SEC 17a-4.

5. Bloquee la directiva. Haga clic con el botón derecho en los puntos suspensivos (**...**) y aparecerá el siguiente menú con acciones adicionales:

    !["Directiva de bloqueo" en el menú](media/storage-blob-immutability-policies-manage/portal-image-4-lock-policy.png)

6. Seleccione **Directiva de bloqueo** y confirme el bloqueo. Ahora la directiva se bloquea y no se puede eliminar; solo se permiten ampliaciones del intervalo de retención. No se permiten eliminaciones e invalidaciones del blob. 

    ![Confirmación de "Directiva de bloqueo" en el menú](media/storage-blob-immutability-policies-manage/portal-image-5-lock-policy.png)

7. Para habilitar suspensiones legales, seleccione **Agregar directiva**. Seleccione **Suspensión legal** en el menú desplegable.

    !["Suspensión legal" en el menú en "Tipo de directiva"](media/storage-blob-immutability-policies-manage/portal-image-legal-hold-selection-7.png)

8. Cree una suspensión legal con una o varias etiquetas.

    ![Cuadro "Nombre de etiqueta" en el tipo de directiva](media/storage-blob-immutability-policies-manage/portal-image-set-legal-hold-tags.png)

9. Para borrar una suspensión legal, quite la etiqueta de identificador de suspensión legal aplicada.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

La característica se incluye en los siguientes grupos de comandos: `az storage container immutability-policy` y `az storage container legal-hold`. Ejecute `-h` en ellos para ver los comandos.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

El módulo Az.Storage admite almacenamiento inmutable.  Para habilitar la característica, siga estos pasos:

1. Asegúrese de tener instalada la versión más reciente de PowerShellGet: `Install-Module PowerShellGet –Repository PSGallery –Force`.
2. Elimine todas las instalaciones anteriores de Azure PowerShell.
3. Instale Azure PowerShell: `Install-Module Az –Repository PSGallery –AllowClobber`.

El siguiente script de PowerShell de ejemplo se utiliza como referencia. Dicho script crea una nueva cuenta de almacenamiento y un contenedor. Luego, le muestra cómo establecer y eliminar suspensiones legales, además de cómo crear y bloquear una directiva de retención con duración definida (también conocida como directiva de inmutabilidad) y ampliar el intervalo de retención.

En primer lugar, cree una cuenta de Azure Storage:

```powershell
$resourceGroup = "<Enter your resource group>"
$storageAccount = "<Enter your storage account name>"
$container = "<Enter your container name>"
$location = "<Enter the storage account location>"

# Log in to Azure
Connect-AzAccount
Register-AzResourceProvider -ProviderNamespace "Microsoft.Storage"

# Create your Azure resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create your Azure storage account
$account = New-AzStorageAccount -ResourceGroupName $resourceGroup -StorageAccountName `
    $storageAccount -SkuName Standard_ZRS -Location $location -Kind StorageV2

# Create a new container using the context
$container = New-AzStorageContainer -Name $container -Context $account.Context

# List the containers in the account
Get-AzStorageContainer -Context $account.Context

# Remove a container
Remove-AzStorageContainer -Name $container -Context $account.Context
```

Establecimiento y eliminación de retenciones legales:

```powershell
# Set a legal hold
Add-AzRmStorageContainerLegalHold -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -Name $container -Tag <tag1>,<tag2>,...

# Clear a legal hold
Remove-AzRmStorageContainerLegalHold -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -Name $container -Tag <tag3>
```

Creación o actualización de directivas de inmutabilidad basadas en tiempo:

```powershell
# Create a time-based immutability policy
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container -ImmutabilityPeriod 10
```

Recuperación de directivas de inmutabilidad:

```powershell
# Get an immutability policy
Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container
```

Bloquee directivas de inmutabilidad (agregue `-Force` para descartar el mensaje):

```powershell
# Lock immutability policies
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container
Lock-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container `
    -Etag $policy.Etag
```

Ampliación de directivas de inmutabilidad:

```powershell
# Extend immutability policies
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container

Set-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy `
    $policy -ImmutabilityPeriod 11 -ExtendPolicy
```

Quite una directiva de inmutabilidad desbloqueada (agregue `-Force` para descartar el mensaje):

```powershell
# Remove an unlocked immutability policy
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container

Remove-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy $policy
```

---

## <a name="enabling-allow-protected-append-blobs-writes"></a>Habilitación de las escrituras de blobs en anexos protegidos

### <a name="portal"></a>[Portal](#tab/azure-portal)

![Permitir escrituras de anexiones adicionales](media/storage-blob-immutability-policies-manage/immutable-allow-additional-append-writes.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

La característica se incluye en los siguientes grupos de comandos: `az storage container immutability-policy` y `az storage container legal-hold`. Ejecute `-h` en ellos para ver los comandos.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# Create an immutability policy with appends allowed
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container -ImmutabilityPeriod 10 -AllowProtectedAppendWrite $true
```

---

## <a name="next-steps"></a>Pasos siguientes

[Almacenamiento inmutable de los datos críticos para la empresa en Azure Blob Storage](storage-blob-immutable-storage.md)
