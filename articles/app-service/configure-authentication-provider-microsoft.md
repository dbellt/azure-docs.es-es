---
title: Configuración de la autenticación de Microsoft
description: Aprenda a configurar la autenticación de la cuenta Microsoft como proveedor de identidades para una aplicación App Service o Azure Functions.
ms.assetid: ffbc6064-edf6-474d-971c-695598fd08bf
ms.topic: article
ms.date: 03/29/2021
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: 5e7b4c7ed1950b2fa585639a3097cc1f5688d739
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077961"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-microsoft-account-login"></a>Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de la cuenta Microsoft.

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

En este tema se muestra cómo configurar Azure App Service o Azure Functions para usar AAD con el fin de admitir inicios de sesión de cuentas de Microsoft personales.

> [!IMPORTANT]
> Aunque todavía se admite el proveedor Cuenta de Microsoft, se recomienda que las aplicaciones usen el proveedor [Plataforma de identidad de Microsoft (Azure AD)](./configure-authentication-provider-aad.md) en su lugar. La Plataforma de identidad de Microsoft ofrece compatibilidad con las cuentas de la organización y las cuentas personales de Microsoft.

## <a name="register-your-app-with-microsoft-account"></a><a name="register-microsoft-account"> </a>Registro de la aplicación con la cuenta de Microsoft

1. Vaya a [**Registros de aplicaciones**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) en Azure Portal. Si es necesario, inicie sesión con la cuenta de Microsoft.
1. Seleccione **Nuevo registro** y escriba el nombre de la aplicación.
1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Cuentas en cualquier directorio organizativo (cualquier directorio de Azure AD: multiinquilino) y cuentas de Microsoft personales (como Skype o Xbox)** .
1. En **URI de redirección**, seleccione **Web** y escriba `https://<app-domain-name>/.auth/login/aad/callback`. Reemplace *\<app-domain-name>* por el nombre de dominio de la aplicación.  Por ejemplo, `https://contoso.azurewebsites.net/.auth/login/aad/callback`. Asegúrese de usar el esquema HTTPS en la dirección URL.

1. Seleccione **Registrar**.
1. Copie el **id. de la aplicación (cliente)** . Lo necesitará más adelante.
1. En el panel izquierdo, seleccione **Certificados y secretos** > **Nuevo secreto de cliente**. Escriba una descripción, seleccione la duración de validez y después seleccione **Agregar**.
1. Copie el valor que aparece en la página **Certificados y secretos**. Una vez que salga de esta página, ya no se volverá a mostrar.

    > [!IMPORTANT]
    > El valor del secreto de cliente (contraseña) es una credencial de seguridad importante. No la comparta con nadie ni la distribuya en una aplicación cliente.

## <a name="add-microsoft-account-information-to-your-app-service-application"></a><a name="secrets"> </a>Incorporación de información de la cuenta de Microsoft a la aplicación de App Service

1. Vaya a la aplicación en [Azure Portal].
1. Haga clic en **Configuración** > **Autenticación/autorización** y asegúrese de que la opción **Autenticación de App Service** está **Activada**.
1. En **Proveedores de autenticación,** seleccione **Azure Active Directory**. Seleccione **Avanzado** en **Modo de administración**. Pegue el identificador de la aplicación (cliente) y el secreto de cliente que obtuvo anteriormente. Use **`https://login.microsoftonline.com/9188040d-6c67-4c5b-b112-36a304b66dad/v2.0`** para el campo **Dirección URL del emisor**.
1. Seleccione **Aceptar**.

   App Service ofrece autenticación pero no restringe el acceso autorizado al contenido del sitio ni a las API. Debe autorizar a los usuarios en el código de la aplicación.

1. (Opcional) Para restringir el acceso a los usuarios de la cuenta de Microsoft, establezca **Acción necesaria cuando la solicitud no está autenticada** en **Iniciar sesión con Azure Active Directory**. Al establecer esta funcionalidad, la aplicación requiere que se autentiquen todas las solicitudes. También redirige todas las solicitudes no autenticadas para usar AAD para la autenticación. Tenga en cuenta que, dado que ha configurado la **dirección URL del emisor** para usar el inquilino de la cuenta de Microsoft, solo las cuentas personales se autenticarán correctamente.

   > [!CAUTION]
   > Este método de restricción del acceso se aplica a todas las llamadas a la aplicación, lo que puede no interesar en las aplicaciones que tienen una página principal disponible públicamente así como en muchas aplicaciones de página única. Para tales aplicaciones, puede ser preferible **Permitir solicitudes anónimas (ninguna acción)** y que la aplicación inicie manualmente la autenticación. Para más información, consulte [Flujo de autenticación](overview-authentication-authorization.md#authentication-flow).

1. Seleccione **Guardar**.

De este modo ya estará listo para usar la cuenta Microsoft para realizar la autenticación en la aplicación.

## <a name="next-steps"></a><a name="related-content"> </a>Pasos siguientes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- URLs. -->

[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Azure Portal]: https://portal.azure.com/
