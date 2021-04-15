---
title: Inserción de la interfaz de usuario de Azure Active Directory B2C en la aplicación con una directiva personalizada
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo insertar la interfaz de usuario de Azure Active Directory B2C en la aplicación con una directiva personalizada.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/21/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: dc4450fb1e21211b43bc178d94cf4bdfe6da58e1
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256793"
---
# <a name="embedded-sign-in-experience"></a>Experiencia de inicio de sesión insertada

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

Para obtener una experiencia de inicio de sesión más sencilla, puede evitar el redireccionamiento de los usuarios a una página de inicio de sesión aparte o la generación de una ventana emergente. Mediante el uso del elemento de marco insertado `<iframe>`, puede insertar la interfaz de usuario de inicio de sesión de Azure AD B2C directamente en la aplicación web.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="web-application-embedded-sign-in"></a>Inicio de sesión insertado en la aplicación web

El elemento de marco insertado `<iframe>` se usa para insertar un documento en una página web de HTML5. Puede usar el elemento iframe para insertar la interfaz de usuario de inicio de sesión de Azure AD B2C directamente en la aplicación web, tal como se muestra en el ejemplo siguiente:

![Inicio de sesión con el desplazamiento de la experiencia DIV](media/embedded-login/login-hovering.png)

Al usar el iframe, tenga en cuenta lo siguiente:

- El inicio de sesión insertado solo admite cuentas locales. La mayoría de los proveedores de identidades sociales (por ejemplo, Google y Facebook) impiden que las páginas de inicio de sesión se representen en marcos insertados.
- Dado que las cookies de sesión de Azure AD B2C dentro de un iframe se consideran cookies de terceros, algunos exploradores (por ejemplo, Safari o Chrome en modo incógnito) bloquean o borran estas cookies, lo que da lugar a una experiencia de usuario no deseada. Para evitar este problema, asegúrese de que el nombre de dominio de la aplicación y el dominio de Azure AD B2C tengan el *mismo origen*. Para usar el mismo origen, [habilite dominios personalizados](custom-domain.md) para el inquilino Azure AD B2C y, a continuación, configure la aplicación web con el mismo origen. Por ejemplo, una aplicación hospedada en "https://app.contoso.com" tiene el mismo origen que la instancia de Azure AD B2C que se ejecuta en "https://login.contoso.com".

## <a name="prerequisites"></a>Prerrequisitos

* Siga los pasos de [Introducción a las directivas personalizadas en Active Directory B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy).
* [Habilite los dominios personalizados](custom-domain.md) para sus directivas.

## <a name="configure-your-policy"></a>Configuración de la directiva

Para permitir que la interfaz de usuario de Azure AD B2C se pueda insertar en un iframe, debe incluirse una directiva de seguridad de contenido `Content-Security-Policy` y opciones de marco `X-Frame-Options` en los encabezados de respuesta HTTP de Azure AD B2C. Estos encabezados permiten que la interfaz de usuario de Azure AD B2C se ejecute en el nombre de dominio de la aplicación.

Agregue un elemento **JourneyFraming** dentro del elemento [RelyingParty](relyingparty.md).  El elemento **UserJourneyBehaviors** debe aparecer después de **DefaultUserJourney**. El elemento **UserJourneyBehavors** debe ser como este:

```xml
<!--
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" /> -->
  <UserJourneyBehaviors> 
    <JourneyFraming Enabled="true" Sources="https://somesite.com https://anothersite.com" /> 
  </UserJourneyBehaviors>
<!--
</RelyingParty> -->
```

El atributo **Sources** contiene el URI de la aplicación web. Agregue un espacio entre los URI. Cada URI debe cumplir los siguientes requisitos:

- El URI debe ser de confianza y pertenecer a la aplicación.
- El URI debe usar el esquema https.  
- Se debe especificar el URI completo de la aplicación web. No se admiten caracteres comodín.

Además, recomendamos que también bloquee su propio nombre de dominio para que no se incluya en un iframe mediante el establecimiento de los encabezados Content-Security-Policy y X-Frame-Option, respectivamente, en las páginas de la aplicación. Esto mitigará los problemas de seguridad de los exploradores más antiguos relacionados con la inserción anidada de iframes.

## <a name="adjust-policy-user-interface"></a>Ajuste de la interfaz de usuario de directiva

Con la [personalización de la interfaz de usuario](customize-ui.md) de Azure AD B2C, tiene un control casi completo sobre el contenido HTML y CSS que se presenta a los usuarios. Siga los pasos para personalizar una página HTML mediante definiciones de contenido. Para ajustar la interfaz de usuario de Azure AD B2C en el tamaño de iframe, proporcione una página HTML limpia sin espacios en segundo plano ni adicionales.  

