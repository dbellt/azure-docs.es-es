---
title: Uso de .NET para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2
description: Use .NET para administrar listas de control de acceso (ACL) en cuentas de almacenamiento que tengan habilitado un espacio de nombres jerárquico (HNS).
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-csharp
ms.openlocfilehash: 156af8eb034f31b9dda11ee39b9030b6d016daff
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632590"
---
# <a name="use-net-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Uso de .NET para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar .NET para obtener, establecer y actualizar las listas de control de acceso de directorios y archivos.

La herencia de ACL ya está disponible para los nuevos elementos secundarios que se crean en un directorio primario. Ahora bien, también se pueden agregar, actualizar y quitar listas de control de acceso de forma recursiva en los elementos secundarios existentes de un directorio primario sin tener que realizar estos cambios individualmente para cada elemento secundario.

[Paquete (NuGet)](https://www.nuget.org/packages/Azure.Storage.Files.DataLake) | [Ejemplos](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake) | [Ejemplo recursivo de ACL](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) | [Referencia de API](/dotnet/api/azure.storage.files.datalake) | [Asignación de Gen1 a Gen2](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake/GEN1_GEN2_MAPPING.md) | [Ofrecer comentarios](https://github.com/Azure/azure-sdk-for-net/issues)

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](create-data-lake-storage-account.md) instrucciones para crear uno.

- CLI de Azure versión `2.6.0` o posterior.

