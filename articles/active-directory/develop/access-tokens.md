---
title: Tokens de acceso de la Plataforma de identidad de Microsoft | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre los tokens de acceso emitidos por Azure AD v1.0 y los puntos de conexión de la Plataforma de identidad de Microsoft (v2.0).
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 04/02/2021
ms.author: hirsin
ms.reviewer: mmacy, hirsin
ms.custom: aaddev, identityplatformtop40, fasttrack-edit
ms.openlocfilehash: f5a60f14799e872d835d651fc043edd27dfc6990
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105602"
---
# <a name="microsoft-identity-platform-access-tokens"></a>Tokens de acceso de la Plataforma de identidad de Microsoft

Los tokens de acceso permiten a los clientes llamar de forma segura a las API web protegidas y los usan las API web para llevar a cabo la autenticación y la autorización. Según la especificación de OAuth, los tokens de acceso son cadenas opacas sin un formato establecido: algunos proveedores de identidades (IDP) usan GUID, mientras que otros usan blobs cifrados. La Plataforma de identidad de Microsoft usa una variedad de formatos de token de acceso en función de la configuración de la API que acepta el token. Las [API personalizadas registradas por desarrolladores](quickstart-configure-app-expose-web-apis.md) en la Plataforma de identidad de Microsoft pueden elegir entre dos formatos distintos de JSON Web Token, denominados "v1" y "v2", y las API desarrolladas por Microsoft como Microsoft Graph o las API en Azure tienen otros formatos de tokens propietarios. Estos formatos propietarios pueden ser tokens cifrados, JWT o tokens especiales tipo JWT que no se validarán.

