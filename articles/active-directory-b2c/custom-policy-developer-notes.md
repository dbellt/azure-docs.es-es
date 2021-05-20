---
title: Notas para desarrolladores sobre flujos de usuario y directivas personalizadas
titleSuffix: Azure AD B2C
description: Notas para desarrolladores sobre la configuración y el mantenimiento de Azure AD B2C con flujos de usuario y directivas personalizadas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/04/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9c9d5ae5fec9b9258527606d352cef83d5b5a41c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108742830"
---
# <a name="developer-notes-for-azure-active-directory-b2c"></a>Notas para desarrolladores sobre Azure Active Directory B2C

Los [flujos de usuario y las directivas personalizadas](user-flow-overview.md) de Azure Active Directory B2C están disponibles con carácter general. Las funcionalidades de Azure AD B2C están en desarrollo continuo, por lo que aunque la mayoría de las características están disponibles con carácter general, algunas de ellas se encuentran en distintas fases del ciclo de lanzamiento de software. En este artículo se examinan las mejoras acumulativas de Azure AD B2C y se especifica la disponibilidad de características.

## <a name="terms-for-features-in-public-preview"></a>Términos de características en versión preliminar pública

- Se aconseja usar las características en versión preliminar pública solo con fines de evaluación.
- Los Acuerdos de Nivel de Servicio (SLA) no se aplican a las características en versión preliminar pública.
- Las solicitudes de soporte técnico referentes a las características en versión preliminar pública pueden enviarse a través de los canales de soporte técnico habituales.

## <a name="user-flows"></a>Flujos de usuario

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
| [Registro e inicio de sesión](add-sign-up-and-sign-in-policy.md) con correo electrónico y contraseña. | GA | GA| |
| [Registro e inicio de sesión](add-sign-up-and-sign-in-policy.md) con nombre de usuario y contraseña.| GA | GA | |
| [Flujo de edición de perfiles](add-profile-editing-policy.md) | GA | GA | |
| [Autoservicio de restablecimiento de contraseña](add-password-reset-policy.md) | GA| GA| |
| [Forzar el restablecimiento de contraseñas](force-password-reset.md) | Vista previa | N/D | |
| [Registro e inicio de sesión por teléfono](phone-authentication-user-flows.md) | GA | GA | |

## <a name="oauth-20-application-authorization-flows"></a>Flujos de autorización de aplicaciones de OAuth 2.0

