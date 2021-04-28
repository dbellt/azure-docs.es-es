---
title: Autorización del acceso a blobs con AzCopy y Azure Active Directory | Microsoft Docs
description: Puede proporcionar credenciales de autorización para las operaciones de AzCopy mediante Azure Active Directory (Azure AD).
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 04/01/2021
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: 9c5e7d1cf318e50e778c59b97b20b03d23fbfe5c
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107903135"
---
# <a name="authorize-access-to-blobs-with-azcopy-and-azure-active-directory-azure-ad"></a>Autorización del acceso a blobs con AzCopy y Azure Active Directory (Azure AD)

Puede proporcionar AzCopy con credenciales de autorización mediante Azure AD. De este modo, no tendrá que anexar un token de firma de acceso compartido (SAS) a cada comando. 

Empiece por comprobar las asignaciones de roles. A continuación, elija qué tipo de _entidad de seguridad_ quiere autorizar. Una [identidad de usuario](../../active-directory/fundamentals/add-users-azure-active-directory.md), una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) y una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md) son tipos de entidad de seguridad.

Una identidad de usuario es cualquier usuario que tiene una identidad en Azure AD. Es la entidad de seguridad más fácil de autorizar. Las identidades administradas y las entidades de servicio son buenas opciones si va a usar AzCopy dentro de un script que se ejecuta sin interacción del usuario. Una identidad administrada es más adecuada para los scripts que se ejecutan desde una máquina virtual (VM) de Azure, y una entidad de servicio es más adecuada para los scripts que se ejecutan localmente. 

Para más información sobre AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md).

## <a name="verify-role-assignments"></a>Comprobación de asignaciones de roles

El nivel de autorización que necesita se basa en si va a cargar los archivos o solo a descargarlos.

Si solo quiere descargar los archivos, compruebe que el rol [Lector de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) se haya asignado a su identidad de usuario, entidad administrada o entidad de servicio.

Si desea cargar archivos, compruebe que uno de estos roles se ha asignado a la entidad de seguridad:

- [Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)

Estos roles pueden asignarse a la entidad de seguridad en cualquiera de estos ámbitos:

- Contenedor (sistema de archivos)
- Cuenta de almacenamiento
- Resource group
- Suscripción

