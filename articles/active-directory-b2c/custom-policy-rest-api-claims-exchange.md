---
title: 'Intercambios de notificaciones de la API de REST: Azure Active Directory B2C'
description: Agregue los intercambios de notificaciones de la API de REST a directivas personalizadas de Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 10d9cd87e15303515c02903c9127c1fd0e1c791a
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070891"
---
# <a name="walkthrough-add-rest-api-claims-exchanges-to-custom-policies-in-azure-active-directory-b2c"></a>Tutorial: Agregue los intercambios de notificaciones de la API de REST a directivas personalizadas de Azure Active Directory B2C.

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) permite a los desarrolladores de identidades integrar una interacción con una API RESTful en un recorrido del usuario. Al final de este tutorial podrá crear un recorrido del usuario de Azure AD B2C que interactúe con [servicios RESTful](api-connectors-overview.md).

En este escenario, enriquecemos los datos de token del usuario mediante la integración con un flujo de trabajo de línea de negocio corporativo. Durante el registro o inicio de sesión con una cuenta local o federada, Azure AD B2C invoca una API REST para obtener los datos de perfil extendido del usuario de un origen de datos remoto. En este ejemplo, Azure AD B2C envía el identificador único del usuario: objectId. Después, la API REST devuelve el saldo de la cuenta del usuario (un número aleatorio). Use este ejemplo como punto de partida para la integración con su propio sistema CRM, base de datos de marketing o cualquier flujo de trabajo de línea de negocio.

También puede diseñar la interacción como un perfil técnico de validación. Esto es adecuado cuando la API REST va a validar datos en pantalla y devolver notificaciones. Para más información, consulte [Tutorial: Integración de intercambios de notificaciones de API REST en el recorrido del usuario de Azure AD B2C para validar la entrada del usuario](custom-policy-rest-api-claims-validation.md).

## <a name="prerequisites"></a>Prerrequisitos

- Realice los pasos del artículo [Introducción a las directivas personalizadas](tutorial-create-user-flows.md?pivots=b2c-custom-policy). Debe tener una directiva personalizada activa para registrar e iniciar sesión de cuentas locales.
- Más información sobre cómo [integrar notificaciones de API REST en la directiva personalizada de Azure AD B2C](api-connectors-overview.md).

## <a name="prepare-a-rest-api-endpoint"></a>Preparación del punto de conexión de API REST

Para este tutorial, debe tener una API REST que valide si un objectId de Azure AD B2C de un usuario está registrado en el sistema de back-end. Si está registrado, la API REST devuelve el saldo de la cuenta de usuario. De lo contrario, la API REST registra la nueva cuenta en el directorio y devuelve el saldo inicial `50.00`.

El siguiente código JSON muestra los datos que Azure AD B2C enviará al punto de conexión de la API REST. 

```json
{
    "objectId": "User objectId",
    "lang": "Current UI language"
}
```

Una vez que la API REST valide los datos, debe devolver un código HTTP 200 (correcto), con los siguientes datos JSON:

