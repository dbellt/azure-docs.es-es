---
title: Configuración del inicio de sesión con un proveedor de Salesforce SAML mediante el protocolo SAML
titleSuffix: Azure AD B2C
description: Configure el inicio de sesión con un proveedor de Salesforce SAML mediante el protocolo SAML en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: cf2f0cdf8b7c540e569067d68374eef55d3479fe
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028220"
---
# <a name="set-up-sign-in-with-a-salesforce-saml-provider-by-using-saml-protocol-in-azure-active-directory-b2c"></a>Configuración del inicio de sesión con un proveedor de Salesforce SAML mediante el protocolo SAML en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"
[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En este artículo se muestra cómo habilitar el inicio de sesión para los usuarios desde una cuenta de organización de Salesforce mediante [directivas personalizadas](custom-policy-overview.md) en Azure Active Directory B2C (Azure AD B2C). Para habilitar el inicio de sesión, agregue un [proveedor de identidades de SAML](identity-provider-generic-saml.md) a una directiva personalizada.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]
- Si aún no lo ha hecho, regístrese en una [cuenta gratuita de Developer Edition](https://developer.salesforce.com/signup). Este artículo usa la [experiencia Salesforce Lightning](https://developer.salesforce.com/page/Lightning_Experience_FAQ).
- [Ha configurado un Mi dominio](https://help.salesforce.com/articleView?id=domain_name_setup.htm&language=en_US&type=0) para su organización de Salesforce.

## <a name="set-up-salesforce-as-an-identity-provider"></a>Configurar Salesforce como proveedor de identidades

1. [Inicie sesión en Salesforce](https://login.salesforce.com/).
2. En el menú de la izquierda, en **Settings** (Configuración), expanda **Identity** (Identidad) y seleccione **Identity Provider** (Proveedor de identidades).
3. Seleccione **Enable Identity Provider** (Habilitar proveedor de identidades).
4. Bajo **Select the certificate** (Seleccionar el certificado), seleccione el certificado que quiere que Salesforce use para comunicarse con Azure AD B2C. Puede usar el certificado predeterminado.
5. Haga clic en **Save**(Guardar).

### <a name="create-a-connected-app-in-salesforce"></a>Crear una aplicación conectada en Salesforce

1. En la página **Identity Provider** (proveedor de identidades), seleccione que **los proveedores de servicios se creen ahora con aplicaciones conectadas. Haga clic aquí.**
2. En **Basic Information**(Información básica), escriba los valores necesarios para la aplicación conectada.
3. En **Web App Settings** (Configuración de aplicación web), active la casilla **Enable SAML** (Habilitar SAML).
4. En el campo **Entity ID** (Id. de entidad), escriba la siguiente dirección URL. Asegúrese de reemplazar el valor de `your-tenant` con el nombre del inquilino de Azure AD B2C.

      ```
      https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase
      ```

      Cuando use un [dominio personalizado](custom-domain.md), use el siguiente formato:

      ```
      https://your-domain-name/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase
      ```

6. En el campo **ACS URL** (Dirección URL de ACS), escriba la siguiente dirección URL. Asegúrese de reemplazar el valor de `your-tenant` con el nombre del inquilino de Azure AD B2C.

      ```
      https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer
      ```

      Cuando use un [dominio personalizado](custom-domain.md), use el siguiente formato:

      ```
      https://your-domain-name/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer
      ```

7. Desplácese hasta la parte inferior de la lista y, después, haga clic en **Guardar**.

### <a name="get-the-metadata-url"></a>Obtener la dirección URL de metadatos

1. En la página de información general de la aplicación conectada, haga clic en **Administrar**.
2. Copie el valor de **punto de conexión de detección de metadatos** y después guárdelo. Lo usará más adelante en este artículo.

### <a name="set-up-salesforce-users-to-federate"></a>Configurar usuarios de Salesforce para la federación

1. En la página **Administrar** de la aplicación conectada, haga clic en **Administrar perfiles**.
2. Seleccione los perfiles (o grupos de usuarios) que quiere que realicen la federación con Azure AD B2C. Como administrador del sistema, active la casilla **Administrador del sistema** para poder realizar la federación con su cuenta de Salesforce.

## <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

[!INCLUDE [active-directory-b2c-create-self-signed-certificate](../../includes/active-directory-b2c-create-self-signed-certificate.md)]

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar el certificado que creó en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. En la página de introducción, seleccione **Identity Experience Framework**.
5. Seleccione **Claves de directiva** y luego **Agregar**.
6. En **Opciones**, elija `Upload`.
7. Escriba un **Nombre** para la directiva. Por ejemplo, SAMLSigningCert. El prefijo `B2C_1A_` se agrega automáticamente al nombre de la clave.
8. Busque el certificado B2CSigningCert.pfx que ha creado y selecciónelo.
9. Escriba la **Contraseña** del certificado.
3. Haga clic en **Crear**.

## <a name="add-a-claims-provider"></a>Incorporación de un proveedor de notificaciones

Si desea que los usuarios inicien sesión con una cuenta de Salesforce, deberá definir la cuenta como un proveedor de notificaciones con el que Azure AD B2C pueda comunicarse mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Puede definir una cuenta de Salesforce como un proveedor de notificaciones; para ello, agréguela al elemento **ClaimsProvider** en el archivo de extensión de la directiva. Para más información, consulte cómo [definir un proveedor de identidades de SAML](identity-provider-generic-saml.md).

1. Abra el archivo *TrustFrameworkExtensions.xml*.
1. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
1. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>salesforce.com</Domain>
      <DisplayName>Salesforce</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Salesforce-SAML2">
          <DisplayName>Salesforce</DisplayName>
          <Description>Login with your Salesforce account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="WantsSignedAssertions">false</Item>
            <Item Key="PartnerEntity">https://contoso-dev-ed.my.salesforce.com/.well-known/samlidp.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="userId"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="username"/>
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="salesforce.com" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-idp"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Actualice el valor de **PartnerEntity** con la dirección URL de metadatos de Salesforce que ha copiado anteriormente.
1. Actualice el valor de ambas instancias de **StorageReferenceId** con el nombre de la clave de su certificado de firma. Por ejemplo, B2C_1A_SAMLSigningCert.
1. Busque la sección `<ClaimsProviders>` y agregue el siguiente fragmento de código XML. Si la directiva ya contiene el perfil técnico `SM-Saml-idp`, vaya al paso siguiente. Para más información, consulte [Administración de sesión de inicio de sesión único](custom-policy-reference-sso.md).

    ```xml
    <ClaimsProvider>
      <DisplayName>Session Management</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SM-Saml-idp">
          <DisplayName>Session Management Provider</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="IncludeSessionIndex">false</Item>
            <Item Key="RegisterServiceProviders">false</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```
1. Guarde el archivo.

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
    <ClaimsExchange Id="SalesforceExchange" TechnicalProfileReferenceId="Salesforce-SAML2" />
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