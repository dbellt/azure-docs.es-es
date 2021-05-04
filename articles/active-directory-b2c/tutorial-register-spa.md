---
title: 'Tutorial: Registro de una aplicación de página única'
titleSuffix: Azure AD B2C
description: Siga este tutorial para aprender a registrar una aplicación de página única en Azure Active Directory B2C con Azure Portal.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 08/19/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 1f79eee17a78bd5cfc9024a333142deaaa44b3e2
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108071143"
---
# <a name="tutorial-register-a-single-page-application-spa-in-azure-active-directory-b2c"></a>Tutorial: Registro de una aplicación de página única en Azure Active Directory B2C

Para que sus [aplicaciones](application-types.md) puedan interactuar con Azure Active Directory B2C (Azure AD B2C), deben estar registradas en un inquilino que administre. En este tutorial se muestra cómo registrar una aplicación de página única (SPA) mediante Azure Portal.

## <a name="overview-of-authentication-options"></a>Introducción a las opciones de autenticación

Muchas aplicaciones web modernas se compilan como aplicaciones de página única (SPA) del lado cliente. Los desarrolladores las escriben mediante JavaScript o un marco SPA, como Angular, Vue y React. Estas aplicaciones se ejecutan en un explorador web y tienen características de autenticación diferentes a las de las aplicaciones web del lado servidor tradicionales.

Azure AD B2C ofrece **dos** opciones para que las aplicaciones de página única inicien la sesión de los usuarios y obtengan tokens para acceder a los servicios de back-end o a las API web:

### <a name="authorization-code-flow-with-pkce"></a>Flujo de códigos de autorización (con PKCE)
- [Flujo de códigos de autorización de OAuth 2.0 (con PKCE)](./authorization-code-flow.md) El flujo del código de autorización permite a la aplicación intercambiar un código de autorización para los tokens de **identificador** para representar al usuario autenticado y también los tokens de **acceso** necesarios para llamar a las API protegidas. Además, devuelve tokens de **actualización** que proporcionan acceso a largo plazo a los recursos en nombre de los usuarios sin necesidad de interacción con estos. 

Este es el enfoque **recomendado**. Contar con tokens de actualización de duración limitada ayuda a que la aplicación se adapte a las [limitaciones de privacidad de cookies de los exploradores modernos](../active-directory/develop/reference-third-party-cookies-spas.md), como ITP de Safari.

Para aprovechar este flujo, la aplicación puede usar una biblioteca de autenticación que lo admita, como [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser). 

![Aplicaciones de página única: autorización](./media/tutorial-single-page-app/spa-app-auth.svg)

### <a name="implicit-grant-flow"></a>Flujo de concesión implícito
- [Flujo implícito de OAuth 2.0](implicit-flow-single-page-application.md). Algunas bibliotecas, como [MSAL.js 1.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core), solo admiten el flujo de concesión implícita. El flujo de concesión implícita permite a la aplicación obtener tokens de **identificador** y de **acceso**. A diferencia del flujo del código de autorización, el flujo de concesión implícita no devuelve un **token de actualización**. 

![Aplicaciones de página única: implícito](./media/tutorial-single-page-app/spa-app.svg)

Este flujo de autenticación no incluye escenarios de aplicaciones que usan marcos de JavaScript multiplataforma, como Electron y React Native, ya que requieren más funcionalidades para la interacción con las plataformas nativas.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si aún no ha creado su propio [inquilino de Azure AD B2C](tutorial-create-tenant.md), cree una ahora. Puede usar un inquilino de Azure AD B2C existente.

## <a name="register-the-spa-application"></a>Registro de la aplicación SPA

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba un **Nombre** para la aplicación. Por ejemplo, *spaapp1*.
1. En **Tipos de cuenta compatibles**, seleccione **Cuentas en cualquier proveedor de identidades o directorio de la organización (para autenticar usuarios con flujos de usuario)** .
1. En **URI de redirección**, seleccione **Aplicación de página única (SPA)** y, a continuación, escriba `https://jwt.ms` en el cuadro de texto de dirección URL.

    El URI de redirección es el punto de conexión al que el servidor de autorización envía al usuario (Azure AD B2C, en este caso) después de completar su interacción con este, y al que se envía un token de acceso o un código de autorización tras una autorización correcta. En una aplicación de producción, suele ser un punto de conexión accesible públicamente donde se ejecuta la aplicación, como `https://contoso.com/auth-response`. Con fines de prueba como este tutorial, puede establecerlo en `https://jwt.ms`, una aplicación web propiedad de Microsoft que muestra el contenido descodificado de un token (el contenido del token nunca sale del explorador). Durante el desarrollo de aplicaciones, puede agregar el punto de conexión en el que la aplicación realiza escuchas localmente, como `http://localhost:5000`. Puede agregar y modificar los URI de redireccionamiento en las aplicaciones registradas en cualquier momento.

    Las siguientes restricciones se aplican a los URI de redireccionamiento:

    * La dirección URL de respuesta debe comenzar con el esquema `https`, a menos que use `localhost`.
    * La dirección URL de respuesta distingue mayúsculas de minúsculas. Sus mayúsculas o minúsculas deben coincidir con las de la ruta de acceso de la dirección URL de la aplicación en ejecución. Por ejemplo, si la aplicación incluye como parte de su ruta de acceso `.../abc/response-oidc`, no especifique `.../ABC/response-oidc` en la dirección URL de respuesta. Dado que el explorador web tiene en cuenta las mayúsculas y minúsculas de la ruta de acceso, se pueden excluir las cookies asociadas con `.../abc/response-oidc` si se redirigen a la dirección URL `.../ABC/response-oidc` con mayúsculas y minúsculas no coincidentes.

1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
1. Seleccione **Registrar**.


## <a name="enable-the-implicit-flow"></a>Habilitar el flujo implícito
Si usa el flujo implícito, debe habilitar el flujo de concesión implícita en el registro de la aplicación.

1. En el menú izquierdo, en **Administrar**, seleccione **Autenticación**.
1. En **Concesión implícita**, active las casillas **Tokens de acceso** y **Tokens de id.**
1. Seleccione **Guardar**.

## <a name="migrate-from-the-implicit-flow"></a>Migración desde el flujo implícito

Si tiene una aplicación que usa el flujo implícito, se recomienda cambiar al flujo de código de autorización mediante el uso de un marco que lo admita, como [MSAL.js 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser).

Cuando todas las aplicaciones de página única en producción representadas por un registro de aplicaciones estén usando el flujo de código de autorización, deshabilite la configuración de flujo de concesión implícita. 

1. En el menú izquierdo, en **Administrar**, seleccione **Autenticación**.
1. En **Concesión implícita**, desactive las casillas **Tokens de acceso** y **Tokens de id.**
1. Seleccione **Guardar**.

Las aplicaciones que usan el flujo implícito pueden seguir funcionando si deja el flujo implícito habilitado (activado).

* * *

## <a name="next-steps"></a>Pasos siguientes

A continuación, aprenda a crear flujos de usuario para permitir que los usuarios se registren, inicien sesión y administren sus perfiles.

> [!div class="nextstepaction"]
> [Creación de flujos de usuario en Azure Active Directory B2C >](tutorial-create-user-flows.md)
