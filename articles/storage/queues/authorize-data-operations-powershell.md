---
title: Ejecución de comandos de PowerShell con credenciales de Azure AD para acceder a los datos de cola
titleSuffix: Azure Storage
description: PowerShell admite el inicio de sesión con credenciales de Azure AD para ejecutar comandos en los datos de Azure Queue Storage. Se proporciona un token de acceso para la sesión y se usa para autorizar operaciones de llamada. Los permisos dependen del rol de Azure asignado a la entidad de seguridad de Azure AD.
author: tamram
services: storage
ms.author: tamram
ms.reviewer: ozgun
ms.date: 02/10/2021
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.openlocfilehash: 862c576ae86cd7d91a1bd73243caa0a68e3689c1
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111900546"
---
# <a name="run-powershell-commands-with-azure-ad-credentials-to-access-queue-data"></a>Ejecución de comandos de PowerShell con credenciales de Azure AD para acceder a los datos de cola

Azure Storage proporciona extensiones para PowerShell que le permiten iniciar sesión y ejecutar comandos de scripting con credenciales de Azure Active Directory (Azure AD). Al iniciar sesión en PowerShell con credenciales de Azure AD, se devuelve un token de acceso OAuth 2.0. PowerShell usa automáticamente ese token para autorizar las operaciones de datos posteriores en Queue Storage. Para las operaciones admitidas, ya no necesita pasar una clave de cuenta o token SAS con el comando.

Puede asignar permisos en los datos de cola a una entidad de seguridad de Azure AD mediante el control de acceso basado en roles de Azure (Azure RBAC). Para más información sobre los roles de Azure en Azure Storage, consulte [cómo administrar los derechos de acceso a los datos de Azure Storage mediante Azure RBAC](assign-azure-role-data-access.md).

## <a name="supported-operations"></a>Operaciones compatibles

Las extensiones de Azure Storage se admiten para las operaciones en datos de cola. Las operaciones a las que podrá llamar dependerán de los permisos que se concedan a la entidad de seguridad de Azure AD con la que inicie sesión en PowerShell. Los permisos para las colas se asignan mediante Azure RBAC. Por ejemplo, si se le asigna el rol de **lector de datos de cola**, puede ejecutar comandos de scripting que lean datos de una cola. Si se le asigna el rol de **colaborador de datos de cola**, podrá ejecutar comandos de scripting que lean, escriban o eliminen una cola, o los datos que contiene.

Para más información sobre los permisos requeridos para cada operación de Azure Storage en una cola, consulte la sección [Llamadas a operaciones de almacenamiento con tokens de OAuth](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens).

> [!IMPORTANT]
> Cuando una cuenta de almacenamiento está bloqueada con un bloqueo **ReadOnly** de Azure Resource Manager, no se permite la operación [Crear lista de claves](/rest/api/storagerp/storageaccounts/listkeys) para esa cuenta de almacenamiento. **Crear lista de claves** es una operación POST y todas las operaciones POST se impiden cuando se configura un bloqueo **ReadOnly** para la cuenta. Por esta razón, cuando la cuenta está bloqueada con un bloqueo **ReadOnly**, los usuarios que no disponen ya de las claves de cuenta deben usar las credenciales de Azure AD para acceder a los datos de la cola. En PowerShell, incluya el parámetro `-UseConnectedAccount` para crear un objeto **AzureStorageContext** con sus credenciales de Azure AD.

## <a name="call-powershell-commands-using-azure-ad-credentials"></a>Llamada a comandos de PowerShell mediante credenciales de Azure AD

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Si quiere usar Azure PowerShell para iniciar sesión y ejecutar operaciones posteriores en Azure Storage mediante credenciales de Azure AD, cree un contexto de almacenamiento para hacer referencia a la cuenta de almacenamiento e incluya el parámetro `-UseConnectedAccount`.

En el ejemplo siguiente se muestra cómo crear una cola en una nueva cuenta de almacenamiento desde Azure PowerShell mediante las credenciales de Azure AD. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

1. Inicie sesión en su cuenta de Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount):

    ```powershell
    Connect-AzAccount
    ```

    Para obtener más información sobre cómo iniciar sesión en Azure con PowerShell, consulte [Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps).

1. Cree un grupo de recursos de Azure mediante una llamada a [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

    ```powershell
    $resourceGroup = "sample-resource-group-ps"
    $location = "eastus"
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

1. Cree una cuenta de almacenamiento mediante una llamada a [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount).

    ```powershell
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
      -Name "<storage-account>" `
      -SkuName Standard_LRS `
      -Location $location `
    ```

1. Obtenga el contexto de la cuenta de almacenamiento que especifica la nueva cuenta de almacenamiento mediante una llamada a [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext). Cuando actúe en una cuenta de almacenamiento, puede hacer referencia al contexto en lugar de proporcionar varias veces las credenciales. Incluya el parámetro `-UseConnectedAccount` para llamar a las operaciones de datos posteriores con sus credenciales de Azure AD:

    ```powershell
    $ctx = New-AzStorageContext -StorageAccountName "<storage-account>" -UseConnectedAccount
    ```

1. Antes de crear la cola, asígnese a sí mismo el rol [Colaborador de datos de Queue Storage](../../role-based-access-control/built-in-roles.md#storage-queue-data-contributor). Aunque sea el propietario de la cuenta, necesita permisos explícitos para realizar operaciones de datos en la cuenta de almacenamiento. Para más información sobre la asignación de roles de Azure, consulte [Asignación de un rol de Azure para acceder a datos de cola](assign-azure-role-data-access.md).

    > [!IMPORTANT]
    > La propagación de las asignaciones de roles de Azure puede tardar unos minutos.

1. Cree una cola mediante una llamada a [New-AzStorageQueue](/powershell/module/az.storage/new-azstoragequeue). Dado que esta llamada usa el contexto creado en los pasos anteriores, la cola se crea con sus credenciales de Azure AD.

    ```powershell
    $queueName = "sample-queue"
    New-AzStorageQueue -Name $queueName -Context $ctx
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un rol de Azure para acceder a datos de cola](assign-azure-role-data-access.md)
- [Autorización del acceso a datos de blobs y colas con identidades administradas para los recursos de Azure](../common/storage-auth-aad-msi.md)
