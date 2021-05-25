---
title: Desafíos y solicitudes de notificaciones
titleSuffix: Microsoft identity platform
description: Explicación de los desafíos y solicitudes de notificaciones en la Plataforma de identidad de Microsoft.
services: active-directory
author: knicholasa
manager: martinco
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 05/11/2021
ms.author: nichola
ms.reviewer: kkrishna, kylemar
ms.openlocfilehash: bf3b6db0bcb5478412837a2fa43f610dbf8690d6
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109795619"
---
# <a name="claims-challenges-and-claims-requests"></a>Desafíos y solicitudes de notificaciones

Un **desafío de notificaciones** es una respuesta enviada desde una API que indica que un token de acceso enviado por una aplicación cliente no tiene notificaciones suficientes. Esto puede deberse a que el token no satisface las directivas de acceso condicional establecidas para esa API o a que el token de acceso se ha revocado.

La aplicación cliente realiza una **solicitud de notificaciones** para redirigir al usuario de vuelta al proveedor de identidades para recuperar un nuevo token con notificaciones que satisfagan los requisitos adicionales que no se hayan cumplido.

Las aplicaciones que usan características de seguridad mejoradas, como [Evaluación continua de acceso (CAE)](../conditional-access/concept-continuous-access-evaluation.md) y el [contexto de autenticación de acceso condicional](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/granular-conditional-access-for-sensitive-data-and-actions/ba-p/1751775), deben estar preparadas para controlar los desafíos de las notificaciones.

La aplicación solo recibirá desafíos de notificaciones si declara que puede controlarlas mediante las **funcionalidades del cliente**.

Para recibir información sobre si las aplicaciones cliente pueden controlar los desafíos de notificaciones, un implementador de API debe solicitar **xms_cc** como una notificación opcional en su manifiesto de aplicación.

## <a name="claims-challenge-header-format"></a>Formato de encabezado de desafío de notificaciones

El desafío de notificaciones es una directiva en el encabezado www-authenticate devuelto por una API cuando un token de acceso no está autorizado y se requiere un nuevo token de acceso. El desafío de notificaciones consta de varias partes: el código de estado HTTP de la respuesta y el encabezado www-authenticate, que a su vez tiene varias partes y debe contener una directiva de notificaciones.

``` https
HTTP 401; Unauthorized

www-authenticate =Bearer realm="", authorization_uri="https://login.microsoftonline.com/common/oauth2/authorize", error="insufficient_claims", claims="eyJhY2Nlc3NfdG9rZW4iOnsiYWNycyI6eyJlc3NlbnRpYWwiOnRydWUsInZhbHVlIjoiYzEifX19"
```

 **Código de estado HTTP**: debe ser **401 No autorizado**.

**Encabezado de respuesta www-authenticate** que contiene:

