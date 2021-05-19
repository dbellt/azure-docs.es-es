---
title: Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)
titleSuffix: Azure Storage
description: Acciones y atributos admitidos para las condiciones de asignación de roles de Azure y el control de acceso basado en atributos de Azure (ABAC de Azure) en Azure Storage.
services: storage
author: santoshc
ms.service: storage
ms.topic: conceptual
ms.date: 05/06/2021
ms.author: santoshc
ms.reviewer: jiacfan
ms.subservice: common
ms.openlocfilehash: 66b5bb47f68f0731b3c7f5cc3192427b87f7c2db
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490067"
---
# <a name="actions-and-attributes-for-azure-role-assignment-conditions-in-azure-storage-preview"></a>Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y ABAC de Azure se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se describen los diccionarios de atributos admitidos que se pueden usar en las condiciones para las asignaciones de roles de Azure para cada elemento [DataAction](../../role-based-access-control/role-definitions.md#dataactions) de Azure Storage. Para obtener la lista de operaciones de Blob service que se ven afectadas por un permiso específico o DataAction, consulte [Permisos para operaciones de Blob service](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-blob-service-operations).

Para comprender el formato de una condición de asignación de roles, consulte [Formato y sintaxis de las condiciones de asignación de roles de Azure (versión preliminar)](../../role-based-access-control/conditions-format.md).

## <a name="suboperations"></a>Suboperaciones

Se pueden asociar varias operaciones de servicio de Azure Storage con un solo permiso o DataAction. Sin embargo, cada una de estas operaciones asociadas al mismo permiso puede admitir parámetros diferentes. Las *suboperaciones* permiten diferenciar entre las operaciones de servicio que requieren el mismo permiso, pero admiten un conjunto diferente de atributos para las condiciones. Por lo tanto, mediante una suboperación, puede especificar una condición para el acceso a un subconjunto de operaciones que admiten un parámetro determinado. A continuación, puede usar otra condición de acceso para las operaciones con la misma acción que no admiten ese parámetro.

Por ejemplo, la acción `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` es necesaria para más de una docena de operaciones de servicio diferentes. Algunas de estas operaciones pueden aceptar etiquetas de índice de blobs como parámetro de la solicitud, mientras que otras no. Para las operaciones que aceptan etiquetas de índice de blobs como parámetro, puede usar etiquetas de índice de blobs en una condición de solicitud. Sin embargo, si dicha condición se define en la acción `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write`, todas las operaciones que no aceptan etiquetas como parámetro de solicitud no pueden evaluar esta condición y se producirá un error en la comprobación de acceso de autorización.

En este caso, se puede usar la suboperación opcional `Blob.Write.WithTagHeaders` para aplicar una condición solo a las operaciones que admiten etiquetas de índice de blobs como parámetro de solicitud.

De forma similar, solo las operaciones de selección de la acción `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` pueden admitir etiquetas de índice de blobs como condición previa para el acceso. La suboperación `Blob.Read.WithTagConditions` identifica este subconjunto de operaciones.

> [!NOTE]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de pares de clave y valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar las etiquetas de índice de blobs para las condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md).

En esta versión preliminar, las cuentas de almacenamiento admiten las siguientes suboperaciones:

> [!div class="mx-tableFixed"]
> | DataAction | Suboperación | Nombre para mostrar | Descripción |
> | :--- | :--- | :--- | :--- |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` | `Blob.Read.WithTagConditions` | Operaciones de lectura de blobs que admiten condiciones en las etiquetas | Incluye las operaciones REST Get Blob, Get Blob Metadata, Get Blob Properties, Get Block List, Get Page Ranges y Query Blob Contents. |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` <br/> `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action` | `Blob.Write.WithTagHeaders` | Escrituras de blobs para contenido con etiquetas opcionales | Incluye las operaciones REST Put Blob, Copy Blob, Copy Blob From URL y Put Block List. |

## <a name="actions-and-suboperations"></a>Acciones y suboperaciones

En la tabla siguiente se enumeran las acciones y suboperaciones admitidas para las condiciones en Azure Storage.

