---
title: RelyingParty - Azure Active Directory B2C | Microsoft Docs
description: Especifique el elemento RelyingParty de una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/26/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9a1771dd6f312129d418cac3ddac8ef2411e95c6
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110496140"
---
# <a name="relyingparty"></a>RelyingParty

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

El elemento **RelyingParty** especifica el recorrido del usuario para ejecutar la solicitud actual en Azure Active Directory B2C (Azure AD B2C). También se especifica la lista de notificaciones que la aplicación del usuario de confianza necesita como parte del token emitido. Una aplicación del usuario de confianza, como una aplicación web, de escritorio o para dispositivos móviles, llama al archivo de la directiva del usuario de confianza. El archivo de directiva de usuario de confianza ejecuta una tarea específica, como iniciar sesión, restablecer una contraseña o editar un perfil. Hay varias aplicaciones que pueden usar la misma directiva del usuario de confianza y una sola aplicación puede usar varias directivas. Todas las aplicaciones de usuario de confianza reciben el mismo token con notificaciones y el usuario sigue el mismo recorrido del usuario.

En el ejemplo siguiente se muestra un elemento **RelyingParty** en el archivo de directiva *B2C_1A_signup_signin*:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="your-tenant.onmicrosoft.com"
  PolicyId="B2C_1A_signup_signin"
  PublicPolicyUri="http://your-tenant.onmicrosoft.com/B2C_1A_signup_signin">

  <BasePolicy>
    <TenantId>your-tenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>

  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <UserJourneyBehaviors>
      <SingleSignOn Scope="Tenant" KeepAliveInDays="7"/>
      <SessionExpiryType>Rolling</SessionExpiryType>
      <SessionExpiryInSeconds>300</SessionExpiryInSeconds>
      <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="your-application-insights-key" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      <ContentDefinitionParameters>
        <Parameter Name="campaignId">{OAUTH-KV:campaignId}</Parameter>
      </ContentDefinitionParameters>
    </UserJourneyBehaviors>
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Description>The policy profile</Description>
      <Protocol Name="OpenIdConnect" />
      <Metadata>collection of key/value pairs of data</Metadata>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="loyaltyNumber" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
  ...
```

El elemento **RelyingParty** opcional contiene los siguientes elementos:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| DefaultUserJourney | 1:1 | El recorrido del usuario predeterminado para la aplicación de usuario de confianza. |
| Puntos de conexión | 0:1 | Una lista de puntos de conexión. Para más información, consulte [Punto de conexión de UserInfo](userinfo-endpoint.md). |
| UserJourneyBehaviors | 0:1 | El ámbito de los comportamientos del recorrido del usuario. |
| TechnicalProfile | 1:1 | Un perfil técnico que es compatible con la aplicación del usuario de confianza. El perfil técnico proporciona un contrato para que la aplicación del usuario de confianza contacte con Azure AD B2C. |

## <a name="endpoints"></a>Puntos de conexión

El elemento **Puntos de conexión** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Punto de conexión | 1:1 | Una referencia a un punto de conexión.|

El elemento **Punto de conexión** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | Un identificador único del punto de conexión.|
| UserJourneyReferenceId | Sí | Identificador del recorrido del usuario en la directiva. Para más información, consulte los [recorridos del usuario](userjourneys.md).  | 

En el ejemplo siguiente se muestra un usuario de confianza con el [punto de conexión de UserInfo](userinfo-endpoint.md):

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <Endpoints>
    <Endpoint Id="UserInfo" UserJourneyReferenceId="UserInfoJourney" />
  </Endpoints>
  ...
```

## <a name="defaultuserjourney"></a>DefaultUserJourney

El elemento `DefaultUserJourney` especifica una referencia al identificador del recorrido del usuario que se define en la directiva Base o Extensions. En los ejemplos siguientes se muestra el recorrido del usuario para registrarse o iniciar sesión que se especifica en el elemento **RelyingParty**:

Directiva de *B2C_1A_signup_signin*:

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn">
  ...
```

*B2C_1A_TrustFrameWorkBase* o *B2C_1A_TrustFrameworkExtensionPolicy*:

```xml
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
  ...
