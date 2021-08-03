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
ms.openlocfilehash: 895f53ca3e8e1c68fa01ef44ffc47d88604bbea5
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110070865"
---
# <a name="security-considerations-for-azure-role-assignment-conditions-in-azure-storage-preview"></a>Aspectos de seguridad para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y ABAC de Azure se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Para proteger completamente los recursos mediante el [control de acceso basado en atributos de Azure (ABAC de Azure)](storage-auth-abac.md), también debe proteger los [atributos](storage-auth-abac-attributes.md) utilizados en las [condiciones de asignación de roles de Azure](../../role-based-access-control/conditions-format.md). Por ejemplo, si la condición se basa en una ruta de acceso de archivo, debe tener en cuenta que el acceso podría verse comprometido si la entidad de seguridad tiene un permiso sin restricciones para cambiar el nombre de una ruta de acceso de archivo.

En este artículo se describen las consideraciones de seguridad que debe tener en cuenta en las condiciones de asignación de roles.

## <a name="use-of-other-authorization-mechanisms"></a>Uso de otros mecanismos de autorización 

Las condiciones de asignación de roles solo se evalúan cuando se usa Azure RBAC para la autorización. Estas condiciones se pueden omitir si se permite el acceso mediante métodos de autorización alternativos:
- Autorización de [clave compartida](/rest/api/storageservices/authorize-with-shared-key)
- [Firma de acceso compartido de cuenta](/rest/api/storageservices/create-account-sas) (SAS)
- [SAS de servicio](/rest/api/storageservices/create-service-sas).

Del mismo modo, las condiciones no se evalúan cuando se concede el acceso mediante [listas de control de acceso (ACL)](../blobs/data-lake-storage-access-control.md) en cuentas de almacenamiento con un [espacio de nombres jerárquico](../blobs/data-lake-storage-namespace.md) (HNS).

Puede evitar la autorización con SAS de nivel de servicio, SAS de nivel de cuenta y con clave compartida [deshabilitando la autorización con clave compartida](shared-key-authorization-prevent.md) para la cuenta de almacenamiento. Dado que la SAS de delegación de usuarios depende de Azure RBAC, las condiciones de asignación de roles se evalúan al usar este método de autorización.

> [!NOTE]
> Las condiciones de asignación de roles no se evalúan cuando se concede acceso mediante listas ACL con Data Lake Storage Gen2. En este caso, debe planear el ámbito de acceso para que no se superponga con el concedido a través de listas ACL.

## <a name="securing-storage-attributes-used-in-conditions"></a>Protección de los atributos de almacenamiento utilizados en condiciones

### <a name="blob-path"></a>Ruta del blob

Al usar una ruta de acceso de blob como atributo *@Resource* de una condición, también debe impedir que los usuarios vuelvan a asignar un nombre a un blob para obtener acceso a un archivo al usar cuentas que tengan un espacio de nombres jerárquico. Por ejemplo, si desea crear una condición basada en la ruta de acceso del blob, también debe restringir el acceso del usuario a las siguientes acciones:

| Acción | Descripción |
| :--- | :--- |
| `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/move/action` | Esta acción permite a los clientes cambiar el nombre de un archivo mediante Path Create API. |
| `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/runAsSuperUser/action` | Esta acción permite el acceso a varias operaciones de ruta de acceso y de sistema de archivos. |

### <a name="blob-index-tags"></a>Etiquetas de índice de blobs

Las [etiquetas de índice de blobs](../blobs/storage-manage-find-blobs.md) se usan como atributos de forma libre para las condiciones en el almacenamiento. Si crea condiciones de acceso mediante estas etiquetas, también debe proteger las propias etiquetas. Específicamente, el elemento DataAction `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` permite a los usuarios modificar las etiquetas de un objeto del almacenamiento. Puede restringir esta acción para evitar que los usuarios manipulen una clave o valor de etiqueta para obtener acceso a objetos no autorizados.

