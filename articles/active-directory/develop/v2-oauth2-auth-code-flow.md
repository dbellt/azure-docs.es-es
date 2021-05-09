---
title: Plataforma de identidad de Microsoft y flujo de código de autorización de OAuth 2.0 1 | Azure
titleSuffix: Microsoft identity platform
description: Compile aplicaciones web mediante la implementación de la plataforma de identidad de Microsoft del protocolo de autenticación OAuth 2.0.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: d2f4fca7f6b5fe7c6a894f4be3b27788799b422e
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108064006"
---
# <a name="microsoft-identity-platform-and-oauth-20-authorization-code-flow"></a>Plataforma de identidad y flujo de código de autorización de OAuth 2.0

La concesión de un código de autorización de OAuth 2.0 se puede usar en aplicaciones que se instalan en un dispositivo para obtener acceso a recursos protegidos, como las API web. Mediante la implementación de la Plataforma de identidad de Microsoft de OAuth 2.0, puede agregar el inicio de sesión y acceso a API a las aplicaciones móviles y de escritorio.

En este artículo se describe cómo programar directamente con el protocolo de la aplicación mediante cualquier lenguaje.  Cuando sea posible, se recomienda usar las bibliotecas de autenticación de Microsoft (MSAL) admitidas, en lugar de [adquirir tokens y API web protegidas por llamadas](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Además, eche un vistazo a las [aplicaciones de ejemplo que usan MSAL](sample-v2-code.md).

El flujo de código de autorización de OAuth 2.0 se describe en la [sección 4.1 de la especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749). Se usa para realizar la autenticación y autorización en la mayoría de los tipos de aplicación, incluidas las [aplicaciones de página única](v2-app-types.md#single-page-apps-javascript), las [aplicaciones web](v2-app-types.md#web-apps) y las [aplicaciones instaladas de forma nativa](v2-app-types.md#mobile-and-native-apps). El flujo permite que las aplicaciones adquieran de forma segura access_tokens que se puedan usar para obtener acceso a los recursos protegidos mediante la Plataforma de identidad de Microsoft, así como tokens de actualización para obtener access_tokens adicionales, y tokens de identificador para el usuario que ha iniciado sesión.

## <a name="protocol-diagram"></a>Diagrama de protocolo

A nivel general, el flujo de autenticación completo de una aplicación tiene un aspecto similar al siguiente:

![Flujo de código de autenticación de OAuth](./media/v2-oauth2-auth-code-flow/convergence-scenarios-native.svg)

## <a name="redirect-uri-setup-required-for-single-page-apps"></a>Configuración del URI de redireccionamiento requerida para aplicaciones de página única

El flujo de código de autorización para aplicaciones de página única requiere configuración adicional.  Siga las instrucciones para [crear la aplicación de una sola página](scenario-spa-app-registration.md#redirect-uri-msaljs-20-with-auth-code-flow) para marcar correctamente el URI de redireccionamiento como habilitado para CORS. Para actualizar un URI de redireccionamiento existente para habilitar CORS, abra el editor de manifiestos y establezca el campo `type` del URI de redireccionamiento en `spa` en la sección `replyUrlsWithType`. También puede hacer clic en el URI de redireccionamiento en la sección "Web" de la pestaña Autenticación y seleccionar los URI a los que quiera migrar mediante el flujo de código de autorización.

El tipo de redireccionamiento `spa` es compatible con las versiones anteriores del flujo implícito. Las aplicaciones que usan actualmente el flujo implícito para obtener tokens pueden moverse al tipo de URI de redireccionamiento `spa` sin problemas y seguir usando el flujo implícito.

Si intenta usar el flujo de código de autorización y ve este error:

`access to XMLHttpRequest at 'https://login.microsoftonline.com/common/v2.0/oauth2/token' from origin 'yourApp.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

Luego, visite el registro de aplicaciones y actualice el URI de redirección de la aplicación al tipo `spa`.

## <a name="request-an-authorization-code"></a>Solicitud de un código de autorización

El flujo del código de autorización comienza con el cliente dirigiendo al usuario al punto de conexión `/authorize` . En esta solicitud, el cliente solicita los permisos `openid`, `offline_access` y `https://graph.microsoft.com/mail.read ` del usuario.  Algunos permisos están restringidos para los administradores; por ejemplo, la escritura de datos en el directorio de una organización mediante `Directory.ReadWrite.All`. Si la aplicación solicita acceso a uno de estos permisos desde un usuario de la organización, el usuario recibe un mensaje de error que indica que no está autorizado para dar el consentimiento a los permisos de la aplicación. Para solicitar acceso a los ámbitos restringidos para el administrador, debe solicitarlos directamente a un administrador global.  Para más información, consulte [Permisos restringidos para el administrador](v2-permissions-and-consent.md#admin-restricted-permissions).

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read%20api%3A%2F%2F
&state=12345
&code_challenge=YTFjNjI1OWYzMzA3MTI4ZDY2Njg5M2RkNmVjNDE5YmEyZGRhOGYyM2IzNjdmZWFhMTQ1ODg3NDcxY2Nl
&code_challenge_method=S256
```

> [!TIP]
> Para ejecutar esta solicitud, haga clic en el vínculo siguiente. Después de iniciar sesión, el explorador se redirigirá a `https://localhost/myapp/` con un elemento `code` en la barra de direcciones.
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=query&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&state=12345" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| Parámetro    | Obligatorio/opcional | Descripción |
|--------------|-------------|--------------|
| `tenant`    | requerido    | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).  |
| `client_id`   | requerido    | El **identificador de aplicación (cliente)** que la experiencia [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación.  |
| `response_type` | requerido    | Debe incluir `code` para el flujo de código de autorización. También puede incluir `id_token` o `token` si usa el [flujo híbrido](#request-an-id-token-as-well-hybrid-flow). |
| `redirect_uri`  | requerido | El redirect_uri de su aplicación, a donde su aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los redirect_uris que registró en el portal, con la excepción de que debe estar codificado como URL. En el caso de las aplicaciones móviles y nativas, debe usar uno de los valores recomendados: `https://login.microsoftonline.com/common/oauth2/nativeclient` (para aplicaciones que usan exploradores insertados) o `http://localhost` (para aplicaciones que usan exploradores del sistema). |
| `scope`  | requerido    | Una lista separada por espacios de [ámbitos](v2-permissions-and-consent.md) que desea que el usuario consienta.  El tramo `/authorize` de la solicitud puede abarcar varios recursos, lo que permite que la aplicación obtenga el consentimiento para las diversas API web a las que quiere llamar. |
| `response_mode`   | recomendado | Especifica el método que debe usarse para enviar el token resultante de nuevo a la aplicación. Puede ser uno de los siguientes:<br/><br/>- `query`<br/>- `fragment`<br/>- `form_post`<br/><br/>`query` proporciona el código como un parámetro de cadena de consulta en el URI de redirección. Si solicita un token de identificador con el flujo implícito, no puede usar `query` según lo indicado en la [especificación de OpenID](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations). Si solicita solo el código, puede usar `query`, `fragment` o `form_post`. `form_post` ejecuta una prueba POST que contiene el código para el URI de redirección. |
| `state`                 | recomendado | Un valor incluido en la solicitud que se devolverá también en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Normalmente se usa un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El valor también puede codificar información sobre el estado del usuario en la aplicación antes de que se produzca la solicitud de autenticación, por ejemplo, la página o vista en la que estaba. |
| `prompt`  | opcional    | Indica el tipo de interacción necesaria con el usuario. Los únicos valores válidos en este momento son `login`, `none`, `consent` y `select_account`.<br/><br/>- `prompt=login` obligará al usuario a escribir sus credenciales en esa solicitud, negando el inicio de sesión único.<br/>- `prompt=none` es lo contrario, se asegurará de que al usuario no se le presenta ninguna solicitud interactiva del tipo que sea. Si la solicitud no se puede completar sin notificaciones mediante el inicio de sesión único, la Plataforma de identidad de Microsoft devolverá un error `interaction_required`.<br/>- `prompt=consent` desencadenará el cuadro de diálogo de consentimiento de OAuth después de que el usuario inicie sesión y solicitará a este que conceda permisos a la aplicación.<br/>- `prompt=select_account` interrumpirá el inicio de sesión único, lo que proporciona una experiencia de selección de cuentas en la que se enumeran todas las cuentas de la sesión o cualquier cuenta recordada, o bien una opción para usar una cuenta diferente.<br/> |
| `login_hint`  | opcional    | Puede usarse para rellenar previamente el campo de nombre de usuario y dirección de correo electrónico de la página de inicio de sesión del usuario, si sabe su nombre de usuario con antelación. A menudo las aplicaciones usarán este parámetro durante la reautenticación, dado que ya han extraído el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username`.   |
| `domain_hint`  | opcional    | Si se incluye, omitirá el proceso de detección basado en correo electrónico por el que pasa el usuario en la página de inicio de sesión, con lo que la experiencia de usuario será ligeramente más sencilla; por ejemplo, lo enviará al proveedor de identidades federado. A menudo las aplicaciones usarán este parámetro durante la reautenticación, para lo que extraen `tid` de un inicio de sesión anterior. Si el valor de la notificación `tid` es `9188040d-6c67-4c5b-b112-36a304b66dad`, debe usar `domain_hint=consumers`. De lo contrario, use `domain_hint=organizations`.  |
| `code_challenge`  | recomendado/requerido | Se usa para proteger concesiones de código de autorización a través de la clave de prueba para intercambio de códigos (PKCE). Se requiere si se incluye `code_challenge_method`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). Esto se recomienda ahora para todos los tipos de aplicaciones (tanto clientes públicos como confidenciales) y la plataforma de identidades de Microsoft lo requiere para las [aplicaciones de una sola página mediante el flujo de código de autorización](reference-third-party-cookies-spas.md). |
| `code_challenge_method` | recomendado/requerido | Método utilizado para codificar `code_verifier` para el parámetro `code_challenge`. Este *DEBERÍA* ser `S256`, pero la especificación permite utilizar `plain` si por alguna razón el cliente no admite SHA256. <br/><br/>Si se excluye, se supone que `code_challenge` es texto no cifrado si se incluye `code_challenge`. La Plataforma de identidad de Microsoft admite tanto `plain` como `S256`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). Esto es necesario para las [aplicaciones de página única que usan el flujo de código de autorización](reference-third-party-cookies-spas.md).|


En este punto, se le pedirá al usuario que escriba sus credenciales y que complete la autenticación. La plataforma de identidad de Microsoft se asegurará también de que el usuario ha dado su consentimiento a los permisos indicados en el parámetro de la consulta `scope`. Si el usuario no ha dado su consentimiento a alguno de esos permisos, se le solicitará al usuario su consentimiento para los permisos necesarios. Aquí puede encontrar información detallada sobre los [permisos, el consentimiento y las aplicaciones multiempresa](v2-permissions-and-consent.md).

Una vez que el usuario se autentica y otorga su consentimiento, la Plataforma de identidad de Microsoft devuelve una respuesta a la aplicación en el `redirect_uri` indicado mediante el método especificado en el parámetro `response_mode`.

#### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta al usar `response_mode=query` tiene el siguiente aspecto:

```HTTP
GET http://localhost?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=12345
```

| Parámetro | Descripción  |
|-----------|--------------|
| `code` | El authorization_code que solicitó la aplicación. La aplicación puede utilizar el código de autorización para solicitar un token de acceso para el recurso de destino. Los authorization_codes son de corta duración y normalmente expiran después de unos 10 minutos. |
| `state` | Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

También puede recibir un token de identificador si lo solicita y tiene habilitada la concesión implícita en el registro de aplicaciones.  A veces, esto se conoce como ["flujo híbrido"](#request-an-id-token-as-well-hybrid-flow) y se usa en marcos como ASP.NET.

#### <a name="error-response"></a>Respuesta de error

Las respuestas de error también pueden enviarse al `redirect_uri` para que la aplicación pueda controlarlas adecuadamente:

```HTTP
GET http://localhost?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| Parámetro | Descripción  |
|----------|------------------|
| `error`  | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de autorización

En la tabla siguiente se describen los distintos códigos de error que pueden devolverse en el parámetro `error` de la respuesta de error.

| Código de error  | Descripción    | Acción del cliente   |
|-------------|----------------|-----------------|
| `invalid_request` | Error de protocolo, como un parámetro obligatorio que falta. | Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que suele detectarse durante las pruebas iniciales. |
| `unauthorized_client` | La aplicación cliente no puede solicitar un código de autorización. | Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se ha agregado al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `access_denied`  | El propietario del recurso ha denegado el consentimiento.  | La aplicación cliente puede notificar al usuario que no puede continuar, salvo que este dé su consentimiento. |
| `unsupported_response_type` | El servidor de autorización no admite el tipo de respuesta de la solicitud. | Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que suele detectarse durante las pruebas iniciales. Cuando aparece en el [flujo híbrido](#request-an-id-token-as-well-hybrid-flow), indica que debe habilitar la configuración de concesión implícita del token de identificador en el registro de la aplicación cliente. |
| `server_error`  | El servidor ha detectado un error inesperado.| Vuelva a intentarlo. Estos errores pueden deberse a condiciones temporales. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a un error temporal. |
| `temporarily_unavailable`   | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. | Vuelva a intentarlo. La aplicación podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |
| `invalid_resource`  | El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o no está configurado correctamente. | Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `login_required` | Se han encontrado demasiados usuarios o no se ha encontrado ninguno | El cliente solicitó la autenticación en modo silencioso (`prompt=none`), pero no se pudo encontrar un único usuario. Esto puede significar que hay varios usuarios activos en la sesión o que no hay ninguno. Esto tiene en cuenta el inquilino elegido (por ejemplo, si hay dos cuentas de Azure AD activas y una cuenta de Microsoft, y se elige `consumers`, la autenticación silenciosa funcionará). |
| `interaction_required` | La solicitud requiere la interacción del usuario. | Es necesario realizar un paso de autenticación o consentimiento más. Vuelva a intentar realizar la solicitud sin `prompt=none`. |

### <a name="request-an-id-token-as-well-hybrid-flow"></a>Solicitar también un token de identificador (flujo híbrido)

Para saber quién es el usuario antes de canjear un código de autorización, es habitual que las aplicaciones también soliciten un token de identificador al solicitar el código de autorización. Esto se conoce como *flujo híbrido* porque combina la concesión implícita con el flujo del código de autorización. El flujo híbrido se usa normalmente en aplicaciones web que quieren representar una página para un usuario sin bloquear el canje del código, especialmente [ASP.NET](quickstart-v2-aspnet-core-webapp.md). Las aplicaciones de una sola página y las aplicaciones web tradicionales se benefician de la menor latencia en este modelo.

El flujo híbrido es el mismo que el flujo del código de autorización descrito anteriormente, pero con tres adiciones, todas ellas necesarias para solicitar un token de identificador: nuevos ámbitos, un nuevo parámetro response_type y un nuevo parámetro de consulta `nonce`.

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code%20id_token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=fragment
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fuser.read
&state=12345
&nonce=abcde
&code_challenge=YTFjNjI1OWYzMzA3MTI4ZDY2Njg5M2RkNmVjNDE5YmEyZGRhOGYyM2IzNjdmZWFhMTQ1ODg3NDcxY2Nl
&code_challenge_method=S256
```

| Parámetros actualizado | Obligatorio/opcional | Descripción |
|---------------|-------------|--------------|
|`response_type`| Obligatorio | La adición de `id_token` indica al servidor que la aplicación desea un token de identificador en la respuesta del punto de conexión `/authorize`.  |
|`scope`| Obligatorio | En el caso de los tokens de identificador, debe actualizarse para incluir los ámbitos de token de identificador `openid` y, opcionalmente, `profile` y `email`. |
|`nonce`| Obligatorio|     Valor generado por la aplicación que se incluye en la solicitud y que se incluirá en el parámetro id_token resultante como una notificación. La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Normalmente, el valor es una cadena única aleatoria que se puede usar para identificar el origen de la solicitud. |
|`response_mode`| Recomendado | Especifica el método que debe usarse para enviar el token resultante de nuevo a la aplicación. Se establece de forma predeterminada en `query` solo para un código de autorización, pero en `fragment` si la solicitud incluye un parámetro id_token `response_type`.  Sin embargo, se recomienda que las aplicaciones usen `form_post`, especialmente cuando se usa `http:/localhost` como un URI de redirección. |

El uso de `fragment` como modo de respuesta puede causar problemas para las aplicaciones web que leen el código de la redirección, ya que los exploradores no pasan el fragmento al servidor web.  En estas situaciones, las aplicaciones deben usar el modo de respuesta `form_post` para asegurarse de que todos los datos se envían al servidor. 

#### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta al usar `response_mode=fragment` tiene el siguiente aspecto:

```HTTP
GET https://login.microsoftonline.com/common/oauth2/nativeclient#
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&id_token=eYj...
&state=12345
```

| Parámetro | Descripción  |
|-----------|--------------|
| `code` | El código de autorización que la aplicación solicitó. La aplicación puede utilizar el código de autorización para solicitar un token de acceso para el recurso de destino. Los códigos de autorización son de corta duración y normalmente expiran después de unos 10 minutos. |
| `id_token` | Un token de identificador para el usuario, emitido a través de la *concesión implícita*. Contiene una notificación `c_hash` especial, que es el hash del elemento `code` en la misma solicitud. |
| `state` | Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de state de la solicitud y la respuesta sean idénticos. |

## <a name="redeem-a-code-for-an-access-token"></a>Canje de un código por un token de acceso

Todos los clientes confidenciales tienen la opción de usar secretos de cliente (secretos compartidos simétricos que genera la plataforma de identidad de Microsoft) y [credenciales de certificados](active-directory-certificate-credentials.md) (claves asimétricas que carga el desarrollador).  Para mayor seguridad, se recomienda usar las credenciales de certificados. Los clientes públicos (aplicaciones nativas y aplicaciones de página única) no deben usar secretos ni certificados al canjear un código de autorización. Siempre debe asegurarse de que los URI de redireccionamiento [sean únicos](reply-url.md#localhost-exceptions) e indiquen correctamente el tipo de aplicación. 

### <a name="request-an-access-token-with-a-client_secret"></a>Solicitud de un token de acceso con un client_secret

Ahora que ha adquirido un código de autorización y el usuario le ha concedido permiso, puede canjear el `code` por un `access_token` al recurso deseado. Puede hacer esto mediante el envío de una solicitud `POST` al punto de conexión `/token`:

```HTTP
// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&grant_type=authorization_code
&code_verifier=ThisIsntRandomButItNeedsToBe43CharactersLong 
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps. This secret needs to be URL-Encoded.
```

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman (No olvide reemplazar `code`) [![Pruebe a ejecutar esta solicitud en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://www.getpostman.com/collections/dba7e9c2e0870702dfc6)

| Parámetro  | Obligatorio/opcional | Descripción     |
|------------|-------------------|----------------|
| `tenant`   | requerido   | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).  |
| `client_id` | requerido  | El identificador de aplicación (cliente) que la página [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `scope`      | opcional   | Lista de ámbitos separados por espacios. Todos los ámbitos deben provenir de un recurso único, junto con los ámbitos de OIDC (`profile`, `openid`, `email`). Para obtener una explicación más detallada de los ámbitos, consulte [permisos, consentimiento y ámbitos](v2-permissions-and-consent.md). Se trata de una extensión de Microsoft para el flujo de código de autorización, diseñada para permitir que las aplicaciones declaren el recurso para el que quieren el token durante el canje de tokens.|
| `code`          | requerido  | El authorization_code que adquirió en el primer segmento del flujo. |
| `redirect_uri`  | requerido  | El mismo valor redirect_uri usado para adquirir el código de autorización. |
| `grant_type` | requerido   | Debe ser `authorization_code` para el flujo de código de autorización.   |
| `code_verifier` | recomendado  | El mismo valor de code_verifier que usó para obtener el valor de authorization_code. Se requiere si PKCE se utilizó en la solicitud de concesión de código de autorización. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |
| `client_secret` | requerido para las aplicaciones web confidenciales | El secreto de la aplicación que creó en el portal de registro de aplicaciones para su aplicación. No debe usar el secreto de aplicación en una aplicación nativa ni en una aplicación de página única, porque no es posible almacenar los client_secrets de manera segura en los dispositivos ni páginas web. Es necesario para aplicaciones web y las API web, que tienen la capacidad de almacenar el client_secret de manera segura en el lado del servidor.  Al igual que todos los parámetros que se describen aquí, el secreto de cliente debe estar codificado como URL antes de enviarse, un paso que normalmente realiza el SDK. Para obtener más información sobre la codificación de URI, consulte la [especificación sobre la sintaxis genérica de URI](https://tools.ietf.org/html/rfc3986#page-12). |

### <a name="request-an-access-token-with-a-certificate-credential"></a>Solicitud de un token de acceso con una credencial de certificados

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&grant_type=authorization_code
&code_verifier=ThisIsntRandomButItNeedsToBe43CharactersLong
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
```

| Parámetro  | Obligatorio/opcional | Descripción     |
|------------|-------------------|----------------|
| `tenant`   | requerido   | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).  |
| `client_id` | requerido  | El identificador de aplicación (cliente) que la página [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `scope`      | opcional   | Lista de ámbitos separados por espacios. Todos los ámbitos deben provenir de un recurso único, junto con los ámbitos de OIDC (`profile`, `openid`, `email`). Para obtener una explicación más detallada de los ámbitos, consulte [permisos, consentimiento y ámbitos](v2-permissions-and-consent.md). Se trata de una extensión de Microsoft para el flujo de código de autorización, diseñada para permitir que las aplicaciones declaren el recurso para el que quieren el token durante el canje de tokens.|
| `code`          | requerido  | El authorization_code que adquirió en el primer segmento del flujo. |
| `redirect_uri`  | requerido  | El mismo valor redirect_uri usado para adquirir el código de autorización. |
| `grant_type` | requerido   | Debe ser `authorization_code` para el flujo de código de autorización.   |
| `code_verifier` | recomendado  | El mismo valor de code_verifier que usó para obtener el valor de authorization_code. Se requiere si PKCE se utilizó en la solicitud de concesión de código de autorización. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |
| `client_assertion_type` | requerido para las aplicaciones web confidenciales | El valor debe establecerse en `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` para usar una credencial de certificados. |
| `client_assertion` | requerido para las aplicaciones web confidenciales  | Una aserción (JSON Web Token) que debe crear y firmar con el certificado que ha registrado como credenciales de la aplicación. Lea el artículo sobre las [credenciales de certificado](active-directory-certificate-credentials.md) para información sobre cómo registrar el certificado y el formato de la aserción.|

Tenga en cuenta que los parámetros son iguales que en el caso de solicitud con un secreto compartido, salvo que el parámetro `client_secret` se sustituye por dos parámetros: `client_assertion_type` y `client_assertion`.  

### <a name="successful-response"></a>Respuesta correcta

Una respuesta de token correcta tendrá un aspecto similar al siguiente:

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```

| Parámetro     | Descripción   |
|---------------|------------------------------|
| `access_token`  | El token de acceso solicitado. La aplicación puede utilizar este token para autenticarse en el recursos protegido, como una API web.  |
| `token_type`    | Indica el valor de tipo de token. El único tipo que admite Azure AD es el portador |
| `expires_in`    | Durante cuánto tiempo es válido el token de acceso (en segundos). |
| `scope`         | Los ámbitos para los que el access_token es válido. Opcional: no es estándar y, si se omite, el token será para los ámbitos solicitados en el segmento inicial del flujo. |
| `refresh_token` | Un token de actualización de OAuth 2.0. La aplicación puede utilizar este token para adquirir tokens de acceso adicionales una vez que expire el token de acceso actual. Los refresh_tokens son de larga duración y pueden usarse para conservar el acceso a los recursos durante largos períodos de tiempo. Para más información acerca de la actualización de un token de acceso, consulte la [siguiente sección](#refresh-the-access-token). <br> **Nota:** Solo se proporciona si se solicitó el ámbito `offline_access`. |
| `id_token`      | Un JSON Web Token (JWT). La aplicación puede decodificar los segmentos de este token para solicitar información acerca del usuario que ha iniciado sesión. La aplicación puede almacenar en memoria caché los valores y mostrarlos, y los clientes confidenciales pueden usar esto para la autorización. Para más información sobre los parámetros id_tokens, consulte [`id_token reference`](id-tokens.md). <br> **Nota:** Solo se proporciona si se solicitó el ámbito `openid`. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error tendrán un aspecto similar al siguiente:

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Parámetro         | Descripción    |
|-------------------|----------------|
| `error`       | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |
| `error_codes` | Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos.  |
| `timestamp`   | La hora a la que se produjo el error. |
| `trace_id`    | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| `correlation_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |

### <a name="error-codes-for-token-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de token

| Código de error         | Descripción        | Acción del cliente    |
|--------------------|--------------------|------------------|
| `invalid_request`  | Error de protocolo, como un parámetro obligatorio que falta. | Corrija la solicitud o el registro de la aplicación y vuelva a enviar la solicitud.   |
| `invalid_grant`    | El código de autorización o el comprobador de código PKCE no son válidos o han expirado. | Intente una nueva solicitud para el punto de conexión `/authorize` y compruebe que el parámetro code_verifier es correcto.  |
| `unauthorized_client` | El cliente autenticado no está autorizado para usar este tipo de concesión de autorización. | Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se ha agregado al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `invalid_client` | Se produjo un error de autenticación de cliente.  | Las credenciales del cliente no son válidas. Para corregirlo, el administrador de la aplicación actualiza las credenciales.   |
| `unsupported_grant_type` | El servidor de autorización no admite el tipo de concesión de autorización. | Cambie el tipo de concesión de la solicitud. Este tipo de error solo debe producirse durante el desarrollo y detectarse en las pruebas iniciales. |
| `invalid_resource` | El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o no está configurado correctamente. | Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD.  |
| `interaction_required` | No es estándar, ya que la especificación de OIDC lo requiere solo en el punto de conexión `/authorize`. La solicitud requiere la interacción del usuario. Por ejemplo, hay que realizar un paso de autenticación más. | Vuelva a tratar de realizar la solicitud `/authorize` con los mismos ámbitos. |
| `temporarily_unavailable` | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. | Vuelva a intentar la solicitud después de un pequeño retraso. La aplicación podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |
|`consent_required` | La solicitud requiere el consentimiento del usuario. Este error no es estándar, ya que normalmente solo se devuelve en el punto de conexión `/authorize` por las especificaciones de OIDC. Se devuelve cuando se usó un parámetro `scope` en el flujo de canje de código que la aplicación cliente no tiene permiso para solicitar.  | El cliente debe devolver al usuario al punto de conexión `/authorize` con el ámbito correcto para desencadenar el consentimiento. |
|`invalid_scope` | El ámbito solicitado por la aplicación no es válido.  | Cambie el valor del parámetro de ámbito en la solicitud de autenticación por un valor válido. |

> [!NOTE]
> Es posible que las aplicaciones de página única reciban un error `invalid_request` que indica que solo se permite el canje de tokens entre orígenes para el tipo de cliente "Aplicación de página única".  Esto indica que el URI de redirección usado para solicitar el token no se ha marcado como un URI de redirección `spa`.  Revise los [pasos de registro de aplicaciones](#redirect-uri-setup-required-for-single-page-apps) sobre cómo habilitar este flujo.

## <a name="use-the-access-token"></a>Uso del token de acceso

Ahora que ha adquirido correctamente un `access_token`, puede usar el token en solicitudes a las API web mediante su inclusión en el encabezado `Authorization`:

> [!TIP]
> Ejecute esta solicitud en Postman (Reemplace primero el encabezado `Authorization`) [![Pruebe a ejecutar esta solicitud en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```HTTP
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-access-token"></a>Actualización del token de acceso

Los Access_tokens tienen una duración breve y debe actualizarlos una vez expiren para seguir teniendo acceso a los recursos. Puede hacerlo mediante el envío de otra solicitud `POST` al punto de conexión `/token`, esta vez proporcionando el elemento `refresh_token` en lugar del elemento `code`.  Los tokens de actualización son válidos para todos los permisos para los que el cliente ya haya obtenido consentimiento; por tanto, un token de actualización emitido en una solicitud de `scope=mail.read` se puede usar para solicitar un nuevo token de acceso para `scope=api://contoso.com/api/UseResource`.

Los tokens de actualización de aplicaciones web y aplicaciones nativas no tienen una duración especificada. Normalmente, las duraciones de este tipo de tokens son relativamente largas. Sin embargo, en algunos casos, los tokens de actualización expiran, se revocan o carecen de privilegios suficientes para realizar la acción deseada. La aplicación debe esperar y controlar los [errores que devuelve el punto de conexión de emisión de tokens](#error-codes-for-token-endpoint-errors) correctamente. Sin embargo, las aplicaciones de página única obtienen un token con una vigencia de 24 horas, lo que requiere una nueva autenticación cada día.  Esto se puede hacer de forma silenciosa en un iframe cuando las cookies de terceros estén habilitadas, pero se debe realizar en un marco de nivel superior (navegación de página completa o emergente) en los exploradores sin cookies de terceros, como Safari.

Si bien los tokens de actualización no se revocan cuando se usan para adquirir nuevos tokens de acceso, se espera que los descarte. La [especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-6) indica: "El servidor de autorización PODRÍA emitir un token de actualización nuevo, en cuyo caso el cliente DEBE descartar el token de actualización anterior y reemplazarlo por el token de actualización nuevo. El servidor de autorización PODRÍA revocar el token de actualización anterior después de la emisión de un token de actualización nuevo al cliente".

>[!IMPORTANT]
> En el caso de los tokens de actualización enviados a un URI de redirección registrado como `spa`, el token de actualización expirará después de 24 horas. Los tokens de actualización adicionales adquiridos con el token de actualización inicial trasladarán esa hora de expiración, por lo que las aplicaciones deben estar preparadas para volver a ejecutar el flujo de código de autorización con una autenticación interactiva para obtener un nuevo token de actualización cada 24 horas. Los usuarios no tienen que escribir sus credenciales y, por lo general, no verán ninguna experiencia de usuario, solo una recarga de la aplicación, pero el explorador debe visitar la página de inicio de sesión en un marco de nivel superior para ver la sesión iniciada.  Esto se debe a las [características de privacidad en los exploradores que bloquean las cookies de terceros](reference-third-party-cookies-spas.md).

```HTTP

// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&grant_type=refresh_token
&client_secret=JqQX2PNo9bpM0uEihUPzyrh      // NOTE: Only required for web apps. This secret needs to be URL-Encoded
```

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman (No olvide reemplazar `refresh_token`) [![Pruebe a ejecutar esta solicitud en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)
>

| Parámetro     | Tipo           | Descripción        |
|---------------|----------------|--------------------|
| `tenant`        | requerido     | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).   |
| `client_id`     | requerido    | El **identificador de aplicación (cliente)** que la experiencia [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `grant_type`    | requerido    | Debe ser `refresh_token` para este segmento del flujo de código de autorización. |
| `scope`         | requerido    | Una lista de ámbitos separada por espacios. Los ámbitos solicitados en este segmento deben ser un subconjunto de los ámbitos solicitados en el segmento de la solicitud del  authorization_code original o un equivalente de este. Si los ámbitos especificados en esta solicitud abarcan varios servidores de recursos, la Plataforma de identidad de Microsoft devolverá un token para el recurso especificado en el primer ámbito. Para obtener una explicación más detallada de los ámbitos, consulte [permisos, consentimiento y ámbitos](v2-permissions-and-consent.md). |
| `refresh_token` | requerido    | El refresh_token que adquirió en el segundo segmento del flujo. |
| `client_secret` | necesario para las aplicaciones web | El secreto de la aplicación que creó en el portal de registro de aplicaciones para su aplicación. No debería utilizarse en una aplicación nativa, porque los client_secrets no se pueden almacenar de forma confiable en los dispositivos. Es necesario para aplicaciones web y las API web, que tienen la capacidad de almacenar el client_secret de manera segura en el lado del servidor. Este secreto debe estar codificado como dirección URL. Para obtener más información, consulte la [especificación sobre la sintaxis genérica de URI](https://tools.ietf.org/html/rfc3986#page-12). |

#### <a name="successful-response"></a>Respuesta correcta

Una respuesta de token correcta tendrá un aspecto similar al siguiente:

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```

| Parámetro     | Descripción         |
|---------------|-------------------------------------------------------------|
| `access_token`  | El token de acceso solicitado. La aplicación puede utilizar este token para autenticarse en el recursos protegido, como una API web. |
| `token_type`    | Indica el valor de tipo de token. El único tipo que admite Azure AD es el portador |
| `expires_in`    | Durante cuánto tiempo es válido el token de acceso (en segundos).   |
| `scope`         | Los ámbitos para los que el access_token es válido.    |
| `refresh_token` | Un nuevo token de actualización de OAuth 2.0. Debe reemplazar el token de actualización antiguo con este token de actualización recientemente adquirido para asegurar que los tokens de actualización siguen siendo válidos durante tanto tiempo como sea posible. <br> **Nota:** Solo se proporciona si se solicitó el ámbito `offline_access`.|
| `id_token`      | Un token web JSON (JWT) sin firmar. La aplicación puede decodificar los segmentos de este token para solicitar información acerca del usuario que ha iniciado sesión. La aplicación puede almacenar en caché los valores y mostrarlos, pero no debe confiar en ellos para cualquier autorización o límite de seguridad. Para más información sobre los parámetros id_tokens, consulte [`id_token reference`](id-tokens.md). <br> **Nota:** Solo se proporciona si se solicitó el ámbito `openid`. |

#### <a name="error-response"></a>Respuesta de error

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Parámetro         | Descripción                                                                                        |
|-------------------|----------------------------------------------------------------------------------------------------|
| `error`           | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación.           |
| `error_codes` |Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos. |
| `timestamp` | La hora a la que se produjo el error. |
| `trace_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| `correlation_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |

Para ver una descripción de los códigos de error y la acción recomendada que tiene que realizar el cliente, consulte la sección [Códigos de error correspondientes a errores de puntos de conexión de token](#error-codes-for-token-endpoint-errors).
