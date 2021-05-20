---
title: Listas de control de acceso en Azure Data Lake Storage Gen2 | Microsoft Docs
description: Descripción de cómo funcionan las listas de control de acceso de tipo POSIX en Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: 142c8b1439447da4d535dd97e191a0ada503fe94
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632608"
---
# <a name="access-control-lists-acls-in-azure-data-lake-storage-gen2"></a>Listas de control de acceso en Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 implementa un modelo de control de acceso compatible con el control de acceso basado en rol (RBAC) de Azure y las listas de control de acceso (ACL) tipo POSIX. En este artículo se describen las listas de control de acceso de Data Lake Storage Gen2. Para aprender a incorporar el control de acceso basado en rol de Azure con las listas de control de acceso y cómo el sistema los evalúa para tomar decisiones relacionadas con la autorización, consulte [Modelo de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control-model.md).

<a id="access-control-lists-on-files-and-directories"></a>

## <a name="about-acls"></a>Acerca de las listas de control de acceso

Puede asociar una [entidad de seguridad](../../role-based-access-control/overview.md#security-principal) a un nivel de acceso de archivos y directorios. Estas asociaciones se capturan en una *lista de control de acceso (ACL)* . Cada archivo y directorio de la cuenta de almacenamiento tiene una lista de control de acceso. Cuando una entidad de seguridad intenta realizar una operación en un archivo o directorio, una comprobación de ACL determina si esa entidad de seguridad (usuario, grupo, entidad de servicio o identidad administrada) tiene el nivel de permiso adecuado para llevarla a cabo.

> [!NOTE]
> Las ACL solo se aplican a las entidades de seguridad del mismo inquilino y no se aplican a los usuarios que usan la autenticación de tokens de clave compartida o de firma de acceso compartido (SAS). Esto se debe a que no hay ninguna identidad asociada al autor de la llamada y, por lo tanto, no se puede realizar la autorización basada en permisos de la entidad de seguridad.  

<a id="set-access-control-lists"></a>

## <a name="how-to-set-acls"></a>Establecimiento de las ACL

Para establecer permisos en el nivel de archivo y de directorio, consulte cualquiera de los siguientes artículos:

| Entorno | Artículo |
|--------|-----------|
|Explorador de Azure Storage |[Uso del Explorador de Azure Storage para administrar listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-explorer-acl.md)|
|Portal de Azure |[Uso de Azure Portal para administrar ACL en Azure Data Lake Storage Gen2](data-lake-storage-acl-azure-portal.md)|
|.NET |[Uso de .NET para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2](data-lake-storage-acl-dotnet.md)|
|Java|[Uso de Java para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2](data-lake-storage-acl-java.md)|
|Python|[Uso de Python para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2](data-lake-storage-acl-python.md)|
|JavaScript (Node.js)|[Uso de JavaScript SDK en Node.js para administrar ACL en Azure Data Lake Storage Gen2](data-lake-storage-directory-file-acl-javascript.md)|
|PowerShell|[Uso de PowerShell para administrar listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-acl-powershell.md)|
|Azure CLI|[Uso de la CLI de Azure para administrar listas de control de acceso (ACL) en Azure Data Lake Storage Gen2](data-lake-storage-acl-cli.md)|
|API DE REST |[Ruta de acceso: Actualización](/rest/api/storageservices/datalakestoragegen2/path/update)|

> [!IMPORTANT]
> Si la entidad de seguridad es una entidad de *servicio*, es importante que utilice el identificador de objeto de la entidad de servicio en lugar del identificador de objeto del registro de aplicación relacionado. Para obtener el identificador de objeto de la entidad de servicio abra la CLI de Azure y, a continuación, use este comando: `az ad sp show --id <Your App ID> --query objectId`. No olvide reemplazar el marcador de posición `<Your App ID>` por el identificador de aplicación del registro de aplicación.

## <a name="types-of-acls"></a>Tipos de ACL

Hay dos tipos de listas de control de acceso: *ACL de acceso* y *ACL predeterminadas*.

las ACL de acceso controlan el acceso a un objeto. Tanto archivos como directorios tienen ACL de acceso.

Las ACL predeterminadas son plantillas de ACL asociadas con un directorio que determina las ACL de acceso de todos los elementos secundarios que se crean en ese directorio. Los archivos no tienen ACL predeterminadas.

Tanto las ACL de acceso como las ACL predeterminadas tienen la misma estructura.

> [!NOTE]
> El cambio de la ACL predeterminada en un elemento primario no afecta a la ACL de acceso o a la ACL predeterminada de los elementos secundarios que ya existen.

## <a name="levels-of-permission"></a>Niveles de permiso

Los permisos sobre directorios y archivos de un contenedor son **Lectura**, **Escritura** y **Ejecución**, y se pueden usar en archivos y directorios, como se muestra en la tabla siguiente:

|            |    Archivo     |   Directorio |
|------------|-------------|----------|
| **Lectura (R)** | Puede leer el contenido de un archivo | Requiere permisos de **lectura** y **ejecución** para enumerar el contenido del directorio. |
| **Escritura (W)** | Puede escribir o anexar a un archivo | Requiere permisos de **lectura** y **ejecución** para crear elementos secundarios en un directorio. |
| **Ejecución (X)** | No significa nada en el contexto de Data Lake Storage Gen2 | Se requiere para atravesar los elementos secundarios de un directorio. |

> [!NOTE]
> Si va a conceder permisos únicamente mediante listas de control de acceso (y no por medio de Azure RBAC), para conceder a una entidad de seguridad acceso de lectura o escritura a un archivo, necesitará otorgarle permisos de **ejecución** sobre la carpeta raíz del contenedor, y sobre cada carpeta de la jerarquía de carpetas que conduzcan al archivo.

### <a name="short-forms-for-permissions"></a>Formas abreviadas de los permisos

**RWX** se usa para indicar **Lectura + Escritura + Ejecución**. Existe un formato numérico más condensado en el que **Lectura = 4**, **Escritura = 2** y **Ejecución = 1**, y su suma representa los permisos. A continuación se muestran algunos ejemplos.

| Formato numérico | Formato abreviado |      Qué significa     |
|--------------|------------|------------------------|
| 7            | `RWX`        | Lectura + Escritura + Ejecución |
| 5            | `R-X`        | Lectura + ejecución         |
| 4            | `R--`        | Lectura                   |
| 0            | `---`        | Sin permisos         |

### <a name="permissions-inheritance"></a>Herencia de permisos

En el modelo de estilo de POSIX usado por Data Lake Storage Gen2, los permisos de un elemento se almacenan en el propio elemento. En otras palabras, los permisos de un elemento no se pueden heredar de los elementos primarios si los permisos se establecen después de que ya se haya creado el elemento secundario. Los permisos solo se heredan si los permisos predeterminados se han establecido en los elementos primarios antes de crear los secundarios.

## <a name="common-scenarios-related-to-acl-permissions"></a>Escenarios comunes relacionados con los permisos de ACL

En la tabla siguiente se muestran las entradas de ACL necesarias para permitir que una entidad de seguridad realice las operaciones indicadas en la columna **Operación**. 

La tabla incluye columnas que representan cada nivel de una jerarquía de directorios ficticia. Hay columnas para el directorio raíz del contenedor (`\`), un subdirectorio denominado **Oregón**, un subdirectorio del directorio Oregón denominado **Portland** y un archivo de texto en el directorio Portland denominado **Data.txt**. 

> [!IMPORTANT]
> En esta tabla se supone que **solo** se usan listas de control de acceso sin asignaciones de roles de Azure. Para consultar una tabla similar que combina RBAC de Azure junto con las ACL, consulte [Tabla de permisos: combinación de RBAC de Azure y ACL](data-lake-storage-access-control-model.md#permissions-table-combining-azure-rbac-and-acl).

|    Operación             |    /    | Oregón/ | Portland/ | Data.txt     |
|--------------------------|---------|----------|-----------|--------------|
| Leer Data.txt            |   `--X`   |   `--X`    |  `--X`      | `R--`          |
| Anexar a Data.txt       |   `--X`   |   `--X`    |  `--X`      | `RW-`          |
| Eliminar Data.txt          |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| Crear Data.txt          |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| Enumerar /                   |   `R-X`   |   `---`    |  `---`      | `---`          |
| Enumerar /Oregón/           |   `--X`   |   `R-X`    |  `---`      | `---`          |
| Enumerar /Oregón/Portland/  |   `--X`   |   `--X`    |  `R-X`      | `---`          |

> [!NOTE]
> Para eliminar un archivo no es preciso tener permisos de escritura en él, siempre y cuando se cumplan las dos condiciones anteriores.

## <a name="users-and-identities"></a>Usuarios e identidades

Todos los archivos y directorios tienen permisos distintos para estas identidades:

- El usuario propietario
- El grupo propietario
- Usuarios designados
- Grupos designados
- Entidades de servicio designadas
- Identidades administradas designadas
- Los restantes usuarios

Las identidades de usuarios y grupos son las identidades de Azure Active Directory (Azure AD). Por tanto, a menos que se indique lo contrario, un *usuario*, en el contexto de Data Lake Storage Gen2, puede referirse a un usuario, a una entidad de servicio, a una identidad administrada o a un grupo de seguridad de Azure AD.

### <a name="the-owning-user"></a>El usuario propietario

El usuario que creó el elemento es automáticamente el usuario propietario del elemento. Un usuario propietario puede:

* Cambiar los permisos de un archivo que le pertenece.
* Cambiar el grupo propietario de un archivo que le pertenece, siempre que el usuario propietario también sea miembro del grupo de destino.

> [!NOTE]
> El usuario propietario *no puede* cambiar el usuario propietario de un archivo o directorio. Solo los superusuarios pueden cambiar el usuario propietario de un archivo o directorio.

### <a name="the-owning-group"></a>El grupo propietario

En las ACL de POSIX, todos los usuarios están asociados a un *grupo principal*. Por ejemplo, el usuario "Alice" puede pertenecer al grupo "finance". Alice puede pertenecer a varios grupos, pero uno de ellos siempre se designa como su grupo principal. En POSIX, cuando Alice crea un archivo, el grupo propietario de dicho archivo se establece como su grupo principal, que en este caso es "finance". De lo contrario, el grupo propietario se comporta de forma similar a los permisos asignados para otros usuarios o grupos.

#### <a name="assigning-the-owning-group-for-a-new-file-or-directory"></a>Asignar el grupo propietario de un nuevo archivo o directorio

* **Caso 1**: El directorio raíz "/". Este directorio se crea cuando se crea un sistema de archivos de Data Lake Storage Gen2. En este caso, el grupo propietario se establece en el usuario que creó el contenedor, en caso de que se haya realizado con OAuth. Si el contenedor se crea con una clave compartida, una SAS de cuenta o una SAS de servicio, el propietario y el grupo propietario se establecen en **$superuser**.
* **Caso 2** (cada dos casos): cuando se crea un nuevo elemento, se copia el grupo propietario del directorio primario.

#### <a name="changing-the-owning-group"></a>Cambiar el grupo propietario

El grupo propietario se puede cambiar por:
* Cualquier superusuario.
* El usuario propietario, si el usuario propietario también es miembro del grupo de destino.

> [!NOTE]
> El grupo propietario no puede cambiar las ACL de un archivo o directorio.  Aunque el grupo propietario se establece en el usuario que creó la cuenta en el caso del directorio raíz, **Caso 1** anterior, una única cuenta de usuario no es válida para proporcionar los permisos mediante el grupo propietario. Puede asignar este permiso a un grupo de usuarios válidos si es aplicable.

## <a name="access-check-algorithm"></a>Algoritmo de comprobación de acceso

El siguiente seudocódigo representa el algoritmo de comprobación de acceso de las cuentas de almacenamiento.

```console
def access_check( user, desired_perms, path ) : 
  # access_check returns true if user has the desired permissions on the path, false otherwise
  # user is the identity that wants to perform an operation on path
  # desired_perms is a simple integer with values from 0 to 7 ( R=4, W=2, X=1). User desires these permissions
  # path is the file or directory
  # Note: the "sticky bit" isn't illustrated in this algorithm
  
  # Handle super users.
  if (is_superuser(user)) :
    return True

  # Handle the owning user. Note that mask isn't used.
  entry = get_acl_entry( path, OWNER )
  if (user == entry.identity)
      return ( (desired_perms & entry.permissions) == desired_perms )

  # Handle the named users. Note that mask IS used.
  entries = get_acl_entries( path, NAMED_USER )
  for entry in entries:
      if (user == entry.identity ) :
          mask = get_mask( path )
          return ( (desired_perms & entry.permissions & mask) == desired_perms)

  # Handle named groups and owning group
  member_count = 0
  perms = 0
  entries = get_acl_entries( path, NAMED_GROUP | OWNING_GROUP )
  mask = get_mask( path )
  for entry in entries:
    if (user_is_member_of_group(user, entry.identity)) :
        if ((desired_perms & entry.permissions & mask) == desired_perms)
            return True 
        
  # Handle other
  perms = get_perms_for_other(path)
  mask = get_mask( path )
  return ( (desired_perms & perms & mask ) == desired_perms)
```

### <a name="the-mask"></a>La máscara

Como se muestra en el algoritmo de comprobación de acceso, la máscara limita el acceso a usuarios con nombre, el grupo propietario y grupos con nombre.  

En un nuevo contenedor de Data Lake Storage Gen2, el valor predeterminado de la máscara para la ACL de acceso del directorio raíz ("/") es **750** para los directorios y **640** para los archivos. En la tabla siguiente se muestra la notación simbólica de estos niveles de permisos.

|Entidad|Directorios|Archivos|
|--|--|--|
|usuario propietario|`rwx`|`r-w`|
|grupo propietario|`r-x`|`r--`|
|Otros|`---`|`---`|

Los archivos no reciben el bit X, ya que no es pertinente para los archivos de un sistema de solo almacenamiento. 

La máscara se puede especificar por cada llamada. Esto permite que diferentes sistemas de consumo, como los clústeres, tengan distintas máscaras eficaces para sus operaciones de archivo. Si se especifica una máscara en una solicitud dada, reemplaza completamente la máscara predeterminada.

### <a name="the-sticky-bit"></a>El bit persistente

El bit persistente es una característica más avanzada de un contenedor POSIX. En el contexto de Data Lake Storage Gen2, es improbable que se necesite el bit persistente. En resumen, si el bit persistente está habilitado en un directorio, solo el usuario propietario del elemento secundario puede eliminarlo o cambiarle el nombre.

El bit persistente no se muestra en Azure Portal.

## <a name="default-permissions-on-new-files-and-directories"></a>Permisos predeterminados en archivos y directorios nuevos

Cuando se crea un archivo o directorio en un directorio existente, la ACL predeterminada del directorio principal determina:

- Una ACL de acceso y una ACL predeterminada del directorio secundario.
- Una ACL de acceso de un archivo secundario (los archivos no tienen ninguna ACL predeterminada).

### <a name="umask"></a>umask

Al crear un archivo o directorio, se usa umask para modificar cómo se establecen las ACL predeterminadas en el elemento secundario. umask es un valor de 9 bits en los directorios principales que contiene un valor RWX para **usuario propietario**, **grupo propietario** y **otro**.

El valor de umask para Azure Data Lake Storage Gen2 es una constante establecida en 007. Este valor se convierte en:

| componente umask     | Formato numérico | Formato abreviado | Significado |
|---------------------|--------------|------------|---------|
| umask.owning_user   |    0         |   `---`      | En el caso del usuario propietario, copie la ACL predeterminada del elemento principal en la ACL de acceso del elemento secundario. | 
| umask.owning_group  |    0         |   `---`      | En el caso del grupo propietario, copie la ACL predeterminada del elemento principal en la ACL de acceso del elemento secundario. | 
| umask.other         |    7         |   `RWX`      | En el caso de otro, quite todos los permisos en la ACL de acceso del elemento secundario. |

El valor de umask usado por Azure Data Lake Storage Gen2 significa realmente que el valor de **otro** nunca se transmite de forma predeterminada en los nuevos elementos secundarios, a no ser que se defina una lista ACL predeterminada en el directorio principal. En ese caso, el valor de umask se omite de manera efectiva y los permisos definidos por la lista ACL predeterminada se aplican al elemento secundario. 

El siguiente pseudocódigo muestra cómo se aplica umask al crear las ACL de un elemento secundario.

```console
def set_default_acls_for_new_child(parent, child):
    child.acls = []
    for entry in parent.acls :
        new_entry = None
        if (entry.type == OWNING_USER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_user))
        elif (entry.type == OWNING_GROUP) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_group))
        elif (entry.type == OTHER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.other))
        else :
            new_entry = entry.clone(perms = entry.perms )
        child_acls.add( new_entry )
