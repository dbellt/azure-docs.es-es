---
title: Requisitos previos para las condiciones de asignación de roles de Azure (versión preliminar)
description: Requisitos previos para las condiciones de asignación de roles de Azure (versión preliminar).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: conceptual
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.openlocfilehash: e0da284f7ce9ac1c8699a6f22f2144ff6f3f8c80
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490031"
---
# <a name="prerequisites-for-azure-role-assignment-conditions-preview"></a>Requisitos previos para las condiciones de asignación de roles de Azure (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Para agregar o editar condiciones de asignación de roles de Azure, debe cumplir los siguientes requisitos previos.

## <a name="storage-accounts"></a>Cuentas de almacenamiento

Para las condiciones que usan etiquetas de índice de blobs, debe usar una cuenta de almacenamiento que sea compatible con la característica de índice de blobs. Por ejemplo, actualmente solo se admiten las cuentas de almacenamiento De uso general v2 (GPv2) con el espacio de nombres jerárquico (HNS) deshabilitado. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../storage/blobs/storage-manage-find-blobs.md#regional-availability-and-storage-account-support).

## <a name="azure-powershell"></a>Azure PowerShell

Al usar Azure PowerShell para agregar o actualizar condiciones, debe usar las siguientes versiones:

- [Módulo Az 5.5.0 o posterior](https://www.powershellgallery.com/packages/Az/5.5.0)
- [Módulo Az.Resources 3.2.1 o posterior](https://www.powershellgallery.com/packages/Az.Resources/3.2.1)
    - Se incluye con el módulo Az v5.5.0 y versiones posteriores, pero se puede instalar manualmente a través de Galería de PowerShell.
- [Módulo de versión preliminarAz.Storage 2.5.2: versión preliminar o posterior](https://www.powershellgallery.com/packages/Az.Storage/2.5.2-preview)

## <a name="azure-cli"></a>Azure CLI

Al usar la CLI de Azure para agregar o actualizar condiciones, debe usar las siguientes versiones:

- [CLI de Azure 2.18 o posterior](/cli/azure/install-azure-cli)

## <a name="permissions"></a>Permisos

Al igual que las asignaciones de roles, para agregar o actualizar condiciones, debe haber iniciado sesión en Azure con un usuario que tenga los permisos `Microsoft.Authorization/roleAssignments/write` y `Microsoft.Authorization/roleAssignments/delete`, como [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator) o [Propietario](built-in-roles.md#owner).

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
- [Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure Portal (versión preliminar)](../storage/common/storage-auth-abac-portal.md)
