---
title: Inicio de sesión de página única mediante flujo implícito
titleSuffix: Azure AD B2C
description: Aprenda a agregar el inicio de sesión de página única usando el flujo implícito de OAuth 2.0 con Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/19/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: d2a4e58201557d64602cc0740b83373ce53c8f69
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070684"
---
# <a name="single-page-sign-in-using-the-oauth-20-implicit-flow-in-azure-active-directory-b2c"></a>Inicio de sesión de página única mediante el flujo implícito de OAuth 2.0 con Azure Active Directory B2C

Muchas aplicaciones modernas tienen un front-end de aplicación de página única escrito principalmente en JavaScript. A menudo, la aplicación se escribe con una plataforma como React, Angular o Vue.js. Las aplicaciones de una sola página y otras aplicaciones JavaScript que se ejecutan principalmente en un explorador tienen algunos retos adicionales para la autenticación:

- Las características de seguridad de estas aplicaciones son diferentes de las de las aplicaciones web tradicionales basadas en el servidor.
- Muchos proveedores de identidades y servidores de autorización no admiten solicitudes de uso compartido de recursos entre orígenes (CORS).
- Los redireccionamientos del explorador a una página completa fuera de la aplicación pueden ser invasivos para la experiencia del usuario.

La forma recomendada de admitir aplicaciones de página única es el [Flujo de código de autorización de OAuth 2.0 (con PKCE)](./authorization-code-flow.md).

Algunos marcos, como [MSAL.js 1.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core), solo admiten el flujo de concesión implícita. En estos casos, Azure Active Directory B2C (Azure AD B2C) admite el flujo de concesión implícita de autorización de OAuth 2.0. El flujo se describe en la [sección 4.2 de la especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749). En el flujo implícito, la aplicación recibe tokens directamente del punto de conexión de autorización de Azure Active Directory (Azure AD), sin ningún intercambio de servidor a servidor. La lógica de autenticación y el control de sesiones tienen lugar por completo en el cliente de JavaScript con un redireccionamiento de página o un cuadro emergente.

Azure AD B2C extiende el flujo implícito de OAuth 2.0 estándar para realizar algo más que una autorización y autenticación simples. Azure AD B2C presenta el [parámetro de directiva](user-flow-overview.md). Con el parámetro de directiva, puede usar OAuth 2.0 para agregar políticas a su aplicación, como registro, inicio de sesión y flujos de usuario de administración de perfiles. En las solicitudes HTTP de ejemplo de este artículo se usa **{tenant}.onmicrosoft.com** como ejemplo. Reemplace `{tenant}` por el nombre de su inquilino si tiene uno y también ha creado un flujo de usuario.

El flujo de inicio de sesión implícito tiene un aspecto similar al de la figura siguiente. Cada paso se describe con detalle más adelante en este artículo.

![Diagrama de estilo carril que muestra el flujo implícito de OpenID Connect](./media/implicit-flow-single-page-application/convergence_scenarios_implicit.png)

## <a name="send-authentication-requests"></a>Envío de solicitudes de autenticación

Cuando su aplicación web necesite autenticar al usuario y ejecutar el flujo de usuario, puede dirigir al usuario al punto de conexión `/authorize`. El usuario realiza una acción en función del flujo de usuario.

En esta solicitud, el cliente indica los permisos que necesita adquirir del usuario en el parámetro `scope` y el flujo de usuario que se va a ejecutar. Para hacerse una idea de cómo funciona la solicitud, intente pegarla en un explorador y ejecutarla. Reemplace `{tenant}` por el nombre del inquilino de Azure AD B2C. Reemplace `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` por el id. de la aplicación que registró previamente en el inquilino. Reemplace `{policy}` por el nombre de una directiva que haya creado en el inquilino, por ejemplo `b2c_1_sign_in`.

```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=id_token+token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=fragment
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
```