Los clientes deben tratar los tokens de acceso como cadenas opacas, ya que el contenido del token está destinado únicamente al recurso (la API). Para fines *únicamente* de validación y depuración, los desarrolladores pueden decodificar los token JWT mediante un sitio como [jwt.ms](https://jwt.ms). No obstante, tenga en cuenta que los tokens que recibe para una API de Microsoft podrían no ser siempre un token JWT y que no siempre se pueden descodificar.

Para detalles sobre lo que hay dentro del token de acceso, los clientes deben usar los datos de respuesta del token que se devuelven con el token de acceso al cliente. Cuando el cliente solicita un token de acceso, la Plataforma de identidad de Microsoft también devuelve algunos metadatos sobre el token de acceso para el consumo de la aplicación. Esta información incluye la hora de expiración del token de acceso y los ámbitos para los que es válido. Estos datos permiten a la aplicación realizar un almacenamiento inteligente en caché de los tokens de acceso sin tener que analizar el mismo token de acceso.

Consulte las secciones siguientes para saber cómo la API puede validar y utilizar las notificaciones dentro de un token de acceso.  

> [!NOTE]
> Toda la documentación de esta página, excepto donde se indique lo contrario, se aplica solo a los tokens emitidos para las API que ha registrado.  No se aplica a los tokens emitidos para las API de Microsoft ni esos tokens se pueden usar para validar el modo en que la Plataforma de identidad de Microsoft emitirá tokens para una API que cree.  

## <a name="token-formats-and-ownership"></a>Formatos y propiedad de los tokens

### <a name="v10-and-v20"></a>v1.0 y v2.0 

Hay dos versiones de tokens de acceso disponibles en la Plataforma de identidad de Microsoft: v1.0 y v2.0.  Estas versiones controlan qué notificaciones están en el token, lo que garantiza que una API web puede controlar el aspecto de sus tokens. Las API web tienen una de estas opciones seleccionada como valor predeterminado durante el registro, v1.0 en el caso de las aplicaciones que solo son de Azure AD y v2.0 para las aplicaciones que admiten cuentas de consumidor.  Las aplicaciones controlan esto mediante la configuración `accessTokenAcceptedVersion` en el [manifiesto de la aplicación](reference-app-manifest.md#manifest-reference), donde `null` y `1` generan tokens v1.0 y `2` genera tokens v2.0.

### <a name="what-app-is-a-token-for"></a>¿Para qué aplicación está destinado un token?

Hay dos partes que participan en una solicitud de token de acceso: el cliente, que es quien solicita el token, y el recurso (la API) que acepta el token cuando se llama a la API. La notificación `aud` de un token indican el recurso al que está dirigido el token (su *público*). Los clientes usan el token, pero no deben entenderlo ni intentar analizarlo. Los recursos aceptan el token.  

La Plataforma de identidad de Microsoft admite la emisión de cualquier versión de token desde cualquier punto de conexión de la versión (no están relacionados). Es por esto que una configuración de recurso `accessTokenAcceptedVersion` a `2` significa que un cliente que llama al punto de conexión v1.0 para obtener un token para esa API recibirá un token de acceso v2.0.  Los recursos siempre poseen sus tokens (aquellos con su notificación `aud`) y son las únicas aplicaciones que pueden modificar los detalles del token. Este es el motivo por el que al cambiar las [notificaciones opcionales](active-directory-optional-claims.md) del token de acceso para el *cliente* no se modifica el token de acceso que se recibe al solicitar un token para `user.read`, que es propiedad del recurso de Microsoft Graph.

### <a name="sample-tokens"></a>Tokens de ejemplo

Los tokens de las versiones v1.0 y v2.0 se ven similares y contienen muchas de las mismas notificaciones. Aquí se proporciona un ejemplo de cada uno. Sin embargo, estos tokens de ejemplo no se [validarán](#validating-tokens), ya que las claves se giraron antes de la publicación y se quitó la información personal.

#### <a name="v10"></a>v1.0

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9.eyJhdWQiOiJlZjFkYTlkNC1mZjc3LTRjM2UtYTAwNS04NDBjM2Y4MzA3NDUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTUyMjIyOS8iLCJpYXQiOjE1MzcyMzMxMDYsIm5iZiI6MTUzNzIzMzEwNiwiZXhwIjoxNTM3MjM3MDA2LCJhY3IiOiIxIiwiYWlvIjoiQVhRQWkvOElBQUFBRm0rRS9RVEcrZ0ZuVnhMaldkdzhLKzYxQUdyU091TU1GNmViYU1qN1hPM0libUQzZkdtck95RCtOdlp5R24yVmFUL2tES1h3NE1JaHJnR1ZxNkJuOHdMWG9UMUxrSVorRnpRVmtKUFBMUU9WNEtjWHFTbENWUERTL0RpQ0RnRTIyMlRJbU12V05hRU1hVU9Uc0lHdlRRPT0iLCJhbXIiOlsid2lhIl0sImFwcGlkIjoiNzVkYmU3N2YtMTBhMy00ZTU5LTg1ZmQtOGMxMjc1NDRmMTdjIiwiYXBwaWRhY3IiOiIwIiwiZW1haWwiOiJBYmVMaUBtaWNyb3NvZnQuY29tIiwiZmFtaWx5X25hbWUiOiJMaW5jb2xuIiwiZ2l2ZW5fbmFtZSI6IkFiZSAoTVNGVCkiLCJpZHAiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMjIyNDcvIiwiaXBhZGRyIjoiMjIyLjIyMi4yMjIuMjIiLCJuYW1lIjoiYWJlbGkiLCJvaWQiOiIwMjIyM2I2Yi1hYTFkLTQyZDQtOWVjMC0xYjJiYjkxOTQ0MzgiLCJyaCI6IkkiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJsM19yb0lTUVUyMjJiVUxTOXlpMmswWHBxcE9pTXo1SDNaQUNvMUdlWEEiLCJ0aWQiOiJmYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkiLCJ1bmlxdWVfbmFtZSI6ImFiZWxpQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJGVnNHeFlYSTMwLVR1aWt1dVVvRkFBIiwidmVyIjoiMS4wIn0.D3H6pMUtQnoJAGq6AHd
```

Vea este token de la versión 1.0 en [JWT.ms](https://jwt.ms/#access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9.eyJhdWQiOiJlZjFkYTlkNC1mZjc3LTRjM2UtYTAwNS04NDBjM2Y4MzA3NDUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTUyMjIyOS8iLCJpYXQiOjE1MzcyMzMxMDYsIm5iZiI6MTUzNzIzMzEwNiwiZXhwIjoxNTM3MjM3MDA2LCJhY3IiOiIxIiwiYWlvIjoiQVhRQWkvOElBQUFBRm0rRS9RVEcrZ0ZuVnhMaldkdzhLKzYxQUdyU091TU1GNmViYU1qN1hPM0libUQzZkdtck95RCtOdlp5R24yVmFUL2tES1h3NE1JaHJnR1ZxNkJuOHdMWG9UMUxrSVorRnpRVmtKUFBMUU9WNEtjWHFTbENWUERTL0RpQ0RnRTIyMlRJbU12V05hRU1hVU9Uc0lHdlRRPT0iLCJhbXIiOlsid2lhIl0sImFwcGlkIjoiNzVkYmU3N2YtMTBhMy00ZTU5LTg1ZmQtOGMxMjc1NDRmMTdjIiwiYXBwaWRhY3IiOiIwIiwiZW1haWwiOiJBYmVMaUBtaWNyb3NvZnQuY29tIiwiZmFtaWx5X25hbWUiOiJMaW5jb2xuIiwiZ2l2ZW5fbmFtZSI6IkFiZSAoTVNGVCkiLCJpZHAiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMjIyNDcvIiwiaXBhZGRyIjoiMjIyLjIyMi4yMjIuMjIiLCJuYW1lIjoiYWJlbGkiLCJvaWQiOiIwMjIyM2I2Yi1hYTFkLTQyZDQtOWVjMC0xYjJiYjkxOTQ0MzgiLCJyaCI6IkkiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJsM19yb0lTUVUyMjJiVUxTOXlpMmswWHBxcE9pTXo1SDNaQUNvMUdlWEEiLCJ0aWQiOiJmYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkiLCJ1bmlxdWVfbmFtZSI6ImFiZWxpQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJGVnNHeFlYSTMwLVR1aWt1dVVvRkFBIiwidmVyIjoiMS4wIn0.D3H6pMUtQnoJAGq6AHd).

#### <a name="v20"></a>v2.0

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9.eyJhdWQiOiI2ZTc0MTcyYi1iZTU2LTQ4NDMtOWZmNC1lNjZhMzliYjEyZTMiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3L3YyLjAiLCJpYXQiOjE1MzcyMzEwNDgsIm5iZiI6MTUzNzIzMTA0OCwiZXhwIjoxNTM3MjM0OTQ4LCJhaW8iOiJBWFFBaS84SUFBQUF0QWFaTG8zQ2hNaWY2S09udHRSQjdlQnE0L0RjY1F6amNKR3hQWXkvQzNqRGFOR3hYZDZ3TklJVkdSZ2hOUm53SjFsT2NBbk5aY2p2a295ckZ4Q3R0djMzMTQwUmlvT0ZKNGJDQ0dWdW9DYWcxdU9UVDIyMjIyZ0h3TFBZUS91Zjc5UVgrMEtJaWpkcm1wNjlSY3R6bVE9PSIsImF6cCI6IjZlNzQxNzJiLWJlNTYtNDg0My05ZmY0LWU2NmEzOWJiMTJlMyIsImF6cGFjciI6IjAiLCJuYW1lIjoiQWJlIExpbmNvbG4iLCJvaWQiOiI2OTAyMjJiZS1mZjFhLTRkNTYtYWJkMS03ZTRmN2QzOGU0NzQiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJhYmVsaUBtaWNyb3NvZnQuY29tIiwicmgiOiJJIiwic2NwIjoiYWNjZXNzX2FzX3VzZXIiLCJzdWIiOiJIS1pwZmFIeVdhZGVPb3VZbGl0anJJLUtmZlRtMjIyWDVyclYzeERxZktRIiwidGlkIjoiNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3IiwidXRpIjoiZnFpQnFYTFBqMGVRYTgyUy1JWUZBQSIsInZlciI6IjIuMCJ9.pj4N-w_3Us9DrBLfpCt
```

Vea este token de la versión 2.0 en [JWT.ms](https://jwt.ms/#access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9.eyJhdWQiOiI2ZTc0MTcyYi1iZTU2LTQ4NDMtOWZmNC1lNjZhMzliYjEyZTMiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3L3YyLjAiLCJpYXQiOjE1MzcyMzEwNDgsIm5iZiI6MTUzNzIzMTA0OCwiZXhwIjoxNTM3MjM0OTQ4LCJhaW8iOiJBWFFBaS84SUFBQUF0QWFaTG8zQ2hNaWY2S09udHRSQjdlQnE0L0RjY1F6amNKR3hQWXkvQzNqRGFOR3hYZDZ3TklJVkdSZ2hOUm53SjFsT2NBbk5aY2p2a295ckZ4Q3R0djMzMTQwUmlvT0ZKNGJDQ0dWdW9DYWcxdU9UVDIyMjIyZ0h3TFBZUS91Zjc5UVgrMEtJaWpkcm1wNjlSY3R6bVE9PSIsImF6cCI6IjZlNzQxNzJiLWJlNTYtNDg0My05ZmY0LWU2NmEzOWJiMTJlMyIsImF6cGFjciI6IjAiLCJuYW1lIjoiQWJlIExpbmNvbG4iLCJvaWQiOiI2OTAyMjJiZS1mZjFhLTRkNTYtYWJkMS03ZTRmN2QzOGU0NzQiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJhYmVsaUBtaWNyb3NvZnQuY29tIiwicmgiOiJJIiwic2NwIjoiYWNjZXNzX2FzX3VzZXIiLCJzdWIiOiJIS1pwZmFIeVdhZGVPb3VZbGl0anJJLUtmZlRtMjIyWDVyclYzeERxZktRIiwidGlkIjoiNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3IiwidXRpIjoiZnFpQnFYTFBqMGVRYTgyUy1JWUZBQSIsInZlciI6IjIuMCJ9.pj4N-w_3Us9DrBLfpCt).

## <a name="claims-in-access-tokens"></a>Notificaciones de tokens de acceso

Los JWT (JSON Web Token) se dividen en tres partes:

* **Encabezado**: proporciona información sobre cómo [validar el token](#validating-tokens) incluida la información sobre el tipo de token y cómo fue firmado.
* **Carga**: contiene todos los datos importantes sobre el usuario o la aplicación que está intentando llamar a su servicio.
* **Firma**: es la materia prima utilizada para validar el token.

Cada fragmento se separa por un punto (`.`) y se codifica por separado en Base64.

Las notificaciones están presentes solo si existe un valor que las rellene. La aplicación no debería depender de la presencia de una notificación. Los ejemplos incluyen `pwd_exp` (no todos los inquilinos requieren que las contraseñas expiren) o `family_name` (los flujos de [credenciales de cliente](v2-oauth2-client-creds-grant-flow.md) representan a las aplicaciones que no tienen nombres). Las notificaciones utilizadas para la validación de los token de acceso siempre estarán presentes.

Algunas notificaciones se utilizan para ayudar a Azure AD a asegurar los tokens en caso de reutilización. En la descripción se indica que no están destinados al consumo público como "opacos". Estas notificaciones pueden o no aparecer en un token, y pueden agregarse otras nuevas sin previo aviso.

### <a name="header-claims"></a>Notificaciones de encabezado

|Notificación | Formato | Descripción |
|--------|--------|-------------|
| `typ` | Cadena: siempre "JWT" | Indica que el token es un token JWT.|
| `nonce` | String | Un identificador único utilizado para proteger contra ataques de repetición de token. El recurso puede registrar este valor para protegerse contra las repeticiones. |
| `alg` | String | Indica el algoritmo que se usó para firmar el token, por ejemplo, "RS256". |
| `kid` | String | Especifica la huella digital de la clave pública que se utiliza para firmar este token. Se emite en ambos tokens de acceso de las versiones 1.0 y 2.0. |
| `x5t` | String | Funciona igual (en uso y valor) que `kid`. `x5t` es una notificación heredada emitida solo en los tokens de acceso de la versión 1.0 para fines de compatibilidad. |

### <a name="payload-claims"></a>Notificaciones de carga

| Notificación | Formato | Descripción |
|-----|--------|-------------|
| `aud` | Cadena, un URI de identificador de aplicación o GUID | Identifica al destinatario previsto del token, su público.  La API debe validar este valor y rechazar el token si el valor no coincide. En los tokens v.20, siempre es el id. de cliente de la API, mientras que en los tokens v.1.0 puede ser el id. de cliente o el URI de recurso que se usa en la solicitud, en función de cómo el cliente solicitó el token.|
| `iss` | Cadena, un identificador URI de STS | Identifica el servicio de token de seguridad (STS) que construye y devuelve el token, así como el inquilino de Azure AD en el que se autenticó al usuario. Si el token emitido es un token v2.0 (consulte la notificación `ver`), el URI finalizará en `/v2.0`. El GUID que indica que el usuario es un usuario consumidor desde una cuenta de Microsoft es `9188040d-6c67-4c5b-b112-36a304b66dad`. La aplicación puede usar la parte del GUID de la notificación para restringir el conjunto de inquilinos que pueden iniciar sesión en la aplicación, si corresponde. |
|`idp`| Cadena, normalmente un identificador URI de STS | Registra el proveedor de identidades que autenticó al firmante del token. Este valor es idéntico al valor de la notificación del emisor, a menos que la cuenta de usuario no esté en el mismo inquilino que el emisor: los invitados, por ejemplo. Si la notificación no está presente, significa que el valor de `iss` se puede usar en su lugar.  Para las cuentas personales que se usan en un contexto de la organización (por ejemplo, una cuenta personal invitada a un inquilino de Azure AD), la notificación `idp` puede ser "live.com" o un identificador URI de STS que contenga al inquilino de la cuenta Microsoft `9188040d-6c67-4c5b-b112-36a304b66dad`. |
| `iat` | entero, una marca de tiempo de UNIX | La notificación "iat" (emitido a las) indica cuándo se produjo la autenticación de este token. |
| `nbf` | entero, una marca de tiempo de UNIX | La notificación "nbf" (no antes de) identifica la hora antes de la cual no debe ser aceptado el token JWT para su procesamiento. |
| `exp` | entero, una marca de tiempo de UNIX | La notificación "exp" (fecha de expiración) identifica la hora de expiración en la que o después de la que el token JWT no debe ser aceptado para su procesamiento. Es importante tener en cuenta que un recurso puede rechazar el token antes de esta hora, como cuando es necesario un cambio en la autenticación o se ha detectado una revocación del token. |
| `aio` | Cadena opaca | Una notificación interna usada por Azure AD para registrar los datos para la reutilización de tokens. Los recursos no deben usar esta notificación. |
| `acr` | Cadena, un 0 o un 1 | Solo está presente en los tokens de la versión 1.0. Notificación " Clase de contexto de autenticación". Un valor de "0" indica que la autenticación del usuario final no ha cumplido los requisitos de ISO/IEC 29115. |
| `amr` | Matriz JSON de cadenas | Solo está presente en los tokens de la versión 1.0. Identifica cómo se autenticó el firmante del token. Consulte [la sección de notificaciones de amr](#the-amr-claim) para más información. |
| `appid` | Cadena, un identificador GUID | Solo está presente en los tokens de la versión 1.0. El identificador de aplicación del cliente mediante el token. La aplicación puede actuar por sí misma o en nombre de un usuario. Normalmente, el id. de aplicación representa un objeto de aplicación, pero también puede representar un objeto de entidad de servicio en Azure AD. |
| `azp` | Cadena, un identificador GUID | Solo está presente en los tokens de la versión 2.0 y es un reemplazo de `appid`. El identificador de aplicación del cliente mediante el token. La aplicación puede actuar por sí misma o en nombre de un usuario. Normalmente, el id. de aplicación representa un objeto de aplicación, pero también puede representar un objeto de entidad de servicio en Azure AD. |
| `appidacr` | 0, 1 o 2 | Solo está presente en los tokens de la versión 1.0. Indica cómo se autenticó el cliente. Para un cliente público, el valor es 0. Si se usan el identificador y el secreto de cliente, el valor es 1. Si se usa un certificado de cliente para la autenticación, el valor es 2. |
| `azpacr` | 0, 1 o 2 | Solo está presente en los tokens de la versión 2.0 y es un reemplazo de `appidacr`. Indica cómo se autenticó el cliente. Para un cliente público, el valor es 0. Si se usan el identificador y el secreto de cliente, el valor es 1. Si se usa un certificado de cliente para la autenticación, el valor es 2. |
| `preferred_username` | String | El nombre de usuario principal que representa al usuario. Puede ser una dirección de correo electrónico, un número de teléfono o un nombre de usuario genérico sin un formato especificado. Su valor es mutable y podría cambiar en el tiempo. Puesto que es mutable, este valor no debe usarse para tomar decisiones de autorización.  Sin embargo, se puede usar para las sugerencias de nombre de usuario y en la interfaz de usuario legible como nombre de usuario. El ámbito `profile` es necesario para recibir esta notificación. Presente solo en los tokens v2.0. |
| `name` | String | Proporciona un valor en lenguaje natural que identifica al firmante del token. No se asegura que el valor sea único, es mutable y está diseñado para usarse solo con fines de visualización. El ámbito `profile` es necesario para recibir esta notificación. |
| `scp` | Cadena, una lista de ámbitos separada por espacios. | El conjunto de ámbitos expuestos por la aplicación para los cuales la aplicación cliente ha solicitado (y recibido) consentimiento. Su aplicación debe comprobar que estos ámbitos son válidos y están expuestos por la aplicación, y tomar decisiones de autorización basadas en el valor de estos ámbitos. Solo se incluye para los [tokens de usuario](#user-and-application-tokens). |
| `roles` | Matriz de cadenas, una lista de permisos | El conjunto de permisos expuestos por la aplicación para la que la aplicación o el usuario solicitante ha recibido permiso para llamar. Para los [tokens de aplicaciones](#user-and-application-tokens), esta acción se usa durante el flujo de credenciales de cliente ([v1.0](../azuread-dev/v1-oauth2-client-creds-grant-flow.md) y [v2.0](v2-oauth2-client-creds-grant-flow.md)) en lugar de los ámbitos de usuario.  Para los [tokens de usuario](#user-and-application-tokens) se rellena con los roles a los que se ha asignado el usuario en la aplicación de destino. |
| `wids` | Matriz de GUID [RoleTemplateID](../roles/permissions-reference.md#all-roles) | Indica los roles de todos los inquilinos asignados a este usuario, de la sección de roles presentes en [Roles integrados de Azure AD](../roles/permissions-reference.md#all-roles).  Esta notificación se configura por aplicación, a través de la propiedad `groupMembershipClaims` del [manifiesto de aplicación](reference-app-manifest.md).  Es necesario establecerla en "All" o "DirectoryRole".  Es posible que no esté presente en los tokens obtenidos a través del flujo implícito por motivos de longitud del token. |
| `groups` | Matriz JSON de identificadores GUID | Proporciona identificadores de objeto que representan la pertenencia al grupo del firmante. Estos valores son únicos (vea el id. de objeto) y se pueden usar de forma segura para administrar el acceso, por ejemplo, para exigir autorización para tener acceso a un recurso. Los grupos incluidos en la notificación de grupos se configuran por aplicación mediante la propiedad `groupMembershipClaims` del [manifiesto de aplicación](reference-app-manifest.md). Un valor NULL excluirá todos los grupos, un valor de "SecurityGroup" incluirá únicamente la pertenencia a grupos de seguridad de Active Directory y un valor de "All" incluirá grupos de seguridad y listas de distribución de Microsoft 365. <br><br>Consulte la notificación `hasgroups` que aparece a continuación para más información sobre el uso de la notificación `groups` con la concesión implícita. <br>Para los demás flujos, si el número de grupos en los que el usuario está supera un límite (150 para SAML, 200 para JWT), se agregará una notificación de uso por encima del límite a los orígenes de notificaciones que apuntan al punto de conexión de Microsoft Graph que contiene la lista de grupos del usuario. |
| `hasgroups` | Boolean | Si está presente, siempre es `true`, lo cual indica que el usuario está en al menos un grupo. Se usa en lugar de la notificación `groups` para métodos JWT en flujos de concesión implícita si las notificaciones completas de los grupos amplían el fragmento URI por encima de los límites de longitud de la URL (actualmente 6 o más grupos). Indica que el cliente debe utilizar Microsoft Graph API para determinar los grupos del usuario (`https://graph.microsoft.com/v1.0/users/{userID}/getMemberObjects`). |
| `groups:src1` | Objeto JSON | Para las solicitudes de tokens que no tienen limitación de longitud (consulte `hasgroups` descrito anteriormente) pero que todavía son demasiado grandes para el token, se incluirá un enlace a la lista completa de grupos del usuario. Para métodos JWT como una notificación distribuida, para SAML como una nueva notificación en lugar de la notificación `groups`. <br><br>**Valor de JWT de ejemplo**: <br> `"groups":"src1"` <br> `"_claim_sources`: `"src1" : { "endpoint" : "https://graph.microsoft.com/v1.0/users/{userID}/getMemberObjects" }` |
| `sub` | String | La entidad de seguridad sobre la que el token declara información como, por ejemplo, el usuario de una aplicación. Este valor es inmutable y no se puede reasignar ni volver a usar. Se puede usar para realizar comprobaciones de autorización de forma segura, por ejemplo, cuando el token se usa para acceder a un recurso, y se puede usar como clave en tablas de base de datos. Dado que el firmante siempre está presente en los tokens que emite Azure AD, se recomienda usar este valor en un sistema de autorización de propósito general. El asunto es, sin embargo, un identificador en pares (es único para un id. de aplicación determinado). Por lo tanto, si un usuario inicia sesión en dos aplicaciones diferentes con dos identificadores de cliente diferente, esas aplicaciones recibirán dos valores diferentes para la notificación de asunto. Esto puede ser o no deseable dependiendo de los requisitos de arquitectura y privacidad. Vea también la notificación `oid` (que sigue siendo la misma en las todas aplicaciones en un inquilino). |
| `oid` | Cadena, un identificador GUID | El identificador inmutable de la "entidad de seguridad" de la solicitud: el usuario o la entidad de seguridad de servicio cuya identidad se ha comprobado.  En tokens de id. y tokens de aplicación y usuario, es el id. de objeto del usuario.  En los tokens de solo aplicación, es el id. de objeto de la entidad de servicio de llamada. También se puede usar para realizar comprobaciones de autorización de forma segura y como clave en tablas de base de datos. Este id. identifica de forma única la entidad de seguridad entre aplicaciones: dos aplicaciones diferentes que inician sesión con el mismo usuario recibirán el mismo valor en la notificación `oid`. Por tanto, `oid` puede usarse al realizar consultas en Microsoft Online Services, como Microsoft Graph. Microsoft Graph devuelve este identificador como la propiedad `id` de una [cuenta de usuario](/graph/api/resources/user) determinada. Dado que la notificación `oid` permite que varias aplicaciones pongan en correlación las entidades de seguridad, se requiere el ámbito `profile` para recibir esta notificación para usuarios. Tenga en cuenta que si un usuario existe en varios inquilinos, el usuario contendrá un identificador de objeto distinto en cada inquilino, se consideran cuentas diferentes, incluso si el usuario inicia sesión en todas las cuentas con las mismas credenciales. |
| `tid` | Cadena, un identificador GUID | Representa el inquilino de Azure AD de donde proviene el usuario. En el caso de las cuentas profesionales y educativas, el GUID es el identificador del inquilino inmutable de la organización a la que pertenece el usuario. En el caso de las cuentas personales, el valor es `9188040d-6c67-4c5b-b112-36a304b66dad`. El ámbito `profile` es necesario para recibir esta notificación. |
| `unique_name` | String | Solo está presente en los tokens de la versión 1.0. Proporciona un valor en lenguaje natural que identifica al firmante del token. No se asegura que este valor sea único en un inquilino y se debe usar solo con fines de visualización. |
| `uti` | Cadena opaca | Una notificación interna que Azure usa para volver a validar los tokens. Los recursos no deben usar esta notificación. |
| `rh` | Cadena opaca | Una notificación interna que Azure usa para volver a validar los tokens. Los recursos no deben usar esta notificación. |
| `ver` | Cadena, `1.0` o `2.0` | Indica la versión del token de acceso. |

**Notificación de grupos por encima del límite**

Para garantizar que el tamaño del token no supere los límites de tamaño del encabezado HTTP, Azure AD limita el número de identificadores de objeto que se incluyen en la notificación de grupo. Si un usuario es miembro de más grupos que el límite de uso por encima del límite (150 para los tokens SAML, 200 para los tokens JWT y solo 6 si se emiten a través del flujo implícito), Azure AD no emite la notificaciones de grupos en el token. En su lugar, incluye una demanda de uso por encima del límite en el token que indica a la aplicación que consulte la Microsoft Graph API para recuperar la pertenencia a grupos del usuario.

```JSON
{
  ...
  "_claim_names": {
   "groups": "src1"
    },
    {
  "_claim_sources": {
    "src1": {
        "endpoint":"[Url to get this user's group membership from]"
        }
       }
     }
  ...
}
```

Puede usar `BulkCreateGroups.ps1`, que se proporciona en la carpeta [scripts de creación de aplicaciones](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/5-WebApp-AuthZ/5-2-Groups/AppCreationScripts), para ayudar a probar los escenarios de uso por encima del límite.

#### <a name="v10-basic-claims"></a>Notificaciones básicas de la versión 1.0

Las siguientes notificaciones se incluyen en los tokens de la versión 1.0 si corresponde, pero no se incluyen en los tokens de la versión 2.0 de forma predeterminada. Si usa la versión 2.0 y necesita una de estas notificaciones, solicítelas mediante [notificaciones opcionales](active-directory-optional-claims.md).

| Notificación | Formato | Descripción |
|-----|--------|-------------|
| `ipaddr`| String | La dirección IP desde la que el usuario se autenticó. |
| `onprem_sid`| Cadena, [en formato de GUID](/windows/desktop/SecAuthZ/sid-components) | En los casos en los que el usuario tiene una autenticación local, esta notificación proporciona el SID. Puede utilizar `onprem_sid` para la autorización en aplicaciones heredadas.|
| `pwd_exp`| entero, una marca de tiempo de UNIX | Indica cuándo expira la contraseña del usuario. |
| `pwd_url`| String | Una dirección URL a donde se envían los usuarios para restablecer la contraseña. |
| `in_corp`| boolean | Indica si el cliente ha iniciado sesión desde la red corporativa. En caso contrario, la notificación no se incluye. |
| `nickname`| String | Nombre adicional del usuario, aparte del nombre y del apellido.|
| `family_name` | String | Proporciona el apellido del usuario según está definido en el objeto de usuario. |
| `given_name` | String | Proporciona el nombre de pila o "dado" del usuario, tal como se establece en el objeto de usuario. |
| `upn` | String | Nombre de usuario del usuario. Puede ser un número de teléfono, una dirección de correo electrónico o una cadena sin formato. Solo debe utilizarse para fines de visualización y para proporcionar sugerencias de nombre de usuario en escenarios de reautenticación. |

#### <a name="the-amr-claim"></a>La notificación `amr`

Las identidades de Microsoft pueden autenticarse de diversas maneras, que pueden ser pertinentes para la aplicación. La notificación `amr` es una matriz que puede contener varios elementos, como `["mfa", "rsa", "pwd"]`, para una autenticación que utiliza tanto una contraseña como la aplicación Autenticator.

| Value | Descripción |
|-----|-------------|
| `pwd` | Autenticación de contraseña, ya sea la contraseña de un usuario de Microsoft o el secreto de cliente de una aplicación. |
| `rsa` | La autenticación se basaba en la prueba de una clave RSA, por ejemplo con la [aplicación Microsoft Authenticator](https://aka.ms/AA2kvvu). Esto incluye si la autenticación la ha realizado un token JWT autofirmado con un certificado X509 de propiedad del servicio. |
| `otp` | Código de acceso de un solo uso que utiliza un correo electrónico o un mensaje de texto. |
| `fed` | Se ha utilizado una aserción de autenticación federada (por ejemplo, JWT o SAML). |
| `wia` | Autenticación integrada de Windows |
| `mfa` | Se utilizó la [autenticación multifactor](../authentication/concept-mfa-howitworks.md). Cuando esto esté presente, también se incluirán los otros métodos de autenticación. |
| `ngcmfa` | Equivalente a `mfa`, que se utiliza para el aprovisionamiento de ciertos tipos de credenciales avanzadas. |
| `wiaormfa`| El usuario utiliza una credencial MFA o Windows para autenticar. |
| `none` | No se realizó ninguna autenticación. |

## <a name="access-token-lifetime"></a>Vigencia del token de acceso

La duración predeterminada de un token de acceso varía en función de la aplicación cliente que está solicitando el token. Por ejemplo, los clientes compatibles con la evaluación de acceso continuo (CAE) que negocian sesiones compatibles con CAE, tendrán una vigencia de token de larga duración (hasta 28 horas).  Cuando el token de acceso expira, el cliente debe usar el token de actualización para adquirir un nuevo token de actualización (normalmente en modo silencioso) y un token de acceso.

Puede ajustar la vigencia de un token de acceso para controlar la frecuencia con la que la aplicación cliente expira la sesión de la aplicación y requerirá que el usuario se vuelva a autenticar (de forma silenciosa o interactiva). Para más información, consulte [Vigencia de tokens configurable](active-directory-configurable-token-lifetimes.md).

## <a name="validating-tokens"></a>Validación de los tokens

No todas las aplicaciones deben validar los tokens. Las aplicaciones deben validar un token solo en escenarios específicos:

* Las [API web](quickstart-configure-app-expose-web-apis.md) deben validar los tokens de acceso que les envía un cliente.  Solo deben aceptar tokens que contengan su notificación `aud`.
* Las aplicaciones web confidenciales como ASP.NET Core deben validar los tokens de identificador que se les envían a través del explorador del usuario en el flujo híbrido, antes de permitir el acceso a los datos de un usuario o de establecer una sesión.

Si no se aplica ninguno de los escenarios anteriores, la aplicación no se beneficiará de la validación del token y puede suponer un riesgo de seguridad y confiabilidad si se toman decisiones en función de la validez del token.  Los clientes públicos como aplicaciones nativas o SPA no se benefician de la validación de los tokens: la aplicación se comunica directamente con el IDP, por lo que la protección SSL garantiza que los tokens son válidos.

Las API y aplicaciones web solo deben validar los tokens que tengan una notificación `aud` que coincida con su aplicación; otros recursos pueden tener reglas de validación de tokens personalizadas. Por ejemplo, los tokens para Microsoft Graph no se validarán según estas reglas debido a su formato propietario. La validación y aceptación de tokens destinados a otro recurso es un ejemplo del problema de [intermediario confundido](https://cwe.mitre.org/data/definitions/441.html).

Si la aplicación necesita validar un id_token o un access_token según lo anterior, la aplicación primero debe validar la firma y el emisor del token con respecto a los valores del documento de detección de OpenID. Por ejemplo, la versión independiente del inquilino del documento se encuentra en [https://login.microsoftonline.com/common/.well-known/openid-configuration](https://login.microsoftonline.com/common/.well-known/openid-configuration).

La información siguiente se proporciona para aquellos que desean entender el proceso subyacente. El middleware de Azure AD tiene funciones integradas para validar los tokens de acceso, y usted puede explorar nuestros [ejemplos](sample-v2-code.md) para buscar uno en el idioma de su elección. También hay varias bibliotecas de código abierto de terceros para la validación de JWT; hay al menos una opción para casi cualquier plataforma e idioma. Para más información sobre los ejemplos de código y las bibliotecas de autenticación de Azure AD, consulte las [bibliotecas de autenticación](reference-v2-libraries.md).

### <a name="validating-the-signature"></a>Validación de la firma

Un JWT contiene tres segmentos, que están separados por el carácter `.` . El primer segmento se conoce como el **encabezado**, el segundo como el **cuerpo** y el tercero como la **firma**. El segmento de firma se puede utilizar para validar la autenticidad del token con el fin de que la aplicación pueda confiar en él.

Los tokens emitidos por Azure AD se firman con algoritmos de cifrado asimétrico estándar del sector, como RS256. El encabezado de JWT contiene información acerca de clave y el método de cifrado utilizados para firmar el token:

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "x5t": "iBjL1Rcqzhiy4fpxIxdZqohM2Yk",
  "kid": "iBjL1Rcqzhiy4fpxIxdZqohM2Yk"
}
```

La notificación `alg` indica el algoritmo que se usó para firmar el token, mientras que la notificación `kid` indica la clave pública concreta que se usó para validar el token.

En cualquier momento, Azure AD puede firmar un id_token mediante cualquiera de un determinado conjunto de pares de claves pública y privada. Azure AD rota los posibles conjuntos de claves de forma periódica, por lo que su aplicación debería estar escrita para manejar esos cambios de clave automáticamente. Una frecuencia razonable para comprobar las actualizaciones de las claves públicas usadas por Azure AD es cada 24 horas.

Puede adquirir los datos de las claves de firmas necesarios para validar la firma utilizando el [documento de metadatos de OpenID Connect](v2-protocols-oidc.md#fetch-the-openid-connect-metadata-document), ubicado en:

```
https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration
```

> [!TIP]
> Pruebe esta [dirección URL](https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration) en un explorador.

Este documento de metadatos:

* Es un objeto JSON que contiene varias fragmentos de información útiles, como la ubicación de los diferentes puntos de conexión necesarios para realizar la autenticación de OpenID Connect.
* Incluye un `jwks_uri`, que ofrece la ubicación del conjunto de claves públicas que se utilizan para firmar los tokens. La clave web JSON (JWK)que se encuentra en `jwks_uri` contiene toda la información de clave pública en uso en ese momento determinado.  El formato JWK se describe en [RFC 7517](https://tools.ietf.org/html/rfc7517).  La aplicación puede usar la notificación `kid` en el encabezado de JWT para seleccionar la clave pública que se ha usado en este documento para firmar un determinado token. Después, puede realizar la validación de la firma mediante la clave pública correcta y el algoritmo indicado.

> [!NOTE]
> Le recomendamos que utilice la notificación `kid` para validar su token. Aunque los tokens v1.0 contienen las notificaciones `x5t` y `kid`, los tokens v2.0 solo contienen la notificación `kid`.

La realización de la validación de la firma queda fuera del ámbito de este documento, pero hay muchas bibliotecas de código abierto disponibles para ayudarle a hacerlo si es necesario.  Sin embargo, la Plataforma de identidad de Microsoft tiene una extensión de firma de tokens para los estándares: las claves de firma personalizadas.

Si la aplicación tiene claves de firma personalizadas como resultado de usar la característica de [asignación de notificaciones](active-directory-claims-mapping.md), debe anexar un parámetro de consulta `appid` que contenga el identificador de la aplicación con el fin de obtener un elemento `jwks_uri` que apunte a la información de la clave de firma de la aplicación, que debe usarse la para la validación. Por ejemplo: `https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration?appid=6731de76-14a6-49ae-97bc-6eba6914391e` contiene el elemento `jwks_uri` de `https://login.microsoftonline.com/{tenant}/discovery/keys?appid=6731de76-14a6-49ae-97bc-6eba6914391e`.

### <a name="claims-based-authorization"></a>Autorización basada en notificaciones

La lógica de negocio de la aplicación dictará este paso; a continuación se presentan algunos métodos de autorización comunes.

* Marque la notificación `scp` o `roles` para comprobar que todos los ámbitos presentes coinciden con los expuestos por la API, y permita que el cliente realice la acción solicitada.
* Asegúrese de que el cliente que llama puede llamar a la API mediante la notificación `appid`.
* Valide el estado de autenticación del cliente que llama mediante `appidacr` (no debería ser 0 si los clientes públicos no pueden llamar a la API).
* Compare con una lista de notificaciones `nonce` anteriores para comprobar que el token no se está repitiendo.
* Compruebe que `tid` coincide con un inquilino al que se le permite llamar a la API.
* Utilice la notificación `amr` para comprobar que el usuario ha realizado la autenticación multifactor. Esto se debe aplicar con [acceso condicional](../conditional-access/overview.md).
* Si ha solicitado las notificaciones `roles` o `groups` en el token de acceso, compruebe que el usuario está en el grupo al que se permite realizar esta acción.
  * Para los tokens recuperados utilizando el flujo implícito, es probable que necesite consultar [Microsoft Graph](https://developer.microsoft.com/graph/) para estos datos, ya que a menudo son demasiado grandes para adaptarse al token.

## <a name="user-and-application-tokens"></a>Tokens de usuario y de aplicación

La aplicación puede recibir tokens para usuario (el flujo habitual) o directamente desde una aplicación (mediante el [flujo de credenciales de cliente](../azuread-dev/v1-oauth2-client-creds-grant-flow.md)). Estos tokens de solo aplicación indican que esta llamada proviene de una aplicación y no tiene un usuario que la respalde. Estos tokens se usan en gran medida de la misma manera:

* Use `roles` para ver los permisos que se concedieron al firmante del token.
* Utilice `oid` o `sub` para validar que la entidad de servicio que realiza la llamada es la que se espera.

Si la aplicación necesita distinguir entre tokens de acceso de solo aplicación y tokens de acceso para los usuarios, use la [notificación opcional](active-directory-optional-claims.md) `idtyp`.  Al agregar la notificación `idtyp` al campo `accessToken` y comprobar el valor `app`, puede detectar tokens de acceso de solo aplicación.  Los tokens de identificador y los tokens de acceso para los usuarios no tendrán incluida la notificación `idtyp`.

## <a name="token-revocation"></a>Revocación de tokens

Los tokens de actualización se pueden invalidar o revocar en cualquier momento por varios motivos. Estos se dividen en dos categorías principales: tiempos de espera y revocaciones.

### <a name="token-timeouts"></a>Tiempos de espera de token

Con la [configuración de vigencia de los tokens](active-directory-configurable-token-lifetimes.md), se puede modificar la duración de los tokens de actualización.  Es normal y habitual que algunos tokens no se usen (por ejemplo, si el usuario no abre la aplicación durante tres meses) y, por lo tanto, expiran.  Las aplicaciones se pueden encontrar escenarios en los que el servidor de inicio de sesión rechaza un token de actualización debido a su antigüedad.

* MaxInactiveTime: si el token de actualización no se ha utilizado en el tiempo determinado por MaxInactiveTime, el token de actualización ya no será válido.
* MaxSessionAge: si MaxAgeSessionMultiFactor o MaxAgeSessionSingleFactor se han establecido en un valor distinto de su valor predeterminado (Hasta que se revoca), la reautenticación será obligatoria después de que transcurra el tiempo establecido en el MaxAgeSession*.
* Ejemplos:
  * El inquilino tiene un valor MaxInactiveTime de cinco días y el usuario se fue de vacaciones durante una semana, por lo que Azure AD no ha visto ninguna nueva solicitud de token de usuario en 7 días. La próxima vez que el usuario solicite un nuevo token, encontrará que su token de actualización se ha revocado, así que tendrá que escribir sus credenciales de nuevo.
  * Una aplicación sensible tiene un valor MaxAgeSessionSingleFactor de un día. Si un usuario inicia sesión el lunes y el martes (una vez que hayan transcurrido 25 horas), se le pedirá que se autentique de nuevo.

### <a name="revocation"></a>Revocación

El servidor puede revocar los tokens de actualización debido a un cambio en las credenciales o a una acción de administración o uso.  Los tokens de actualización se dividen en dos clases: emitidos para clientes confidenciales (la columna situada más a la derecha) y emitidos para clientes públicos (el resto de columnas).

| Change | Cookie basada en contraseñas | Token basado en contraseñas | Cookie no basada en contraseñas | Token no basado en contraseñas | Token de cliente confidencial |
|---|-----------------------|----------------------|---------------------------|--------------------------|---------------------------|
| La contraseña expira | Permanece activa | Permanece activa | Permanece activa | Permanece activa | Permanece activa |
| Contraseña cambiada por el usuario | Revocada | Revocada | Permanece activa | Permanece activa | Permanece activa |
| Usuario realiza SSPR | Revocada | Revocada | Permanece activa | Permanece activa | Permanece activa |
| Administrador restablece la contraseña | Revocada | Revocada | Permanece activa | Permanece activa | Permanece activa |
| Usuario revoca sus tokens de actualización [a través de PowerShell](/powershell/module/azuread/revoke-azureadsignedinuserallrefreshtoken) | Revocada | Revocada | Revocada | Revocada | Revocada |
| El administrador revoca todos los tokens de actualización de un usuario [a través de PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken) | Revocada | Revocada |Revocada | Revocada | Revocada |
| Cierre de sesión único ([v 1.0](../azuread-dev/v1-protocols-openid-connect-code.md#single-sign-out) y [v 2.0](v2-protocols-oidc.md#single-sign-out)) en la web | Revocada | Permanece activa | Revocada | Permanece activa | Permanece activa |

#### <a name="non-password-based"></a>No basado en contraseña

Un *inicio de sesión no basado en contraseña* es aquel en el que el usuario no escribe una contraseña para llevarlo a cabo. Entre los ejemplos de inicio de sesión no basado en contraseña están:

- Uso del reconocimiento facial en Windows Hello
- Clave FIDO2
- SMS
- Voz
- PIN

Consulte [Tokens de actualización principales](../devices/concept-primary-refresh-token.md) para obtener más detalles sobre estos tokens.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [`id_tokens` en Azure AD](id-tokens.md).
* Obtenga información sobre permisos y consentimiento ([v1.0](../azuread-dev/v1-permissions-consent.md) y [v2.0](v2-permissions-and-consent.md)).
