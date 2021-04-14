---
title: Configuración del registro y el inicio de sesión con el proveedor de identidades de SAML
titleSuffix: Azure Active Directory B2C
description: Configure el registro y el inicio de sesión con cualquier proveedor de identidades (IdP) de SAML en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/08/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 944eff6e76f4e5759f70105fe9d09aa61093917f
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028305"
---
# <a name="set-up-sign-up-and-sign-in-with-saml-identity-provider-using-azure-active-directory-b2c"></a>Configuración del registro y del inicio de sesión con un proveedor de identidades de SAML mediante Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) admite la Federación con proveedores de identidades de SAML 2.0. En este artículo se muestra cómo habilitar el inicio de sesión con una cuenta de usuario del proveedor de identidades de SAML, lo que permite a los usuarios iniciar sesión con sus identidades empresariales o de redes sociales existentes, como [ADFS](./identity-provider-adfs.md) y [Salesforce](identity-provider-salesforce-saml.md).

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="scenario-overview"></a>Información general de escenario

Puede configurar Azure AD B2C para permitir que los usuarios inicien sesión en su aplicación con las credenciales de proveedores de identidades (IdP) SAML externos de redes sociales o de empresa. Cuando Azure AD B2C se federa con un proveedor de identidades de SAML, actúa como **proveedor de servicios**. Para ello, inicia una solicitud SAML al **proveedor de identidades** de SAML y espera una respuesta SAML. En el diagrama siguiente:

1. La aplicación inicia una solicitud de autorización para Azure AD B2C. La aplicación puede ser una aplicación [OAuth 2.0](protocols-overview.md) u [OpenID Connect](openid-connect.md), o bien un [proveedor de servicios de SAML](saml-service-provider.md). 
1. En la página de inicio de sesión de Azure AD B2C, el usuario elige iniciar sesión con una cuenta de proveedor de identidades de SAML (por ejemplo, *Contoso*). Azure AD B2C inicia una solicitud de autorización de SAML y lleva al usuario al proveedor de identidades de SAML para completar el inicio de sesión.
1. El proveedor de identidades de SAML devuelve una respuesta SAML.
1. Azure AD B2C valida el token SAML, extrae notificaciones, emite su propio token y devuelve el usuario a la aplicación.  

![Inicio de sesión con el flujo del proveedor de identidades de SAML](./media/identity-provider-generic-saml/sign-in-with-saml-identity-provider-flow.png)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="components-of-the-solution"></a>Componentes de la solución

Hay tres componentes principales necesarios para este escenario:

* Un **proveedor de servicios** de SAML con capacidad para enviar, recibir, descodificar y responder solicitudes de SAML desde Azure AD B2C.
* Un **punto de conexión de metadatos** de SAML disponible públicamente para el proveedor de identidades.
* Un [inquilino de Azure AD B2C](tutorial-create-tenant.md).
 

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Para establecer la confianza entre Azure AD B2C y el proveedor de identidades de SAML, debe proporcionar un certificado X509 válido junto con la clave privada. Azure AD B2C firma las solicitudes de SAML mediante la clave privada del certificado. El proveedor de identidades valida la solicitud mediante la clave pública del certificado. Se puede acceder a la clave pública a través de los metadatos del perfil técnico. También puede cargar manualmente el archivo .cer en el proveedor de identidades SAML.

En la mayoría de los casos se acepta un certificado autofirmado. En los entornos de producción, se recomienda usar un certificado X509 emitido por una entidad de certificación. Además, como se describe más adelante en este documento, en los entornos que no son de producción, puede deshabilitar la firma de SAML en ambos lados.

### <a name="obtain-a-certificate"></a>Obtener un certificado

[!INCLUDE [active-directory-b2c-create-self-signed-certificate](../../includes/active-directory-b2c-create-self-signed-certificate.md)]

### <a name="upload-the-certificate"></a>Carga del certificado