- Uno de los siguientes permisos de seguridad:

  - Una [entidad de seguridad](../../role-based-access-control/overview.md#security-principal) aprovisionada de Azure Active Directory a la que se ha asignado el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito del contenedor de destino, el grupo de recursos primario o la suscripción.  

  - El usuario propietario del contenedor o directorio de destino al que va a aplicar la configuración de ACL. Para establecer listas de control de acceso de forma recursiva, se incluyen todos los elementos secundarios en el contenedor o el directorio de destino.
  
  - Clave de la cuenta de almacenamiento.

## <a name="set-up-your-project"></a>Configurar su proyecto

Para empezar, instale el paquete NuGet [Azure.Storage.Files.DataLake](https://www.nuget.org/packages/Azure.Storage.Files.DataLake/).

1. Abra una ventana de comando (por ejemplo: Windows PowerShell).

2. En el directorio del proyecto, instale el paquete Azure.Storage.Files.DataLake de la versión preliminar mediante el comando `dotnet add package`.

   ```console
   dotnet add package Azure.Storage.Files.DataLake -v 12.6.0 -s https://pkgs.dev.azure.com/azure-sdk/public/_packaging/azure-sdk-for-net/nuget/v3/index.json
   ```

   A continuación, agregue estas instrucciones USING al inicio del archivo de código.

   ```csharp
   using Azure;
   using Azure.Core;
   using Azure.Storage;
   using Azure.Storage.Files.DataLake;
   using Azure.Storage.Files.DataLake.Models;
   using System.Collections.Generic;
   using System.Threading.Tasks;
   ```

## <a name="connect-to-the-account"></a>Conexión con la cuenta

Para usar los fragmentos de código de este artículo, tiene que crear una instancia de [DataLakeServiceClient](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) que represente la cuenta de almacenamiento. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Conexión con Azure Active Directory (AD)

> [!NOTE]
> Si usa Azure Active Directory (Azure AD) para autorizar el acceso, asegúrese de que la entidad de seguridad tenga asignado el [rol Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Para obtener más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, consulte [Modelo de control de acceso de Azure Data Lake Storage Gen2](./data-lake-storage-access-control-model.md).

Puede usar la [biblioteca cliente de identidad de Azure para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) para autenticar la aplicación con Azure AD.

Después de instalar el paquete, agregue esta instrucción "using" al principio del archivo de código.

```csharp
using Azure.Identity;
```

Obtenga un identificador de cliente, un secreto de cliente y un identificador de inquilino. Para ello, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../common/storage-auth-aad-app.md). Como parte de ese proceso, tendrá que asignar uno de los siguientes roles de [control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md) a la entidad de seguridad. 

|Role|Capacidad de configuración de ACL|
|--|--|
|[Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Todos los directorios y archivos de la cuenta.|
|[Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Solo los directorios y archivos que pertenecen a la entidad de seguridad.|

En este ejemplo se crea una instancia de [DataLakeServiceClient](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) con un identificador de cliente, un secreto de cliente y un identificador de inquilino.  

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Authorize_DataLake.cs" id="Snippet_AuthorizeWithAAD":::

> [!NOTE]
> Para ver más ejemplos, consulte la documentación de la [biblioteca cliente de identidad de Azure para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity).

### <a name="connect-by-using-an-account-key"></a>Conexión con una clave de cuenta

Es la manera más sencilla de conectarse a una cuenta.

En este ejemplo se crea una instancia de [DataLakeServiceClient](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) con una clave de cuenta.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Authorize_DataLake.cs" id="Snippet_AuthorizeWithKey":::

## <a name="set-acls"></a>Establecimiento de listas de control de acceso

Cuando *establece* una ACL, debe **reemplazar** toda la ACL, incluidas todas sus entradas. Si quiere cambiar el nivel de permiso de una entidad de seguridad o agregar una nueva entidad de seguridad a la ACL sin que esto afecte a otras entradas existentes, debe *actualizar* la ACL en su lugar. Para actualizar una ACL en lugar de reemplazarla, consulte la sección [Actualización de ACL](#update-acls) de este artículo.  

Si elige *establecer* la lista de control de acceso, debe agregar una entrada para el usuario propietario, otra para el grupo propietario y una tercera para los demás usuarios. Para más información sobre el usuario propietario, el grupo propietario y el resto de usuarios, consulte [Usuarios e identidades](data-lake-storage-access-control.md#users-and-identities).

Esta sección le muestra cómo:

- Establecer la ACL de un directorio
- Establecer la ACL de un archivo
- Establecimiento de listas de control de acceso de forma recursiva

### <a name="set-the-acl-of-a-directory"></a>Establecer la ACL de un directorio

Obtenga la lista de control de acceso (ACL) de un directorio al llamar al método [DataLakeDirectoryClient.GetAccessControlAsync](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.getaccesscontrolasync) y establezca dicha lista con una llamada al método [DataLakeDirectoryClient.SetAccessControlList](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist).

En este ejemplo se obtiene y después se establece la ACL de un directorio denominado `my-directory`. La cadena `user::rwx,group::r-x,other::rw-` concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_ACLDirectory":::

También puede obtener y establecer la ACL del directorio raíz de un contenedor. Para obtener el directorio raíz, pase una cadena vacía (`""`) en el método [DataLakeFileSystemClient.GetDirectoryClient](/dotnet/api/azure.storage.files.datalake.datalakefilesystemclient.getdirectoryclient).

### <a name="set-the-acl-of-a-file"></a>Establecer la ACL de un archivo

Obtenga la lista de control de acceso (ACL) de un archivo al llamar al método [DataLakeFileClient.GetAccessControlAsync](/dotnet/api/azure.storage.files.datalake.datalakefileclient.getaccesscontrolasync) y establezca dicha lista con una llamada al método [DataLakeFileClient.SetAccessControlList](/dotnet/api/azure.storage.files.datalake.datalakefileclient.setaccesscontrollist).

En este ejemplo se obtiene y después se establece la ACL de un archivo denominado `my-file.txt`. La cadena `user::rwx,group::r-x,other::rw-` concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_FileACL":::

### <a name="set-acls-recursively"></a>Establecimiento de listas de control de acceso de forma recursiva

Establezca las listas de control de acceso de forma recursiva llamando al método **DataLakeDirectoryClient.SetAccessControlRecursiveAsync**. Pase a este método una [lista](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Cada [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) define una entrada de la ACL.

Si desea establecer una entrada de la ACL **predeterminada**, puede establecer la propiedad [PathAccessControlItem.DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) en **true**. 

En este ejemplo se establece la ACL de un directorio denominado `my-parent-directory`. Este método acepta un parámetro booleano denominado `isDefaultScope` que especifica si se debe establecer la ACL predeterminada. Ese parámetro se utiliza en el constructor de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Las entradas de la ACL conceden al usuario propietario permisos de lectura, escritura y ejecución, permisos solo de lectura y ejecución al grupo propietario, y ningún permiso al resto. La última entrada de ACL de este ejemplo proporciona a un usuario específico con el identificador de objeto "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" permisos de lectura y ejecución.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_SetACLRecursively":::

Para ver un ejemplo que establezca ACL de forma recursiva en lotes especificando un tamaño de lote, consulte el [ejemplo](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) de .NET.

## <a name="update-acls"></a>Actualización de ACL

Cuando *actualiza* una ACL, modifica la ACL en lugar de reemplazarla. Por ejemplo, puede agregar una nueva entidad de seguridad a la ACL sin que esto afecte a otras entidades de seguridad que se enumeran en la ACL.  Para reemplazar la ACL en lugar de actualizarla, consulte la sección [Establecimiento de listas de control de acceso](#set-acls) de este artículo.

Esta sección le muestra cómo:

- Actualización de una ACL
- Actualización de listas de control de acceso de forma recursiva

### <a name="update-an-acl"></a>Actualización de una ACL

En primer lugar, para obtener la ACL de un directorio, llame al método [DataLakeDirectoryClient.GetAccessControlAsync](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.getaccesscontrolasync). Copie la lista de entradas de ACL en una nueva [lista](/dotnet/api/system.collections.generic.list-1) de objetos [PathAccessControl](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrol). A continuación, busque la entrada que desea actualizar y reemplácela en la lista. Establezca la ACL mediante una llamada al método [DataLakeDirectoryClient.SetAccessControlList](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist).

En este ejemplo se actualiza la ACL raíz de un contenedor mediante la sustitución de la entrada de ACL de todos los demás usuarios. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_UpdateACL":::

### <a name="update-acls-recursively"></a>Actualización de listas de control de acceso de forma recursiva

Para actualizar una ACL de forma recursiva, cree un nuevo objeto ACL con la entrada de la ACL que quiera actualizar y, a continuación, use ese objeto en la operación para actualizar la ACL. No debe obtener la ACL existente, simplemente proporcione las entradas de la ACL que se vayan a actualizar.

Actualice una ACL de forma recursiva llamando al método **DataLakeDirectoryClient.UpdateAccessControlRecursiveAsync**.  Pase a este método una [lista](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Cada [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) define una entrada de la ACL.

Si desea actualizar una entrada de la ACL **predeterminada**, puede establecer la propiedad [PathAccessControlItem.DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) en **true**.

En este ejemplo se actualiza una entrada de ACL con permiso de escritura. Este método acepta un parámetro booleano denominado `isDefaultScope` que especifica si se debe actualizar la ACL predeterminada. Ese parámetro se utiliza en el constructor de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_UpdateACLsRecursively":::

Para ver un ejemplo que actualice ACL de forma recursiva en lotes especificando un tamaño de lote, consulte el [ejemplo](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) de .NET.

## <a name="remove-acl-entries"></a>Eliminación de entradas de ACL

Puede quitar una o varias entradas de ACL. Esta sección le muestra cómo:

- Eliminación de una entrada de ACL
- Eliminación de las entradas de ACL de forma recursiva

### <a name="remove-an-acl-entry"></a>Eliminación de una entrada de ACL

En primer lugar, para obtener la ACL de un directorio, llame al método [DataLakeDirectoryClient.GetAccessControlAsync](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.getaccesscontrolasync). Copie la lista de entradas de ACL en una nueva [lista](/dotnet/api/system.collections.generic.list-1) de objetos [PathAccessControl](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrol). A continuación, busque la entrada que desea quitar y llame al método [Remove](/dotnet/api/system.collections.ilist.remove) de la colección. Establezca la ACL actualizada mediante una llamada al método [DataLakeDirectoryClient.SetAccessControlList](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist).

En este ejemplo se actualiza la ACL raíz de un contenedor mediante la sustitución de la entrada de ACL de todos los demás usuarios. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_RemoveACLEntry":::

### <a name="remove-acl-entries-recursively"></a>Eliminación de las entradas de ACL de forma recursiva

Para quitar entradas de ACL de forma recursiva, cree un nuevo objeto de ACL para la entrada de ACL que se va a quitar y, a continuación, use ese objeto en la operación para quitar la ACL. No debe obtener la ACL existente; simplemente proporcione las entradas de la ACL que se van a quitar. 

Elimine entradas de ACL llamando al método **DataLakeDirectoryClient.RemoveAccessControlRecursiveAsync**. Pase a este método una [lista](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Cada [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) define una entrada de la ACL. 

Si desea quitar una entrada de la ACL **predeterminada**, puede establecer la propiedad [PathAccessControlItem.DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) en **true**. 

En este ejemplo se quita una entrada de la ACL del directorio llamado `my-parent-directory`. Este método acepta un parámetro booleano denominado `isDefaultScope` que especifica si se debe quitar la entrada de la ACL predeterminada. Ese parámetro se utiliza en el constructor de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_RemoveACLRecursively":::

Para ver un ejemplo que quite ACL de forma recursiva en lotes especificando un tamaño de lote, consulte el [ejemplo](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) de .NET.

## <a name="recover-from-failures"></a>Recuperación de errores

Al modificar listas de control de acceso de forma recursiva, es posible que se produzcan errores del entorno de ejecución o de los permisos. En el caso de los errores de tiempo de ejecución, reinicie el proceso desde el principio. Los errores de permisos pueden producirse si la entidad de seguridad no tiene permisos suficientes para modificar la ACL de un directorio o archivo que se encuentra en la jerarquía de directorios que se está modificando. Solucione el problema de permisos y, a continuación, elija reanudar el proceso desde el punto de error mediante un token de continuación, o bien reinicie el proceso desde el principio. No tiene que usar el token de continuación si prefiere reiniciar desde el principio. Puede volver a aplicar las entradas de ACL sin ningún efecto negativo.

Este ejemplo devuelve un token de continuación en caso de que se produzca un error. La aplicación puede volver a llamar a este método de ejemplo después de que se haya solucionado el error y pasar el token de continuación. Si se llama a este método de ejemplo por primera vez, la aplicación puede pasar un valor de `null` para el parámetro del token de continuación. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_ResumeContinuationToken":::

Para ver un ejemplo que establezca ACL de forma recursiva en lotes especificando un tamaño de lote, consulte el [ejemplo](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) de .NET.

Si desea que el proceso se complete sin que se interrumpa por errores de permisos, puede especificarlo.

Para asegurarse de que el proceso se completa sin interrupciones, pase un objeto **AccessControlChangedOptions** y establezca la propiedad **ContinueOnFailure** de ese objeto en ``true``.

En este ejemplo se establecen las entradas de ACL de forma recursiva. Si este código se encuentra un error de permiso, lo registra y continúa la ejecución. En este ejemplo se imprime el número de errores en la consola.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_ContinueOnFailure":::

Para ver un ejemplo que establezca ACL de forma recursiva en lotes especificando un tamaño de lote, consulte el [ejemplo](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) de .NET.

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>Consulte también

- [Documentación de referencia de API](/dotnet/api/azure.storage.files.datalake)
- [Paquete (NuGet)](https://www.nuget.org/packages/Azure.Storage.Files.DataLake)
- [Muestras](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake)
- [Asignación de Gen1 a Gen2](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake/GEN1_GEN2_MAPPING.md)
- [Problemas conocidos](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Envíenos sus comentarios](https://github.com/Azure/azure-sdk-for-net/issues)
- [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)
- [Listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)