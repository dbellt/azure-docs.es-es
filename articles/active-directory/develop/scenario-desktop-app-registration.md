---
title: Registro de aplicaciones de escritorio que llaman a las API web | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una aplicación de escritorio que llame a las API web (registro de aplicaciones).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/09/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: b0295c994a736e26d7b581bd13b6167819833360
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748716"
---
# <a name="desktop-app-that-calls-web-apis-app-registration"></a>Aplicación de escritorio que llama a las API web: Registro de aplicación

En este artículo se tratan los detalles del registro de una aplicación de escritorio.

## <a name="supported-account-types"></a>Tipos de cuenta admitidos

Los tipos de cuenta que se admiten en una aplicación de escritorio dependen de la experiencia que quiera destacar. Debido a esta relación, los tipos de cuenta admitidos dependen de los flujos que desee usar.

### <a name="audience-for-interactive-token-acquisition"></a>Público para la adquisición interactiva de tokens

Si la aplicación de escritorio usa la autenticación interactiva, puede iniciar la sesión de los usuarios desde cualquier [tipo de cuenta](quickstart-register-app.md).

### <a name="audience-for-desktop-app-silent-flows"></a>Audiencia para flujos silenciosos de la aplicación de escritorio

- Para usar la autenticación integrada o un nombre de usuario y una contraseña, la aplicación debe iniciar la sesión de los usuarios en su propio inquilino, por ejemplo, si es un desarrollador de línea de negocio (LOB). O bien, en organizaciones de Azure Active Directory, la aplicación debe iniciar la sesión de los usuarios en su propio inquilino si se trata de un escenario de ISV. Estos flujos de autenticación no son compatibles con las cuentas personales de Microsoft.
- Si inicia la sesión de los usuarios con identidades sociales que pasan una autoridad y directiva de negocio a comercio (B2C), solo puede usar la autenticación interactiva y de nombre de usuario y contraseña.

## <a name="redirect-uris"></a>URI de redirección

Los URI de redirección que se utilizan en una aplicación de escritorio dependen del flujo que se quiera utilizar.

Especifique el URI de redirección de la aplicación mediante la [configuración de la plataforma](quickstart-register-app.md#add-a-redirect-uri) de la aplicación en **Registros de aplicaciones** en Azure Portal.

- Para las aplicaciones que usan la autenticación interactiva:

  - Aplicaciones que usan exploradores incrustados: `https://login.microsoftonline.com/common/oauth2/nativeclient` (Nota: Si la aplicación abriría una ventana que normalmente no contiene ninguna barra de direcciones, se está usando el "explorador incrustado").
  - Aplicaciones que usan exploradores del sistema: `http://localhost` (Nota: Si la aplicación abriría el explorador predeterminado del sistema (como Edge, Chrome, Firefox, etc.) para visitar el portal de inicio de sesión de Microsoft, se está usando el "explorador del sistema").
  
  > [!IMPORTANT]
  > Como procedimiento recomendado de seguridad, se aconseja establecer explícitamente `https://login.microsoftonline.com/common/oauth2/nativeclient` o `http://localhost` como URI de redirección. Algunas bibliotecas de autenticación como MSAL.NET usan un valor predeterminado de `urn:ietf:wg:oauth:2.0:oob` cuando no se especifica ningún otro URI de redirección, lo que no se recomienda. Este valor predeterminado se actualizará como un cambio importante en la próxima versión principal.

- Si compila una aplicación nativa de Objective-C o Swift para macOS, registre el URI de redirección en función del identificador de agrupación de la aplicación, con el formato siguiente: `msauth.<your.app.bundle.id>://auth`. Reemplace `<your.app.bundle.id>` por el identificador de paquete de la aplicación.
- Si compila una aplicación Electron de Node.js, use un protocolo de archivo personalizado en lugar de un URI de redirección web normal (https://) para controlar el paso de redireccionamiento del flujo de autorización, por ejemplo `msal://redirect`. El nombre del protocolo de archivo personalizado no debe ser obvio de adivinar y debe seguir las sugerencias de la [especificación de OAuth 2.0 para aplicaciones nativas](https://tools.ietf.org/html/rfc8252#section-7.1).
- Si la aplicación solo utiliza la autenticación integrada de Windows o un nombre de usuario y una contraseña, no es necesario que registre ningún URI de redirección para la aplicación. Estos flujos realizan un recorrido de ida y vuelta al punto de conexión de la plataforma de identidad de Microsoft v2.0. No se volverá a llamar a la aplicación en ningún URI específico.
- Para distinguir el [flujo de código de dispositivo](scenario-desktop-acquire-token.md#device-code-flow), la [autenticación integrada de Windows](scenario-desktop-acquire-token.md#integrated-windows-authentication) y el [nombre de usuario y la contraseña](scenario-desktop-acquire-token.md#username-and-password) de una aplicación cliente confidencial mediante un flujo de credenciales de cliente usado en [aplicaciones de demonio](scenario-daemon-overview.md), ninguna de las cuales requiere un URI de redirección, configúrela como aplicación cliente pública. Para lograr esta configuración:

    1. En <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>, seleccione la aplicación en **Registros de aplicaciones** y, a continuación, seleccione **Autenticación**.
    1. En **Configuración avanzada** > **Permitir flujos de cliente público** > **Habilitar los flujos móviles y de escritorio siguientes:** , seleccione **Sí**.

        :::image type="content" source="media/scenarios/default-client-type.png" alt-text="Habilite la opción de cliente público en el panel Autenticación de Azure Portal":::

## <a name="api-permissions"></a>Permisos de API

Las aplicaciones de escritorio llaman a las API en nombre del usuario que tiene la sesión iniciada. Tienen que solicitar permisos delegados. No pueden solicitar permisos de la aplicación, que solo se controlan en las [aplicaciones de demonio](scenario-daemon-overview.md).

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo de este escenario, [Configuración del código de la aplicación](scenario-desktop-app-configuration.md).
