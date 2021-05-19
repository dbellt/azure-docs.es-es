---
title: Aspectos de seguridad para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)
titleSuffix: Azure Storage
description: Aspectos de seguridad para las condiciones de asignación de roles de Azure y el control de acceso basado en atributos de Azure (ABAC de Azure).
services: storage
author: santoshc
ms.service: storage
ms.topic: conceptual
ms.date: 05/06/2021
ms.author: santoshc
ms.reviewer: jiacfan
ms.subservice: common
ms.openlocfilehash: 83b3f4f9b84c25e3fe2822cbabb63adec1fb22ab
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490072"
---
# <a name="security-considerations-for-azure-role-assignment-conditions-in-azure-storage-preview"></a>Aspectos de seguridad para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y ABAC de Azure se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Para proteger completamente los recursos mediante el [control de acceso basado en atributos de Azure (ABAC de Azure)](storage-auth-abac.md), también debe proteger los [atributos](storage-auth-abac-attributes.md) utilizados en las [condiciones de asignación de roles de Azure](../../role-based-access-control/conditions-format.md). Esto requiere que proteja todos los permisos o acciones que se pueden usar para modificar los atributos usados en las condiciones de asignación de roles. Por ejemplo, si crea una condición para una cuenta de almacenamiento basada en una ruta de acceso, debe tener en cuenta que el acceso podría verse comprometido si la entidad de seguridad tiene un permiso sin restricciones para cambiar el nombre de una ruta de acceso de archivo.

En este artículo se describen las consideraciones de seguridad que debe tener en cuenta en las condiciones de asignación de roles.

## <a name="use-of-other-authorization-mechanisms"></a>Uso de otros mecanismos de autorización 

ABAC de Azure se implementa como condiciones en las asignaciones de roles. Dado que estas condiciones solo se evalúan cuando se usa el [control de acceso basado en roles de Azure (RBAC de Azure)](../../role-based-access-control/overview.md) con Azure Active Directory (Azure AD), se pueden omitir si se habilita el acceso mediante métodos de autorización alternativos. Por ejemplo, las condiciones no se evalúan al usar la autorización de clave compartida o de firma de acceso compartido. De forma similar, las condiciones no se evalúan cuando se concede acceso a un archivo o una carpeta mediante [listas de control de acceso (ACL)](../blobs/data-lake-storage-access-control.md) en cuentas que tienen habilitada la característica de espacio de nombres jerárquico. 

Para evitarlo, [deshabilite la autorización de clave compartida](shared-key-authorization-prevent.md) para la cuenta de almacenamiento.

## <a name="securing-storage-attributes-used-in-conditions"></a>Protección de los atributos de almacenamiento utilizados en condiciones

### <a name="blob-path"></a>Ruta del blob

Al usar una ruta de acceso de blob como atributo *@Resource* de una condición, también debe impedir que los usuarios vuelvan a asignar un nombre a un blob para obtener acceso a un archivo al usar cuentas que tengan un espacio de nombres jerárquico. Por ejemplo, si desea crear una condición basada en la ruta de acceso del blob, también debe restringir el acceso del usuario a las siguientes acciones:

| Acción | Descripción |
| :--- | :--- |
| `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/move/action` | Permite a los clientes cambiar el nombre de un archivo mediante la API Path Create. |
| `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/runAsSuperUser/action` | Permite el acceso a varias operaciones de ruta de acceso y del sistema de archivos. |

### <a name="blob-index-tags"></a>Etiquetas de índice de blobs

Las [etiquetas de índice de blobs](../blobs/storage-manage-find-blobs.md) se usan como atributos de forma libre para las condiciones en el almacenamiento. Si crea condiciones de acceso mediante estas etiquetas, también debe proteger las propias etiquetas. Específicamente, el elemento DataAction `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` permite a los usuarios modificar las etiquetas de un objeto del almacenamiento. También se debe restringir adecuadamente el acceso de una entidad de seguridad a esta acción para evitar que modifique una clave o un valor de etiqueta para obtener acceso a un objeto almacenado al que de otro modo no podría acceder.

Además, si se usan etiquetas de índice de blobs en las condiciones, los datos pueden ser vulnerables periódicamente si los datos y las etiquetas de índice asociadas se actualizan en operaciones independientes. Para proteger los datos desde el momento en que se escriben en el almacenamiento, las condiciones de las operaciones de escritura de blobs deben requerir el valor adecuado para que las etiquetas de índice asociadas al blob se establezcan en los valores adecuados en la misma operación de actualización.

#### <a name="tags-on-copied-blobs"></a>Etiquetas en blobs copiados

Las etiquetas de índice de blobs no se copian de un blob de origen al destino de manera predeterminada cuando se usa la API [Copy Blob](/rest/api/storageservices/Copy-Blob) o cualquiera de sus variantes. Para conservar el ámbito de acceso del blob tras la copia, también se deben copiar explícitamente sus etiquetas.

#### <a name="tags-on-snapshots"></a>Etiquetas en instantáneas

