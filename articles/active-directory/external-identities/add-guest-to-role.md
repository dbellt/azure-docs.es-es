---
title: 'Adición de un usuario de colaboración B2B a un rol: Azure Active Directory'
description: Adición de un usuario invitado a un rol en Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/08/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 455d0ee4855645104dda662785794225f5a23d2d
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109520824"
---
# <a name="grant-permissions-to-users-from-partner-organizations-in-your-azure-active-directory-tenant"></a>Conceda permisos a los usuarios de organizaciones asociadas en el inquilino de Azure Active Directory

Los usuarios de colaboración B2B de Azure Active Directory (Azure AD) se agregan como usuarios invitados al directorio y los permisos de invitado del directorio están restringidos de forma predeterminada. Asimismo, su empresa puede necesitar algunos usuarios invitados para cubrir más roles con privilegios en su organización. Para respaldar la definición de roles de privilegios más elevados, se pueden agregar usuarios invitados a los roles que desee, según las necesidades de su organización.

Si se asigna un rol de directorio a un usuario invitado, se le concederán permisos adicionales que incluyen el rol, incluidos los permisos de lectura básicos. Consulte [Roles integrados de Azure AD](../roles/permissions-reference.md).

## <a name="default-role"></a>Rol predeterminado

![Captura de pantalla que muestra el rol de directorio predeterminado](./media/add-guest-to-role/default-role.png)

## <a name="global-administrator-role"></a>Rol de administrador global

![Captura de pantalla en la que aparece el rol de administrador global](./media/add-guest-to-role/global-admin-role.png)

## <a name="limited-administrator-role"></a>Rol de administrador limitado

![Captura de pantalla en la que aparece el rol Administrador limitado](./media/add-guest-to-role/limited-admin-role.png)

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Propiedades de usuario de la colaboración B2B](user-properties.md)