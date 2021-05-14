---
title: Definición de un perfil técnico de OpenID Connect en una directiva personalizada
titleSuffix: Azure AD B2C
description: Defina un perfil técnico de OpenID Connect en una directiva personalizada de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/04/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: fea42cb89dce717431c188deeb2ce83f9413f560
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107283888"
---
# <a name="define-an-openid-connect-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Definición de un perfil técnico de OpenID Connect en una directiva personalizada de Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) ofrece compatibilidad con el proveedor de identidades del protocolo [OpenID Connect](https://openid.net/2015/04/17/openid-connect-certification-program/). OpenID Connect 1.0 define un nivel de identidad sobre OAuth 2.0 y representa la tecnología más avanzada en los protocolos de autenticación moderna. Con un perfil técnico de OpenID Connect, puede realizar la federación con un proveedor de identidades basado en OpenID Connect, como Azure AD. Esta federación permite a los usuarios iniciar sesión con sus identidades de redes sociales o de empresa existentes.

## <a name="protocol"></a>Protocolo

El atributo **Name** del elemento **Protocol** tiene que establecerse en `OpenIdConnect`. Por ejemplo, el protocolo del perfil técnico **MSA-OIDC** es `OpenIdConnect`:

```xml
<TechnicalProfile Id="MSA-OIDC">
  <DisplayName>Microsoft Account</DisplayName>
  <Protocol Name="OpenIdConnect" />
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

El elemento **OutputClaims** contiene una lista de notificaciones proporcionada por el proveedor de identidades de OpenID Connect. Puede que tenga que asignar el nombre de la notificación definida en la directiva al nombre definido en el proveedor de identidades. También puede incluir notificaciones no especificadas por el proveedor de identidades, siempre que establezca el atributo `DefaultValue`.

El elemento **OutputClaimsTransformations** puede contener una colección de elementos **OutputClaimsTransformation** que se usan para modificar las notificaciones de salida o para generar nuevas.

En el ejemplo siguiente, se muestran las notificaciones proporcionadas por el proveedor de identidades de cuentas Microsoft:

- La notificación **sub** que se asigna a la notificación **issuerUserId**.
- La notificación **name** que se asigna a la notificación **displayName**.
- El **correo electrónico** sin asignación de nombre.

El perfil técnico también muestra la notificaciones no proporcionadas por el proveedor de identidades:

- La notificación **identityProvider** que contiene el nombre del proveedor de identidades.
- La notificación **authenticationSource** con un valor predeterminado de **socialIdpAuthentication**.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="live.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
</OutputClaims>
```

