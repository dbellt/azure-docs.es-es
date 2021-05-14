---
title: Uso de Java para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2
description: Use bibliotecas de Azure Storage para Java para administrar listas de control de acceso (ACL) en cuentas de almacenamiento que tengan habilitado el espacio de nombres jerárquico (HNS).
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.custom: devx-track-java
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.openlocfilehash: 75f65c87d47232aa0809475d38a82ffe68203df6
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633706"
---
# <a name="use-java-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Uso de Java para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar Java para obtener, establecer y actualizar las listas de control de acceso de directorios y archivos. 

La herencia de ACL ya está disponible para los nuevos elementos secundarios que se crean en un directorio primario. Ahora bien, también se pueden agregar, actualizar y quitar listas de control de acceso de forma recursiva en los elementos secundarios existentes de un directorio primario sin tener que realizar estos cambios individualmente para cada elemento secundario. 

[Paquete (Maven)](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake) | [Muestras](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake) | [Referencia de API](/java/api/overview/azure/storage-file-datalake-readme) | [Asignación de Gen1 a Gen2](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md) | [Envíenos sus comentarios](https://github.com/Azure/azure-sdk-for-java/issues)

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](create-data-lake-storage-account.md) instrucciones para crear uno.

- CLI de Azure versión `2.6.0` o posterior.

