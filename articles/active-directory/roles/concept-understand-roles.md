---
title: Descripción de los conceptos de roles de Azure Active Directory
description: Aprenda a entender los roles personalizados e integrados de Azure Active Directory con ámbito de recurso en Azure Active Directory.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/20/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 19d15ea184603587eb3477216bb5a93d6d07f905
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111440527"
---
# <a name="understand-roles-in-azure-active-directory"></a>Descripción de los roles en Azure Active Directory

En Azure Active Directory (Azure AD) hay unos 60 roles integrados, que son roles con un conjunto fijo de permisos de rol. Para complementar los roles integrados, Azure AD también admite roles personalizados. Use roles personalizados para seleccionar los permisos de rol que desee. Por ejemplo, puede crear uno para administrar determinados recursos de Azure AD, como aplicaciones o entidades de servicio.

En este artículo se explica qué son los roles de Azure AD y cómo se pueden usar.

## <a name="how-azure-ad-roles-are-different-from-other-microsoft-365-roles"></a>Diferencias entre los roles de Azure AD y los de Microsoft 365

Hay muchos servicios diferentes en Microsoft 365, como Azure AD e Intune. Algunos de estos servicios tienen sus propios sistemas de control de acceso basado en rol, como:

- Azure AD
- Exchange
- Intune
- Security Center
- Centro de cumplimiento
- Microsoft Cloud App Security
- Operaciones comerciales

Otros servicios como Teams, SharePoint y Managed Desktop, no tienen sistemas de control de acceso basados en rol independientes. Usan los roles de Azure AD para el acceso administrativo. Azure tiene su propio sistema de control de acceso basado en rol para los recursos de Azure, como las máquinas virtuales, que no es el mismo que los roles de Azure AD.

