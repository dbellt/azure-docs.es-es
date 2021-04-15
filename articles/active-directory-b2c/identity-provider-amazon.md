---
title: Configuración del registro y del inicio de sesión con una cuenta de Amazon
titleSuffix: Azure AD B2C
description: Permita suscribirse e iniciar sesión a los clientes con cuentas de Amazon en las aplicaciones con Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.custom: project-no-code
ms.date: 03/17/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: b6c0d9d5430d84006b208c50e78b8d875c95b8ac
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028390"
---
# <a name="set-up-sign-up-and-sign-in-with-an-amazon-account-using-azure-active-directory-b2c"></a>Configuración de la suscripción y del inicio de sesión con una cuenta de Amazon mediante Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-an-app-in-the-amazon-developer-console"></a>Creación de una aplicación en la consola de desarrolladores de Amazon

Para habilitar el inicio de sesión para los usuarios con una cuenta de Amazon en Azure Active Directory B2C (Azure AD B2C), tiene que crear una aplicación en [Servicios y tecnologías para desarrolladores de Amazon](https://developer.amazon.com). Para obtener más información, vea [Registro para iniciar sesión con Amazon](https://developer.amazon.com/docs/login-with-amazon/register-web.html). Si aún no tiene una cuenta de Amazon, puede suscribirse en [https://www.amazon.com/](https://www.amazon.com/).

1. Inicie sesión en [Amazon Developer Console](https://developer.amazon.com/dashboard) con las credenciales de su cuenta de Amazon.
1. Si aún no lo ha hecho, seleccione **Sign Up** (Registrarse), siga los pasos de registro para desarrolladores y, a continuación, acepte la directiva.
1. En el panel, seleccione **Login with Amazon** (Iniciar sesión con Amazon).
1. Seleccione **Create a New Security Profile** (Crear perfil de seguridad).
1. Escriba la información de los campos **Security Profile Name** (Nombre del perfil de seguridad), **Security Profile Description** (Descripción del perfil de seguridad) y **Consent Privacy Notice URL** (URL del aviso de privacidad de consentimiento); por ejemplo, `https://www.contoso.com/privacy`. La dirección URL del aviso de privacidad es una página que usted administra y que proporciona información de privacidad a los usuarios. A continuación, haga clic en **Save**(Guardar).
1. En la sección **Login with Amazon Configurations** (Configuración del inicio de sesión con Amazon), seleccione la opción **Security Profile Name** (Nombre del perfil de seguridad) que estableció, seleccione el icono **Manage** (Administrar) y, a continuación, **Web Settings** (Configuración web).
1. En la sección **Web Settings** (Configuración web), copie los valores de **Client ID** (Identificador de cliente). Seleccione **Show secret** (Mostrar secreto) para obtener el secreto de cliente y, a continuación, cópielo. Necesitará ambos valores para configurar la cuenta de Amazon como proveedor de identidades de su inquilino. **secreto de cliente** es una credencial de seguridad importante.
1. En la sección **Web Settings** (Configuración web), seleccione **Edit** (Editar). 
    1. En **Orígenes permitidos**, escriba `https://your-tenant-name.b2clogin.com`. Reemplace `your-tenant-name` por el nombre del inquilino. Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name`.
    1.  En **Allowed Return URLs** (Direcciones URL de retorno permitidas), escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`.  Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp`.  Reemplace `your-tenant-name` por el nombre del inquilino, y `your-domain-name` por el dominio personalizado.
1. Seleccione **Guardar**.

::: zone pivot="b2c-user-flow"

## <a name="configure-amazon-as-an-identity-provider"></a>Configuración de Amazon como proveedor de identidades

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Amazon**.
1. Escriba un **nombre**. Por ejemplo, *Amazon*.
1. En **Id. de cliente**, escriba el identificador de cliente de Amazon que ha creado anteriormente.
1. En **Secreto de cliente**, escriba el secreto de cliente que ha anotado.
1. Seleccione **Guardar**.

## <a name="add-amazon-identity-provider-to-a-user-flow"></a>Adición del proveedor de identidades de Amazon a un flujo de usuario 

En este momento, el proveedor de identidades de Amazon ya se ha configurado, pero no está disponible en ninguna de las pantallas de inicio de sesión. Para agregar el proveedor de identidades de Amazon a un flujo de usuario

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Haga clic en el flujo de usuario al que quiere agregar el proveedor de identidades de Amazon.
1. En **Proveedores de identidades sociales**, seleccione **Amazon**.
1. Seleccione **Guardar**.
1. Para probar la directiva, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *testapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar flujo de usuario**.
1. En la página de registro o de inicio de sesión, seleccione **Amazon** para iniciar sesión con la cuenta de Amazon.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar el secreto de cliente que haya registrado previamente en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. En la página de introducción, seleccione **Identity Experience Framework**.
5. Seleccione **Claves de directiva** y luego **Agregar**.
6. En **Opciones**, elija `Manual`.
7. Escriba un **nombre** para la clave de directiva. Por ejemplo, `AmazonSecret`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
8. En **Secreto**, escriba el secreto de cliente que haya registrado previamente.
9. En **Uso de claves**, seleccione `Signature`.
10. Haga clic en **Crear**.

## <a name="configure-amazon-as-an-identity-provider"></a>Configuración de Amazon como proveedor de identidades

Para permitir que los usuarios inicien sesión mediante una cuenta de Amazon, debe definir la cuenta como proveedor de notificaciones. con el que Azure AD B2C pueda comunicarse por medio de un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Puede definir una cuenta de Amazon como proveedor de notificaciones; para ello, agréguela al elemento **ClaimsProviders** en el archivo de extensión de la directiva.


1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>amazon.com</Domain>
      <DisplayName>Amazon</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Amazon-OAuth2">
        <DisplayName>Amazon</DisplayName>
        <Protocol Name="OAuth2" />
        <Metadata>
          <Item Key="ProviderName">amazon</Item>
          <Item Key="authorization_endpoint">https://www.amazon.com/ap/oa</Item>
          <Item Key="AccessTokenEndpoint">https://api.amazon.com/auth/o2/token</Item>
          <Item Key="ClaimsEndpoint">https://api.amazon.com/user/profile</Item>
          <Item Key="scope">profile</Item>
          <Item Key="HttpBinding">POST</Item>
          <Item Key="UsePolicyInRedirectUri">false</Item>
          <Item Key="client_id">Your Amazon application client ID</Item>
        </Metadata>
        <CryptographicKeys>
          <Key Id="client_secret" StorageReferenceId="B2C_1A_AmazonSecret" />
        </CryptographicKeys>
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="user_id" />
          <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
          <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
          <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="amazon.com" />
          <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
        </OutputClaims>
          <OutputClaimsTransformations>
          <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
          <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
          <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
        </OutputClaimsTransformations>
        <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Establezca **client_id** en el identificador de la aplicación desde el registro de aplicación.
5. Guarde el archivo.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="AmazonExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="AmazonExchange" TechnicalProfileReferenceId="Amazon-OAuth2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **Amazon** para iniciar sesión con la cuenta de Amazon.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end
