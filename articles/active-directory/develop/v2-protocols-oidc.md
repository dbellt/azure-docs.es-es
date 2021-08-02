---
title: Plataforma de identidad de Microsoft y protocolo OpenID Connect | Azure
titleSuffix: Microsoft identity platform
description: Cree aplicaciones web mediante la implementación de la plataforma de identidad de Microsoft del protocolo de autenticación OpenID Connect.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/22/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: a01566341a1c5aa1700b68938410ee0c08c17ddd
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747552"
---
# <a name="microsoft-identity-platform-and-openid-connect-protocol"></a>Plataforma de identidad de Microsoft y protocolo OpenID Connect

OpenID Connect (OIDC) es un protocolo de autenticación basado en OAuth 2.0 que se puede usar para que un usuario inicie sesión de forma segura en una aplicación. Cuando se usa la implementación de la plataforma de identidad de Microsoft de OpenID Connect, se puede agregar acceso con información de inicio de sesión y API a las aplicaciones. En este artículo se muestra cómo hacer esto de forma independiente del lenguaje y se describe cómo enviar y recibir mensajes HTTP sin usar ninguna [biblioteca de código abierto de Microsoft](reference-v2-libraries.md).

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) amplía el protocolo de *autorización* de OAuth 2.0 para usarlo como un protocolo de *autenticación*, lo que permite realizar inicios de sesión únicos mediante OAuth. OpenID Connect presenta el concepto de un *token de identificador*, que es un token de seguridad que permite al cliente comprobar la identidad del usuario. El token de identificador también obtiene información de perfil básica sobre el usuario. También presenta el [Punto de conexión de UserInfo](userinfo.md), una API que devuelve información sobre el usuario. 


## <a name="protocol-diagram-sign-in"></a>Diagrama de protocolo: Inicio de sesión

El diagrama siguiente muestra los pasos de un flujo de inicio de sesión más básico. En este artículo se describe detalladamente cada paso.

![Protocolo OpenID Connect: Inicio de sesión](./media/v2-protocols-oidc/convergence-scenarios-webapp.svg)

## <a name="fetch-the-openid-connect-metadata-document"></a>Captura del documento de metadatos de OpenID Connect

