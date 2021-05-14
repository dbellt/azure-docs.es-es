---
title: 'Flujo de código de autorización: Azure Active Directory B2C | Microsoft Docs'
description: Aprenda a crear aplicaciones web mediante el protocolo de autenticación de Azure AD B2C y OpenID Connect.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/04/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: d37ad4571df0912cdb2ac8954a2292df8c8a238e
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108742812"
---
# <a name="oauth-20-authorization-code-flow-in-azure-active-directory-b2c"></a>Flujo de código de autorización de OAuth 2.0 en Azure Active Directory B2C

Puede usar la concesión de un código de autorización de OAuth 2.0 en las aplicaciones instaladas en un dispositivo para obtener acceso a recursos protegidos, como las API web. Mediante la implementación de OAuth 2.0 de Azure Active Directory B2C (Azure AD B2C) puede agregar tareas de registro, inicio de sesión y otras tareas de administración de identidades a las aplicaciones de página única, móviles y de escritorio. Este artículo es independiente del lenguaje y en él describimos cómo enviar y recibir mensajes HTTP sin usar ninguna biblioteca de código abierto. Cuando sea posible, se recomienda usar las bibliotecas de autenticación de Microsoft (MSAL) admitidas. Consulte las [aplicaciones de ejemplo que usan MSAL](code-samples.md).

El flujo de código de autorización de OAuth 2.0 se describe en la [sección 4.1 de la especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749). Puede usarlo para llevar a cabo la autenticación y la autorización en la mayoría de los [tipos de aplicación](application-types.md), incluidas las aplicaciones web, las aplicaciones de página única y las aplicaciones instaladas de forma nativa. Puede usar el flujo de código de autorización de OAuth 2.0 para adquirir de forma segura tokens de acceso y tokens de actualización para las aplicaciones, que se pueden usar para acceder a recursos protegidos por un [servidor de autorización](protocols-overview.md).  El token de actualización permite que el cliente adquiera nuevos tokens de acceso (y de actualización) cuando expira el token de acceso, normalmente al cabo de una hora.

Este artículo se centra en el flujo de código de autorización de OAuth 2.0 de **clientes públicos**. Un cliente público es cualquier aplicación cliente que no es de confianza para mantener la integridad de una contraseña secreta de forma segura. Esto incluye las aplicaciones de página única, las aplicaciones móviles, las aplicaciones de escritorio y prácticamente cualquier aplicación que no se ejecute en un servidor.

> [!NOTE]
> Para agregar la administración de identidades a una aplicación web mediante Azure AD B2C, use [OpenID Connect](openid-connect.md) en lugar de OAuth 2.0.

Azure AD B2C extiende los flujos de OAuth 2.0 estándar para realizar algo más que una autorización y autenticación simples. Incorpora el [flujo de usuario](user-flow-overview.md). Con los flujos de usuario, puede usar OAuth 2.0 para agregar experiencias de usuario a la aplicación, como el registro, el inicio de sesión y la administración de perfiles. Entre los proveedores de identidades que usan el protocolo OAuth 2.0 se incluyen: [Amazon](identity-provider-amazon.md), [Azure Active Directory](identity-provider-azure-ad-single-tenant.md), [Facebook](identity-provider-facebook.md), [GitHub](identity-provider-github.md), [ Google](identity-provider-google.md) y [LinkedIn](identity-provider-linkedin.md).

Para probar las solicitudes HTTP en este artículo, haga lo siguiente:

1. Reemplace `{tenant}` por el nombre del inquilino de Azure AD B2C.
1. Reemplace `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` por el id. de la aplicación que registró previamente en el inquilino de Azure AD B2C.
1. Reemplace `{policy}` por el nombre de una directiva que haya creado en el inquilino, por ejemplo `b2c_1_sign_in`.

## <a name="redirect-uri-setup-required-for-single-page-apps"></a>Configuración del URI de redireccionamiento requerida para aplicaciones de página única

