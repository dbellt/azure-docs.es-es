---
title: Configuración de la autenticación de Twitter
description: Aprenda a configurar la autenticación de Twitter como proveedor de identidades para una aplicación App Service o Azure Functions.
ms.assetid: c6dc91d7-30f6-448c-9f2d-8e91104cde73
ms.topic: article
ms.date: 03/29/2021
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: 6ecce954991d9f3901c54a6f87fc803b32469862
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077982"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-twitter-login"></a>Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Twitter

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este artículo se muestra cómo configurar Azure App Service o Azure Functions para usar Twitter como proveedor de autenticación.

Para llevar a cabo el procedimiento descrito en este artículo, debe tener una cuenta de Twitter asociada a una dirección de correo electrónico verificada y a un número de teléfono. Para crear una cuenta de Twitter, vaya a [twitter.com].

## <a name="register-your-application-with-twitter"></a><a name="register"> </a>Registrar una aplicación con Twitter

1. Inicie sesión en [Azure Portal] y vaya a la aplicación. Copie el valor de **Dirección URL**. Lo usará para configurar la aplicación de Twitter.
1. Vaya al sitio web para [desarrolladores de Twitter], inicie sesión con las credenciales de la cuenta de Twitter y seleccione **Create an app** (Crear una aplicación).
1. Introduzca el **nombre de la aplicación** y la **descripción de la aplicación** para su nueva aplicación. Pegue la **dirección URL** de la aplicación en el campo **Website URL** (Dirección URL del sitio web). En la sección **Callback URLs** (Direcciones URL de devolución de llamada), escriba la dirección URL HTTPS de la aplicación de App Service y anexe la ruta de acceso`/.auth/login/twitter/callback`. Por ejemplo, `https://contoso.azurewebsites.net/.auth/login/twitter/callback`.
1. En la parte inferior de la página, escriba al menos 100 caracteres en **Tell us how this app will be used** (Infórmenos de cómo se usará esta aplicación) y, después, seleccione **Create** (Crear). Vuelva a hacer clic en **Create** (Crear) en el elemento emergente. Se muestran los detalles de la aplicación.
1. Seleccione la pestaña **Claves y tokens de acceso** .

   Anote estos valores:
   - Clave de API
   - Clave secreta de API

   > [!IMPORTANT]
   > La clave secreta de API es una credencial de seguridad importante. por lo que no debe compartirlo con nadie ni distribuirlo con su aplicación.

## <a name="add-twitter-information-to-your-application"></a><a name="secrets"> </a>Adición de información de Twitter a su aplicación

1. Inicie sesión en [Azure Portal] y vaya a la aplicación.
1. Seleccione **Autenticación** en el menú de la izquierda. Haga clic en **Agregar proveedor de identidades**.
1. Seleccione **Twitter** en la lista desplegable de proveedores de identidades. Pegue los valores `API key` y `API secret key` que obtuvo anteriormente.

    El secreto se almacenará como una [configuración de la aplicación](./configure-common.md#configure-app-settings) con espacios fijos denominada `TWITTER_PROVIDER_AUTHENTICATION_SECRET`. Puede actualizar esa configuración más adelante para usar [referencias de Key Vault](./app-service-key-vault-references.md) si desea administrar el secreto en Azure Key Vault.

1. Si este es el primer proveedor de identidades configurado para la aplicación, también se mostrará una sección de **configuración de autenticación de App Service**. De lo contrario, puede pasar al siguiente paso.
    
    Estas opciones determinan el modo en que la aplicación responde a las solicitudes no autenticadas y las selecciones predeterminadas redirigirán todas las solicitudes para iniciar sesión con este nuevo proveedor. Puede cambiar este comportamiento ahora o ajustar esta configuración más adelante desde la pantalla principal **Autenticación**; para ello, elija **Editar** junto a **Configuración de la autenticación**. Para obtener más información acerca de estas opciones, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).

1. Haga clic en **Agregar**.

De este modo ya estará listo para usar Twitter para realizar la autenticación en la aplicación. El proveedor se mostrará en la pantalla **Autenticación**. Desde allí, puede editar o eliminar esta configuración de proveedor.

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- URLs. -->

[desarrolladores de Twitter]: https://go.microsoft.com/fwlink/p/?LinkId=268300
[twitter.com]: https://go.microsoft.com/fwlink/p/?LinkID=268287
[Azure Portal]: https://portal.azure.com/
[xamarin]: ../app-services-mobile-app-xamarin-ios-get-started-users.md