El siguiente código CSS oculta los elementos HTML de Azure AD B2C y ajusta el tamaño del panel para llenar el iframe.

```css
div.social, div.divider {
    display: none;
}

div.api_container{
    padding-top:0;
}

.panel {
    width: 100%!important
}
```

En algunos casos, puede que desee informar a la aplicación de la página de Azure AD B2C que se está presentando actualmente. Por ejemplo, cuando un usuario selecciona la opción de registro, es posible que desee que la aplicación responda ocultando los vínculos para iniciar sesión con una cuenta de red social o ajustando el tamaño del iframe.

Para informar a su aplicación de la página actual de Azure AD B2C, [habilite su directiva para JavaScript](./javascript-and-page-layout.md) y luego utilice mensajes post de HTML5. El siguiente código JavaScript envía un mensaje post a la aplicación con `signUp`:

```javascript
window.parent.postMessage("signUp", '*');
```

## <a name="configure-a-web-application"></a>Configuración de una aplicación web

Cuando un usuario selecciona el botón de inicio de sesión, la [aplicación web](code-samples.md#web-apps-and-apis) genera una solicitud de autorización que lleva al usuario a la experiencia de inicio de sesión de Azure AD B2C. Una vez completado el inicio de sesión, Azure AD B2C devuelve un token de identificador, o código de autorización, al URI de redirección configurado dentro de la aplicación.

Para admitir el inicio de sesión insertado, la propiedad **src** de iframe apunta al controlador de inicio de sesión, como `/account/SignUpSignIn`, que genera la solicitud de autorización y redirige al usuario a la directiva de Azure AD B2C.

```html
<iframe id="loginframe" frameborder="0" src="/account/SignUpSignIn"></iframe>
``` 

Una vez que la aplicación recibe y valida el token de identificador, el flujo de autorización se completa y la aplicación reconoce al usuario y confía en él. Dado que el flujo de autorización se produce dentro del iframe, debe volver a cargar la página principal. Una vez que se vuelve a cargar la página, el botón de inicio de sesión cambia a "cerrar sesión" y el nombre de usuario se presenta en la interfaz de usuario.  

El siguiente es un ejemplo que muestra cómo el URI de redireccionamiento de inicio de sesión puede actualizar la página principal:

```javascript
window.top.location.reload();
```

### <a name="add-sign-in-with-social-accounts-to-a-web-app"></a>Adición de inicio de sesión con cuentas de redes sociales a una aplicación web

Los proveedores de identidades sociales impiden que sus páginas de inicio de sesión se representen en marcos insertados. Puede usar una directiva independiente para las cuentas de redes sociales, o puede usar una sola directiva para el inicio de sesión y el registro con cuentas locales y sociales. Después, puede usar el parámetro de cadena de consulta `domain_hint`. El parámetro de sugerencia de dominio lleva al usuario directamente a la página de inicio de sesión del proveedor de identidad social.

En su aplicación, agregue los botones de inicio de sesión con cuenta de red social. Cuando un usuario hace clic en uno de los botones de la cuenta de red social, el control debe cambiar el nombre de la directiva o establecer el parámetro de la sugerencia de dominio.

<!-- TBD: add a diagram -->

El URI de redireccionamiento puede ser el mismo URI de redireccionamiento usado por el iframe. Puede omitir la recarga de la página.

## <a name="configure-a-single-page-application"></a>Configuración de una aplicación de página única

En el caso de una aplicación de página única, también necesitará una segunda página HTML de "inicio de sesión" que se cargue en el iframe. Esta página de inicio de sesión hospeda el código de la biblioteca de autenticación que genera el código de autorización y devuelve el token.

Cuando la aplicación de página única necesite el token de acceso, use código JavaScript para obtener el token de acceso del iframe y el objeto que lo contiene.

> [!NOTE]
> Actualmente no se admite la ejecución de MSAL 2.0 en un iframe.

El código siguiente es un ejemplo que se ejecuta en la página principal y llama al código JavaScript de un iframe:

```javascript
function getToken()
{
  var token = document.getElementById("loginframe").contentWindow.getToken("adB2CSignInSignUp");

  if (token === "LoginIsRequired")
    document.getElementById("tokenTextarea").value = "Please login!!!"
  else
    document.getElementById("tokenTextarea").value = token.access_token;
}

function logOut()
{
  document.getElementById("loginframe").contentWindow.policyLogout("adB2CSignInSignUp", "B2C_1A_SignUpOrSignIn");
}
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los artículos relacionados siguientes:

- [Personalización de la interfaz de usuario](customize-ui.md)
- Referencia del elemento [RelyingParty](relyingparty.md)
- [Habilitación de la directiva para JavaScript](./javascript-and-page-layout.md)
- [Ejemplos de código](code-samples.md)

::: zone-end