OpenID Connect describe un documento de metadatos [(RFC)](https://openid.net/specs/openid-connect-discovery-1_0.html) que contiene la mayor parte de la información necesaria para que una aplicación realice el inicio de sesión. Esta información incluye las direcciones URL que se usan y la ubicación de las claves de firma pública del servicio. Se puede encontrar este documento mediante el anexo de la ruta de acceso del documento de detección a la dirección URL de la autoridad:

Ruta de acceso del documento de detección: `/.well-known/openid-configuration`

Autoridad: `https://login.microsoftonline.com/{tenant}/v2.0`

`{tenant}` puede tomar uno de estos cuatro valores:

| Value | Descripción |
| --- | --- |
| `common` |Los usuarios con una cuenta personal de Microsoft y una cuenta profesional o educativa de Azure AD pueden iniciar sesión en la aplicación. |
| `organizations` |Solo los usuarios con cuentas profesionales o educativas de Azure AD pueden iniciar sesión en la aplicación. |
| `consumers` |Solo los usuarios con una cuenta de Microsoft personal pueden iniciar sesión en la aplicación. |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` o `contoso.onmicrosoft.com` | Solo los usuarios de un inquilino de Azure AD específico (tanto si son miembros del directorio con una cuenta profesional o educativa, como si son invitados en el directorio con un cuenta de Microsoft personal) pueden iniciar sesión en la aplicación. Puede usarse el nombre de dominio descriptivo del inquilino de Azure AD o bien el identificador de GUID del inquilino. También puede usar el inquilino consumer `9188040d-6c67-4c5b-b112-36a304b66dad`, en lugar del inquilino `consumers`.  |

La autoridad difiere en las nubes nacionales: (por ejemplo, `https://login.microsoftonline.de` para la instancia de Azure AD de Alemania). Si no usa la nube pública, revise los [puntos de conexión de nube nacional](authentication-national-cloud.md#azure-ad-authentication-endpoints) para encontrar el adecuado. Asegúrese de que el inquilino y `/v2.0/` están presentes en la solicitud para que se pueda usar la versión 2.0 del punto de conexión.

> [!TIP]
> ¡Pruébelo! Haga clic en [https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration](https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration) para ver la configuración `common`.

### <a name="sample-request"></a>Solicitud de ejemplo

Para llamar al punto de conexión de UserInfo de la autoridad común en la nube pública, use lo siguiente:

```http
GET /common/v2.0/.well-known/openid-configuration
Host: login.microsoftonline.com
```

### <a name="sample-response"></a>Respuesta de muestra

Los metadatos son un documento de notación de objetos JavaScript (JSON) simple. Consulte el fragmento de código siguiente para ver un ejemplo. El contenido se describe detalladamente en la [especificación de OpenID Connect](https://openid.net/specs/openid-connect-discovery-1_0.html#rfc.section.4.2).

```json
{
  "authorization_endpoint": "https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize",
  "token_endpoint": "https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token",
  "token_endpoint_auth_methods_supported": [
    "client_secret_post",
    "private_key_jwt"
  ],
  "jwks_uri": "https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys",
  "userinfo_endpoint": "https://graph.microsoft.com/oidc/userinfo",
  "subject_types_supported": [
      "pairwise"
  ],
  ...

}
```

Si la aplicación tiene claves de firma personalizadas como resultado de usar la característica de [asignación de notificaciones](active-directory-claims-mapping.md), debe anexar un parámetro de consulta `appid` que contenga el identificador de aplicación con el fin de obtener un elemento `jwks_uri` que apunte a la información de la clave de firma de la aplicación. Por ejemplo: `https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration?appid=6731de76-14a6-49ae-97bc-6eba6914391e` contiene el elemento `jwks_uri` de `https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys?appid=6731de76-14a6-49ae-97bc-6eba6914391e`.

Normalmente, este documento de metadatos se usa para configurar una biblioteca o SDK de OpenID Connect; la biblioteca usa los metadatos para realizar su trabajo. Sin embargo, si no usa una biblioteca de OpenID Connect precompilada, puede seguir los pasos del resto de este artículo para realizar el inicio de sesión en una aplicación web mediante la plataforma de identidad de Microsoft.

## <a name="send-the-sign-in-request"></a>Envío de la solicitud de inicio de sesión

Cuando su aplicación web deba autenticar al usuario, puede dirigirlo al punto de conexión `/authorize` . Esta solicitud es similar al primer segmento del [flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md), con estas importantes diferencias:

* La solicitud debe incluir el ámbito `openid` en el parámetro `scope`.
* El parámetro `response_type` debe incluir `id_token`.
* La solicitud debe incluir el parámetro `nonce` .

> [!IMPORTANT]
> Con el fin de solicitar correctamente un token de identificador del /punto de conexión de autorización, el registro de la aplicación en el [portal de registro](https://portal.azure.com) debe tener la concesión implícita de id_tokens habilitada en la pestaña Autenticación (que establece la marca `oauth2AllowIdTokenImplicitFlow` del [manifiesto de aplicación](reference-app-manifest.md) en `true`). Si no está habilitada, se devolverá un error `unsupported_response`: "The provided value for the input parameter 'response_type' isn't allowed for this client. Expected value is 'code'" ("No se permite el valor proporcionado para el parámetro de entrada "response_type" para este cliente. El valor esperado es "code"")

Por ejemplo:

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=form_post
&scope=openid
&state=12345
&nonce=678910
```

| Parámetro | Condición | Descripción |
| --- | --- | --- |
| `tenant` | Obligatorio | Puede usar el valor `{tenant}` en la ruta de acceso de la solicitud para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints). |
| `client_id` | Obligatorio | El **identificador de aplicación (cliente)** que la experiencia [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `response_type` | Obligatorio | Debe incluir `id_token` para el inicio de sesión en OpenID Connect. También puede incluir otros valores `response_type`, como `code`. |
| `redirect_uri` | Recomendado | El URI de redireccionamiento de la aplicación, adonde la aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los URI de redireccionamiento que registró en el portal, con la excepción de que debe estar codificado como URL. Si no existe, el punto de conexión seleccionará al azar un valor de redirect_uri registrado para volver a enviar al usuario. |
| `scope` | Obligatorio | Una lista de ámbitos separada por espacios. Asegúrese de incluir el ámbito `openid`para OpenID Connect, lo que se traduce en el permiso de inicio de sesión en la interfaz de usuario de consentimiento. También puede incluir otros ámbitos en esta solicitud para solicitar el consentimiento. |
| `nonce` | Obligatorio | Un valor incluido en la solicitud, generado por la aplicación, que se incluirá en el valor id_token resultante como una notificación. La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Habitualmente, el valor es una cadena única aleatoria que se puede usar para identificar el origen de la solicitud. |
| `response_mode` | Recomendado | Especifica el método que se debe usar para enviar el código de autorización resultante de nuevo a la aplicación. Puede ser `form_post` o `fragment`. En el caso de las aplicaciones web, se recomienda usar `response_mode=form_post` para asegurar la transferencia más segura de tokens a la aplicación. |
| `state` | Recomendado | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Se usa normalmente un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, como la página o la vista en la que estaba el usuario. |
| `prompt` | Opcional | Indica el tipo de interacción necesaria con el usuario. Los únicos valores válidos en este momento son `login`, `none`, `consent` y `select_account`. La notificación `prompt=login` obliga al usuario a escribir sus credenciales en esa solicitud, lo que niega el inicio de sesión único. El parámetro `prompt=none` es lo contrario y se debe emparejar con `login_hint` para indicar qué usuario debe haber iniciado sesión. Estos parámetros garantizan que al usuario no se le presente ninguna solicitud interactiva. Si la solicitud no se puede completar sin notificaciones mediante el inicio de sesión único (porque ningún usuario inició sesión, el usuario sugerido no inició sesión o porque varios usuarios iniciaron sesión y no se proporcionó ninguna sugerencia), la plataforma de identidad de Microsoft devuelve un error. La notificación `prompt=consent` desencadena el cuadro de diálogo de consentimiento de OAuth después de que el usuario inicia sesión. El cuadro de diálogo le pide al usuario que conceda permisos a la aplicación. Por último, `select_account` muestra al usuario un selector de cuenta, lo que niega un SSO silencioso pero permite al cliente elegir con qué cuenta quiere iniciar sesión, sin necesidad de ingresar ninguna credencial. No es posible utilizar `login_hint` y `select_account` juntos.|
| `login_hint` | Opcional | Puede usar este parámetro para rellenar previamente el campo de nombre de usuario y dirección de correo electrónico de la página de inicio de sesión del usuario, si sabe el nombre de usuario con anticipación. A menudo, las aplicaciones usan este parámetro durante la reautenticación, una vez que ya se extrajo el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username`. |
| `domain_hint` | Opcional | El dominio del usuario en un directorio federado.  Se omite el proceso de detección basado en correo electrónico por el que pasa el usuario de la página de inicio de sesión, para obtener una experiencia de usuario ligeramente más sencilla. En el caso de los inquilinos que se federan mediante un directorio local como AD FS, con frecuencia se produce un inicio de sesión completo debido a la sesión de inicio de sesión existente. |

En este punto, se le pide al usuario que escriba sus credenciales y que complete la autenticación. La plataforma de identidad de Microsoft comprueba que el usuario ha dado su consentimiento a los permisos indicados en el parámetro de consulta `scope`. Si el usuario no dio su consentimiento a ninguno de estos permisos, la plataforma de identidad de Microsoft solicitará al usuario que dé su consentimiento a los permisos requeridos. Puede leer más información sobre los [permisos, el consentimiento y las aplicaciones multiinquilino](v2-permissions-and-consent.md).

Una vez que el usuario se autentica y da su consentimiento, la plataforma de identidad de Microsoft devuelve una respuesta a su aplicación en el URI de redireccionamiento indicado mediante el método especificado en el parámetro `response_mode`.

### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta cuando usa `response_mode=form_post` tiene el aspecto siguiente:

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&state=12345
```

| Parámetro | Descripción |
| --- | --- |
| `id_token` | El token de identificador que la aplicación solicitó. Puede usar el parámetro `id_token` para comprobar la identidad del usuario y comenzar una sesión con el usuario. Para más información sobre los tokens de identificador y su contenido, consulte la [`id_tokens` referencia](id-tokens.md). |
| `state` | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas. Una respuesta de error tiene el aspecto siguiente:

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Parámetro | Descripción |
| --- | --- |
| `error` | Una cadena de código de error que puede usar para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

### <a name="error-codes-for-authorization-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de autorización

En la tabla siguiente se describen los códigos de error que puede devolver el parámetro `error` de la respuesta de error:

| Código de error | Descripción | Acción del cliente |
| --- | --- | --- |
| `invalid_request` | Error de protocolo, como un parámetro obligatorio que falta. |Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que habitualmente se detecta durante las pruebas iniciales. |
| `unauthorized_client` | La aplicación cliente no puede solicitar un código de autorización. |Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se ha agregado al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `access_denied` | El propietario del recurso denegó el consentimiento. |La aplicación cliente puede notificar al usuario que no puede continuar, salvo que este dé su consentimiento. |
| `unsupported_response_type` |El servidor de autorización no admite el tipo de respuesta de la solicitud. |Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que habitualmente se detecta durante las pruebas iniciales. |
| `server_error` | El servidor ha detectado un error inesperado. |Vuelva a intentarlo. Estos errores pueden deberse a condiciones temporales. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a un error temporal. |
| `temporarily_unavailable` | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. |Vuelva a intentarlo. La aplicación podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |
| `invalid_resource` | El recurso de destino no es válido porque no existe, porque Azure AD no puede encontrarlo o porque no está configurado correctamente. |Esto indica que el recurso, si existe, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |

## <a name="validate-the-id-token"></a>Validar el token de identificador

Recibir un solo valor de id_token no siempre es suficiente para autenticar al usuario; también es necesario validar la firma de id_token y comprobar las notificaciones en el token según los requisitos de su aplicación. Al igual que todas las plataformas OIDC, la plataforma de identidad de Microsoft usa los [JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519) y criptografía de clave pública para firmar los tokens de id. y comprobar que son válidos.

No todas las aplicaciones se benefician de comprobar el token de id.: las aplicaciones nativas y las aplicaciones de página única, por ejemplo, casi nunca se benefician de la validación del token de id.  Un usuario con acceso físico al dispositivo (o explorador) puede omitir la validación de muchas maneras: desde editar el tráfico web hasta el dispositivo para proporcionar claves y tokens falsos a fin de depurar simplemente la aplicación y omitir la lógica de validación.  Por otro lado, las API y las aplicaciones web que usan un token de id. para la autorización deben validar cuidadosamente el token de id., ya que están canalizando el acceso a los datos.

Una vez haya validado la firma de id_token, se le solicitará que compruebe algunas notificaciones. Para más información, consulte la [Referencia de `id_token`](id-tokens.md), incluidas la sección [Validación de los tokens](id-tokens.md#validating-an-id-token) y [Sustitución de claves de firma de Azure Active Directory](active-directory-signing-key-rollover.md). Hay al menos una disponible para la mayoría de los lenguajes y las plataformas.

Se recomienda que valide notificaciones adicionales según su escenario. Algunas validaciones comunes incluyen:

* Asegurarse de que la organización/el usuario se ha registrado en la aplicación.
* Asegurarse de que el usuario tiene la autorización/los privilegios adecuados
* Asegurarse de que se haya producido un determinado nivel de autenticación, como la [autenticación multifactor](../authentication/concept-mfa-howitworks.md).

Una vez haya validado completamente el valor id_token, puede iniciar una sesión con el usuario y utilizar las notificaciones de id_token para obtener información sobre el usuario en la aplicación. Esta información puede usarse para visualización, registros, autorizaciones, etc.

## <a name="protocol-diagram-access-token-acquisition"></a>Diagrama de protocolo: Obtención de tokens de acceso

Muchas aplicaciones web no solo deben iniciar la sesión del usuario, sino también obtener acceso a un servicio web en nombre del usuario mediante OAuth. En este escenario se combina OpenID Connect para la autenticación de usuario, al tiempo que se obtiene, a la vez, un código de autorización que puede usar para obtener tokens de acceso si usa el flujo de código de autorización de OAuth.

El flujo completo de adquisición de tokens e inicio de sesión de OpenID Connect es similar al diagrama siguiente. En las secciones siguientes del artículo se describe detalladamente cada paso.

![Protocolo OpenID Connect: Obtención de tokens](./media/v2-protocols-oidc/convergence-scenarios-webapp-webapi.svg)

## <a name="get-an-access-token-to-call-userinfo"></a>Obtención de un token de acceso para llamar a UserInfo

Para adquirir un token para el punto de conexión de UserInfo de OIDC, modifique la solicitud de inicio de sesión:

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e        // Your registered Application ID
&response_type=id_token%20token                       // this will return both an id_token and an access token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F       // Your registered redirect URI, URL encoded
&response_mode=form_post                              // 'form_post' or 'fragment'
&scope=openid+profile+email                           // `openid` is required.  `profile` and `email` provide additional information in the UserInfo endpoint the same way they do in an ID token. 
&state=12345                                          // Any value, provided by your app
&nonce=678910                                         // Any value, provided by your app
```

También puede usar el [flujo de código de autorización](v2-oauth2-auth-code-flow.md), el [flujo de código de dispositivo](v2-oauth2-device-code.md) o un [token de actualización](v2-oauth2-auth-code-flow.md#refresh-the-access-token) en lugar de `response_type=token` a fin de obtener un token para la aplicación.

> [!TIP]
> Haga clic en el vínculo siguiente para ejecutar esta solicitud. Una vez que inicie sesión, el explorador se redirigirá a `https://localhost/myapp/`, con un token de id. y un token en la barra de direcciones. Tenga en cuenta que esta solicitud usa `response_mode=fragment` solo con fines de demostración: en el caso de una aplicación web, se recomienda usar `form_post` para obtener mayor seguridad siempre que sea posible. 
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token%20token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=fragment&scope=openid+profile+email&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

### <a name="successful-token-response"></a>Respuesta correcta del token

Una respuesta correcta al usar `response_mode=form_post` tiene el siguiente aspecto:

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
 access_token=eyJ0eXAiOiJKV1QiLCJub25jZSI6I....
 &token_type=Bearer
 &expires_in=3598
 &scope=email+openid+profile
 &id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI....
 &state=12345
```

Los parámetros de respuesta significan lo mismo, independientemente del flujo usado para adquirirlos.

| Parámetro | Descripción |
| --- | --- |
| `access_token` | El token que se va a usar para llamar al punto de conexión de UserInfo.|
| `token_type` | Siempre "Portador" |
| `expires_in`| Cuánto tiempo transcurre hasta que el token de acceso expira, en segundos. |
| `scope` | Permisos concedidos en el token de acceso.  Tenga en cuenta que, dado que el punto de conexión UserInfo se hospeda en MS Graph, puede haber otros ámbitos de gráfico aquí (por ejemplo, user.read) si se concedieron previamente a la aplicación.  Esto se debe a que un token de un recurso determinado siempre incluye todos los permisos concedidos actualmente al cliente.  |
| `id_token` | El token de identificador que la aplicación solicitó. Puede usar el token de identificador para comprobar la identidad del usuario y comenzar una sesión con el usuario. Encontrará más detalles sobre los tokens de identificador y su contenido en la [`id_tokens` referencia](id-tokens.md). |
| `state` | Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas correctamente. Una respuesta de error tiene el aspecto siguiente:

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Parámetro | Descripción |
| --- | --- |
| `error` | Una cadena de código de error que puede usar para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

Para una descripción de los posibles códigos de error y las respuestas de cliente recomendadas, consulte [Códigos de error correspondientes a errores de puntos de conexión de autorización](#error-codes-for-authorization-endpoint-errors).

Cuando tiene un código de autorización y un token de identificador, puede iniciar la sesión del usuario y obtener tokens de acceso en su nombre. Para iniciar la sesión del usuario, debe validar el token de identificador [exactamente como se describió](id-tokens.md#validating-an-id-token). Para obtener tokens de acceso, siga los pasos descritos en la [documentación del flujo de código de OAuth](v2-oauth2-auth-code-flow.md#redeem-a-code-for-an-access-token).

### <a name="calling-the-userinfo-endpoint"></a>Llamada al punto de conexión de UserInfo

Revise la [documentación de UserInfo](userinfo.md#calling-the-api) para ver cómo llamar al punto de conexión de UserInfo con este token.

## <a name="send-a-sign-out-request"></a>Envío de una solicitud de cierre de sesión

Si desea cerrar la sesión del usuario en la aplicación, no basta con borrar las cookies de la aplicación o finalizar la sesión del usuario. También debe redirigir al usuario a la plataforma de identidad de Microsoft para cerrar la sesión. Si no lo hace, el usuario se vuelve a autenticar en su aplicación sin escribir sus credenciales de nuevo, porque tendrá una sesión válida de inicio de sesión único con la plataforma de identidad de Microsoft.

Puede redirigir al usuario al objeto `end_session_endpoint` que aparece en el documento de metadatos OpenID Connect:

```HTTP
GET https://login.microsoftonline.com/common/oauth2/v2.0/logout?
post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
```

| Parámetro | Condición | Descripción |
| ----------------------- | ------------------------------- | ------------ |
| `post_logout_redirect_uri` | Recomendado | La dirección URL al que se redirige al usuario tras cerrar sesión correctamente. Si el parámetro no se incluye, se muestra al usuario un mensaje genérico que genera la plataforma de identidad de Microsoft. Esta dirección URL debe coincidir con uno de los URI de redirección registrados para su aplicación en el portal de registro de aplicaciones. |

## <a name="single-sign-out"></a>Cierre de sesión único

Cuando redirige al usuario a `end_session_endpoint`, la plataforma de Microsoft borra la sesión del usuario del explorador. Sin embargo, el usuario puede permanecer con la sesión iniciada en otras aplicaciones que usan cuentas de Microsoft para la autenticación. Para permitir que esas aplicaciones cierren la sesión del usuario de manera simultánea, la plataforma de identidad de Microsoft envía una solicitud HTTP GET al elemento `LogoutUrl` registrado de todas las aplicaciones en las que el usuario tiene una sesión iniciada. Las aplicaciones deben responder a esta solicitud mediante la eliminación de la sesión que identifica al usuario y la devolución de una respuesta `200`. Si quiere admitir el inicio de sesión único en la aplicación, debe implementar este elemento `LogoutUrl` en el código de la aplicación. Para configurar `LogoutUrl`, hágalo en el portal de registro de aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

* Revisar la [documentación de UserInfo](userinfo.md).
* Obtener información sobre cómo [personalizar los valores de un token](active-directory-claims-mapping.md) con datos desde los sistemas locales. 
* Obtener información sobre cómo [incluir notificaciones adicionales estándar en los tokens](active-directory-optional-claims.md).  
