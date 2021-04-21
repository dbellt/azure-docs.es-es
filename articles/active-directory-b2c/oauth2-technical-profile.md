---
title: Definición de un perfil técnico de OAuth2 en una directiva personalizada
titleSuffix: Azure AD B2C
description: Defina un perfil técnico de OAuth2 en una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: e87772b6911e69b94f66cf09f0700f0025947fd0
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107283837"
---
# <a name="define-an-oauth2-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Definición de un perfil técnico de OAuth2 en una directiva personalizada en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) proporciona compatibilidad con el proveedor de identidades del protocolo OAuth2. OAuth2 es el protocolo principal para la autorización y la autenticación delegada. Para obtener más información, consulte el artículo [RFC 6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749) (RFC 6749 Plataforma de autorización de OAuth 2.0). Con un perfil técnico de OAuth2, puede realizar la federación con un proveedor de identidades basado en OAuth2, como Facebook. Esta federación permite a los usuarios iniciar sesión con sus identidades de redes sociales o de empresa existentes.

## <a name="protocol"></a>Protocolo

El atributo **Name** del elemento **Protocol** tiene que establecerse en `OAuth2`. Por ejemplo, el protocolo para el perfil técnico **Facebook-OAUTH** es `OAuth2`:

```xml
<TechnicalProfile Id="Facebook-OAUTH">
  <DisplayName>Facebook</DisplayName>
  <Protocol Name="OAuth2" />
  ...
```

## <a name="input-claims"></a>Notificaciones de entrada

Los elementos **InputClaims** y **InputClaimsTransformations** no son necesarios. Pero puede que quiera enviar otros parámetros al proveedor de identidades. En el ejemplo siguiente se agrega el parámetro de cadena de consulta **domain_hint** con el valor de `contoso.com` a la solicitud de autorización.

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="domain_hint" DefaultValue="contoso.com" />
</InputClaims>
```

## <a name="output-claims"></a>Notificaciones de salida

El elemento **OutputClaims** contiene una lista de notificaciones devueltas por el proveedor de identidades de OAuth2. Puede que tenga que asignar el nombre de la notificación definida en la directiva al nombre definido en el proveedor de identidades. También puede incluir notificaciones no especificadas por el proveedor de identidades, siempre que establezca el atributo `DefaultValue`.

El elemento **OutputClaimsTransformations** puede contener una colección de elementos **OutputClaimsTransformation** que se usan para modificar las notificaciones de salida o para generar nuevas.

El ejemplo siguiente muestra las notificaciones devueltas por el proveedor de identidades de Facebook:

- La notificación **first_name** se asigna a la notificación **givenName**.
- La notificación **last_name** se asigna a la notificación **surname**.
- La notificación **displayName** sin asignación de nombre.
- La notificación **email** sin asignación de nombre.

El perfil técnico también muestra la notificaciones no proporcionadas por el proveedor de identidades:

- La notificación **identityProvider** que contiene el nombre del proveedor de identidades.
- La notificación **authenticationSource** con un valor predeterminado de **socialIdpAuthentication**.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="id" />
  <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
  <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="facebook.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
</OutputClaims>
```