## <a name="metadata"></a>Metadatos

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| client_id | Sí | El identificador de la aplicación del proveedor de identidades. |
| IdTokenAudience | No | El público de id_token. Si se especifica, Azure AD B2C comprueba si la notificación `aud` de un token proporcionada por el proveedor de identidades es igual a la especificada en los metadatos de IdTokenAudience.  |
| METADATOS | Sí | Una dirección URL que apunta a un documento de configuración del proveedor de identidades de OpenID Connect, que es un punto de conexión de configuración conocido de OpenID. La dirección URL puede contener la expresión `{tenant}`, que se reemplaza por el nombre del inquilino.  |
| authorization_endpoint | No | Una dirección URL que apunta a un punto de conexión de autorización de configuración del proveedor de identidades de OpenID Connect. El valor de los metadatos de authorization_endpoint tiene prioridad sobre el elemento `authorization_endpoint` especificado en el punto de conexión de configuración conocido de OpenID. La dirección URL puede contener la expresión `{tenant}`, que se reemplaza por el nombre del inquilino. |
| end_session_endpoint | No | Dirección URL del último punto de conexión de la sesión. El valor de los metadatos de authorization_endpoint tiene prioridad sobre el elemento `end_session_endpoint` especificado en el punto de conexión de configuración conocido de OpenID. |
| issuer | No | El identificador único de un proveedor de identidades de OpenID Connect. El valor de los metadatos de issuer tiene prioridad sobre el elemento `issuer` especificado en el punto de conexión de configuración conocido de OpenID.  Si se especifica, Azure AD B2C comprueba si la notificación `iss` de un token proporcionada por el proveedor de identidades es igual a la especificada en los metadatos de issuer. |
| ProviderName | No | Nombre del proveedor de identidades.  |
| response_types | No | Tipo de respuesta según la especificación OpenID Connect Core 1.0. Valores posibles: `id_token`, `code` o `token`. |
| response_mode | No | Método que usará el proveedor de identidades para enviar de vuelta el resultado Azure AD B2C. Valores posibles: `query`, `form_post` (predeterminado) o `fragment`. |
| scope | No | Ámbito de la solicitud definida según la especificación OpenID Connect Core 1.0. Por ejemplo, `openid`, `profile` y `email`. |
| HttpBinding | No | Enlace HTTP esperado al token de acceso y los puntos de conexión del token de notificaciones. Valores posibles: `GET` o `POST`.  |
| ValidTokenIssuerPrefixes | No | Clave que puede usarse para iniciar sesión en todos los inquilinos al usar un proveedor de identidades multiinquilino, como Azure Active Directory. |
| UsePolicyInRedirectUri | No | Indica si se usará una directiva al crear el URI de redireccionamiento. Al configurar la aplicación en el proveedor de identidades, necesita especificar el URI de redireccionamiento. El URI de redireccionamiento apunta a Azure AD B2C, `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp`.  Si especifica `true`, tendrá que agregar un URI de redireccionamiento por cada directiva que use. Por ejemplo: `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/{policy-name}/oauth2/authresp`. |
| MarkAsFailureOnStatusCode5xx | No | Indica si una solicitud a un servicio externo tiene que marcarse como un error si el código de estado HTTP se encuentra en el intervalo 5xx. El valor predeterminado es `false`. |
| DiscoverMetadataByTokenIssuer | No | Indica si los metadatos de OIDC tienen que detectarse con el emisor en el token JWT. |
| IncludeClaimResolvingInClaimsHandling  | No | En el caso de las notificaciones de entrada y salida, especifica si se incluye la [resolución de notificaciones](claim-resolver-overview.md) en el perfil técnico. Valores posibles: `true` o `false` (valor predeterminado). Si desea utilizar un solucionador de notificaciones en el perfil técnico, establézcalo en `true`. |
|token_endpoint_auth_method| No | Especifica cómo Azure AD B2C envía el encabezado de autenticación al punto de conexión del token. Valores posibles: `client_secret_post` (valor predeterminado), `client_secret_basic` (versión preliminar pública) y `private_key_jwt` (versión preliminar pública). Para obtener más información, consulte la sección [Autenticación de cliente de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication). |
|token_signing_algorithm| No | Especifica el algoritmo de firma que se va a usar cuando `token_endpoint_auth_method` se establece en `private_key_jwt`. Valores posibles: `RS256` (predeterminado) o `RS512`.|
| SingleLogoutEnabled | No | Indica si, durante el inicio de sesión, el perfil técnico intenta cerrar sesión desde los proveedores de identidades federados. Para obtener más información, consulte [Cierre de sesión de Azure AD B2C](./session-behavior.md#sign-out).  Valores posibles: `true` (opción predeterminada) o `false`. |
|ReadBodyClaimsOnIdpRedirect| No| Establézcalo en `true` para leer notificaciones del cuerpo de la respuesta en la redirección del proveedor de identidades. Estos metadatos se usan con el [identificador de Apple](identity-provider-apple-id.md), donde las notificaciones devuelven la carga de respuesta.|

```xml
<Metadata>
  <Item Key="ProviderName">https://login.live.com</Item>
  <Item Key="METADATA">https://login.live.com/.well-known/openid-configuration</Item>
  <Item Key="response_types">code</Item>
  <Item Key="response_mode">form_post</Item>
  <Item Key="scope">openid profile email</Item>
  <Item Key="HttpBinding">POST</Item>
  <Item Key="UsePolicyInRedirectUri">false</Item>
  <Item Key="client_id">Your Microsoft application client ID</Item>
</Metadata>
```

### <a name="ui-elements"></a>Elementos de interfaz de usuario
 
La configuración siguiente se puede usar para establecer el mensaje de error que se muestra cuando se produce un error. Los metadatos se deben configurar en el perfil técnico de OpenID Connect. Los mensajes de error se pueden [localizar](localization-string-ids.md#sign-up-or-sign-in-error-messages).

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| UserMessageIfClaimsPrincipalDoesNotExist | No | Mensaje que se mostrará al usuario si una cuenta con el nombre de usuario proporcionado no está en el directorio |
| UserMessageIfInvalidPassword | No | Mensaje que se mostrará al usuario si la contraseña es incorrecta |
| UserMessageIfOldPasswordUsed| No |  Mensaje que se mostrará al usuario si se usa una contraseña antigua|

## <a name="cryptographic-keys"></a>Claves de cifrado

El elemento **CryptographicKeys** contiene el atributo siguiente:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| client_secret | Sí | Secreto de cliente de la aplicación del proveedor de identidades. La clave criptográfica es necesaria solo si los metadatos de **response_types** se establecen en `code` y **token_endpoint_auth_method** se establece en `client_secret_post` o `client_secret_basic`. En este caso, Azure AD B2C realiza otra llamada para cambiar el código de autorización por un token de acceso. Si los metadatos se establecen en `id_token`, puede omitir la clave criptográfica.  |
| assertion_signing_key | Sí | Clave privada RSA que se utilizará para firmar la aserción del cliente. Esta clave criptográfica solo es necesaria si los metadatos de **token_endpoint_auth_method** se establecen en `private_key_jwt`. |

## <a name="redirect-uri"></a>URI de redireccionamiento

Al configurar el URI de redireccionamiento del proveedor de identidades, escriba `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp`. Asegúrese de reemplazar `{your-tenant-name}` por el nombre del inquilino. El URI de redireccionamiento necesita estar escrito todo en minúsculas.

Ejemplos:

- [Agregar una cuenta Microsoft (MSA) como un proveedor de identidades mediante directivas personalizadas](identity-provider-microsoft-account.md)
- [Iniciar sesión con cuentas de Azure AD](identity-provider-azure-ad-single-tenant.md)
- [Permitir que los usuarios inicien sesión en un proveedor de identidades de Azure AD multiinquilino mediante directivas personalizadas](identity-provider-azure-ad-multi-tenant.md)