En la tabla siguiente se resumen los flujos de autenticación de aplicaciones de OAuth  2.0 y OpenId Connect que se pueden integrar con Azure AD B2C.

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
[Código de autorización](authorization-code-flow.md) | GA | GA | Permite a los usuarios iniciar sesión en aplicaciones web. La aplicación web recibe un código de autorización. El código de autorización se canjea para adquirir un token para llamar a las API web.|
[Código de autorización con PKCE](authorization-code-flow.md)| GA | GA | Permite a los usuarios iniciar sesión en aplicaciones móviles y de página única. La aplicación recibe un código de autorización mediante la clave de prueba para el intercambio de código (PKCE). El código de autorización se canjea para adquirir un token para llamar a las API web.  |
[Concesión de las credenciales de cliente](https://tools.ietf.org/html/rfc6749#section-4.4)| GA | GA | Permite acceder a los recursos hospedados en la Web mediante la identidad de una aplicación. Se usa principalmente para las interacciones entre servidores que se deben ejecutar en segundo plano, sin la interacción inmediata con un usuario.  <br />  <br />  Para usar esta característica en un inquilino de Azure AD B2C, use el punto de conexión de Azure AD del inquilino de Azure AD B2C. Para más información, vea el [flujo de credenciales de cliente de OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Este flujo no usa la configuración de [flujo de usuario o directiva personalizada](user-flow-overview.md) de Azure AD B2C. |
[Concesión de autorización de dispositivos](https://tools.ietf.org/html/rfc8628)| N/D | N/D | Permite a los usuarios iniciar sesión en dispositivos con limitaciones de entrada, como un televisor inteligente, dispositivo IoT o impresora.  |
[Flujo implícito](implicit-flow-single-page-application.md) | GA | GA |  Permite a los usuarios iniciar sesión en aplicaciones de página única. La aplicación obtiene los tokens directamente sin tener que realizar un intercambio de credenciales de servidor back-end.|
[En nombre de](../active-directory/develop/v2-oauth2-on-behalf-of-flow.md)| N/D | N/D | Una aplicación invoca un servicio o API web, que, a su vez, debe llamar a otro servicio o API web. <br />  <br /> Para que el servicio de nivel intermedio realice solicitudes autenticadas al servicio de nivel inferior, pase un token de *credencial de cliente* en el encabezado de autorización. Opcionalmente, puede incluir un encabezado personalizado con el token del usuario de Azure AD B2C.  |
[OpenId Connect](openid-connect.md) | GA | GA | OpenID Connect presenta el concepto de un token de identificador, que es un token de seguridad que permite al cliente comprobar la identidad del usuario. |
[Flujo híbrido de OpenID Connect](openid-connect.md) | GA | GA | Permite que una aplicación web recupere el token de identificador en la solicitud de autorización junto con un código de autorización.  |
[Credenciales de contraseña del propietario del recurso (ROPC)](add-ropc-policy.md) | Vista previa | Vista previa | Permite a una aplicación móvil iniciar la sesión del usuario al controlar directamente la contraseña. |

### <a name="oauth-20-options"></a>Opciones de OAuth 2.0

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
| [Redirección del inicio de sesión a un proveedor social](direct-signin.md#redirect-sign-in-to-a-social-provider) | GA | GA | Parámetro de cadena de consulta `domain_hint`. |
| [Rellenar previamente el nombre de inicio de sesión](direct-signin.md#prepopulate-the-sign-in-name) | GA | GA | Parámetro de cadena de consulta `login_hint`. |
| Inserción de JSON en el recorrido del usuario a través de `client_assertion`| N/D| Obsoleto |  |
| Inserción de JSON en el recorrido del usuario como [id_token_hint](id-token-hint.md) | N/D | GA | |
| [Paso del token del proveedor de identidades a la aplicación](idp-pass-through-user-flow.md)| Vista previa| Vista previa| Por ejemplo, de Facebook a la aplicación. |

## <a name="saml2-application-authentication-flows"></a>Flujos de autenticación de aplicaciones SAML2

En la tabla siguiente se resumen los flujos de autenticación de aplicaciones del Lenguaje de marcado de aserción de seguridad (SAML) que se pueden integrar con Azure AD B2C.

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
[Iniciado por SP](saml-service-provider.md) | N/D | GA | Enlaces POST y de redirección. |
[Iniciado por IDP](saml-service-provider-options.md#identity-provider-initiated-flow) | N/D | GA | Donde el proveedor de identidades de inicio es Azure AD B2C.  |

## <a name="user-experience-customization"></a>Personalización de la experiencia del usuario

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
| [Compatibilidad con varios lenguajes](localization.md)| GA | GA | |
| [Verificación de correo electrónico personalizado](custom-email-mailjet.md) | N/D | GA| |
| [Personalización de la interfaz de usuario con plantillas integradas](customize-ui.md) | GA| GA| |
| [Personalización de la interfaz de usuario con plantillas personalizadas](customize-ui-with-html.md) | GA| GA| Mediante plantillas HTML. |
| [JavaScript](javascript-and-page-layout.md) | GA | GA | |
| [Experiencia de inicio de sesión insertada](embedded-login.md) | N/D |  Vista previa| Mediante el elemento de marco en línea `<iframe>`. |
| [Complejidad de la contraseña](password-complexity.md) | GA | GA | |
| [Deshabilitación de la comprobación de correo electrónico](disable-email-verification.md) | GA|  GA| No se recomienda para entornos de producción. Deshabilitar la comprobación de correos electrónicos en el proceso de registro puede provocar que se reciban correos no deseados. |



## <a name="identity-providers"></a>Proveedores de identidades

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
|[AD FS](identity-provider-adfs.md) | N/D | GA | |
|[Amazon](identity-provider-amazon.md) | GA | GA | |
|[Apple](identity-provider-apple-id.md) | Vista previa | Vista previa | |
|[Azure AD (inquilino único)](identity-provider-azure-ad-single-tenant.md) | GA | GA | |
|[Azure AD (multiinquilino)](identity-provider-azure-ad-multi-tenant.md) | N/D  | GA | |
|[Azure AD B2C](identity-provider-azure-ad-b2c.md) | GA | GA | |
|[eBay](identity-provider-ebay.md) | N/D | Vista previa | |
|[Facebook](identity-provider-facebook.md) | GA | GA | |
|[GitHub](identity-provider-github.md) | GA | GA | |
|[Google](identity-provider-google.md) | GA | GA | |
|[ID.me](identity-provider-id-me.md) | GA | GA | |
|[LinkedIn](identity-provider-linkedin.md) | GA | GA | |
|[Cuenta Microsoft](identity-provider-microsoft-account.md) | GA | GA | |
|[QQ](identity-provider-qq.md) | Vista previa | GA | |
|[Salesforce](identity-provider-salesforce.md) | GA | GA | |
|[Salesforce (protocolo SAML)](identity-provider-salesforce-saml.md) | N/D | GA | |
|[Twitter](identity-provider-twitter.md) | GA | GA | |
|[WeChat](identity-provider-wechat.md) | Vista previa | GA | |
|[Weibo](identity-provider-weibo.md) | Vista previa | GA | |

## <a name="generic-identity-providers"></a>Proveedores de identidades genéricos

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
|[OAuth2](oauth2-technical-profile.md) | N/D | GA | Por ejemplo, [Google](identity-provider-google.md), [GitHub](identity-provider-github.md) y [Facebook](identity-provider-facebook.md).|
|[OAuth1](oauth1-technical-profile.md) | N/D | GA | Por ejemplo, [Twitter](identity-provider-twitter.md). |
|[OpenID Connect](openid-connect-technical-profile.md) | GA | GA | Por ejemplo, [Azure AD](identity-provider-azure-ad-single-tenant.md).  |
|[SAML2](identity-provider-generic-saml.md) | N/D | GA | Por ejemplo, [Salesforce](identity-provider-salesforce-saml.md) y [AD-FS].(identity-provider-adfs.md) |
| WSFED | N/D | N/D | |

### <a name="api-connectors"></a>Conectores de API

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
|[Conectores de API](api-connectors-overview.md) | Vista previa | GA | |
|[Protección con autenticación básica](secure-rest-api.md#http-basic-authentication) | Vista previa | GA | |
|[Protección con autenticación de certificados de cliente](secure-rest-api.md#https-client-certificate-authentication) | Vista previa | GA | |
|[Protección con la autenticación de portador de OAuth2](secure-rest-api.md#oauth2-bearer-authentication) | N/D | GA | |
|[Protección con autenticación de clave de API](secure-rest-api.md#api-key-authentication) | N/D | GA | |


## <a name="custom-policy-features"></a>Características de la directiva personalizada

### <a name="session-management"></a>Administración de sesiones

| Característica |  Directiva personalizada | Notas |
| ------- | :--: | ----- |
| [Proveedor predeterminado de sesión de SSO](custom-policy-reference-sso.md#defaultssosessionprovider) | GA |  |
| [Proveedor externo de sesión de inicio de sesión](custom-policy-reference-sso.md#externalloginssosessionprovider) | GA |  |
| [Proveedor de sesión de SSO de SAML](custom-policy-reference-sso.md#samlssosessionprovider) | GA |  |
| [Proveedor de sesión de SSO de OAuth](custom-policy-reference-sso.md#oauthssosessionprovider)  | GA|  |
| [Cierre de sesión único](session-behavior.md#sign-out)  |  Vista previa |  |

### <a name="components"></a>Componentes

| Característica | Directiva personalizada | Notas |
| ------- | :--: | ----- |
| [Autenticación de factor de teléfono](phone-factor-technical-profile.md) | GA |  |
| [Autenticación de MFA de Azure AD](multi-factor-auth-technical-profile.md) | Vista previa |  |
| [Contraseña de un solo uso](one-time-password-technical-profile.md) | GA |  |
| [Azure Active Directory](active-directory-technical-profile.md) como directorio local | GA |  |
| [Validaciones de predicado](predicates.md) | GA | Por ejemplo, la complejidad de la contraseña. |
| [Controles de presentación](display-controls.md) | GA |  |

### <a name="developer-interface"></a>Interfaz del desarrollador

| Característica | Directiva personalizada | Notas |
| ------- | :--: | ----- |
| Portal de Azure | GA |   |
| [Registros de recorridos del usuario de Application Insights](troubleshoot-with-application-insights.md) | Vista previa | Se usa para solucionar problemas durante el desarrollo.  |
| [Registros de eventos de Application Insights](analytics-with-application-insights.md) | Vista previa | Se usa para supervisar flujos de usuario en producción. |

## <a name="responsibilities-of-custom-policy-feature-set-developers"></a>Responsabilidades de los desarrolladores de conjunto de características de directivas personalizadas

La configuración manual de directivas concede un acceso de nivel inferior a la plataforma subyacente de Azure AD B2C y da como resultado la creación de un marco de confianza único. Las múltiples permutaciones de proveedores de identidades personalizados, las relaciones de confianza, las integraciones con servicios externos y los flujos de trabajo paso a paso exigen un enfoque de diseño y configuración más metódico.

Los desarrolladores que utilizan el conjunto de características de directivas personalizadas deben cumplir las siguientes directrices:

- Conocer el lenguaje de configuración de las directivas personalizadas y la administración de claves/secretos. Para más información, consulte [TrustFrameworkPolicy](trustframeworkpolicy.md).
- Tomar posesión de escenarios e integraciones personalizadas. Documentar su trabajo e informar a la organización del sitio activo.
- Realizar pruebas metódicas de los escenarios.
- Siga los procedimientos recomendados de desarrollo de software y almacenamiento provisional. Se recomienda un mínimo de un entorno de desarrollo y pruebas.
- Mantenerse informado sobre los nuevos desarrollos de los proveedores de identidad y los servicios con los que está integrado. Por ejemplo, realizar un seguimiento de cambios en los secretos y los cambios programados y no programados en el servicio.
- Configurar la supervisión activa y supervisar la capacidad de respuesta de los entornos de producción. Para más información sobre la integración con Application Insights, vea [Azure Active Directory B2C: Recopilación de registros](analytics-with-application-insights.md).
- Mantenga actualizadas las direcciones de correo electrónico de contacto y responda a los correos mensajes de correo electrónico del equipo de Microsoft.
- Realizar las acciones pertinentes cuando se lo indique el equipo del sitio activo de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- Consulte las [Operaciones de Microsoft Graph disponibles para Azure AD B2C](microsoft-graph-operations.md).
- Obtenga más información sobre las [directivas personalizadas y las diferencias con los flujos de usuario](custom-policy-overview.md).
