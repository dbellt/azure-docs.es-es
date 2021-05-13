---
title: Uso de la CLI de Azure para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2
description: Use la CLI de Azure para administrar listas de control de acceso (ACL) en cuentas de almacenamiento que tengan un espacio de nombres jerárquico.
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1a68ec9868e5a441dbfd11f97bc8f169df246598
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633742"
---
# <a name="use-azure-cli-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Uso de la CLI de Azure para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar la [Interfaz de la línea de comandos (CLI) de Azure](/cli/azure/) para obtener, establecer y actualizar las listas de control de acceso de directorios y archivos.

La herencia de ACL ya está disponible para los nuevos elementos secundarios que se crean en un directorio primario. Ahora bien, también se pueden agregar, actualizar y quitar listas de control de acceso de forma recursiva en los elementos secundarios existentes de un directorio primario sin tener que realizar estos cambios individualmente para cada elemento secundario.

[Referencia](/cli/azure/storage/fs/access) | [Ejemplos](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md) | [Enviar comentarios](https://github.com/Azure/azure-cli-extensions/issues)

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- Una cuenta de almacenamiento que tenga habilitado un espacio de nombres jerárquico. Siga [estas](create-data-lake-storage-account.md) instrucciones para crear uno.

- CLI de Azure versión `2.14.0` o posterior.

- Uno de los siguientes permisos de seguridad:

  - Una [entidad de seguridad](../../role-based-access-control/overview.md#security-principal) aprovisionada de Azure Active Directory (Azure AD) a la que se haya asignado el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito del contenedor de destino, el grupo de recursos primario o la suscripción.  

  - El usuario propietario del contenedor o directorio de destino al que va a aplicar la configuración de ACL. Para establecer listas de control de acceso de forma recursiva, se incluyen todos los elementos secundarios en el contenedor o el directorio de destino.
  
  - Clave de la cuenta de almacenamiento.

## <a name="ensure-that-you-have-the-correct-version-of-azure-cli-installed"></a>Asegúrese de que tiene instalada la versión correcta de la CLI de Azure.

1. Abra [Azure Cloud Shell](../../cloud-shell/overview.md) o, si ha [instalado](/cli/azure/install-azure-cli) la CLI de Azure localmente, abra una aplicación de consola de comandos como Windows PowerShell.

2. Use el comando siguiente para verificar que la versión de la CLI de Azure que ha instalado sea `2.14.0` o posterior.

   ```azurecli
    az --version
   ```

   Si la versión de la CLI de Azure es anterior a `2.14.0`, instale una versión posterior. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="connect-to-the-account"></a>Conexión con la cuenta

1. Si usa la CLI de Azure localmente, ejecute el comando de inicio de sesión.

   ```azurecli
   az login
   ```

   Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

   En caso contrario, abra una página del explorador en [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal. Inicie sesión con las credenciales de su cuenta en el explorador.

   Para obtener más información sobre los distintos métodos de autenticación, consulte [Autorización del acceso a los datos de blobs o colas con la CLI de Azure](./authorize-data-operations-cli.md).

2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento que hospedará el sitio web estático.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

> [!NOTE]
> En el ejemplo que se presenta en este artículo se muestra la autorización de Azure Active Directory (Azure AD). Para obtener más información sobre los métodos de autorización, consulte [Autorización del acceso a los datos de blobs o colas con la CLI de Azure](./authorize-data-operations-cli.md).

## <a name="get-acls"></a>Obtención de listas de control de acceso

Obtenga la ACL de un **directorio** mediante el comando `az storage fs access show`.

En este ejemplo se obtiene la ACL de un directorio y luego se imprime la ACL en la consola.

```azurecli
az storage fs access show -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

Obtenga los permisos de acceso de un **archivo** mediante el comando `az storage fs access show`. 

En este ejemplo se obtiene la ACL de un archivo y luego se imprime la ACL en la consola.

```azurecli
az storage fs access show -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

En la siguiente imagen se muestra la salida después de obtener la ACL de un directorio.

![Obtención de una salida de ACL](./media/data-lake-storage-directory-file-acl-cli/get-acl.png)

En este ejemplo, el usuario propietario tiene permisos de lectura, escritura y ejecución. El grupo propietario tiene permisos de solo lectura y ejecución. Para más información sobre las listas de control de acceso, vea [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

## <a name="set-acls"></a>Establecimiento de listas de control de acceso

Cuando *establece* una ACL, debe **reemplazar** toda la ACL, incluidas todas sus entradas. Si quiere cambiar el nivel de permiso de una entidad de seguridad o agregar una nueva entidad de seguridad a la ACL sin que esto afecte a otras entradas existentes, debe *actualizar* la ACL en su lugar. Para actualizar una ACL en lugar de reemplazarla, consulte la sección [Actualización de ACL](#update-acls) de este artículo.  

Si elige *establecer* la lista de control de acceso, debe agregar una entrada para el usuario propietario, otra para el grupo propietario y una tercera para los demás usuarios. Para más información sobre el usuario propietario, el grupo propietario y el resto de usuarios, consulte [Usuarios e identidades](data-lake-storage-access-control.md#users-and-identities).

Esta sección le muestra cómo:

- Establecimiento de una ACL
- Establecimiento de listas de control de acceso de forma recursiva

### <a name="set-an-acl"></a>Establecimiento de una ACL

Use el comando `az storage fs access set` para establecer la ACL de un **directorio**. 

En este ejemplo se establece la ACL en un directorio del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```azurecli
az storage fs access set --acl "user::rw-,group::rw-,other::-wx" -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

En este ejemplo, se establece la ACL *predeterminada* en un directorio del usuario propietario, el grupo propietario o de otros usuarios y, luego, se imprime la ACL en la consola.

```azurecli
az storage fs access set --acl "default:user::rw-,group::rw-,other::-wx" -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

Use el comando `az storage fs access set` para establecer la ACL de un **archivo**. 

En este ejemplo se establece la ACL en un archivo del usuario propietario, el grupo propietario o de otros usuarios, y luego se imprime la ACL en la consola.

```azurecli
az storage fs access set --acl "user::rw-,group::rw-,other::-wx" -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Para establecer la ACL de un grupo o usuario específico, use sus identificadores de objeto correspondientes. Por ejemplo, `group:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` o `user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

En la siguiente imagen se muestra la salida después de establecer la ACL de un archivo.

![Obtención de una salida de ACL 2](./media/data-lake-storage-directory-file-acl-cli/set-acl-file.png)

En este ejemplo, el usuario propietario y el grupo propietario tienen permisos de solo lectura y escritura. Los demás usuarios tienen permisos de escritura y ejecución. Para más información sobre las listas de control de acceso, vea [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="set-acls-recursively"></a>Establecimiento de listas de control de acceso de forma recursiva

Establezca listas de control de acceso de forma recursiva mediante el comando [az storage fs access set-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_set_recursive).

En este ejemplo se establece la ACL de un directorio denominado `my-parent-directory`. En estas entradas se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos solo de lectura y ejecución al grupo propietario, y ningún permiso al resto. La última entrada de ACL de este ejemplo proporciona a un usuario específico con el identificador de objeto "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" permisos de lectura y ejecución.

```azurecli
az storage fs access set-recursive --acl "user::rwx,group::r-x,other::---,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Si desea establecer entradas de ACL **predeterminadas**, agregue el prefijo `default:` a cada una de ellas. Por ejemplo, `default:user::rwx` o `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x`.

## <a name="update-acls"></a>Actualización de ACL

Cuando *actualiza* una ACL, modifica la ACL en lugar de reemplazarla. Por ejemplo, puede agregar una nueva entidad de seguridad a la ACL sin que esto afecte a otras entidades de seguridad que se enumeran en la ACL.  Para reemplazar la ACL en lugar de actualizarla, consulte la sección [Establecimiento de listas de control de acceso](#set-acls) de este artículo.

Para actualizar una ACL, cree un nuevo objeto ACL con la entrada de la ACL que quiera actualizar y, a continuación, use ese objeto en la operación para actualizar la ACL. No debe obtener la ACL existente, simplemente proporcione las entradas de la ACL que se vayan a actualizar.

Esta sección le muestra cómo:

- Actualización de una ACL
- Actualización de listas de control de acceso de forma recursiva

### <a name="update-an-acl"></a>Actualización de una ACL

Otra manera de establecer este permiso es usar el comando `az storage fs access set`. 

Actualice la ACL de un directorio o archivo estableciendo el parámetro `-permissions` en el formato abreviado de una ACL.

En este ejemplo se actualiza la ACL de un **directorio**.

```azurecli
az storage fs access set --permissions rwxrwxrwx -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

En este ejemplo se actualiza la ACL de un **archivo**.

```azurecli
az storage fs access set --permissions rwxrwxrwx -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Para actualizar la ACL de un grupo o usuario específico, use sus identificadores de objeto correspondientes. Por ejemplo, `group:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` o `user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

También puede actualizar el usuario propietario y el grupo de un directorio o archivo estableciendo los parámetros `--owner` o `group` en el identificador de entidad o en el nombre principal de usuario (UPN) de un usuario.

En este ejemplo se cambia el propietario de un directorio.

```azurecli
az storage fs access set --owner xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

En este ejemplo se cambia el propietario de un archivo. 

```azurecli
az storage fs access set --owner xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login

```

### <a name="update-acls-recursively"></a>Actualización de listas de control de acceso de forma recursiva

Actualice listas de control de acceso de forma recursiva mediante el comando [az storage fs access update-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_update_recursive).

En este ejemplo se actualiza una entrada de ACL con permiso de escritura.

```azurecli
az storage fs access update-recursive --acl "user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:rwx" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Si desea actualizar entradas de ACL **predeterminadas**, agregue el prefijo `default:` a cada una de ellas. Por ejemplo, `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x`.

## <a name="remove-acl-entries-recursively"></a>Eliminación de las entradas de ACL de forma recursiva

Puede quitar una o varias entradas de ACL de forma recursiva. Para quitar una entrada de ACL, cree un nuevo objeto de ACL para la entrada de ACL que se va a quitar y, a continuación, use ese objeto en la operación para quitar la ACL. No debe obtener la ACL existente; simplemente proporcione las entradas de la ACL que se van a quitar.

Quite las entradas de ACL mediante el comando [az storage fs access remove-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_remove_recursive).

En este ejemplo se quita una entrada de ACL del directorio raíz del contenedor.  

```azurecli
az storage fs access remove-recursive --acl "user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Si desea quitar entradas de ACL **predeterminadas**, agregue el prefijo `default:` a cada una de ellas. Por ejemplo, `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

## <a name="recover-from-failures"></a>Recuperación de errores

Al modificar listas de control de acceso de forma recursiva, es posible que se produzcan errores del entorno de ejecución o de los permisos. En el caso de los errores de tiempo de ejecución, reinicie el proceso desde el principio. Los errores de permisos pueden producirse si la entidad de seguridad no tiene permisos suficientes para modificar la ACL de un directorio o archivo que se encuentra en la jerarquía de directorios que se está modificando. Solucione el problema de permisos y, a continuación, elija reanudar el proceso desde el punto de error mediante un token de continuación, o bien reinicie el proceso desde el principio. No tiene que usar el token de continuación si prefiere reiniciar desde el principio. Puede volver a aplicar las entradas de ACL sin ningún efecto negativo.

En caso de que se produzca un error, establezca el parámetro `--continue-on-failure` en `false` para devolver un token de continuación. Después de resolver los errores, puede reanudar el proceso desde el punto de error si ejecuta el comando de nuevo y, a continuación, establece el parámetro `--continuation` en el token de continuación.

```azurecli
az storage fs access set-recursive --acl "user::rw-,group::r-x,other::---" --continue-on-failure false --continuation xxxxxxx -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login  
```

Si desea que el proceso se complete sin que se interrumpa por errores de permisos, puede especificarlo.

Para asegurarse de que el proceso se completa sin interrupciones, establezca el parámetro `--continue-on-failure` en `true`.

```azurecli
az storage fs access set-recursive --acl "user::rw-,group::r-x,other::---" --continue-on-failure true --continuation xxxxxxx -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login  
```

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>Consulte también

- [Muestras](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)
- [Envíenos sus comentarios](https://github.com/Azure/azure-cli-extensions/issues)
- [Problemas conocidos](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)
- [Listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)