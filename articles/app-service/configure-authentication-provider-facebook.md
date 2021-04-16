---
title: Configuración de la autenticación de Facebook
description: Aprenda a configurar la autenticación de Facebook como proveedor de identidades para una aplicación de App Service o Azure Functions.
ms.assetid: b6b4f062-fcb4-47b3-b75a-ec4cb51a62fd
ms.topic: article
ms.date: 03/29/2021
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: bc639eaea76b3309d6ed047e73c726040da19639
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106078016"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-facebook-login"></a>Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Facebook

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este artículo se muestra cómo configurar Azure App Service o Azure Functions para usar Facebook como proveedor de autenticación.

Para llevar a cabo el procedimiento descrito en este artículo, debe tener una cuenta de Facebook asociada a una dirección de correo electrónico verificada y a un número de teléfono móvil. Para crear una cuenta de Facebook, vaya a [facebook.com].

## <a name="register-your-application-with-facebook"></a><a name="register"> </a>Registro de la aplicación con Facebook

1. Vaya al sitio web para [desarrolladores de Facebook] e inicie sesión con las credenciales de su cuenta de Facebook.

   Si no tiene una cuenta de Facebook for Developers, seleccione **Empezar** y siga los pasos de registro.
1. Seleccione **My Apps** > **Add New App** (Mis aplicaciones > Agregar nueva aplicación).
1. En el campo **Display Name** (Nombre para mostrar):
   1. Escriba un nombre único para la aplicación.
   1. Indique el valor de **Contact Email** (Correo electrónico de contacto).
   1. Seleccione **Create App ID** (Crear identificador de aplicación).
   1. Complete la comprobación de seguridad.

   Se abre el panel del desarrollador de la nueva aplicación de Facebook.
1. Haga clic en **Dashboard** > **Facebook Login** > **Set up** > **Web** (Panel > Inicio de sesión de Facebook > Configurar > Web).
1. En el panel de navegación izquierdo, en **Facebook Login** (Inicio de sesión de Facebook), haga clic en **Settings** (Configuración).
1. En el campo **Valid OAuth redirect URIs** (URI de redireccionamiento OAuth válidos), escriba `https://<app-name>.azurewebsites.net/.auth/login/facebook/callback`. No olvide reemplazar `<app-name>` por el nombre de la aplicación de Azure App Service.
1. Seleccione **Save changes** (Guardar los cambios).
1. En el panel izquierdo, seleccione **Settings** > **Basic** (Configuración > Básica). 
1. En el campo **App Secret** (Secreto de aplicación), seleccione **Show** (Mostrar). Copie los valores de **ID de la aplicación** y **Secreto de la aplicación**. Más adelante los usará para configurar la aplicación de App Service en Azure.

   > [!IMPORTANT]
   > El secreto de aplicación es una credencial de seguridad importante, No comparta este secreto con nadie ni lo distribuya en una aplicación cliente.
   >

1. La cuenta de Facebook que utilizó para registrar la aplicación es un administrador de la aplicación. En este momento, solo los administradores pueden iniciar sesión en esta aplicación.

   Para autenticar otras cuentas de Facebook, seleccione **App Review** (Revisión de aplicaciones) y habilite **Make \<your-app-name> public** (Hacer pública) para permitir que el público general acceda a la aplicación mediante la autenticación de Facebook.

## <a name="add-facebook-information-to-your-application"></a><a name="secrets"> </a>Agregar información de Facebook a la aplicación

1. Inicie sesión en [Azure Portal] y vaya a la aplicación.
1. Seleccione **Autenticación** en el menú de la izquierda. Haga clic en **Agregar proveedor de identidades**.
1. Seleccione **Facebook** en la lista desplegable de proveedores de identidades. Pegue los valores de identificador y secreto de la aplicación que obtuvo anteriormente.

    El secreto se almacenará como una [configuración de la aplicación](./configure-common.md#configure-app-settings) con espacios fijos denominada `FACEBOOK_PROVIDER_AUTHENTICATION_SECRET`. Puede actualizar esa configuración más adelante para usar [referencias de Key Vault](./app-service-key-vault-references.md) si desea administrar el secreto en Azure Key Vault.

1. Si este es el primer proveedor de identidades configurado para la aplicación, también se mostrará una sección de **configuración de autenticación de App Service**. De lo contrario, puede pasar al siguiente paso.
    
    Estas opciones determinan el modo en que la aplicación responde a las solicitudes no autenticadas y las selecciones predeterminadas redirigirán todas las solicitudes para iniciar sesión con este nuevo proveedor. Puede cambiar este comportamiento ahora o ajustar esta configuración más adelante desde la pantalla principal **Autenticación**; para ello, elija **Editar** junto a **Configuración de la autenticación**. Para obtener más información acerca de estas opciones, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).

1. (Opcional) Haga clic en **Siguiente: Ámbitos** y agregue los ámbitos que necesite la aplicación. Estos se solicitarán en el momento del inicio de sesión para los flujos basados en explorador.
1. Haga clic en **Agregar**.

De este modo ya estará listo para usar Facebook para realizar la autenticación en la aplicación. El proveedor se mostrará en la pantalla **Autenticación**. Desde allí, puede editar o eliminar esta configuración de proveedor.

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- URLs. -->
[Desarrolladores de Facebook]: https://go.microsoft.com/fwlink/p/?LinkId=268286
[facebook.com]: https://go.microsoft.com/fwlink/p/?LinkId=268285
[Get started with authentication]: /en-us/develop/mobile/tutorials/get-started-with-users-dotnet/
[Azure Portal]: https://portal.azure.com/
