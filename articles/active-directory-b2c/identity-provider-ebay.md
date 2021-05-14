---
title: Configuración del registro y del inicio de sesión con una cuenta de eBay
titleSuffix: Azure AD B2C
description: Permita suscribirse e iniciar sesión a los clientes con cuentas de eBay en las aplicaciones con Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/29/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: e2be6fd7ee6d42fe22e4735420ff2b16fa9974ba
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108285329"
---
# <a name="set-up-sign-up-and-sign-in-with-an-ebay-account-using-azure-active-directory-b2c"></a>Configuración de la suscripción y del inicio de sesión con una cuenta de eBay mediante Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="create-an-ebay-application"></a>Crear una aplicación de eBay

Para habilitar el inicio de sesión para los usuarios con una cuenta de eBay en Azure Active Directory B2C (Azure AD B2C), tiene que crear una aplicación en la [consola para desarrolladores de eBay](https://developer.ebay.com). Para obtener más información, consulte [Creación de una cuenta de desarrollador](https://developer.ebay.com/api-docs/static/creating-edp-account.html). Si aún no tiene una cuenta de desarrollador de eBay, puede suscribirse en [https://developer.ebay.com/signin](https://developer.ebay.com/signin?tab=register).

Para crear una aplicación de eBay, siga estos pasos:

1. Inicie sesión en [Application Keys](https://developer.ebay.com/my/keys) de la consola para desarrolladores de eBay con las credenciales de la cuenta de desarrollador de eBay.
1. Escriba el título de la aplicación en **Application Title**.
1. En **Production** (Producción), seleccione **Create a keyset** (Crear un conjunto de claves). 
1. En la página **Confirm the Primary Contact for this Account** (Confirmar el contacto principal de esta cuenta), indique los detalles de la cuenta. Para completar el proceso de registro, seleccione **Continue to Create Keys** (Continuar hasta Crear claves).
1. Copie los valores de **App ID (Client ID)** (Id. de aplicación [id. de cliente]) y **App ID (Client ID)** (Id. de aplicación [id. de cliente]). Necesitará los dos para agregar el proveedor de identidades a su inquilino.
1. Seleccione **User Tokens** (Tokens de usuario) y, a continuación, seleccione **Get a Token from eBay via Your Application** (Obtener un token de eBay a través de la aplicación).
1. Seleccione **Add eBay Redirect URL** (Agregar dirección URL de redireccionamiento de eBay).
    1. Escriba una dirección URL válida para **Your privacy policy URL** (URL de la directiva de privacidad), por ejemplo: `https://www.contoso.com/privacy`. La dirección URL de directiva es una página que sirve para proporcionar información de privacidad de la aplicación.
    1. En **Your auth accepted URL** (URL de autenticación aceptada), escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Reemplace `your-tenant-name` por el nombre del inquilino, y `your-domain-name` por el dominio personalizado.
1. Seleccione **Guardar**.

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar el secreto de cliente que haya registrado previamente en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripciones** del menú superior y elija el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. En la página de introducción, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Manual`.
1. Escriba un **nombre** para la clave de directiva. Por ejemplo, `eBaySecret`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
1. En **Secreto**, escriba el secreto de cliente que haya registrado previamente.
1. En **Uso de claves**, seleccione `Signature`.
1. Seleccione **Crear**.

## <a name="configure-ebay-as-an-identity-provider"></a>Configuración de eBay como proveedor de identidades

Para permitir que los usuarios inicien sesión con una cuenta de eBay, debe definir la cuenta como proveedor de notificaciones con el que Azure AD B2C pueda comunicarse a través de un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Puede definir una cuenta de eBay como proveedor de notificaciones; para ello, agréguela al elemento **ClaimsProvider** en el archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <!--
    <ClaimsProviders> -->
      <ClaimsProvider>
        <Domain>ebay.com</Domain>
        <DisplayName>eBay</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="eBay-OAUTH2">
            <DisplayName>eBay</DisplayName>
            <Protocol Name="OAuth2" />
            <Metadata>
              <Item Key="ProviderName">ebay.com</Item>
              <Item Key="authorization_endpoint">https://auth.ebay.com/oauth2/authorize</Item>
              <Item Key="AccessTokenEndpoint">https://api.ebay.com/identity/v1/oauth2/token</Item>
              <Item Key="ClaimsEndpoint">https://apiz.ebay.com/commerce/identity/v1/user/</Item>
              <Item Key="HttpBinding">POST</Item>
              <Item Key="BearerTokenTransmissionMethod">AuthorizationHeader</Item>
              <Item Key="token_endpoint_auth_method">client_secret_basic</Item>
              <Item Key="scope">https://api.ebay.com/oauth/api_scope/commerce.identity.readonly</Item>
              <Item Key="UsePolicyInRedirectUri">0</Item>
              <!-- Update the Client ID below to the Application ID -->
              <Item Key="client_id">Your eBay app ID</Item>
            </Metadata>
            <CryptographicKeys>
              <Key Id="client_secret" StorageReferenceId="eBaySecret"/>
            </CryptographicKeys>
            <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="userId"/>
              <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="individualAccount.firstName"/>
              <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="individualAccount.lastName"/>
              <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="username"/>
              <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
              <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="ebay.com" AlwaysUseDefaultValue="true" />
              <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            </OutputClaims>
            <OutputClaimsTransformations>
              <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
              <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
              <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            </OutputClaimsTransformations>
            <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
          </TechnicalProfile>
        </TechnicalProfiles>
      </ClaimsProvider>
    <!--
    </ClaimsProviders> -->
    ```

4. Establezca **client_id** en el identificador de la aplicación desde el registro de aplicación.
5. Guarde el archivo.


[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="eBayExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="eBayExchange" TechnicalProfileReferenceId="eBay-OAUTH2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **eBay** para iniciar sesión con la cuenta de eBay.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [pasar el token de Facebook a la aplicación](idp-pass-through-user-flow.md).

::: zone-end
