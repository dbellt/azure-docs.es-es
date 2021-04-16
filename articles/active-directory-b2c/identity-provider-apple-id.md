---
title: Configuración del registro y del inicio de sesión con un id. de Apple
titleSuffix: Azure AD B2C
description: Permita suscribirse e iniciar sesión a los clientes con el id. de Apple en las aplicaciones con Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/22/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: f5e4ff93b90c7644c1d1498a14c8b9954041cb34
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028373"
---
# <a name="set-up-sign-up-and-sign-in-with-an-apple-id--using-azure-active-directory-b2c-preview"></a>Configuración de la suscripción y del inicio de sesión con un id. de Apple mediante Azure Active Directory B2C (versión preliminar)

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-an-apple-id-application"></a>Creación de una aplicación de id. de Apple

Para habilitar el inicio de sesión para los usuarios con un id. de Apple en Azure Active Directory B2C (Azure AD B2C), tiene que crear una aplicación en [https://developer.apple.com](https://developer.apple.com). Para más información, vea [Inicio de sesión con Apple](https://developer.apple.com/sign-in-with-apple/get-started/). Si aún no tiene una cuenta de desarrollador de Apple, puede registrarse en el [Programa Apple Developer](https://developer.apple.com/programs/enroll/).

1. Inicie sesión en el [portal de Apple Developer](https://developer.apple.com/account) con sus credenciales de la cuenta.
1. En el menú, seleccione **Certificados, identificadores y perfiles** y, a continuación, seleccione **(+)** .
1. En **Register a New Identifier** (Registrar un nuevo identificador), seleccione **Identificadores de aplicaciones** y, a continuación, seleccione **Continuar**.
1. En **Seleccionar un tipo**, seleccione **Aplicación** y, a continuación, seleccione **Continuar**.
1. Para **Registrar un identificador de aplicación**:
    1. Escriba una **Descripción**. 
    1. Escriba el **Id. de agrupación**, como `com.contoso.azure-ad-b2c`. 
    1. En **Funcionalidades**, seleccione **Iniciar sesión con Apple** en la lista de funcionalidades. 
    1. Tome nota de su prefijo de identificador de aplicación (id. de equipo) en este paso. Lo necesitará más adelante.
    1. Seleccione **Continue** (Continuar) y **Register** (Registrar).
1. En el menú, seleccione **Certificados, identificadores y perfiles** y, a continuación, seleccione **(+)** .
1. En **Register a New Identifier** (Registrar un nuevo identificador), seleccione **Services IDs** (Identificadores de servicio) y, a continuación, seleccione **Continuar**.
1. Para **Registrar un identificador de servicios**:
    1. Escriba una **Descripción**. La descripción se mostrará al usuario en la pantalla de consentimiento.
    1. Escriba el **Identificador**, como `com.consoto.azure-ad-b2c-service`. El identificador es el identificador de cliente para el flujo de OpenID Connect.
    1. Seleccione **Continuar** y **Registrar**.
1. En **Identificadores**, seleccione el identificador que ha creado.
1. Seleccione **Iniciar sesión con Apple** y, a continuación, seleccione **Configurar**.
    1. Seleccione **Primary App ID** (Identificador de aplicación principal) con el que quiere configurar el inicio de sesión con Apple.
    1. En **Domains and Subdomains** (Dominios y subdominios), escriba `your-tenant-name.b2clogin.com`. Especifique el nombre del inquilino en el campo de nombre. Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name`.
    1. En **Return URLs** (Direcciones URL de retorno), escriba `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Si usa un [dominio personalizado](custom-domain.md), escriba `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp`. Reemplace `your-tenant-name` por el nombre del inquilino, y `your-domain-name` por el dominio personalizado.
    1. Seleccione **Siguiente** y después **Listo**.
    1. Cuando se cierre la ventana emergente, seleccione **Continuar** y, a continuación, seleccione **Guardar**.

## <a name="creating-an-apple-client-secret"></a>Creación de un secreto de cliente de Apple

1. En el menú del portal de Apple Developer, seleccione **Claves** y, a continuación, seleccione **(+)** .
1. Para **Registrar una nueva clave**:
    1. Escriba un **Nombre de clave**.
    1. Seleccione **Iniciar sesión con Apple** y, a continuación, seleccione **Configurar**.
    1. Para **Primary App ID** (Identificador de aplicación principal), seleccione la aplicación que creó anteriormente y seleccione **Guardar**.
    1. Seleccione **Configurar** y, a continuación, seleccione **Registrar** para finalizar el proceso de registro de claves.
1. Para **Download Your Key** (Descargar su clave), seleccione **Descargar** para descargar un archivo .p8 que contenga la clave.


::: zone pivot="b2c-user-flow"

## <a name="configure-apple-as-an-identity-provider"></a>Configuración de Apple como proveedor de identidades

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como un administrador global del inquilino de Azure AD B2C.
1. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.
1. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. Seleccione **Proveedores de identidades** y luego **Apple (versión preliminar)** .
1. Escriba un **nombre**. Por ejemplo, *Apple*.
1. Escriba el **Apple developer ID (Team ID)** [Id. de desarrollador de Apple (id. de equipo)].
1. Escriba el **Apple service ID (client ID)** [Id. de servicio de Apple (id. de cliente)].
1. Escriba el **Apple key ID** (Id. de clave de Apple).
1. Seleccione y cargue los **Apple certificate data** (Datos del certificado de Apple).
1. Seleccione **Guardar**.


> [!IMPORTANT] 
> - El inicio de sesión con Apple requiere que el administrador renueve el secreto de cliente cada 6 meses. 
> - Durante la versión preliminar pública de esta característica, deberá renovar manualmente el secreto de cliente de Apple si expira. Se mostrará una advertencia de antemano en la página Configuración de IDP social de los proveedores de identidades de Apple, pero le recomendamos que establezca su propio recordatorio. 
> - Si necesita renovar el secreto, abra Azure AD B2C en Azure Portal, vaya a **Proveedores de identidades** > **Apple** y seleccione **Renew secret** (Renovar secreto).

## <a name="add-the-apple-identity-provider-to-a-user-flow"></a>Adición del proveedor de identidades de Apple a un flujo de usuario

Para permitir que los usuarios inicien sesión con un identificador de Apple, debe agregar el proveedor de identidades de Apple a un flujo de usuario. El inicio de sesión con Apple solo puede configurarse para la versión **recomendada** de flujos de usuario. Para agregar el proveedor de identidades de Apple a un flujo de usuario:

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Seleccione un flujo de usuario para el que desee agregar el proveedor de identidades de Apple. 
1. En **Proveedores de identidades sociales**, seleccione **Apple (versión preliminar)** .
1. Seleccione **Guardar**.
1. Para probar la directiva, seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *testapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar flujo de usuario**.
1. En la página de registro o de inicio de sesión, seleccione **Apple** para iniciar sesión con el identificador de Apple.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="signing-the-client-secret"></a>Firma del secreto de cliente

Use el archivo .p8 que descargó anteriormente para firmar el secreto de cliente en un token JWT. Hay muchas bibliotecas que pueden crear y firmar el JWT por usted. Use la [función de Azure que crea un token](https://github.com/azure-ad-b2c/samples/tree/master/policies/sign-in-with-apple/azure-function) automáticamente. 

1. Cree una [Función de Azure](../azure-functions/functions-create-function-app-portal.md).
1. En **Desarrollador**, seleccione **Código y prueba**. 
1. Copie el contenido del archivo [run.csx](https://github.com/azure-ad-b2c/samples/blob/master/policies/sign-in-with-apple/azure-function/run.csx) y péguelo en el editor.
1. Seleccione **Guardar**.
1. Realice una solicitud HTTP `POST` y proporcione la siguiente información:

    - **appleTeamId**: identificador del equipo de Apple Developer.
    - **appleServiceId**: el identificador de servicio de Apple (también el identificador de cliente).
    - **p8key**: la clave de formato PEM. Para obtenerla, abra el archivo .p8 en un editor de texto y copie todo lo comprendido entre `-----BEGIN PRIVATE KEY-----` y `-----END PRIVATE KEY-----` sin saltos de línea.
 
El siguiente código JSON es un ejemplo de una llamada a la función de Azure:

```json
{
    "appleTeamId": "ABC123DEFG",
    "appleKeyId": "URKEYID001",
    "appleServiceId": "com.yourcompany.app1",
    "p8key": "MIGTAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBHkwdwIBAQQg+s07NiAcuGEu8rxsJBG7ttupF6FRe3bXdHxEipuyK82gCgYIKoZIzj0DAQehRANCAAQnR1W/KbbaihTQayXH3tuAXA8Aei7u7Ij5OdRy6clOgBeRBPy1miObKYVx3ki1msjjG2uGqRbrc1LvjLHINWRD"
}
```

La función de Azure responde con un JWT de secreto de cliente con el formato correcto y firmado en una respuesta; por ejemplo:

```json
{
    "token": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJjb20ueW91cmNvbXBhbnkuYXBwMSIsIm5iZiI6MTU2MDI2OTY3NSwiZXhwIjoxNTYwMzU2MDc1LCJpc3MiOiJBQkMxMjNERUZHIiwiYXVkIjoiaHR0cHM6Ly9hcHBsZWlkLmFwcGxlLmNvbSJ9.Dt9qA9NmJ_mk6tOqbsuTmfBrQLFqc9BnSVKR6A-bf9TcTft2XmhWaVODr7Q9w1PP3QOYShFXAnNql5OdNebB4g"
}
```

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar el secreto de cliente que haya registrado previamente en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.
2. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. En la página de **introducción**, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva** y, luego, **Agregar**.
1. En **Opciones**, elija **Manual**.
1. Escriba un **nombre** para la clave de directiva. Por ejemplo, "AppleSecret". Se agrega el prefijo "B2C_1A_" automáticamente al nombre de la clave.
1. En **Secreto**, escriba el valor de un token devuelto por la función de Azure (un token JWT).
1. En **Uso de claves**, seleccione **Firma**.
1. Seleccione **Crear**.

> [!IMPORTANT] 
> - El inicio de sesión con Apple requiere que el administrador renueve el secreto de cliente cada 6 meses.
> - Tendrá que renovar manualmente el secreto de cliente de Apple, si expira, y almacenar el nuevo valor en la clave de la directiva.
> - Se recomienda establecer su propio recordatorio en 6 meses para generar un nuevo secreto de cliente. 

## <a name="configure-apple-as-an-identity-provider"></a>Configuración de Apple como proveedor de identidades

Para permitir que los usuarios inicien sesión con una identificador de Apple, deberá definir la cuenta como un proveedor de notificaciones con el que Azure AD B2C pueda comunicarse mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto está autenticado.

Puede definir un identificador de Apple como proveedor de notificaciones; para ello, agréguelo al elemento **ClaimsProvider** en el archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>apple.com</Domain>
      <DisplayName>Apple</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Apple-OIDC">
          <DisplayName>Apple</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="ProviderName">apple</Item>
            <Item Key="authorization_endpoint">https://appleid.apple.com/auth/authorize</Item>
            <Item Key="AccessTokenEndpoint">https://appleid.apple.com/auth/token</Item>
            <Item Key="JWKS">https://appleid.apple.com/auth/keys</Item>
            <Item Key="issuer">https://appleid.apple.com</Item>
            <Item Key="scope">name email openid</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="response_types">code</Item>
            <Item Key="external_user_identity_claim_id">sub</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="ReadBodyClaimsOnIdpRedirect">user.name.firstName user.name.lastName user.email</Item>
            <Item Key="client_id">You Apple ID</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_AppleSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="https://appleid.apple.com" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="user.name.firstName"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="user.name.lastName"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="user.email"/>
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Establezca **client_id** en el identificador del servicio. Por ejemplo, `com.consoto.azure-ad-b2c-service`.
5. Guarde el archivo.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="AppleExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="AppleExchange" TechnicalProfileReferenceId="Apple-OIDC" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **Apple** para iniciar sesión con el identificador de Apple.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

::: zone-end
