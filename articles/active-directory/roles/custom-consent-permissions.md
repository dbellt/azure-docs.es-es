---
title: Permisos para el consentimiento de aplicaciones para roles personalizados en Azure Active Directory | Microsoft Docs
description: Obtenga una vista previa de los permisos de aplicaciones empresariales para roles de Azure AD personalizados en Azure Portal, PowerShell o Graph API.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: psignoret
ms.custom: it-pro
ms.openlocfilehash: a7a26bfd979fd58ba0519aeba274b22ae502f8a8
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110085931"
---
# <a name="app-consent-permissions-for-custom-roles-in-azure-active-directory"></a>Permisos para el consentimiento de aplicaciones para roles personalizados en Azure Active Directory

Este artículo contiene los permisos del registro de aplicaciones disponibles actualmente para las definiciones de roles personalizados en Azure Active Directory (Azure AD). En este artículo, encontrará los permisos necesarios para algunos escenarios comunes relacionados con los permisos y el consentimiento de aplicaciones.

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [License requirement for using custom roles in Azure AD](../../../includes/active-directory-p1-license.md)]

## <a name="app-consent-permissions"></a>Permisos para el consentimiento de aplicaciones

Use los permisos que se muestran en este artículo para administrar las directivas de consentimiento de aplicaciones, así como el permiso para conceder consentimiento a las aplicaciones.

> [!NOTE]
> El portal de administración de Azure AD todavía no permite incorporar los permisos enumerados en este artículo para una definición de rol de directorio personalizado. Debe [usar Azure AD PowerShell para crear un rol de directorio personalizado](custom-create.md#create-a-role-using-powershell) con los permisos enumerados en este artículo.

### <a name="granting-delegated-permissions-to-apps-on-behalf-of-self-user-consent"></a>Concesión de permisos delegados a las aplicaciones en nombre propio (consentimiento del usuario)

Para permitir a los usuarios conceder consentimiento a las aplicaciones en nombre propio (consentimiento del usuario), de acuerdo con una directiva de consentimiento de aplicaciones.

- microsoft.directory/servicePrincipals/managePermissionGrantsForSelf.{id}

En donde `{id}` se reemplaza por el identificador de una [directiva de consentimiento de aplicaciones](../manage-apps/manage-app-consent-policies.md) que establecerá las condiciones que deben cumplirse para que este permiso esté activo.

Por ejemplo, para permitir a los usuarios conceder consentimiento en nombre propio, de acuerdo con la directiva de consentimiento de aplicaciones integrada con el identificador `microsoft-user-default-low`, se usaría el permiso `...managePermissionGrantsForSelf.microsoft-user-default-low`.

### <a name="granting-permissions-to-apps-on-behalf-of-all-admin-consent"></a>Concesión de permisos a las aplicaciones en nombre de todos (consentimiento del administrador)

Para delegar el consentimiento de administrador a las aplicaciones para todo el inquilino, tanto permisos delegados como permisos de aplicación (roles de aplicación):

- microsoft.directory/servicePrincipals/managePermissionGrantsForAll.{id}

En donde `{id}`, se reemplaza por el identificador de una [directiva de consentimiento de aplicaciones](../manage-apps/manage-app-consent-policies.md) que establecerá las condiciones que deben cumplirse para que este permiso se pueda usar.

Por ejemplo, para permitir que los destinatarios del rol concedan consentimiento de administrador a las aplicaciones para todo el inquilino, de acuerdo con una [directiva de consentimiento de aplicaciones](../manage-apps/manage-app-consent-policies.md) personalizada con el identificador `low-risk-any-app`, se usaría el permiso `microsoft.directory/servicePrincipals/managePermissionGrantsForAll.low-risk-any-app`.

### <a name="managing-app-consent-policies"></a>Administración de directivas de consentimiento de aplicaciones

Para delegar la creación, actualización y eliminación de [directivas de consentimiento de aplicaciones](../manage-apps/manage-app-consent-policies.md).

- microsoft.directory/permissionGrantPolicies/create
- microsoft.directory/permissionGrantPolicies/standard/read
- microsoft.directory/permissionGrantPolicies/basic/update
- microsoft.directory/permissionGrantPolicies/delete

## <a name="full-list-of-permissions"></a>Lista completa de permisos

Permiso | Descripción
---------- | -----------
microsoft.directory/servicePrincipals/managePermissionGrantsForSelf.{id} | Permite conceder consentimiento a las aplicaciones en nombre propio (consentimiento del usuario), de acuerdo con la directiva de consentimiento de aplicaciones `{id}`.
microsoft.directory/servicePrincipals/managePermissionGrantsForAll.{id} | Concede el permiso para dar consentimiento a las aplicaciones en nombre de todos (consentimiento del administrador para todo el inquilino), de acuerdo con la directiva de consentimiento de aplicaciones `{id}`.
microsoft.directory/permissionGrantPolicies/standard/read | Permite leer las directivas de consentimiento de aplicaciones.
microsoft.directory/permissionGrantPolicies/basic/update | Permite actualizar las propiedades básicas de las directivas de consentimiento de aplicaciones existentes.
microsoft.directory/permissionGrantPolicies/create | Permite crear las directivas de consentimiento de aplicaciones.
microsoft.directory/permissionGrantPolicies/delete | Permite eliminar las directivas de consentimiento de aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de roles personalizados mediante Azure Portal, PowerShell de Azure AD y Graph API](custom-create.md)
- [Visualización de las asignaciones de un rol personalizado](../roles/view-assignments.md).
