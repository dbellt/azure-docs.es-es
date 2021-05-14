---
title: 'Ejemplos de comunicación del servicio emisor (versión preliminar): credenciales verificables de Azure Active Directory'
description: Detalles de la comunicación entre el proveedor de identidades y el servicio emisor
author: barclayn
manager: davba
ms.service: identity
ms.subservice: verifiable-credentials
ms.workload: identity
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: 6aa502e1ed0e49192220174d5a8573690035a4a3
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739141"
---
# <a name="issuer-service-communication-examples-preview"></a>Ejemplos de comunicación del servicio emisor (versión preliminar)

El servicio de credenciales verificables de Azure AD puede emitir credenciales verificables mediante la recuperación de notificaciones de un token de identificación generado por el proveedor de identidades compatible con la instancia de OpenID de la organización. En este artículo se indica cómo configurar el proveedor de identidades para que Authenticator pueda comunicarse con él y recuperar el token de id. correcto para pasarlo al servicio de emisión. 

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


Para emitir una credencial verificable, se dan instrucciones a Authenticator a través de la descarga del contrato que recopile la entrada del usuario y envíe esa información al servicio de emisión. Si necesita usar un token de id., debe configurar el proveedor de identidades para permitir que Authenticator inicie sesión en un usuario mediante el protocolo OpenID Connect. Las notificaciones del token de id. resultante se utilizan para rellenar el contenido de la credencial verificable. Authenticator autentica al usuario mediante el flujo de código de autorización de OpenID Connect. El proveedor de OpenID debe admitir las siguientes características de OpenID Connect: 

| Característica | Descripción |
| ------- | ----------- |
| Tipo de concesión | Debe ser compatible con el tipo de concesión de código de autorización. |
| Formato de tokens | Debe generar JWT compactos sin cifrar. |
| Algoritmo de firma | Debe generar JWT firmados mediante RS 256. |
| Documento de configuración | Debe ser compatible con el documento de configuración de OpenID Connect y `jwks_uri`. | 
| Registro de cliente | Debe admitir el registro de cliente público mediante un valor `redirect_uri` de `vcclient://openid/`. | 
| PKCE | Se recomienda por motivos de seguridad, pero no es necesario. |

A continuación, se incluyen ejemplos de las solicitudes HTTP enviadas al proveedor de identidades. El proveedor de identidades debe aceptar y responder a estas solicitudes de acuerdo con el estándar de autenticación de OpenID Connect.

## <a name="client-registration"></a>Registro de cliente

Para recibir una credencial verificable, los usuarios deben iniciar sesión en el IDP desde la aplicación Microsoft Authenticator. 

Para habilitar este intercambio, registre una aplicación en el proveedor de identidades. Si usa Azure AD, puede encontrar las instrucciones [aquí](../develop/quickstart-register-app.md). Al registrarse, use los siguientes valores.

| Configuración | Value |
| ------- | ----- |
| Nombre de la aplicación | `<Issuer Name> Verifiable Credential Service` |
| URI de redireccionamiento | `vcclient://openid/ ` |


Después de registrar una aplicación en el proveedor de identidades, registre su id. de cliente. Lo usará en la siguiente sección. También debe anotar la dirección URL del punto de conexión conocido para el proveedor de identidades compatible con OIDC. El servicio de emisión usa este punto de conexión para descargar las claves públicas necesarias para validar el token de id., después de que Authenticator lo envíe.

Authenticator usa el URI de redireccionamiento configurado para saber cuándo se completa el inicio de sesión y puede recuperar el token de id. 

## <a name="authorization-request"></a>Solicitud de autorización

La solicitud de autorización enviada al proveedor de identidades usa el formato siguiente.

```HTTP
GET /authorize?client_id=<client-id>&redirect_uri=portableidentity%3A%2F%2Fverify&response_mode=query&response_type=code&scope=openid&state=12345&nonce=12345 HTTP/1.1
Host: www.contoso.com
Connection: Keep-Alive
```

| Parámetro | Value |
| ------- | ----------- |
| `client_id` | El id. de cliente obtenido durante el proceso de registro de la aplicación. |
| `redirect_uri` | Debe usar `vcclient://openid/`. |
| `response_mode` | Debe ser compatible con `query`. |
| `response_type` | Debe ser compatible con `code`. |
| `scope` | Debe ser compatible con `openid`. |
| `state` | Debe devolverse al cliente según el estándar de OpenID Connect. |
| `nonce` | Debe devolverse como una notificación en el token de id. según el estándar de OpenID Connect. |

Cuando recibe una solicitud de autorización, el proveedor de identidades debe autenticar al usuario y realizar los pasos necesarios para completar el inicio de sesión, como la autenticación multifactor.

Puede personalizar el proceso de inicio de sesión para satisfacer sus necesidades. Puede pedir a los usuarios que proporcionen información adicional, acepten los términos de servicio, paguen sus credenciales, etc. Una vez completados todos los pasos, responda a la solicitud de autorización mediante el redireccionamiento al URI de redireccionamiento, como se muestra a continuación. 

```HTTP
vcclient://openid/?code=nbafhjbh1ub1yhbj1h4jr1&state=12345
```

| Parámetro | Value |
| ------- | ----------- |
| `code` |  Código de autorización devuelto por el proveedor de identidades. |
| `state` | Debe devolverse al cliente según el estándar de OpenID Connect. |

## <a name="token-request"></a>Solicitud de token

La solicitud de token enviada al proveedor de identidades tendrá el formato siguiente.

```HTTP
POST /token HTTP/1.1
Host: www.contoso.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 291

client_id=<client-id>&redirect_uri=vcclient%3A%2F%2Fopenid%2F&grant_type=authorization_code&code=nbafhjbh1ub1yhbj1h4jr1&scope=openid
```

| Parámetro | Value |
| ------- | ----------- |
| `client_id` | El id. de cliente obtenido durante el proceso de registro de la aplicación. |
| `redirect_uri` | Debe usar `vcclient://openid/`. |
| `scope` | Debe ser compatible con `openid`. |
| `grant_type` | Debe ser compatible con `authorization_code`. |
| `code` | Código de autorización devuelto por el proveedor de identidades. |

Tras recibir la solicitud de token, el proveedor de identidades debe responder con un token de id.

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
"id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzc
    yI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5
    NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZ
    fV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5Nz
    AKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6q
    Jp6IcmD3HP99Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJ
    NqeGpe-gccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7Tpd
    QyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoS
    K5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4
    XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
}
```

El token de id. debe usar el formato de serialización compacto de JWT y no debe estar cifrado. El token de id. debe contener las siguientes notificaciones.

| Notificación | Value |
| ------- | ----------- |
| `kid` | Identificador de la clave utilizada para firmar el token de id., que corresponde a una entrada en el `jwks_uri` del proveedor de OpenID. |
| `aud` | El id. de cliente obtenido durante el proceso de registro de la aplicación. |
| `iss` | Debe ser el valor `issuer` del documento de configuración de OpenID Connect. |
| `exp` | Debe contener la hora de expiración del token de id. |
| `iat` | Debe contener la hora a la que se emitió el token de id. |
| `nonce` | El valor incluido en la solicitud de autorización. |
| Notificaciones adicionales | El token de id. debe contener cualquier notificación adicional cuyos valores se incluirán en la credencial verificable que se emitirá. En esta sección se debe incluir cualquier atributo sobre el usuario, como su nombre. |

## <a name="next-steps"></a>Pasos siguientes

- [Procedimiento para personalizar las credenciales verificables de Azure Active Directory](credential-design.md)
