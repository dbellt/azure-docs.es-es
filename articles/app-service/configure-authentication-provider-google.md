---
title: Configuración de la autenticación de Google
description: Aprenda a configurar la autenticación de Google como proveedor de identidades para una aplicación App Service o Azure Functions.
ms.assetid: 2b2f9abf-9120-4aac-ac5b-4a268d9b6e2b
ms.topic: article
ms.date: 03/29/2021
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: f6bec32fa928e840569ed95c35a056db91ea9737
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077999"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-google-login"></a>Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Google

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este tema se muestra cómo configurar Azure App Service o Azure Functions para usar Google como proveedor de autenticación.

Para llevar a cabo el procedimiento descrito en este tema, debe tener una cuenta de Google asociada a una dirección de correo electrónico verificada. Para crear una cuenta de Google, vaya a [accounts.google.com](https://go.microsoft.com/fwlink/p/?LinkId=268302).

## <a name="register-your-application-with-google"></a><a name="register"> </a>Registro de la aplicación con Google

1. Siga la documentación de Google sobre el [inicio de sesión de Google para aplicaciones de servidor](https://developers.google.com/identity/sign-in/web/server-side-flow) para crear un identificador de cliente y un secreto de cliente. No es necesario realizar ningún cambio en el código. Use la siguiente información:
    - En **Authorized JavaScript Origins** (Orígenes de JavaScript autorizados), use `https://<app-name>.azurewebsites.net` con el nombre de la aplicación en *\<app-name>* .
    - En **Authorized Redirect URI** (URI de redireccionamiento autorizado), use `https://<app-name>.azurewebsites.net/.auth/login/google/callback`.
1. Copie los valores de identificador y secreto de la aplicación.

    > [!IMPORTANT]
    > El secreto de aplicación es una credencial de seguridad importante. No comparta este secreto con nadie ni lo distribuya en una aplicación cliente.

## <a name="add-google-information-to-your-application"></a><a name="secrets"> </a>Incorporación de información de Google a la aplicación

1. Inicie sesión en [Azure Portal] y vaya a la aplicación.
1. Seleccione **Autenticación** en el menú de la izquierda. Haga clic en **Agregar proveedor de identidades**.
1. Seleccione **Google** en la lista desplegable de proveedores de identidades. Pegue los valores de identificador y secreto de la aplicación que obtuvo anteriormente.

    El secreto se almacenará como una [configuración de la aplicación](./configure-common.md#configure-app-settings) con espacios fijos denominada `GOOGLE_PROVIDER_AUTHENTICATION_SECRET`. Puede actualizar esa configuración más adelante para usar [referencias de Key Vault](./app-service-key-vault-references.md) si desea administrar el secreto en Azure Key Vault.

1. Si este es el primer proveedor de identidades configurado para la aplicación, también se mostrará una sección de **configuración de autenticación de App Service**. De lo contrario, puede pasar al siguiente paso.
    
    Estas opciones determinan el modo en que la aplicación responde a las solicitudes no autenticadas y las selecciones predeterminadas redirigirán todas las solicitudes para iniciar sesión con este nuevo proveedor. Puede cambiar este comportamiento ahora o ajustar esta configuración más adelante desde la pantalla principal **Autenticación**; para ello, elija **Editar** junto a **Configuración de la autenticación**. Para obtener más información acerca de estas opciones, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).

1. (Opcional) Haga clic en **Siguiente: Ámbitos** y agregue los ámbitos que necesite la aplicación. Estos se solicitarán en el momento del inicio de sesión para los flujos basados en explorador.
1. Haga clic en **Agregar**.

De este modo ya estará listo para usar Google para realizar la autenticación en la aplicación. El proveedor se mostrará en la pantalla **Autenticación**. Desde allí, puede editar o eliminar esta configuración de proveedor.

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Anchors. -->

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-settings.png

<!-- URLs. -->

[Google apis]: https://go.microsoft.com/fwlink/p/?LinkId=268303

[Azure Portal]: https://portal.azure.com/