```

## <a name="faq"></a>Preguntas más frecuentes

### <a name="do-i-have-to-enable-support-for-acls"></a>¿Es preciso habilitar la compatibilidad con las ACL?

No. El control de acceso mediante ACL se habilita para una cuenta de almacenamiento siempre que el espacio de nombres jerárquico esté ACTIVADO.

Si el espacio de nombres jerárquico está DESACTIVADO, las reglas de autorización de RBAC de Azure siguen siendo aplicables.

### <a name="what-is-the-best-way-to-apply-acls"></a>¿Cuál es la mejor manera de aplicar las ACL?

[!INCLUDE [Security groups](../../../includes/azure-storage-data-lake-groups.md)] 

### <a name="how-are-azure-rbac-and-acl-permissions-evaluated"></a>¿Cómo se evalúan los permisos de RBAC de Azure y ACL?

Para obtener información sobre cómo el sistema evalúa las listas de control de acceso y los roles RBAC de Azure en conjunto para tomar decisiones relacionadas con la autorización para los recursos de cuentas de almacenamiento, consulte [Evaluación de los permisos](data-lake-storage-access-control-model.md#how-permissions-are-evaluated).

### <a name="what-are-the-limits-for-azure-role-assignments-and-acl-entries"></a>¿Cuáles son los límites de las asignaciones de roles de Azure y las entradas de ACL?

En la tabla siguiente se proporciona una vista resumida de los límites que se deben tener en cuenta al usar RBAC de Azure para administrar los permisos "generales" (permisos que se aplican a las cuentas de almacenamiento o los contenedores) y usar ACL para administrar permisos "específicos" (permisos que se aplican a archivos y directorios). Use grupos de seguridad para las asignaciones de ACL. Mediante el uso de los grupos, es menos probable que supere el número máximo de asignaciones de roles por suscripción y el número máximo de entradas de ACL por archivo o directorio. 

[!INCLUDE [Security groups](../../../includes/azure-storage-data-lake-rbac-acl-limits.md)] 

### <a name="does-data-lake-storage-gen2-support-inheritance-of-azure-rbac"></a>¿Admite Data Lake Storage Gen2 la herencia de RBAC de Azure?

Las asignaciones de roles de Azure se heredan. Las asignaciones fluyen desde la suscripción, el grupo de recursos y los recursos de la cuenta de almacenamiento hasta el recurso del contenedor.

### <a name="does-data-lake-storage-gen2-support-inheritance-of-acls"></a>¿Admite Data Lake Storage Gen2 la herencia de ACL?

Las ACL predeterminadas pueden usarse para establecer las ACL de nuevos subdirectorios y archivos secundarios creados en el directorio principal. Para actualizar las ACL de los elementos secundarios existentes, debe agregar, actualizar o quitar las ACL de forma recursiva para la jerarquía de directorios deseada. Para obtener instrucciones, consulte la sección [Establecimiento de las ACL](#set-access-control-lists) de este artículo. 

### <a name="which-permissions-are-required-to-recursively-delete-a-directory-and-its-contents"></a>¿Qué permisos son necesarios para eliminar de forma recursiva un directorio y su contenido?

- El autor de la llamada tiene permisos de "superusuario",

Or

- El directorio principal debe tener permisos de escritura y ejecución.
- Tanto el directorio que se va a eliminar como todos los directorios que contiene requieren permisos de lectura, escritura y ejecución.

> [!NOTE]
> No necesita permisos de escritura para eliminar archivos en directorios. Además, el directorio raíz "/" nunca se puede eliminar.

### <a name="who-is-the-owner-of-a-file-or-directory"></a>¿Quién es el propietario de un archivo o directorio?

El creador de un archivo o directorio se convierte en el propietario. En el caso del directorio raíz, esta es la identidad del usuario que creó el contenedor.

### <a name="which-group-is-set-as-the-owning-group-of-a-file-or-directory-at-creation"></a>¿Qué grupo se establece como grupo propietario de un archivo o directorio al crearlo?

El grupo propietario se copia del directorio primario en la que se crea el archivo o directorio.

### <a name="i-am-the-owning-user-of-a-file-but-i-dont-have-the-rwx-permissions-i-need-what-do-i-do"></a>Soy el usuario propietario de un archivo, pero no tengo los permisos de RWX que necesito. ¿Qué puedo hacer?

El usuario propietario puede cambiar los permisos del archivo y concederse los permisos de RWX que necesite.

### <a name="why-do-i-sometimes-see-guids-in-acls"></a>¿Por qué a veces veo GUID en ACL?

Un GUID se muestra si la entrada representa un usuario y este ya no existe en Azure AD. Normalmente esto sucede cuando el usuario ha dejado la empresa o si se ha eliminado su cuenta de Azure AD. Además, las entidades de servicio y los grupos de seguridad no tienen un nombre principal de usuario (UPN) para identificarlos y, por tanto, se representan mediante su atributo OID (un GUID).

### <a name="how-do-i-set-acls-correctly-for-a-service-principal"></a>¿Cómo configuro las ACL correctamente para una entidad de servicio?

Cuando defina las ACL para entidades de servicio, es importante que utilice el identificador de objeto (OID) de la *entidad de servicio* para el registro de aplicación que creó. Es importante tener en cuenta que las aplicaciones registradas tienen una entidad de servicio independiente en el inquilino específico de Azure AD. Las aplicaciones registradas tienen un OID que es visible en Azure Portal, pero la *entidad de servicio* tiene otro OID diferente.

Para obtener el OID de la entidad de servicio que corresponde a un registro de aplicación, puede usar el comando `az ad sp show`. Especifique el identificador de aplicación como parámetro. Este es un ejemplo sobre cómo obtener el OID de la entidad de servicio que corresponde a un registro de aplicación con el identificador de aplicación = 18218b12-1895-43e9-ad80-6e8fc1ea88ce. Ejecute el siguiente comando en la CLI de Azure:

```azurecli
az ad sp show --id 18218b12-1895-43e9-ad80-6e8fc1ea88ce --query objectId
```

Se mostrará OID.

Cuando tenga el OID correcto de la entidad de servicio, vaya a la página **Administrar acceso** del Explorador de Azure Storage para agregar el OID y asignar los permisos adecuados para este. No olvide seleccionar **Guardar**.

### <a name="can-i-set-the-acl-of-a-container"></a>¿Se puede establecer la ACL de un contenedor?

No. Un contenedor no tiene una ACL. Sin embargo, puede establecer la ACL del directorio raíz del contenedor. Cada contenedor tiene un directorio raíz, que comparte el mismo nombre que el contenedor. Por ejemplo, si el contenedor se denomina `my-container`, el directorio raíz se denomina `myContainer/`. 

La API REST de Azure Storage contiene una operación denominada [Set Container ACL](/rest/api/storageservices/set-container-acl) (Establecer ACL del contenedor), pero esa operación no se puede usar para establecer la ACL de un contenedor ni del directorio raíz de un contenedor; se usa para indicar si los blobs de un contenedor [permiten el acceso público](anonymous-read-access-configure.md). 

### <a name="where-can-i-learn-more-about-posix-access-control-model"></a>¿Dónde puedo obtener más información sobre el modelo de control de acceso POSIX?

* [POSIX Access Control Lists on Linux](https://www.linux.com/news/posix-acls-linux) (Listas de control de acceso de POSIX en Linux)
* [HDFS Permission Guide](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html) (Guía de permisos de HDFS)
* [POSIX FAQ (PREGUNTAS MÁS FRECUENTES SOBRE POSIX)](https://www.opengroup.org/austin/papers/posix_faq.html)
* [POSIX 1003.1 2008](https://standards.ieee.org/findstds/standard/1003.1-2008.html)
* [POSIX 1003.1 2013](https://pubs.opengroup.org/onlinepubs/9699919799.2013edition/)
* [POSIX 1003.1 2016](https://pubs.opengroup.org/onlinepubs/9699919799.2016edition/)
* [ACL de POSIX en Ubuntu](https://help.ubuntu.com/community/FilePermissionsACLs)
* [ACL: Using Access Control Lists on Linux](https://bencane.com/2012/05/27/acl-using-access-control-lists-on-linux/) (ACL: uso de listas de control de acceso en Linux)

## <a name="see-also"></a>Consulte también

- [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-access-control-model.md)