Para aprender a comprobar y asignar roles, consulte [Uso de Azure Portal para asignar un rol de Azure para el acceso a datos de blobs y colas](./storage-auth-aad-rbac-portal.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

> [!NOTE]
> Tenga en cuenta que las asignaciones de roles de Azure pueden tardar hasta cinco minutos en propagarse.

No es necesario tener asignado uno de estos roles a su entidad de seguridad si esta se agrega a la lista de control de acceso (ACL) del directorio o contenedor de destino. En la ACL, la entidad de seguridad necesita permiso de escritura en el directorio de destino y permiso de ejecución en el contenedor y cada directorio primario.

Para más información, consulte [Modelo de control de acceso de Azure Data Lake Storage Gen2](../blobs/data-lake-storage-access-control-model.md).

## <a name="authorize-a-user-identity"></a>Autorización de una identidad de usuario

Después de comprobar que se ha dado el nivel de autorización necesario a su identidad de usuario, abra un símbolo del sistema, escriba el comando siguiente y, a continuación, presione la tecla ENTRAR.

```azcopy
azcopy login
```

Si recibe un error, intente incluir el identificador de inquilino de la organización a la que pertenece la cuenta de almacenamiento.

```azcopy
azcopy login --tenant-id=<tenant-id>
```

Reemplace el marcador de posición `<tenant-id>` por el identificador de inquilino de la organización a la que pertenece la cuenta de almacenamiento. Para buscar el identificador de inquilino, seleccione **Azure Active Directory > Propiedades > Identificador de directorio** en Azure Portal.

Este comando devuelve un código de autenticación y la dirección URL de un sitio web. Abra el sitio web, proporcione el código y, después, elija el botón **Siguiente**.

![Crear un contenedor](media/storage-use-azcopy-v10/azcopy-login.png)

Aparece una ventana de inicio de sesión. En esa ventana, inicie sesión en la cuenta de Azure con sus credenciales de la cuenta de Azure. Cuando haya iniciado sesión correctamente, puede cerrar la ventana del explorador y comenzar a usar AzCopy.

<a id="managed-identity"></a>

## <a name="authorize-a-managed-identity"></a>Creación y autorización de una identidad administrada

Esta es una buena opción si va a usar AzCopy dentro de un script que se ejecuta sin interacción del usuario a partir de una máquina virtual de Azure. Al usar esta opción, no tendrá que almacenar credenciales en la máquina virtual.

Puede iniciar sesión en su cuenta mediante el uso de una identidad administrada que abarque todo el sistema que haya habilitado en la máquina virtual, o bien con el identificador de cliente, de objeto o de recurso de una identidad administrada asignada por el usuario que haya asignado a la máquina virtual.

Para obtener más información sobre cómo habilitar una identidad administrada que abarque todo el sistema o crear una identidad administrada asignada por el usuario, consulte [Configurar identidades administradas para recursos de Azure en una VM mediante Azure Portal](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#enable-system-assigned-managed-identity-on-an-existing-vm).

#### <a name="authorize-by-using-a-system-wide-managed-identity"></a>Autorización mediante el uso de una identidad administrada que abarque todo el sistema

En primer lugar, asegúrese de haber habilitado una identidad administrada para todo el sistema en la máquina virtual. Consulte [Identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

A continuación, en la consola de comandos, escriba el comando siguiente y presione la tecla ENTRAR.

```azcopy
azcopy login --identity
```

#### <a name="authorize-by-using-a-user-assigned-managed-identity"></a>Autorización mediante el uso de una identidad administrada asignada por el usuario

En primer lugar, asegúrese de haber habilitado una identidad administrada asignada por el usuario en la máquina virtual. Consulte [Identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#user-assigned-managed-identity).

A continuación, en la consola de comandos, escriba cualquiera de los siguientes comandos y, luego, presione la tecla Entrar.

```azcopy
azcopy login --identity --identity-client-id "<client-id>"
```

Reemplace el marcador de posición `<client-id>` por el identificador de cliente de la identidad administrada asignada por el usuario.

```azcopy
azcopy login --identity --identity-object-id "<object-id>"
```

Reemplace el marcador de posición `<object-id>` por el identificador de objeto de la identidad administrada asignada por el usuario.

```azcopy
azcopy login --identity --identity-resource-id "<resource-id>"
```

Reemplace el marcador de posición `<resource-id>` por el identificador de recurso de la identidad administrada asignada por el usuario.

<a id="service-principal"></a>

## <a name="authorize-a-service-principal"></a>Autorización de una entidad de servicio

Esta es una buena opción si va a usar AzCopy dentro de un script que se ejecuta sin interacción del usuario, en particular si se ejecuta de forma local. Si tiene previsto ejecutar AzCopy en máquinas virtuales que se ejecutan en Azure, una identidad de servicio administrada es más fácil de administrar. Para más información, consulte la sección [Creación y autorización de una identidad administrada](#authorize-a-managed-identity) de este artículo.

Antes de ejecutar un script, deberá iniciar sesión interactivamente al menos una vez para poder proporcionar a AzCopy las credenciales de la entidad de servicio.  Esas credenciales se almacenan en un archivo seguro y cifrado para que el script no tenga que proporcionar esa información confidencial.

Para iniciar sesión en su cuenta, use un secreto de cliente o la contraseña de un certificado asociado al registro de la aplicación de la entidad de servicio.

Para obtener más información sobre cómo crear una entidad de servicio, consulte [Cómo: portal para crear una aplicación de Azure AD y una entidad de servicio que puedan acceder a los recursos](../../active-directory/develop/howto-create-service-principal-portal.md).

Para obtener más información sobre las entidades de servicio en general, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md).

#### <a name="authorize-a-service-principal-by-using-a-client-secret"></a>Autorización de una entidad de servicio mediante el uso de un secreto de cliente

Para comenzar, establezca la variable de entorno `AZCOPY_SPA_CLIENT_SECRET` en el secreto de cliente del registro de la aplicación de la entidad de servicio.

> [!NOTE]
> Asegúrese de establecer este valor desde el símbolo del sistema y no en la configuración de la variable de entorno del sistema operativo. De este modo, el valor está disponible solo en la sesión actual.

En este ejemplo se muestra cómo podría hacer esto en PowerShell.

```azcopy
$env:AZCOPY_SPA_CLIENT_SECRET="$(Read-Host -prompt "Enter key")"
```

> [!NOTE]
> Considere la posibilidad de utilizar un símbolo del sistema como se muestra en este ejemplo. De este modo, la contraseña no aparecerá en el historial de comandos de la consola.  

A continuación, escriba el comando siguiente y presione la tecla ENTRAR.

```azcopy
azcopy login --service-principal  --application-id application-id --tenant-id=tenant-id
```

Reemplace el marcador de posición `<application-id>` por el identificador de aplicación del registro de la aplicación de la entidad de servicio. Reemplace el marcador de posición `<tenant-id>` por el identificador de inquilino de la organización a la que pertenece la cuenta de almacenamiento. Para buscar el identificador de inquilino, seleccione **Azure Active Directory > Propiedades > Identificador de directorio** en Azure Portal. 

#### <a name="authorize-a-service-principal-by-using-a-certificate"></a>Autorización de una entidad de servicio mediante el uso de un certificado

Si prefiere usar sus propias credenciales para la autorización, puede cargar un certificado en el registro de la aplicación y utilizar ese certificado para iniciar sesión.

Además de cargar el certificado en el registro de la aplicación, también deberá tener una copia del certificado guardada en el equipo o la máquina virtual donde se ejecutará AzCopy. Esta copia del certificado debe tener el formato .PFX o .PEM e incluir la clave privada. La clave privada debe estar protegida por contraseña. Si está usando Windows y el certificado solo existe en un almacén de certificados, asegúrese de exportarlo a un archivo PFX (incluida la clave privada). Para obtener instrucciones, consulte [Export-PfxCertificate](/powershell/module/pki/export-pfxcertificate)

A continuación, establezca la variable de entorno `AZCOPY_SPA_CERT_PASSWORD` en la contraseña del certificado.

> [!NOTE]
> Asegúrese de establecer este valor desde el símbolo del sistema y no en la configuración de la variable de entorno del sistema operativo. De este modo, el valor está disponible solo en la sesión actual.

En este ejemplo se muestra cómo podría realizar esta tarea en PowerShell.

```azcopy
$env:AZCOPY_SPA_CERT_PASSWORD="$(Read-Host -prompt "Enter key")"
```

A continuación, escriba el comando siguiente y presione la tecla ENTRAR.

```azcopy
azcopy login --service-principal --certificate-path <path-to-certificate-file> --tenant-id=<tenant-id>
```

Reemplace el marcador de posición `<path-to-certificate-file>` por la ruta de acceso completa o relativa al archivo del certificado. AzCopy guarda la ruta de acceso a este certificado, pero no guarda una copia del certificado, así que asegúrese de tener el certificado listo para usar. Reemplace el marcador de posición `<tenant-id>` por el identificador de inquilino de la organización a la que pertenece la cuenta de almacenamiento. Para buscar el identificador de inquilino, seleccione **Azure Active Directory > Propiedades > Identificador de directorio** en Azure Portal.

> [!NOTE]
> Considere la posibilidad de utilizar un símbolo del sistema como se muestra en este ejemplo. De este modo, la contraseña no aparecerá en el historial de comandos de la consola. 

## <a name="authorize-without-a-secret-store"></a>Autorización sin un almacén secreto

El comando `azcopy login` recupera un token de OAuth y, a continuación, coloca ese token en un almacén secreto del sistema. Si el sistema operativo no tiene un almacén secreto como un *conjunto de claves* de Linux, el comando `azcopy login` no funcionará porque no hay ningún lugar donde colocar el token. 

En lugar de usar el comando `azcopy login`, puede establecer variables de entorno en memoria. A continuación, ejecute cualquier comando de AzCopy. AzCopy recuperará el token de autenticación necesario para completar la operación. Una vez finalizada la operación, el token desaparece de la memoria. 

### <a name="authorize-a-user-identity"></a>Autorización de una identidad de usuario

Después de comprobar que se ha dado el nivel de autorización necesario a su identidad de usuario, escriba el comando siguiente y, a continuación, presione la tecla ENTRAR.

```bash
export AZCOPY_AUTO_LOGIN_TYPE=DEVICE
```

A continuación, ejecute cualquier comando azcopy (por ejemplo: `azcopy list https://contoso.blob.core.windows.net`).

Este comando devuelve un código de autenticación y la dirección URL de un sitio web. Abra el sitio web, proporcione el código y, después, elija el botón **Siguiente**.

![Crear un contenedor](media/storage-use-azcopy-v10/azcopy-login.png)

Aparece una ventana de inicio de sesión. En esa ventana, inicie sesión en la cuenta de Azure con sus credenciales de la cuenta de Azure. Una vez que haya iniciado sesión correctamente, la operación se puede completar.

### <a name="authorize-by-using-a-system-wide-managed-identity"></a>Autorización mediante el uso de una identidad administrada que abarque todo el sistema

En primer lugar, asegúrese de haber habilitado una identidad administrada para todo el sistema en la máquina virtual. Consulte [Identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

Escriba el comando siguiente y presione la tecla ENTRAR.

```bash
export AZCOPY_AUTO_LOGIN_TYPE=MSI
```

A continuación, ejecute cualquier comando azcopy (por ejemplo: `azcopy list https://contoso.blob.core.windows.net`).

### <a name="authorize-by-using-a-user-assigned-managed-identity"></a>Autorización mediante el uso de una identidad administrada asignada por el usuario

En primer lugar, asegúrese de haber habilitado una identidad administrada asignada por el usuario en la máquina virtual. Consulte [Identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#user-assigned-managed-identity).

Escriba el comando siguiente y presione la tecla ENTRAR.

```bash
export AZCOPY_AUTO_LOGIN_TYPE=MSI
```

A continuación, escriba cualquiera de los comandos siguientes y presione la tecla ENTRAR.

```bash
export AZCOPY_MSI_CLIENT_ID=<client-id>
```

Reemplace el marcador de posición `<client-id>` por el identificador de cliente de la identidad administrada asignada por el usuario.

```bash
export AZCOPY_MSI_OBJECT_ID=<object-id>
```

Reemplace el marcador de posición `<object-id>` por el identificador de objeto de la identidad administrada asignada por el usuario.

```bash
export AZCOPY_MSI_RESOURCE_STRING=<resource-id>
```

Reemplace el marcador de posición `<resource-id>` por el identificador de recurso de la identidad administrada asignada por el usuario.

Después de establecer estas variables, puede ejecutar cualquier comando de azcopy (por ejemplo: `azcopy list https://contoso.blob.core.windows.net`).

### <a name="authorize-a-service-principal"></a>Autorización de una entidad de servicio

Para iniciar sesión en su cuenta, use un secreto de cliente o la contraseña de un certificado asociado al registro de la aplicación de la entidad de servicio.

#### <a name="authorize-a-service-principal-by-using-a-client-secret"></a>Autorización de una entidad de servicio mediante el uso de un secreto de cliente

Escriba el comando siguiente y presione la tecla ENTRAR.

```bash
export AZCOPY_AUTO_LOGIN_TYPE=SPN
export AZCOPY_SPA_APPLICATION_ID=<application-id>
export AZCOPY_SPA_CLIENT_SECRET=<client-secret>
export AZCOPY_TENANT_ID=<tenant-id>
```

Reemplace el marcador de posición `<application-id>` por el identificador de aplicación del registro de la aplicación de la entidad de servicio. Reemplace el marcador de posición `<client-secret>` por el secreto de cliente. Reemplace el marcador de posición `<tenant-id>` por el identificador de inquilino de la organización a la que pertenece la cuenta de almacenamiento. Para buscar el identificador de inquilino, seleccione **Azure Active Directory > Propiedades > Identificador de directorio** en Azure Portal. 

> [!NOTE]
> Considere la posibilidad de usar una solicitud para recopilar la contraseña del usuario. De este modo, la contraseña no aparecerá en el historial de comandos. 

A continuación, ejecute cualquier comando azcopy (por ejemplo: `azcopy list https://contoso.blob.core.windows.net`).

#### <a name="authorize-a-service-principal-by-using-a-certificate"></a>Autorización de una entidad de servicio mediante el uso de un certificado

Si prefiere usar sus propias credenciales para la autorización, puede cargar un certificado en el registro de la aplicación y utilizar ese certificado para iniciar sesión.

Además de cargar el certificado en el registro de la aplicación, también deberá tener una copia del certificado guardada en el equipo o la máquina virtual donde se ejecutará AzCopy. Esta copia del certificado debe tener el formato .PFX o .PEM e incluir la clave privada. La clave privada debe estar protegida por contraseña. 

Escriba el comando siguiente y presione la tecla ENTRAR.

```bash
export AZCOPY_AUTO_LOGIN_TYPE=SPN
export AZCOPY_SPA_CERT_PATH=<path-to-certificate-file>
export AZCOPY_SPA_CERT_PASSWORD=<certificate-password>
export AZCOPY_TENANT_ID=<tenant-id>
```

Reemplace el marcador de posición `<path-to-certificate-file>` por la ruta de acceso completa o relativa al archivo del certificado. AzCopy guarda la ruta de acceso a este certificado, pero no guarda una copia del certificado, así que asegúrese de tener el certificado listo para usar. Reemplace el marcador de posición `<certificate-password>` por la contraseña del certificado. Reemplace el marcador de posición `<tenant-id>` por el identificador de inquilino de la organización a la que pertenece la cuenta de almacenamiento. Para buscar el identificador de inquilino, seleccione **Azure Active Directory > Propiedades > Identificador de directorio** en Azure Portal. 

> [!NOTE]
> Considere la posibilidad de usar una solicitud para recopilar la contraseña del usuario. De este modo, la contraseña no aparecerá en el historial de comandos. 

A continuación, ejecute cualquier comando azcopy (por ejemplo: `azcopy list https://contoso.blob.core.windows.net`).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md).

- Si tiene alguna pregunta, problemas o comentarios generales, envíelos en [la página GitHub](https://github.com/Azure/azure-storage-azcopy).