Además, si se usan etiquetas de índice de blobs en las condiciones, los datos pueden ser vulnerables si dichos datos y las etiquetas de índice asociadas se actualizan en distintas operaciones. Puede usar condiciones `@Request` en operaciones de escritura de blobs para exigir que las etiquetas de índice se establezcan en la misma operación de actualización. Este enfoque puede ayudar a proteger los datos desde el momento en que se escriben en el almacenamiento.

#### <a name="tags-on-copied-blobs"></a>Etiquetas en blobs copiados

De forma predeterminada, las etiquetas de índice de blobs no se copian de un blob de origen al destino cuando se usa [Copy Blob API](/rest/api/storageservices/Copy-Blob) o cualquiera de sus variantes. Para conservar el ámbito de acceso del blob tras la copia, se deben copiar también las etiquetas.

#### <a name="tags-on-snapshots"></a>Etiquetas en instantáneas

Las etiquetas de las instantáneas de blob no se pueden modificar. Esto implica que debe actualizar las etiquetas de un blob antes de tomar la instantánea. Si modifica las etiquetas de un blob base, las etiquetas de su instantánea seguirán teniendo su valor anterior.

Si se modifica una etiqueta de un blob base después de tomar una instantánea, el ámbito de acceso del blob base podría ser diferente al de la instantánea.

#### <a name="tags-on-blob-versions"></a>Etiquetas en versiones de blobs

Las etiquetas de índice de blobs no se copian cuando se crea una versión de blob mediante las API [Put Blob](/rest/api/storageservices/put-blob), [Put Block List](/rest/api/storageservices/put-block-list) o [Copy Blob](/rest/api/storageservices/Copy-Blob). Puede especificar etiquetas mediante el encabezado de estas API.

Las etiquetas se pueden establecer individualmente en un blob base actual y en cada versión de blob. Al modificar etiquetas en un blob base, las etiquetas de las versiones anteriores no se actualizan. Si quiere cambiar el ámbito de acceso de un blob y todas sus versiones mediante etiquetas, debe actualizar las etiquetas del blob base, así como las de todas sus versiones.

#### <a name="querying-and-filtering-limitations-for-versions-and-snapshots"></a>Limitaciones de consulta y filtrado para versiones e instantáneas

Cuando se usan etiquetas para consultar y filtrar los blobs de un contenedor, solo se incluyen los blobs base en la respuesta. No se incluyen las versiones del blob ni las instantáneas con las claves y los valores solicitados.

## <a name="roles-and-permissions"></a>Roles y permisos

Si usa condiciones de asignación de roles para [roles integrados de Azure](../../role-based-access-control/built-in-roles.md), debe revisar cuidadosamente todos los permisos que el rol concede a una entidad de seguridad.

### <a name="inherited-role-assignments"></a>Asignaciones de roles heredadas

Las asignaciones de roles se pueden configurar para un grupo de administración, una suscripción, un grupo de recursos, una cuenta de almacenamiento o un contenedor, y se heredan en cada nivel en el orden indicado. RBAC de Azure tiene un modelo de adición, por lo que los permisos efectivos son la suma de las asignaciones de roles en cada nivel. Si una entidad de seguridad tiene asignado el mismo permiso mediante varias asignaciones de roles, el acceso de una operación que usa ese permiso se evalúa por separado para cada asignación en cada nivel.

Puesto que las condiciones se implementan como tales en las asignaciones de roles, cualquier asignación de roles incondicional puede permitir a los usuarios omitir la condición. Supongamos que asigna el rol *Colaborador de datos de Storage Blob* a un usuario para una cuenta de almacenamiento y en una suscripción, pero agrega una condición solo a la asignación de la cuenta de almacenamiento. En este caso, el usuario tendrá acceso sin restricciones a la cuenta de almacenamiento mediante la asignación de roles en el nivel de suscripción.

Por eso debe aplicar condiciones de forma coherente para todas las asignaciones de roles de una jerarquía de recursos.

## <a name="other-considerations"></a>Otras consideraciones

### <a name="condition-operations-that-write-blobs"></a>Operaciones de condición que escriben blobs

Muchas de las operaciones de escritura de blobs requieren el permiso `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` o `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action`. Los roles integrados, como [Propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) y [Colaborador de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor), conceden ambos permisos a una entidad de seguridad.

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