| Parámetro | Obligatorio | Descripción |
| --------- | -------- | ----------- |
|{tenant}| Sí | Nombre de su inquilino de Azure AD B2C.|
|{policy}| Sí| Flujo de usuario que se va a ejecutar. Especifique el nombre del flujo de usuario que creó en el inquilino de Azure AD B2C. Por ejemplo: `b2c_1_sign_in`, `b2c_1_sign_up` o `b2c_1_edit_profile`. |
| client_id | Sí | Identificador de aplicación que [Azure Portal](https://portal.azure.com/) asignó a la aplicación. |
| response_type | Sí | Debe incluir `id_token` para el inicio de sesión en OpenID Connect. También puede incluir el tipo de respuesta `token`. Si usa `token`, la aplicación puede recibir un token de acceso inmediatamente desde el punto de conexión autorizado sin realizar una segunda solicitud para autorizar el punto de conexión.  Si usa el tipo de respuesta `token`, el parámetro `scope` debe contener un ámbito que indica para qué recurso se va a emitir el token. |
| redirect_uri | No | El URI de redireccionamiento de la aplicación, adonde la aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los URI de redireccionamiento que ha registrado en el portal, con la excepción de que debe estar codificado como URL. |
| response_mode | No | Especifica el método que se usará para enviar el token resultante de nuevo a la aplicación.  Para los flujos implícitos, use `fragment`. |
| scope | Sí | Una lista de ámbitos separada por espacios. Un valor de ámbito único indica a Azure AD los dos permisos que se solicitan. El ámbito `openid` indica un permiso para iniciar sesión con el usuario y obtener los datos del usuario en forma de tokens de identificador. El ámbito `offline_access` es opcional para las aplicaciones web. Indica que la aplicación necesita un token de actualización para un acceso de larga duración a los recursos. |
| state | No | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que quiera usar. Se suele usar un valor único generado de manera aleatoria para evitar los ataques de falsificación de solicitudes entre sitios. El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página en la que estaban. |
| valor de seguridad | Sí | Un valor incluido en la solicitud (generada por la aplicación), que se incluye en el token de identificador resultante como una notificación. La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Normalmente, el valor es una cadena única aleatoria que puede usarse para identificar el origen de la solicitud. |
| símbolo del sistema | No | El tipo de interacción con el usuario que se necesita. Actualmente, el único valor válido es `login`. Este parámetro obliga al usuario a escribir sus credenciales en esa solicitud. El inicio de sesión único no surte efecto. |

En este punto se pedirá al usuario que complete el flujo de trabajo de la directiva. Puede que el usuario tenga que escribir su nombre de usuario y contraseña, iniciar sesión con una identidad social, registrarse en el directorio o realizar otros pasos. Las acciones del usuario dependerán de cómo se defina el flujo de usuario.

Cuando el usuario haya completado el flujo de usuario, Azure AD devuelve una respuesta a la aplicación en el valor que ha usado para `redirect_uri`. Usa el método especificado en el parámetro `response_mode`. La respuesta es exactamente la misma para cada uno de los escenarios de acción del usuario, independientemente del flujo de usuario que se haya ejecutado.

### <a name="successful-response"></a>Respuesta correcta
Una respuesta correcta que usa `response_mode=fragment` y `response_type=id_token+token` es como la siguiente, con saltos de línea para mejorar la legibilidad:

```http
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&token_type=Bearer
&expires_in=3599
&scope="90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
```

| Parámetro | Descripción |
| --------- | ----------- |
| access_token | El token de acceso que solicitó la aplicación. |
| token_type | El valor del tipo de token. El único tipo que admite Azure AD es portador. |
| expires_in | El período de validez del token de acceso (en segundos). |
| scope | Los ámbitos para los que el token es válido. También puede usar ámbitos para almacenar en caché los tokens para su posterior uso. |
| ID_token | El token de identificador que la aplicación solicitó. Puede usar el token de identificador para comprobar la identidad del usuario y comenzar una sesión con el usuario. Para obtener más información sobre los tokens de identificador y su contenido, vea [Azure AD B2C: referencia de tokens](tokens-overview.md). |
| state | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |

### <a name="error-response"></a>Respuesta de error
Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas correctamente:

```http
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| Parámetro | Descripción |
| --------- | ----------- |
| error | Código que se usa para clasificar los tipos de errores que se producen. |
| error_description | Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |
| state | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos.|

## <a name="validate-the-id-token"></a>Validar el token de identificador

Recibir un token de identificador no es suficiente para autenticar al usuario. Valide la firma del token de identificador y compruebe las notificaciones en el token en función de los requisitos de la aplicación. Azure AD B2C usa [tokens web JSON (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) y la criptografía de clave pública para firmar los tokens y comprobar que son válidos.

Muchas bibliotecas de código abierto están disponibles para validar los JWT, según su lenguaje preferido. Considere la posibilidad de explorar las bibliotecas de código abierto disponibles, en lugar de implementar su propia lógica de validación. Puede usar la información de este artículo para ayudarle a obtener información sobre cómo usar correctamente esas bibliotecas.

Azure AD B2C tiene un punto de conexión de metadatos OpenID Connect. Una aplicación puede usar el punto de conexión para obtener información sobre Azure AD B2C en tiempo de ejecución. En esta información se incluyen los extremos, los contenidos del token y las claves de firma de los token. Hay un documento de metadatos JSON para cada flujo de usuario en su inquilino de Azure AD B2C. Por ejemplo, el documento de metadatos para el flujo de usuario b2c_1_sign_in del inquilino fabrikamb2c.onmicrosoft.com se encuentra en:

```http
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/v2.0/.well-known/openid-configuration
```

Una de las propiedades de este documento de configuración es `jwks_uri`. El valor para el mismo flujo de usuario sería:

```http
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/discovery/v2.0/keys
```

Para determinar qué flujo de usuario se ha usado para firmar un token de identificador (y de dónde obtener los metadatos), tiene dos opciones. En primer lugar, el nombre del flujo de usuario se incluye en la notificación `acr` en `id_token`. Para obtener más información sobre cómo analizar las notificaciones de un token de identificador, vea [Azure AD B2C: referencia de tokens](tokens-overview.md). La otra opción consiste en codificar el flujo de usuario en el valor del parámetro `state` al emitir la solicitud. Después, descodifique el parámetro `state` para determinar qué flujo de usuario se ha usado. Cualquiera de estos métodos es válido.

Cuando haya adquirido el documento de metadatos del punto de conexión de metadatos de OpenID Connect, podrá usar las claves públicas RSA-256 (que están ubicadas en este punto de conexión) para validar la firma del token de identificador. Podría haber varias claves enumeradas en este punto de conexión en cualquier momento, cada una identificada con una `kid`. El encabezado de `id_token` también contiene una notificación `kid`. Indica cuál de estas claves se ha usado para firmar el token de identificador. Para obtener más información, incluida la información para [validar los tokens](tokens-overview.md), vea [Azure AD B2C: referencia de tokens](tokens-overview.md).
<!--TODO: Improve the information on this-->

Después de que valide la firma del token de identificador, varias notificaciones necesitan comprobarse. Por ejemplo:

* Valide la notificación `nonce` para evitar ataques de reproducción del token. Su valor debería ser el que especificó en la solicitud de inicio de sesión.
* Valide la notificación `aud` para asegurarse de que se ha generado el token de identificador para su aplicación. Su valor debería ser el identificador de la aplicación.
* Valide las notificaciones `iat` y `exp` para asegurarse de que el token de identificador no ha expirado.

Varias validaciones más que debe realizar se describen en detalle en la [especificación de OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html). Podría también querer validar notificaciones adicionales, en función de su escenario. Algunas validaciones comunes incluyen:

* Asegurarse de que la organización o el usuario se han registrado en la aplicación.
* Asegurarse de que el usuario tiene la autorización y los privilegios adecuados.
* Asegurarse de que se haya producido un determinado nivel de autenticación, como el asociado al uso de Multi-Factor Authentication de Azure AD.

Para obtener más información sobre las notificaciones en un token de identificador, vea [Azure AD B2C: referencia de tokens](tokens-overview.md).

Cuando haya validado el token de identificador, podrá iniciar una sesión con el usuario. Use las notificaciones del token de identificador para obtener información sobre el usuario en la aplicación. Esta información puede usarse para su visualización, registros, autorizaciones, etc.

## <a name="get-access-tokens"></a>Obtención de tokens de acceso
Si lo único que sus aplicaciones web necesitan hacer es ejecutar flujos de usuario, puede omitir las secciones siguientes. La información de las siguientes secciones es válida solo para las aplicaciones web que necesitan realizar llamadas autenticadas a una API web y que están protegidas por Azure AD B2C.

Ahora que ha iniciado la sesión del usuario en su aplicación de una página, puede obtener tokens de acceso para llamar a las API web que están protegidas por Azure AD. Incluso si ya ha recibido un token mediante el tipo de respuesta `token`, puede usar este método para adquirir tokens para recursos adicionales sin redirigir al usuario para que vuelva a iniciar sesión.

En el flujo de aplicación web normal, realizaríamos una solicitud al punto de conexión `/token`. Sin embargo, el punto de conexión no admite solicitudes CORS, por lo que las llamadas a AJAX para obtener un token de actualización no son una opción. En su lugar, puede usar el flujo implícito en un elemento iframe HTML oculto para obtener nuevos tokens para otras API web. Aquí se muestra un ejemplo, con saltos de línea por motivos de legibilidad:

```http
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
&response_mode=fragment
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&prompt=none
```

| Parámetro | ¿Necesario? | Descripción |
| --- | --- | --- |
|{tenant}| Obligatorio | Nombre de su inquilino de Azure AD B2C.|
{policy}| Obligatorio| Flujo de usuario que se va a ejecutar. Especifique el nombre del flujo de usuario que creó en el inquilino de Azure AD B2C. Por ejemplo: `b2c_1_sign_in`, `b2c_1_sign_up` o `b2c_1_edit_profile`. |
| client_id |Obligatorio |Identificador de aplicación asignado a la aplicación en [Azure Portal](https://portal.azure.com). |
| response_type |Obligatorio |Debe incluir `id_token` para el inicio de sesión en OpenID Connect.  También puede incluir el tipo de respuesta `token`. Si usa `token` aquí, la aplicación puede recibir un token de acceso inmediatamente desde el punto de conexión autorizado sin realizar una segunda solicitud para autorizar el punto de conexión. Si usa el tipo de respuesta `token`, el parámetro `scope` debe contener un ámbito que indica para qué recurso se va a emitir el token. |
| redirect_uri |Recomendado |El URI de redireccionamiento de la aplicación, adonde la aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los URI de redireccionamiento que ha registrado en el portal, con la excepción de que debe estar codificado como URL. |
| scope |Obligatorio |Una lista de ámbitos separada por espacios.  Para obtener los tokens, incluya todos los ámbitos que necesita para el recurso previsto. |
| response_mode |Recomendado |Especifica el método que se usará para enviar el token resultante de nuevo a la aplicación. Para los flujos implícitos, use `fragment`. Se pueden especificar otros dos modos, `query` y `form_post`, pero no funcionan en el flujo implícito. |
| state |Recomendado |Un valor incluido en la solicitud que se devolverá en la respuesta del token.  Puede ser una cadena de cualquier contenido que quiera usar.  Se suele usar un valor único generado de manera aleatoria para evitar los ataques de falsificación de solicitudes entre sitios.  El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación. Por ejemplo, la página o la vista en la que ha estado el usuario. |
| valor de seguridad |Obligatorio |Un valor incluido en la solicitud, generada por la aplicación, que se incluirá en el token de identificador resultante como una notificación.  La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Normalmente, el valor es una cadena única aleatoria que puede identificar el origen de la solicitud. |
| símbolo del sistema |Obligatorio |Para actualizar y obtener tokens en un iframe oculto, use `prompt=none` para asegurarse de que el iframe no se bloquea en la página de inicio de sesión y se devuelve inmediatamente. |
| login_hint |Obligatorio |Para actualizar y obtener tokens en un iframe oculto, incluya el nombre de usuario del usuario en esta sugerencia para distinguir entre varias sesiones que el usuario pueda tener en un momento dado. Puede extraer el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username` (el ámbito `profile` es obligatorio para recibir la notificación `preferred_username`). |
| domain_hint |Obligatorio |Puede ser `consumers` o `organizations`.  Para actualizar y obtener tokens en un iframe oculto, incluya el valor `domain_hint` en la solicitud.  Extraiga la notificación `tid` del token de identificador de un inicio de sesión anterior para determinar qué valor usar (el ámbito `profile` es obligatorio para recibir la notificación `tid`). Si el valor de la notificación `tid` es `9188040d-6c67-4c5b-b112-36a304b66dad`, use `domain_hint=consumers`.  De lo contrario, use `domain_hint=organizations`. |

Al establecer el parámetro `prompt=none`, esta solicitud se realiza correctamente o produce un error inmediatamente, y vuelve a la aplicación.  Se envía una respuesta correcta a la aplicación en el URI de redireccionamiento indicado, mediante el método especificado en el parámetro `response_mode`.

### <a name="successful-response"></a>Respuesta correcta
Una respuesta correcta al usar `response_mode=fragment` tiene el aspecto del siguiente ejemplo:

```http
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
&token_type=Bearer
&expires_in=3599
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
```

| Parámetro | Descripción |
| --- | --- |
| access_token |El token que solicitó la aplicación. |
| token_type |El tipo de token siempre será un token de portador. |
| state |Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |
| expires_in |Durante cuánto tiempo es válido el token de acceso (en segundos). |
| scope |Los ámbitos para los que el token de acceso es válido. |

### <a name="error-response"></a>Respuesta de error
Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas correctamente.  Para `prompt=none`, un error esperado tiene el aspecto del siguiente ejemplo:

```http
GET https://aadb2cplayground.azurewebsites.net/#
error=user_authentication_required
&error_description=the+request+could+not+be+completed+silently
```

| Parámetro | Descripción |
| --- | --- |
| error |Una cadena de código de error que puede usarse para clasificar los tipos de errores que se producen. También puede usar la cadena para reaccionar frente a errores. |
| error_description |Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

Si recibe este error en la solicitud de iframe, el usuario debe iniciar sesión de nuevo de manera interactiva para recuperar un nuevo token.

## <a name="refresh-tokens"></a>Tokens de actualización
Los tokens de identificador y los tokens de acceso expiran tras un breve período de tiempo. La aplicación debe estar preparada para actualizar estos tokens periódicamente.  Para actualizar cualquier tipo de token, realice la misma solicitud de iframe oculto que hemos usado en un ejemplo anterior, mediante el parámetro `prompt=none` para controlar los pasos de Azure AD.  Para recibir un nuevo valor `id_token`, asegúrese de usar `response_type=id_token` y `scope=openid`, y un parámetro `nonce`.

## <a name="send-a-sign-out-request"></a>Envío de una solicitud de cierre de sesión
Cuando quiera iniciar sesión fuera de la aplicación, redirija al usuario a Azure AD para cerrar la sesión. Si no lo redirige, este podría volver a autenticar la aplicación sin escribir sus credenciales de nuevo, ya que tienen un inicio de sesión único válido con el punto de conexión de Azure AD.

Lo que puede hacer es simplemente redirigir al usuario al elemento `end_session_endpoint` que se enumera en el mismo documento de metadatos de OpenID Connect que se ha descrito en [Validar el token de identificador](#validate-the-id-token). Por ejemplo:

```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/logout?post_logout_redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
```

| Parámetro | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| {tenant} | Sí | Nombre de su inquilino de Azure AD B2C. |
| {policy} | Sí | El flujo de usuario que quiere usar para cerrar la sesión del usuario en la aplicación. |
| post_logout_redirect_uri | No | La dirección URL a la que se debe redirigir al usuario después de un cierre de sesión correcto. Si no se incluye, Azure AD B2C mostrará un mensaje genérico al usuario. |
| state | No | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |


> [!NOTE]
> Al dirigir al usuario a `end_session_endpoint` se borra parte del estado del inicio de sesión único del usuario con Azure AD B2C. En cambio, no cierra la sesión del proveedor de identidades sociales del usuario. Si el usuario selecciona el mismo proveedor de identidades durante un inicio de sesión posterior, este se vuelve a autenticar sin especificar sus credenciales. Si un usuario quiere cerrar sesión en su aplicación de Azure AD B2C, eso no significa necesariamente que quiera cerrar sesión completamente en su cuenta de Facebook, por ejemplo. En cambio, para las cuentas locales, la sesión del usuario se terminará correctamente.
>

## <a name="next-steps"></a>Pasos siguientes

Consulte el ejemplo de código: [Inicio de sesión con Azure AD B2C en una aplicación de página única de JavaScript](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-core-samples/VanillaJSTestApp/app/b2c).
