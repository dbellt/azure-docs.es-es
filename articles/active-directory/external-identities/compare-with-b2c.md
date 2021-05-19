---
title: 'Comparación de External Identities: Azure Active Directory | Microsoft Docs'
description: Azure AD External Identities permite que personas ajenas a la organización tengan acceso a sus aplicaciones y recursos mediante su propia identidad. Compare soluciones para External Identities, incluida la colaboración B2B de Azure Active Directory y Azure AD B2C.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: overview
ms.date: 03/02/2021
ms.custom: project-no-code
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: d64fd91d3f033c0a04c4936f11d096aa640942a1
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743892"
---
# <a name="what-are-external-identities-in-azure-active-directory"></a>¿Qué son External Identities de Azure Active Directory?

Mediante External Identities de Azure AD, puede facilitar a personas ajenas a una organización el acceso a sus aplicaciones y recursos, además de permitirles iniciar sesión con la identidad que prefieran. Los asociados, distribuidores, proveedores y otros usuarios invitados pueden "traer sus propias identidades". Tanto si tienen una identidad digital corporativa o emitida por una entidad gubernamental, como si tienen una identidad social no administrada, como Google o Facebook, pueden usar sus propias credenciales para iniciar sesión. El proveedor de identidades administra la identidad del usuario externo y el usuario administra el acceso a sus aplicaciones con Azure AD para mantener protegidos los recursos.

## <a name="external-identities-scenarios"></a>Escenarios de External Identities

Azure AD External Identities se centra menos en la relación de un usuario con su organización y más en la forma en que un usuario quiere iniciar sesión en sus aplicaciones y recursos. Dentro de este marco, Azure AD admite varios escenarios, desde la colaboración de negocio a negocio (B2B) a la administración de accesos para aplicaciones orientadas a consumidores, clientes o ciudadanos (negocio a cliente, o B2C).

