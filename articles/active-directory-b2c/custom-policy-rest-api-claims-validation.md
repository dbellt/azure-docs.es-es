---
title: Intercambios de notificaciones de API REST como validación
titleSuffix: Azure AD B2C
description: Tutorial para la creación de un recorrido del usuario de Azure AD B2C que interactúe con servicios RESTful.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 89542a0a30bacf5cb45755fb578b22ce5c34ba4e
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070720"
---
# <a name="walkthrough-integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-user-journey-to-validate-user-input"></a>Tutorial: Integrar intercambios de notificaciones de API REST en el recorrido del usuario de Azure AD B2C para validar la entrada del usuario

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

El marco de experiencia de identidad (IEF) subyacente a Azure Active Directory B2C (Azure AD B2C) permite a los desarrolladores de identidades integrar una interacción con una API RESTful en un recorrido del usuario.  Al final de este tutorial podrá crear un recorrido del usuario de Azure AD B2C que interactúe con [servicios RESTful](api-connectors-overview.md) para validar la entrada del usuario.

En este escenario, se agregará la posibilidad de que los usuarios escriban un número de fidelidad en la página de inicio de sesión de Azure AD B2C. Validaremos si esta combinación de correo electrónico y número de fidelidad está asignada a un código promocional mediante el envío de estos datos a una API REST. Si la API REST encuentra un código promocional para este usuario, se devolverá a Azure AD B2C. Por último, el código promocional se insertará en las notificaciones del token para que la aplicación lo consuma.

La interacción también se puede diseñar como un paso de orquestación. Esto es adecuado cuando la API REST no va a validar datos en pantalla y siempre devuelve notificaciones. Para más información, consulte [Tutorial: Integración de intercambios de notificaciones de API REST en los recorridos de usuario de Azure AD B2C como un paso de orquestación](custom-policy-rest-api-claims-exchange.md).

## <a name="prerequisites"></a>Prerrequisitos

- Realice los pasos del artículo [Introducción a las directivas personalizadas](tutorial-create-user-flows.md?pivots=b2c-custom-policy). Debe tener una directiva personalizada activa para registrar e iniciar sesión de cuentas locales.
- Más información sobre cómo [integrar notificaciones de API REST en la directiva personalizada de Azure AD B2C](api-connectors-overview.md).

## <a name="prepare-a-rest-api-endpoint"></a>Preparación del punto de conexión de API REST

Para este tutorial, debe tener una API REST que valide si una dirección de correo electrónico está registrada en el sistema de back-end con un identificador de fidelidad. Si lo está, la API REST debe devolver un código de promoción de registro, que el cliente puede usar para comprar mercancías desde la aplicación. De lo contrario, la API REST debería devolver un mensaje de error HTTP 409 similar al siguiente: "El id. de fidelidad "{id. fidelidad}" no está asociado con la dirección de correo "{correo}"".

El siguiente código JSON muestra los datos que Azure AD B2C enviará al punto de conexión de la API REST. 

```json
{
    "email": "User email address",
    "language": "Current UI language",
    "loyaltyId": "User loyalty ID"
}
```

Una vez que la API REST valide los datos, debe devolver un código HTTP 200 (correcto), con los siguientes datos JSON:

```json
{
    "promoCode": "24534"
}
```

Si se produce un error en la validación, la API REST debe devolver un código HTTP 409 (conflicto), con el elemento JSON `userMessage`. IEF espera la notificación `userMessage` que devuelve la API REST. Esta notificación se presentará como una cadena para el usuario si se produce un error en la validación.

```json
{
    "version": "1.0.1",
    "status": 409,
    "userMessage": "LoyaltyId ID '1234' is not associated with 'david@contoso.com' email address."
}
```