Cuando decimos "un sistema de control de acceso basado en rol independiente", significa que hay un almacén de datos diferente en el que se almacenan las definiciones y las asignaciones de roles. Del mismo modo, se realizan controles de acceso en puntos definidos por distintas directivas. Para más información, consulte [Roles de los servicios de Microsoft 365 en Azure AD](m365-workload-docs.md) y [Roles de administrador de la suscripción clásica, roles de Azure y roles de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

## <a name="why-some-azure-ad-roles-are-for-other-services"></a>¿Por qué algunos roles de Azure AD son para otros servicios?

Microsoft 365 tiene varios sistemas de control de acceso basado en rol que se han desarrollado de forma independiente con el tiempo, cada uno con su propio portal de servicios. Para que sea práctico administrar las identidades en Microsoft 365 desde Azure Portal, se han agregado roles integrados de servicios específicos, cada uno de los cuales concede acceso administrativo a un servicio de Microsoft 365. Un ejemplo de esta incorporación es el rol Administrador de Exchange en Azure AD. Este rol es equivalente al [grupo de roles Administración de la organización](/exchange/organization-management-exchange-2013-help) en el sistema de control de acceso basado en rol de Exchange y administra todos los aspectos de Exchange. Del mismo modo, se ha agregado el rol Administrador de Intune, Administrador de Teams, Administrador de SharePoint, etc. Los roles específicos del servicio son una de las categorías de los roles integrados de Azure AD que se tratan en la sección siguiente.

## <a name="categories-of-azure-ad-roles"></a>Categorías de roles de Azure AD

Los roles integrados de Azure AD se diferencian según el lugar en el que se usan, el cual se puede dividir en tres categorías.

- **Roles específicos de Azure AD**: estos roles conceden permisos para administrar recursos solo en Azure AD. Por ejemplo, Administrador de usuarios, Administrador de aplicaciones, Administrador de grupos, todos conceden permisos para administrar recursos que residen en Azure AD.
- **Roles de servicio específico:** en el caso de los principales servicios de Microsoft 365 (distintos de Azure AD), hemos creado roles de servicio específico que conceden permisos para administrar todas las características de ese servicio.  Por ejemplo, los roles Administrador de Exchange, Administrador de Intune, Administrador de SharePoint y Administrador de Teams pueden administrar características de sus servicios respectivos. El Administrador de Exchange administra los buzones de correo, el Administrador de Intune administra las directivas de dispositivo, el Administrador de SharePoint administra las colecciones de sitios, el Administrador de Teams administra la calidad de las llamadas, etc.
- **Roles multiservicios**: hay algunos roles que abarcan varios servicios. Tenemos dos roles globales: Administrador global y Lector global. Estos dos roles abarcan todos los servicios de Microsoft 365. Además, hay algunos roles relacionados con la seguridad, como el Administrador de seguridad y el Lector de seguridad, que conceden acceso a varios servicios de seguridad en Microsoft 365. Por ejemplo, con los roles de Administrador de seguridad en Azure AD puede administrar Security Center de Microsoft 365, Microsoft Defender para punto de conexión y Microsoft Cloud App Security. Del mismo modo, en el rol Administrador de cumplimiento puede administrar la configuración relacionada con el cumplimiento en el Centro de cumplimiento de Microsoft 365, Exchange, etc.

![Las tres categorías de roles integrados de Azure AD](./media/concept-understand-roles/role-overlap-diagram.png)

La tabla siguiente se ofrece como ayuda para comprender estas categorías de rol. Las categorías tienen nombres asignados arbitrariamente y no pretenden incluir otras funcionalidades aparte de los [permisos de rol de Azure AD documentados](permissions-reference.md).

Category | Role
---- | ----
Roles específicos de Azure AD | Administrador de aplicaciones<br>Desarrollador de aplicaciones<br>Administrador de autenticación<br>Administrador de conjuntos de claves B2C con IEF<br>Administrador de directivas B2C con IEF<br>Administrador de aplicaciones en la nube<br>Administrador de dispositivos en la nube<br>Administrador de acceso condicional<br>Administradores de dispositivos<br>Lectores de directorios<br>Cuentas de sincronización de directorios<br>Escritores de directorios<br>Administrador de flujos de usuarios con id. externo<br>Administrador de atributos de flujos de usuarios con id. externo<br>Administrador de proveedor de identidades externo<br>Administrador de grupos<br>Invitador de usuarios<br>Administrador del departamento de soporte técnico<br>Administrador de identidades híbridas<br>Administrador de licencias<br>Soporte para asociados de nivel 1<br>Soporte para asociados de nivel 2<br>Administrador de contraseñas<br>Administrador de autenticación con privilegios<br>Administrador de roles con privilegios<br>Lector de informes<br>Administrador de usuarios
Roles multiservicios | Administrador global<br>Administrador de cumplimiento<br>Administrador de datos de cumplimiento<br>Lector global<br>Administrador de seguridad<br>Operador de seguridad<br>Lector de seguridad<br>Administrador del soporte técnico del servicio
Roles de servicio específico | Administrador de Azure DevOps<br>Administrador de Azure Information Protection<br>Administrador de facturación<br>Administrador de servicios de CRM<br>Aprobador del acceso a la Caja de seguridad del cliente<br>Administrador de análisis de escritorio<br>Administrador de servicios de Exchange<br>Administrador de Insights<br>Coordinador de Insights de la empresa<br>Administrador de servicios de Intune<br>Administrador de Kaizala<br>Administrador de servicios de Lync<br>Lector de privacidad del Centro de mensajes<br>Lector del Centro de mensajes<br>Usuario de comercio moderno<br>Administrador de red<br>Administrador de aplicaciones de Office<br>Administrador de servicios de Power BI<br>Administrador de Power Platform<br>Administrador de impresoras<br>Técnico de impresoras<br>Administrador de búsqueda<br>Editor de búsqueda<br>Administrador de servicios de SharePoint<br>Administrador de comunicaciones de Teams<br>Ingeniero de soporte técnico de comunicaciones de Teams<br>Especialista de soporte técnico de comunicaciones de Teams<br>Administrador de dispositivos de Teams<br>Administrador de Teams

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al control de acceso basado en rol de Azure AD](custom-overview.md)
- Creación de asignaciones de roles con [Azure Portal, Azure AD PowerShell y Graph API](custom-create.md)
- [Lista de asignaciones de roles](view-assignments.md)
