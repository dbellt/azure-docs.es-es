---
title: Novedades Notas de la versión de Azure Active Directory | Microsoft Docs
description: Obtenga información acerca de las novedades de Azure Active Directory, como son las notas de la versión más recientes, los problemas conocidos, las correcciones de errores, las funcionalidades en desuso y los próximos cambios.
services: active-directory
author: ajburnle
manager: daveba
featureFlags:
- clicktale
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 4/30/2021
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7b07e27139ce4a09c5e9964401c9e2b533a94c96
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109838994"
---
# <a name="whats-new-in-azure-active-directory"></a>¿Cuáles son las novedades de Azure Active Directory?

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue la dirección URL `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Active+Directory%22&locale=en-us` en el ![icono del lector de fuentes icono RSS](./media/whats-new/feed-icon-16x16.png) lector de fuentes.

En Azure AD se realizan mejoras de forma continua. Para mantenerse al día de los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas
- Planes de cambios

Esta página se actualiza mensualmente, por lo que se recomienda visitarla con frecuencia. Si busca elementos que tengan más de seis meses, puede encontrarlos en el [Archivo de novedades de Azure Active Directory](whats-new-archive.md).

---

## <a name="april-2021"></a>Abril de 2021

### <a name="bug-fixed---azure-ad-will-no-longer-double-encode-the-state-parameter-in-responses"></a>Error corregido: Azure AD ya no codificará dos veces el parámetro de estado de las respuestas