> [!div class="mx-tableFixed"]
> | Nombre para mostrar | Descripción | DataAction |
> | --- | --- | --- |
> | Eliminar un blob | DataAction para eliminar blobs. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete` |
> | Leer un blob | DataAction para leer blobs. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` |
> | Leer el contenido de un blob con condiciones de etiquetas  | Operaciones REST: Get Blob, Get Blob Metadata, Get Blob Properties, Get Block List, Get Page Ranges y Query Blob Contents. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`<br/>**Suboperación**<br/>`Blob.Read.WithTagConditions` | 
> | Escribir en un blob | DataAction para escribir en blobs. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` |
> | Escribir en un blob con etiquetas de índice de blobs | Operaciones REST: Put Blob, Put Block List, Copy Blob y Copy Blob From URL. |`Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write`<br/>**Suboperación**<br/>`Blob.Write.WithTagHeaders` | 
> | Crear un blob o una instantánea, o anexar datos | DataAction para crear blobs. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action` |
> | Escribir contenido en un blob con etiquetas de índice de blobs | Operaciones REST: Put Blob, Put Block List, Copy Blob y Copy Blob From URL. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action`<br/>**Suboperación**<br/>`Blob.Write.WithTagHeaders` | 
> | Eliminar una versión de un blob | DataAction para eliminar una versión de un blob. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/deleteBlobVersion/action` |
> | Cambiar la propiedad de un blob | DataAction para cambiar la propiedad de un blob. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/manageOwnership/action` |
> | Modificar los permisos de un blob | DataAction para modificar los permisos de un blob. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/modifyPermissions/action` |
> | Cambiar el nombre de un archivo o directorio | DataAction para cambiar el nombre de archivos o directorios. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/move/action` |
> | Eliminar permanente un blob invalidando la eliminación temporal | DataAction para eliminar permanente un blob invalidando la eliminación temporal. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/permanentDelete/action` |
> | Todas las operaciones de datos para cuentas con HNS | DataAction para todas las operaciones de datos en cuentas de almacenamiento con HNS. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/runAsSuperUser/action` |
> | Leer etiquetas de índice de blobs | DataAction para leer etiquetas de índice de blobs. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` |
> | Escribir etiquetas de índice de blobs | DataAction para escribir etiquetas de índice de blobs. | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` |

## <a name="attributes"></a>Atributos

En la tabla siguiente se enumeran las descripciones de los atributos admitidos para las condiciones en Azure Storage.

> [!div class="mx-tableFixed"]
> | Nombre para mostrar | Descripción | Atributo |
> | --- | --- | --- |
> | Nombre del contenedor| Nombre de un contenedor de almacenamiento o sistema de archivos. Se usa cuando quiera comprobar el nombre del contenedor. | `containers:name` |
> | Ruta del blob | Ruta de acceso de un recurso de directorio virtual, blob, carpeta o archivo. Se usa cuando quiera comprobar el nombre del blob o las carpetas de una ruta de acceso de blob. | `blobs:path` |
> | Etiquetas de índice de blobs [Claves] | Etiquetas de índice en un recurso de blob. Propiedades arbitrarias de pares de clave y valor definidas por el usuario que puede almacenar junto con un recurso de blob. Se usa cuando quiera comprobar la clave en etiquetas de índice de blobs. | `tags&$keys$&` |
> | Etiquetas de índice de blobs [Valores de la clave] | Etiquetas de índice en un recurso de blob. Propiedades arbitrarias de pares de clave y valor definidas por el usuario que puede almacenar junto con un recurso de blob. Se usa cuando quiera comprobar la clave (distingue mayúsculas de minúsculas) y el valor en las etiquetas de índice de blobs. | `tags:`*keyname*`<$key_case_sensitive$>` |

> [!NOTE]
> Se considera que los atributos y valores enumerados no tienen en cuenta las mayúsculas y minúsculas, a menos que se indique lo contrario.

> [!NOTE]
> Al especificar las condiciones para el atributo `Microsoft.Storage/storageAccounts/blobServices/containers/blobs:path`, los valores no deben incluir el nombre del contenedor ni un carácter "/" al comienzo. Use los caracteres de la ruta de acceso sin ninguna codificación de dirección URL.

## <a name="attributes-available-for-each-action"></a>Atributos disponibles para cada acción

En la tabla siguiente se enumeran los atributos que puede usar en las expresiones de condición en función de la acción que quiera realizar. Si selecciona varias acciones para una sola condición, puede que tenga menos atributos entre los que elegir para la condición, ya que los atributos deben estar disponibles en todas las acciones seleccionadas.

> [!div class="mx-tableFixed"]
> | DataAction | Atributo | Tipo | Se aplica a |
> | --- | --- | --- | --- |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`<br/>**Suboperación**<br/>`Blob.Read.WithTagConditions` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> |  | `tags` | dictionaryOfString | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write`<br/>**Suboperación**<br/>`Blob.Write.WithTagHeaders` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> |  | `tags` | dictionaryOfString | RequestAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/add/action`<br/>**Suboperación**<br/>`Blob.Write.WithTagHeaders` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> |  | `tags` | dictionaryOfString | RequestAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/deleteBlobVersion/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/manageOwnership/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/modifyPermissions/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/move/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/permanentDelete/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/runAsSuperUser/action` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> |  | `tags` | dictionaryOfString | ResourceAttributeOnly |
> | `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` | `containers:name` | string | ResourceAttributeOnly |
> |  | `blobs:path` | string | ResourceAttributeOnly |
> |  | `tags` | dictionaryOfString | RequestAttributeOnly |

## <a name="see-also"></a>Consulte también

- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](storage-auth-abac-examples.md)
- [Formato y sintaxis de las condiciones de asignación de roles de Azure (versión preliminar)](../../role-based-access-control/conditions-format.md)
- [¿Qué es el control de acceso basado en atributos de Azure (ABAC de Azure) (versión preliminar)?](../../role-based-access-control/conditions-overview.md)