Debe almacenar el certificado en el inquilino de Azure AD B2C.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripciones** del menú superior y elija el directorio que contiene el inquilino.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. En la página de introducción, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Upload`.
1. Escriba un **nombre** para la clave de directiva. Por ejemplo, `SAMLSigningCert`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
1. Busque el archivo .pfx de certificado con la clave privada y selecciónelo.
1. Haga clic en **Crear**.

## <a name="configure-the-saml-technical-profile"></a>Configuración del perfil técnico de SAML

Para definir el proveedor de identidades de SAML, agréguelo al elemento **ClaimsProvider** en el archivo de extensión de la directiva. Los proveedores de notificaciones contienen un perfil técnico de SAML que determina los puntos de conexión y los protocolos necesarios para comunicarse con el proveedor de identidades de SAML. Para agregar un proveedor de notificaciones con un perfil técnico de SAML:

1. Abra el archivo *TrustFrameworkExtensions.xml*.
1. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
1. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>Contoso.com</Domain>
      <DisplayName>Contoso</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Contoso-SAML2">
          <DisplayName>Contoso</DisplayName>
          <Description>Login with your SAML identity provider account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="PartnerEntity">https://your-AD-FS-domain/federationmetadata/2007-06/federationmetadata.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="assertionSubjectName" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="http://schemas.microsoft.com/identity/claims/displayname" />
            <OutputClaim ClaimTypeReferenceId="email"  />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
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

Actualice los siguientes elementos XML con el valor pertinente:

|Elemento XML  |Value  |
|---------|---------|
|ClaimsProvider\Domain  | El nombre de dominio que se usa para el [inicio de sesión directo](direct-signin.md?pivots=b2c-custom-policy#redirect-sign-in-to-a-social-provider). Escriba el nombre de dominio que quiere usar en el inicio de sesión directo. Por ejemplo, *Contoso.com*. |
|TechnicalProfile\DisplayName|Este valor se mostrará en el botón de inicio de sesión de la pantalla de inicio de sesión. Por ejemplo, *Contoso*. |
|Metadata\PartnerEntity|URL de los metadatos del proveedor de identidades SAML. O bien, puede copiar los metadatos del proveedor de identidades y agregarlos en el elemento CDATA `<![CDATA[Your IDP metadata]]>`.|

## <a name="map-the-claims"></a>Asignación de las notificaciones

El elemento **OutputClaims** contiene una lista de notificaciones devueltas por el proveedor de identidades de SAML. Asigne el nombre de la notificación definida en la directiva al nombre de aserción definido en el proveedor de identidades. Consulte el proveedor de identidades para obtener la lista de notificaciones (aserciones). Para más información, consulte [Asignación de notificaciones](identity-provider-generic-saml-options.md#claims-mapping).

En el ejemplo anterior, *Contoso-SAML2* incluye las notificaciones devueltas por un proveedor de identidades de SAML:

* La notificación **issuerUserId** se asigna a la notificación **assertionSubjectName**.
* La notificación **first_name** se asigna a la notificación **givenName**.
* La notificación **last_name** se asigna a la notificación **surname**.
* La notificación **displayName** se asigna a la notificación `http://schemas.microsoft.com/identity/claims/displayname`.
* La notificación **email** sin asignación de nombre.

El perfil técnico también muestra la notificaciones no proporcionadas por el proveedor de identidades:

* La notificación **identityProvider** que contiene el nombre del proveedor de identidades.
* La notificación **authenticationSource** con un valor predeterminado de **socialIdpAuthentication**.
 
### <a name="add-the-saml-session-technical-profile"></a>Adición del perfil técnico de sesión de SAML

Si aún no tiene el perfil técnico de sesión de SAML `SM-Saml-idp`, agregue uno a la directiva de extensión. Busque la sección `<ClaimsProviders>` y agregue el siguiente fragmento de código XML. Si la directiva ya contiene el perfil técnico `SM-Saml-idp`, vaya al paso siguiente. Para más información, consulte [Administración de sesión de inicio de sesión único](custom-policy-reference-sso.md).

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

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="Contoso-SAML2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-create-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="configure-your-saml-identity-provider"></a>Configuración del proveedor de identidades de SAML

Una vez configurada la directiva, debe configurar el proveedor de identidades de SAML con los metadatos de SAML de Azure AD B2C. Los metadatos de SAML son información que se usa en el protocolo SAML para exponer la configuración de la directiva, el **proveedor de servicios**. Definen la ubicación de los servicios, como el inicio y el cierre de sesión, los certificados, el método de inicio de sesión, etc.

Cada proveedor de identidades de SAML tiene diferentes pasos para establecer un proveedor de servicios. Algunos proveedores de identidades de SAML solicitan los metadatos de Azure AD B2C, mientras que otros requieren que recorra el archivo de metadatos manualmente y proporcione la información. Consulte la documentación del proveedor de identidades para obtener instrucciones.

En el ejemplo siguiente se muestra una dirección URL para los metadatos de SAML de un perfil técnico de Azure AD B2C:

```
https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/<your-policy>/samlp/metadata?idptp=<your-technical-profile>
```

Cuando use un [dominio personalizado](custom-domain.md), use el siguiente formato:

```
https://your-domain-name/<your-tenant-name>.onmicrosoft.com/<your-policy>/samlp/metadata?idptp=<your-technical-profile>
```

Reemplace los siguientes valores:

- **your-tenant-name** por el nombre del inquilino, como inquilino.onmicrosoft.com.
- **your-tenant-name** por el nombre del dominio personalizado, como login.contoso.com.
- **your-policy** por el nombre de la directiva. Por ejemplo, B2C_1A_signup_signin_adfs.
- **your-technical-profile** con el nombre de su perfil técnico del proveedor de identidades de SAML. Por ejemplo, Contoso-SAML2.

Abra un explorador y vaya a esta dirección URL. Asegúrese de escribir la dirección URL correcta y que tenga acceso al archivo XML de metadatos.

## <a name="test-your-custom-policy"></a>Prueba de la directiva personalizada

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione la directiva de usuarios de confianza, por ejemplo `B2C_1A_signup_signin`.
1. En **Aplicación**, seleccione la aplicación web que [registró anteriormente](tutorial-register-applications.md). La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione el botón **Ejecutar ahora**.
1. En la página de registro o de inicio de sesión, seleccione **Contoso** para iniciar sesión con la cuenta de Contoso.

Si el proceso de inicio de sesión se completa correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las opciones del proveedor de identidades de SAML con Azure Active Directory B2C](identity-provider-generic-saml-options.md)

::: zone-end