- **Uso compartido de aplicaciones y recursos con usuarios externos (colaboración B2B)** . Invite a usuarios externos a su propio inquilino como usuarios "invitados" a los que puede asignar permisos (para autorización) al tiempo que les permite usar sus credenciales existentes (para autenticación). Los usuarios inician sesión en los recursos compartidos con un proceso sencillo de invitación y canje con su cuenta profesional o educativa o con cualquier cuenta de correo electrónico. También puede usar la [administración de derechos de Azure AD](../governance/entitlement-management-overview.md) para configurar directivas que [administren el acceso para los usuarios externos](../governance/entitlement-management-external-users.md#how-access-works-for-external-users). Y ahora, con la disponibilidad de los [flujos de registro de usuarios en modo de autoservicio](self-service-sign-up-overview.md), puede permitir que los usuarios externos se registren por sí mismos en las aplicaciones. La experiencia se puede personalizar para permitir el registro con una identidad profesional, educativa o de redes sociales (como Google o Facebook). También puede recopilar información sobre el usuario durante el proceso de registro. Para obtener más información, consulte la [Documentación de Azure AD B2B](index.yml).

- **Cree recorridos del usuario con una solución de administración de identidades de marca blanca para aplicaciones orientadas a consumidores y clientes (Azure AD B2C)** . Si es un negocio o un desarrollador que crea aplicaciones orientadas a clientes, puede escalar a millones de consumidores, clientes o ciudadanos mediante Azure AD B2C. Los desarrolladores pueden usar Azure AD como sistema completo de administración de identidades y acceso de cliente (CIAM) para sus aplicaciones. Los clientes pueden iniciar sesión con una identidad ya establecida (como Facebook o Gmail). Con Azure AD B2C, puede personalizar y controlar el modo en que los clientes se suscriben, inician sesión y administran sus perfiles al usar las aplicaciones. Para obtener más información, consulte la [Documentación de Azure AD B2C](../../active-directory-b2c/index.yml).

## <a name="compare-external-identities-solutions"></a>Comparación de las soluciones de External Identities

En la tabla siguiente se proporciona una comparación detallada de los escenarios que se pueden habilitar con Azure AD External Identities.

|   | Colaboración de usuario externo (B2B) | Acceso a aplicaciones orientadas a consumidores o clientes (B2C)  |
| ---- | --- | --- |
| **Escenario principal** | Colaboración mediante aplicaciones de Microsoft (Microsoft 365, Teams, etc.) o sus propias aplicaciones (aplicaciones SaaS, aplicaciones personalizadas, etc.).  | Administración de identidades y acceso para aplicaciones SaaS o personalizadas modernas (no aplicaciones propias de Microsoft).   |
| **Destinado a**    | Colaborar con socios comerciales de organizaciones externas como proveedores o asociados. Los usuarios aparecen como usuarios invitados en el directorio. Estos usuarios pueden tener TI administrado o no.  | Clientes de su producto. Estos usuarios se administran en un directorio de Azure AD independiente.  |
| **Se admiten proveedores de identidades** | Los usuarios externos pueden colaborar mediante cuentas profesionales, cuentas educativas, cualquier dirección de correo electrónico, proveedores de identidades basados en SAML y WS-Fed, Gmail y Facebook.  | Usuarios consumidores con cuentas de aplicaciones locales (cualquier dirección de correo electrónico o nombre de usuario), diversas identidades admitidas de redes sociales y usuarios con identidades corporativas o emitidas por una entidad gubernamental mediante la federación del proveedor de identidades basada en SAML o WS-Fed.       |
| **Administración de usuarios externos**   | Los usuarios externos se administran en el mismo directorio que los empleados, pero normalmente se les etiqueta como usuarios invitados. Los usuarios invitados pueden administrarse del mismo modo que los empleados, pueden agregarse a los mismos grupos, etc.    | Los usuarios externos se administran en el directorio de Azure AD B2C. Se administran de manera independiente del directorio de asociados y de empleados de la organización (si existe).  |
| **Inicio de sesión único (SSO)**      | se admite SSO en todas las aplicaciones conectadas a Azure AD. Por ejemplo, puede proporcionar acceso a Microsoft 365, o bien a aplicaciones locales y a otras aplicaciones SaaS como Salesforce o Workday.    | Se admite el inicio de sesión único para aplicaciones propiedad de los clientes dentro de los inquilinos de Azure AD B2C. No se admite el inicio de sesión único en Microsoft 365 ni en otras aplicaciones SaaS de Microsoft.    |
| **Directiva de seguridad y cumplimiento**        | los administra la organización anfitriona o que realiza la invitación (por ejemplo, con [directivas de acceso condicional](conditional-access.md)). | Las administra la organización mediante el acceso condicional y Identity Protection.        |
| **Personalización de marca**  | se utiliza la marca de la organización anfitriona o que realiza la invitación.    | Personalización de marca totalmente personalizable por aplicación u organización.   |
| **Modelo de facturación** | Los [precios de las identidades externas](https://azure.microsoft.com/en-us/pricing/details/active-directory/external-identities/) se basan en los usuarios activos mensuales (MAU). <br>(Consulte también: [Detalles de la configuración B2B](external-identities-pricing.md)) | Los [precios de las identidades externas](https://azure.microsoft.com/en-us/pricing/details/active-directory/external-identities/) se basan en los usuarios activos mensuales (MAU). <br>(Consulte también: [Detalles de la configuración B2C](../../active-directory-b2c/billing.md)) |
| **Más información** | [Entrada de blog](https://blogs.technet.microsoft.com/enterprisemobility/2017/02/01/azure-ad-b2b-new-updates-make-cross-business-collab-easy/), [Documentación](what-is-b2b.md)                   | [página de producto](https://azure.microsoft.com/services/active-directory-b2c/), [documentación](../../active-directory-b2c/index.yml)       |

Proteja y administre los clientes y asociados más allá de los límites organizacionales con Azure AD External Identities.

## <a name="about-multitenant-applications"></a>Acerca de las aplicaciones multiinquilino

Si va a proporcionar una aplicación como servicio y no desea administrar las cuentas de usuario de los clientes, es probable que una aplicación multiinquilino sea la opción más adecuada. Al desarrollar aplicaciones para otros inquilinos de Azure AD, puede dirigirse a usuarios de una única organización (inquilino único) o usuarios de cualquier organización que ya tenga un inquilino de Azure AD (aplicaciones multiinquilino). Los registros de aplicaciones en Azure AD son de inquilino único de forma predeterminada, pero puede hacer que el registro sea multiinquilino. Usted mismo registra una vez la aplicación multiinquilino en su propia instancia de Azure AD. Pero, después, cualquier usuario de Azure AD de cualquier organización puede usar la aplicación sin ningún trabajo adicional por su parte. Para más información, consulte [Administración de identidades en aplicaciones multiinquilino](/azure/architecture/multitenant-identity/), [Guía de procedimientos](../develop/howto-convert-app-to-be-multi-tenant.md).

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Acerca de Azure AD B2C](../../active-directory-b2c/overview.md)