No se admite la actualización de etiquetas en instantáneas de blobs en la versión preliminar. Esto implica que debe actualizar las etiquetas de un blob antes de tomar la instantánea. Cualquier actualización de etiquetas solo se aplicará al blob base. Las etiquetas de la instantánea seguirán teniendo el valor anterior.

Si se modifica una etiqueta en el blob base después de tomar una instantánea y hay una condición que usa esa etiqueta, el ámbito de acceso para el blob base podría ser diferente al de la instantánea del blob.

#### <a name="tags-on-blob-versions"></a>Etiquetas en versiones de blobs

Las etiquetas de índice de blobs no se copian cuando se crea una versión de blob mediante las API [Put Blob](/rest/api/storageservices/put-blob), [Put Block List](/rest/api/storageservices/put-block-list) o [Copy Blob](/rest/api/storageservices/Copy-Blob). Puede especificar etiquetas mediante el encabezado de estas API.

Puede modificar etiquetas en distintas versiones de un blob, pero estas no se actualizan automáticamente cuando se modifican las etiquetas de un blob base. Si quiere cambiar el ámbito de acceso de un blob y todas sus versiones mediante etiquetas, debe actualizar las etiquetas en el blob base, así como en todas sus versiones.

#### <a name="querying-and-filtering-limitations-for-versions-and-snapshots"></a>Limitaciones de consulta y filtrado para versiones e instantáneas

Cuando se usan etiquetas para consultar y filtrar los blobs de un contenedor, solo se incluyen los blobs base en la respuesta. No se incluyen las versiones del blob ni las instantáneas con las claves y los valores solicitados.

## <a name="roles-and-permissions"></a>Roles y permisos

Si usa condiciones de asignación de roles para [roles integrados de Azure](../../role-based-access-control/built-in-roles.md), debe revisar cuidadosamente todos los permisos que el rol concede a una entidad de seguridad.

### <a name="inherited-role-assignments"></a>Asignaciones de roles heredadas

Las asignaciones de roles se pueden configurar para un grupo de administración, una suscripción, un grupo de recursos, una cuenta de almacenamiento o un contenedor, y se heredan en cada nivel en el orden indicado. RBAC de Azure tiene un modelo de adición, por lo que los permisos efectivos son la suma de las asignaciones de roles en cada nivel. Si una entidad de seguridad tiene un permiso asignado mediante varios roles o un rol determinado en varios niveles, el acceso para una operación con ese permiso se evalúa por separado para cada rol asignado en cada nivel.

Puesto que las condiciones se implementan como condiciones en las asignaciones de roles, cualquier asignación de roles incondicional puede permitir a los usuarios omitir las restricciones de acceso previstas por la directiva de la condición. Por ejemplo, si se asigna un rol a una entidad de seguridad, como *Colaborador de datos de Storage Blob*, en los niveles de suscripción y de cuenta de almacenamiento y la condición de la asignación de roles solo se define en el nivel de cuenta de almacenamiento, la entidad de seguridad tendrá acceso sin restricciones a la cuenta mediante la asignación de roles en el nivel de suscripción y viceversa.

Por lo tanto, las condiciones se deben aplicar de forma coherente en todos los niveles de la jerarquía de recursos en los que se haya concedido acceso a un recurso a las entidades de seguridad.

## <a name="other-considerations"></a>Otras consideraciones

### <a name="condition-operations-that-write-blobs"></a>Operaciones de condición que escriben blobs

Muchas de las operaciones que escriben blobs requieren el permiso `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` o `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action`. Los roles integrados, como [Propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) y [Colaborador de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor), conceden ambos permisos a una entidad de seguridad.

Al definir una condición de asignación de roles para estos roles, debe usar condiciones idénticas en ambos permisos para garantizar restricciones de acceso coherentes para las operaciones de escritura.

### <a name="behavior-for-copy-blob-and-copy-blob-from-url"></a>Comportamiento de Copy Blob y Copy Blob from URL

Para las operaciones [Copy Blob](/rest/api/storageservices/Copy-Blob) y [Copy Blob From URL](/rest/api/storageservices/copy-blob-from-url), las condiciones `@Request` que usan la ruta de acceso del blob como atributo en la acción `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` y sus suboperaciones solo se evalúan para el blob de destino.

En el caso de las condiciones del blob de origen, se evalúan las condiciones `@Resource` de la acción `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read`.

### <a name="behavior-for-get-page-ranges"></a>Comportamiento de Get Page Ranges

Para la operación [Get Page Ranges](/rest/api/storageservices/get-page-ranges), las condiciones `@Resource` que usan `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags` como atributo en la acción `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` y sus suboperaciones solo se evalúan para el blob de destino.

No se aplican las condiciones para el acceso al blob especificado por el parámetro de identificador URI `prevsnapshot` en la API.

## <a name="see-also"></a>Consulte también

- [Autorización del acceso a blobs mediante las condiciones de asignación de roles de Azure (versión preliminar)](storage-auth-abac.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md)
- [¿Qué es el control de acceso basado en atributos de Azure (ABAC de Azure) (versión preliminar)?](../../role-based-access-control/conditions-overview.md)

