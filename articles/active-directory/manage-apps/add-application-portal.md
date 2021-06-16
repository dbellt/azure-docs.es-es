---
title: 'Inicio rápido: Incorporación de una aplicación al inquilino de Azure Active Directory (Azure AD)'
description: Esta guía de inicio rápido usa Azure Portal para agregar una aplicación de la galería a su inquilino de Azure Active Directory (Azure AD).
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 10/29/2019
ms.author: mtillman
ms.openlocfilehash: e3748734d714812a40b78ff35fd9e0a31d3fa86e
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112077472"
---
# <a name="quickstart-add-an-application-to-your-azure-active-directory-azure-ad-tenant"></a>Inicio rápido: Incorporación de una aplicación al inquilino de Azure Active Directory (Azure AD)

Azure Active Directory (Azure AD) incluye una galería que contiene miles de aplicaciones previamente integradas. Muchas de las aplicaciones que su organización usa probablemente estén ya en la galería.

Después de agregar una aplicación a su inquilino de Azure AD, puede:

- Configurar las propiedades de la aplicación.
- Administrar el acceso de usuario a la aplicación con una directiva de acceso condicional.
- Configurar el inicio de sesión único para que los usuarios puedan iniciar sesión en la aplicación con sus credenciales de Azure AD.

## <a name="prerequisites"></a>Requisitos previos

Para agregar una aplicación al inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Uno de los siguientes roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o Propietario de la entidad de servicio.
- (Opcional: finalización de [Visualización de sus aplicaciones](view-applications-portal.md)).

>[!IMPORTANT]
>Para probar los pasos de este inicio rápido, se recomienda usar un entorno que no sea de producción.

## <a name="add-an-app-to-your-azure-ad-tenant"></a>Incorporación de una aplicación al inquilino de Azure AD

Para agregar una aplicación a un inquilino de Azure AD:

1. En [Azure Portal](https://portal.azure.com), en el panel de navegación izquierdo, seleccione **Azure Active Directory**.
2. En el panel **Azure Active Directory**, seleccione **Aplicaciones empresariales**. Se abre el panel **Todas las aplicaciones**, en el que se ve una muestra aleatoria de las aplicaciones que hay en su inquilino de Azure AD.
3. En el panel **Aplicaciones empresariales**, seleccione **Nueva aplicación**. 
    ![Selección de Nueva aplicación para agregar una aplicación de la galería al inquilino](media/add-application-portal/new-application.png)
4. Cambie a la nueva experiencia de versión preliminar de la galería: En el banner situado en la parte superior de la **página para agregar una aplicación**, seleccione el vínculo que indica **Haga clic aquí para probar la nueva y mejorada experiencia de la galería de aplicaciones.** .
5. Se abre el panel **Examinar la Galería de Azure AD (versión preliminar)** , que muestra iconos para plataformas en la nube, aplicaciones locales y aplicaciones destacadas. Las aplicaciones que aparecen en la sección **Aplicaciones destacadas** incluyen iconos que indican si admiten el inicio de sesión único federado (SSO) y el aprovisionamiento. 
    ![Búsqueda de una aplicación por nombre o categoría](media/add-application-portal/browse-gallery.png)
6. Puede buscar en la galería la aplicación que quiere agregar o escribir su nombre en el cuadro de búsqueda para localizarla. Después, seleccione la aplicación en los resultados. 
7. El paso siguiente depende de la forma en que el desarrollador de la aplicación haya implementado el inicio de sesión único (SSO). Los desarrolladores de aplicaciones pueden implementar el inicio de sesión único de cuatro maneras: SAML, OpenID Connect, Contraseña y Vinculado. Al agregar una aplicación, puede elegir filtrar y ver solo las aplicaciones que usan una implementación de inicio de sesión único determinada, como se muestra en la captura de pantalla. Por ejemplo, un estándar conocido para implementar el inicio de sesión único es Lenguaje de marcado de aserción de seguridad (SAML). Otro estándar conocido es OpenId Connect (OIDC). La forma de configurar el inicio de sesión único con estos estándares es diferente, por lo que debe tener en cuenta el tipo de inicio de sesión único implementado por la aplicación que va a agregar.

    :::image type="content" source="media/add-application-portal/sso-types.png" alt-text="Captura de pantalla que muestra el selector de tipos de inicio de sesión único." lightbox="media/add-application-portal/sso-types.png":::

    - Si el desarrollador de la aplicación usó el **estándar OIDC** para el inicio de sesión único, seleccione **Registrarse**. Aparece una página de instalación. A continuación, vaya al inicio rápido sobre cómo configurar el inicio de sesión único basado en OIDC.
    :::image type="content" source="media/add-application-portal/sign-up-oidc-sso.png" alt-text="Captura de pantalla que muestra cómo agregar una aplicación de inicio de sesión único basado en OIDC.":::

    - Si el desarrollador de la aplicación usó el **estándar SAML** para el inicio de sesión único, seleccione **Crear**. Aparece una página de introducción con las opciones para configurar la aplicación para su organización. En el formulario, puede editar el nombre de la aplicación para que se ajuste a las necesidades de su organización. A continuación, vaya al inicio rápido sobre cómo configurar el inicio de sesión único basado en SAML.
    :::image type="content" source="media/add-application-portal/create-application.png" alt-text="Captura de pantalla que muestra cómo agregar una aplicación de inicio de sesión único basado en SAML.":::


> [!IMPORTANT]
> Existen algunas diferencias importantes entre las implementaciones del inicio de sesión único basado en SAML y basado en OIDC. Con las aplicaciones basadas en SAML, puede agregar varias instancias de la misma aplicación. Por ejemplo, GitHub1, GitHub2, etc. En el caso de las aplicaciones basadas en OIDC, solo puede agregar una instancia de una aplicación. Si ya ha agregado una aplicación basada en OIDC e intenta agregar la misma aplicación de nuevo y proporciona el consentimiento dos veces, no se agregará de nuevo en el inquilino.

Si la aplicación que busca no está en la galería, puede hacer clic en el vínculo **Cree su propia aplicación** y, luego, en **¿Cuál es el objetivo de utilizar la aplicación?** , elija **Integrar cualquier otra aplicación que no se encuentre en la galería**. Microsoft ha trabajado con muchos desarrolladores de aplicaciones para configurar estas previamente con el fin de que funcionen con Azure AD. Las aplicaciones preconfiguradas se muestran en la galería. Pero si la aplicación que quiere agregar no aparece en la lista, puede crear una nueva, genérica y configurarla usted mismo o siguiendo las instrucciones del desarrollador que la haya creado.

Ha terminado de agregar una aplicación. El siguiente inicio rápido muestra cómo cambiar el logotipo y modificar otras propiedades de la aplicación.

> [!TIP]
> La administración de aplicaciones se puede automatizar mediante Graph API, consulte el artículo sobre la [automatización de la administración de aplicaciones con Microsoft Graph API](/graph/application-saml-sso-configure-api).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a continuar con la serie de inicios rápidos, considere la posibilidad de eliminar la aplicación para limpiar el inquilino de prueba. La eliminación de la aplicación se trata en el último inicio rápido de esta serie, consulte [Eliminación de una aplicación](delete-application-portal.md).

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo, donde aprenderá a configurar una aplicación.
> [!div class="nextstepaction"]
> [Configuración de una aplicación](add-application-portal-configure.md)