```

El elemento **DefaultUserJourney** contiene el siguiente atributo:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ReferenceId | Sí | Identificador del recorrido del usuario en la directiva. Para más información, consulte los [recorridos del usuario](userjourneys.md). |

## <a name="userjourneybehaviors"></a>UserJourneyBehaviors

El elemento **UserJourneyBehaviors** contiene los siguientes elementos:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| SingleSignOn | 0:1 | Ámbito del comportamiento de sesión de inicio de sesión único (SSO) de un recorrido del usuario. |
| SessionExpiryType |0:1 | El comportamiento de autenticación de la sesión. Valores posibles: `Rolling` o `Absolute`. El valor (predeterminado) `Rolling` indica que el usuario permanece conectado siempre y cuando el usuario esté activo en la aplicación. El valor `Absolute` indica que el usuario está obligado a autenticarse de nuevo tras el período de tiempo especificado por la duración de sesión de la aplicación. |
| SessionExpiryInSeconds | 0:1 | La duración de la cookie de la sesión de Azure AD B2C especificada como un entero se ha almacenado en el explorador del usuario tras la autenticación correcta. El valor predeterminado es 86 400 segundos (24 horas). El mínimo es 900 segundos (15 minutos). El máximo es 86 400 segundos (24 horas). |
| JourneyInsights | 0:1 | La clave de instrumentación de Application Insights de Azure que se va a usar. |
| ContentDefinitionParameters | 0:1 | La lista de pares clave-valor que se anexará a la URI de carga de la definición de contenido. |
|ScriptExecution| 0:1| Modos de ejecución de [JavaScript](javascript-and-page-layout.md) admitidos. Valores posibles: `Allow` o `Disallow` (valor predeterminado).
| JourneyFraming | 0:1| Permite que la interfaz de usuario de esta directiva se cargue en un iframe. |

### <a name="singlesignon"></a>SingleSignOn

El elemento **SingleSignOn** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Ámbito | Sí | El ámbito del comportamiento de inicio de sesión único. Valores posibles: `Suppressed`, `Tenant`, `Application` o `Policy`. El valor `Suppressed` indica que el comportamiento se suprime y siempre se solicita al usuario una selección del proveedor de identidades.  El valor `Tenant` indica que el comportamiento se aplica a todas las directivas del inquilino. Por ejemplo, a un usuario que sigue dos recorridos de directiva para un inquilino no se le solicita que seleccione el proveedor de identidades. El valor `Application` indica que el comportamiento se aplica a todas las directivas de la aplicación que hace la solicitud. Por ejemplo, a un usuario que sigue dos recorridos de directiva para una aplicación no se le solicita que seleccione el proveedor de identidades. El valor `Policy` indica que el comportamiento solo se aplica a una directiva. Por ejemplo, a un usuario que sigue dos recorridos de directiva para un marco de confianza se le solicita que seleccione el proveedor de identidades al cambiar de una directiva a otra. |
| KeepAliveInDays | No | Controla cuánto tiempo permanece el usuario con la sesión iniciada. Si se establece el valor en 0, se desactiva la funcionalidad KMSI. Para más información, consulte [Mantener la sesión iniciada](session-behavior.md?pivots=b2c-custom-policy#enable-keep-me-signed-in-kmsi). |
|EnforceIdTokenHintOnLogout| No|  Haga que un token de id. emitido previamente se pase al punto de conexión de cierre de sesión como una sugerencia sobre la sesión autenticada actual del usuario final con el cliente. Valores posibles: `false` (opción predeterminada) o `true`. Para más información, consulte [Inicio de sesión web con OpenID Connect](openid-connect.md).  |


## <a name="journeyinsights"></a>JourneyInsights

El elemento **JourneyInsights** contiene los siguientes atributos:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| TelemetryEngine | Sí | El valor tiene que ser `ApplicationInsights`. |
| InstrumentationKey | Sí | Cadena que contiene la clave de instrumentación para el elemento de Application Insights. |
| DeveloperMode | Sí | Valores posibles: `true` o `false`. Si es `true`, Application Insights acelera la telemetría a través de la canalización de procesamiento. Este valor es bueno para el desarrollo, pero está restringido en volúmenes elevados. Los registros de actividad descritos aquí están diseñados solo para ayudar en el desarrollo de directivas personalizadas. No use el modo de desarrollo en producción. Los registros recopilan todas las notificaciones que se envían y se reciben de los proveedores de identidad durante el desarrollo. Si se utilizan en producción, el programador asume la responsabilidad de la información personal recopilada en el registro de Application Insights que le pertenece. Estos registros detallados solo se recopilan cuando este valor se establece en `true`.|
| ClientEnabled | Sí | Valores posibles: `true` o `false`. Si es `true`, se envía el script del lado cliente ApplicationInsights para realizar un seguimiento de la vista de página y de los errores del lado cliente. |
| ServerEnabled | Sí | Valores posibles: `true` o `false`. Si es `true`, se envía el JSON UserJourneyRecorder existente como evento personalizado a Application Insights. |
| TelemetryVersion | Sí | El valor tiene que ser `1.0.0`. |

Para obtener más información, vea [Recopilación de registros](troubleshoot-with-application-insights.md).

## <a name="contentdefinitionparameters"></a>ContentDefinitionParameters

Mediante el uso de las directivas de Azure AD B2C, puede enviar un parámetro en una cadena de consulta. Al pasar dicho parámetro al punto de conexión HTML, puede cambiar de forma dinámica el contenido de la página. Por ejemplo, puede cambiar la imagen de fondo en la página de inicio de sesión o de registro de Azure AD B2C en función de un parámetro que se pasa desde la aplicación web o dispositivo móvil. Azure AD B2C pasa los parámetros de la cadena de consulta al archivo HTML dinámico, como un archivo .aspx.

En el ejemplo siguiente se pasa un parámetro denominado `campaignId` con un valor de `hawaii` en la cadena de consulta:

`https://login.microsoft.com/contoso.onmicrosoft.com/oauth2/v2.0/authorize?pB2C_1A_signup_signin&client_id=a415078a-0402-4ce3-a9c6-ec1947fcfb3f&nonce=defaultNonce&redirect_uri=http%3A%2F%2Fjwt.io%2F&scope=openid&response_type=id_token&prompt=login&campaignId=hawaii`

