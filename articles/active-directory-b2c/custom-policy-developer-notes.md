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
ms.date: 04/30/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: e6261699166e0157750fc691bc0c1726d8cefd50
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108324066"
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


## <a name="protocols-and-authorization-flows"></a>Protocolos y flujos de autorización

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
|[Código de autorización OAuth2](authorization-code-flow.md) | GA | GA |
|[Código de autorización OAuth2 con PKCE](authorization-code-flow.md)| GA | GA | Clientes públicos y aplicaciones de página única. |
|[Flujo implícito de OAuth2](implicit-flow-single-page-application.md) | GA | GA | |
|[Credenciales de contraseña del propietario del recurso OAuth2](add-ropc-policy.md) | Vista previa | Vista previa | |
|OAuth1 | N/D | N/D | No compatible. |
|[OpenId Connect](openid-connect.md) | GA | GA | |
|[SAML2](saml-service-provider.md) | N/D | GA | Enlaces POST y de redirección. |
| WSFED | N/D | N/D | No compatible. |

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

### <a name="application-and-azure-ad-b2c-integration"></a>Integración de aplicaciones y Azure AD B2C

|Característica  |Flujo de usuario  |Directiva personalizada  |Notas  |
|---------|:---------:|:---------:|---------|
| [Redirección del inicio de sesión a un proveedor social](direct-signin.md#redirect-sign-in-to-a-social-provider) | GA | GA | Parámetro de cadena de consulta `domain_hint`. |
| [Rellenar previamente el nombre de inicio de sesión](direct-signin.md#prepopulate-the-sign-in-name) | GA | GA | Parámetro de cadena de consulta `login_hint`. |
| Inserción de JSON en el recorrido del usuario a través de `client_assertion`| N/D| Obsoleto |  |
| Inserción de JSON en el recorrido del usuario como [id_token_hint](id-token-hint.md) | N/D | GA | |
| [Paso del token del proveedor de identidades a la aplicación](idp-pass-through-user-flow.md)| Vista previa| Vista previa| Por ejemplo, de Facebook a la aplicación. |


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