```json
{
    "balance": "760.50"
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
<ClaimType Id="balance">
  <DisplayName>Your Balance</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="userLanguage">
  <DisplayName>User UI language (used by REST API to return localized error messages)</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

## <a name="add-the-restful-api-technical-profile"></a>Incorporación del perfil técnico de la API RESTful 

Un [perfil técnico de RESTful](restful-technical-profile.md) proporciona compatibilidad para interactuar con su propio servicio RESTful. Azure AD B2C envía datos al servicio RESTful en una colección `InputClaims` y recibe los datos en una colección `OutputClaims`. Busque el elemento **ClaimsProviders** en el archivo <em> **`TrustFrameworkExtensions.xml`**</em> y agregue un nuevo proveedor de notificaciones como se indica a continuación:

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- Set the ServiceUrl with your own REST API endpoint -->
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <!-- Set AuthenticationType to Basic or ClientCertificate in production environments -->
        <Item Key="AuthenticationType">None</Item>
        <!-- REMOVE the following line in production environments -->
        <Item Key="AllowInsecureAuthInProduction">true</Item>
      </Metadata>
      <InputClaims>
        <!-- Claims sent to your REST API -->
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <!-- Claims parsed from your REST API -->
        <OutputClaim ClaimTypeReferenceId="balance" />
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

## <a name="add-an-orchestration-step"></a>Agregar un paso de orquestación

Los [recorridos del usuario](userjourneys.md) especifican rutas de acceso explícitas con las que una directiva permite que una aplicación de usuario de confianza obtenga las notificaciones deseadas para un usuario. El recorrido del usuario se representa como una secuencia de orquestación por la que hay que pasar para lograr una transacción correcta. Puede agregar o quitar pasos de orquestación. En este caso, agregará un nuevo paso de orquestación que se usa para aumentar la información proporcionada a la aplicación después del registro o el inicio de sesión del usuario a través de la llamada a la API REST.

1. Abra el archivo base de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/`**`TrustFrameworkBase.xml`**</em>.
1. Busque el elemento `<UserJourneys>`. Copie todo el elemento y luego elimínelo.
1. Abra el archivo de extensiones de la directiva. Por ejemplo, <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Pegue `<UserJourneys>` en el archivo de extensiones, después del cierre del elemento `<ClaimsProviders>`.
1. Localice `<UserJourney Id="SignUpOrSignIn">` y agregue el siguiente paso de orquestación antes del último.

    ```xml
    <OrchestrationStep Order="7" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="RESTGetProfile" TechnicalProfileReferenceId="REST-GetProfile" />
      </ClaimsExchanges>
    </OrchestrationStep>
    ```

1. Refactorice el último paso de orquestación cambiando `Order` a `8`. Los dos pasos finales de la orquestación deben tener un aspecto similar al siguiente:

    ```xml
    <OrchestrationStep Order="7" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="RESTGetProfile" TechnicalProfileReferenceId="REST-GetProfile" />
      </ClaimsExchanges>
    </OrchestrationStep>

    <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    ```

1. Repita los dos últimos pasos para los recorridos del usuario **ProfileEdit** y **PasswordReset**.


## <a name="include-a-claim-in-the-token"></a>Incorporación de una notificación en el token 

Para devolver la notificación `balance` a la aplicación de usuario de confianza, agregue una notificación de salida al archivo <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. La incorporación de una notificación de salida emitirá la notificación al token después de un recorrido del usuario correcto y se enviará a la aplicación. Modifique el elemento de perfil técnico en la sección de usuario de confianza para agregar `balance` como una notificación de salida.
 
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
      <OutputClaim ClaimTypeReferenceId="balance" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

Repita este paso para los recorridos del usuario **ProfileEdit.xml** y **PasswordReset.xml**.

Guarde los archivos que ha cambiado: *TrustFrameworkBase.xml* y *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* y *PasswordReset.xml*. 

## <a name="test-the-custom-policy"></a>Prueba de la directiva personalizada

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD. Para ello, seleccione el filtro **Directorio y suscripción** que se encuentra en el menú superior y elija el directorio que contiene el inquilino de Azure AD.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
1. Seleccione **Marco de experiencia de identidad**.
1. Seleccione **Cargar directiva personalizada** y cargue los archivos de directiva modificados: *TrustFrameworkBase.xml* y *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* y *PasswordReset.xml*. 
1. Seleccione la directiva de registro o inicio de sesión que cargó y haga clic en el botón **Ejecutar ahora**.
1. Debe poder suscribirse con una dirección de correo electrónico o una cuenta de Facebook.
1. El token enviado de vuelta a la aplicación contiene la notificación `balance`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1584961516,
  "nbf": 1584957916,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1584957916,
  "auth_time": 1584957916,
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "balance": "202.75"
  ...
}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo proteger las API, vea los siguientes artículos:

- [Tutorial: Integración de intercambios de notificaciones de API REST en los recorridos de usuario de Azure AD B2C como un paso de orquestación](custom-policy-rest-api-claims-exchange.md)
- [Protección de la API RESTful](secure-rest-api.md)
- [Referencia: Perfil técnico de RESTful](restful-technical-profile.md)