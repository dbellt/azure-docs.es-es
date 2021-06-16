---
title: 'Documentos del rol de administrador en los servicios de Microsoft 365: Azure AD | Microsoft Docs'
description: Buscar contenido y las referencias de API de los roles Administrador para los servicios de Microsoft 365 en Azure Active Directory
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: reference
ms.date: 11/05/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: d2fd75ab871916dae166b2ead54e7379cb3c972f
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110781877"
---
# <a name="roles-for-microsoft-365-services-in-azure-active-directory"></a>Roles para servicios de Microsoft 365 en Azure Active Directory

Todos los productos de Microsoft 365 pueden administrarse con roles administrativos de Azure Active Directory (Azure AD). Algunos productos también proporcionan roles adicionales que son específicos de ese producto. Para obtener información sobre los roles compatibles con cada producto, consulte la tabla siguiente. Para obtener instrucciones sobre el planeamiento de la seguridad de roles, consulte [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](security-planning.md).

## <a name="where-to-find-content"></a>Dónde encontrar el contenido

> [!div class="mx-tableFixed"]
> | Servicio de Microsoft 365 | Contenido del rol | Contenido de la API |
> | ---------------------- | ------------------ | ----------------- |
> | Roles de administrador de los planes de negocios de Office 365 y Microsoft 365 | [Roles de administrador de Microsoft 365](/office365/admin/add-users/about-admin-roles) | No disponible |
> | Azure Active Directory (Azure AD) y Azure AD Identity Protection| [Roles integrados de Azure AD](permissions-reference.md) | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |
> | Exchange Online| [Control de acceso basado en roles de Exchange](/exchange/understanding-role-based-access-control-exchange-2013-help) |  [PowerShell para Exchange](/powershell/module/exchange/role-based-access-control/add-managementroleentry)<br>[Obtener asignaciones de roles](/powershell/module/exchange/role-based-access-control/get-rolegroup) |
> | SharePoint Online | [Roles integrados de Azure AD](permissions-reference.md)<br>También, [Acerca del rol de administrador de SharePoint en Microsoft 365](/sharepoint/sharepoint-admin-role). | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |
> | Teams/Skype Empresarial | [Roles integrados de Azure AD](permissions-reference.md) | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |
> | Centro de seguridad y cumplimiento (Office 365 Advanced Threat Protection, Exchange Online Protection, Information Protection) | [Funciones de administración de Office 365](/office365/SecurityCompliance/permissions-in-the-security-and-compliance-center) | [Exchange PowerShell](/powershell/module/exchange/role-based-access-control/add-managementroleentry)<br>[Obtener asignaciones de roles](/powershell/module/exchange/role-based-access-control/get-rolegroup) |
> | Puntuación segura | [Roles integrados de Azure AD](permissions-reference.md) | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |
> | Administrador de cumplimiento | [Roles del Administrador de cumplimiento](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud#permissions-and-role-based-access-control) | No disponible |
> | Azure Information Protection | [Roles integrados de Azure AD](permissions-reference.md) | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |
> | Microsoft Cloud App Security | [Control de acceso basado en rol](/cloud-app-security/manage-admins) | [Referencia de API](/cloud-app-security/api-tokens)  |
> | Azure Advanced Threat Protection | [Grupos de roles de Azure ATP](/azure-advanced-threat-protection/atp-role-groups) | No disponible |
> | Advanced Threat Protection de Windows Defender | [Control de acceso basado en roles de ATP de Windows Defender](/windows/security/threat-protection/windows-defender-atp/rbac-windows-defender-advanced-threat-protection) | No disponible |
> | Privileged Identity Management | [Roles integrados de Azure AD](permissions-reference.md) | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |
> | Intune | [Control de acceso basado en roles de Intune](/intune/role-based-access-control) | [Graph API](/graph/api/resources/intune-rbac-conceptual?view=graph-rest-beta&preserve-view=true)<br>[Obtener asignaciones de roles](/graph/api/intune-rbac-roledefinition-list?view=graph-rest-beta&preserve-view=true) |
> | Escritorio administrado | [Roles integrados de Azure AD](permissions-reference.md) | [Graph API](/graph/api/overview)<br>[Obtener asignaciones de roles](/graph/api/directoryrole-list) |

## <a name="next-steps"></a>Pasos siguientes

* [Asignación o eliminación de roles de administrador de Azure AD](manage-roles-portal.md)
* [Roles integrados de Azure AD](permissions-reference.md)