El flujo de código de autorización para aplicaciones de página única requiere configuración adicional.  Siga las instrucciones para [crear la aplicación de una sola página](tutorial-register-spa.md) para marcar correctamente el URI de redireccionamiento como habilitado para CORS. Para actualizar un URI de redirección existente para habilitar CORS, puede hacer clic en la solicitud de migración en la sección "Web" de la pestaña **Autenticación** del **Registro de aplicación**. Como alternativa, puede abrir el **editor de manifiestos de registros de aplicaciones** y establecer el campo `type` para el URI de redirección en `spa` en la sección `replyUrlsWithType`.

El tipo de redireccionamiento `spa` es compatible con las versiones anteriores del flujo implícito. Las aplicaciones que usan actualmente el flujo implícito para obtener tokens pueden moverse al tipo de URI de redireccionamiento `spa` sin problemas y seguir usando el flujo implícito.

## <a name="1-get-an-authorization-code"></a>1. Obtener un código de autorización
El flujo del código de autorización comienza con el cliente dirigiendo al usuario al punto de conexión `/authorize` . Esta es la parte interactiva del flujo, en la que el usuario lleva a cabo la acción. En esta solicitud, el cliente indica en el parámetro `scope` los permisos que debe obtener por parte del usuario. Los siguientes tres ejemplos (con saltos de línea para facilitar la lectura) usan cada uno un flujo de usuario diferente.


```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code
&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob
&response_mode=query
&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&code_challenge=YTFjNjI1OWYzMzA3MTI4ZDY2Njg5M2RkNmVjNDE5YmEyZGRhOGYyM2IzNjdmZWFhMTQ1ODg3NDcxY2Nl
&code_challenge_method=S256
```

