---
title: Configuración del registro y del inicio de sesión con una cuenta de Twitter
titleSuffix: Azure AD B2C
description: Permita suscribirse e iniciar sesión a los clientes con cuentas de Twitter en las aplicaciones con Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/06/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 97a8134e858112d7e1deff6744b5555c172692f2
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028186"
---
# <a name="set-up-sign-up-and-sign-in-with-a-twitter-account-using-azure-active-directory-b2c"></a>Configuración de la suscripción y del inicio de sesión con una cuenta de Twitter mediante Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]
::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-an-application"></a>Crear una aplicación

Para habilitar el inicio de sesión de los usuarios con una cuenta de Twitter en Azure AD B2C, debe crear una aplicación de Twitter. Si aún no tiene una cuenta de Twitter, puede suscribirse en [https://twitter.com/signup](https://twitter.com/signup). También debe [solicitar una cuenta de desarrollador](https://developer.twitter.com/en/apply/user.html). Para obtener más información, consulte [Solicitud de acceso](https://developer.twitter.com/en/apply-for-access).

1. Inicie sesión en el [portal para desarrolladores de Twitter](https://developer.twitter.com/portal/projects-and-apps) con las credenciales de la cuenta de Twitter.
1. En **Standalone Apps** (aplicaciones independientes), seleccione **+Create App** (+ crear aplicación).
1. Escriba un **nombre de aplicación** y, a continuación, seleccione **Complete** (completo).
1. Copie el valor de la **clave de la aplicación** y **secreto de clave de API**.  Use los dos para configurar Twitter como proveedor de identidades de su inquilino. 
1. En **Setup your App** (preparar la aplicación), seleccione **App settings** (configuración de la aplicación).
1. En **Authentication settings** (configuración de autenticación), seleccione **Edit** (editar).
    1. Active la casilla **Enable 3-legged OAuth** (habilitar OAuth de 3 fases).
    1. Active la casilla **Request email address from users** (solicitar dirección de correo electrónico de los usuarios).
    1. En **Callback URLs** (URL de devolución de llamada), escriba `https://your-tenant.b2clogin.com/your-tenant-name.onmicrosoft.com/your-user-flow-Id/oauth1/authresp`.  Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name/your-tenant-name.onmicrosoft.com/your-user-flow-Id/oauth1/authresp`. Cuando especifique el nombre de inquilino y el identificador de flujo de usuario, use solo letras en minúscula, aunque se hayan definido en mayúscula en Azure AD B2C. Sustituya:
        - `your-tenant-name` por el nombre del inquilino.
        - `your-domain-name` por el dominio personalizado.
        - `your-user-flow-Id` por el identificador de su flujo de usuario. Por ejemplo, `b2c_1a_signup_signin_twitter`. 
    
    1. En **Website URL** (dirección URL del sitio web), escriba `https://your-tenant.b2clogin.com`. Reemplace `your-tenant` por el nombre del inquilino. Por ejemplo, `https://contosob2c.b2clogin.com`. Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name`.
    1. Escriba una dirección URL para **Terms of service** (términos del servicio), por ejemplo `http://www.contoso.com/tos`. La dirección URL de la directiva es una página que se mantiene para proporcionar los términos y condiciones de la aplicación.
    1. Escriba una dirección URL en **Privacy Policy** (directiva de privacidad), por ejemplo `http://www.contoso.com/privacy`. La dirección URL de directiva es una página que sirve para proporcionar información de privacidad de la aplicación.
    1. Seleccione **Guardar**.

::: zone pivot="b2c-user-flow"

## <a name="configure-twitter-as-an-identity-provider"></a>Configuración de Twitter como proveedor de identidades

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Twitter**.
1. Escriba un **nombre**. Por ejemplo, *Twitter*.
1. En **Id. de cliente**, escriba la *clave de API* de la aplicación de Twitter que creó anteriormente.
1. En el **secreto de cliente**, escriba el *secreto de clave de API* que anotó.
1. Seleccione **Guardar**.

## <a name="add-twitter-identity-provider-to-a-user-flow"></a>Adición del proveedor de identidades de Twitter a un flujo de usuario 

En este momento, el proveedor de identidades de Twitter ya se ha configurado, pero no está disponible en ninguna de las pantallas de inicio de sesión. Adición del proveedor de identidades de Twitter a un flujo de usuario:

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Seleccione el flujo de usuario al que quiera agregar el proveedor de identidades de Twitter.
1. En **Proveedores de identidades sociales**, seleccione **Twitter**.
1. Seleccione **Guardar**.
1. Para probar la directiva, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *testapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar flujo de usuario**.
1. En la página de registro o de inicio de sesión, seleccione **Twitter** para iniciar sesión con la cuenta de Twitter.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar la clave del secreto que haya registrado previamente en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripciones** del menú superior y elija el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. En la página de introducción, seleccione **Identity Experience Framework**.
5. Seleccione **Claves de directiva** y luego **Agregar**.
6. En **Opciones**, elija `Manual`.
7. Escriba un **nombre** para la clave de directiva. Por ejemplo, `TwitterSecret`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
8. En **Secreto**, escriba el secreto de cliente que haya registrado previamente.
9. En **Uso de claves**, seleccione `Encryption`.
10. Haga clic en **Crear**.

## <a name="configure-twitter-as-an-identity-provider"></a>Configuración de Twitter como proveedor de identidades

Para habilitar que los usuarios inicien sesión con una cuenta de Twitter, deberá definirla como un proveedor de notificaciones con el que Azure AD B2C pueda comunicarse mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Puede definir una cuenta de Twitter como proveedor de notificaciones; para ello, agréguela al elemento **ClaimsProvider** en el archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>twitter.com</Domain>
      <DisplayName>Twitter</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Twitter-OAuth1">
          <DisplayName>Twitter</DisplayName>
          <Protocol Name="OAuth1" />
          <Metadata>
            <Item Key="ProviderName">Twitter</Item>
            <Item Key="authorization_endpoint">https://api.twitter.com/oauth/authenticate</Item>
            <Item Key="access_token_endpoint">https://api.twitter.com/oauth/access_token</Item>
            <Item Key="request_token_endpoint">https://api.twitter.com/oauth/request_token</Item>
            <Item Key="ClaimsEndpoint">https://api.twitter.com/1.1/account/verify_credentials.json?include_email=true</Item>
            <Item Key="ClaimsResponseFormat">json</Item>
            <Item Key="client_id">Your Twitter application API key</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_TwitterSecret" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="user_id" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="screen_name" />
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="twitter.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Reemplace el valor de **client_id** por el *secreto de clave de API* que anotó anteriormente.
5. Guarde el archivo.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="TwitterExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="TwitterExchange" TechnicalProfileReferenceId="Twitter-OAuth1" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **Twitter** para iniciar sesión con la cuenta de Twitter.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end