## <a name="metadata"></a>Metadatos

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| client_id | Sí | El identificador de la aplicación del proveedor de identidades. |
| IdTokenAudience | No | El público de id_token. Si se especifica, Azure AD B2C comprueba si el token se encuentra en una notificación proporcionada por el proveedor de identidades y si es igual al especificado. |
| authorization_endpoint | Sí | La dirección URL del punto de conexión de autorización de acuerdo con RFC 6749. |
| AccessTokenEndpoint | Sí | La dirección URL del punto de conexión del token de acuerdo con RFC 6749. |
| ClaimsEndpoint | Sí | La dirección URL del punto de conexión de la información de usuario de acuerdo con RFC 6749. |
| end_session_endpoint | Sí | Dirección URL del último punto de conexión de la sesión según RFC 6749. |
| AccessTokenResponseFormat | No | El formato de la llamada al punto de conexión del token de acceso. Por ejemplo, Facebook requiere un método HTTP GET, pero la respuesta del token de acceso está en formato JSON. |
| AdditionalRequestQueryParameters | No | Parámetros de consulta de solicitud adicionales. Por ejemplo, es posible que desee enviar parámetros adicionales a su proveedor de identidades. Puede incluir varios parámetros usando la coma como delimitador. |
| ClaimsEndpointAccessTokenName | No | El nombre del parámetro de la cadena de consulta del token de acceso. Los puntos de conexión de notificaciones de algunos proveedores de identidades admiten la solicitud GET HTTP. En este caso, se envía el token de portador mediante un parámetro de cadena de consulta en lugar del encabezado de autorización. Valor predeterminado: `access_token`. |
| ClaimsEndpointFormatName | No | El nombre del parámetro de cadena de consulta de formato. Por ejemplo, puede establecer el nombre como `format` en este punto de conexión de notificaciones de LinkedIn `https://api.linkedin.com/v1/people/~?format=json`. |
| ClaimsEndpointFormat | No | El valor del parámetro de cadena de consulta de formato. Por ejemplo, puede establecer el valor como `json` en este punto de conexión de notificaciones de LinkedIn `https://api.linkedin.com/v1/people/~?format=json`. |
| BearerTokenTransmissionMethod | No | Especifica cómo se envía el token. El método predeterminado es una cadena de consulta. Para enviar el token como un encabezado de solicitud, establezca en `AuthorizationHeader`. |
| ProviderName | No | Nombre del proveedor de identidades. |
| response_mode | No | Método que usará el proveedor de identidades para enviar de vuelta el resultado Azure AD B2C. Valores posibles: `query`, `form_post` (predeterminado) o `fragment`. |
| scope | No | El ámbito de la solicitud que se define según la especificación del proveedor de identidades de OAuth2. Por ejemplo, `openid`, `profile` y `email`. |
| HttpBinding | No | Enlace HTTP esperado al token de acceso y los puntos de conexión del token de notificaciones. Valores posibles: `GET` o `POST`.  |
| ResponseErrorCodeParamName | No | El nombre del parámetro que contiene el mensaje de error devuelto a través de HTTP 200 (OK). |
| ExtraParamsInAccessTokenEndpointResponse | No | Contiene los parámetros adicionales que pueden devolverse en la respuesta de **AccessTokenEndpoint** por algunos proveedores de identidades. Por ejemplo, la respuesta de **AccessTokenEndpoint** contiene un parámetro adicional, como `openid`, que es un parámetro obligatorio además del elemento access_token en una cadena de consulta de solicitud **ClaimsEndpoint**. Varios nombres de parámetro deben convertirse en caracteres de escape y estar separados por el delimitador de coma ",". |
| ExtraParamsInClaimsEndpointRequest | No | Contiene los parámetros adicionales que pueden devolverse en la solicitud **ClaimsEndpoint** por algunos proveedores de identidades. Varios nombres de parámetro deben convertirse en caracteres de escape y estar separados por el delimitador de coma ",". |
| IncludeClaimResolvingInClaimsHandling  | No | En el caso de las notificaciones de entrada y salida, especifica si se incluye la [resolución de notificaciones](claim-resolver-overview.md) en el perfil técnico. Valores posibles: `true` o `false` (valor predeterminado). Si desea utilizar un solucionador de notificaciones en el perfil técnico, establézcalo en `true`. |
| ResolveJsonPathsInJsonTokens  | No | Indica si el perfil técnico resuelve las rutas de acceso JSON. Valores posibles: `true` o `false` (valor predeterminado). Use estos metadatos para leer datos de un elemento JSON anidado. En un objeto [OutputClaim](technicalprofiles.md#output-claims), establezca `PartnerClaimType` en el elemento de la ruta de acceso JSON que quiere generar. Por ejemplo: `firstName.localized` o `data.0.to.0.email`.|
|token_endpoint_auth_method| No| Especifica cómo Azure AD B2C envía el encabezado de autenticación al punto de conexión del token. Valores posibles: `client_secret_post` (valor predeterminado), `client_secret_basic` (versión preliminar pública) y `private_key_jwt` (versión preliminar pública). Para obtener más información, consulte la sección [Autenticación de cliente de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication). |
|token_signing_algorithm| No | Especifica el algoritmo de firma que se va a usar cuando `token_endpoint_auth_method` se establece en `private_key_jwt`. Valores posibles: `RS256` (predeterminado) o `RS512`.|
|SingleLogoutEnabled| No | Indica si, durante el inicio de sesión, el perfil técnico intenta cerrar sesión desde los proveedores de identidades federados. Para obtener más información, consulte [Cierre de sesión de Azure AD B2C](session-behavior.md#sign-out). Valores posibles: `true` (opción predeterminada) o `false`.|
| UsePolicyInRedirectUri | No | Indica si se usará una directiva al crear el URI de redireccionamiento. Al configurar la aplicación en el proveedor de identidades, necesita especificar el URI de redireccionamiento. El URI de redireccionamiento apunta a Azure AD B2C, `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp`. Si especifica `true`, tendrá que agregar un URI de redireccionamiento por cada directiva que use. Por ejemplo: `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/{policy-name}/oauth2/authresp`. |

## <a name="cryptographic-keys"></a>Claves de cifrado

El elemento **CryptographicKeys** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| client_secret | Sí | Secreto de cliente de la aplicación del proveedor de identidades. La clave de cifrado es necesaria solo si los metadatos de **response_type** se establecen en `code`. En este caso, Azure AD B2C realiza otra llamada para cambiar el código de autorización por un token de acceso. Si los metadatos se establecen en `id_token`, puede omitir la clave criptográfica. |

## <a name="redirect-uri"></a>URI de redireccionamiento

Al configurar el URI de redireccionamiento del proveedor de identidades, escriba `https://{tenant-name}.b2clogin.com/{tenant-name}.onmicrosoft.com/oauth2/authresp`. Asegúrese de reemplazar `{tenant-name}` con el nombre de su inquilino (por ejemplo, contosob2c). El URI de redireccionamiento necesita estar escrito todo en minúsculas.

Ejemplos:

- [Adición de Google + como un proveedor de identidades de OAuth2 mediante directivas personalizadas](identity-provider-google.md)