| Parámetro | ¿Necesario? | Descripción |
| --- | --- | --- |
|{tenant}| Obligatorio | Nombre de su inquilino de Azure AD B2C.|
| {policy} | Obligatorio | Flujo de usuario que se va a ejecutar. Especifique el nombre del flujo de usuario que creó en el inquilino de Azure AD B2C. Por ejemplo: `b2c_1_sign_in`, `b2c_1_sign_up` o `b2c_1_edit_profile`. |
| client_id |Obligatorio |Identificador de aplicación asignado a la aplicación en [Azure Portal](https://portal.azure.com). |
| response_type |Obligatorio |El tipo de respuesta, que debe incluir `code` para el flujo de código de autorización. |
| redirect_uri |Obligatorio |El URI de redirección de la aplicación, donde su aplicación envía y recibe las respuestas de autenticación. Debe coincidir exactamente con uno de los URI de redireccionamiento que ha registrado en el portal, con la excepción de que debe estar codificado como URL. |
| scope |Obligatorio |Una lista de ámbitos separada por espacios. Un valor de ámbito único indica a Azure Active Directory (Azure AD) los dos permisos que se solicitan. El uso del identificador de cliente como ámbito indica que la aplicación necesita un token de acceso que se puede usar con su propio servicio o API web, representado por el mismo identificador de cliente.  El ámbito `offline_access` indica que la aplicación necesita un token de actualización para un acceso de larga duración a los recursos. También puede usar el ámbito `openid` para solicitar un token de identificador desde Azure AD B2C. |
| response_mode |Recomendado |El método que se usa para devolver el código de autorización resultante a la aplicación. Puede ser `query`, `form_post` o `fragment`. |
| state |Recomendado |Un valor incluido en la solicitud que puede ser una cadena de cualquier contenido que desee usar. Se suele usar un valor único generado de forma aleatoria para evitar los ataques de falsificación de solicitudes entre sitios. El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación. Por ejemplo, la página en la que se encontraba el usuario o el flujo de usuario que se estaba ejecutando. |
| símbolo del sistema |Opcional |El tipo de interacción necesaria con el usuario. Actualmente, el único valor válido es `login`, que obliga al usuario a escribir sus credenciales en esa solicitud. El inicio de sesión único no surtirá efecto. |
| code_challenge  | recomendado/requerido | Se usa para proteger concesiones de código de autorización a través de la clave de prueba para intercambio de códigos (PKCE). Se requiere si se incluye `code_challenge_method`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). Ahora se recomienda para todos los tipos de aplicación: aplicaciones nativas; aplicaciones de página única; y clientes confidenciales, como aplicaciones web. | 
| `code_challenge_method` | recomendado/requerido | Método utilizado para codificar `code_verifier` para el parámetro `code_challenge`. Este *DEBERÍA* ser `S256`, pero la especificación permite utilizar `plain` si por alguna razón el cliente no admite SHA256. <br/><br/>Si se excluye, se supone que `code_challenge` es texto no cifrado si se incluye `code_challenge`. La Plataforma de identidad de Microsoft admite tanto `plain` como `S256`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). Esto es necesario para las [aplicaciones de página única que usan el flujo de código de autorización](tutorial-register-spa.md).|
| login_hint | No| Se puede usar para rellenar previamente el campo de nombre de inicio de sesión de la página de inicio de sesión. Para más información, consulte [Rellenar previamente el nombre de inicio de sesión](direct-signin.md#prepopulate-the-sign-in-name).  |
| domain_hint | No| Proporciona una sugerencia a Azure AD B2C acerca del proveedor de identidades sociales que debe usarse para iniciar sesión. Si se incluye un valor válido, el usuario va directamente a la página de inicio de sesión del proveedor de identidades.  Para más información, consulte [Redirección del inicio de sesión a un proveedor social](direct-signin.md#redirect-sign-in-to-a-social-provider). |
| Parámetros personalizados | No| Parámetros personalizados que se pueden usar con [directivas personalizadas](custom-policy-overview.md). Por ejemplo, el [URI de contenido de página personalizada dinámica](customize-ui-with-html.md?pivots=b2c-custom-policy#configure-dynamic-custom-page-content-uri) o los [solucionadores de notificaciones de clave-valor](claim-resolver-overview.md#oauth2-key-value-parameters). |

En este punto, se pedirá al usuario que complete el flujo de trabajo del flujo de usuario. Esto puede implicar que el usuario tenga que escribir su nombre de usuario y contraseña, iniciar sesión con una identidad social, registrarse en el directorio o realizar otros pasos. Las acciones del usuario dependerán de cómo se defina el flujo de usuario.

Cuando el usuario haya completado el flujo de usuario, Azure AD devuelve una respuesta a la aplicación en el valor que ha usado para `redirect_uri`. Usa el método especificado en el parámetro `response_mode`. La respuesta es exactamente la misma para cada uno de los escenarios de acción del usuario, independientemente del flujo de usuario que se haya ejecutado.

Una respuesta correcta que usa `response_mode=query` tiene este aspecto:

```http
GET urn:ietf:wg:oauth:2.0:oob?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...        // the authorization_code, truncated
&state=arbitrary_data_you_can_receive_in_the_response                // the value provided in the request
```

| Parámetro | Descripción |
| --- | --- |
| código |El código de autorización que la aplicación solicitó. La aplicación puede usar el código de autorización para solicitar un token de acceso para un recurso de destino. Los códigos de autorización tienen una duración muy breve. Normalmente, caducan al cabo de unos 10 minutos. |
| state |Vea la descripción completa en la tabla de la sección anterior. Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |

Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas correctamente:

```http
GET urn:ietf:wg:oauth:2.0:oob?
error=access_denied
&error_description=The+user+has+cancelled+entering+self-asserted+information
&state=arbitrary_data_you_can_receive_in_the_response
```

| Parámetro | Descripción |
| --- | --- |
| error |Una cadena de código de error que se puede usar para clasificar los tipos de errores que se producen. También puede usar la cadena para reaccionar frente a errores. |
| error_description |Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |
| state |Vea la descripción completa en la tabla anterior. Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debe comprobar que los valores `state` de la solicitud y de la respuesta sean idénticos. |

## <a name="2-get-an-access-token"></a>2. Obtención de un token de acceso
Ahora que ha adquirido un código de autorización, puede canjear el elemento `code` por un token al recurso deseado mediante el envío de una solicitud POST al punto de conexión `/token`. En Azure AD B2C, puede [solicitar tokens de acceso para otras API](access-tokens.md#request-a-token) como de costumbre especificando sus ámbitos en la solicitud.

También puede solicitar un token de acceso para su propia API web de back-end de la aplicación mediante la convención del uso del identificador del cliente de la aplicación como el ámbito solicitado (lo que resultará en un token de acceso con ese identificador de cliente como el "público"):

```http
POST https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1

Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob&code_verifier=ThisIsntRandomButItNeedsToBe43CharactersLong 
```

| Parámetro | ¿Necesario? | Descripción |
| --- | --- | --- |
|{tenant}| Obligatorio | Nombre de su inquilino de Azure AD B2C.|
|{policy}| Obligatorio| El flujo de usuario usado para adquirir el código de autorización. No puede usar un flujo de usuario diferente en esta solicitud. |
| client_id |Obligatorio |Identificador de aplicación asignado a la aplicación en [Azure Portal](https://portal.azure.com).|
| client_secret | Sí, en Web Apps. | El secreto de la aplicación se generó en [Azure Portal](https://portal.azure.com/). Los secretos de cliente se usan en este flujo para escenarios de aplicaciones web, donde el cliente puede almacenar de forma segura un secreto de cliente. En escenarios de aplicación nativa (cliente público), los secretos de cliente no se pueden almacenar de forma segura y, por lo tanto, no se usan en esta llamada. Si usa un secreto de cliente, cámbielo periódicamente. |
| grant_type |Obligatorio |Tipo de concesión. Para el flujo de código de autorización, el tipo de concesión debe ser `authorization_code`. |
| scope |Obligatorio |Una lista de ámbitos separada por espacios. Un valor de ámbito único indica a Azure AD los dos permisos que se solicitan. El uso del identificador de cliente como ámbito indica que la aplicación necesita un token de acceso que se puede usar con su propio servicio o API web, representado por el mismo identificador de cliente.  El ámbito `offline_access` indica que la aplicación necesita un token de actualización para un acceso de larga duración a los recursos.  También puede usar el ámbito `openid` para solicitar un token de identificador desde Azure AD B2C. |
| código |Obligatorio |El código de autorización que adquirió en el primer segmento del flujo. |
| redirect_uri |Obligatorio |El URI de redirección de la aplicación en la que recibió el código de autorización. |
| code_verifier | recomendado | El mismo valor de code_verifier que usó para obtener el valor de authorization_code. Se requiere si PKCE se utilizó en la solicitud de concesión de código de autorización. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |

Una respuesta correcta del token tiene el siguiente aspecto:

```json
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```

| Parámetro | Descripción |
| --- | --- |
| not_before |Hora a la que el token se considera válido, en tiempo de época. |
| token_type |El valor del tipo de token. El único tipo que admite Azure AD es portador. |
| access_token |El JSON Web Token (JWT) firmado que ha solicitado. |
| scope |Los ámbitos para los que el token es válido. También puede usar ámbitos para almacenar en caché los tokens para su posterior uso. |
| expires_in |El período de tiempo que el token es válido (en segundos). |
| refresh_token |Un token de actualización de OAuth 2.0. La aplicación puede usar este token para adquirir tokens adicionales una vez que expire el token actual. Los tokens de actualización tienen una duración larga. Puede usarlos para conservar el acceso a los recursos durante largos períodos de tiempo. Para obtener más información, consulte [Azure AD B2C: referencia de tokens](tokens-overview.md). |

Las respuestas de error tienen un aspecto similar al siguiente:

```json
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Parámetro | Descripción |
| --- | --- |
| error |Una cadena de código de error que se puede usar para clasificar los tipos de errores que se producen. También puede usar la cadena para reaccionar frente a errores. |
| error_description |Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

## <a name="3-use-the-token"></a>3. Uso del token
Ahora que ha adquirido correctamente un token de acceso, puede usar el token en las solicitudes a las API web de back-end incluyéndolo en el encabezado `Authorization`:

```http
GET /tasks
Host: mytaskwebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="4-refresh-the-token"></a>4. Actualización del token
Los tokens de acceso y los tokens de identificación tienen una corta duración. Una vez expirados, debe actualizarlos para poder seguir obteniendo acceso a los recursos. Para ello, envíe otra solicitud POST al punto de conexión `/token`. Esta vez, proporcione el elemento `refresh_token`, en lugar de `code`:

```http
POST https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1

Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&refresh_token=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob
```

| Parámetro | ¿Necesario? | Descripción |
| --- | --- | --- |
|{tenant}| Obligatorio | Nombre de su inquilino de Azure AD B2C.|
|{policy} |Obligatorio |El flujo de usuario usado para adquirir el token de actualización original. No puede usar un flujo de usuario diferente en esta solicitud. |
| client_id |Obligatorio |Identificador de aplicación asignado a la aplicación en [Azure Portal](https://portal.azure.com). |
| client_secret | Sí, en Web Apps. | El secreto de la aplicación se generó en [Azure Portal](https://portal.azure.com/). Los secretos de cliente se usan en este flujo para escenarios de aplicaciones web, donde el cliente puede almacenar de forma segura un secreto de cliente. En escenarios de aplicación nativa (cliente público), los secretos de cliente no se pueden almacenar de forma segura y, por lo tanto, no se usan en esta llamada. Si usa un secreto de cliente, cámbielo periódicamente. |
| grant_type |Obligatorio |Tipo de concesión. Para este segmento del flujo de código de autorización, el tipo de concesión debe ser `refresh_token`. |
| scope |Recomendado |Una lista de ámbitos separada por espacios. Un valor de ámbito único indica a Azure AD los dos permisos que se solicitan. El uso del identificador de cliente como ámbito indica que la aplicación necesita un token de acceso que se puede usar con su propio servicio o API web, representado por el mismo identificador de cliente.  El ámbito `offline_access` indica que la aplicación necesitará un token de actualización para un acceso de larga duración a los recursos.  También puede usar el ámbito `openid` para solicitar un token de identificador desde Azure AD B2C. |
| redirect_uri |Opcional |El URI de redirección de la aplicación en la que recibió el código de autorización. |
| refresh_token |Obligatorio |El token de actualización original que adquirió en el segundo segmento del flujo. |

Una respuesta correcta del token tiene el siguiente aspecto:

```json
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```
| Parámetro | Descripción |
| --- | --- |
| not_before |Hora a la que el token se considera válido, en tiempo de época. |
| token_type |El valor del tipo de token. El único tipo que admite Azure AD es portador. |
| access_token |El JWT firmado que ha solicitado. |
| scope |Los ámbitos para los que el token es válido. También puede usar los ámbitos para almacenar en caché los tokens para su posterior uso. |
| expires_in |El período de tiempo que el token es válido (en segundos). |
| refresh_token |Un token de actualización de OAuth 2.0. La aplicación puede usar este token para adquirir tokens adicionales una vez que expire el token actual. Los tokens de actualización son de larga duración y pueden usarse para conservar el acceso a los recursos durante largos periodos. Para obtener más información, consulte [Azure AD B2C: referencia de tokens](tokens-overview.md). |

Las respuestas de error tienen un aspecto similar al siguiente:

```json
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Parámetro | Descripción |
| --- | --- |
| error |Una cadena de código de error que se puede usar para clasificar los tipos de errores que se producen. También puede usar la cadena para reaccionar frente a errores. |
| error_description |Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

## <a name="use-your-own-azure-ad-b2c-directory"></a>Usar su propio directorio de Azure AD B2C
Para probar estas solicitudes, lleve a cabo los siguientes pasos. Reemplace los valores de ejemplo que hemos usado en este artículo por los suyos propios.

1. [Cree un directorio de Azure AD B2C](tutorial-create-tenant.md). Use el nombre del directorio en las solicitudes.
2. [Cree una aplicación](tutorial-register-applications.md) para obtener un identificador de aplicación y un URI de redirección. Incluya un cliente nativo en la aplicación.
3. [Cree los flujos de usuario](user-flow-overview.md) para obtener sus nombres de flujo de usuario.
