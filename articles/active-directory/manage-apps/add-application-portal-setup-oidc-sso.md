---
title: 'Inicio rápido: Configuración del inicio de sesión único (SSO) basado en OIDC de una aplicación en el inquilino de Azure Active Directory (Azure AD)'
description: En este inicio rápido se describe el proceso de configuración del inicio de sesión único (SSO) basado en OIDC de una aplicación en el inquilino de Azure Active Directory (Azure AD).
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 07/01/2020
ms.author: mtillman
ms.openlocfilehash: 853ae1bf9bb230a28ad4e1e8509ddbeeb9dc34d8
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112077508"
---
# <a name="quickstart-set-up-oidc-based-single-sign-on-sso-for-an-application-in-your-azure-active-directory-azure-ad-tenant"></a>Inicio rápido: Configuración del inicio de sesión único (SSO) basado en OIDC de una aplicación en el inquilino de Azure Active Directory (Azure AD)

Comience a usar inicios de sesión de usuario simplificados mediante la configuración del inicio de sesión único (SSO) para una aplicación que haya agregado al inquilino de Azure Active Directory (Azure AD). Después de configurar el inicio de sesión único, los usuarios podrán iniciar sesión en una aplicación con sus credenciales de Azure AD. El inicio de sesión único se incluye en la edición gratuita de Azure AD.

Para más información sobre el inicio de sesión único basado en OIDC, consulte [Descripción del inicio de sesión único basado en OIDC](configure-oidc-single-sign-on.md).

## <a name="prerequisites"></a>Requisitos previos

Para configurar el inicio de sesión único de una aplicación que haya agregado a su inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Uno de los siguientes roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o Propietario de la entidad de servicio.
- Una aplicación que admita el inicio de sesión único y que se haya configurado previamente y agregado ya a la galería de Azure AD. La mayoría de las aplicaciones pueden usar Azure AD para el inicio de sesión único. Las aplicaciones de la galería de Azure AD están preconfiguradas. Aunque la aplicación no aparezca en la lista, o se trate de una aplicación desarrollada de forma personalizada, podrá usarla de todas formas con Azure AD. Consulte los tutoriales y otra documentación adicional en la tabla de contenido. Este inicio rápido se centra en las aplicaciones que se configuraron previamente para el inicio de sesión único, y que los desarrolladores de aplicaciones han agregado a la galería de Azure AD.
- Opcional: Haber finalizado el inicio rápido de [Visualización de las aplicaciones](view-applications-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Incorporación de una aplicación](add-application-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Configuración de una aplicación](add-application-portal-configure.md).
- Opcional: haber finalizado [Asignación de usuarios a una aplicación](add-application-portal-assign-users.md).

>[!IMPORTANT]
>Use un entorno que no sea de producción para probar los pasos de esta guía de inicio rápido.

## <a name="enable-single-sign-on-for-an-app"></a>Habilitar el inicio de sesión único de una aplicación

Al agregar una aplicación que usa el OIDC estándar para el inicio de sesión único, tendrá un botón de configuración. Al seleccionarlo, irá al sitio de aplicaciones para completar el proceso de registro de la aplicación. El proceso de incorporación de una aplicación se trata en el inicio rápido con la incorporación de una aplicación anterior en esta serie. Si está configurando una aplicación que ya se había agregado, consulte el primer inicio rápido. Este le guía por la visualización de las aplicaciones existentes en el inquilino. 

Para configurar el inicio de sesión único de una aplicación:

1. En el inicio rápido anterior en esta serie, aprendió a agregar una aplicación que usará el inquilino de Azure AD para la administración de identidades. Si el desarrollador de la aplicación usó el OIDC estándar para implementar el inicio de sesión único, se le presentará un botón de registro al agregar la aplicación. 

    :::image type="content" source="media/add-application-portal-setup-oidc-sso/sign-up-oidc-sso.png" alt-text="Captura de pantalla que muestra la opción Inicio de sesión único y el botón de registro." lightbox="media/add-application-portal-setup-oidc-sso/sign-up-oidc-sso.png":::


2. Seleccione **Registrarse** y se abrirá la página de inicio de sesión para desarrolladores de aplicaciones. Inicie sesión con las credenciales de Azure Active Directory. 

   > [!IMPORTANT]
    > Si ya tiene una suscripción a la aplicación, se producirá la validación de los detalles del usuario y la información del directorio o del inquilino. Si la aplicación no puede comprobar el usuario, se le redirigirá a registrarse en el servicio de aplicación o a la página de error.

3. Después de la autenticación aparece un cuadro de diálogo que solicita el consentimiento del administrador. Seleccione **Consentimiento en nombre de la organización** y, luego, **Aceptar**. Para más información sobre el consentimiento del usuario y del administrador, consulte [Consentimiento del usuario y del administrador](../develop/howto-convert-app-to-be-multi-tenant.md#understand-user-and-admin-consent).

    :::image type="content" source="media/add-application-portal-setup-oidc-sso/consent.png" alt-text="Captura de pantalla que muestra la pantalla de consentimiento de una aplicación." lightbox="media/add-application-portal-setup-oidc-sso/consent.png":::

4. La aplicación se agrega al inquilino y la página principal de la aplicación aparece.


> [!TIP]
> La administración de aplicaciones se puede automatizar mediante Graph API, consulte el artículo sobre la [automatización de la administración de aplicaciones con Microsoft Graph API](/graph/application-saml-sso-configure-api).

Este es un vídeo en el que se explica de forma más detallada cómo agregar una aplicación basada en OIDC a Azure AD.

Adición de una aplicación basada en OIDC en Azure Active Directory

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4HoNI]

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con esta serie de inicios rápidos, considere la posibilidad de eliminar la aplicación para limpiar el inquilino de prueba. La eliminación de la aplicación se trata en el último inicio rápido de esta serie, consulte [Eliminación de una aplicación](delete-application-portal.md).

## <a name="next-steps"></a>Pasos siguientes

En el siguiente artículo aprenderá a eliminar una aplicación.
> [!div class="nextstepaction"]
> [Eliminación de una aplicación](delete-application-portal.md)
