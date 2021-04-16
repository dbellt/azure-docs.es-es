---
title: Configuración del registro y el inicio de sesión con una cuenta de Salesforce
titleSuffix: Azure AD B2C
description: Permita suscribirse e iniciar sesión a los clientes con cuentas de Salesforce en las aplicaciones con Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/17/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 008d47fe7086322ea0bb7ef26a6c4b449f4269d2
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028832"
---
# <a name="set-up-sign-up-and-sign-in-with-a-salesforce-account-using-azure-active-directory-b2c"></a>Configuración de la suscripción y del inicio de sesión con una cuenta de Salesforce mediante Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]


## <a name="create-a-salesforce-application"></a>Creación de una aplicación de Salesforce

Para permitir el inicio de sesión de los usuarios con una cuenta de Salesforce en Azure Active Directory B2C (Azure AD B2C), debe crear una aplicación en el [administrador de aplicaciones](https://login.salesforce.com/) de Salesforce. Para obtener más información, consulte [Configurar ajustes de aplicaciones conectadas básicos](https://help.salesforce.com/articleView?id=connected_app_create_basics.htm) y [Activar configuración de OAuth para integración de API](https://help.salesforce.com/articleView?id=connected_app_create_api_integration.htm).

1. [Inicie sesión en Salesforce](https://login.salesforce.com/).
1. En el menú, seleccione **Setup** (Configuración).
1.  Expanda **Apps** (Aplicaciones) y, a continuación, seleccione **App Manager** (Administrador de aplicaciones).
1. Seleccione **New Connected App** (Nueva aplicación conectada).
1. En **Basic Information** (Información básica), escriba:
    1. **Connected App Name** (Nombre de la aplicación conectada): el nombre de la aplicación conectada se muestra en el administrador de aplicaciones y en su icono de App Launcher (Iniciador de aplicaciones). El nombre debe ser único dentro de una organización. 
    1. **API Name** (Nombre de la API) 
    1. **Contact Email** (Correo electrónico de contacto): el correo electrónico de contacto para Salesforce.
1. En **API (Enable OAuth Settings)** [API (Habilitar configuración de OAuth)], seleccione **Enable OAuth Settings** (Habilitar configuración de OAuth).
    1. Para la **URL de devolución de llamada**, escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Reemplace `your-tenant-name` por el nombre del inquilino, y `your-domain-name` por el dominio personalizado. Cuando especifique el nombre de inquilino, escriba todas las letras en minúscula, aunque se haya definido con letras en mayúscula en Azure AD B2C.
    1. En **Selected OAuth Scopes** (Ámbitos de OAuth seleccionados), seleccione **Access your basic information (id, profile, email, address, phone)** [Acceder a la información básica (identificador, perfil, correo electrónico, dirección y teléfono)] y **Allow access to your unique identifier (openid)** [Permitir el acceso al identificador único (OpenID)].
    1. Seleccione **Require Secret for Web Server Flow** (Requerir secreto para flujo de servidor web).
1. Seleccione **Configure ID Token** (Configurar token de identificador). 
    1. Establezca el valor **Token Valid for** (Token válido durante) en 5 minutos.
    1. Seleccione **Include Standard Claims** (Incluir notificaciones estándar).
1. Haga clic en **Guardar**.
1. Copie los valores de **Consumer Key** (Clave de consumidor) y **Consumer Secret** (Secreto de consumidor). Necesitará ambos para configurar Salesforce como proveedor de identidades de su inquilino. **Secreto del cliente** es una credencial de seguridad importante.

::: zone pivot="b2c-user-flow"

## <a name="configure-salesforce-as-an-identity-provider"></a>Configuración de Salesforce como proveedor de identidades

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Nuevo proveedor de OpenID Connect**.
1. Escriba un **nombre**. Por ejemplo, escriba *Salesforce*.
1. En **URL de metadatos**, escriba la dirección URL del [documento de configuración de Salesforce OpenID Connect](https://help.salesforce.com/articleView?id=remoteaccess_using_openid_discovery_endpoint.htm). En el caso de un espacio aislado, login.salesforce.com se reemplaza por test.salesforce.com. En el caso de una comunidad, login.salesforce.com se reemplaza por la dirección URL de la comunidad, como username.force.com/.well-known/openid-configuration. La dirección URL debe ser HTTPS.

    ```
    https://login.salesforce.com/.well-known/openid-configuration
    ```

1. En **Id. de cliente**, escriba el identificador de aplicación que ha anotado anteriormente.
1. En **Secreto de cliente**, escriba el secreto de cliente que ha anotado anteriormente.
1. En **Ámbito**, escriba el valor de `openid id profile email`.
1. Deje los valores predeterminados para **Tipo de respuesta** y **Modo de respuesta**.
1. (Opcional) En **Sugerencia de dominio**, escriba `contoso.com`. Para más información, consulte [Configuración de inicio de sesión directo con Azure Active Directory B2C](direct-signin.md#redirect-sign-in-to-a-social-provider).
1. En **Asignación de notificaciones del proveedor de identidades**, seleccione las siguientes notificaciones:

    - **Id. de usuario**: *sub*
    - **Nombre para mostrar**: *name*
    - **Nombre propio**: *given_name*
    - **Apellido**: *family_name*
    - **Correo electrónico**: *email*

1. Seleccione **Guardar**.

## <a name="add-salesforce-identity-provider-to-a-user-flow"></a>Adición de un proveedor de identidades de Salesforce a un flujo de usuario 

En este punto se ha configurado el proveedor de identidades de Salesforce, pero aún no está disponible en ninguna de las páginas de inicio de sesión. Para agregar un proveedor de identidades de Salesforce a un flujo de usuario:

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Haga clic en el flujo de usuario al que quiera agregar el proveedor de identidades de Salesforce.
1. En **Proveedores de identidades sociales**, seleccione **Salesforce**.
1. Seleccione **Guardar**.
1. Para probar la directiva, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *testapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar flujo de usuario**.
1. En la página de registro o de inicio de sesión, seleccione **Salesforce** para iniciar sesión con la cuenta de Salesforce.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar el secreto de cliente que haya registrado previamente en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripciones** del menú superior y elija el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. En la página de introducción, seleccione **Identity Experience Framework**.
5. Seleccione **Claves de directiva** y luego **Agregar**.
6. En **Opciones**, elija `Manual`.
7. Escriba un **nombre** para la clave de directiva. Por ejemplo, `SalesforceSecret`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
8. En **Secreto**, escriba el secreto de cliente que haya registrado previamente.
9. En **Uso de claves**, seleccione `Signature`.
10. Haga clic en **Crear**.

## <a name="configure-salesforce-as-an-identity-provider"></a>Configuración de Salesforce como proveedor de identidades

Si quiere que los usuarios inicien sesión con una cuenta de Salesforce, deberá definir la cuenta como un proveedor de notificaciones con el que Azure AD B2C pueda comunicarse mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Puede definir una cuenta de Salesforce como un proveedor de notificaciones; para ello, agréguela al elemento **ClaimsProvider** en el archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>salesforce.com</Domain>
      <DisplayName>Salesforce</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Salesforce-OpenIdConnect">
          <DisplayName>Salesforce</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="METADATA">https://login.salesforce.com/.well-known/openid-configuration</Item>
            <Item Key="response_types">code</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="scope">openid id profile email</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">0</Item>
            <!-- Update the Client ID below to the Application ID -->
            <Item Key="client_id">Your Salesforce application ID</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_SalesforceSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="salesforce.com" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
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

4. **METADATA** se establece en la dirección URL del [documento de configuración de Salesforce OpenID Connect](https://help.salesforce.com/articleView?id=remoteaccess_using_openid_discovery_endpoint.htm). En el caso de un espacio aislado, login.salesforce.com se reemplaza por test.salesforce.com. En el caso de una comunidad, login.salesforce.com se reemplaza por la dirección URL de la comunidad, como username.force.com/.well-known/openid-configuration. La dirección URL debe ser HTTPS.
5. Establezca **client_id** en el identificador de la aplicación desde el registro de aplicación.
6. Guarde el archivo.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="SalesforceExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="SalesforceExchange" TechnicalProfileReferenceId="Salesforce-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **Salesforce** para iniciar sesión con la cuenta de Salesforce.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.


::: zone-end

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [pasar el token de Salesforce a la aplicación](idp-pass-through-user-flow.md).