| Parámetro    | Obligatorio/opcional | Descripción |
|--------------|-------------|--------------|
| Tipo de autenticación | Obligatorio | Debe ser **Portador**.|
| Dominio | Opcional | Identificador de inquilino o nombre de dominio de inquilino (por ejemplo, microsoft.com) al que se accede. DEBE ser una cadena vacía en el caso de que la autenticación pase por el [punto de conexión común](howto-convert-app-to-be-multi-tenant.md#update-your-code-to-send-requests-to-common). |
| `authorization_uri` | Obligatorio | El URI del punto de conexión de autorización donde se puede realizar una autenticación interactiva si es necesario. Si se especifica en el dominio, la información del inquilino DEBE incluirse en authorization_uri. Si el dominio es una cadena vacía, authorization_uri DEBE estar en el [punto de conexión común](howto-convert-app-to-be-multi-tenant.md#update-your-code-to-send-requests-to-common). |
| `error` | Obligatorio | Tiene que ser "insufficient_claims" cuando se deba generar un desafío de notificaciones. | 
| `claims` | Se requiere cuando el error es "insufficient_claims". | Cadena entre comillas que contiene una [solicitud de notificaciones](https://openid.net/specs/openid-connect-core-1_0.html#ClaimsParameter) codificada en base 64. La solicitud de notificaciones debe solicitarlas para "access_token" en el nivel superior del objeto JSON. El valor (notificaciones solicitadas) dependerá del contexto y se especificará más adelante en este documento. Por motivos de tamaño, las aplicaciones de usuario de confianza DEBEN minificar el objeto JSON antes de la codificación en base 64. El objeto JSON sin formato del ejemplo anterior es {"access_token":{"acrs":{"essential":true,"value":"cp1"}}}. |

La respuesta 401 puede contener más de un encabezado www-authenticate. Todos los campos anteriores deben estar incluidos en el mismo encabezado www-authenticate. El encabezado www-authenticate con el desafío de notificaciones PUEDE contener otros campos. Los campos del encabezado están desordenados. Según RFC 7235, cada nombre de parámetro debe aparecer solo una vez por cada desafío del esquema de autenticación.

## <a name="claims-request"></a>Solicitud de notificaciones

Cuando una aplicación recibe un desafío de notificaciones que indica que el token de acceso anterior ya no se considera válido, la aplicación debe borrar el token de cualquier caché local o sesión de usuario. A continuación, debe redirigir al usuario que ha iniciado sesión de nuevo a Azure AD para recuperar un nuevo token mediante el [flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md) con un parámetro **claims** que cumplirá los requisitos adicionales que no se han satisfecho.

Se incluye un ejemplo a continuación:

``` https
GET https://login.microsoftonline.com/14c2f153-90a7-4689-9db7-9543bf084dad/oauth2/v2.0/authorize
?client_id=2810aca2-a927-4d26-8bca-5b32c1ef5ea9
&redirect_uri=https%3A%2F%contoso.com%3A44321%2Fsignin-oidc
&response_type=code
&scope=openid%20profile%20offline_access%20user.read%20Sites.Read.All
&response_mode=form_post
&login_hint=kalyan%ccontoso.onmicrosoft.com
&domain_hint=organizations
claims=%7B%22access_token%22%3A%7B%22acrs%22%3A%7B%22essential%22%3Atrue%2C%22value%22%3A%22urn%3Amicrosoft%3Areq1%22%7D%7D%7D
```

El desafío de notificaciones debe pasarse como parte de todas las llamadas al punto de conexión [/authorize](v2-oauth2-auth-code-flow.md#request-an-authorization-code) de Azure AD hasta que se recupere correctamente un token, después del cual ya no es necesario.

Para rellenar el parámetro de notificaciones, el desarrollador tiene que:

1. Descodificar la cadena en base 64 recibida anteriormente.
2. Codificar la cadena y volver a agregarla al parámetro **claims**.

Tras la finalización de este flujo, la aplicación recibirá un token de acceso que tiene las notificaciones adicionales que demuestran que el usuario cumple las condiciones requeridas.

## <a name="client-capabilities"></a>Funcionalidades del cliente

La aplicación solo recibirá desafíos de notificaciones si declara que puede controlarlas mediante las **funcionalidades del cliente**.

Para evitar el tráfico adicional o el impacto en la experiencia del usuario, Azure D no asume que la aplicación puede controlar las notificaciones desafiadas a menos que se opte explícitamente por ello. La aplicación no recibirá desafíos de notificaciones (y no podrá usar las características relacionadas, como los tokens CAE) a menos que declare que está preparada para controlarlas con la funcionalidad "cp1".

### <a name="how-to-communicate-client-capabilities-to-azure-ad"></a>Cómo comunicar las funcionalidades del cliente a Azure AD

El parámetro de notificaciones del ejemplo siguiente muestra cómo una aplicación cliente comunica su funcionalidad a Azure AD en un [flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md).

```json
Claims: {"access_token":{"xms_cc":{"values":["cp1"]}}}
```

Los que usen la biblioteca MSAL utilizarán el código siguiente:

```c#
_clientApp = PublicClientApplicationBuilder.Create(App.ClientId)
 .WithDefaultRedirectUri()
 .WithAuthority(authority)
 .WithClientCapabilities(new [] {"cp1"})
 .Build();*
```

Los que usan Microsoft.Identity.Web pueden agregar el código siguiente al archivo de configuración:

```c#
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    // the remaining settings
    // ... 
    "ClientCapabilities": [ "cp1" ]
},
```

Un ejemplo de cómo será la solicitud a Azure AD:

```https
GET https://login.microsoftonline.com/14c2f153-90a7-4689-9db7-9543bf084dad/oauth2/v2.0/authorize
?client_id=2810aca2-a927-4d26-8bca-5b32c1ef5ea9
&redirect_uri=https%3A%2F%contoso.com%3A44321%2Fsignin-oidc
&response_type=code
&scope=openid%20profile%20offline_access%20user.read%20Sites.Read.All
&response_mode=form_post
&login_hint=kalyan%ccontoso.onmicrosoft.com
&domain_hint=organizations
&claims=%7B%22access_token%22%3A%7B%22xms_cc%22%3A%7B%22values%22%3A%5B%22cp1%22%5D%7D%7D%7D
```

Cuando ya tenga una carga existente para el parámetro de notificaciones, puede agregar esto al conjunto existente.

Por ejemplo, si ya tiene la siguiente respuesta de una operación de contexto de autenticación de acceso de condición,

```json
{"access_token":{"acrs":{"essential":true,"value":"c25"}}}
```

va a anteponer la funcionalidad del cliente en la carga de **notificaciones** existente.

```json
{"access_token":{"xms_cc":{"values":["cp1"]},"acrs":{"essential":true,"value":"c25"}}}
```

## <a name="receiving-xms_cc-claim-in-an-access-token"></a>Recepción de la notificación xms_cc en un token de acceso

Para recibir información sobre si las aplicaciones cliente pueden controlar los desafíos de notificaciones, un implementador de API debe solicitar **xms_cc** como una notificación opcional en su manifiesto de aplicación.

La notificación **xms_cc** con un valor de "cp1" en el token de acceso es la manera autoritativa de identificar que una aplicación cliente es capaz de controlar un desafío de notificaciones. **xms_cc** es una notificación opcional que no siempre se emite en el token de acceso, incluso si el cliente envía una solicitud de notificaciones con "xms_cc". Para que un token de acceso contenga la notificación **xms_cc**, la aplicación de recursos (es decir, el implementador de API) debe solicitar xms_cc como una [notificación opcional](active-directory-optional-claims.md) en su manifiesto de aplicación. Cuando se solicita como una notificación opcional, **xms_cc** se agregará al token de acceso solo si la aplicación cliente envía **xms_cc** en la solicitud de notificaciones. El valor de la solicitud de notificación **xms_cc** se incluirá como el valor de la notificación **xms_cc** en el token de acceso, si es un valor conocido. El único valor conocido actualmente es **cp1**.

Los valores no distinguen mayúsculas de minúsculas y no están ordenados. Si se especifica más de un valor en la solicitud de notificación **xms_cc**, esos valores serán una colección con varios valores como valor de la notificación **xms_cc**.

Una solicitud de:

```json
{ "access_token": { "xms_cc":{"values":["cp1","foo", "bar"] } }}
```

dará como resultado una notificación de

```json
"xms_cc": ["cp1", "foo", "bar"]
```

en el token de acceso, si **cp1**, **foo** y **bar** son funcionalidades conocidas.

Este es el aspecto del manifiesto de la aplicación después de que la [notificación opcional](active-directory-optional-claims.md) **xms_cc** se haya solicitado.

```c#
"optionalClaims":
{
    "accessToken": [
    {
        "additionalProperties": [],
        "essential": false,
        "name": "xms_cc",
        "source": null
    }],
    "idToken": [],
    "saml2Token": []
}
```

A continuación, la API puede personalizar sus respuestas en función de si el cliente es capaz de controlar el desafío de notificaciones o no.

Un ejemplo en C#

```c#
Claim ccClaim = context.User.FindAll(clientCapabilitiesClaim).FirstOrDefault(x => x.Type == "xms_cc");
if (ccClaim != null && ccClaim.Value == "cp1")
{
    // Return formatted claims challenge as this client understands this
}
else
{
    // Throw generic exception
    throw new UnauthorizedAccessException("The caller does not meet the authentication bar to carry our this operation. The service cannot allow this operation");
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Plataforma de identidad y flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md#request-an-authorization-code)
- [Uso de las API habilitadas para la evaluación continua de acceso en las aplicaciones](app-resilience-continuous-access-evaluation.md)
- [Acceso condicional granular para acciones y datos confidenciales](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/granular-conditional-access-for-sensitive-data-and-actions/ba-p/1751775)