- Uno de los siguientes permisos de seguridad:

  - Una [entidad de seguridad](../../role-based-access-control/overview.md#security-principal) aprovisionada de Azure Active Directory a la que se ha asignado el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito del contenedor de destino, el grupo de recursos primario o la suscripción.  

  - El usuario propietario del contenedor o directorio de destino al que va a aplicar la configuración de ACL. Para establecer listas de control de acceso de forma recursiva, se incluyen todos los elementos secundarios en el contenedor o el directorio de destino.
  
  - Clave de la cuenta de almacenamiento.

## <a name="set-up-your-project"></a>Configurar su proyecto

Para empezar, abra [esta página](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake) y busque la versión más reciente de la biblioteca de Java. Después, abra el archivo *pom.xml* en el editor de texto. Agregue un elemento de dependencia que haga referencia a esa versión.

Si planea autenticar la aplicación cliente mediante Azure Active Directory (AD), agregue una dependencia a la biblioteca cliente de secreto de Azure. Consulte [Adición del paquete de la biblioteca cliente de secreto al proyecto](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity#adding-the-package-to-your-project).

A continuación, agregue estas instrucciones Imports al archivo de código.

```java
import com.azure.storage.common.StorageSharedKeyCredential;
import com.azure.storage.file.datalake.DataLakeDirectoryClient;
import com.azure.storage.file.datalake.DataLakeFileClient;
import com.azure.storage.file.datalake.DataLakeFileSystemClient;
import com.azure.storage.file.datalake.DataLakeServiceClient;
import com.azure.storage.file.datalake.DataLakeServiceClientBuilder;
import com.azure.storage.file.datalake.models.ListPathsOptions;
import com.azure.storage.file.datalake.models.PathItem;
import com.azure.storage.file.datalake.models.AccessControlChangeCounters;
import com.azure.storage.file.datalake.models.AccessControlChangeResult;
import com.azure.storage.file.datalake.models.AccessControlType;
import com.azure.storage.file.datalake.models.PathAccessControl;
import com.azure.storage.file.datalake.models.PathAccessControlEntry;
import com.azure.storage.file.datalake.models.PathPermissions;
import com.azure.storage.file.datalake.models.PathRemoveAccessControlEntry;
import com.azure.storage.file.datalake.models.RolePermissions;
import com.azure.storage.file.datalake.options.PathSetAccessControlRecursiveOptions;
```

## <a name="connect-to-the-account"></a>Conexión con la cuenta

Para usar los fragmentos de código de este artículo, tiene que crear una instancia de **DataLakeServiceClient** que represente la cuenta de almacenamiento. 

### <a name="connect-by-using-azure-active-directory-azure-ad"></a>Conexión con Azure Active Directory (Azure AD)

Puede usar la [biblioteca cliente de identidad de Azure para Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity) para autenticar la aplicación con Azure AD.

Obtenga un identificador de cliente, un secreto de cliente y un identificador de inquilino. Para ello, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../common/storage-auth-aad-app.md). Como parte de ese proceso, tendrá que asignar uno de los siguientes roles de [control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md) a la entidad de seguridad. 

|Role|Capacidad de configuración de ACL|
|--|--|
|[Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Todos los directorios y archivos de la cuenta.|
|[Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Solo los directorios y archivos que pertenecen a la entidad de seguridad.|

En este ejemplo se crea una instancia de **DataLakeServiceClient** con un identificador de cliente, un secreto de cliente y un identificador de inquilino.  

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/Authorize_DataLake.java" id="Snippet_AuthorizeWithAzureAD":::

> [!NOTE]
> Para ver más ejemplos, consulte la documentación [Biblioteca cliente de identidad de Azure para Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity).

### <a name="connect-by-using-an-account-key"></a>Conexión con una clave de cuenta

Es la manera más sencilla de conectarse a una cuenta. 

En este ejemplo se crea una instancia de **DataLakeServiceClient** con una clave de cuenta.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/Authorize_DataLake.java" id="Snippet_AuthorizeWithKey":::

## <a name="set-acls"></a>Establecimiento de listas de control de acceso

Cuando *establece* una ACL, debe **reemplazar** toda la ACL, incluidas todas sus entradas. Si quiere cambiar el nivel de permiso de una entidad de seguridad o agregar una nueva entidad de seguridad a la ACL sin que esto afecte a otras entradas existentes, debe *actualizar* la ACL en su lugar. Para actualizar una ACL en lugar de reemplazarla, consulte la sección [Actualización de ACL](#update-acls) de este artículo.  

Si elige *establecer* la lista de control de acceso, debe agregar una entrada para el usuario propietario, otra para el grupo propietario y una tercera para los demás usuarios. Para más información sobre el usuario propietario, el grupo propietario y el resto de usuarios, consulte [Usuarios e identidades](data-lake-storage-access-control.md#users-and-identities).

Esta sección le muestra cómo:

- Establecer la ACL de un directorio
- Establecer la ACL de un archivo
- Establecimiento de listas de control de acceso de forma recursiva 

### <a name="set-the-acl-of-a-directory"></a>Establecer la ACL de un directorio

En este ejemplo se obtiene y después se establece la ACL de un directorio denominado `my-directory`. En este ejemplo se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ManageDirectoryACLs":::

También puede obtener y establecer la ACL del directorio raíz de un contenedor. Para obtener el directorio raíz, pase una cadena vacía (`""`) en el método **DataLakeFileSystemClient.getDirectoryClient**.

### <a name="set-the-acl-of-a-file"></a>Establecer la ACL de un archivo

En este ejemplo se obtiene y después se establece la ACL de un archivo denominado `upload-file.txt`. En este ejemplo se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ManageFileACLs":::

### <a name="set-acls-recursively"></a>Establecimiento de listas de control de acceso de forma recursiva

Establezca ACL de forma recursiva mediante una llamada al método **DataLakeDirectoryClient.SetAccessControlRecursive**. Pase a este método una [lista](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) de objetos [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Cada objeto [PathAccessControlItem](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) define una entrada de la ACL.

Si desea establecer una entrada de ACL **predeterminada**, puede llamar al método **setDefaultScope** del objeto [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) y pasar un valor de **true**. 

En este ejemplo se establece la ACL de un directorio denominado `my-parent-directory`. Este método acepta un parámetro booleano denominado `isDefaultScope` que especifica si se debe establecer la ACL predeterminada. Ese parámetro se usa en cada llamada al método **setDefaultScope** del objeto [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Las entradas de la ACL conceden al usuario propietario permisos de lectura, escritura y ejecución, permisos solo de lectura y ejecución al grupo propietario, y ningún permiso al resto. La última entrada de ACL de este ejemplo proporciona a un usuario específico con el identificador de objeto "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" permisos de lectura y ejecución.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_SetACLRecursively":::

## <a name="update-acls"></a>Actualización de ACL

Cuando *actualiza* una ACL, modifica la ACL en lugar de reemplazarla. Por ejemplo, puede agregar una nueva entidad de seguridad a la ACL sin que esto afecte a otras entidades de seguridad que se enumeran en la ACL.  Para reemplazar la ACL en lugar de actualizarla, consulte la sección [Establecimiento de listas de control de acceso](#set-acls) de este artículo.

Esta sección le muestra cómo:

- Actualización de una ACL
- Actualización de listas de control de acceso de forma recursiva

### <a name="update-an-acl"></a>Actualización de una ACL

En primer lugar, para obtener la ACL de un directorio, llame al método [PathAccessControl.getAccessControlList](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrol.getaccesscontrollist). Copie la lista de entradas de ACL en un nuevo objeto de **lista** del tipo [PathAccessControlListEntry](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrolentry). A continuación, busque la entrada que desea actualizar y reemplácela en la lista. Establezca la ACL mediante una llamada al método [DataLakeDirectoryClient.setAccessControlList](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist).

En este ejemplo se actualiza la ACL de un directorio denominado `my-parent-directory` reemplazando la entrada para todos los demás usuarios. 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_UpdateACL":::

También puede obtener y establecer la ACL del directorio raíz de un contenedor. Para obtener el directorio raíz, pase una cadena vacía (`""`) en el método **DataLakeFileSystemClient.getDirectoryClient**.

### <a name="update-acls-recursively"></a>Actualización de listas de control de acceso de forma recursiva

Para actualizar una ACL de forma recursiva, cree un nuevo objeto ACL con la entrada de la ACL que quiera actualizar y, a continuación, use ese objeto en la operación para actualizar la ACL. No debe obtener la ACL existente, simplemente proporcione las entradas de la ACL que se vayan a actualizar.

Actualice ACL de forma recursiva mediante una llamada al método **DataLakeDirectoryClient.updateAccessControlRecursive**.  Pase a este método una [lista](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) de objetos [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Cada objeto [PathAccessControlItem](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) define una entrada de la ACL. 

Si desea establecer una entrada de ACL **predeterminada**, puede llamar al método **setDefaultScope** del objeto [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) y pasar un valor de **true**. 

En este ejemplo se actualiza una entrada de ACL con permiso de escritura. Este método acepta un parámetro booleano denominado `isDefaultScope` que especifica si se debe actualizar la ACL predeterminada. Ese parámetro se usa en la llamada al método **setDefaultScope** del objeto [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_UpdateACLRecursively":::

## <a name="remove-acl-entries"></a>Eliminación de entradas de ACL

Puede quitar una o varias entradas de ACL. Esta sección le muestra cómo:

- Eliminación de una entrada de ACL
- Eliminación de las entradas de ACL de forma recursiva

### <a name="remove-an-acl-entry"></a>Eliminación de una entrada de ACL

En primer lugar, para obtener la ACL de un directorio, llame al método [PathAccessControl.getAccessControlList](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrol.getaccesscontrollist). Copie la lista de entradas de ACL en un nuevo objeto de **lista** del tipo [PathAccessControlListEntry](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrolentry). A continuación, busque la entrada que desea quitar y llame al método **Remove** del objeto **Lista**. Establezca la ACL actualizada mediante una llamada al método [DataLakeDirectoryClient.setAccessControlList](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist).

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_RemoveACLEntry":::

### <a name="remove-acl-entries-recursively"></a>Eliminación de las entradas de ACL de forma recursiva

Para quitar entradas de ACL de forma recursiva, cree un nuevo objeto de ACL para la entrada de ACL que se va a quitar y, a continuación, use ese objeto en la operación para quitar la ACL. No debe obtener la ACL existente; simplemente proporcione las entradas de la ACL que se van a quitar.

Elimine entradas de ACL mediante una llamada al método **DataLakeDirectoryClient.removeAccessControlRecursive**. Pase a este método una [lista](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) de objetos [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Cada objeto [PathAccessControlItem](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) define una entrada de la ACL. 

Si desea quitar una entrada de ACL **predeterminada**, puede llamar al método **setDefaultScope** del objeto [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) y pasar un valor de **true**.  

En este ejemplo se quita una entrada de la ACL del directorio llamado `my-parent-directory`. Este método acepta un parámetro booleano denominado `isDefaultScope` que especifica si se debe quitar la entrada de la ACL predeterminada. Ese parámetro se usa en la llamada al método **setDefaultScope** del objeto [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html).

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_RemoveACLRecursively":::

## <a name="recover-from-failures"></a>Recuperación de errores

Podría encontrar errores de tiempo de ejecución o de permisos. En el caso de los errores de tiempo de ejecución, reinicie el proceso desde el principio. Los errores de permisos pueden producirse si la entidad de seguridad no tiene permisos suficientes para modificar la ACL de un directorio o archivo que se encuentra en la jerarquía de directorios que se está modificando. Solucione el problema de permisos y, a continuación, elija reanudar el proceso desde el punto de error mediante un token de continuación, o bien reinicie el proceso desde el principio. No tiene que usar el token de continuación si prefiere reiniciar desde el principio. Puede volver a aplicar las entradas de ACL sin ningún efecto negativo.

Este ejemplo devuelve un token de continuación en caso de que se produzca un error. La aplicación puede volver a llamar a este método de ejemplo después de que se haya solucionado el error y pasar el token de continuación. Si se llama a este método de ejemplo por primera vez, la aplicación puede pasar un valor de `null` para el parámetro del token de continuación.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ResumeSetACLRecursively":::

Si desea que el proceso se complete sin que se interrumpa por errores de permisos, puede especificarlo.

Para asegurarse de que el proceso se completa de forma ininterrumpida, llame al método **setContinueOnFailure** de un objeto [PathSetAccessControlRecursiveOptions](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) y pase un valor de **true**.

En este ejemplo se establecen las entradas de ACL de forma recursiva. Si este código se encuentra un error de permiso, lo registra y continúa la ejecución. En este ejemplo se imprime el número de errores en la consola.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ContinueOnFailure":::

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>Consulte también

- [Documentación de referencia de API](/java/api/overview/azure/storage-file-datalake-readme)
- [Paquete (Maven)](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake)
- [Muestras](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake)
- [Asignación de Gen1 a Gen2](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)
- [Problemas conocidos](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Envíenos sus comentarios](https://github.com/Azure/azure-sdk-for-java/issues)
- [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)
- [Listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)