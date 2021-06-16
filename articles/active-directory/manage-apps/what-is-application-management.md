---
title: Qué es la administración de aplicaciones en Azure Active Directory
description: Información general sobre el uso de Azure Active Directory (AD) como sistema de administración de identidades y acceso (IAM) para aplicaciones locales y en la nube.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: overview
ms.workload: identity
ms.date: 01/22/2021
ms.author: mtillman
ms.reviewer: ''
ms.openlocfilehash: 31bfbe1405e1180f8909333da8947d72d68dd7db
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112079362"
---
# <a name="what-is-application-management"></a>¿Qué es la administración de aplicaciones?

Azure AD es un sistema de administración de identidades y acceso (IAM). Proporciona un lugar centralizado para almacenar la información de las identidades digitales. Puede configurar las aplicaciones de software para que usen Azure AD como el lugar donde se almacena la información del usuario. 

Azure AD debe estar configurada para poder integrarse con una aplicación. En otras palabras, debe saber qué aplicaciones lo usan para las identidades. Hacer que Azure AD conozca estas aplicaciones y cómo debe controlarlas, se conoce como administración de aplicaciones.

Las aplicaciones se administran en la página **Aplicaciones empresariales**, que se encuentra en la sección Administrar del portal de Azure Active Directory.

![Opción Aplicaciones empresariales de la sección Administrar del portal de Azure AD.](media/what-is-application-management/enterprise-applications-in-nav.png)

## <a name="what-is-an-identity-and-access-management-iam-system"></a>¿Qué es un sistema de administración de identidades y acceso (IAM)?
Una aplicación es un fragmento de software que se usa para algún propósito. La mayoría de las aplicaciones requieren que los usuarios inicien sesión.

Un sistema de identidades centralizado proporciona un único lugar para almacenar la información de los usuario que, después, todas las aplicaciones pueden usar. Estos sistemas se conocen como sistemas de administración de identidades y acceso (IAM). Azure Active Directory es el sistema de IAM para la nube de Microsoft.

>[!TIP]
>Un sistema de IAM proporciona un lugar centralizado en el que hacer un seguimiento de las identidades de los usuarios. Azure AD es el sistema de IAM de la nube de Microsoft.

## <a name="why-manage-applications-with-a-cloud-solution"></a>¿Por qué administrar aplicaciones con una solución en la nube?

Las organizaciones suelen tener cientos de aplicaciones de las que los usuarios dependen para poder realizar su trabajo. Los usuarios acceden a estas aplicaciones desde muchos dispositivos y ubicaciones. Constantemente se agregan, desarrollan y retiran nuevas aplicaciones. Con tantas aplicaciones y puntos de acceso, es importante usar una solución de identidad que funcione con todos.

>[!TIP]
>La galería de aplicaciones de Azure AD contiene muchas aplicaciones populares que ya están preconfiguradas para trabajar con Azure AD como proveedor de identidades.

## <a name="how-does-azure-ad-work-with-apps"></a>¿Cómo funciona Azure AD con las aplicaciones?

Azure AD se encuentra en el centro y proporciona administración de identidades tanto para las aplicaciones en la nube como para las locales. 

![Diagrama que muestra las aplicaciones federadas mediante Azure AD](media/what-is-application-management/app-management-overview.png)

>[!TIP]
>Para reducir los costos administrativos, [automatice el aprovisionamiento de usuarios](../app-provisioning/user-provisioning.md), con el fin de que los usuarios se pueden agregar automáticamente a Azure AD al agregarlos al sistema de recursos humanos de la empresa. 

## <a name="what-types-of-applications-can-i-integrate-with-azure-ad"></a>¿Qué tipos de aplicaciones puedo integrar con Azure AD?

Puede usar Azure AD como sistema de identidades de casi todas las aplicaciones. Muchas aplicaciones ya están configuradas previamente y pueden configurarse con un esfuerzo mínimo. Estas aplicaciones preconfiguradas se publican en la [Galería de aplicaciones de Azure AD](/azure/active-directory/saas-apps/). 

La mayoría de las aplicaciones se pueden configurar manualmente para el inicio de sesión único si no están ya en la galería. Azure AD proporciona varias opciones de inicio de sesión único. Algunas de los más populares son SSO basado en SAML y SSO basado en OIDC. Para más información sobre la integración de aplicaciones para habilitar el inicio de sesión único, consulte las [opciones de inicio de sesión único](sso-options.md). 

¿Su organización usa aplicaciones locales? Puede integrarlas mediante Proxy de aplicaciones. Para más información, consulte [Acceso remoto a aplicaciones locales mediante Azure Active Directory Application Proxy](../app-proxy/application-proxy.md).

>[!TIP]
>Al crear sus propias aplicaciones de línea de negocio, puede integrarlas con Azure AD para admitir el inicio de sesión único. Para más información sobre el desarrollo de aplicaciones para Azure AD, consulte [Plataforma de identidad de Microsoft](..//develop/v2-overview.md).

## <a name="manage-risk-with-conditional-access-policies"></a>Administración de riesgos con directivas de acceso condicional

El acoplamiento del inicio de sesión único (SSO) de Azure AD con [acceso condicional](../conditional-access/concept-conditional-access-cloud-apps.md) ofrece niveles altos de seguridad para acceder a las aplicaciones. Las directivas de acceso condicional proporcionan un control pormenorizado de las aplicaciones en función de las condiciones establecidas. 

## <a name="improve-productivity-with-single-sign-on"></a>Mejora de la seguridad con el inicio de sesión único

El inicio de sesión único proporciona una experiencia de usuario unificada entre Microsoft 365 y las demás aplicaciones que se usen. No será preciso volver a escribir constantemente el nombre de usuario y la contraseña.

Para más información sobre el inicio de sesión único, vea [¿Qué es el inicio de sesión único?](what-is-single-sign-on.md)

## <a name="address-governance-and-compliance"></a>Abordar la gobernanza y el cumplimiento

Supervise las aplicaciones a través de informes que usan herramientas de Security Incident and Event Monitoring (SIEM). Puede acceder a los informes desde el portal o con las API. Audite mediante programación quién accede a sus aplicaciones y retire el acceso a usuarios inactivos mediante revisiones de acceso.

## <a name="manage-costs"></a>Administrar costos

Al migrar a Azure AD, puede ahorrar costos y las molestias de administrar la infraestructura local. Azure AD también proporciona acceso de autoservicio a las aplicaciones, lo que ahorra tiempo para los administradores y usuarios. El inicio de sesión único elimina las contraseñas específicas de aplicaciones. Esta posibilidad de iniciar sesión solo una vez ahorra costos relacionados con el restablecimiento de contraseñas para las aplicaciones y la pérdida de productividad al recuperar estas.

En el caso de las aplicaciones que se centran es los recursos humanos u otras aplicaciones con un gran conjunto de usuarios, puede usar el aprovisionamiento de aplicaciones para facilitar el trabajo. El aprovisionamiento de aplicaciones automatiza el proceso de agregar y quitar usuarios. Para más información, consulte [¿Qué es el aprovisionamiento de aplicaciones?](../app-provisioning/user-provisioning.md)

## <a name="next-steps"></a>Pasos siguientes

- [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
- [Introducción a la integración de aplicaciones](plan-an-application-integration.md)
- [Obtenga información sobre cómo automatizar el aprovisionamiento](../app-provisioning/user-provisioning.md)