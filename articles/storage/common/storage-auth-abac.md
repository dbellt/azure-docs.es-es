---
title: Autorización del acceso a blobs mediante las condiciones de asignación de roles de Azure (versión preliminar)
titleSuffix: Azure Storage
description: Autorice el acceso a blobs de Azure mediante las condiciones de asignación de roles de Azure y el control de acceso basado en atributos (ABAC) de Azure. Defina condiciones en las asignaciones de roles mediante atributos de almacenamiento.
services: storage
author: santoshc
ms.service: storage
ms.topic: conceptual
ms.date: 05/06/2021
ms.author: santoshc
ms.reviewer: jiacfan
ms.subservice: common
ms.openlocfilehash: 5bba36eb13405d70379feaa561d258813ab52bec
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490071"
---
# <a name="authorize-access-to-blobs-using-azure-role-assignment-conditions-preview"></a>Autorización del acceso a blobs mediante las condiciones de asignación de roles de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El control de acceso basado en atributos (ABAC) es una estrategia de autorización que define los niveles de acceso en función de atributos asociados a entidades de seguridad, recursos, solicitudes y entorno. El control de acceso basado en atributos (ABAC) de Azure se basa en el control de acceso basado en roles (RBAC) de Azure mediante la adición de [condiciones a las asignaciones de roles de Azure](../../role-based-access-control/conditions-overview.md) en el sistema de administración de identidad y acceso (IAM) existente. Esta versión preliminar incluye compatibilidad con las condiciones de asignación de roles en blobs y permite crear condiciones de asignación de roles basadas en atributos de recursos y solicitudes.

## <a name="overview-of-conditions-in-azure-storage"></a>Introducción a las condiciones de Azure Storage

Azure Storage permite el [uso de Azure Active Directory](storage-auth-aad.md) para autorizar solicitudes a Blob Storage y Queue Storage. Azure AD autoriza derechos de acceso para recursos protegidos mediante el control de acceso basado en roles (RBAC). Azure Storage define un conjunto de [roles integrados](../../role-based-access-control/built-in-roles.md#storage) de Azure que engloban los conjuntos comunes de permisos que se usan para acceder a los datos de los blobs y de las colas. También puede definir roles personalizados con un conjunto de permisos seleccionado. Azure Storage admite asignaciones de roles para cuentas de almacenamiento o contenedores de blobs.

Sin embargo, en algunos casos es posible que tenga que habilitar un acceso más específico a los recursos de Storage o simplificar la administración de cientos de asignaciones de roles para un recurso de Storage. En tales casos, las condiciones se pueden configurar como [condiciones en asignaciones de roles](../../role-based-access-control/conditions-overview.md) para [DataActions](../../role-based-access-control/role-definitions.md#dataactions). Las condiciones no se admiten actualmente para las operaciones de administración en el [proveedor de recursos de Storage](/rest/api/storagerp).

Puede usar condiciones con un [rol personalizado](../../role-based-access-control/custom-roles.md) o seleccionar roles integrados. El uso de roles personalizados tiene la ventaja de que se pueden diseñar para que contengan solo los permisos específicos o las acciones de datos que desea conceder a los usuarios. Los roles integrados admitidos en esta versión preliminar incluyen Colaborador de datos de Storage Blob y Lector de datos de Storage Blob. Si trabaja con condiciones basadas en etiquetas de índice de blobs, puede que tenga que usar el rol Propietario de datos de Storage Blob, ya que los permisos para las operaciones de etiquetas están incluidos en este rol.

Los blobs admiten las condiciones de Azure Storage. Esto incluye los blobs de las cuentas que tienen la característica del espacio de nombres jerárquico habilitada en ellas. Las condiciones no se admiten actualmente en Files, Queues y Tables.

## <a name="supported-attributes-and-operations"></a>Atributos y operaciones admitidos

En esta versión preliminar, puede agregar condiciones a los roles integrados para acceder a los datos de blobs. Entre estos roles se incluye el de [Colaborador de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor) y [Lector de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Si trabaja con condiciones basadas en [etiquetas de índice de blobs](../blobs/storage-manage-find-blobs.md), puede que tenga que usar el rol [Propietario de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner), ya que los permisos para las operaciones de etiquetas están incluidos en este rol.

El [formato de las condiciones de asignación de roles de Azure](../../role-based-access-control/conditions-format.md) permite el uso de atributos `@Resource` o `@Request` en las condiciones. Un atributo `@Resource` hace referencia a un atributo existente de un recurso de almacenamiento al que se accede, como una cuenta de almacenamiento, un contenedor o un blob. Un atributo `@Request` hace referencia a un atributo incluido en una solicitud de operación de almacenamiento.

Azure Storage admite un conjunto selecto de atributos de solicitud o recurso que se pueden usar en condiciones en las asignaciones de roles de cada DataAction. Para obtener la lista completa de los atributos admitidos para cada DataAction, consulte [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md).

## <a name="see-also"></a>Consulte también

- [Aspectos de seguridad para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-security.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md)
- [¿Qué es el control de acceso basado en atributos de Azure (versión preliminar)?](../../role-based-access-control/conditions-overview.md)