**Tipo:** Corregido  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Azure AD identificó, probó y publicó la corrección para un error en la respuesta `/authorize` a una aplicación cliente.  Azure AD codificaba de manera incorrecta la URL del parámetro `state` dos veces al enviar las respuestas de vuelta al cliente.  Esto puede generar que la aplicación cliente rechace la solicitud debido a una discrepancia en los parámetros de estado. [Más información](../develop/reference-breaking-changes.md#bug-fix-azure-ad-will-no-longer-url-encode-the-state-parameter-twice). 

---

### <a name="users-can-only-create-security-and-microsoft-365-groups-in-azure-portal-being-deprecated"></a>La restricción para que los usuarios solo puedan crear grupos de seguridad y de Microsoft 365 en Azure Portal está en desuso

**Tipo:** Plan de cambio  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Directorio
 
Los usuarios ya no estarán limitados a crear grupos de seguridad y de Microsoft 365 solo en Azure Portal. La configuración nueva permitirá que los usuarios creen grupos de seguridad en Azure Portal, PowerShell y en la API. Los usuarios deberán comprobar y actualizar la configuración nueva. [Más información](../enterprise-users/groups-self-service-management.md).

---

### <a name="public-preview---external-identities-self-service-sign-up-in-aad-using-email-one-time-passcode-accounts"></a>Versión preliminar pública: registro de autoservicio de identidades externas en AAD mediante cuentas de código de acceso de un solo uso de correo electrónico

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Los usuarios externos ahora pueden usar las cuentas de código de acceso de un solo uso de correo electrónico para registrarse o iniciar sesión en las aplicaciones de línea de negocio y de Azure AD de Microsoft. [Más información](../external-identities/one-time-passcode.md).

---

### <a name="general-availability---external-identities-self-service-sign-up"></a>Disponibilidad general: registro de autoservicio de External Identities

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
El registro de autoservicio para usuarios externos ahora está en disponibilidad general. Con esta característica nueva, los usuarios externos ahora pueden autoregistrarse en una aplicación. 

Puede crear experiencias personalizadas para estos usuarios externos, incluida la recopilación de información sobre los usuarios durante el proceso de registro y la posibilidad de permitir proveedores de identidades externos como Facebook y Google. También puede integrarse con proveedores de servicios en la nube de terceros para diversas funcionalidades, como la verificación de identidad o la aprobación de usuarios. [Más información](../external-identities/self-service-sign-up-overview.md).
 
---

### <a name="general-availability---azure-ad-b2c-phone-sign-up-and-sign-in-using-built-in-policy"></a>Disponibilidad general: registro e inicio de sesión mediante teléfono en Azure AD B2C con una directiva integrada

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
El registro e inicio de sesión mediante teléfono en B2C con una directiva integrada permiten a los administradores de TI y desarrolladores de organizaciones dejar que sus usuarios finales inicien sesión y se registren con un número de teléfono en los flujos de usuario. Con esta característica, los vínculos de declinación de responsabilidades, como la directiva de privacidad y las condiciones de uso, se pueden personalizar y mostrar en la página antes de que el usuario final siga recibiendo el código de acceso de un solo uso a través de un mensaje de texto. [Más información](../../active-directory-b2c/phone-authentication-user-flows.md).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---april-2021"></a>Aplicaciones federadas nuevas disponibles en la galería de aplicaciones de Azure AD, abril de 2021

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En abril de 2021, agregamos 31 aplicaciones nuevas a la galería de aplicaciones con compatibilidad de federación

[Zii Travel Azure AD Connect](http://ziitravel.com/), [Cerby](../saas-apps/cerby-tutorial.md), [Selflessly](https://app.selflessly.io/sign-in), [Apollo CX](https://apollo.cxlabs.de/sso/aad), [Pedagoo](https://account.pedagoo.com/), [Measureup](https://account.measureup.com/), [Wistec Education](https://wisteceducation.fi/login/index.php), [ProcessUnity](../saas-apps/processunity-tutorial.md), [Cisco Intersight](../saas-apps/cisco-intersight-tutorial.md), [Codility](../saas-apps/codility-tutorial.md), [H5mag](https://account.h5mag.com/auth/request-access/ms365), [Check Point Identity Awareness](../saas-apps/check-point-identity-awareness-tutorial.md), [Jarvis](https://jarvis.live/login), [desknet's NEO](../saas-apps/desknets-neo-tutorial.md), [SDS & Chemical Information Management](../saas-apps/sds-chemical-information-management-tutorial.md), [Wúru App](../saas-apps/wuru-app-tutorial.md), [Holmes](../saas-apps/holmes-tutorial.md), [Tide Multi Tenant](https://gallery.tideapp.co.uk/), [Telenor](https://admin.smartansatt.telenor.no/), [Yooz US](https://us1.getyooz.com/?kc_idp_hint=microsoft), [Mooncamp](https://app.mooncamp.com/#/login), [inwise SSO](https://app.inwise.com/defaultsso.aspx), [Ecolab Digital Solutions](https://ecolabb2c.b2clogin.com/account.ecolab.com/oauth2/v2.0/authorize?p=B2C_1A_Connect_OIDC_SignIn&client_id=01281626-dbed-4405-a430-66457825d361&nonce=defaultNonce&redirect_uri=https://jwt.ms&scope=openid&response_type=id_token&prompt=login), [Taguchi Digital Marketing System](https://login.taguchi.com.au/), [XpressDox EU Cloud](https://test.xpressdox.com/Authentication/Login.aspx), [EZSSH](https://docs.keytos.io/getting-started/registering-a-new-tenant/registering_app_in_tenant/), [EZSSH Client](https://portal.ezssh.io/signup), [Verto 365](https://www.vertocloud.com/Login/), [KPN Grip](https://www.grip-on-it.com/), [AddressLook](https://portal.bbsonlineservices.net/Manage/AddressLook), [Cornerstone Single Sign-On](../saas-apps/cornerstone-ondemand-tutorial.md)

También puede consultar la documentación de todas ellas aquí: https://aka.ms/AppsTutorial.

Para incluir su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---april-2021"></a>Conectores de aprovisionamiento nuevos disponibles en la galería de aplicaciones de Azure AD, abril de 2021

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Bentley - Automatic User Provisioning](../saas-apps/bentley-automatic-user-provisioning-tutorial.md)
- [Boxcryptor](../saas-apps/boxcryptor-provisioning-tutorial.md)
- [Inicio de sesión único de BrowserStack](../saas-apps/browserstack-single-sign-on-provisioning-tutorial.md)
- [Eletive](../saas-apps/eletive-provisioning-tutorial.md)
- [Jostle](../saas-apps/jostle-provisioning-tutorial.md)
- [Olfeo SAAS](../saas-apps/olfeo-saas-provisioning-tutorial.md)
- [Proware](../saas-apps/proware-provisioning-tutorial.md)
- [Segmento](../saas-apps/segment-provisioning-tutorial.md)

Para más información sobre cómo proteger mejor la organización a través del aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
 
---

### <a name="introducing-new-versions-of-page-layouts-for-b2c"></a>Introducción de versiones nuevas de diseños de página para B2C

**Tipo:** Característica modificada  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
Los [diseños de página](../../active-directory-b2c/page-layout.md) para escenarios de B2C en Azure AD B2C se actualizaron a fin de reducir los riesgos de seguridad al introducir las versiones nuevas de jQuery y Handlebars JS.
 
---

### <a name="updates-to-sign-in-diagnostic"></a>Actualizaciones del diagnóstico de inicio de sesión

**Tipo:** Característica modificada  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
La cobertura de escenarios de la herramienta de diagnóstico de inicio de sesión ha aumentado. 

Con esta actualización, los escenarios siguientes relacionados con eventos ahora se incluirán en los resultados del diagnóstico de inicio de sesión: 
- Eventos de problemas de configuración de aplicaciones empresariales.
- Eventos del proveedor de servicios de aplicaciones empresariales (lado de la aplicación).
- Eventos de credenciales incorrectas. 

Estos resultados mostrarán detalles contextuales y pertinentes sobre el evento y las acciones que se deben realizar para resolver estos problemas. Además, en escenarios en los que no tenemos diagnósticos contextuales profundos, el diagnóstico de inicio de sesión presentará contenido más descriptivo sobre el evento de error.

Para más información, consulte [¿Qué es el diagnóstico de inicio de sesión en Azure AD?](../reports-monitoring/overview-sign-in-diagnostics.md)

---
### <a name="azure-ad-connect-cloud-sync-general-availability-refresh"></a>Actualización de la disponibilidad general de Azure AD Connect Cloud Sync 
**Tipo:** Característica modificada  
**Categoría del servicio:** Azure AD Connect Cloud Sync **Funcionalidad del producto:** Directorio

Azure AD Connect Cloud Sync ahora tiene un agente actualizado (n.° de versión 1.1.359). Para más información sobre las actualizaciones del agente, incluidas las correcciones de errores, consulte el [historial de versiones](../cloud-sync/reference-version-history.md). Con el agente actualizado, los clientes de la sincronización en la nube pueden usar cmdlets de GMSA para establecer y restablecer su permiso de gMSA en un nivel detallado. Además, cambiamos el límite de sincronización de los miembros mediante el filtrado del ámbito de grupo de 1499 a 50 000 miembros. 

Consulte el [generador de expresiones](../cloud-sync/how-to-expression-builder.md#deploy-the-expression) recién disponible para la sincronización en la nube, que lo ayuda a crear expresiones complejas y expresiones sencillas cuando realiza transformaciones de valores de atributo de AD a Azure AD mediante la asignación de atributos.

---

## <a name="march-2021"></a>Marzo de 2021

### <a name="guidance-on-how-to-enable-support-for-tls-12-in-your-environment-in-preparation-for-upcoming-azure-ad-tls-1011-deprecation"></a>Instrucciones sobre cómo habilitar la compatibilidad con TLS 1.2 en su entorno, en preparación para el próximo desuso de Azure AD TLS 1.0/1.1.

**Tipo:** Plan de cambio  
**Categoría del servicio:** N/D  
**Funcionalidad del producto:** Estándares

Azure Active Directory dejará de usar los siguientes protocolos en las regiones de todo el mundo de Azure Active Directory a partir del 30 de junio de 2021:


- TLS 1.0
- TLS 1.1
- Conjunto de cifrado 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Los entornos afectados son:

- Nube comercial de Azure
- Office 365 GCC y WW

Para más información, consulte [Habilitar la compatibilidad con TLS 1.2 en el entorno para el desuso de TLS 1.1 y 1.0 de Azure AD](/troubleshoot/azure/active-directory/enable-support-tls-environment).

---

### <a name="public-preview---azure-ad-entitlement-management-now-supports-multi-geo-sharepoint-online"></a>Versión preliminar pública: la administración de derechos de Azure AD ahora admite SharePoint Online con múltiples geografías

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos
 
En el caso de las organizaciones que usan SharePoint Online con múltiples geografías, ahora puede incluir sitios de varios entornos geográficos específicos en los paquetes de acceso de administración de derechos. [Más información](../governance/entitlement-management-catalog-create.md#add-a-multi-geo-sharepoint-site-preview).

---

### <a name="public-preview---restore-deleted-apps-from-app-registrations"></a>Versión preliminar pública: restauración de aplicaciones eliminadas de Registros de aplicaciones

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
Los clientes ahora pueden ver, restaurar y quitar permanentemente los registros de aplicaciones eliminados de Azure Portal. Esto solo se aplica a las aplicaciones asociadas a un directorio, no a las aplicaciones de un cuenta personal de Microsoft. [Más información](../develop/howto-restore-app.md).
 
---

### <a name="public-preview---new-user-action-in-conditional-access-for-registering-or-joining-devices"></a>Versión preliminar pública: nueva "acción del usuario" en acceso condicional para registrar o unir dispositivos

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
 Hay disponible una nueva acción de usuario denominada "Registrar o unir dispositivos" en el acceso condicional. Esta acción del usuario permite controlar las directivas de Multi-Factor Authentication (MFA) para el registro de dispositivos de Azure AD. 

Actualmente, esta acción del usuario solo permite habilitar MFA como un control cuando los usuarios registran o unen dispositivos a Azure AD. Otros controles que dependen del registro de dispositivos de Azure AD, o que no se aplican, se deshabilitan con esta acción del usuario. [Más información](../conditional-access/concept-conditional-access-cloud-apps.md#user-actions). 
 
---

### <a name="public-preview---optimize-connector-groups-to-use-the-closest-application-proxy-cloud-service"></a>Versión preliminar pública: optimización de los grupos de conectores para usar el servicio en la nube de Application Proxy más cercano

**Tipo:** Nueva característica  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso
 
Con esta nueva funcionalidad, los grupos de conectores se pueden asignar al servicio de Application Proxy regional más cercano en el que se hospeda una aplicación. Esto puede mejorar el rendimiento de la aplicación en escenarios en los que las aplicaciones se hospedan en regiones distintas de la región de inicio del inquilino. [Más información](../app-proxy/application-proxy-network-topology.md#optimize-connector-groups-to-use-closest-application-proxy-cloud-service-preview). 
 
---

### <a name="public-preview---external-identities-self-service-sign-up-in-aad-using-email-one-time-passcode-accounts"></a>Versión preliminar pública: registro de autoservicio de identidades externas en AAD mediante cuentas de código de acceso de un solo uso de correo electrónico

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C

Ahora, los usuarios externos podrán usar las cuentas de código de acceso de un solo uso de correo electrónico para registrarse en las aplicaciones de Azure AD de Microsoft y de LOB. [Más información](../external-identities/one-time-passcode.md).

---

### <a name="public-preview---availability-of-ad-fs-sign-ins-in-azure-ad"></a>Versión preliminar pública: disponibilidad de inicios de sesión de AD FS en Azure AD

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Supervisión e informes
 
La actividad de inicio de sesión de AD FS ahora se puede integrar con los informes de actividad de Azure AD, lo que proporciona una vista unificada de la infraestructura de identidades híbridas. Con el informe de inicio de sesión de Azure AD, Log Analytics y los libros de Azure Monitor, es posible realizar un análisis exhaustivo de los escenarios de inicio de sesión de AAD y AD FS como bloqueos de cuentas de AD FS, intentos incorrectos de contraseña y aumentos de los intentos de inicio de sesión inesperados.

Para más información, consulte [Inicios de sesión de AD FS en Azure AD con Connect Health (versión preliminar)](../hybrid/how-to-connect-health-ad-fs-sign-in.md).

---

### <a name="general-availability---staged-rollout-to-cloud-authentication"></a>Disponibilidad general: lanzamiento preconfigurado para la autenticación en la nube

**Tipo:** Nueva característica  
**Categoría del servicio:** AD Connect  
**Funcionalidad del producto:** Autenticación de usuarios
 
El lanzamiento preconfigurado en la autenticación en la nube ya está disponible con carácter general. La característica del lanzamiento preconfigurado permite probar de forma selectiva grupos de usuarios con métodos de autenticación en la nube, como la autenticación de paso a través (PTA) o la sincronización de hash de contraseña (PHS). Mientras tanto, todos los demás usuarios de los dominios federados siguen usando los servicios de federación, como AD FS o cualquier otro servicio de federación para autenticar a los usuarios. [Más información](../hybrid/how-to-connect-staged-rollout.md).

---

### <a name="general-availability---user-type-attribute-can-now-be-updated-in-the-azure-admin-portal"></a>Disponibilidad general: el atributo de tipo de usuario ahora se puede actualizar en el portal de administración de Azure.

**Tipo:** Nueva característica  
**Categoría de servicio:** Experiencia y administración de usuarios  
**Funcionalidad del producto:** User Management
 
Los clientes ahora pueden actualizar el tipo de usuario de Azure AD cuando actualicen su información de perfil de usuario desde el portal de administración de Azure. También se puede actualizar el tipo de usuario en Microsoft Graph. Para más información, consulte [Adición o actualización de la información de perfil de un usuario mediante Azure Active Directory](active-directory-users-profile-azure-portal.md).
 
---

### <a name="general-availability---replica-sets-for-azure-active-directory-domain-services"></a>Disponibilidad general: conjuntos de réplicas para Azure Active Directory Domain Services

**Tipo:** Nueva característica  
**Categoría del servicio:** Azure AD Domain Services  
**Funcionalidad del producto:** Azure AD Domain Services
 
La funcionalidad de los conjuntos de réplicas en Azure AD DS ahora está disponible con carácter general. [Más información](../../active-directory-domain-services/concepts-replica-sets.md).
 
---

### <a name="general-availability---collaborate-with-your-partners-using-email-one-time-passcode-in-the-azure-government-cloud"></a>Disponibilidad general: colaboración con los asociados mediante el código de acceso de un solo uso de correo electrónico en la nube de Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Ahora, las organizaciones de la nube de Microsoft Azure Government pueden permitir a sus invitados canjear las invitaciones por el código de acceso de un solo uso de correo electrónico. Esto garantiza que todos los usuarios invitados que no tengan cuentas de Azure AD, Microsoft o Gmail en la nube de Azure Government puedan colaborar con los asociados solicitando y escribiendo un código temporal para iniciar sesión en los recursos compartidos. [Más información](../external-identities/one-time-passcode.md#note-for-azure-us-government-customers).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---march-2021"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, marzo de 2021

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En marzo de 2021, agregamos 37 aplicaciones nuevas a la galería de aplicaciones con compatibilidad de federación:

[Bambuser Live Video Shopping](https://lcx.bambuser.com/), [DeepDyve Inc](https://www.deepdyve.com/azure-sso), [Moqups](../saas-apps/moqups-tutorial.md), [RICOH Spaces Mobile](https://ricohspaces.app/welcome), [Flipgrid](https://auth.flipgrid.com/), [hCaptcha Enterprise](../saas-apps/hcaptcha-enterprise-tutorial.md), [SchoolStream ASA](https://jsd.schoolstreamk12.com/ASA/ASAlogin.aspx), [TransPerfect GlobalLink Dashboard](../saas-apps/transperfect-globallink-dashboard-tutorial.md), [SimplificaCI](https://app.simplificaci.com.br/), [Thrive LXP](../saas-apps/thrive-lxp-tutorial.md), [Lexonis TalentScape](../saas-apps/lexonis-talentscape-tutorial.md), [Exium](../saas-apps/exium-tutorial.md), [Sapient](../saas-apps/sapient-tutorial.md), [TrueChoice](../saas-apps/truechoice-tutorial.md), [RICOH Spaces](https://ricohspaces.app/welcome), [Saba Cloud](../saas-apps/learning-at-work-tutorial.md), [Acunetix 360](../saas-apps/acunetix-360-tutorial.md), [Exceed.ai](../saas-apps/exceed-ai-tutorial.md), [GitHub Enterprise Managed User](../saas-apps/github-enterprise-managed-user-tutorial.md), [Enterprise Vault.cloud para Outlook](https://login.microsoftonline.com/common/oauth2/v2.0/authorize?response_type=id_token&scope=openid%20profile%20User.Read&client_id=7176efe5-e954-4aed-b5c8-f5c85a980d3a&nonce=4b9e1981-1bcb-4938-a283-86f6931dc8cb), [Smartlook](../saas-apps/smartlook-tutorial.md), [Accenture Academy](../saas-apps/accenture-academy-tutorial.md), [Onshape](../saas-apps/onshape-tutorial.md), [Tradeshift](../saas-apps/tradeshift-tutorial.md), [JuriBlox](../saas-apps/juriblox-tutorial.md), [SecurityStudio](../saas-apps/securitystudio-tutorial.md), [ClicData](https://app.clicdata.com/), [Evergreen](../saas-apps/evergreen-tutorial.md), [Patchdeck](https://patchdeck.com/ad_auth/authenticate/), [FAX.PLUS](../saas-apps/fax.plus-tutorial.md), [ValidSign](../saas-apps/validsign-tutorial.md), [AWS Single Sign-on](../saas-apps/aws-single-sign-on-tutorial.md), [Nura Space](https://dashboard.nuraspace.com/login), [Broadcom DX SaaS](../saas-apps/broadcom-dx-saas-tutorial.md), [Interplay Learning](https://skilledtrades.interplaylearning.com/#login), [SendPro Enterprise](../saas-apps/sendpro-enterprise-tutorial.md), [FortiSASE SIA](../saas-apps/fortisase-sia-tutorial.md)

También puede consultar la documentación de todas ellas aquí: https://aka.ms/AppsTutorial.

Para incluir su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---march-2021"></a>Nuevos conectores de aprovisionamiento disponibles en la galería de aplicaciones de Azure AD, marzo de 2021

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Inicio de sesión único de AWS](../saas-apps/aws-single-sign-on-provisioning-tutorial.md)
- [Bpanda](../saas-apps/bpanda-provisioning-tutorial.md)
- [Britive](../saas-apps/britive-provisioning-tutorial.md)
- [GitHub Enterprise Managed User](../saas-apps/github-enterprise-managed-user-provisioning-tutorial.md)
- [Grammarly](../saas-apps/grammarly-provisioning-tutorial.md)
- [LogicGate](../saas-apps/logicgate-provisioning-tutorial.md)
- [SecureLogin](../saas-apps/secure-login-provisioning-tutorial.md)
- [TravelPerk](../saas-apps/travelperk-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
 
---

### <a name="introducing-ms-graph-api-for-company-branding"></a>Introducción a MS Graph API para la personalización de marca de empresa

**Tipo:** Característica modificada  
**Categoría del servicio:** MS Graph  
**Funcionalidad del producto:** B2B/B2C

[MS Graph API para la personalización de marca de empresa](/graph/api/resources/organizationalbrandingproperties) está disponible para la experiencia de inicio de sesión Azure AD o de Microsoft 365 para permitir la administración de los parámetros de personalización de marca mediante programación.

---

### <a name="general-availability---header-based-authentication-sso-with-application-proxy"></a>Disponibilidad general: inicio de sesión único de autenticación basada en encabezados con proxy de aplicación

**Tipo:** Característica modificada  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso
 
Ahora está disponible con carácter general la compatibilidad nativa del proxy de aplicaciones de Azure AD con la autenticación basada en encabezados. Con esta característica, puede configurar los atributos de usuario necesarios como encabezados HTTP para la aplicación sin necesidad de componentes adicionales para la implementación. [Más información](../manage-apps/application-proxy-configure-single-sign-on-with-headers.md).

---

### <a name="two-way-sms-for-mfa-server-is-no-longer-supported"></a>Ya no se admite el servidor de SMS bidireccional para MFA

**Tipo:** Obsoleto  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

El servidor SMS bidireccional para el servidor de MFA se dejó de utilizar originalmente en 2018, y ya no tiene soporte técnico desde el 24 de febrero de 2021. Los administradores deben habilitar otro método para los usuarios que siguen usando el SMS bidireccional.

Se enviaron notificaciones por correo electrónico y de Service Health en Azure Portal a los administradores afectados el 8 de diciembre de 2020 y el 28 de enero de 2021. Las alertas se dirigieron a los roles RBAC de propietario, copropietario, administrador y administrador de servicios vinculados a las suscripciones. [Más información](../authentication/how-to-authentication-two-way-sms-unsupported.md).
 
---
 
## <a name="february-2021"></a>Febrero de 2021

### <a name="email-one-time-passcode-authentication-on-by-default-starting-october-2021"></a>La autenticación con código de acceso de un solo uso por correo electrónico estará activada de manera predeterminada a partir de octubre de 2021

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 

A partir del 31 de octubre de 2021, la [autenticación con código de acceso de un solo uso por correo electrónico](../external-identities/one-time-passcode.md) de Microsoft Azure Active Directory se convertirá en el método predeterminado para invitar a cuentas e inquilinos en escenarios de colaboración B2B. En ese momento, Microsoft ya no permitirá el canje de invitaciones mediante cuentas de Azure Active Directory no administradas. 

---

### <a name="unrequested-but-consented-permissions-will-no-longer-be-added-to-tokens-if-they-would-trigger-conditional-access"></a>Los permisos no solicitados pero consentidos ya no se agregarán a los tokens si esto supone desencadenar el acceso condicional

**Tipo:** Plan de cambio  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** plataforma
 
Actualmente, las aplicaciones que usan [permisos dinámicos](../develop/v2-permissions-and-consent.md#requesting-individual-user-consent) reciben todos los permisos a los que se les consiente el acceso. Esto incluye las aplicaciones que no se han solicitado, incluso si desencadenan el acceso condicional. Por ejemplo, esto puede dar lugar a que una aplicación que solicite únicamente el permiso `user.read` y tenga también el consentimiento para el permiso `files.read` deba pasar el acceso condicional asignado al permiso `files.read`. 

Para reducir el número de mensajes de acceso condicional innecesarios, Azure AD está cambiando la forma en que se proporcionan los ámbitos no solicitados a las aplicaciones. Las aplicaciones solo desencadenarán el acceso condicional correspondiente al permiso que solicitan explícitamente. Para obtener más información, consulte [Novedades en la autenticación](../develop/reference-breaking-changes.md#conditional-access-will-only-trigger-for-explicitly-requested-scopes).
 
---
 
### <a name="public-preview---use-a-temporary-access-pass-to-register-passwordless-credentials"></a>Versión preliminar pública: uso de un pase de acceso temporal para registrar credenciales sin contraseña

**Tipo:** Nueva característica  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad

El pase de acceso temporal es un código de acceso de tiempo limitado que sirve como credencial segura y permite la incorporación de credenciales sin contraseña y su recuperación cuando un usuario ha perdido u olvidado su factor de autenticación sólida (por ejemplo, una clave de seguridad FIDO2 o la aplicación Microsoft Authenticator) y debe iniciar sesión para registrar nuevos métodos de autenticación sólida. [Más información](../authentication/howto-authentication-temporary-access-pass.md).

---

### <a name="public-preview---keep-me-signed-in-kmsi-in-next-generation-of-user-flows"></a>Versión preliminar pública: mantenimiento de la sesión iniciada (KMSI) en la próxima generación de los flujos de usuario

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

La próxima generación de flujos de usuario de B2C ahora es compatible con la funcionalidad de [mantenimiento de la sesión iniciada (KMSI)](../../active-directory-b2c/session-behavior.md?pivots=b2c-custom-policy#enable-keep-me-signed-in-kmsi) que permite a los clientes ampliar la duración de la sesión de los usuarios de sus aplicaciones web y nativas mediante el uso de una cookie persistente.  La característica mantiene la sesión activa incluso cuando el usuario cierra y vuelve a abrir el explorador y la revoca cuando el usuario cierra sesión.

---

### <a name="public-preview---external-identities-self-service-sign-up-in-aad-using-msa-accounts"></a>Versión preliminar pública: registro de autoservicio de identidades externas en AAD mediante cuentas MSA

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Ahora los usuarios externos podrán usar cuentas de Microsoft para iniciar sesión en aplicaciones propias y de línea de negocio de Azure AD. [Más información](../external-identities/self-service-sign-up-overview.md).

---

### <a name="public-preview---reset-redemption-status-for-a-guest-user"></a>Versión preliminar pública: restablecimiento del estado de canje de un usuario invitado

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Ahora los clientes pueden volver a invitar a los usuarios externos invitados existentes a restablecer su estado de canje, lo que permite que la cuenta de usuario invitado se mantenga sin que se pierda ningún acceso. [Más información](../external-identities/reset-redemption-status.md).
 
---

### <a name="public-preview---synchronization-provisioning-apis-now-support-application-permissions"></a>Versión preliminar pública: las API de sincronización (aprovisionamiento) ahora admiten permisos de aplicación

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Ahora los clientes pueden usar application.readWrite.ownedby como permiso de aplicación para llamar a las API de sincronización. Tenga en cuenta que esta opción solo se admite para el aprovisionamiento de Azure AD a aplicaciones de terceros (por ejemplo, AWS, Data Bricks, etc.). Actualmente no se admite en el aprovisionamiento de recursos humanos (WorkDay/SuccessFactors) ni en la sincronización en la nube (AD a Azure AD). [Más información](/graph/api/resources/provisioningobjectsummary?view=graph-rest-beta&preserve-view=true).
 
---

### <a name="general-availability---authentication-policy-administrator-built-in-role"></a>Disponibilidad general: rol integrado de administrador de directivas de autenticación

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Los usuarios con este rol pueden configurar la directiva de métodos de autenticación, la configuración de MFA para todos los inquilinos y la directiva de protección de contraseñas. Este rol concede permiso para administrar la configuración de protección de contraseñas: configuraciones de bloqueo inteligente y actualización de la lista de contraseñas prohibidas personalizadas. [Más información](../roles/permissions-reference.md#authentication-policy-administrator).

---

### <a name="general-availability---user-collections-on-my-apps-are-available-now"></a>Disponibilidad general: ahora las colecciones de los usuarios están disponibles en Mis aplicaciones

**Tipo:** Nueva característica  
**Categoría del servicio:** Mis aplicaciones  
**Funcionalidad del producto:** Experiencias de usuario final
 
Ahora los usuarios pueden crear sus propias agrupaciones de aplicaciones en el iniciador Mis aplicaciones. También pueden reordenar y ocultar colecciones compartidas con ellos por su administrador. [Más información](../user-help/my-apps-portal-user-collections.md).

---

### <a name="general-availability---autofill-in-authenticator"></a>Disponibilidad general: autorrellenado en Authenticator

**Tipo:** Nueva característica  
**Categoría de servicio:** Aplicación Microsoft Authenticator  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Microsoft Authenticator proporciona funciones de autenticación multifactor (MFA) y de administración de cuentas, y ahora también autorellenará las contraseñas de los sitios y las aplicaciones que los usuarios visitan en sus dispositivos móviles (iOS y Android). 

Para usar el autorrellenado en Authenticator, los usuarios deben agregar sus cuenta de Microsoft personales en esta aplicación y usarla para sincronizar las contraseñas. Por el momento, no se pueden usar las cuentas profesionales ni educativas para sincronizar contraseñas. [Más información](../user-help/user-help-auth-app-faq.md#autofill-for-it-admins).

---

### <a name="general-availability---invite-internal-users-to-b2b-collaboration"></a>Disponibilidad general: invitación a usuarios internos para la colaboración B2B

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Ahora los clientes pueden invitar a los invitados internos a usar la colaboración B2B en lugar de enviar una invitación a una cuenta interna existente. Esto permite a los clientes mantener el identificador de objeto, el UPN, la pertenencia a grupos y las asignaciones de aplicaciones del usuario. [Más información](../external-identities/invite-internal-users.md).

---

### <a name="general-availability---domain-name-administrator-built-in-role"></a>Disponibilidad general: rol integrado de administrador de nombres de dominio

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Los usuarios con este rol pueden administrar (leer, agregar, comprobar, actualizar y eliminar) los nombres de dominio. También pueden leer información de directorio sobre usuarios, grupos y aplicaciones, ya que estos objetos tienen dependencias de dominio. 

En entornos locales, los usuarios con este rol pueden configurar nombres de dominio para la federación, de modo que los usuarios asociados siempre se autentiquen de forma local. Estos usuarios pueden iniciar sesión en servicios basados en Azure AD con sus contraseñas locales a través del inicio de sesión único. La configuración de federación se debe sincronizar a través de Azure AD Connect, por lo que los usuarios también tienen permisos para administrar Azure AD Connect. [Más información](../roles/permissions-reference.md#domain-name-administrator).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---february-2021"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (febrero de 2021)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En febrero de 2021, agregamos 37 aplicaciones nuevas a la galería de aplicaciones con compatibilidad de federación:

[Extensión de Loop Messenger](https://loopworks.com/loop-flow-messenger/), [Adaptador de Silverfort para Azure AD](http://www.silverfort.com/), [Interplay Learning](https://skilledtrades.interplaylearning.com/#login), [Nura Space](https://dashboard.nuraspace.com/login), [Yooz EU](https://eu1.getyooz.com/?kc_idp_hint=microsoft), [UXPressia](https://uxpressia.com/users/sign-in), [introDus Pre- and Onboarding Platform](http://app.introdus.dk/login), [Happybot](https://login.microsoftonline.com/organizations/oauth2/v2.0/authorize?client_id=34353e1e-dfe5-4d2f-bb09-2a5e376270c8&response_type=code&redirect_uri=https://api.happyteams.io/microsoft/integrate&response_mode=query&scope=offline_access%20User.Read%20User.Read.All), [LeaksID](https://app.leaksid.com/), [ShiftWizard](http://www.shiftwizard.com/), [PingFlow SSO](https://app.pingview.io/), [Swiftlane](https://admin.swiftlane.com/login), [Quasydoc SSO](https://www.quasydoc.eu/login), [Fenwick Gold Account](https://businesscentral.dynamics.com/), [SeamlessDesk](https://www.seamlessdesk.com/login), [Learnsoft LMS y TMS](http://www.learnsoft.com/), [P-TH+](https://p-th.jp/), [myViewBoard](https://api.myviewboard.com/auth/microsoft/), [Tartabit IoT Bridge](https://bridge-us.tartabit.com/), [AKASHI](../saas-apps/akashi-tutorial.md), [Rewatch](../saas-apps/rewatch-tutorial.md), [Zuddl](../saas-apps/zuddl-tutorial.md), [Parkalot - Car park management](../saas-apps/parkalot-car-park-management-tutorial.md), [HSB ThoughtSpot](../saas-apps/hsb-thoughtspot-tutorial.md), [IBMid](../saas-apps/ibmid-tutorial.md), [SharingCloud](../saas-apps/sharingcloud-tutorial.md), [PoolParty Semantic Suite](../saas-apps/poolparty-semantic-suite-tutorial.md), [GlobeSmart](../saas-apps/globesmart-tutorial.md), [Samsung Knox and Business Services](../saas-apps/samsung-knox-and-business-services-tutorial.md), [Penji](../saas-apps/penji-tutorial.md), [Kendis- Scaling Agile Platform](../saas-apps/kendis-scaling-agile-platform-tutorial.md), [Maptician](../saas-apps/maptician-tutorial.md), [Olfeo SAAS](../saas-apps/olfeo-saas-tutorial.md), [Sigma Computing](../saas-apps/sigma-computing-tutorial.md), [plataforma de administración de permisos de CloudKnox](../saas-apps/cloudknox-permissions-management-platform-tutorial.md), [Klaxoon SAML](../saas-apps/klaxoon-saml-tutorial.md), [Enablon](../saas-apps/enablon-tutorial.md)

También puede consultar la documentación de todas ellas aquí: https://aka.ms/AppsTutorial.

Para incluir su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

--- 

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---february-2021"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD (febrero de 2021)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Atea](../saas-apps/atea-provisioning-tutorial.md)
- [Getabstract](../saas-apps/getabstract-provisioning-tutorial.md)
- [HelloID](../saas-apps/helloid-provisioning-tutorial.md)
- [Hoxhunt](../saas-apps/hoxhunt-provisioning-tutorial.md)
- [Iris Intranet](../saas-apps/iris-intranet-provisioning-tutorial.md)
- [Preciate](../saas-apps/preciate-provisioning-tutorial.md)

Para más información, consulte [Qué es el aprovisionamiento automatizado de usuarios de aplicaciones SaaS en Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="general-availability---10-azure-active-directory-roles-now-renamed"></a>Disponibilidad general: se ha cambiado el nombre de 10 roles de Azure Active Directory

**Tipo:** Característica modificada  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Se ha cambiado el nombre de 10 roles integrados de Azure AD para que estén alineados en el [centro de administración de Microsoft 365](/microsoft-365/admin/microsoft-365-admin-center-preview), [el portal de Azure AD](https://portal.azure.com/) y [Microsoft Graph](https://developer.microsoft.com/graph/). Para más información sobre los nuevos roles, consulte [Permisos de roles de administrador en Azure Active Directory](../roles/permissions-reference.md#all-roles).

![Tabla que muestra los nombres de rol en MS Graph API y Azure Portal, y el nombre final propuesto en la API, Azure Portal y Mac.](media/whats-new/roles-table-rbac.png)

---

### <a name="new-company-branding-in-mfasspr-combined-registration"></a>Nueva personalización de marca de la empresa en el registro combinado de MFA/SSPR

**Tipo:** Característica modificada  
**Categoría de servicio:** Experiencia y administración de usuarios  
**Funcionalidad del producto:** Experiencias de usuario final
 
En el pasado, no se usaban los logotipos de empresa en las páginas de inicio de sesión de Azure Active Directory. Ahora, la personalización de marca de la empresa se encuentra en la parte superior izquierda del registro combinado de MFA/SSPR. La personalización de marca de la empresa también se incluye en Mis inicios de sesión y en la página de información de seguridad. [Más información](../fundamentals/customize-branding.md).

---

### <a name="general-availability---second-level-manager-can-be-set-as-alternate-approver"></a>Disponibilidad general: el administrador de segundo nivel se puede establecer como aprobador alternativo

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 
Ahora, en la administración de derechos, se encuentra disponible una opción adicional al seleccionar aprobadores. Si selecciona "Administrador como aprobador" para el primer aprobador, tendrá otra opción disponible para elegir en el campo de aprobador alternativo, "Administrador de segundo nivel como aprobador alternativo". Si selecciona esta opción, debe agregar un aprobador de reserva al que reenviar la solicitud en caso de que el sistema no encuentre el administrador de segundo nivel. [Más información](../governance/entitlement-management-access-package-approval-policy.md#alternate-approvers).
 
---

### <a name="authentication-methods-activity-dashboard"></a>Panel Actividad de los métodos de autenticación

**Tipo:** Característica modificada  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 

El panel Actividad de los métodos de autenticación actualizado proporciona a los administradores una visión general de la actividad de registro y uso de los métodos de autenticación en su inquilino. El informe resume el número de usuarios registrados para cada método y los métodos que se usan durante el inicio de sesión y el restablecimiento de contraseña. [Más información](../authentication/howto-authentication-methods-activity.md).
 
---

### <a name="refresh-and-session-token-lifetimes-configurability-in-configurable-token-lifetime-ctl-are-retired"></a>Se retira la capacidad de configuración de la vigencia de los tokens de sesión y actualización en la directiva de vigencia configurable del token

**Tipo:** Obsoleto  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Autenticación de usuarios
 
Se retira la capacidad de configuración de la vigencia de los tokens de sesión y actualización en la directiva de vigencia configurable del token. Azure Active Directory ya no respeta la configuración de los tokens de actualización y de sesión en las directivas existentes. [Más información](../develop/active-directory-configurable-token-lifetimes.md#token-lifetime-policies-for-refresh-tokens-and-session-tokens).
 
---
 
## <a name="january-2021"></a>Enero de 2021

### <a name="secret-token-will-be-a-mandatory-field-when-configuring-provisioning"></a>El token secreto será un campo obligatorio al configurar el aprovisionamiento

**Tipo:** Plan de cambio  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Anteriormente, el campo de token secreto podía quedarse vacío al configurar el aprovisionamiento en la aplicación personalizada/BYOA. Esta función estaba pensada para usarse únicamente para realizar pruebas. Actualizaremos la interfaz de usuario para que el campo sea obligatorio. 

Los clientes pueden evitar este requisito para fines de prueba mediante una marca de características en la dirección URL del explorador. [Más información](../app-provisioning/use-scim-to-provision-users-and-groups.md#authorization-to-provisioning-connectors-in-the-application-gallery).
 
---

### <a name="public-preview---customize-and-configure-android-shared-devices-for-frontline-workers-at-scale"></a>Versión preliminar pública: personalización y configuración de dispositivos compartidos Android a gran escala para trabajadores de primera línea

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración y registro de dispositivos  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Los equipos de Azure AD y Microsoft Endpoint Manager se han combinado para ofrecer la funcionalidad de personalizar, escalar y proteger los dispositivos de los trabajadores de primera línea.

Las siguientes funcionalidades en versión preliminar le permitirán hacer lo siguiente:
- Aprovisionar dispositivos compartidos Android a gran escala con Microsoft Endpoint Manager
- Proteger el acceso de los trabajadores por turnos mediante el acceso condicional basado en dispositivos
- Personalizar las experiencias de inicio de sesión de los trabajadores por turnos con Managed Home Screen

Para más información, consulte [Personalización y configuración de dispositivos compartidos a gran escala para trabajadores de primera línea](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/customize-and-configure-shared-devices-for-firstline-workers-at/ba-p/1751708).

---

### <a name="public-preview---provisioning-logs-can-now-be-downloaded-as-a-csv-or-json"></a>Versión preliminar pública: los registros de aprovisionamiento ahora se pueden descargar como CSV o JSON

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Los clientes pueden descargar los registros de aprovisionamiento como un archivo CSV o JSON desde la interfaz de usuario y con Graph API. Para más información, vea [Informes de aprovisionamiento en el portal de Azure Active Directory](../reports-monitoring/concept-provisioning-logs.md).

---

### <a name="public-preview---assign-cloud-groups-to-azure-ad-custom-roles-and-admin-unit-scoped-roles"></a>Versión preliminar pública: asignación de grupos en la nube a roles con ámbito de unidad de administración y roles personalizados de Azure AD

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Los clientes pueden asignar un grupo en la nube a un rol con ámbito de unidad de administración o a roles personalizados de Azure AD. Para obtener información sobre cómo usar esta característica, vea [Uso de grupos en la nube para administrar asignaciones de roles en Azure Active Directory](../roles/groups-concept.md).

---

### <a name="general-availability---azure-ad-connect-cloud-sync-previously-known-as-cloud-provisioning"></a>Disponibilidad general: sincronización en la nube de Azure AD Connect (conocida anteriormente como aprovisionamiento en la nube)

**Tipo:** Nueva característica  
**Categoría del servicio:** sincronización en la nube de Azure AD Connect  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
La sincronización en la nube de Azure AD Connect ahora está disponible con carácter general para todos los clientes.

La nube de Azure AD Connect transfiere el trabajo pesado de la lógica de transformación a la nube, lo que reduce la superficie local. Además, hay disponibles varias implementaciones de agentes ligeros para una mayor disponibilidad de la sincronización. [Más información](https://aka.ms/cloudsyncGA).
 
---
### <a name="general-availability---attack-simulation-administrator-and-attack-payload-author-built-in-roles"></a>Disponibilidad general: roles integrados Administrador de simulación de ataque y Autor de carga de ataque

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Hay disponibles dos nuevos roles en el control de acceso basado en rol para asignar a los usuarios, que son Administrador de simulación de ataque y Autor de carga de ataque. 

Los usuarios del rol [Administrador de simulación de ataque](../roles/permissions-reference.md#attack-simulation-administrator) tienen acceso a todas las simulaciones del inquilino y pueden hacer lo siguiente:
- Crear y administrar todos los aspectos de la creación de simulaciones de ataque
- Iniciar y programar una simulación
-  Revisar los resultados de la simulación 

Los usuarios del rol [Autor de carga de ataque](../roles/permissions-reference.md#attack-payload-author) pueden crear cargas de ataque, pero no iniciarlas ni programarlas. Las cargas de ataque están disponibles para todos los administradores del inquilino, que pueden usarlas para crear una simulación.

---

### <a name="general-availability---usage-summary-reports-reader-built-in-role"></a>Disponibilidad general: rol integrado Lector de informes de resumen de uso

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Los usuarios con el rol Lector de informes de resumen de uso pueden acceder a los datos agregados de nivel de inquilino y a la información asociada del Centro de administración de Microsoft 365 para obtener la puntuación de uso y productividad. Sin embargo, no pueden acceder a los detalles ni a la información de nivel de usuario. 

En el Centro de administración de Microsoft 365 de los dos informes, se diferencia entre los datos agregados de nivel de inquilino y los detalles de nivel de usuario. Este rol agrega una capa adicional de protección a los datos de identificación de usuarios individuales. [Más información](../roles/permissions-reference.md#usage-summary-reports-reader).

---

### <a name="general-availability---require-app-protection-policy-grant-in-azure-ad-conditional-access"></a>Disponibilidad general: concesión Requerir la directiva de protección de aplicaciones en el acceso condicional de Azure AD

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
La concesión de acceso condicional de Azure AD para "Requerir la directiva de protección de aplicaciones" ahora se encuentra en disponibilidad general. 

La directiva ofrece las siguientes funcionalidades:
- Permite el acceso solo cuando se usa una aplicación móvil que admite Intune App Protection.
- Permite el acceso solo cuando un usuario tiene una directiva de protección de aplicaciones de Intune que se entrega a la aplicación móvil.

Obtenga más información [aquí](../conditional-access/app-protection-based-conditional-access.md) sobre cómo configurar una directiva de acceso condicional para la protección de aplicaciones.
 
---

### <a name="general-availability---email-one-time-passcode"></a>Disponibilidad general: código de acceso de un solo uso de correo electrónico

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
El código de acceso de un solo uso de correo electrónico permite a las organizaciones de todo el mundo colaborar con cualquier persona mediante el envío de un vínculo o una invitación por correo electrónico. Los usuarios invitados pueden comprobar su identidad con el código de acceso de un solo uso enviado a su correo electrónico para acceder a los recursos de su asociado. [Más información](../external-identities/one-time-passcode.md). 
 
---

 ### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---january-2021"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD (enero de 2021)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:
- [Fortes Change Cloud](../saas-apps/fortes-change-cloud-provisioning-tutorial.md)
- [Gtmhub](../saas-apps/gtmhub-provisioning-tutorial.md)
- [monday.com](../saas-apps/mondaycom-provisioning-tutorial.md)
- [Splashtop](../saas-apps/splashtop-provisioning-tutorial.md)
- [Templafy OpenID Connect](../saas-apps/templafy-openid-connect-provisioning-tutorial.md)
- [WEDO](../saas-apps/wedo-provisioning-tutorial.md)

Para más información, vea [Qué es el aprovisionamiento automatizado de usuarios de aplicaciones SaaS en Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---january-2021"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, enero de 2021

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En enero de 2021, agregamos 29 aplicaciones nuevas a la galería de aplicaciones con compatibilidad de federación:

[mySCView](https://dev.myscview.com/), [Talentech](https://talentech.com/contact/), [Bipsync](https://www.bipsync.com/), [OroTimesheet](https://app.orotimesheet.com/login.php), [Mio](https://app.m.io/auth/install/microsoft?scopetype=hub), [Sovelto Easy](https://login.soveltoeasy.fi/), [Supportbench](https://account.supportbench.net/agent/login/),[Bienvenue Formation](https://formation.bienvenue.pro/login), [AIDA Healthcare SSO](https://aidaforparents.com/login/organizations), [International SOS Assistance Products](../saas-apps/international-sos-assistance-products-tutorial.md), [NAVEX One](../saas-apps/navex-one-tutorial.md), [LabLog](../saas-apps/lablog-tutorial.md), [Oktopost SAML](../saas-apps/oktopost-saml-tutorial.md), [EPHOTO DAM](../saas-apps/ephoto-dam-tutorial.md), [Notion](../saas-apps/notion-tutorial.md), [Syndio](../saas-apps/syndio-tutorial.md), [Yello Enterprise](../saas-apps/yello-enterprise-tutorial.md), [Timeclock 365 SAML](../saas-apps/timeclock-365-saml-tutorial.md), [Nalco E-data](https://www.ecolab.com/), [Vacancy Filler](https://app.vacancy-filler.co.uk/VFMVC/Account/Login), [Synerise AI Growth Ecosystem](../saas-apps/synerise-ai-growth-ecosystem-tutorial.md), [Imperva Data Security](../saas-apps/imperva-data-security-tutorial.md), [Illusive Networks](../saas-apps/illusive-networks-tutorial.md), [Proware](../saas-apps/proware-tutorial.md), [Splan Visitor](../saas-apps/splan-visitor-tutorial.md), [Aruba User Experience Insight](../saas-apps/aruba-user-experience-insight-tutorial.md), [Contentsquare SSO](../saas-apps/contentsquare-sso-tutorial.md), [Perimeter 81](../saas-apps/perimeter-81-tutorial.md), [Burp Suite Enterprise Edition](../saas-apps/burp-suite-enterprise-edition-tutorial.md)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí https://aka.ms/AzureADAppRequest. 

---

### <a name="public-preview---second-level-manager-can-be-set-as-alternate-approver"></a>Versión preliminar pública: el administrador de segundo nivel se puede establecer como aprobador alternativo

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 
Ahora, en la administración de derechos, se encuentra disponible una opción adicional al seleccionar aprobadores. Si selecciona "Administrador como aprobador" para el primer aprobador, tendrá otra opción disponible para elegir en el campo de aprobador alternativo, "Administrador de segundo nivel como aprobador alternativo". Si selecciona esta opción, debe agregar un aprobador de reserva al que reenviar la solicitud en caso de que el sistema no encuentre el administrador de segundo nivel. [Más información](../governance/entitlement-management-access-package-approval-policy.md#alternate-approvers)
 
---

### <a name="general-availability---navigate-to-teams-directly-from-my-access-portal"></a>Disponibilidad general: navegación a Teams directamente desde el portal Mi acceso

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 
Ahora puede iniciar Teams directamente desde el portal Mi acceso. 

Para ello, inicie sesión en Mi acceso (https://myaccess.microsoft.com/) ), diríjase a "Paquetes de acceso" y, a continuación, vaya a la pestaña "Activos" para ver todos los paquetes de acceso a los que ya tiene acceso. Al expandir el paquete de acceso seleccionado y mantener el ratón sobre Teams, puede iniciarla al hacer clic en el botón "Abrir". [Más información](../governance/entitlement-management-request-access.md).
 
---

### <a name="improved-logging--end-user-prompts-for-risky-guest-users"></a>Mensajes de registro y usuario final mejorados para los usuarios invitados de riesgo

**Tipo:** Característica modificada  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

Se han actualizado los mensajes de registro y usuario final para los usuarios invitados de riesgo. Más información en [Protección de identidades y usuarios de Colaboración B2B](../identity-protection/concept-identity-protection-b2b.md).
 
---
 
## <a name="december-2020"></a>Diciembre de 2020

### <a name="public-preview---azure-ad-b2c-phone-sign-up-and-sign-in-using-built-in-policy"></a>Versión preliminar: registro e inicio de sesión mediante teléfono en Azure AD B2C con una directiva integrada

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
El registro e inicio de sesión mediante teléfono en B2C permiten a los administradores de TI y desarrolladores de organizaciones dejar que sus usuarios finales inicien sesión y se registren con un número de teléfono en los flujos de usuario. Lea [Configuración del registro e inicio de sesión telefónico para flujos de usuario (versión preliminar)](../../active-directory-b2c/phone-authentication-user-flows.md) para obtener más información.

---

### <a name="general-availability---security-defaults-now-enabled-for-all-new-tenants-by-default"></a>Disponibilidad general: valores predeterminados de seguridad ahora habilitados para todos los inquilinos nuevos de forma predeterminada

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Para proteger las cuentas de usuario, todos los inquilinos nuevos creados el 12 de noviembre de 2020 o después vendrán con Valores predeterminados de seguridad habilitado. Valores predeterminados de seguridad aplica varias directivas, entre las que se incluyen:
- Requiere que todos los usuarios y administradores se registren para MFA mediante la aplicación de Microsoft Authenticator
- Requiere que los roles de administrador críticos usen MFA cada vez que inicien sesión. A todos los demás usuarios se les pedirá MFA cuando sea necesario. 
- La autenticación heredada se bloqueará en todo el inquilino. 

Para obtener más información, consulte [¿Cuáles son los valores de seguridad predeterminados?](../fundamentals/concept-fundamentals-security-defaults.md)

---

### <a name="general-availability---support-for-groups-with-up-to-250k-members-in-aadconnect"></a>Disponibilidad general: compatibilidad con grupos de hasta 250 000 miembros en AADConnect

**Tipo:** Característica modificada  
**Categoría del servicio:** AD Connect  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Microsoft ha implementado un nuevo punto de conexión (API) para Azure AD Connect que mejora el rendimiento de las operaciones del servicio de sincronización para Azure Active Directory. Al usar el nuevo [punto de conexión V2](../hybrid/how-to-connect-sync-endpoint-api-v2.md), experimentará mejoras de rendimiento notables en la exportación y la importación respecto a Azure AD. Este nuevo punto de conexión admite los siguientes escenarios:

- Grupos de sincronización de hasta 250 000 miembros
- Mejoras en el rendimiento de la exportación y la importación a Azure AD

---

### <a name="general-availability---entitlement-management-available-for-tenants-in-azure-china-cloud"></a>Disponibilidad general: administración de derechos disponible para los inquilinos en la nube de Azure China

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 

Las funcionalidades de administración de derechos están ahora disponibles para todos los inquilinos en la nube de Azure China. Para obtener información, visite nuestro sitio de [documentación de Identity Governance](https://docs.azure.cn/zh-cn/active-directory/governance/).

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---december-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD: diciembre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Bizagi Studio para la automatización de procesos digitales](../saas-apps/bizagi-studio-for-digital-process-automation-provisioning-tutorial.md)
- [CybSafe](../saas-apps/cybsafe-provisioning-tutorial.md)
- [GroupTalk](../saas-apps/grouptalk-provisioning-tutorial.md)
- [PaperCut Cloud Print Management](../saas-apps/papercut-cloud-print-management-provisioning-tutorial.md)
- [Parsable](../saas-apps/parsable-provisioning-tutorial.md)
- [Shopify Plus](../saas-apps/shopify-plus-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---december-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: diciembre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En diciembre de 2020 se han agregado las siguientes 18 aplicaciones nuevas a la galería de aplicaciones con compatibilidad con la federación:

[AwareGo](../saas-apps/awarego-tutorial.md), [HowNow SSO](https://gethownow.com/), [ZyLAB ONE Legal Hold](https://www.zylab.com/en/product/legal-hold), [Guider](http://www.guider-ai.com/), [Softcrisis](https://www.softcrisis.se/sv/), [Pims 365](http://www.omega365.com/pims), [InformaCast](../saas-apps/informacast-tutorial.md), [RetrieverMediaDatabase](../saas-apps/retrievermediadatabase-tutorial.md), [vonage](../saas-apps/vonage-tutorial.md), [Count Me In - Operations Dashboard](../saas-apps/count-me-in-operations-dashboard-tutorial.md), [ProProfs Knowledge Base](../saas-apps/proprofs-knowledge-base-tutorial.md), [RightCrowd Workforce Management](../saas-apps/rightcrowd-workforce-management-tutorial.md), [JLL TRIRIGA](../saas-apps/jll-tririga-tutorial.md), [Shutterstock](../saas-apps/shutterstock-tutorial.md), [FortiWeb Web Application Firewall](../saas-apps/linkedin-talent-solutions-tutorial.md), [LinkedIn Talent Solutions](../saas-apps/linkedin-talent-solutions-tutorial.md), [Equinix Federation App](../saas-apps/equinix-federation-app-tutorial.md) y [KFAdvance](../saas-apps/kfadvance-tutorial.md)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí https://aka.ms/AzureADAppRequest.

---

### <a name="navigate-to-teams-directly-from-my-access-portal"></a>Navegación a Teams directamente desde el portal Mi acceso

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso a usuarios **Funcionalidad del producto:** Administración de derechos

Ahora puede iniciar Teams directamente desde el portal Mi acceso. Para ello, inicie sesión en [Mi acceso](https://myaccess.microsoft.com/), diríjase a **Paquetes de acceso** y, a continuación, vaya a la pestaña **Activos** para ver todos los paquetes de acceso a los que ya tiene acceso. Al expandir el paquete de acceso y mantener el ratón sobre Teams, puede iniciarla al hacer clic en el botón **Abrir**. 

Para obtener más información sobre el uso del portal Mi acceso, visite [Solicitud de acceso a un paquete de acceso en la administración de derechos de Azure AD](../governance/entitlement-management-request-access.md#sign-in-to-the-my-access-portal).

---

### <a name="public-preview---second-level-manager-can-be-set-as-alternate-approver"></a>Versión preliminar pública: el administrador de segundo nivel se puede establecer como aprobador alternativo

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos

Ahora está disponible una opción adicional en el proceso de aprobación de la administración de derechos. Si selecciona Administrador como aprobador para el primer aprobador, tendrá otra opción disponible para elegir en el campo de aprobador alternativo, Administrador de segundo nivel como aprobador alternativo. Cuando selecciona esta opción, debe agregar un aprobador de reserva al que reenviar la solicitud en caso de que el sistema no encuentre el administrador de segundo nivel.

Para obtener más información, diríjase a [Cambio de la configuración de aprobación para un paquete de acceso de administración de derechos de Azure AD](../governance/entitlement-management-access-package-approval-policy.md#alternate-approvers).

--- 

## <a name="november-2020"></a>Noviembre de 2020

### <a name="azure-active-directory-tls-10-tls-11-and-3des-deprecation"></a>Entrada en desuso de TLS 1.0, TLS 1.1 y 3DES en Azure Active Directory

**Tipo:** Plan de cambio  
**Categoría del servicio:** todas las aplicaciones de Azure AD  
**Funcionalidad del producto:** Estándares

Azure Active Directory dejará de usar los siguientes protocolos en las regiones de todo el mundo de Azure Active Directory a partir del 30 de junio de 2021:

- TLS 1.0
- TLS 1.1
- Conjunto de cifrado 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Los entornos afectados son:
- Nube comercial de Azure
- Office 365 GCC y WW

Para obtener instrucciones para quitar las dependencias de los protocolos en desuso, consulte [Habilitar la compatibilidad con TLS 1.2 en el entorno para la desuso de TLS 1.1 y 1.0 de Azure AD](/troubleshoot/azure/active-directory/enable-support-tls-environment).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---november-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, noviembre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En noviembre de 2020 se han agregado las siguientes 52 aplicaciones nuevas a la galería de aplicaciones con compatibilidad con la federación:

[Travel & Expense Management](https://app.expenseonce.com/Account/Login), [Tribeloo](../saas-apps/tribeloo-tutorial.md), [Itslearning File Picker](https://pmteam.itslearning.com/), [Crises Control](../saas-apps/crises-control-tutorial.md), [CourtAlert](https://www.courtalert.com/), [StealthMail](https://stealthmail.com/), [Edmentum - Study Island](https://app.studyisland.com/cfw/login/), [Virtual Risk Manager](../saas-apps/virtual-risk-manager-tutorial.md), [TIMU](../saas-apps/timu-tutorial.md), [Looker Analytics Platform](../saas-apps/looker-analytics-platform-tutorial.md), [Talview - Recruit](https://recruit.talview.com/login), Real Time Translator, [Klaxoon](https://access.klaxoon.com/login), [Podbean](../saas-apps/podbean-tutorial.md), [zcal](https://zcal.co/signup), [expensemanager](https://api.expense-manager.com/), [Netsparker Enterprise](../saas-apps/netsparker-enterprise-tutorial.md), [En-trak Tenant Experience Platform](https://portal.en-trak.app/), [Appian](../saas-apps/appian-tutorial.md), [Panorays](../saas-apps/panorays-tutorial.md), [Builterra](https://portal.builterra.com/), [EVA Check-in](https://my.evacheckin.com/organization), [HowNow WebApp SSO](../saas-apps/hownow-webapp-sso-tutorial.md), [Coupa Risk Assess](../saas-apps/coupa-risk-assess-tutorial.md), [Lucid (All Products)](../saas-apps/lucid-tutorial.md), [GoBright](https://portal.brightbooking.eu/), [SailPoint IdentityNow](../saas-apps/sailpoint-identitynow-tutorial.md),[Resource Central](../saas-apps/resource-central-tutorial.md), [UiPathStudioO365App](https://www.uipath.com/product/platform), [Jedox](../saas-apps/jedox-tutorial.md), [Cequence Application Security](../saas-apps/cequence-application-security-tutorial.md), [PerimeterX](../saas-apps/perimeterx-tutorial.md), [TrendMiner](../saas-apps/trendminer-tutorial.md), [Lexion](../saas-apps/lexion-tutorial.md), [WorkWare](../saas-apps/workware-tutorial.md), [ProdPad](../saas-apps/prodpad-tutorial.md), [AWS ClientVPN](../saas-apps/aws-clientvpn-tutorial.md), [AppSec Flow SSO](../saas-apps/appsec-flow-sso-tutorial.md), [Luum](../saas-apps/luum-tutorial.md), [Freight Measure](https://www.gpcsl.com/freight.html), [Terraform Cloud](../saas-apps/terraform-cloud-tutorial.md), [Nature Research](../saas-apps/nature-research-tutorial.md), [Play Digital Signage](https://login.playsignage.com/login), [RemotePC](../saas-apps/remotepc-tutorial.md), [Prolorus](../saas-apps/prolorus-tutorial.md), [Hirebridge ATS](../saas-apps/hirebridge-ats-tutorial.md), [Teamgage](https://www.teamgage.com/Account/ExternalLoginAzure), [Roadmunk](../saas-apps/roadmunk-tutorial.md), [Sunrise Software Relations CRM](https://cloud.relations-crm.com/), [Procaire](../saas-apps/procaire-tutorial.md), [Mentor® by eDriving: Business](https://www.edriving.com/), [Gradle Enterprise](https://gradle.com/)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí https://aka.ms/AzureADAppRequest.

---

### <a name="public-preview---custom-roles-for-enterprise-apps"></a>Versión preliminar pública: roles personalizados para aplicaciones empresariales

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
 [Los roles de RBAC personalizados para la administración delegada de aplicaciones empresariales](../roles/custom-available-permissions.md) ahora están en versión preliminar pública. Estos nuevos permisos se basan en los roles personalizados para la administración del registro de aplicaciones, lo que permite un control exhaustivo sobre el acceso que tienen los administradores. Con el tiempo, se publicarán permisos adicionales para la administración delegada de Azure AD.

Estos son algunos escenarios de delegación comunes:
- asignación de usuarios y grupos que pueden acceder a aplicaciones de inicio de sesión único basadas en SAML
- creación de aplicaciones de la galería de Azure AD
- actualización y lectura de configuraciones básicas de SAML para aplicaciones de inicio de sesión único basadas en SAML
- administración de certificados de firma para aplicaciones de inicio de sesión único basadas en SAML
- actualización de las direcciones de correo electrónico de las notificaciones de expiración de los certificados de inicio de sesión para aplicaciones de inicio de sesión único basadas en SAML
- actualización de la firma del token de SAML y del algoritmo de inicio de sesión para aplicaciones de inicio de sesión único basadas en SAML
- creación, eliminación y actualización de atributos y notificaciones de usuario de las aplicaciones de inicio de sesión único basadas en SAML
- capacidad de activar, desactivar y reiniciar los trabajos de aprovisionamiento
- actualizaciones en la asignación de atributos
- capacidad de leer la configuración de aprovisionamiento asociada al objeto
- capacidad de leer la configuración de aprovisionamiento asociada a la entidad de servicio
- capacidad de autorización del acceso a la aplicación para el aprovisionamiento

---

### <a name="public-preview---azure-ad-application-proxy-natively-supports-single-sign-on-access-to-applications-that-use-headers-for-authentication"></a>Versión preliminar pública: Application Proxy de Azure AD admite de manera nativa el acceso de inicio sesión único a las aplicaciones que usan encabezados para la autenticación

**Tipo:** Nueva característica  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso
 
Application Proxy de Azure Active Directory (Azure AD) admite de forma nativa el acceso de inicio sesión único a las aplicaciones que usan encabezados para la autenticación. Puede configurar los valores de encabezado que necesita la aplicación en Azure AD. Los valores de encabezado se envían a la aplicación a través de Application Proxy. Para más información, consulte [Inicio de sesión único basado en encabezados para aplicaciones locales con App Proxy de Azure AD (versión preliminar)](../manage-apps/application-proxy-configure-single-sign-on-with-headers.md).
 
---

### <a name="general-availability---azure-ad-b2c-phone-sign-up-and-sign-in-using-custom-policy"></a>Disponibilidad general: registro e inicio de sesión mediante teléfono en Azure AD B2C con una directiva personalizada

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Con el registro y el inicio de sesión mediante el número de teléfono, los desarrolladores y las empresas pueden permitir a sus clientes registrarse e iniciar sesión con una contraseña de un solo uso enviada al número de teléfono del usuario a través de un SMS. Esta característica también permite al cliente cambiar su número de teléfono si pierde el acceso a su dispositivo. Con la eficacia de las directivas personalizadas, permite a los desarrolladores y a las empresas comunicar su marca mediante la personalización de la página. Aprenda cómo [configurar el registro y el inicio de sesión en el teléfono con directivas personalizadas en Azure AD B2C](../../active-directory-b2c/phone-authentication-user-flows.md).
 
---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---november-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD: noviembre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Adobe Identity Management](../saas-apps/adobe-identity-management-provisioning-tutorial.md)
- [Blogin](../saas-apps/blogin-provisioning-tutorial.md)
- [Clarizen One](../saas-apps/clarizen-one-provisioning-tutorial.md)
- [Contentful](../saas-apps/contentful-provisioning-tutorial.md)
- [GitHub AE](../saas-apps/github-ae-provisioning-tutorial.md)
- [Playvox](../saas-apps/playvox-provisioning-tutorial.md)
- [PrinterLogic SaaS](../saas-apps/printer-logic-saas-provisioning-tutorial.md)
- [Tic - Tac Mobile](../saas-apps/tic-tac-mobile-provisioning-tutorial.md)
- [Visibly](../saas-apps/visibly-provisioning-tutorial.md)

Para más información, consulte [Qué es el aprovisionamiento automatizado de usuarios de aplicaciones SaaS en Azure AD](../app-provisioning/user-provisioning.md).
 
---

### <a name="public-preview---email-sign-in-with-proxyaddresses-now-deployable-via-staged-rollout"></a>Versión preliminar pública: el inicio de sesión mediante correo electrónico con direcciones proxy ahora se puede implementar mediante la implementación por fases

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Los administradores de inquilinos ahora pueden usar la implementación por fases para implementar el inicio de sesión mediante correo electrónico con direcciones proxy en grupos de Azure AD específicos. Esto puede ayudar a probar la característica antes de implementarla en todo el inquilino mediante la directiva de detección del dominio de inicio. Las instrucciones para implementar el inicio de sesión mediante correo electrónico con direcciones proxy con la implementación por fases se encuentran en la [documentación](../authentication/howto-authentication-use-email-signin.md).
 
---

### <a name="limited-preview---sign-in-diagnostic"></a>Versión preliminar limitada: diagnóstico de inicio de sesión

**Tipo:** Nueva característica  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
Con la versión preliminar inicial del diagnóstico de inicio de sesión, los administradores ahora pueden revisar los inicios de sesión de los usuarios. Los administradores pueden recibir detalles de importancia contextuales y específicos, e instrucciones sobre lo que ha sucedido durante un inicio de sesión y cómo solucionar los problemas. El diagnóstico está disponible tanto en el nivel de Azure AD como en las hojas de diagnóstico y resolución del acceso condicional. Los escenarios de diagnóstico admitidos en esta versión son el acceso condicional, Multi-Factor Authentication y el inicio de sesión correcto.

Para obtener más información, consulte [¿Qué es el diagnóstico de inicio de sesión en Azure AD?](../reports-monitoring/overview-sign-in-diagnostics.md)
 
---

### <a name="improved-unfamiliar-sign-in-properties"></a>Mejora de las propiedades de inicio de sesión desconocidas

**Tipo:** Característica modificada  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad

  Se han actualizado las detecciones de propiedades de inicio de sesión desconocidas. Es posible que los clientes perciban más detecciones de propiedades de inicio de sesión desconocidas de alto riesgo. Para más información, consulte [¿Qué es el riesgo?](../identity-protection/concept-identity-protection-risks.md)
 
---

### <a name="public-preview-refresh-of-cloud-provisioning-agent-now-available-version-112810"></a>Ya está disponible la actualización de la versión preliminar pública del agente de aprovisionamiento en la nube (versión: 1.1.281.0)

**Tipo:** Característica modificada  
**Categoría del servicio:** Aprovisionamiento en la nube de Azure AD  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El agente de aprovisionamiento en la nube se ha publicado en versión preliminar pública y ahora está disponible mediante el portal. Esta versión contiene varias mejoras, entre las que se incluyen la compatibilidad con GMSA para los dominios, lo que proporciona una mejor seguridad, ciclos de sincronización inicial mejorados y compatibilidad con grupos grandes. Consulte el [historial](../app-provisioning/provisioning-agent-release-version-history.md) de lanzamiento de versiones para más información. 
 
---

### <a name="bitlocker-recovery-key-api-endpoint-now-under-informationprotection"></a>El punto de conexión de la API de la clave de recuperación de BitLocker ahora está en /informationProtection

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso de dispositivos  
**Funcionalidad del producto:** Administración del ciclo de vida de dispositivos
 
Anteriormente, podía recuperar las claves de BitLocker mediante el punto de conexión /bitlocker. Finalmente vamos a dejar de usar este punto de conexión y los clientes deberían empezar a consumir la API que ahora está en /informationProtection. 

Consulte [Tipo de recurso bitlockerRecoveryKey](/graph/api/resources/bitlockerrecoverykey?view=graph-rest-beta&preserve-view=true) para conocer las actualizaciones de la documentación para reflejar estos cambios.

---

### <a name="general-availability-of-application-proxy-support-for-remote-desktop-services-html5-web-client"></a>Disponibilidad general de la compatibilidad de Application Proxy con el cliente web HTML5 de los Servicios de Escritorio remoto

**Tipo:** Característica modificada  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso
 
La compatibilidad de Application Proxy de Azure AD con el cliente web de los Servicios de Escritorio remoto (RDS) ahora está disponible con carácter general. El cliente web de RDS permite a los usuarios acceder a la infraestructura de Escritorio remoto mediante cualquier explorador compatible con HTLM5, como Microsoft Edge, Internet Explorer 11, Google Chrome, etc. Los usuarios pueden interactuar con escritorios o aplicaciones remotos como lo harían con un dispositivo local desde cualquier lugar. 

Mediante el uso de Application Proxy de Azure AD, puede aumentar la seguridad de la implementación de RDS mediante la exigencia de directivas de acceso condicional y autenticación previa para todos los tipos de aplicaciones cliente enriquecidas. Para más información, consulte [Publicación del Escritorio Remoto con Application Proxy de Azure AD](../manage-apps/application-proxy-integrate-with-remote-desktop-services.md).
 
---

### <a name="new-enhanced-dynamic-group-service-is-in-public-preview"></a>El nuevo servicio de grupo dinámico mejorado está en versión preliminar pública

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración
 
El servicio de grupo dinámico mejorado está ahora en versión preliminar pública. Los nuevos clientes que creen grupos dinámicos en sus inquilinos utilizarán el nuevo servicio. El tiempo necesario para crear un grupo dinámico será proporcional al tamaño del grupo que se va a crear en lugar de al tamaño del inquilino. Esta actualización mejorará significativamente el rendimiento en inquilinos grandes cuando los clientes crean grupos más pequeños. 

El nuevo servicio también pretende completar la adición y eliminación de miembros debidas a cambios en los atributos en pocos minutos. Además, los errores de procesamiento únicos no bloquearán el procesamiento del inquilino. Para más información sobre la creación de grupos dinámicos, consulte nuestra [documentación](../enterprise-users/groups-create-rule.md).
 
---