El elemento **ContentDefinitionParameters** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| ContentDefinitionParameter | 0:n | Una cadena que contiene el par clave-valor que se anexa a la cadena de consulta de un URI de carga de definición de contenido. |

El elemento **ContentDefinitionParameter** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Nombre | Sí | El nombre del par clave-valor. |

Para obtener más información, vea [Configuración de la interfaz de usuario con contenido dinámico usando directivas personalizadas](customize-ui-with-html.md#configure-dynamic-custom-page-content-uri).

### <a name="journeyframing"></a>JourneyFraming

El elemento **JourneyFraming** contiene los siguientes atributos:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| habilitado | Sí | Habilita esta directiva para que se cargue en un iframe. Valores posibles: `false` (opción predeterminada) o `true`. |
| Orígenes | Sí | Contiene los dominios que cargarán el iframe. Para más información, consulte cómo [cargar Azure B2C en un iframe](embedded-login.md). |

## <a name="technicalprofile"></a>TechnicalProfile

El elemento **TechnicalProfile** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | El valor tiene que ser `PolicyProfile`. |

El elemento **TechnicalProfile** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| DisplayName | 1:1 | La cadena que contiene el nombre del perfil técnico. |
| Descripción | 0:1 | La cadena que contiene la descripción del perfil técnico. |
| Protocolo | 1:1 | Protocolo usado para la federación. |
| Metadatos | 0:1 | La colección de *Item* de los pares clave-valor usados por el protocolo para comunicarse con el punto de conexión en el transcurso de una transacción para configurar la interacción entre el usuario de confianza y otros participantes de la comunidad. |
| OutputClaims | 1:1 | Una lista de tipos de notificación que se consideran el resultado del perfil técnico. Cada uno de estos elementos contiene la referencia a un **ClaimType** ya definido en la sección **ClaimsSchema** o en una directiva de la que este archivo de directiva es heredero. |
| SubjectNamingInfo | 1:1 | El nombre del sujeto usado en los tokens. |

El elemento **Protocol** contiene el siguiente atributo:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Nombre | Sí | El nombre de un protocolo válido admitido por Azure AD B2C que se usará como parte del perfil técnico. Valores posibles: `OpenIdConnect` o `SAML2`. El valor `OpenIdConnect` representa el estándar del protocolo OpenID Connect 1.0 según la especificación de la fundación de OpenID. `SAML2` representa el estándar del protocolo SAML 2.0 según la especificación de OASIS. |

### <a name="metadata"></a>Metadatos

Cuando el protocolo es `SAML`, un elemento de metadatos contiene los siguientes elementos. Para obtener más información, consulte [Opciones para registrar una aplicación SAML en Azure AD B2C](saml-service-provider-options.md).

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| IdpInitiatedProfileEnabled | No | Indica si se admite el flujo iniciado por IDP. Valores posibles: `true` o `false` (valor predeterminado). | 
| XmlSignatureAlgorithm | No | El método que Azure AD B2C usa para firmar la respuesta SAML. Valores posibles: `Sha256`, `Sha384`, `Sha512` o `Sha1`. Asegúrese de configurar el algoritmo de firma en ambos lados con el mismo valor. Use solo el algoritmo que admite el certificado. Para configurar la aserción de SAML, consulte los [metadatos de perfil técnico del emisor de SAML](saml-issuer-technical-profile.md#metadata). |
| DataEncryptionMethod | No | Indica el método que usa Azure AD B2C para cifrar los datos, mediante el algoritmo de Estándar de cifrado avanzado (AES). Los metadatos controlan el valor del elemento `<EncryptedData>` en la respuesta de SAML. Valores posibles: `Aes256` (predeterminado), `Aes192`, `Sha512` o ` Aes128`. |
| KeyEncryptionMethod| No | Indica el método que usa Azure AD B2C para cifrar la copia de la clave que se usó para cifrar los datos. Los metadatos controlan el valor del elemento `<EncryptedKey>` en la respuesta de SAML. Valores posibles: ` Rsa15` (predeterminado): algoritmo Public Key Cryptography Standard (PKCS), versión 1.5, ` RsaOaep`: algoritmo de cifrado Optimal Asymmetric Encryption Padding (OAEP) de RSA. |
| UseDetachedKeys | No |  Valores posibles: `true` o `false` (valor predeterminado). Cuando el valor se establece en `true`, Azure AD B2C cambia el formato de las aserciones cifradas. El uso de claves desasociadas agrega la aserción cifrada como un elemento secundario de EncrytedAssertion, por oposición a EncryptedData. |
| WantsSignedResponses| No | Indica si Azure AD B2C firma la sección `Response` de la respuesta SAML. Valores posibles: `true` (predeterminado) o `false`.  |
| RemoveMillisecondsFromDateTime| No | Indica si se quitarán los milisegundos de los valores datetime de la respuesta SAML (es decir, de IssueInstant, NotBefore, NotOnOrAfter y AuthnInstant). Valores posibles: `false` (predeterminado) o `true`.  |


### <a name="outputclaims"></a>OutputClaims

El elemento **OutputClaims** contiene el elemento siguiente:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| OutputClaim | 0:n | El nombre de un tipo de notificación esperado en la lista admitida para la directiva en la que se suscribe el usuario de confianza. Esta notificación actúa como un resultado del perfil técnico. |

El elemento **OutputClaim** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Sí | Una referencia a un **ClaimType** ya definido en la sección **ClaimsSchema** del archivo de directiva. |
| DefaultValue | No | Un valor predeterminado que se puede usar si el valor de notificación está vacío. |
| PartnerClaimType | No | Envía la notificación con un nombre distinto al configurado en la definición de ClaimType. |

### <a name="subjectnaminginfo"></a>SubjectNamingInfo

Con el elemento **SubjectNameingInfo**, controla el valor del asunto del token:

- **Token JTW**: la notificación `sub`. Esta es la entidad de seguridad sobre la que el token declara información como, por ejemplo, el usuario de una aplicación. Este valor es inmutable y no se puede reasignar ni volver a usar. Se puede usar para realizar comprobaciones de autorización de forma segura, por ejemplo, cuando el token se usa para acceder a un recurso. De manera predeterminada, la notificación del asunto se rellena con el identificador de objeto del usuario del directorio. Para obtener más información, vea [Configuración de token, sesión e inicio de sesión único](session-behavior.md).
- **Token SAML**: el elemento `<Subject><NameID>`, que identifica el elemento del asunto. Se puede modificar el formato de NameId.

El elemento **SubjectNamingInfo** contiene el siguiente atributo:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ClaimType | Sí | Una referencia a un **PartnerClaimType** de una notificación de salida. La notificación de salida debe definirse en la colección **OutputClaims** de la directiva del usuario de confianza. |
| Formato | No | Se usa para que los usuarios de confianza de SAML establezcan el **formato de NameId** devuelto en la aserción de SAML. |

En el ejemplo siguiente se muestra cómo definir un usuario de confianza de OpenID Connect. La información sobre el nombre del asunto se configura como `objectId`:

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
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```
El token JWT incluye la notificación `sub` con el objectId del usuario:

```json
{
  ...
  "sub": "6fbbd70d-262b-4b50-804c-257ae1706ef2",
  ...
}
```

En el ejemplo siguiente se muestra cómo definir un usuario de confianza de SAML. La información del nombre de sujeto se ha configurado como `objectId` y se ha proporcionado el NameId `format`:

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient"/>
  </TechnicalProfile>
</RelyingParty>
```