La configuración del punto de conexión de API REST está fuera del ámbito de este artículo. Hemos creado una muestra de [Azure Functions](../azure-functions/functions-reference.md). Puede acceder al código completo de la función de Azure en [GitHub](https://github.com/azure-ad-b2c/rest-api/tree/master/source-code/azure-function).

## <a name="define-claims"></a>Definición de notificaciones

Una notificación proporciona un almacenamiento temporal de datos durante la ejecución de una directiva de Azure AD B2C. Puede declarar notificaciones dentro de la sección del [esquema de notificaciones](claimsschema.md). 

1. Abra el archivo de extensiones de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/` **`TrustFrameworkExtensions.xml`**</em>.
1. Busque el elemento [BuildingBlocks](buildingblocks.md). Si el elemento no existe, agréguelo.
1. Busque el elemento [ClaimsSchema](claimsschema.md). Si el elemento no existe, agréguelo.
1. Agregue las notificaciones siguientes al elemento **ClaimsSchema**.  

```xml
<ClaimType Id="loyaltyId">
  <DisplayName>Your loyalty ID</DisplayName>
  <DataType>string</DataType>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
<ClaimType Id="promoCode">
  <DisplayName>Your promo code</DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
</ClaimType>
  <ClaimType Id="userLanguage">
  <DisplayName>User UI language (used by REST API to return localized error messages)</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

## <a name="add-the-restful-api-technical-profile"></a>Incorporación del perfil técnico de la API RESTful 

Un [perfil técnico de RESTful](restful-technical-profile.md) proporciona compatibilidad para interactuar con su propio servicio RESTful. Azure AD B2C envía datos al servicio RESTful en una colección `InputClaims` y recibe los datos en una colección `OutputClaims`. Busque el elemento **ClaimsProviders** y agregue un nuevo proveedor de notificaciones como se indica a continuación:

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-ValidateProfile">
      <DisplayName>Check loyaltyId Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- Set the ServiceUrl with your own REST API endpoint -->
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/ValidateProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <!-- Set AuthenticationType to Basic or ClientCertificate in production environments -->
        <Item Key="AuthenticationType">None</Item>
        <!-- REMOVE the following line in production environments -->
        <Item Key="AllowInsecureAuthInProduction">true</Item>
      </Metadata>
      <InputClaims>
        <!-- Claims sent to your REST API -->
        <InputClaim ClaimTypeReferenceId="loyaltyId" />
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <!-- Claims parsed from your REST API -->
        <OutputClaim ClaimTypeReferenceId="promoCode" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

En este ejemplo, `userLanguage` se enviará al servicio REST como `lang` desde la carga de JSON. El valor de la notificación `userLanguage` contiene el identificador de idioma del usuario actual. Para obtener más información, consulte el [solucionador de notificaciones](claim-resolver-overview.md).

### <a name="configure-the-restful-api-technical-profile"></a>Configuración del perfil técnico de la API RESTful 

Después de implementar la API REST, configure los metadatos del perfil técnico `REST-ValidateProfile` para que reflejen su propia API REST, incluidos:

- **ServiceUrl**. Establezca la dirección URL del punto de conexión de la API REST.
- **SendClaimsIn**. Especifique cómo se envían las notificaciones de entrada al proveedor de notificaciones RESTful.
- **AuthenticationType**. Establezca el tipo de autenticación realizada por el proveedor de notificaciones RESTful. 
- **AllowInsecureAuthInProduction**. En un entorno de producción, asegúrese de establecer estos metadatos en `true`.
    
Consulte los [metadatos del perfil técnico de RESTful](restful-technical-profile.md#metadata) para obtener más configuraciones.

Los comentarios anteriores `AuthenticationType` y `AllowInsecureAuthInProduction` especifican los cambios que se deben realizar al pasar a un entorno de producción. Para aprender a proteger las API RESTful para la producción, consulte [Proteger la API RESTful](secure-rest-api.md).

## <a name="validate-the-user-input"></a>Validación de la entrada del usuario

Para obtener el número de fidelidad del usuario durante el registro, debe permitir que el usuario escriba estos datos en la pantalla. Agregue la notificación de salida **loyaltyId** a la página de registro. Para hacerlo, agréguela al elemento `OutputClaims` de la sección del perfil técnico de registro existente. Especifique la lista completa de notificaciones de salida para controlar el orden en que se presentan las notificaciones en la pantalla.  

Agregue la referencia del perfil técnico de validación al perfil técnico de registro, que llama a `REST-ValidateProfile`. El nuevo perfil técnico de validación se agregará al principio de la colección `<ValidationTechnicalProfiles>` definida en la directiva base. Este comportamiento significa que, solo después de la validación correcta, Azure AD B2C pasa a crear la cuenta en el directorio.   

1. Busque el elemento **ClaimsProviders**. Agregue un nuevo proveedor de notificaciones como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <DisplayName>Local Account</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="newPassword" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surName"/>
            <!-- Required to present the text box to collect the data from the user -->
            <OutputClaim ClaimTypeReferenceId="loyaltyId"/>
            <!-- Required to pass the promoCode returned from "REST-ValidateProfile" 
            to subsequent orchestration steps and token issuance-->
            <OutputClaim ClaimTypeReferenceId="promoCode" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="REST-ValidateProfile" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>
      <DisplayName>Self Asserted</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SelfAsserted-Social">
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
            <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surname"/>
            <!-- Required to present the text box to collect the data from the user -->
            <OutputClaim ClaimTypeReferenceId="loyaltyId"/>
            <!-- Required to pass the promoCode returned from "REST-ValidateProfile" 
            to subsequent orchestration steps and token issuance-->
            <OutputClaim ClaimTypeReferenceId="promoCode" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="REST-ValidateProfile"/>
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

## <a name="include-a-claim-in-the-token"></a>Incorporación de una notificación en el token 

Para devolver la notificación del código de promoción a la aplicación de usuario de confianza, agregue una notificación de salida al archivo <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. La notificación de salida permitirá agregar la notificación al token después de un recorrido del usuario correcto y se enviará a la aplicación. Modifique el elemento de perfil técnico en la sección de usuario de confianza para agregar `promoCode` como una notificación de salida.
 
```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <OutputClaim ClaimTypeReferenceId="promoCode" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

## <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD. Para ello, seleccione el filtro **Directorio y suscripción** que se encuentra en el menú superior y elija el directorio que contiene el inquilino de Azure AD.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
1. Seleccione **Marco de experiencia de identidad**.
1. Seleccione **Cargar directiva personalizada** y cargue los archivos de directiva modificados: *TrustFrameworkExtensions.xml* y *SignUpOrSignin.xml*. 
1. Seleccione la directiva de registro o inicio de sesión que cargó y haga clic en el botón **Ejecutar ahora**.
1. Debe poder registrarse con una dirección de correo electrónico.
1. Haga clic en el vínculo **Registrarse ahora**.
1. En **Your loyalty ID** (Id. de fidelidad), escriba 1234 y haga clic en **Continuar**. Llegados a este punto, debería obtener un mensaje de error de validación.
1. Cambie a otro valor y haga clic en **Continuar**.
1. El token enviado de vuelta a la aplicación contiene la notificación `promoCode`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1584295703,
  "nbf": 1584292103,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1584292103,
  "auth_time": 1584292103,
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "promoCode": "84362"
  ...
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo proteger las API, vea los siguientes artículos:

- [Tutorial: Integración de intercambios de notificaciones de API REST en los recorridos de usuario de Azure AD B2C como un paso de orquestación](custom-policy-rest-api-claims-exchange.md)
- [Protección de la API RESTful](secure-rest-api.md)
- [Referencia: Perfil técnico de RESTful](restful-technical-profile.md)